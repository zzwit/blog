#不小心 crontab -r 内容恢复
> 从 cron cp 一个临时文件
```
cat /var/log/cron* | grep -i "`which cron`" > ./all_temp
cat  ./all_temp | grep -v "<command>" > ./cmd_temp
```
> 用awk读取cmd_temp,即可得到命令。

```
cat cmd_temp  | grep username | awk -F 'CMD' '{print $2}' | sort | uniq | head -n 30
查看-所有的执行的命令的 list
 (bash   /home/appadmin/script/security/auto_restart.sh)
```
> 查看所有的命令的定时的执行是多少

```
cat cmd_temp  | grep /home/appadmin/script/security/auto_restart.sh | head -n 30
查看-所有的执行的命令的 list
Sep 11 03:44:01 script-02 CROND[18487]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:45:01 script-02 CROND[18522]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:46:01 script-02 CROND[18844]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:47:01 script-02 CROND[18870]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:48:01 script-02 CROND[18901]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:49:01 script-02 CROND[19036]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:50:01 script-02 CROND[19073]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:51:01 script-02 CROND[19147]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:52:01 script-02 CROND[19286]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
Sep 11 03:53:01 script-02 CROND[19312]: (appadmin) CMD (bash   /home/appadmin/script/security/auto_restart.sh)
```
> 通过上边的执行记录 每分钟执行一次

 总结： 通过上面的操作基本就找回了定时脚本的内容 
    - 将定时脚本的备份
    - 执行命令要谨慎


