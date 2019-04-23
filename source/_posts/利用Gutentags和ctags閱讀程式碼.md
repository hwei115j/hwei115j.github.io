---
title: 利用Gutentags和ctags閱讀程式碼
date: 2019-04-23 10:37:27
categories:
    - 筆記
tags: 
    - 筆記
    - vim  
---

## 前言

因為閱讀code，如果沒有可以查詢定義的功能的話，會很不方便，加上使用vim，所以之前就參考[這裡](https://zhuanlan.zhihu.com/p/36279445)來設定
但不知道為何，在我的桌機上一直沒辦法正常運作(win10)
因為很麻煩，放棄了一段時間，又因為要用到，所以又重新繼續搞


## 設定

發現之前搞錯了一些東西
首先能動的一直是ctags(gtags&gtags-cscope界面太難用了....)
我以前一直以為我用的是gtags但其實是ctags.....

所以一開始就沒有用到gutentags\_plus這個插件的功能
用的只有`vim-gutentags`
這是一個可以在vim8上，在後台執行tags更新的插件

他的設定
 
```vimscript
let g:gutentags_modules = ['ctags', 'gtags_cscope']
let g:gutentags_project_root = ['.root', '.git']
let g:gutentags_cache_dir = expand('~/.cache/tags')
```

意思是說會在打開的目錄向上搜尋，直到遇到`.root`或`.git`，然後在`~/.cache/tags`建立ctags和gtags的tag檔

因為只要用到ctags所以改成
`let g:gutentags_modules = ['ctags']`
就只會建立ctags檔了

照理來說，此時只要打開專案裡的原始檔，此插件就會自動產生tags擋了，然後自動設定ctags
就能用<C-\]\> <C-t\>在程式碼之間跳轉了
但我發現，在win10的wsl和msys2環境裡，在`~/.cache/tags`裡的tags檔會呈現

```
!_TAG_FILE_FORMAT       2       /extended format; --format=1 will not append ;\
" to
lines/
!_TAG_FILE_SORTED       1       /0=unsorted, 1=sorted, 2=foldcase/
!_TAG_PROGRAM_AUTHOR    Darren Hiebert  /dhiebert@users.sourceforge.net/
!_TAG_PROGRAM_NAME      Exuberant Ctags //
!_TAG_PROGRAM_URL       http://ctags.sourceforge.net    /official site/
!_TAG_PROGRAM_VERSION   5.8     //
```

這樣的狀態
就是沒有產生tags，為何會這樣讓我百思不得其解（或許只有我的電腦如此....）

但他有一個變數
`let g:gutentags_ctags_extra_args = []`
功能是傳入參數，讓ctags處理

一開始我用
`let g:gutentags_ctags_extra_args = ['*']`
這樣確實產生了tags檔了，但他的路徑是相對路徑，在跳轉程式碼的時候，他會在`~/.cache/tags`下尋找原始檔，但這樣自然是找不到
所以要傳入的是絕對路徑

後來在翻閱了相關的資料後（感謝google和google翻譯）
終於找到了解決方法，在.vimrc中加入以下程式碼便能傳入絕對路徑

```vimscript
"獲得打開的檔案路徑，並把所有的空白' '轉成問號'?'
function FindSessionDirectory() abort
  if len(argv()) > 0
    return substitute(fnamemodify(argv()[0], ':p:h'), "\ ", "?", "g")
  endif
  return getcwd()
endfunction
let g:gutentags_ctags_extra_args = [FindSessionDirectory()]
```

之所以要把空白轉成問號的原因是因為如果有空白的參數傳入，tags也會沒辦法產生，就算加上跳出字元也一樣，原因不明

大guy4這樣
不怎麼高明的解決方法，但現在終於能比較簡單的閱讀程式碼了.....

## 參考資料

>https://zhuanlan.zhihu.com/p/36279445
https://bit.ly/2IQYXQT
