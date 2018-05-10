---
layout: post
author: "Paycis"
title: 리눅스 서버 배포환경 구축하기 3 - 톰캣 설치 및 아파치 연동
tag: [리눅스,Tomcat]
---

이번 순서는 톰캣설치입니다. [톰캣설치하기](https://suwoni-codelab.com/linux/2017/05/28/Linux-CentOS-Tomcat/)
*수워니님 포스팅을 참고하여 진행했습니다.

톰캣설치 과정에서 특별하게 막혔던 부분은 없었습니다만.. 아파치와 연동하는 과정에서 뭔가 이상한 일들이 많이 생겼었습니다.

[아파치-톰캣 연동하기](https://suwoni-codelab.com/linux/2017/05/29/Linux-CentOS-Apache-Tomcat/)

# Apache설정 (httpd.conf)

VirtualHost를 사용하지 않고 다르게 진행했습니다.

### LoadModule

httpd.conf 파일을 아래와 같이 수정했습니다.

{% highlight markdown %}
...

# Example:
# LoadModule foo_module modules/mod_foo.so
#
LoadModule jk_module modules/mod_jk.so
<IfModule mod_jk.c>
     JkWorkersFile conf/workers.properties
     #JkShmFile 내용 반드시 넣어줄 것 - Selinux 보안때문에 공유 메모리파일 위치 반드시 명시해야함
     JkShmFile run/mod_jk.shm
     JkLogFile logs/mod_jk.log
     JkLogLevel info
     JkLogStampFormat "[%y %m %d %H:%M:%S] "
     JkMountFile conf/uriworkermap.properties
</IfModule>


Include conf.modules.d/*.conf

#
# If you wish httpd to run as a different user or group, you must run
# httpd as root initially and it will switch.

...

*JkMount 내용을 uriworkermap.properties 파일로 분리
*/etc/httpd/conf.modules.d/mod_jk.conf 파일 생성 안하고 해당내용 httpd.conf에 작성

{% endhighlight %}


### httpd 기동이 안될 때

{% highlight markdown %}
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: Starting The Apache HTTP Server...
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: httpd.service: main process exited, code=exited, status=1/FAILURE
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal kill[18250]: kill: cannot find process ""
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: httpd.service: control process exited, code=exited status=1
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: Failed to start The Apache HTTP Server.
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: Unit httpd.service entered failed state.
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: httpd.service failed.
{% endhighlight %}

`systemctl -l status httpd`로 상태확인을 했더니 위와같이 나온다면 아래처럼 Selinux설정을 반영하는 명령어를 추가로 입력하면 해결된다.

{% highlight markdown %}
chcon -R --type=httpd_sys_rw_content_t /usr/share/tomcat/webapps/ROOT

setenforce 0  //설정반영 - 추가입력

{% endhighlight %}

위와같이 입력한 후 `systemctl restart httpd`로 httpd 재기동해주면 정상적으로 실행되는걸 확인할 수 있다.

### 리눅스 서버 재부팅할때마다 httpd가 정상기동 안될 때

> localhost.localdomain kill[1306]: kill: cannot find process ""

{% highlight markdown %}
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since 화 2018-05-08 01:48:50 KST; 8h left
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 1306 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
  Process: 1083 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
 Main PID: 1083 (code=exited, status=1/FAILURE)

 5월 08 01:48:49 localhost.localdomain systemd[1]: Starting The Apache HTTP Server...
 5월 08 01:48:49 localhost.localdomain systemd[1]: httpd.service: main process exited, code=exited, status=1/FAILURE
 5월 08 01:48:50 localhost.localdomain kill[1306]: kill: cannot find process ""
 5월 08 01:48:50 localhost.localdomain systemd[1]: httpd.service: control process exited, code=exited status=1
 5월 08 01:48:50 localhost.localdomain systemd[1]: Failed to start The Apache HTTP Server.
 5월 08 01:48:50 localhost.localdomain systemd[1]: Unit httpd.service entered failed state.
 5월 08 01:48:50 localhost.localdomain systemd[1]: httpd.service failed.
{% endhighlight %}

리눅스 서버를 다시 키면 위와같이 httpd 서비스가 정상동작하지 않는 문제가 생겨서 매번 Selinux 보안관련 설정 + setenforce 를 다시 한 뒤 재기동하면 해결됐는데, 정확한 원인을 찾기 위해 로그를 확인해봤다.


{% highlight markdown %}
vi /etc/httpd/logs/error_log

[Mon May 07 16:57:39.471984 2018] [jk:emerg] [pid 1042] Initializing shm:/etc/httpd/logs/jk-runtime-status.1042 errno=13. Unable to start due to shared memory failure.
{% endhighlight %}

> /etc/httpd/logs/jk-runtime-status.1042 errno=13. Unable to start due to shared memory failure.

원인을 확인해보니 `httpd.conf` 파일에 `JkShmFile` 에 대한 내용을 빼먹어서 그랬다. 해당내용을 추가하고 재시작하면 Selinux의 보안설정을 매번해주지 않아도 된다. (httpd.conf 수정하는 부분에 추가해뒀다.)

### localhost,,, ServerName 관련 문제
ServerName이 어쩌구저쩌구,, 하는 에러가 발생한다면 `httpd.conf` 파일을 열어보면 ServerName과 관련된 부분이 주석처리 되어있다. 그 아래 `ServerName localhost` 라고 추가하고 재기동하면 된다.
