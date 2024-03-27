---
title: macOS 安装Anaconda和Miniconda
url: https://zhuanlan.zhihu.com/p/136380298
clipped_at: 2024-03-28 00:19:41
category: default
tags: 
 - zhuanlan.zhihu.com
---


# macOS 安装Anaconda和Miniconda

## Homebrew 安装

```text
brew cask install miniconda
```

## PKG下载包安装

### 清华大学开元软件镜像站：

Anaconda：

[https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M&O=D](https://link.zhihu.com/?target=https%3A//mirrors.tuna.tsinghua.edu.cn/anaconda/archive/%3FC%3DM%26O%3DD)

Miniconda：

[https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/?C=M&O=D](https://link.zhihu.com/?target=https%3A//mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/%3FC%3DM%26O%3DD)

```text
pkg 包清理工具uninstallpkg
https://www.corecode.io/uninstallpkg/
```

### 官方下载地址

Anaconda：

[https://www.anaconda.com/distribution/#download-section](https://link.zhihu.com/?target=https%3A//www.anaconda.com/distribution/%23download-section)

## Anaconda 下载加速设置国内镜像

使用命令：

```text
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --set show_channel_urls yes
```

或者编辑 `~/.condarc`

```text
channels:
  - defaults
show_channel_urls: true
channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

### 提示 conda 命令未找到

```text
cd /opt/miniconda3/condabin
# 使用 zsh 初始化
./conda init zsh 

# 如果使用 bash
# ./conda init bash 
```

### 测试安装,创建虚拟环境

```text
conda create -n tf2 python=3.7
```

## 常用conda命令

```text
# 创建虚拟环境
conda create -n xxx

# 启用虚拟环境

conda activate xxx

# 关闭虚拟环境

conda deativate xxx
```