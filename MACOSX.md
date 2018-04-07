# 有关MAC的一些作弊条

##

安装Virtualbox死活失败，都快丧尸信心了，最后发现是mac的安全规则，一条命令搞定，去系统设置的安全中设置为任何源都可以吧。

```
sudo spctl --master-disable
```


## 同步你的文件

开一个samba或是sftp都让我非常不爽，我最喜欢的还是使用了git同步
