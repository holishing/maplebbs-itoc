# MapleBBS-itoc GNU Debug mode

目前gdb方式: 將 Makefile 中 LDFLAG 裡的 -s 拿掉, CFLAGS 裡加上 -g.

若有任何建議, 還請不吝提點orz

# 快速安裝手冊 
**(encoding: utf-8)**

[原文件](http://processor.tfcis.org/~itoc/doc/101_install.htm)是寫給要安裝 itoc 所維護的 bbs 程式版本所使用的，並不適用其他 bbs 版本。

本 fork 調整調整內容:

source code 擺放的位置移到第一層以方便將 repositoty 改在 `/home/bbs` 下時做版本控制。

以及嘗試盡可能照[原本的壓縮檔](http://processor.tfcis.org/~itoc/download/MapleBBS-3.10-20121021-PACK.tgz)呈現補齊目錄格式 (by adding `.gitkeep`)
<br>若安裝完成後有刪除 `.gitkeep` 需求請自行用 `find` 等指令執行

將點歌範本(`ktv`)與相關安裝手冊(`gem`)併入現在`sample/bbs/`裡部分看板的精華區:

    ktv -> sample/bbs/gem/brd/ktv (直接取代原本只有一首歌的精華區)
    gem -> sample/bbs/gem/brd/admin (直接給有站長權限的使用者參考)

**以下安裝方式步驟可能會略有調整。**

# 一、安裝作業系統

本分支僅在 Debian 9 GNU/Linux (32bit) 測試過**勉強**可以運作。
<br>其他系統則沒有試過不清楚，這篇也會砍掉關於此部分，轉信設定部分也會砍
<br>對該部分有興趣建議按[原文件](http://processor.tfcis.org/~itoc/doc/101_install.htm)嘗試。

安裝作業系統時就像平常安裝一樣，沒什麼特別要注意的，唯一要提醒您的是，請安裝 **nano vim make gcc git xinetd** 等程式套件，因為 bbs 會用到。


# 二、建立 BBS 帳號

以 root 身分登入。

    -root- # mkdir /home/bbs
    -root- # vipw

如果您是 Linux 的話，在最後一行加上

    bbs:x:9999:999:BBS Administrator,,,:/home/bbs:/bin/bash
    (為求保險起見跟 `/etc/passwd` 裡列出其他使用者的格式一樣)

(當然您也可以用 `useradd`,`adduser` 的指令來完成相同的動作)

    -root- # vim /etc/group

(假設編輯器是 `vim`，如果不是的話，請自行改用其他編輯器如 `nano` )

如果您是 Linux 的話，在最後一行加上

    bbs:x:999:bbs

(當然您也可以用 `addgroup` 或 `groupadd` 的指令來完成相同的動作)

    -root- # passwd bbs

輸入 管理帳號 bbs 的密碼

    -root- # chown -R bbs:bbs /home/bbs

記得設定好bbs家目錄的權限, 尤其如果常以root身分從複製資料過去時更要注意

# 三、下載 BBS 程式

在 [這裡](https://github.com/holishing/maplebbs-itoc/tree/test-bbs) 可以找到最新的程式版本。

在 bbs 的家目錄(`/home/bbs`)載下來就對了

    -bbs- ~$ git clone https://github.com/holishing/maplebbs-itoc
    
# 四、安裝 BBS

以 bbs 身分登入，進去 source 目錄 (`/home/bbs/maplebbs-itoc`) 。

    -bbs- $ cd /home/bbs/maplebbs-itoc; cp sample/config.h include/
    -bbs- $ vim /home/bbs/maplebbs-itoc/include/config.h

修改 `HOST_ALIASES`，把您所有的 fqdn 都加進去

    #define HOST_ALIASES    {MYHOSTNAME, MYIPADDR, \
        "wolf.twbbs.org", "wolf.twbbs.org.tw", \
        NULL}

如果您是 Linux 的話，改 BBSGID 為 999

    #define BBSGID          999                     /* Linux 請設為 999 */


修改 schoolname bbsname ... 等數項，例如改成以下這樣
(請參考 `sample/sh/install.sh` 文件前面的註解，對名稱有些限制)

    schoolname="交大電子"
    bbsname="蘋果樂園"
    bbsname2="AppleBBS"
    sysopnick="站長大大"
    myipaddr="140.113.55.66"
    myhostname="nctu5566.dorm3.nctu.edu.tw"

之後還要記得將 bbs 預設的架構範例複製到家目錄(`/home/bbs`)底下建立起來:

    -bbs- $ cd /home/bbs/maplebbs-itoc (確認自己還在source目錄底下)
    -bbs- $ cp -r sample/bbs/* ~;cp sample/bbs/.* ~

之後確定自己在`/home/bbs/maplebbs-itoc/`目錄底下之後就開始編譯

    -bbs- $ make clean linux install 

如果您是使用 其他作業系統環境 的話，還要指令裡的 `linux` 改成其他類型如:
<br>`sun` `solaris` `sol-x86` `freebsd` `bsd`

您需要等待一段時間來完成編譯
<br>如果有相關編譯問題可參考 [原文件說明](http://processor.tfcis.org/~itoc) 或至各BBS站MapleBBS/BBS架站相關看板查詢

    -bbs- $ crontab /home/bbs/maplebbs-itoc/sample/crontab

把 sample/crontab 的內容加入 crontab 讓作業系統自動排程


# 五 (Ａ)、設定 BBS 環境 -- 如果有 xinetd

如果沒有 /etc/xinetd.d/ 這目錄，請跳到五（Ｂ或Ｃ），通常 Linux 應該有 xinetd 套件可以安裝才對。
在 Debian GNU/Linux 下可以嘗試用以下指令先行安裝並開啟相關服務

確定安裝完成後

以 root 身分登入。

    -root- # vim /etc/xinetd.d/telnet

將這檔案改成此內容 (這檔案有可能原本是沒有任何文字的開新檔案)

    service telnet
    {
            disable         = no
            flags           = REUSE
            socket_type     = stream
            wait            = yes
            user            = bbs
            server          = /home/bbs/bin/bbsd
            server_args     = -i
    }

.

    -root- # vim /etc/xinetd.d/xchat

將這檔案改成此內容 (這檔案有可能原本是沒有任何文字的開新檔案)

    service xchat
    {
            disable         = no
            flags           = REUSE
            socket_type     = stream
            wait            = yes
            user            = bbs
            server          = /home/bbs/bin/xchatd
            server_args     = -i
    }

.

    -root- # vim /etc/rc.local

加入以下數行 (這檔案有可能原本是沒有任何文字的開新檔案)

    #!/bin/sh
    #
    # MapleBBS
    #
    su bbs -c '/home/bbs/bin/camera'
    su bbs -c '/home/bbs/bin/account'

之後再

    -root- # chmod +x /etc/rc.local

這樣開機後 rc.local 才跑得動

# 五 (Ｂ)、設定 BBS 環境 -- 如果沒有 xinetd

沒 xinetd，改用 standalone 啟動。

以 root 身分登入。

    -root- # vim /etc/rc.local

加入以下數行 (這檔案有可能原本是沒有任何文字的開新檔案)

    #!/bin/sh
    #
    # MapleBBS
    #
    /home/bbs/bin/bbsd
    #/home/bbs/bin/bmtad   #如果沒要開啟相關服務可忽略
    #/home/bbs/bin/gemd    #如果沒要開啟相關服務可忽略
    #/home/bbs/bin/bguard  #如果沒要開啟相關服務可忽略
    #/home/bbs/bin/bpop3d  #如果沒要開啟相關服務可忽略
    #/home/bbs/bin/bnntpd  #如果沒要開啟相關服務可忽略
    /home/bbs/bin/xchatd
    #/home/bbs/innd/innbbsd #如果沒要開啟相關服務可忽略

    su bbs -c '/home/bbs/bin/camera'
    su bbs -c '/home/bbs/bin/account'

之後再

    -root- # chmod +x /etc/rc.local

這樣開機後 rc.local 才跑得動


# 六、其他設定

以 root 身分登入。

    -root- # vim /etc/services

加入以下數行

    xchat           3838/tcp
    xchat           3838/udp
    #bbsnntp         7777/tcp   usenet       #Network News Transfer Protocol #如果沒要開啟相關服務可忽略
    #bbsnntp         7777/udp   usenet       #Network News Transfer Protocol #如果沒要開啟相關服務可忽略

.

    -root- # reboot

重開機吧

# 七、享用您自己的 BBS

您的 BBS 應該已經架好了，試著 telnet 看看，那就這樣好好享用吧。

# --


交大電子 杜宇軒

itoc.bbs@bbs.tnfsh.tn.edu.tw

WWW: http://processor.tfcis.org/~itoc

## --

Adapt by holishing.bbs@ptt.cc

other post for ref.: [1](https://www.ptt.cc/bbs/Maple/M.1497541796.A.936.html) [2](https://www.ptt.cc/bbs/Maple/M.1497605619.A.383.html)
