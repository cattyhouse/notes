## 用户账户

```bash
useradd -m -G users,wheel -s /bin/zsh username
# 新建一个普通用户
# -m 创建 home
# -G 加入 group
# -s 定义使用的 shell

useradd -r -s /usr/sbin/nologin username # debian, ubuntu
useradd -r -s /usr/bin/nologin username # archlinux
# 创建三无用户, 无 home, 无密码, 无 login
# 这样的用户有什么用呢? 用来运行一些程序, 提高安全性.

useradd -D -s /bin/zsh
# 更新当前用户 shell
# 更好的方法是 chsh -s /bin/zsh username

userdel -rf username
# -f 强制删除用户, 即使是登陆状态
# -r 删除 /home/ 下面的用户文件夹

passwd username
# 修改用户密码

passwd -d -l username 
# -d 设置密码为空, -l 禁止修改密码, 同时禁止此用户密码登陆

passwd -u username
# 取消 lock 状态, 也就是 unlock

passwd -aS 
# 查看所有用户状态, L 表示密码被锁定且不允许修改, NP 表示没有密码, P 表示密码正常.
# | grep -w P 可以快速寻找系统中有密码的用户

cat /etc/passwd | grep -Ev 'nologin|false'
# 查询系统中有 shell 的用户
```