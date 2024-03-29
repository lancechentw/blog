+++
title = "Attach mails as attachments in Mutt"
date = "2013-08-24T16:04:00+08:00"
type = "post"
tags = ["Mutt"]
+++


We all know how to attach a file as an attachment in Mutt. Just press `a` in
[attach mode][0], and either type the file name or press `?` to open up a file
list.

But what to do when it comes to attaching a mail or even mails? For a single
mail, just forward it. For multiple mails, here's what I found. Mutt has a
command `attach-message`, which is default bind to `A`. The help message says,

```
A      attach-message                  attach message(s) to this message
```

It is a little bit confusing after pressing `A` and choose a mail box to open
up. You just need to press `t` to tag any messages you want to attach and press
`q` to return to the attach mode. Then you will see those messages appear in the
attachment list.

Moreover, you can press `^D` on a message to toggle it between inline and attachment.

```
^D     toggle-disposition              toggle disposition between inline/attachment
```


[0]: http://dev.mutt.org/trac/wiki/MuttGuide/Actions#primarymodes "MuttGuide/Actions"


