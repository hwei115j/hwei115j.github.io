---
title: 用rclone的筆記
date: 2019-02-12 01:20:07
categories:
    - 筆記
tags:
    - 筆記
    - 備份
---

## 前言

還是記個筆記吧

暑假時用的rclone，寒假就全忘光了....
雖然有存當初的網址，但重新掌握還是需要時間
加上這次學了不少新東西，還是記個筆記比較實在...

## 使用方法

        rclone 動作 來源地址 目標地址 flag

##  常用動作
``` bash
    config  (設定)
    ls      (列出目錄與子目錄底下全部的元素)
    lsd     (只列出目錄底下的元素)
    copy    (複製，略過已經複製過的)
    sync    (同步，以來源為準，目標少的會複製、多的會刪除)
    來源位置&目標位置可為 本機位置 或 遠端位置
```
flag常用的有
```bash
    -v
    （顯示完整資訊，想要顯示進度要有這項）
    --bwlimit xxxxK/M/G
    （限制上傳速度，後面是單位可為K/M/G，例如：--bwlimit 8000k)
    --checksum
    （上傳時做完整檢查，以確保來源與目標完全相同(hash?)，需要注意的是，過程中硬碟會不停的讀取且速度會降低）
    --size-only
    （上傳時只比對檔案的大小，相對checksum快，但可能有錯）
    --dry-run
    （試跑一次，看看會對目標做什麼變動）
    --transfers int
    (一次傳送多少個檔案，預設為4)
```
範例
```bash
     rclone copy D1: D2: -v -size-only -bwlimit 1800k
     (從 D1: 複製到 D2:，顯示完整資訊 只比對雙方檔案大小 上傳速度1800KB/s)
     rclone sync D1:test/ D2:test/ -v -size-only
     (從 D1: 同步到 D2:，顯示完整資訊 只比對雙方檔案大小)
```
## 過濾器

rclone 不能直接在路徑上使用萬用字元，要配合一種flag來使用

例如不能用 `D1:test/*.jpg` 來表示 `D1:test/`  目錄下的全部jpg檔要使用 `D1:test/ --include *.jpg` 來表示
其中規則應該是支援正規表達式的，不過我對正規不太熟.....

因為我對這也不熟，所以我就直接用官網的範例了....
如果模式以"/"開頭，代表他在目錄的頂層，不往下搜尋

例如:
```bash
    rclone ls D1: --include *.jpg
    會列出:
        D1:1.jpg
        D1:2.jpg
    但也會列出:
        D1:test/1.jpg
        D1:test/2.jpg
    如果改成：
        rclone ls D1: --include /*.jpg
    則只會印出
        D1:1.jpg
        D1:2.jpg
```
p.s. windows v1.44好像沒辦法用"/"?

## 結合管線使用

這不做筆記我一定會忘掉......

rclone 有兩個指令:

    cat
    rcat

cat 和 linux 上的基本一樣，就是把檔案從stdout印出來，不過rclone cat 可以印目錄
（ex: `rclone cat D1:` 把D1:和子目錄底下的檔案全部印出來）

rcat 做的事和 cat 相反，他是將stdin的東西存到一個檔案裡
(ex: `cat file.txt | rclone rcat D1:file.txt`)

結合tar便可以將小檔案打包起來，藉由管線傳到rclone去處理
再加上這個好心人的[腳本](https://github.com/Riebart/pipechunker)，便可分割上傳
(我不會python不知道他有沒有暫存檔案，但好像沒有？)

### tar使用

tar 常用指令（原諒我不學無術又偷懶:P)
http://www.vixual.net/blog/archives/127

### gpg ASE 加密

用gnupg ase 256加密

`gpg --symmetric --cipher-algo AES256`
輸入來自stdin，輸出至stdout
這個指令會要你輸入兩次密碼

解密時用gpg就好了
輸入一樣來至stdin，輸出一樣至stdout
如果要存入檔案，可以用`gpg | cat > file`

有些裝置會顯示
`gpg: signing failed: Inappropriate ioctl for device`

可用 `export GPG_TTY=$(tty)` 解決

### pv顯示進度

這東西好像會顯示管線的進度（已經傳多少MB，現在每秒多少MB)
使用 | pv | 夾在管線和管線中間即可使用

### pipechunk.py

好心人寫的[腳本](https://github.com/Riebart/pipechunker)，可以把tar的資料串流，分隔成一個一個的塊，再借由rclone上傳到雲端硬碟

作者給的使用範例：
```bash
    tar -cf - MyMachine/ |
    gpg --compress-algo none -er me@example.com | pv |
    python3 ~/Desktop/pipechunk.py
    --name
    "od:/WindowsImageBackup/MyMachine.2018-12-01/MyMachine-2018-12-01
    .tar.enc"
    --command '["rclone", "rcat", "--verbose", "--stats", "10s"]'
    --chunk-size 50000000
    --parallel 8
```
`--chunk-size`是指要每個分割檔要多大，範例是50MB一個分割，上傳後的檔案名會是 file\_name.0001，file\_name.0002 以此類推

需要注意的是`--command` 裡的參數每有一個空格就要加一次""
例如我想要加上 `--bwlimit 1500K`，就要寫成`'["--bwlimit", "1500K"]'`

### 注意事項

* 不會python，看了老半天也不知道他是全部分割好再上傳還是，串流上傳，不過我傾向於後者

* 補充，上傳大檔案後發現應該是串流上傳，並且會使用RAM做暫存如果一個區塊大小是1G `--parallel`為8 的話會吃8G的RAM，可以適時的調整區塊大小和`--parallel`

* `--parallel`參數是開幾個rclone同時上傳，如果`--parallel`為4，`--bwlimit 500k`上傳限制就會是500k\*4 = 2000k

### 下載再還原

如果還是要使用管線來達成可用rclone cat指令，將全部的分割檔全部下載並還原

例如： `rclone cat D1:tar/ | gpg | pv | tar -x`


大概4jone，之後想到什麼再寫上來...
