# 无限期试用 macOS 上的 app

## 前提条件

- 这个 app 必须具备试用期

## 操作方法

- 我这里以 Carbon Copy Cloner.app 为例
- 首先去官网下载 [30天试用版](https://bombich.com)并安装
- 然后找到它的配置文件的路径, 终端操作
    ```bash
    find ~/Library -iname "*ccc*" 2>/dev/null | grep plist
    # 这条命令的意思是:
    # 在 ~/Library 搜索文件
    # 文件名包含 ccc, 大小写都可以, ccc是Carbon Copy Cloner的缩写
    # 别的软件可能不叫ccc, 别傻傻的复制黏贴
    # 然后通过 grep 只显示 包含 'plist' 关键字的搜索结果, 如果你搜到了多个, 自行判断哪个才是正确目标.
    ```
    得到 : 

    ```bash
    # xxx 是你的用户名, 别傻傻的复制黏贴
    /Users/xxx/Library/Preferences/com.bombich.ccc.plist
    ```
- 用 Xcode 或者 plist edit pro 打开这个 plist, 确认它包含试用天数的相关条目

    ```bash
    TrialStartDateV5 Date Nov 14, 2019 at 22:13:57
    # 每个软件有自己的试用期条目的名称和定义, 别傻傻的复制黏贴, 自行判断.
    ```
- 然后我们修改这个 `TrialStartDateV5` 的值

    ````
    我跟你们开玩笑的, 这样显然不行
    ````
- 正确的方式是, 等试用期快到了或者结束了, 彻底关闭 app, 然后直接删除这个文件

    ```bash
    # xxx 是你的用户名, 别傻傻的复制黏贴
    rm /Users/xxx/Library/Preferences/com.bombich.ccc.plist

    ```
- 然后这个文件会重新生成, 再打开看看, 是不是 `TrialStartDateV5` 更新了

- END

## 注意事项

- 通常这个 plist 文件也包含你对软件的自定义设置, 删除前, 导出自定义设置, 删除后, 导入, 可以保证不丢失配置. 
