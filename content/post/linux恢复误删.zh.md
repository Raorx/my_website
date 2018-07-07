+++
title = "如何不方不盲地恢复Linux下误删的文件"
date = 2018-07-02T15:04:36+08:00
draft = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ['Linux']
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

{{% toc %}}

## 杯具的缘由

太久没回来广州，没用实验室的服务器写东西、算东西。刚刚清用户目录的时候，误删了一个目录，全是过年期间跑数据的中间结果，WTF，有点high。
众所周知，如果没做alias的话，Linux是没有“回收站”这个说法的。也就是说，比如我这样`rm -rf workflow`把这个叫做workflow的目录删了，那么删了就是删了，那就是彻底删除了。
虽然说按照专业素养，我是肯定有命令啊、参数啊和环境等等的记录的。然鹅算力和时间才是最贵的，平白无故让我再来一轮，我肯定不开心啊！
然后我就想到，老衲其实是可以把它恢复的！

## 我为什么能恢复“彻底删除”的文件

这样，就要提到Linux的文件删除原理了。在Windows下面，删个大文件那是要很久的。但是在Linux下面，由于所谓的删除只是将存储文件的inode和blocks标记为未使用，所以删除是一件甚至根本察觉不到时间流逝的行为！
也就是说，数据其实并未被彻底从硬盘抹去（什么？你说你删除是用`dd`命令去删的？呵呵呵呵...），只是这块硬盘空间已经被标记为空闲了，其他进程分分钟可以覆盖上面的数据，也就是删除了。

## 抢救过程

{{% alert note %}}
其实这里有必要提一句，Linux是不会删除进程尚在使用的文件的。所以如果有进程还在使用误删的文件，用`lsof`配合`grep`可以很容易将其恢复，具体细节不属于本文记录范围，不详细说了。
{{% /alert %}}

{{% alert warning %}}
为了防止这块盘再有什么厉害的读/写，覆盖了这些“已删除”的文件占用的磁盘空间，是要马上解挂载的。
{{% /alert %}}

挂载之前，我做了一件好事，用`ls -di /home/ysun`查看了我目录的inode（说出来你们可能不信，我是编剧，这是一个伟大的伏笔）。服务器上`/home`是挂载到另外一块硬盘（整一个分区）`/dev/sdb1`上的。想要`umount /home`，发现不行，设备忙。这个时候可以用`fuser -km /home`强制kill掉占用了`/home`的进程。这个时候就可以解挂载了，然后我用putty开的终端就消失了（因为ssh连接也被kill了）。

为了方便接下来的操作，是想用root登录的。然鹅，发现root居然不可以用ssh登录，顿觉有点不适。又重新用普通用户修改了`/etc/ssh/sshd_config`的内容，将`PermitRootLogin without-password`中的`without-password`改为`yes`，再用`sudo systemctl restart sshd.service`重启一下ssh服务，可以了。

上网搜了一圈，一些比较多人用的工具，比如ext3grep，已经不适用与ext4文件系统了。先尝试了extundelete，但是两年多没维护的工具了，各种和现有的软件包冲突，各种Segment Fault，各种Core Dumped（卧槽居然押韵）。冲突就算了，我想把冲突的包降级都不成，那么旧的版本人家都不保留了。去GitHub找了其他人fork过去修改的版本，发现还是跟不上底层库更新的脚步，不好使。

然后就转向了ext4magic。先安装：`aurman -S ext4magic`。

安装好了之后，先`ext4magic /dev/sdb1 -I 50331649`（这串数字就是上文的伏笔了嗯），看看我中午睿智地做了什么:

![ext4magic的输出](/img/post_img/ext4magic.png)

卧槽，辣眼睛啊！就这么给删了...删了...了...

这个时候，我以迅雷不及掩耳到铃儿响叮当之势掏出了一个小本本，记下了这个被误删的目录的inode值，57606262。
然后`ext4magic /dev/sdb1 -r -I 57606262  -d /root`尝试恢复：

![文件恢复](/img/post_img/文件恢复结果.png)

舒服了啊！
赶紧挂载上硬盘`mv`走...

{{% alert note %}}
理论上讲，是可以用参数指定时间戳（关于什么是时间戳，那要追溯到1970年了，请百度）来指定时间范围查看文件增删记录的。但是不知道为什么在extundelete里好用，在ext4magic里就瓦特了。比如想获取今天（如果你是好久之后才看到你可能不知道哪天是今天hhhhh）下午2点50的时间戳，那么就`date -d "2018-07-02 14:50:00" +"%s"`，然后终端内得到的一串数字就是了。
{{% /alert %}}

## 总结

1. Linux没有回收站，可以的话，还是做个alias。我以前也认为“TMD这么蠢的事情我会做吗，不存在的”，然后今天不记得了目录的内容，脑子一热就GG了；
2. 发生意外时，不要慌，虽然自己手贱踩了雷，对操作系统的理解又深刻了一层，长远看来，还是值得的；
3. 还是Linux好啊，辣鸡Windows，动辄要收费**；
4. Linux果然是好；
5. Linux真是好啊！