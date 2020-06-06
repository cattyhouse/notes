## 清理系统的垃圾

```bash
yes | pacman -Scc
```
## 了解系统的结构

```bash
lsblk -f 
sda
├─sda1 vfat    FAT32             B056-26FF                             196.7M     0% /efi
└─sda2 ext4    1.0               243f7910-8857-4e16-9187-62aa2f2a7e70   15.6G    14% /
# 后面需要用到
```
## 创建排除文件

```bash
echo '
/tmp/*
/mnt/*
/proc/*
/dev/*
/sys/*
/run/*
' > /mnt/exclude.txt
```
## 备份所有文件

```bash
tar \
--exclude-from=/mnt/exclude.txt \
-czpv \
--xattrs \
-f /mnt/arch.tar.gz /
```
## 拷贝出来

```bash
# 从另外一台机器
scp user@arch:/mnt/arch.tar.xz ~/
```

## 还原

```bash
# 用 arch iso 启动新机器
passwd root
systemctl start ssh
# 从另外一台机器上
ssh root@ip # 进入 arch iso
# arch ios 里面
fdisk /dev/sda
# g 创建 gpt 格式
# sda1 = 200M, EFI
# sda2 = rest, Linux
mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mount /dev/sda2 /mnt
mkdir -p /mnt/efi
mount /dev/sda1 /mnt/efi
# 从另外一台机器上
scp ~/arch.tar.gz root@ip:/mnt
# arch iso 里面
cd /mnt
tar -xvp --xattrs -f arch.tar.gz
rm arch.tar.gz
genfstab -U /mnt > /mnt/etc/fstab
arch-chroot /mnt /bin/bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

# 查看启动项目
efibootmgr
# 修改网络
ip a # 找到网卡的名称
vim /etc/systemd/network/*.conf # 更换上对应的名称
exit # 退出 chroot
umount -vfR /mnt
reboot
```