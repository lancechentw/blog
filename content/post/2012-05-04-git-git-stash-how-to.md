+++
title = "Git stash how to"
date = "2012-05-04T17:33:00+08:00"
type = "post"
tags = ["Git"]
+++

故事是這樣的，我在github上開了一個repo打算拿來放系統上的設定檔，我把一些設定檔
整理好並加了一些README，以防以後痴呆了，忘記設定檔該放在哪裡，就在我準備要
commit時，我想到我還有台laptop...
  
laptop和pc的環境很不一樣，設定檔相對的也會有不少差別，所以我應該要開兩個branch，
一個放laptop的設定，一個放pc的設定，生性懶惰使然，我不想重作一次工作，也不想把
完成的部份commit到master branch上，於是`git stash`就派上用場了，看一下man page
怎麼說：
```bash
$ man git stash
...
DESCRIPTION
Use git stash when you want to record the current state of the working directory and the index,
but want to go back to a clean working directory...
...
```
以下是我的流程：
```bash
# do something...
$ git add . # 將目前的變更加入stage
$ git stash # stage的狀態會被存起來，並將working directory reset
$ git checkout -b newbranch # 產生一個新的branch並checkout
$ git stash pop # 將存起來的stage狀態pop出來
```
man page裡面提供了另外兩種情境可以參考

### Pulling into a dirty tree
upstream有了更新，但跟你的local changes有conflict，導致無法直接用`git pull`解決
```bash
$ git stash
$ git pull
$ git stash pop
```
  
### Interrupted workflow
你老闆要你馬上修正某個問題，但你正在做些偉大的事，一個方法是，先把目前的工作
commit到一個臨時的branch上，等到問題解決了，在soft reset回來，用`git stash`可以
大大簡化這步驟
```bash
# changing the world...
$ git stash
# fix something...
$ git commit -a -m "Stupid bug fixed."
$ git stash pop
# continue on changing the world...
```
`git stash pop`有可能會遇到conflict的狀況，哪天遇到了再回來補充...

