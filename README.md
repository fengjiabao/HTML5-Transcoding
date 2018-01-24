# HTML5-Transcoding
# 使用 HTML5 video 播放 RTSP 流 

## 原理

1. 利用 FFmpeg 将 RTSP 流的转码为 Fragmented MP4。

```
ffmpeg -re -rtsp_transport tcp -i <RTSP-URL> -vcodec copy -f mp4 -movflags frag_keyframe -reset_timestamps 1 -vsync 1 -flags global_header -bsf:v dump_extra -y -
```

> Ffmpeg 命令选项参考 https://xdsnet.gitbooks.io/other-doc-cn-ffmpeg/content/ffmpeg-doc-cn-05.html

2. 将转码后的 fMP4 流通过 http 发送到客户端的浏览器中。

## 安装

### 服务器 (Linux)

1. 安装 FFMpeg 环境

```shell
// 添加 FFMpeg 源
su -c 'yum localinstall --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-7.noarch.rpm'

rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm

// 安装 ffmpeg
yum -y install ffmpeg ffmpeg-devel
```

2. 运行 RTSP Proxy

```shell
$ cd <RTSPProxy directory>
$ ./proxy
```

### 客户端

在浏览器中打开 `http://<server>:9000/app/viewer.html`，即可直接观看监控视频。



## 编译

### 优化编译

增加  `-ldflags "-s -w"` 参数。

```shell
$ go build -ldflags "-s -w" -o proxy proxy.go
```



### 交叉编译

#### macOS 环境

 linux 版本

```shell
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" -o lproxy proxy.go
```

windows 版本

```shell
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -ldflags "-s -w" -o wproxy.exe proxy.go
```



#### Windows 环境

 Mac 版本

```Shell
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build -o iproxy proxy.go
```

linux 版本

```shell
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build -o lproxy proxy.go
```



> 说明
>
> - GOOS：目标平台的操作系统（darwin、freebsd、linux、windows） 
> - GOARCH：目标平台的体系架构（386、amd64、arm） 
> - 交叉编译时，不支持 CGO， 所以要禁用它
