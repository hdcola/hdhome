# 码农作弊条

## 准备开发环境

我是OSX，别的也差（da）不（bu）多（tong）啦。

第一件事肯定是安装Homebrew啦，码农必备、生活必须：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

然后是python 3

```
brew install python3
```

## 设置本地代码库

去[https://github.com/home-assistant/home-assistant] Fork一个home-assistant到自己的git里。然后设置本地的复本：

```
git clone https://github.com/YOUR_GIT_USERNAME/home-assistant.git
cd home-assistant
git remote add upstream https://github.com/home-assistant/home-assistant.git
```

## 设置虚拟环境

我们会使用venv设置一个独立的环境。在home-assistant目录下做这样的操作

```
python3 -m venv .
source bin/activate
```

安装需要的软件包：

```
script/setup
```

启动安装：

```
hass
```
