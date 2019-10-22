## 设置

除了常规插件之外的设置

```bash
# 禁用命令纠正提示
unsetopt correct
# Redirect 的时候, 不显示前一个命令的输出
unsetopt MULTIOS
# 自动 rehash 命令, 比如安装了新的软件之后, 软件名立刻能补全
zstyle ":completion:*:commands" rehash 1
# 用兼容模式修改文件, > 可以直接将输出写入文件, 而不需要用 zsh 专用的 >! or >|
setopt clobber
# 保存命令日期
setopt EXTENDED_HISTORY
# 保存命令到 history 文件
setopt SHARE_HISTORY
# 显示所有历史命令的 function
h() {
fc -li 1
}

# 加载 alias 文件
source ~/alias
```
