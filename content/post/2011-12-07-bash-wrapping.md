+++
title = "bash wrapping 位置錯誤"
date = "2011-12-07T10:41:00+08:00"
type = "post"
tags = ["Bash"]
+++


在bash環境下，紀錄terminal寬度的變數叫做COLUMNS，可以用
```bash
$ echo $COLUMNS
```
來查看目前terminal的寬度(字元數)
  
bash藉由這個變數來決定哪個位置該wrap，改變terminal大小時，通常這個值也會跟著改
變，但有時候就會有靈異現象，造成terminal大小改變，但COLUMNS的值卻沒有改變
  
bash的一個built-in command shopt，或許可以解決這個問題，shopt中有一個選項
checkwinsize，會在每次做完一個指令後檢查terminal的大小，並視情況更新LINES &
COLUMNS，可以把他加在 ~/.bashrc 裡面
```bash
$ shopt -s checkwinsize
```

至於效果如何，需要再觀察看看囉
  
Reference:  
[Bash Reference Manual](http://www.gnu.org/software/bash/manual/bashref.html#The-Shopt-Builtin)

