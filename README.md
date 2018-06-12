# 如果你又双叒叕要重装linux了（写给弱智的自己）

------
##双硬盘windows+ubuntu18.04 MBR+BIOS grub引导
> * 准备系统盘 (ultraiso)
> * 关闭windows快速启动
> * 主系统盘分割空间留给 **/boot** 至少1GB空间
> * 关闭fast boot和secure boot (华硕主板可以清空secure key来关闭secure boot)
> * 选择非UEFI模式启动系统盘，键入```tab```显示可用命令,再次键入```live```打开试用版ubuntu安装
> * 分区时系统盘放 **/boot**, 副盘放 **/swap** 2倍物理内存, **/** 50GB, **/home** 剩下所有空间。同时挂载boot文件到主硬盘 **sda**

##修复grub引导
方法1
```bash
sudo update-grub
```
方法2
使用repair-boot
```bash
sudo add-apt-repository ppa:yannubuntu/boot-repair && sudo apt-get update
sudo apt-get install -y boot-repair && boot-repair
```
-----
#Nvidia显卡驱动 + Anaconda 3.5 + Tensorflow 1.8 + CUDA 9.0 + cuDNN 7.1环境配置

-----
##Nvidia显卡驱动
#### 1. 查看显卡驱动信息
```lspci | grep VGA ```
#### 2. 下载驱动程序
#### 3. 删除原有驱动
```bash
sudo apt-get remove --purge nvidia*
```
#### 4. 编辑 ```/etc/modprobe.d/blacklist-nouveau.conf``` 文件，添加以下内容：
```bash
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```
然后保存并关闭nouveau
```bash
echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf  
```
#### 5. 重启
```bash
update-initramfs -u
sudo reboot
```
#### 6. 安装Nvidia驱动
```bash
sudo sh ./NVIDIA-Linux-x86_64-xxx.xx.run
```
#### 7. 挂载Nvidia驱动
```bash
modprobe nvidia
```
#### 8. 检查驱动是否安装成功
```bash
nvidia-smi
```

## Anaconda3.5
上TUNA下载包直接bash运行安装

## CUDA9.0
由于CUDA 9.0仅支持GCC 6.0及以下版本，而Ubuntu 18.04预装GCC版本为7.3，故手动安装gcc-6与g++-6：
```bash
sudo apt-get install gcc-6 g++-6
```
之后切换至/usr/bin目录修改符号链接，使GCC 6成为默认使用版本：
```bash
cd /usr/bin
sudo rm gcc
sudo ln -s gcc-6 gcc
sudo rm g++
sudo ln -s g++-6 g++
```
下载CUDA9.0以及所有的补丁并安装
打开```~/.bashrc```
```bash
sudo gedit ~/.bashrc
```
添加path
```bash
export PATH=/usr/local/cuda-9.0/bin${PATH:+:$PATH} 
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64/
```
到sample目录```make```测试是否完成安装

## cuDNN7.1
下载library for linux并解压文件，然后进入其include目录
```bash
cd cuda/include
sudo cp cudnn.h /usr/local/cuda/include  #复制头文件
```
然后运行下面的命令
```bash
cd cuda/lib64
sudo cp lib* /usr/local/cuda/lib64/    #复制动态链接库
cd /usr/local/cuda/lib64/
sudo rm -rf libcudnn.so libcudnn.so.7     
sudo ln -s libcudnn.so.7.1.4 libcudnn.so.7  
sudo ln -s libcudnn.so.7 libcudnn.so    
```
然后使用命令检测
```bash
nvcc -V
```

## tensorflow 1.8
```bash
sudo apt-get install python3-pip python3-dev
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow-gpu
```
检测
```python
# Python
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```


------
# Reference
https://www.tensorflow.org/install/install_linux#ValidateYourInstallation
https://blog.csdn.net/qq_31261509/article/details/78755968
http://www.mamicode.com/info-detail-2287182.html
https://blog.csdn.net/stories_untold/article/details/78521925

