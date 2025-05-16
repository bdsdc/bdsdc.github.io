




## 手动升级内核
```
https://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/
```
手动安装rpm内核
- kernel-lt-5.4.278-1.el7.elrepo.x86_64.rpm
- kernel-lt-tools-5.4.278-1.el7.elrepo.x86_64.rpm
- kernel-lt-tools-libs-5.4.278-1.el7.elrepo.x86_64.rpm

## 安装5.0+内核
```
yum localinstall -y kernel-lt-5.4.278-1.el7.elrepo.x86_64.rpm kernel-lt-tools-5.4.278-1.el7.elrepo.x86_64.rpm  kernel-lt-tools-libs-5.4.278-1.el7.elrepo.x86_64.rpm
```

查看安装后内核信息
```
# 查看已经安装的内核，可以看到有新安装的5.4.278版本
grubby --info=ALL | grep ^kernel

kernel=/boot/vmlinuz-5.4.278-1.el7.elrepo.x86_64
kernel=/boot/vmlinuz-3.10.0-1160.119.1.el7.x86_64
kernel=/boot/vmlinuz-3.10.0-957.el7.x86_64
kernel=/boot/vmlinuz-0-rescue-439410e29e454c50af1945d12f037511

# 查看当前默认内核
grubby --default-kernel

```
## 设置默认内核

```
grubby --set-default "/boot/vmlinuz-5.4.278-1.el7.elrepo.x86_64"
```

## 重启
```
# 重启
reboot
# 查看当前版本
uname -rs 
```
## 内核版本查看
```
rpm -qa | grep kernel
```
