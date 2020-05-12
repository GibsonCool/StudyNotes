### 启动 nginx 时提示80端口自身占用

&emsp;&emsp;查看nginx 状态的时候是没有运行的

```
[root@localhost conf.d]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since 六 2019-09-14 04:21:17 UTC; 6min ago
     Docs: http://nginx.org/en/docs/
  Process: 6483 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=1/FAILURE)

....
9月 14 04:21:17 localhost.localdomain systemd[1]: nginx.service: control process exited, code=exited status=1
9月 14 04:21:17 localhost.localdomain nginx[6483]: nginx: [emerg] still could not bind()
9月 14 04:21:17 localhost.localdomain systemd[1]: Failed to start nginx - high performance web server.
9月 14 04:21:17 localhost.localdomain systemd[1]: Unit nginx.service entered failed state.
9月 14 04:21:17 localhost.localdomain systemd[1]: nginx.service failed.
```

&emsp;&emsp;此时无论使用 systemctl 还是直接 nginx  都无法启动

```
[root@localhost conf.d]# systemctl start nginx
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
[root@localhost conf.d]# nginx
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] still could not bind()

// 查看 nginx 进程
[root@localhost conf.d]# ps -ef |grep nginx
root      6706     1  0 04:51 ?        00:00:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx     6707  6706  0 04:51 ?        00:00:00 nginx: worker process
root      6711  5029  0 04:54 pts/0    00:00:00 grep --color=auto nginx
```

&emsp;&emsp;我遇到过两次，均是由于我直接修改了 nginx.conf 文件且没有校验直接 reload 导致重启失败，所以查看状态的时候Nginx 的状态是 ”Active: failed (Result: exit-code)“，但是nginx 进程貌似依然占用了着端口，所以直接启动的时候提示  "bind() to 0.0.0.0:80 failed (98: Address already in use)"。

&emsp;&emsp;解决方案：强行杀死，在启动

```
[root@localhost conf.d]# pkill -9 nginx
[root@localhost conf.d]# systemctl start nginx
```



### 2、