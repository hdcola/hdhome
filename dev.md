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

## hello world

在.homeassistant目录中建一个目录custom_components，加入一个文件helloworld.py，内容为：

```
"""
helloworld.py
"""

# 定义domain
DOMAIN = "helloworld"

def setup(hass, config):
    # 设置hello state
    hass.states.set( DOMAIN+".hello", "侬好！")
    # setup成功
    return True
```

在configuration.yaml里加一行

```
helloworld:
```

重启后，就会发现面板中多了一个helloworld的卡片，它的一个属性hello，值为侬好！

* HomeAssistant在配置文件中发现helloworld域的配置
* 自动调用helloworld.py文件中的setup函数，setup传入的第一个参数（hass）代表运行的hass对象。
* hass.states.set函数用于设置状态，第一个参数代表实体的ID（格式为域.OBJECT_ID），第二个参数是状态值（状态值可以是任何字符串，也可以是任何可以转换成字符串的类型）。
在HomeAssistant中，并不需要创建实体——在对一个实体设置状态的时候，如果以前不存在这个实体，系统自然就认为出现了一个新的实体。 参考阅读：https://www.hachina.io/docs/468.html
setup函数返回True，代表这个域初始化成功了。
