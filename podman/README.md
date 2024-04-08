# Podman 使用说明
<!-- TOC depthFrom:2 -->

- [1. 前期工作](#1-前期工作)
    - [1.1. Podman环境](#11-podman环境)
    - [从Docker迁移](#从docker迁移)
- [2. 运行容器](#2-运行容器)
    - [2.1. 极简版](#21-极简版)
    - [2.2. 综合版](#22-综合版)
- [登录](#登录)
- [3. 自己构建镜像（非必须）](#3-自己构建镜像非必须)
- [4. 其他](#4-其他)

<!-- /TOC -->

## 1. 前期工作

### 1.1. Podman环境

请确认已安装了Podman所需环境（[Podman](https://podman.io/)

安装完成后，请执行`podman -v`检查是否安装成功，请执行`podman info`检查虚拟机环境是否正常。

常用命令参考：

```
# 查看版本
podman -v

# 初始化虚拟机
podman machine init

# 启动虚拟机
podman machine start

# 查看信息
podman info
```

### 从Docker迁移

Podman可以和Docker共存，命令也基本可以通用。

但挂载逻辑有点区别，podman挂载时，如果宿主机下没有指定的文件夹，podman不会像docker一样去自动创建文件夹，而是会报异常。

所以在挂载文件夹时，需要先手动在宿主机上mkdir创建文件夹。

## 2. 运行容器

以下提供极简版和综合版两个版本，一个简单一个复杂，供参考

### 2.1. 极简版

```
# 生成并运行容器
podman run -itd --name="bili" docker.io/zai7lou/bilibili_tool_pro

# 查看实时日志
podman logs -f bili
```

其中，`cookie`需要替换为自己真实的cookie字符串

### 2.2. 综合版

```
# 创建文件和文件夹
mkdir -p /bili && cd /bili
mkdir -p Logs

# 下载appsettings.json
wget https://raw.githubusercontent.com/RayWangQvQ/BiliBiliToolPro/main/src/Ray.BiliBiliTool.Console/appsettings.json
wget https://raw.githubusercontent.com/RayWangQvQ/BiliBiliToolPro/main/docker/sample/cookies.json

# 运行
podman run -itd --name="bili" \
    -v /bili/Logs:/app/Logs \
    -v /bili/appsettings.json:/app/appsettings.json \
    -v /bili/cookies.json:/app/cookies.json \
    -e Ray_DailyTaskConfig__Cron="0 15 * * *" \
    -e Ray_LiveLotteryTaskConfig__Cron="0 22 * * *" \
    -e Ray_UnfollowBatchedTaskConfig__Cron="0 6 1 * *" \
    -e Ray_VipBigPointConfig__Cron="7 1 * * *" \
    -e Ray_DailyTaskConfig__NumberOfCoins="5"
    docker.io/zai7lou/bilibili_tool_pro

# 查看实时日志
podman logs -f bili
```

其他指令参考：

```
# 查看容器运行状态
podman ps -a

# 进入容器
podman exec -it bili bash
```

## 登录

在宿主机运行`podman exec -it bili bash -c "dotnet Ray.BiliBiliTool.Console.dll --runTasks=Login"`

扫码进行登录。

![login](../docs/imgs/docker-login.png)

## 3. 自己构建镜像（非必须）

目前我提供和维护的镜像：`[zai7lou/bilibili_tool_pro](https://hub.docker.com/repository/docker/zai7lou/bilibili_tool_pro)`;

如果有需要（大部分都不需要），可以使用源码自己构建镜像，如下：

在有项目的Dockerfile的目录运行

`podman build -t TARGET_NAME .`

 `TARGET_NAME`为镜像名称和版本，可以自己起个名字

## 4. 其他

镜像使用的是docker仓库的镜像。
