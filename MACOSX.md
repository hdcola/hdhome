# 有关MAC的一些作弊条

##

安装Virtualbox死活失败，都快丧尸信心了，最后发现是mac的安全规则，一条命令搞定，去系统设置的安全中设置为任何源都可以吧。

```
sudo spctl --master-disable
```


## 同步你的文件

开一个samba或是sftp都让我非常不爽，我最喜欢的还是使用了git同步，这里记录下如何使用git来将你的更改自动同步到你的ha中去。

方案是使用git的hook，我们在hook中加入post-commit，在这个post-commit中加入执行sync.sh的代码，然后自己编写shell scp你想做的同步。做好后，我在atom中修改配置，存盘后commit变更，就自动scp到ha目录中去了。这是一个多么简单的操作啊。

首先，在.git/hook中建立一个post-commit的文件，内容：

```
#!/bin/sh

BasePath="/Users/hd/work/hassbian/sync.sh"

exec $BasePath
```

然后在hassbian中建议sync.sh，内容：

```
scp -r /Users/hd/work/git/hassbian/homeassistant/* pi@hassbian.local:/home/homeassistant/.homeassistant
```

记得chmod +x post-commit和sync.sh
