## 判断是否为正确的 ip 地址

```bash

valid_ip_opts="(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"
# do a simple test
echo "1.1.1.1" | grep -E -q ${valid_ip_opts} && echo "good ip address" || echo "bad ip address"

```

## 从多个源获取公网 ip 地址

```bash

mypubip () {

    valid_ip_opts='(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)'
    source_one='ifcfg.cn'
    source_two='http://members.3322.org/dyndns/getip'
    source_three='https://www.ipip.net/ip.html'

    # 容灾, 如果第一个source挂了, 就用第二个, 依此类推, 如果所有 souce 都挂了, 直接退出
    if curl -sL ${source_one} | grep -E -q ${valid_ip_opts} ; then
        curl -sL ${source_one}

    elif curl -sL ${source_two} | grep -E -q ${valid_ip_opts} ; then
        curl -sL ${source_two}

    elif curl -sL ${source_three} | grep value= | cut -d '"' -f6 | grep -E -q ${valid_ip_opts} ; then
        curl -sL ${source_three} | grep value= | cut -d '"' -f6

    else
        echo 'not able to get my public ip'
        exit 0
    fi

}

```

## Afraid.org DDNS 更新脚本

`cat /root/bin/ddns.sh`

> 脚本内容

```bash

#!/bin/bash
user_name='用户名'
pass_word='密码'
domain_name='域名'

valid_ip_opts="(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"
source_one='ifcfg.cn'
source_two='http://members.3322.org/dyndns/getip'
source_three='https://www.ipip.net/ip.html'

# 容灾, 如果第一个source挂了, 就用第二个, 依此类推, 如果所有 souce 都挂了, 直接退出
if curl -sL ${source_one} | grep -E -q ${valid_ip_opts} ; then
    china_ip=$(curl -sL ${source_one})

elif curl -sL ${source_two} | grep -E -q ${valid_ip_opts} ; then
    china_ip=$(curl -sL ${source_two})

elif curl -sL ${source_three} | grep value= | cut -d '"' -f6 | grep -E -q ${valid_ip_opts} ; then
    china_ip=$(curl -sL ${source_three} | grep value= | cut -d '"' -f6)

else
    echo 'not able to get my public ip'
    exit 0
fi
# update ip if there is a valid ${china_ip}
curl -sL "https://${user_name}:${pass_word}@freedns.afraid.org/nic/update?hostname=${domain_name}&myip=${china_ip}"

```
`crontable -e` 

> 每5分钟运行一次

````
3,8,13,18,23,28,33,38,43,48,53,58 * * * * /root/bin/ddns.sh > /dev/null &
````