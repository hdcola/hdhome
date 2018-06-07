# 有关MAC的一些作弊条

## 备份和恢复SD卡

### 备份

* 插入SD卡到你的mac，这时会自动挂载相关的分区
* ```sudo umount /dev/disk2s1```
* ```sudo dd if=/dev/disk2 | bzip2 -9f > ~/hassbian.img.bz2```

注意，要自己用mount看看你的dos分区是不是disk2s1，做dd则就是disk2整个磁盘。

### 恢复

可以使用```etcher```来直接恢复，也可以使用

```
sudo bunzip2 -dc ~/hassbian.img.bz2 | sudo dd of=/dev/disk2
```

## Virtualbox安装

安装Virtualbox死活失败，都快丧尸信心了，最后发现是mac的安全规则，一条命令搞定，去系统设置的安全中设置为任何源都可以吧。

```
sudo spctl --master-disable
```

## 好用的Atom插件

### pretty-json

[https://atom.io/packages/pretty-json] 提供json的格式化能力

### Minimap

[https://atom.io/packages/minimap] 提供代码缩略图，可以放在右侧用于代码概览和导航

### atom-beautify

[https://atom.io/packages/atom-beautify]提供代码的格式化功能，支持格式化HTML, CSS, JavaScript, PHP, Python, Ruby, Java, C, C++, C#, Objective-C, CoffeeScript, TypeScript, Coldfusion, SQL等等

### terminal-plus

[https://atom.io/packages/terminal-plus]支持将终端集成到编辑器中。在编辑器界面就可以打开终端执行命令，省去了另开窗口的麻烦。打开终端时，路径会定位到编辑器项目中的根目录，可以非常方便的对当前项目进行命令操作。

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
