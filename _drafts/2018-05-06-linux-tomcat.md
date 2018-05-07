---
layout: post
author: "Paycis"
title: 리눅스 서버 배포환경 구축하기 1 - 리눅스 설치
tag: [리눅스]
---

환경적인 부분에 대해 너무 아는것이 없어서 직접 집에서 배포환경을 구축해보기로 결심했습니다. 
처음이라 모르는것도 많고 시행착오도 많았는데 그 부분들에 대해서 간략하게 정리했습니다. 

# References


## 구축환경

* 데스크탑 남는 하드디스크에 리눅스(CentOS7) 인프라 서버형태로 설치
* 데스크탑 - 공유기 - 노트북 연결 (유선으로)
* 데크스탑 CentOS로 부팅 / 노트북에서 [SSH Tool-MobaXterm](https://mobaxterm.mobatek.net/download.html) 으로 접속

## CentOS7 설치

다양한 종류의 리눅스가 존재하는데 저는 `CentOS7`를 사용했습니다. (회사서버와 같은 OS로 진행)

> CentOS Linux release 7.4.1708 (Core)

웹서버용으로 설치를 진행했고 아래 두 사이트를 참고하여 완료했습니다.

* [부팅디스크 만들기](http://blog.hangyeong.com/504)
* [웹서버용 CentOS 설치](https://pageinkr.com/web-server-사용을-위한-linux-centos-7-설치/)
* [파티션 분할](https://shovelblog.wordpress.com/2016/09/13/centos-7-2-%EC%84%A4%EC%B9%98/)

### 부팅순서 문제
데스크탑에는 기존에 사용하는 Window가 존재하는데, 처음 리눅스를 설치하고 나니 무조건 리눅스로 부팅이 됐다. 리눅스 내에서 부팅목록이 존재하는데 Window가 보이지 않아 해결방법을 이것저것 찾아보았는데...바이오스에서 부팅 순서를 바꿔주니 해결됐다. (부팅목록에 Window7을 추가하는 방법에 대한 포스팅들을 보고 따라했지만 전부 효과가 없었다...)

나중에 아파치, 톰캣 세팅하다 꼬여서 결국 OS설치부터 다시했는데 그때는 부팅순서를 바꿔뒀던게 있어서 그런지 Window7이 우선적으로 부팅되었다.

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>> 얘는 지울 것 >>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

---
layout: post
author: "Paycis"
title: 리눅스 서버 배포환경 구축하기 2 - 아파치 설치
tag: [리눅스,Apache]
---

순수한(?) 나머지 `아파치톰캣 = 톰캣`으로만 이해했었는데, 아파치와 톰캣을 연동한다는 얘기를 듣고 `??????` 무슨소리인지 이해가 안갔다. `아파치`는 웹서버이고 `톰캣`은 WAS(Web Application Server)이다. `아파치`는 정적인 데이터를 처리, 즉 상황에 따라 변하지 않는 단순한 결과를 응답하는데 사용되고 `WAS-톰캣`의 경우 사용자의 요청이나 상황에 따라 보여줘야 하는 데이터가 다른 경우를 처리한다. (ex> 입력한 전화번호에 따라 각각의 고객정보 응답) [WAS와 웹서버 차이](http://sungbine.github.io/tech/post/2015/02/15/tomcat과%20apache의%20연동.html)


그리고 아파치를 앞단에 두면 운영환경에서 여러서버에 동일한 WAS를 띄워 많은 요청을 각 WAS에 분산하는 `로드 밸런싱`이 가능하다. (아니면 호출URL을 보고 해당 WAS가 존재하는 서버로 보낼수 도 있다.)


# Apache 설치하기

여기서부터는 수워니님의 포스팅을 보고 진행했으며 막힌 부분과 어떻게 해결했는지 정리했습니다. (아무것도 몰랐는데 보면서 많은 도움을 받았습니다ㅠㅠ)

[아파치 설치하기](https://suwoni-codelab.com/linux/2017/05/27/Linux-CentOS-Apache/)

위의 포스팅에서 index.html을 만들어보는 과정은 넘겨도 큰 문제는 없습니다..

### httpd 실행상태 확인

우선 간략하게 확인하는 방법은 `ps -ef |grep httpd` 명령어를 통해 확인이 가능하고, 상세하게 보기위해선 `systemctl -l status httpd`명령어를 통해 확인이 가능하다. 

{% highlight linux %}
systemctl -l status httpd
--위와같이 입력하면 아래와 비슷하게 상태를 확인할 수 있다 (문제가 있는경우엔 간략하게 원인이 나오고, 자세한건 로그를 통해 확인 가능하다)

● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 월 2018-05-07 00:02:22 KST; 3s ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 1296 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
 Main PID: 4270 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─4270 /usr/sbin/httpd -DFOREGROUND
           ├─4271 /usr/sbin/httpd -DFOREGROUND
           ├─4272 /usr/sbin/httpd -DFOREGROUND
           ├─4273 /usr/sbin/httpd -DFOREGROUND
           ├─4274 /usr/sbin/httpd -DFOREGROUND
           └─4275 /usr/sbin/httpd -DFOREGROUND

 5월 07 00:02:21 localhost.localdomain systemd[1]: Starting The Apache HTTP Server...
 5월 07 00:02:22 localhost.localdomain systemd[1]: Started The Apache HTTP Server.

{% endhighlight %}

### 정상적인 기동이 안되는 경우 에러 확인하기 (로그)
위 명령어로 에러내용 확인이 가능하지만 세부적인 내용을 보기 위해선 로그파일을 확인해 볼 수 있다. 
{% highlight %}
cd /etc/httpd/logs
vi error_log

*ll -ltrh 를 통해 변경된 시간별로 파일을 정렬하고 맨 아래쪽에 있는 에러로그 파일을 열어본다.
{% endhighlight %}

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>> 얘는 지울 것 >>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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

{% highlight %}
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

{% highlight %}
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: Starting The Apache HTTP Server...
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: httpd.service: main process exited, code=exited, status=1/FAILURE
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal kill[18250]: kill: cannot find process ""
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: httpd.service: control process exited, code=exited status=1
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: Failed to start The Apache HTTP Server.
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: Unit httpd.service entered failed state.
Apr 14 13:35:03 ip-172-31-28-227.ap-northeast-2.compute.internal systemd[1]: httpd.service failed.
{% endhighlight %}

`systemctl -l status httpd`로 상태확인을 했더니 위와같이 나온다면 아래처럼 Selinux설정을 반영하는 명령어를 추가로 입력하면 해결된다.

{% highlight %}
chcon -R --type=httpd_sys_rw_content_t /usr/share/tomcat/webapps/ROOT

setenforce 0  //설정반영 - 추가입력

{% endhighlight %}

위와같이 입력한 후 `systemctl restart httpd`로 httpd 재기동해주면 정상적으로 실행되는걸 확인할 수 있다.

### 리눅스 서버 재부팅할때마다 httpd가 정상기동 안될 때

> localhost.localdomain kill[1306]: kill: cannot find process ""

{% highlight %}
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


{% highlight %}
vi /etc/httpd/logs/error_log

[Mon May 07 16:57:39.471984 2018] [jk:emerg] [pid 1042] Initializing shm:/etc/httpd/logs/jk-runtime-status.1042 errno=13. Unable to start due to shared memory failure.
{% endhighlight %}

> /etc/httpd/logs/jk-runtime-status.1042 errno=13. Unable to start due to shared memory failure.

원인을 확인해보니 `httpd.conf` 파일에 `JkShmFile` 에 대한 내용을 빼먹어서 그랬다. 해당내용을 추가하고 재시작하면 Selinux의 보안설정을 매번해주지 않아도 된다. (httpd.conf 수정하는 부분에 추가해뒀다.)

### localhost,,, ServerName 관련 문제
ServerName이 어쩌구저쩌구,, 하는 에러가 발생한다면 `httpd.conf` 파일을 열어보면 ServerName과 관련된 부분이 주석처리 되어있다. 그 아래 `ServerName localhost` 라고 추가하고 재기동하면 된다.

