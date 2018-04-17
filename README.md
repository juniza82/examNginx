

VirtualBox + tomcat8.5(3대) + nginx 를 이용하여 웹서버 구동
==============================================================================
~~~~
문제
VirtualBox 에 가성os(centos7)을 설치 톰캣 3개를 이용하여 nginx 와 연동하는 문제
 - abc1.site.co.kr localhost:8080
 - abc2.site.co.kr localhost:8081
 - abc3.site.co.kr localhost:8082
엔지닉스에 요청이 아래로 왔을때 해당하는 톰켓으로 가도록 구성하는 예제를 만드세요

nignx worker thread 는 20개로 셋팅하세요
access / error log 경로도 따로 만드세요
~~~~


1. virtualbox 설치(v5.2.8) (<https://www.virtualbox.org/wiki/Downloads>)
  * 참고 사이트 : VirtualBox에 CentOS 7 설치 (<https://zetawiki.com/wiki/VirtualBox에_CentOS_7_설치>)
  
1.1. centOS (<https://www.centos.org/download/>) 에서 DVD ISO 버젼을 다운로드

1.2. 호스트 PC 와 CentOS7 SSH 접속하기 위한 네트워크 설정
* virtualbox 를 설치 후 네트워크 설정에서 삽질을 많이 하였다.
    필요한 설정은 '호스트 네트워크 관리자' 인데 어디서 만들어야 할지에 대한 삽질을 많이 하였다.
    - 전역도구 - 만들기 - vboxnet0 란 이름으로 네트워크가 하나 생성 됨
    ![Alt text](https://github.com/juniza82/examNginx/blob/master/호스트PC와CentOS7SSH접속하기위한네트워크설정.png)


2. centOS 에 자바 설치 (참조: <https://zetawiki.com/wiki/CentOS_JDK_설치>)

2-1. java 설치 확인
  - java -version
 
2-2. yum 을 이용하여 java 설치 가능 확인
  - yum list java*jdk-devel

2-3. yum 을 이용하여 java 설치
  - yum install java-1.8.0-openjdk-devel.x86_64 (설치하고자 하는 java 파일을 입력하면 된다.)

3. apache tomcat-8.5.30 설치 (참조: http://apache.mirror.cdnetworks.com/tomcat/tomcat-8/v8.5.30/bin/apache-tomcat-8.5.30.tar.gz)

3-1. 다운받은 톰캣 파일 [tar -zxvf apache-tomcat-8.5.30.tar.gz] 를 이용하여 해당 폴더에 압축 풀기

3-2. 톰캣을 3개를 구동하기 위해서 특정 폴더에 복사 이동
  >ex) /tomcat-8.5.30 을 /example 밑에 3개의 톰캣을 만들려고 할때
  >  - cp /tomcat-8.5.30 /example/tomcat1
  >  - cp /tomcat-8.5.30 /example/tomcat2
  >  - cp /tomcat-8.5.30 /example/tomcat3

3-3. 톰캣의 포트 변경처리 (참조: <http://wookoa.tistory.com/102>)
  * 1씩 증가하였음[편할대로 설정]
    server port, http, ajp 의 포트들을 변경 처리

3-4. 각각의 톰캣(tomcat1, tomcat2, tomcat3) 구동하여 동시에 실행되는지 확인

3-5. 호스트 PC 에서 각각의 톰캣(tomcat1, tomcat2, tomcat3)에 접속이 가능 한지 확인
  - http://가상PC의아이피:설정한포트

* 가상서버의 방화벽 해제하기 (참조: <http://blog.netchk.net/?p=1116>)
  필자는 특정포트의 허용이 아닌 방화벽을 해제하였음.

4. nginx 설치 (참조: <https://www.lesstif.com/pages/viewpage.action?pageId=25100304>)
  - sudo firewall-cmd --permanent --zone=public --add-service=http
  - sudo firewall-cmd --permanent --zone=public --add-service=https
  - sudo firewall-cmd --reload

*최신의 nginx를 받기 위한 설정 
  vi /etc/yum.repos.d/nginx.repo

  [nginx]
  name=Nginx Repository $basearch - Archive
  baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
  enabled=1
  gpgcheck=0

  제대로 반영 되었는지 yum info 로 확인

4-1. nginx 설치
  - yum install -y nginx

4-2. nginx 부팅시 자동 구동하도록 설정
  - systemctl start nginx
  - systemctl enable nginx

4-3. 관리하고자 하는 사이트의 설정 파일 디렉토리를 만들어 준다.
  - mkdir /etc/nginx/sites-enabled/ 

4-4. 관리하고자 하는 사이트의 설정 파일들을 생성
  - abc1.site.co.kr.conf
~~~~
server {
    listen       80;
    server_name  abc1.site.co.kr;
    client_max_body_size 10M;
    charset utf-8;
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location ~ /\.ht {
        allow all;
    }
}
~~~~
  - abc2.site.co.kr.conf
  - abc3.site.co.kr.conf

4-5. nginx 의 설정을 reload 한다.
  - systemctl reload Nginx

5. 호스트 pc 의 hosts 파일에 가상서버의 IP 를 입력한다.
  - vi /etc/hosts
  - 192.168.56.1(가상서버의 IP)  abc1.site.co.kr abc2.site.co.kr abc3.site.co.kr

6. CentOS 에서는 웹서버가 다른 외부 URL을 호출이 처음에는 금지되어 있다. (참조: http://www.systemhook.net/?tag=mysql)
  - setsebool -P httpd_can_network_connect on
