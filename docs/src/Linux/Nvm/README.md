# 

# Nvm安装

## 1、下载

> [官网下载地址](https://github.com/nvm-sh/nvm/releases)

上传至 `/usr/local`目录，解压
 
```bash
tar -zxvf nvm-0.39.7.tar.gz
```

## 2、安装

编辑配置文件 `vim ~/.bashrc`

在最后写入配置， NVM_DIR 安装目录

```bash
# nvm
export NVM_DIR="/usr/local/nvm-0.39.7"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

刷新配置

```bash
source ~/.bashrc
```

## 3、检查

```bash
nvm -v
```

# Nvm 常用命令

```bash
nvm -v              // 检查nvm版本 
nvm ls              // 查看目前已安装的 node 及当前所使用的 node
nvm ls-remote       // 查看目前线上所能安装的所有 node 版本
nvm install x.x.x   // 安装指定版本
nvm uninstall x.x.x // 移除指定版本
nvm use x.x.x       // 使用指定版本
```



