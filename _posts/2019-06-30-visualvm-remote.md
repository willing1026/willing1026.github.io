---
layout: post
author: "Paycis"
title: visualVM으로 remote host 모니터링하기
tag: [자바,모니터링]
---

# visualVM
visualVM을 이용한 remote host 모니터링
- remote서버에 존재하는 자바프로세스를 모니터링 하는 방법을 찾아보고 정리했습니다.

## 성공CASE

### 테스트 환경
* 리눅스서버 : CentOS7
* JDK : 1.8

### Remote서버 JDK 설치 확인
* 서버에 JDK가 설치되어 있어야 한다. ( JAVA Process 실행을 위해서도 설치 필요 )
* 자바 버전 확인 :  `java -version`

### 방화벽 설정
* remote서버에 접속할때 사용할 port를 열어놔야한다
* 포트 추가 : `firewall-cmd --zone-public --add-port=1099/tcp`
* 추가한 설정 적용 : `firewall-cmd --reload`
* port 등록 확인 : `firewall-cmd --zone=public --list-ports`
    
### 모니터링용 테스트 파일 만들기
* test파일 생성
{% highlight markdown linenos %}
cd /home/test
vi Hello.java
{% endhighlight %}

* `hello` 라는 문구를 5초 단위로 출력하도록 설정
{% highlight java linenos %}
public class Hello {
    public static void main(String[] args) throws Exception {
        while(Boolean.TRUE) {
            System.out.println("hello");
            Thread.sleep(5000);
        }
    }
}
{% endhighlight %}
* 컴파일 : `javac Hello.java`
* Hello.class 파일 생성 확인

### 모니터링용 테스트 파일 실행
* JMX Remote 설정을 추가한 실행 스크립트 생성 : `vi run.sh`
* 실행 스크립트 입력

{% highlight markdown linenos %}
# !/bin/bash

# -Dcom.sun.management.jmxremote.local.only=false : localOnly가 true 켜져있으면 remote 연결이 안될 수 있다.
# -Dcom.sun.management.jmxremote.port=1099 : remote port설정
# -Dcom.sun.management.jmxremote.ssl=false : ssl접속설정
# -Dcom.sun.management.jmxremote.authenticate=false : 접속인증 설정. true하면 인증관련된 추가 설정 필요
# -Djava.rmi.server.hostname=192.168.0.7 : remote서버 ip, ifconfig로 확인
# -Dcom.sun.management.jmxremote.rmi.port=1099 : rmi포트 설정

java -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=192.168.0.7 -Dcom.sun.management.jmxremote.rmi.port=1099 Hello
{% endhighlight %}

* 쉘 스크립트 권한 설정 : `chmod 777 run.sh`
* 쉘 스크립트 실행 : ./run.sh

### visualVM 모니터링
* visualVM 실행 : 로컬에 설치한 자바 경로 .. /bin 디렉토리에 있는 `jvisualvm.exe` 실행
* Remote 항목 추가 : `Remote` 아이콘 더블클릭 > Host name 추가
* JMX Connection 추가 : 추가된 Remote 항목 우클릭 > `Add JMX Connection` 클릭
* `Connection : ` 뒤에 port 번호만 추가입력.  OK
* 여기까지하면 `Hello` 프로세스가 VisualVM 항목에 추가되서 확인 할 수 있다.


## 실패 CASE
* jstatd 를 이용해서 remote host를 추가하려 했는데, 실패했다.
* port - 방화벽 허용
* rmiregistry 실행
* jstatd.all.policy 파일 추가
* jstatd 실행
* 발생했던 에러
    * bind 오류 : 해당 port가 이미 사용중일 수 있다. 다른 port로 변경해서 사용
    * access denied : jstatd.all.policy 파일 문제. 오타확인 및 jstatd 실행시 경로를 절대경로로 줘서 실행하기
* 이렇게까지 했는데, 그래도 항목이 뜨지않아서 포기. 위의 방법으로 모니터링 확인.
