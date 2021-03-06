# GPU服务器安装配置与使用维护

## 1 GPU服务器安装配置
1. 安装16.04英文版系统
    * 注意安装在固态硬盘
    * 注意设置swap-area大小为内存的2倍以上
    * 安装不要区分/home和/root，直接选择/的根挂载点安装

2. 卸载不必要的系统软件：
```
# 卸载libreoffices
sudo apt-get remove libreoffice-common
# 删除Amazon图标
sudo rm -f /usr/share/applications/com.canonical.launcher.amazon.desktop
sudo rm -f /usr/share/applications/ubuntu-amazon-default.desktop
# 删除多余的软件
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese gnome-orca webbrowser-app gnome-sudoku landscape-client-ui-install transmission-common
```

3. 必备软件安装安装（openssh vim git curl tmux gcc）：
```
# openssh安装
sudo apt-get install openssh-server
sudo apt-get install openssh-client
# vim git curl
sudo apt-get install vim git curl tmux
# upgrade gcc and g++
sudo add-apt-repository ppa:ubuntu-toolchain-r/test 
sudo apt-get update 
sudo apt-get install gcc-7
sudo apt-get install g++-7
```

4. 第三方ppa源安装Python3.6/3.7：
```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt install python3.6 python3.6-dev -y
sudo apt install python3.7 python3.7-dev -y
```

5. Python-pip以及跟换国内镜像源：
```
sudo apt-get install python-pip
mkdir ~/.pip
sudo apt-get install vim 
vim ~/.pip/pip.conf
# 清华国内镜像源，添加如下内容
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = pypi.tuna.tsinghua.edu.cn
```

6. 查询和挂载硬盘：
```
# 查询硬盘
sudo fdisk -l
# 创建文件夹、挂载
cd /media/dell
mkdir dataset_disk
sudo mount /dev/sdb /media/dell/dataset_disk
# 降低权限
sudo chmod -R -v 777 *
```

7. 安装NVIDIA驱动：
```
# 禁用nouveau
sudo vim /etc/modprobe.d/blacklist.conf
# 文件末尾添加如下两行
blacklist nouveau
options nouveau modeset=0
# 更新系统
sudo update-initramfs -u
# 验证是否已禁用，若无输出，禁用成功。
lsmod | grep nouveau
# 官网下载驱动，进入命令界面ctrl+alt+f1，关闭GUI服务
sudo service lightdm stop
# 卸载安装残留
sudo apt-get remove nvidia-*
# 增加安装包权限
chmod a+x NVIDIA-Linux-x86_64-430.50.run
# 运行安装包
sudo ./NVIDIA-Linux-x86_64-430.50.run -no-x-check -no-nouveau-check -no-opengl-files 
# 安装注意如下选型选yes，其他按照默认选择
Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up. 选择 Yes 继续
# 检查是否安装成功
nvidia-smi
# 启动GUI服务
sudo service lightdm start
# 回到图形界面
ctrl+alt+f7 
```

8. 安装CUDA，设置环境变量：
```
# 运行安装包，安装顺序安装
sudo sh cuda_10.1.243_418.87.00_linux.run
# 设置环境变量
vim ~/.bashrc
# 末尾添加如下内容
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
# source 生效
source ~/.bashrc
# 验证安装
nvcc -V
```

9. 安装CUDNN：
```
# 官网注册下载CUDA版本对应CUDNN安装包
tar -xzvf cudnn-10.0-linux-x64-v7.tgz
# 安装
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
# 安装验证
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

10. 安装zsh和oh-my-zsh：
```
# 安装zsh
sudo apt-get install zsh
# 安装ohmyzsh,本仓库备份了该脚本文件
sh ohmyzsh_install.sh
# 安装插件
sudo apt-get install autojump
cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-autosuggestions.git
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
```
用如下内容替换原.zshrc文件内容:
```
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="ys"
plugins=(
          git
          autojump
          zsh-autosuggestions
          zsh-syntax-highlighting
         )
# autojump
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
source $ZSH/oh-my-zsh.sh
# CUDA
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
# CXX
export CXX=g++-7
```
改动生效
```
source ~/.zshrc
```

10. 多用户创建：
```
# 添加用户
sudo adduser 用户名
# 显示用户列表
grep bash /etc/passwd
```

11. [duf](https://github.com/muesli/duf)磁盘容量查询：
```
sudo mv duf /usr/bin
duf
```

## 2 使用教程
1. 基本shell操作
2. 命令行连接校园网
```
# 登录
curl 'http://202.204.48.66' --data "DDDDD=学号&upass=密码&0MKKey="
# 注销
curl http://202.204.48.66/F.htm
```

3. 结合ssh访问，vim编辑，git版本控制，tmux窗口管理（必须使用）

4. 创建virtualenv虚拟python环境
```
# 创建环境
virtualenv -p /usr/bin/python3.7 virtual_3.7
# 激活环境
source ~/Github/virtual_3.7/bin/activate
# 退出环境
deactivate
```

## 3 问题处理
卡显存，使用ctrl+c

查询所有网络内设备

定期更新

## 4 自动连接网络




