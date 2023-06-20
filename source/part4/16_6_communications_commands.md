# 16.6 通信命令

以下某些命令可用于网络数据传输和分析以及[定位垃圾邮件发送者](https://tldp.org/LDP/abs/html/writingscripts.html#CSPAMMERS)。

## 信息和统计命令

### host

使用DNS(Domain Name System 域名解析协议)，根据主机名或ip地址搜索相关Internet主机的信息。

```shell
bash$ host surfacemail.com
surfacemail.com. has address 202.92.42.236
```

### ipcalc

显示主机的IP信息。当使用`-h`选项时，**ipcalc**将进行反向DNS查找，根据ip地址查找主机 (服务器) 的名称。

```shell
bash$ ipcalc -h 202.92.42.236
HOSTNAME=surfacemail.com
```

### nslookup

通过ip地址在主机上查找Internet “名称服务器”。这基本上等同于**ipcalc -h**或**dig -x**。该命令可以交互地或非交互地运行，即在脚本中运行。

据称**nslookup**命令已被 “弃用”，但它仍然有用。

```shell
bash$ nslookup -sil 66.97.104.180
nslookup kuhleersparnis.ch
 Server:         135.116.137.2
 Address:        135.116.137.2#53

 Non-authoritative answer:
 Name:   kuhleersparnis.ch
```

### dig

域信息扒手。类似与**nslookup**命令，*dig*命令在主机上*查找*Internet*名称服务器*。可以从命令行或脚本中运行。

*dig*有一些有趣的选项，`+time=N`将请求超时时间设置为*N*秒，`+nofail`直到收到回复持续向服务器发起请求，`-x`反向地址查找。

请比较**dig -x**与**ipcalc -h**和**nslookup**之间的输出结果的差异。

```shell
bash$ dig -x 81.9.6.2
;; Got answer:
 ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 11649
 ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

 ;; QUESTION SECTION:
 ;2.6.9.81.in-addr.arpa.         IN      PTR

 ;; AUTHORITY SECTION:
 6.9.81.in-addr.arpa.    3600    IN      SOA     ns.eltel.net. noc.eltel.net.
 2002031705 900 600 86400 3600

 ;; Query time: 537 msec
 ;; SERVER: 135.116.137.2#53(135.116.137.2)
 ;; WHEN: Wed Jun 26 08:35:24 2002
 ;; MSG SIZE  rcvd: 91
```

**样例16-40. 找出垃圾邮件发送者**

```shell
#!/bin/bash
# spam-lookup.sh: 查找被滥用的联系人以举报垃圾邮件发送者。
# 感谢Michael Zick.

# 检查命令行参数。
ARGCOUNT=1
E_WRONGARGS=85
if [ $# -ne "$ARGCOUNT" ]
then
  echo "Usage: `basename $0` domain-name"
  exit $E_WRONGARGS
fi


dig +short $1.contacts.abuse.net -c in -t txt
# 你也可以尝试以下命令：
#     dig +nssearch $1
#     尝试找到"权威名称服务器"并且展示SOA记录。

# 以下命令同样奏效：
#     whois -h whois.abuse.net $1
#           ^^ ^^^^^^^^^^^^^^^  指定的域名。
#     甚至可以用这个查找多个垃圾邮件发送者，即
#     whois -h whois.abuse.net $spamdomain1 $spamdomain2 . . .


#  练习：
#  --------
#  扩展此脚本的功能，
#  使它能够自动通过电子邮件通知到
#  相关负责的ISP的联系地址。
#  提示：使用"mail"命令。

exit $?

# spam-lookup.sh chinatietong.com
#                已知的垃圾邮件域名。

# "crnet_mgr@chinatietong.com"
# "crnet_tec@chinatietong.com"
# "postmaster@chinatietong.com"


#  对于这个脚本的更详细的版本，
#  请前往SpamViz主页。http://www.spamviz.net/index.html。
```

**样例16-41. 分析一个垃圾邮件域名**

```shell
#! /bin/bash
# is-spammer.sh: 识别垃圾邮件域名

# $Id: is-spammer, v 1.4 2004/09/01 19:37:52 mszick Exp $
# 上一行是RCS ID信息。
#
#  这是附录Contributed Scripts中
#  is_spammer.bash脚本的简化版本。

# is-spammer <domain.name>

# 使用的外部程序：'dig'
# 测试版本：9.2.4rc5

# 使用函数。
# 使用IFS赋值到数组的方式来解析字符串。
# 甚至做了一些其他有用的事情：检查电子邮箱黑名单。

# 使用文本正文中的domain.name(s)：
# http://www.good_stuff.spammer.biz/just_ignore_everything_else
#                       ^^^^^^^^^^^
# 或者使用任意电子邮箱地址的domain.name(s):
# Really_Good_Offer@spammer.biz
#
# 作为此脚本的唯一参数。
#(PS: 确保你的Inet运行正常)
#
# 因此，在上述两个实例中调用此脚本的方法为:
#       is-spammer.sh spammer.biz


# Whitespace == :Space:Tab:Line Feed:Carriage Return:
WSP_IFS=$'\x20'$'\x09'$'\x0A'$'\x0D'

# No Whitespace == Line Feed:Carriage Return
No_WSP=$'\x0A'$'\x0D'

# 点分割的十进制ip地址的字段分隔符
ADR_IFS=${No_WSP}'.'

# 获取dns文本资源记录。
# get_txt <error_code> <list_query>
get_txt() {

    # 在点处解析$1
    local -a dns
    IFS=$ADR_IFS
    dns=( $1 )
    IFS=$WSP_IFS
    if [ "${dns[0]}" == '127' ]
    then
        # 查看是否有原因
        echo $(dig +short $2 -t txt)
    fi
}

# 获取dns地址资源记录。
# chk_adr <rev_dns> <list_server>
chk_adr() {
    local reply
    local server
    local reason

    server=${1}${2}
    reply=$( dig +short ${server} )

    # 如果回复是错误代码. . .
    if [ ${#reply} -gt 6 ]
    then
        reason=$(get_txt ${reply} ${server} )
        reason=${reason:-${reply}}
    fi
    echo ${reason:-' not blacklisted.'}
}

# 需要从域名中获取IP地址。
echo 'Get address of: '$1
ip_adr=$(dig +short $1)
dns_reply=${ip_adr:-' no answer '}
echo ' Found address: '${dns_reply}

# 一个有效的回复至少包含4个数字和3个点。
if [ ${#ip_adr} -gt 6 ]
then
    echo
    declare query

    # 通过点处的赋值来解析。
    declare -a dns
    IFS=$ADR_IFS
    dns=( ${ip_adr} )
    IFS=$WSP_IFS

    # 将八进制数重新排序为dns查询顺序。
    rev_dns="${dns[3]}"'.'"${dns[2]}"'.'"${dns[1]}"'.'"${dns[0]}"'.'

# 参见：http://www.spamhaus.org (传统型， 维护良好)
    echo -n 'spamhaus.org says: '
    echo $(chk_adr ${rev_dns} 'sbl-xbl.spamhaus.org')

# 参见：http://ordb.org (开放邮件中继)
    echo -n '   ordb.org  says: '
    echo $(chk_adr ${rev_dns} 'relays.ordb.org')

# 参见：http://www.spamcop.net/ (你可以在该网站报告垃圾邮件发送者)
    echo -n ' spamcop.net says: '
    echo $(chk_adr ${rev_dns} 'bl.spamcop.net')

# # # 其他黑名单操作 # # #

# 参见：http://cbl.abuseat.org。
    echo -n ' abuseat.org says: '
    echo $(chk_adr ${rev_dns} 'cbl.abuseat.org')

# 参见：http://dsbl.org/usage (各种邮件中继)
    echo
    echo 'Distributed Server Listings'
    echo -n '       list.dsbl.org says: '
    echo $(chk_adr ${rev_dns} 'list.dsbl.org')

    echo -n '   multihop.dsbl.org says: '
    echo $(chk_adr ${rev_dns} 'multihop.dsbl.org')

    echo -n 'unconfirmed.dsbl.org says: '
    echo $(chk_adr ${rev_dns} 'unconfirmed.dsbl.org')

else
    echo
    echo 'Could not use that address.'
fi

exit 0

# 练习:
# --------

# 1) 检查脚本的参数，
#    在必要时打印适当的错误消息并退出。

# 2) 在调用脚本时检查是否在线，
#    在必要时打印适当的错误消息并退出。

# 3) 将全局变量替换为"硬编码"BHL域(domain)。

# 4) 通过给'dig'命令添加"+time="选项来给脚本设置超时时间。
```

有关上述脚本的更详细的版本，请参阅[样例A-28](https://tldp.org/LDP/abs/html/contributed-scripts.html#ISSPAMMER2)。

### traceroute

跟踪发送到远程主机的数据包所采用的路由。此命令对LAN（局域网）、WAN（广域网）以及Internet均有效。远程主机可以由ip地址指定。该命令的输出可以由管道中的[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)或[sed](https://tldp.org/LDP/abs/html/sedawk.html#SEDREF) 过滤。

```shell
bash$ traceroute 81.9.6.2
traceroute to 81.9.6.2 (81.9.6.2), 30 hops max, 38 byte packets
 1  tc43.xjbnnbrb.com (136.30.178.8)  191.303 ms  179.400 ms  179.767 ms
 2  or0.xjbnnbrb.com (136.30.178.1)  179.536 ms  179.534 ms  169.685 ms
 3  192.168.11.101 (192.168.11.101)  189.471 ms  189.556 ms *
 ...
```

### ping

能够将<code>*ICMP ECHO_REQUEST*</code>数据包广播到本地或远程网络上的另一台主机。这是用于测试网络连接的诊断工具，应谨慎使用。

```shell
bash$ ping localhost
PING localhost.localdomain (127.0.0.1) from 127.0.0.1 : 56(84) bytes of data.
 64 bytes from localhost.localdomain (127.0.0.1): icmp_seq=0 ttl=255 time=709 usec
 64 bytes from localhost.localdomain (127.0.0.1): icmp_seq=1 ttl=255 time=286 usec

 --- localhost.localdomain ping statistics ---
 2 packets transmitted, 2 packets received, 0% packet loss
 round-trip min/avg/max/mdev = 0.286/0.497/0.709/0.212 ms
```

一个成功的*ping*返回0的[退出状态](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)。可以在脚本中进行测试。

```shell
  HNAME=news-15.net  # 臭名昭著的垃圾邮件发送者。
# HNAME=$HOST     # 调试： 测试localhost。
  count=2  # 只发送两次ping。

if [[ `ping -c $count "$HNAME"` ]]
then
  echo ""$HNAME" still up and broadcasting spam your way."
else
  echo ""$HNAME" seems to be down. Pity."
fi
```

### whois

执行DNS (域名系统) 查找。`-h`选项允许指定要查询哪个特定的*whois*服务器。参阅[样例4-6](https://tldp.org/LDP/abs/html/othertypesv.html#EX18)和[样例16-40](https://tldp.org/LDP/abs/html/communications.html#SPAMLOOKUP)。

### finger

检索有关网络上用户的信息。此命令可以选择显示用户的`~/.plan`、`~/.project` 和`~/.forward`文件，如果以上文件存在。

```shell
bash$ finger
Login  Name           Tty      Idle  Login Time   Office     Office Phone
 bozo   Bozo Bozeman   tty1        8  Jun 25 16:59                (:0)
 bozo   Bozo Bozeman   ttyp0          Jun 25 16:59                (:0.0)
 bozo   Bozo Bozeman   ttyp1          Jun 25 17:07                (:0.0)



bash$ finger bozo
Login: bozo                             Name: Bozo Bozeman
 Directory: /home/bozo                   Shell: /bin/bash
 Office: 2355 Clown St., 543-1234
 On since Fri Aug 31 20:13 (MST) on tty1    1 hour 38 minutes idle
 On since Fri Aug 31 20:13 (MST) on pts/0   12 seconds idle
 On since Fri Aug 31 20:13 (MST) on pts/1
 On since Fri Aug 31 20:31 (MST) on pts/2   1 hour 16 minutes idle
 Mail last read Tue Jul  3 10:08 2007 (MST) 
 No Plan.
```

出于安全考虑，许多网络禁用了**finger**及其关联的守护程序(daemon)。[[1]](https://tldp.org/LDP/abs/html/communications.html#FTN.AEN13320)

### chfn

改变**finger**命令揭露的信息。

### vrfy

验证Internet电子邮件地址。

较新的Linux发行版似乎缺少此命令。

## 远程主机访问命令

### sx, rx

**sx**和**rx**命令集使用*xmodem*协议在远程主机之间传输文件。这些通常是通信包的一部分，例如**minicom**。

### sz, rz

**sz**和**rz**命令集使用*zmodem*协议在远程主机之间传输文件。*Zmodem*比*xmodem*有一定的优势，例如更快的传输速率和恢复中断的文件传输。与**sx**和**rx**命令集一样，它们通常是通信包的一部分。

### ftp

用于远程主机中上传/下载文件的实用程序和协议。ftp会话可以在脚本中自动化 (请参阅[样例19-6](https://tldp.org/LDP/abs/html/here-docs.html#EX72)和[样例A-4](https://tldp.org/LDP/abs/html/contributed-scripts.html#ENCRYPTEDPW))。

### uucp, uux, cu

**uucp**：*Unix 拷贝到 Unix*。这是一个用于在UNIX服务器之间传输文件的通信包。shell脚本是处理**uucp**命令序列的有效方法。

自从互联网和电子邮件出现以来，**uucp**似乎已经变得晦涩难懂，但它仍然存在，并且在互联网连接不可用或不合适的情况下仍然完全可行。**uucp**的优点在与它的容错能力，因此即使存在服务中断，复制操作也会在恢复连接时从停止的地方恢复。

---

**uux**：*Unix 到 Unix执行*。在远程系统上执行一条指令。这条指令是**uucp**包的一部分。

---

**cu**：调用远程系统并作为一个简单的终端进行连接。这是[telnet](https://tldp.org/LDP/abs/html/communications.html#TELNETREF)的愚蠢版本。该命令是**uucp**包的一部分。

### telnet

用于连接到远程主机的实用程序和协议。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)telnet协议包含安全漏洞，因此应该尽量避免使用。不建议在shell脚本中使用它。

### wget

**wget**实用程序以*非交互方式*从Web或ftp站点检索或下载文件。它非常适合用于脚本中。

```shell
wget -p http://www.xyz23.com/file01.html
#  -p或者--page-requisite选项使得wget命令可以获取
#  指定页面显示的所有文件。

wget -r ftp://ftp.xyz24.net/~bozo/project_files/ -O $SAVEFILE
#  -r选项递归地跟随并检索指定站点上的所有链接。

wget -c ftp://ftp.xyz25.net/bozofiles/filename.tar.bz2
#  -c选项使得wget命令可以断点下载。
#  这在ftp服务器和许多HTTP站点上均有效。
```

**样例16-42. 获取股票报价**

```shell
#!/bin/bash
# quote-fetch.sh: 下载一个股票报价。


E_NOPARAMS=86

if [ -z "$1" ]  # 必须指定要获取的股票 (代号)。
  then echo "Usage: `basename $0` stock-symbol"
  exit $E_NOPARAMS
fi

stock_symbol=$1

file_suffix=.html
# 获取HTML文件，因此请适当命名。
URL='http://finance.yahoo.com/q?s='
# 雅虎金融板块，后缀查询的股票代号。

# -----------------------------------------------------------
wget -O ${stock_symbol}${file_suffix} "${URL}${stock_symbol}"
# -----------------------------------------------------------


# 要查询http://search.yahoo.com上的内容:
# -----------------------------------------------------------
# URL="http://search.yahoo.com/search?fr=ush-news&p=${query}"
# wget -O "$savefilename" "${URL}"
# -----------------------------------------------------------
# 保存相关URL的列表。

exit $?

# 练习：
# ---------
#
# 1) 添加测试以确保用户运行脚本时网络在线。
#    (提示：解析"ppp"或者"connect"的'ps -ax'命令的输出。)
#
# 2) 修改此脚本以获取本地天气报告，
#    并将用户的邮政编码作为参数。
```

另请参阅[样例A-30](https://tldp.org/LDP/abs/html/contributed-scripts.html#WGETTER2)和[样例A-31](https://tldp.org/LDP/abs/html/contributed-scripts.html#BASHPODDER)。

### lynx

**lynx** Web和文件浏览器可以在脚本 (带有`-dump`选项) 中使用，以非交互方式从Web或ftp站点检索文件。

```shell
lynx -dump http://www.xyz23.com/file01.html >$SAVEFILE
```

使用`-traversal`选项，**lynx**会从指定为参数的HTTP URL开始，然后 “抓取” 位于指定服务器上的所有链接。与`-crawl`选项一起使用，可以将页面文本输出到日志中。

### rlogin

<code>*远程登录*</code>，在远程主机上启动一个会话。这个命令存在安全隐患，请使用[ssh](https://tldp.org/LDP/abs/html/communications.html#SSHREF)替代。

### rsh

<code>*远程shell*</code>，在远程主机上执行命令。这个命令存在安全隐患，请使用[ssh](https://tldp.org/LDP/abs/html/communications.html#SSHREF)替代。

### rcp

<code>*远程复制*</code>，在两台不同的联网主机之间复制文件。

### rsync

<code>*远程同步*</code>，在两个不同的联网主机之间更新 (同步) 文件。

```shell
bash$ rsync -a ~/sourcedir/*txt /node1/subdirectory/
```

**样例16-43. 升级FC4**

```shell
#!/bin/bash
# fc4upd.sh

# 脚本作者：Frank Wang.
# 本书作者对风格进行了轻微修改。
# 经许可在本书中使用。


#  使用rsync从镜像站点下载Fedora Core 4更新。
#  对与更新版的Fedora Core应该也同样奏效，如5, 6, . . .
#  多版本同时存在时仅下载最新版本，
#  以节省空间。

URL=rsync://distro.ibiblio.org/fedora-linux-core/updates/
# URL=rsync://ftp.kddilabs.jp/fedora/core/updates/
# URL=rsync://rsync.planetmirror.com/fedora-linux-core/updates/

DEST=${1:-/var/www/html/fedora/updates/}
LOG=/tmp/repo-update-$(/bin/date +%Y-%m-%d).txt
PID_FILE=/var/run/${0##*/}.pid

E_RETURN=85        # 非预期的事情发生了。


# 通用rsync选项
# -r: 递归下载
# -t: 预留时间
# -v: 详细模式

OPTS="-rtv --delete-excluded --delete-after --partial"

# rsync的包含模式
# 前导斜杠匹配绝对路径名。
INCLUDE=(
    "/4/i386/kde-i18n-Chinese*" 
#   ^                         ^
# 引号是必要的，防止调用glob。
) 


# rsync的排除模式
# 使用"#"暂时注释掉不需要的包. . .
EXCLUDE=(
    /1
    /2
    /3
    /testing
    /4/SRPMS
    /4/ppc
    /4/x86_64
    /4/i386/debug
   "/4/i386/kde-i18n-*"
   "/4/i386/openoffice.org-langpack-*"
   "/4/i386/*i586.rpm"
   "/4/i386/GFS-*"
   "/4/i386/cman-*"
   "/4/i386/dlm-*"
   "/4/i386/gnbd-*"
   "/4/i386/kernel-smp*"
#  "/4/i386/kernel-xen*" 
#  "/4/i386/xen-*" 
)


init () {
    # 让管道命令返回可能的rsync错误，例如网络中断。
    set -o pipefail                  # Bash 3新引入的特性。

    TMP=${TMPDIR:-/tmp}/${0##*/}.$$  # 存储下载目录。
    trap "{
        rm -f $TMP 2>/dev/null
    }" EXIT                          # 退出时清空临时文件。
}


check_pid () {
# 检查进程是否存在。
    if [ -s "$PID_FILE" ]; then
        echo "PID file exists. Checking ..."
        PID=$(/bin/egrep -o "^[[:digit:]]+" $PID_FILE)
        if /bin/ps --pid $PID &>/dev/null; then
            echo "Process $PID found. ${0##*/} seems to be running!"
           /usr/bin/logger -t ${0##*/} \
                 "Process $PID found. ${0##*/} seems to be running!"
            exit $E_RETURN
        fi
        echo "Process $PID not found. Start new process . . ."
    fi
}


#  根据上述模式，
#  设置从根或$URL开始的总体文件更新范围。
set_range () {
    include=
    exclude=
    for p in "${INCLUDE[@]}"; do
        include="$include --include \"$p\""
    done

    for p in "${EXCLUDE[@]}"; do
        exclude="$exclude --exclude \"$p\""
    done
}


# 检索和细化rsync更新列表。
get_list () {
    echo $$ > $PID_FILE || {
        echo "Can't write to pid file $PID_FILE"
        exit $E_RETURN
    }

    echo -n "Retrieving and refining update list . . ."

    # 检索列表 -- 'eval'用于将rsync运行为单个命令。
    # $3 和 $4 是文件创建日期和时间。
    # $5 是包的全名。
    previous=
    pre_file=
    pre_date=0
    eval /bin/nice /usr/bin/rsync \
        -r $include $exclude $URL | \
        egrep '^dr.x|^-r' | \
        awk '{print $3, $4, $5}' | \
        sort -k3 | \
        { while read line; do
            # 从时期中获取秒数，以过滤掉过时的包。
            cur_date=$(date -d "$(echo $line | awk '{print $1, $2}')" +%s)
            #  echo $cur_date

            # Get file name.
            cur_file=$(echo $line | awk '{print $3}')
            #  echo $cur_file

            # 可以的话，从文件名中获取rpm包名。
            if [[ $cur_file == *rpm ]]; then
                pkg_name=$(echo $cur_file | sed -r -e \
                    's/(^([^_-]+[_-])+)[[:digit:]]+\..*[_-].*$/\1/')'
            else
                pkg_name=
            fi
            # echo $pkg_name

            if [ -z "$pkg_name" ]; then   #  如果不是rpm文件，
                echo $cur_file >> $TMP    #  就把它添加到下载列表。
            elif [ "$pkg_name" != "$previous" ]; then   # 找到一个新的包。
                echo $pre_file >> $TMP                  # 输出最新的文件。
                previous=$pkg_name                      # 保存现在的包。
                pre_date=$cur_date
                pre_file=$cur_file
            elif [ "$cur_date" -gt "$pre_date" ]; then
                                                #  如果是相同的包，但更新，
                pre_date=$cur_date              #  则更新latest指针。
                pre_file=$cur_file
            fi
            done
            echo $pre_file >> $TMP              #  TMP现在包含所有优化列表。

            # echo "subshell=$BASH_SUBSHELL"

    }       # 此处需要括号以使最终"echo $pre_file >> $TMP"成功执行。
            # 与整个循环保持在同一子shell(1)中。

    RET=$?  # 获取管道命令的返回码。

    [ "$RET" -ne 0 ] && {
        echo "List retrieving failed with code $RET"
        exit $E_RETURN
    }

    echo "done"; echo
}

# 真正的rsync下载部分。
get_file () {

    echo "Downloading..."
    /bin/nice /usr/bin/rsync \
        $OPTS \
        --filter "merge,+/ $TMP" \
        --exclude '*'  \
        $URL $DEST     \
        | /usr/bin/tee $LOG

    RET=$?

   #  --filter merge,+/ 至关重要。
   #  + 修饰器指包含，/ 指绝对路径。 
   #  然后$TMP中的排序列表将包含升序目录名称，
   #  并从 --exclude '*'中排除防止“短路”。

    echo "Done"

    rm -f $PID_FILE 2>/dev/null

    return $RET
}

# -------
# 主函数
init
check_pid
set_range
get_list
get_file
RET=$?
# -------

if [ "$RET" -eq 0 ]; then
    /usr/bin/logger -t ${0##*/} "Fedora update mirrored successfully."
else
    /usr/bin/logger -t ${0##*/} \
    "Fedora update mirrored with failure code: $RET"
fi

exit $RET
```

另请参阅[样例A-32](https://tldp.org/LDP/abs/html/contributed-scripts.html#NIGHTLYBACKUP)。

> ![note](https://tldp.org/LDP/abs/images/note.gif)在shell脚本中不建议使用[rcp](https://tldp.org/LDP/abs/html/communications.html#RCPREF)、[rsync](https://tldp.org/LDP/abs/html/communications.html#RSYNCREF)和类似的具有安全隐患的实用程序。相反，请考虑使用**ssh**、[scp](https://tldp.org/LDP/abs/html/communications.html#SCPREF)或**expect**脚本。

### ssh

<code>*安全的shell*</code>，登录到远程主机并在那里执行命令。它使用了身份认证和加密技术，是**telnet**、**rlogin**、**rcp**和**rsh**的安全替代品。请参阅其[man手册](https://tldp.org/LDP/abs/html/basic.html#MANREF)。

**样例16-44. 使用*ssh*命令**

```shell
#!/bin/bash
# remote.bash: 使用ssh命令。

# 此样例来自于Michael Zick.
# 授权使用。


#   假设：
#   ------------
#   fd-2 没有被捕获( '2>/dev/null' ).
#   ssh/sshd 推测标准错误stderr ('2')会展示给用户。
#
#   sshd 正在你的主机上运行。
#   对于任何 “标准” 发行版，可能如此，
#   并且没有装载任何ssh-keygen。

# 使用命令行尝试ssh连接你的计主机
#
# $ ssh $HOSTNAME
# 如果没有额外的设置，你会被要求输入密码。
#   输入密码
#   做完后,  $ exit
#
# 有用吗？如果是这样，你已经准备好享受更多的乐趣了。

# 使用'root'用户登录你的计算主机#
#   $  ssh -l root $HOSTNAME
#   当询问密码时，输入root的密码而不是你的。
#          Last login: Tue Aug 10 20:25:49 2004 from localhost.localdomain
#   当完成后键入'exit'。

#  上面给了你一个交互式shell。
#  sshd可以设置为 “单一命令” 模式，
#  但这超出了本示例的范围。
#  唯一需要注意的是，
#  以下内容将在 “单一命令” 模式下工作。


# 一个基本的，写入标准输出stdout(本地)的命令。

ls -l

# 现在同样在远程主机上执行相同的基本命令。
# 当期望传递不同的'USERNAME'、'HOSTNAME'时：
USER=${USERNAME:-$(whoami)}
HOST=${HOSTNAME:-$(hostname)}

#  现在，在远程主机上执行上述命令行，
#  并加密所有传输。

ssh -l ${USER} ${HOST} " ls -l "

#  预期的结果是列出远程主机主机用户名主目录。
#  要查看任何差异，请在主目录以外的任何地方运行此脚本。

#  换句话说，Bash命令作为引用行传递给远程shell，
#  该命令在远程主机上执行。
#  在这种情况下，sshd以你的名义执行' bash -c "ls -l" '。

#  有关其他内容，例如不必为每个命令行输入密码，请参阅
#     man ssh
#     man ssh-keygen
#     man sshd_config。

exit 0
```

> ![note](https://tldp.org/LDP/abs/images/caution.gif)在循环中，**ssh**可能会导致非预期的行为。根据comp.unix shell存档中的[Usenet帖子](http://groups-beta.google.com/group/comp.unix.shell/msg/dcb446b5fff7d230)所述，ssh继承了循环的stdin。要解决此问题，请将`-n`或`-f`选项传递给**ssh**。

感谢 Jason Bechtel指出。

### scp

<code>*安全的复制行为*</code>，在功能上类似于**rcp**，在两个不同的联网主机之间复制文件，但这样做需要身份验证，并具有类似于**ssh**的安全级别。

## 本地网络命令

### write

这是用于终端到终端通信的实用程序。它允许将内容从你的终端 (控制台或*xterm*) 发送到另一个用户的。当然，[mesg](https://tldp.org/LDP/abs/html/system.html#MESGREF)命令可以用于禁用对终端的写权限。

由于**write**是交互式的，因此通常不会用于脚本。

### netconfig

用于配置网络适配器 (使用*DHCP*) 的命令行实用程序。此命令源于Red Hat centric Linux发行版。

## 邮件命令

## mail

发送或者阅读电子邮件信息。

这个精简的命令行邮件客户端适配脚本使用。

**样例16-45. 一个给自己发邮件的脚本**

```shell
#!/bin/sh
# self-mailer.sh: 给自己发邮件的脚本

adr=${1:-`whoami`}     # 如果没有指定，选择当前用户。
#  键入 'self-mailer.sh wiseguy@superdupergenius.com'
#  会将此脚本发送给该收件人。
#  仅仅执行'self-mailer.sh'(没有参数)会将该脚本发送给
#  调用它的用户，例如，bozo@localhost.localdomain。
#
#  了解更多${parameter:-default}结构，
#  请参阅"换个角度看变量"章节的"变量替换"板块。

# ============================================================================
  cat $0 | mail -s "Script \"`basename $0`\" has mailed itself to you." "$adr"
# ============================================================================

# --------------------------------------------
#  来自自我邮寄脚本的问候。
#  一个调皮的人运行了这个脚本，这导致它把自己邮寄给你。
#  显然，有些人闲得发慌。
# --------------------------------------------

echo "At `date`, script \"`basename $0`\" mailed to "$adr"."

exit 0

#  请注意，(在"发送"模式下的)"mailx"命令可能可以被"mail"命令替换 ...
#  但是选项会有所不同。
```

### mailto

与**mail**命令类似，**mailto**从命令行或脚本中发送电子邮件。但是，**mailto**也允许发送MIME (多媒体) 消息。

### mailstats

显示*邮件统计信息*。此命令只能由*root用户*调用。

```shell
root# mailstats
Statistics from Tue Jan  1 20:32:08 2008
  M   msgsfr  bytes_from   msgsto    bytes_to  msgsrej msgsdis msgsqur  Mailer
  4     1682      24118K        0          0K        0       0       0  esmtp
  9      212        640K     1894      25131K        0       0       0  local
 =====================================================================
  T     1894      24758K     1894      25131K        0       0       0
  C      414                    0
```

### vacation

此实用程序会以电子邮件的形式自动回复预定收件人正在休假且暂时不可用。该命令在联机环境下运行并与**sendmail**结合使用，不适用于拨号POPmail帐户。

## 注记

[[1]](https://tldp.org/LDP/abs/html/communications.html#AEN13320)*守护进程*是未附加(attach)到终端会话的后台进程。守护进程是在指定时间或由某些事件显式触发从而执行的特定服务。
