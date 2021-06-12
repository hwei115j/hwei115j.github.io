---
title: chrome備份外掛設定
date: 2021-06-13 03:30:23
categories:
    - 筆記
tags:
    - 筆記
    - chrome
---

## 前言

雖然說Chrome有非常好用的同步功能，只要登入Google帳號，就能同步所有設定和已安裝的外掛
但美中不足的是，外掛中的設定並不會被同步，每次遷移瀏覽器時都要手動去備份和還原每一個外掛的設定，如果是一兩個倒還好，但如果安裝了十幾個外掛，遷移的過程就變得非常煩人
而以前我遷移瀏覽器時，都是直接備份User Data資料夾，可以一次將所有資料轉移到新的環境，完成無痛遷移
但某次Chrome更新後就不能使用User Data大法
所以我一直想換掉Chrome，改用開源的Chromium，但想到每個外掛都要手動備份設定檔就懶得動orz
最近想說在這樣拖下去不行了，就開始找方法來搞

## 設定
首先先安裝Chromium，然後開始同步，把基本的東西先同步過來
同步完成後打開設定檔

Chrome是在
```cmd
C:\Users\%Username%\AppData\Local\Google\Chrome\User Data\Default
```

Chromium是在
```cmd
C:\Users\%Username%\AppData\Local\Chromium\User Data\Default
```

然後複製這三個資料夾過去

```
Local App Settings
Local Extension Settings
Local Storage
```

第一個是app的設定（像是pttchrome）
第二個是外掛的設定
第三個是有些網頁會存一些資料在你電腦上，如果沒有就要重新設定（可有可無）

接下來就可以用惹
然後雖然

```
Extension Rules
Extension State
Extensions
```

這三個是放外掛的的資料夾
但實測如果是手動安裝的外掛，或已經在Google下架的外掛，都沒辦法這樣載入
如果要載入還是要乖乖備份原來的外掛（不過備份好後設定一樣在那三個資料夾裡）
大概就這樣，做個紀錄，防止我忘記:P
