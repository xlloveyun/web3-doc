# Ubuntu 环境搭建指南

## 1. 安装必要组件

1. 更新软件包列表
`sudo apt-get update`

2. 安装基础工具
`sudo apt-get install inotify-tools && sudo apt install make && sudo apt install g++`

3. 安装开发库
`sudo apt-get install libudev-dev zip unzip build-essential cmake -y`

4. 安装Git及其他依赖
`sudo apt-get install git automake libtool inotify-tools libgmp-dev libgmp10 build-essential cmake -y`

## 2. 安装ASDF版本管理器

1. 克隆ASDF仓库
`git clone https://github.com/asdf-vm/asdf.git ~/.asdf`

2. 编辑用户配置文件
`nano .profile`

3. 在文件末尾添加
`. $HOME/.asdf/asdf.sh`
按Ctrl+X保存退出

4. 刷新配置文件
`source ~/.profile`

5. 验证安装
`asdf version`
应显示类似：v0.13.1-fad23bc

6. 添加语言插件
```
asdf plugin add erlang
asdf plugin add elixir
asdf plugin add nodejs
```

7. 安装额外依赖
```
sudo apt-get -y install build-essential √
sudo apt-get -y install autoconf √
sudo apt-get -y install m4 √
sudo apt-get -y install libncurses5-dev √
sudo apt-get -y install libwxgtk3.0-gtk3-dev × Ubuntu 24.04+ 默认提供 libwxgtk3.2-dev（基于 wxWidgets 3.2），官方文档在你的系统里不一定生效 cat /etc/lsb-release 看下你的系统版本先
sudo apt-get -y install libwxgtk-webview3.0-gtk3-dev x libwxgtk-webview3.2-dev（适用于 Ubuntu 24.04+），同上
sudo apt-get -y install libgl1-mesa-dev √
sudo apt-get -y install libglu1-mesa-dev √
sudo apt-get -y install libpng-dev √
sudo apt-get -y install libssh-dev √
sudo apt-get -y install unixodbc-dev √
sudo apt-get -y install xsltproc √
sudo apt-get -y install fop √
sudo apt-get -y install libxml2-utils √
sudo apt-get -y install libncurses-dev √
sudo apt-get -y install openjdk-11-jdk √
```

## 3. 安装PostgreSQL-14

1. 添加PGP密钥
`curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg`

2. 添加仓库源
`sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'`

3. 更新并安装
```
sudo apt update
sudo apt install postgresql-14
sudo systemctl status postgresql
```

## 4. 创建数据库用户

1. 创建系统用户
`adduser dbusername`
（请将dbusername替换为您的用户名）

2. 切换到postgres用户
`sudo -i -u postgres`

3. 创建数据库用户
`createuser --interactive dbusername`

4. 创建数据库
`createdb blockscout`

5. 进入PSQL
`psql`

6. 设置密码
`ALTER USER dbusername WITH PASSWORD 'dbuserpassword';`

7. 授予权限
`GRANT ALL PRIVILEGES ON DATABASE blockscout TO dbusername;`

8. 退出PSQL
`\q`

9. 重启PostgreSQL
`sudo systemctl restart postgresql`

## 5. 安装Rust工具链

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## 6. 部署Blockscout

1. 克隆仓库
`git clone https://github.com/blockscout/blockscout blockscout-backend`

2. 进入目录
`cd blockscout-backend`

3. 安装语言环境
`asdf install`

<提示>
🎉 您已准备好进行手动部署！请继续执行[后端准备步骤](/setup/deployment/manual-deployment-guide#1.-prepare-the-backend)
</提示>