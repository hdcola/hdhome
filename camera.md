# 有关摄像头的一些作弊条

## 莹石的接入

树莓派需要加入视频相关的库

```
sudo apt-get install libav-tools
```

安装如果出现问题，哪么

```
sudo apt-get update
sudo apt-get --fix-missing install libav-tools
```

另外说明一下莹石的rtsp地址：

```
rtsp://username:password@ip:554/h264/ch1/main/av_stream
```

这里的username：admin，password：萤石摄像头背面的验证码，注意大小写，必须一致，ip是萤石摄像头的ip地址。安装好ffmpeg后，可以测试下是否ok：

```
ffmpeg -rtsp_transport tcp -i rtsp://username:password@ip:554/h264/ch1/main/av_stream -an -f null
```

### homebridge 接入

安装homebridge-camera-ffmpeg:

```
sudo npm install -g homebridge-camera-ffmpeg
```

在config.json中加入

```
{
  "platform": "Camera-ffmpeg",
  "cameras": [
    {
      "name": "Camera",
      "videoConfig": {
              "source": "-re -i rtsp://username:password@ip:554/h264/ch1/main/av_stream",
              "maxStreams": 2,
              "maxWidth": 1280,
              "maxHeight": 720,
              "maxFPS": 30
      }
    }
  ]
}
```

### homeassistant接入

注意，ha还不支持把camera输出到homekit中，所以做这个意义不大。在configuration.yaml中加入

```
ffmpeg:
  ffmpeg_bin: /usr/bin/ffmpeg

camera:
  - platform: ffmpeg
    input: -rtsp_transport tcp -i rtsp://admin:passwd@1.2.3.4/Streaming/Channels/101
    extra_arguments: -pred 1 -q:v 2 -s 704x576

```

完成！
