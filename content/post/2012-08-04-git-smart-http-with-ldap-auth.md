+++
title = "Git: Smart Http with LDAP Auth"
date = "2012-08-04T02:28:00+08:00"
type = "post"
tags = ["Git"]
+++

過去工作上用svn，私底下用git，為了跟上潮流，打算在原本有的svn機器上架起git的服務，
svn commit權限是透過LDAP來管理的，所以把這樣的功能apply到git上也是逃不掉的orz...

## How To
以下是一個基本的virtual host設定，

{{< gist Lance0312 3250315 >}}

使用的情境是這樣的，當使用者clone了某個repository時，apache會要求輸入帳號密碼，
並將帳號密碼拿去ldaps://your.ldap.server這台ldap server上作認證
```bash
$ git clone http://your.domain.name/git/testing.git
Username for 'http://your.domain.name': username
Password for 'http://username@your.domain.name':
...
Unpacking objects: 100% (??/??), done.
```
同樣的，當`git push`時，apache也會向使用者要求帳號密碼

有幾點需要注意

* git bare repositories放在`/opt/git`下，ex. `/opt/git/testing.git`
* http://your.domain.name/git/testing.git 對應到 `/opt/git/testing.git`
* 由於ldap server用的是self-signed certificate，所以用`LDAPVerifyServerCert off`
  暫時略過檢查

基本的設定都可以在[git-http-backend man page][1]中找到

## Situation ##

man page中有一段snippet長這個樣子，
```apache
<LocationMatch "^/git/.*/git-receive-pack$">
    ...
</LocationMatch>
```
上面說，你可以藉此達到`git clone`時不認證，`git push`時才認證的的效果，這正是我想要的東西，但我試了千萬遍依舊不成功...orz
```bash
$ git push
error: Cannot access URL http://your.domain.name/git/test.git/, return code 22
fatal: git-http-push failed
```

要知道為什麼失敗，先來看一下正常的`git push`在apache上留下了什麼log
```
- [04/Aug/2012:02:19:34 +0800] "GET /git/test.git/info/refs?service=git-receive-pack HTTP/1.1"
- [04/Aug/2012:02:19:38 +0800] "GET /git/test.git/info/refs?service=git-receive-pack HTTP/1.1"
username [04/Aug/2012:02:19:38 +0800] "GET /git/test.git/info/refs?service=git-receive-pack HTTP/1.1"
username [04/Aug/2012:02:19:38 +0800] "POST /git/test.git/git-receive-pack HTTP/1.1"
```
在真正送出POST前，會先送幾次GET，而且在最後一次GET時，已經**通過**LDAP認證，  
再回來看看我們的設定
    <LocationMatch "^/git/.*/git-receive-pack$">
翻譯成中文就是，當我們的`Location`符合`^/git/.*/git-receive-pack$`這段regex時，
就**開始**作LDAP認證，這聽起來不像是有問題的樣子，但悲慘的是，LocationMatch不吃
query string，所以跟本認不得request裡面有git-receive-pack的字眼，Shawn O. Pearce
在[mailing list][2]上有對這個問題說明，並提供了一段patch

結論很簡單，大家還是多打幾次密碼吧XD，最好的辦法還是捨棄http，改用ssh key :p

[1]: http://www.kernel.org/pub/software/scm/git/docs/git-http-backend.html "git-http-backend man page"
[2]: http://lists-archives.com/git/714311-git-http-backend-and-authenticated-pushes.html "Re: git-http-backend and Authenticated Pushes"

