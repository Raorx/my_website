+++
title = "Git该如何免密码提交？"
date = 2018-06-27T23:41:43+08:00
draft = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ['Network', 'VCS']
categories = []

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
# Use `caption` to display an image caption.
#   Markdown linking is allowed, e.g. `caption = "[Image credit](http://example.org)"`.
# Set `preview` to `false` to disable the thumbnail in listings.
[header]
image = ""
caption = ""
preview = true

+++

早年的时候比较迷，看到GitHub上的ssh key和repo的ssh地址总是傻傻不知道有什么用...然后使用GitLab的private仓库的时候因为强制要求使用ssh key感到很迷茫（因为在GitHub根本用不起）（什么？Student Pack？）（什么？不知道5个根本不够用吗2333333）。今天终于抽点时间解释一下来给后人看。

直到前阵子幡然醒悟，md老子这么久以来为什么一直在用账号密码提交代码？

本文中就不以CMD举例了，反正看这个东西的人，你肯定有一个git bash，这比CMD好用n倍，那么命令都是一样的。

先查看当前用户有已经生成的ssh公钥吗：
```shell
cat ~/.ssh/id_rsa.pub
```

如果没有，从头生成一个：

```shell
ssh-keygen -C "你可以写邮箱，但是无所谓，不加这个参数都无所谓"
```

嗯，就是这么简单，对于普通用户而言，什么`-t`啊`-b`啊，忘了它们吧。
中间还会提示你选个路径/文件名去存放这个key，无所谓，默认最好了，方便。
中间还会提示你输入一个“短语”，我去你*的短语，这就是密码啊...这个可能得找个小本本悄悄记下来了hhhhh，一路回车，然后再来一轮：

```shell
cat ~/.ssh/id_rsa.pub
```

是不是有内容了！是不是敲厉害！默默把标准输出复制下来吧！

然后去GitHub（当然GitLab什么的同理啦）找到设置再找到SSH key那项，选择New（账户设置啊，别去repo设置玩半天），把之前复制的内容贴进去。写个妥善的Title（比如这是你何年何月何日在何地结缘的电脑，网吧什么的你还是别搞什么SSH key这种幺蛾子了），让自己记住是干嘛用的。

好了，这个时候估计你就会和以前的我一样开始玩蛋：
兴高采烈地push了一波发现输入账号和密码的提示又出来了！
因为你其实之前就是通过https方式访问repo的，改成ssh就好了，比如这样：

```shell
git remote rm origin
git remote add origin git@你那个仓库的地址（详见下图）.git
```

当然，如果你之前没有clone过仓库，那就直接从git地址克隆就没毛病了。

之后再push就无痛了。