﻿.. admin-log:

제 12 장 로그
******************

이 장에서는 로그를 다룬다. 서비스 로그에 시작 로그에 끝난다. 로그는 금이고 법이고, 분쟁 지역의 평화 유지군이다.

로그는 글로벌 및 가상 호스트로 구분된다. 모든 로그 기록할지 여부를 설정할 수 있으며, 공통의 속성을 갖는다. ::

   <XXX Type="time" Unit="1440" Retention="10" Compression="OFF">ON</XXX>

-  ``Type (基本: time)`` , ``Unit (基本: 1440분)`` 로그 롤링 조건을 설정한다.

   - ``time`` 설정된 ``unit`` 시간 (분)마다 로그 파일을 롤링한다.
   - ``size`` 설정된 ``unit`` 크기 (단위 : MB)마다 로그 파일을 롤링한다.
   - ``both`` 쉼표 (,)로 구분하여 시간과 크기를 동시에 설정한다. 예를 들어 Unit = "1440,100"이면 시간이 24 시간 (1440 분) 또는 100MB의 경우 로그 파일을 롤링한다.

-  ``Retention (기본: 10개)`` 단위의 로그 파일을 최대 n 개의 유지이다.

-  ``Compression (기본: OFF)`` 로그 롤링되는 압축한다. 예를 들어 access_20140715_0000.log 파일이 롤링되면 access_20140715_0000.log.gz에 압축되어 저장된다.

``Type`` 이 "time" , ``Unit`` 10의 경우 로그는 10 분마다 롤링된다. 예를 들어, 서비스를 2:18 분에 시작해서 로그는 10 분의 2 : 20,2 : 30,2 : 40 롤링된다. 마찬가지로, 하루에 한 번 매일 0시 0 분에 롤링 1440 (60 분 X 24 시간)에서 ``Unit`` 값으로 설정한다.
``time`` 설정에서 로그는 하루에 한번 무조건 롤링되므로 ``Unit`` 의 최대 값은 1440을 초과 할 수 없다.

.. figure:: img/log_rolling1.jpg
   :align: center

최대 24 시간 (Unit = 1440) 시간마다 로그 롤링하도록 설정 한 경우 다음 그림과 같이 로그가 기록된다.

.. figure:: img/log_rolling2.jpg
   :align: center


.. toctree::
   :maxdepth: 2



.. admin-log-install:

Install 로그
====================================

설치 / 업데이트시에 모든 내용이 install.log에 기록된다. 이 로그는 다른 설정이 없다. ::

    #DownloadURL: http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
    #DownloadTime: 13 sec
    #Target: STON 2.0.0
    #Date: 2014.03.03 16:48:35
    Prepare for STON 2.0.0 install process
        Stopping STON...
        STON stopped
    [Copying files]
        `./fuse.conf' -> `/etc/fuse.conf'
        `./libfuse.so.2' -> `/usr/local/ston/libfuse.so.2'
        `./libtbbmalloc_proxy.so' -> `/usr/local/ston/libtbbmalloc_proxy.so'
        `./start-stop-daemon' -> `/usr/sbin/start-stop-daemon'
        `./libtbbmalloc_proxy.so.2' -> `/usr/local/ston/libtbbmalloc_proxy.so.2'
        `./libtbbmalloc.so' -> `/usr/local/ston/libtbbmalloc.so'
        `./libtbbmalloc.so.2' -> `/usr/local/ston/libtbbmalloc.so.2'
        `./libtbb.so' -> `/usr/local/ston/libtbb.so'
        `./libtbb.so.2' -> `/usr/local/ston/libtbb.so.2'
        `./stond' -> `/usr/local/ston/stond'
        `./stonx' -> `/usr/local/ston/stonx'
        `./stonr' -> `/usr/local/ston/stonr'
        `./stonu' -> `/usr/local/ston/stonu'
        `./stonapi' -> `/usr/local/ston/stonapi'
        `./server.xml.default' -> `/usr/local/ston/server.xml.default'
        `./vhosts.xml.default' -> `/usr/local/ston/vhosts.xml.default'
        `./ston_format.sh' -> `/usr/local/ston/ston_format.sh'
        `./ston_diskinfo.sh' -> `/usr/local/ston/ston_diskinfo.sh'
        `./wm.sh' -> `/usr/local/ston/wm.sh'
    [Exporting config files]
        #Export so directory
        /usr/local/ston/ to ld.so.conf
        #Export sysctl to /etc/sysctl.conf
        vm.swappiness=0
        vm.min_free_kbytes=524288
        #Export sudoers for WM
        Defaults    !requiretty
        winesoft ALL=NOPASSWD: /etc/init.d/ston stop, /etc/init.d/ston start, /bin/ps -ef
    [Configuring STON daemon script]
        STON deamon activate in run-level 2345.
    [Installing sub-packages]
        curl installed.
        libjpeg installed.
        libgomp installed.
        rrdtool installed.
    [Installing WM]
        Stopping WM...
        WM stopped
        `./wm.server_default.xml' -> `/usr/local/ston/wm/tmp/conf/server_default.xml'
        `./wm.vhost_default.xml' -> `/usr/local/ston/wm/tmp/conf/vhost_default.xml'
        WM configuration found. Current WM port : 8500
        PHP module for Legacy(CentOS 5.5) installed
        `./libphp5.so.5.5' -> `/usr/local/ston/wm/modules/libphp5.so'
        WM installation almost complete. Changing WM privileges.
    Installation successfully complete



.. _admin-log-info:

Info 로그
====================================

Info 로그는 전역 설정 (server.xml)로 설정한다. ::

   # server.xml - <Server><Cache>

   <InfoLog Type="size" Unit="1" Retention="5">ON</InfoLog>

-  ``<InfoLog> (기본: ON, Type: size, Unit: 1)`` STON 동작 및 설정 변경에 대해 기록한다.


.. _admin-log-deny:

Deny 로그
====================================

Deny 로그는 전역 설정 (server.xml)로 설정한다. ::

   # server.xml - <Server><Cache>

   <DenyLog Type="size" Unit="1" Retention="5">ON</DenyLog>

-  ``<DenyLog> (기본: ON, Type: size, Unit: 1)``

   :ref:`access-control-serviceaccess` 에 의해 액세스 차단 된 IP 주소를 기록한다. ::

      #Fields: date time c-ip deny
      2012.11.15 07:06:10 1.1.1.1 AP
      2012.11.15 07:06:26 2.2.2.2 GIN
      2012.11.15 07:06:30 3.3.3.3 3.3.3.1-255

   모든 필드는 공백으로 구분 된 각 필드의 의미는 다음과 같다.

   - ``date`` 날짜
   - ``time`` 시간
   - ``c-ip`` 클라이언트의 IP 주소
   - ``deny`` 블록 조건


.. _admin-log-originerror:

OriginError 로그
====================================

OriginError 로그는 전역 설정 (server.xml)로 설정한다. ::

   # server.xml - <Server><Cache>

   <OriginErrorLog Type="size" Unit="5" Retention="5" Warning="OFF">ON</OriginErrorLog>

-  ``<OriginErrorLog> (기본: OFF, Type: size, Unit: 5, Warning: OFF)``

   모든 가상 호스트의 소스 서버에서 발생한 오류 만 기록한다. 장애는 연결 실패 및 전송 장애를 의미하며, 원래의 서버 제거 / 회복의 결과가 기록된다. ::

      #Fields: date time vhostname level s-domain s-ip cs-method cs-uri time-taken sc-error sc-resinfo
      2012.11.15 07:06:10 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:26 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:30 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      #2012.11.15 07:06:30 [example.com] 192.168.0.13 excluded from service
      #2012.11.15 07:06:31 [example.com] Origin server list: 192.168.0.14
      #2012.11.15 07:11:11 [example.com] 192.168.0.13 recovered back in service
      #2012.11.15 07:11:12 [example.com] Origin server list: 192.168.0.13

   모든 필드는 공백으로 구분 된 각 필드의 의미는 다음과 같다.

   - ``date`` 장애가 발생한 날짜
   - ``time`` 장애 발생 시간
   - ``vhostname`` [가상 호스트]
   - ``level`` [장애 수준 (Error 또는 Warning)]
   - ``s-domain`` 소스 서버의 도메인
   - ``s-ip`` 소스 서버의 IP
   - ``cs-method`` STON가 원래 서버로 전송 HTTP Method
   - ``cs-uri`` STON가 원래 서버로 전송 URI
   - ``time-taken`` 장애가 발생하기까지 걸린 시간
   - ``sc-error`` 장애의 종류
   - ``sc-resinfo`` 장애 발생시 서버의 응답 정보 ( ","문자로 구분)

   ``Warning`` 속성이 ``ON`` 이면, 다음의 예와 같이 잘못된 HTTP 통신이 발생했을 경우에 기록한다. ::

      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 PartialResponseOnNormalRequest Res=206,Len=2635
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 ClosedWithoutResponse -

   잘못된 HTTP 통신의 경우는 다음과 같다.

   - ``ClosedWithoutResponse`` 원래 서버가 연결 종료. HTTP 응답을받지 않았다.
   - ``ClosedWhenDownloading`` 원래 서버가 연결 종료. Content-Length 만 다운로드하지 않았다.
   - ``NotPartialResponseOnRangeRequest`` Range 요청을했지만, 응답 코드가 206이 아니다.
   - ``DifferentContentLengthOnRangeRequest`` 요청한 Range 및 Content-Length가 다르다.
   - ``PartialResponseOnNormalRequest`` Range 요청이 없는데 응답 코드가 206이다.



.. admin-log-syslog:

SysLog 전송
====================================

`syslog <http://en.wikipedia.org/wiki/Syslog>`_ 프로토콜을 사용하여 로그를 UDP로 실시간 전송한다. 모든 로그를 syslog에 전송되도록 설정할 수있다. ::

   # server.xml - <Server><Cache>

   <InfoLog SysLog="OFF">ON</InfoLog>
   <DenyLog SysLog="OFF">ON</DenyLog>
   <OriginErrorLog SysLog="OFF">ON</OriginErrorLog>

-  ``SysLog``

   - ``OFF (기본)`` syslog를 사용하지 않는다.

   - ``ON`` 이 태그의 하위에 설정된 ``<SysLog>`` 에 로그를 전송합니다.

다음은  ``<OriginErrorLog>`` 가 기록 될 때 syslog를 설정하는 예이다. ::

   # server.xml - <Server><Cache>

   <OriginErrorLog SysLog="ON">
      <SysLog Priority="local3.info" Dest="192.168.0.1:514" />
      <SysLog Priority="user.alert" Dest="192.168.0.2" />
      <SysLog Priority="mail.debug" Dest="log.example.com" />
   </OriginErrorLog>

1. ``<OriginErrorLog>`` 의 ``SysLog`` 속성을 ``ON`` 으로 설정한다.
#. ``<OriginErrorLog>`` 하위에 ``<SysLog>`` 태그를 생성한다. n 개의 서버에 동시에 전송 가능하다.
#. ``<SysLog>`` 의 ``Priority`` 속성을 설정한다. 이 표현은 syslog의 `Facility Levels <http://en.wikipedia.org/wiki/Syslog#Facility_levels>`_ 와
   `Severity levels <http://en.wikipedia.org/wiki/Syslog#Severity_levels>`_ 의 조합으로 구성한다.
#. ``<SysLog>`` 의 ``Dest`` 속성을 설정한다. syslog 수신 서버를 의미하고, 수신 포트가 514 인 경우 생략 가능하다.

위의 설정에서 기록 된 sys 로그의 예는 다음과 같다. syslog의 tag는 STON / {로그 이름}에 기록된다. ::

    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:20 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd 1996 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:22 [ERROR] [example.com] - 192.168.0.14 GET /favicon.ico 1995 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:24 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd22 2020 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] 192.168.0.14:102 excluded from service
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] Origin server list:



가상 호스트 별 로그 저장
====================================

가상 호스트별로 로그는 별도로 기록된다. 로그가 ``OFF`` 로 설정되어 있어도, 로컬 파일에만 작성되지 않은 뿐이므로
:ref:`api-monitoring-logtrace` 는 정상적으로 작동한다. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Log Dir="/cache_log">
      ... (생략) ...
   </Log>

-  ``<Log>`` ``Dir`` 속성에 로그가 기록되는 디렉토리를 설정합니다. 로그는 설정 한 디렉토리 하위의 가상 호스트의 디렉토리에 생성된다.



.. _admin-log-dns:

DNS 로그
====================================

원래 서버의 주소가 Domain에 설정된 경우 Resolving 결과를 기록한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Dns Type="size" Unit="10" Retention="10" SysLog="OFF" Compression="OFF">ON</Dns>

::

   #Fields: date time domain ttl ip-list ip-count time-taken result
   2014-07-30 12:10:33 example.com 157 173.194.127.15,173.194.127.23,173.194.127.24,173.194.127.31 4 5007 success
   2014-07-30 12:10:38 example.com 152 173.194.127.23,173.194.127.24,173.194.127.31,173.194.127.15 4 9 success
   2014-07-30 12:11:03 example.com 127 173.194.127.31,173.194.127.15,173.194.127.23,173.194.127.24 4 15007 success
   2014-07-30 12:12:53 example.com 17 173.194.127.15,173.194.127.23,173.194.127.24,173.194.127.31 4 6 success
   2014-07-30 12:23:16 test.com 0 - 0 10008 fail
   2014-07-30 12:23:21 test.com 0 - 0 5007 fail
   2014-07-30 12:23:26 test.com 0 - 0 5011 fail
   2014-07-30 12:24:38 example.com 152 173.194.127.23,173.194.127.24,173.194.127.31,173.194.127.15 4 9 success
   2014-07-30 12:25:03 example.com 127 173.194.127.31,173.194.127.15,173.194.127.23,173.194.127.24 4 15007 success

모든 필드는 공백으로 구분 된 각 필드의 의미는 다음과 같다.

-  ``date`` 날짜
-  ``time`` 시간
-  ``domain`` 대상 Domain
-  ``ttl`` 레코드의 유효 시간 (Time To Live)
-  ``ip-list`` IP 목록
-  ``ip-count`` IP 수
-  ``time-taken`` 실행 시간
-  ``result`` success 또는 fail


.. _admin-log-access:

Access 로그
====================================

모든 클라이언트의 HTTP 트랜잭션을 기록한다. 로깅 시점에서는 HTTP 트랜잭션이 완료되면 전송 완료 또는 전송 중단 시점을 의미한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Access Type="time" Unit="1440" Retention="10" XFF="on" Form="ston" Local="Off">ON</Access>

-  ``XFF``

   - ``ON (기본)`` 클라이언트가 보낸 XFF (X-Forwarded-For) 헤더의 값과 클라이언트의 IP 주소를 같이 기록한다. 없으면 ``OFF`` 과 같다.
   - ``OFF`` 클라이언트의 IP 주소를 기록한다.
   - ``TrimCIP``  XFF 헤더가없는 경우는 클라이언트의 IP 주소를 한 경우 (클라이언트의 IP 주소를 제외) XFF 헤더만을 기록한다.

-  ``Form``

   - ``ston (기본)`` W3C 표준 + 확장 필드
   - ``apache`` Apache 형식
   - ``iis`` IIS 형식
   - ``custom`` `admin-log-access-custom`

-  ``Local``

   - ``OFF (기본)`` 로컬 통신 (Loopback)은 기록하지 않는다.
   - ``ON`` 로컬 통신 (Loopback)도 기록한다.

::

    #Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) sc-status sc-bytes time-taken cs-referer sc-resinfo cs-range sc-cachehit cs-acceptencoding session-id sc-content-length
    2012.06.27 16:52:24 220.134.10.5 GET /web/h.gif - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 98141 5 - Bypass+gzip+SSL3 - TCP_HIT gzip+deflate 7 1273735
    2012.06.27 16:52:26 220.134.10.5 GET /favicon.ico - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 949 2 - - - TCP_HIT gzip+deflate 35 14875
    2012.06.27 17:00:06 220.168.0.13 GET /setup.Eexe - 80 - 61.168.0.102  Mozilla/5.0+(Windows+NT+6.1;+WOW64)+AppleWebKit/536.11+(KHTML,+like+Gecko)+Chrome/20.0.1132.57+Safari/536.11 206 20971800 7008 - - 398458880-419430399 TCP_HIT - 41 89764358

모든 필드는 공백으로 구분 된 각 필드의 의미는 다음과 같다.

-  ``date`` HTTP 트랜잭션이 완료된 날짜
-  ``time`` HTTP 트랜잭션이 완료된 시간
-  ``s-ip`` 서버의 IP
-  ``cs-method`` 클라이언트가 전송 한 HTTP Method
-  ``cs-uri-stem`` 클라이언트가 전송 한 URL에서 QueryString을 제외한 부분
-  ``cs-uri-query`` 클라이언트가 전송 한 URL에서 QueryString
-  ``s-port`` 서버의 포트
-  ``cs-username`` 클라이언트 username
-  ``c-ip`` 클라이언트의 IP 주소입니다. XFF 설정이 "ON"이면 X-Forwarded-For 헤더 값과 클라이언트의 IP 주소를 기록한다.
-  ``cs(User-Agent)`` 클라이언트가 보낸 HTTP User-Agent
-  ``sc-status`` 서버의 응답 코드
-  ``sc-bytes`` 서버에서 보낸 Bytes (헤더 + 내용)
-  ``time-taken`` HTTP 트랜잭션이 완료 될 때까지 걸린 총 시간 (밀리 초)
-  ``cs-referer`` 클라이언트가 전송 한 HTTP Referer
-  ``sc-resinfo`` 부가 정보. "+"문자로 구분된다. 압축 된 내용을 서비스하는 경우 압축 옵션 (gzip 또는 deflate)이 명시된다. 안전한 통신이면 SSL 프로토콜 버전 (SSL3, TLS1, TLS1.1, TLS1.2)이 명시된다. 우회 통신이면 "Bypass"가 명시되어있다.

-  ``cs-range`` 클라이언트가 보낸 Range 헤더를 기록한다.
-  ``sc-cachehit`` 캐시 HIT 결과
-  ``cs-acceptencoding`` 클라이언트가 보낸 Accept-Encoding 헤더
-  ``session-id`` HTTP 클라이언트 세션 ID (unsigned int64)
-  ``sc-content-length`` 서버의 응답 Content-Length 헤더 값

Access 로그는 전송 성공 / 실패 여부에 관계없이 모든 HTTP 트랜잭션을 기록한다. HTTP 트랜잭션은 클라이언트가 HTTP 요청을 보낼 때 시작된다. STON가 클라이언트에 응답을 전송하기 전에 HTTP 연결이 종료 된 경우 HTTP 트랜잭션을 종료 된 것으로 본다. 로그에는 ``sc-status`` 와 ``sc-bytes`` 가 0에 기록된다. 주로 STON가 원래 서버에서 응답을 수신하기 전에 클라이언트가 연결을 종료하는 경우 이러한 로그가 기록된다.



.. _admin-log-access-custom:

사용자 정의 Access 로그 포맷
====================================

Access 로그 형식을 사용자 정의 로그로 설정한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Access Form="custom">ON</Access>
   <AccessFormat>%a %A %b id=%{userid}C %f %h %H "%{user-agent}i" %m %P "%r" %s %t %T %X %I %O %R %e %S %K</AccessFormat>

-  ``<Access>`` 의 ``Form`` 속성을 ``custom`` 으로 설정한다.

-  ``<AccessFormat>`` 사용자 정의 로그 형식.

위 예제의 경우는 다음과 같이 Access 로그가 기록된다. (#Fields은 기록하지 않는다.) ::

    192.168.0.88 192.168.0.12 163276 id=winesoft; image.jpg example.com HTTP "STON" GET 80 "GET /ston/image.jpg?type=png HTTP/1.1" 200 2014-04-03 21:21:54 1 C 204 163276 1 2571978 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 63276 id=winesoft; vod.mp4 example.com HTTP "STON" POST 80 "GET /ston/vod.mp4?start=10 HTTP/1.1" 200 2014-04-03 21:21:54 12 C 304 363276 2 2571979 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 3634276 id=ston; news.html example.com HTTPS "STON" GET 443 "GET /news.html HTTP/1.1" 200 2014-04-03 21:21:54 30 X 156 2632576 1 2571980 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 6332476 id=winesoft; style.css example.com HTTP "STON" HEAD 80 "GET /style.css HTTP/1.1" 200 2014-04-03 21:21:54 10 X 234 653276 2 2571981 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 6276 id=ston; ui.js example.com HTTP "STON" GET 80 "GET /ui.js HTTP/1.1" 200 2014-04-03 21:21:54 1 X 233 63276 1 2571982 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 626 id=winesoft; hls.m4u8 example.com HTTP "STON" GET 80 "GET /hls.m4u8 HTTP/1.1" 200 2014-04-03 21:21:54 2 X 124 6312333276 2 2571983 TCP_REFRESH_HIT HTTP/1.1

`Apache 로그 형식 <https://httpd.apache.org/docs/2.2/ko/mod/mod_log_config.html>`_ 을 기반으로 개발 된 일부 확장 필드가 있습니다. 각 필드의 구분자는 제한이 없습니다 Space를 사용하는 경우 User-Agent처럼 Space가 포함되어있는 필드는 큰 따옴표 ("...")로 묶어 설정합니다.

-  ``%...a`` 클라이언트의 IP ::

      192.168.0.66

-  ``%...A`` 서버의 IP 주소 ::

      192.168.0.14

-  ``%...b`` HTTP 헤더를 제외한 전송 바이트 수 ::

      1024

-  ``%...{foobar}C`` 서버가 수신 한 요청 Foobar 쿠키의 내용  ::

      %{id=}c 에 입력하면 Cookie의 id =에 해당하는 값을 기록 

-  ``%...D`` 요청을 처리하는데 걸린 시간 (MS) ::

      3000

-  ``%...f`` 파일명 ::

      /mp4/iu.mp4 라면 iu.mp4를 기록 

-  ``%...h`` HostName ::

      example.com

-  ``%...H`` 요청 프로토콜 ::

      http 또는 https

-  ``%...{foobar}i`` 서버가 수신 한 요청에서 foobar : 헤더의 내용 ::

      %{User-Agent}i 로 입력하는 경우 User - Agent 값을 기록

-  ``%...m`` 요청 Method ::

      GET 또는 POST 또는 HEAD

-  ``%...P`` Server PORT ::

      80

-  ``%...q`` QueryString ::

      Id=10&value=20

-  ``%...r`` 요청의 첫번째 줄 (Request Line) ::

      GET /img.jpg HTTP/1.1

-  ``%...s`` 의 응답 코드 ::

      200

-  ``%...t`` STON 기본 시간 형식	::

      2014-01-01 15:27:02

-  ``%...{format}t`` Format에 정의되어있는 날짜 형식 ::

      %{%Y-%m-%d %H:%M:%S}T를 입력하면 2014-08-07 06:12:23에 기록했다.

-  ``%...T`` TimeTaken (초) ::

      10

-  ``%...U`` ShortURI ::

      /img/img.jpg

-  ``%...u`` FullURI ::

      /img/img.jpg?session=1232&id=37

-  ``%...X`` 트랜잭션이 완료했을 때의 상태

   - ``X`` 응답이 완료되기 전에 종료
   - ``C`` 응답이 완료된

   ::

      C

-  ``%...I`` 요청 헤더를 포함한 수신 바이트 ::

      2048

-  ``%...O`` 응답 헤더를 포함한 송신 바이트 ::

      2048

-  ``%...R`` 응답 시간 (MS) ::

      2

-  ``%...e`` Session-ID ::

      1

-  ``%...S`` 캐시 HIT 결과 ::

      TCP_HIT

-  ``%...K`` 요청 HTTP 버전	::

      HTTP/1.1

-  ``%...y`` 요청의 HTTP 헤더 크기	::

      488

-  ``%...z`` 응답의 HTTP 헤더 크기	::

      362

설정 한 필드의 값이 존재하지 않는 경우 -로 표기한다. 형식이 잘못하고 있으면 STON 기본 형식 (Form = "ston")로 동작한다.

위의 표에서는 각 필드의 ...는 (eg "% h % U % r % b) 아무것도 명시하지 않거나 기록 조건을 명시 할 수있다 (조건을 충족하지 않는 경우 - 기록) . 조건은 HTTP 상태 코드 목록으로 설정하거나!에서 NOT 조건을 설정할 수있다.

다음의 예에서는 400 (Bad Request) 오류 또는 501 (Not Implemented) 오류의 경우에만 User-agent를 기록한다. ::

    "%400,501{User-agent}i"

다음 예제에서는 정상적인 상태가 아닌 모든 요청에 대해 Referer를 로그에 남긴다. ::

    "%!200,304,302{Referer}i"



.. _admin-log-origin:

Origin 로그
====================================

원본 서버의 모든 HTTP 트랜잭션을 기록한다. 기록 시점에서는 HTTP 트랜잭션이 완료되면 전송 완료 또는 전송 중단 시점을 의미한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Origin Type="time" Unit="1440" Retention="10" Local="Off">ON</Origin>

::

    #Fields: date time cs-sid cs-tcount c-ip cs-method s-domain cs-uri s-ip sc-status cs-range sc-sock-error sc-http-error sc-content-length cs-requestsize sc-responsesize sc-bytes time-taken time-dns time-connect time-firstbyte time-complete cs-reqinfo cs-acceptencoding sc-cachecontrol s-port sc-contentencoding session-id session-type
    2012.06.27 17:40:00 357 899 192.168.0.13 GET i.example.com /t/2.gif 115.71.9.136 200 - - - 3874 197 271 3874 20 0 0 17 3 - gzip+deflate - 80 gzip 7 cache
    2012.06.27 17:40:00 357 900 192.168.0.13 GET i.example.com /ex1.gif 115.71.9.136 200 - - - 5673 223 272 5673 24 0 0 21 3 - - - 80 - 8 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9  4 Bypass - - 80 - 7 cache
    #[ERROR:01] 2012.06.27 17:40:01 220.73.216.5 220.73.216.5 GET /web/nmb/img/main/v1/h1.gif 1824 Connect-Timeout - 11 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB1.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9 4 - max-age=3600 80 - 12 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB2.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9 4 - no-cache 80 - 35 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB3.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9 4 - - 80 - 35 cache

소스 서버에 장애가 발생한 경우 # [ERROR : xx] 시작 오류 로그가 기록된다. 모든 필드는 공백으로 구분 된 각 필드의 의미는 다음과 같다.

.. figure:: img/time_taken.jpg
   :align: center

   원래 시간 측정 구간

-  ``date`` HTTP 트랜잭션이 완료된 날짜
-  ``time`` HTTP 트랜잭션이 완료된 시간
-  ``cs-sid`` 세션의 고유 ID입니다. 같은 세션을 통해 처리 된 (다시 한) HTTP 트랜잭션은 같은 값을 가진다.
-  ``cs-tcount`` 트랜잭션 수입니다. 이 HTTP 트랜잭션이 현재 세션에서 몇 번째에 처리 된 트랜잭션임을 기록한다. 같은 ``cs-sid`` 값을 갖는 트랜잭션이면,이 값은 중복 될 수 없다.
-  ``c-ip`` STON의 IP
-  ``cs-method`` 원래 서버로 전송 HTTP Method
-  ``s-domain`` 소스 서버의 도메인
-  ``cs-uri`` 원래 서버로 전송 URI
-  ``s-ip`` 소스 서버의 IP
-  ``sc-status`` 소스 서버 HTTP 응답 코드
-  ``cs-range`` 소스 서버로 전송되는 Range 요구 값
-  ``sc-sock-error`` 소켓 오류 코드 (1 = 전송 실패, 2 = 전송 지연, 3 = 연결 종료)
-  ``sc-http-error`` 소스 서버가 4xx 또는 5xx 응답을 준 때 응답 코드를 기록
-  ``sc-content-length`` 소스 서버가 보낸 Content Length
-  ``cs-requestsize (단위: Bytes)`` 소스 서버로 전송되는 HTTP 요청 헤더 크기
-  ``sc-responsesize (단위: Bytes)`` 소스 서버가 응답 HTTP 헤더의 크기
-  ``sc-bytes (단위: Bytes)`` 받은 콘텐츠 크기 (헤더 제외)
-  ``time-taken (단위: ms)`` HTTP 트랜잭션이 완료 될 때까지 걸린 총 시간. 세션의 재사용이없는 경우는 소켓 연결 시간까지 포함한다.
-  ``time-dns (단위: ms)`` DNS 쿼리에 걸린 시간
-  ``time-connect (단위: ms)`` 원래의 서버와 소켓 Established까지 걸린 시간
-  ``time-firstbyte (단위: ms)`` 요청을 보내고 응답이 올 때까지 걸린 시간
-  ``time-complete (단위: ms)`` 첫 번째 응답에서 완료까지 걸린 시간
-  ``cs-reqinfo`` 부가 정보. "+"문자로 구분되어있다. 우회 통신이면 "Bypass"Private 우회라면 "PrivateBypass"에 기록된다.
-  ``cs-acceptencoding`` 소스 서버에 압축 된 콘텐츠를 요청하면 "gzip으로 deflate"에 기록된다.
-  ``sc-cachecontrol`` 원본 서버가 보낸 cachecontrol 헤더
-  ``s-port`` 소스 서버의 포트
-  ``sc-contentencoding`` 원본 서버가 보낸 Content-Encoding 헤더
-  ``session-id`` 소스 서버 요청을 발생시킨 HTTP 클라이언트 세션 ID (unsigned int64)
-  ``session-type`` 소스 서버에 요청 된 세션의 유형

   -  ``cache`` 캐시 용도로 사용 된 세션
   -  ``recovery`` :ref:`origin_exclusion_and_recovery` 에서 복구 목적으로 사용 된 세션
   -  ``healthcheck`` :ref:`origin-health-checker` 가 사용 세션


.. admin-log-monitoring:

Monitoring 로그
====================================

5 분 평균 통계를 기록한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Monitoring Type="size" Unit="10" Retention="10" Form="json">ON</Monitoring>

-  ``Form`` 로그 형식을 지정한다. ( ``json`` 또는 ``xml`` )



.. admin-log-filesystem:

FileSystem 로그
====================================

:ref:`filesystem` 을 사용하여 발생하는 모든 File I / O 트랜잭션을 기록한다. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <FileSystem Type="time" Unit="1440" Retention="10">ON</FileSystem>

File I / O 트랜잭션의 종료시에 기록된다. 트랜잭션의 종료 시점에서 cs-method의 형태에 의존한다. ::

    #Fields: date time cs-method cs-path sc-status sc-bytes response-time time-taken sc-cachehit attr session-id
    2012.06.27 16:52:24 ATTR /t 200 0 100 100 TCP_HIT FOLDER 1
    2012.06.27 16:52:24 ATTR /t/2.gif 200 0 100 100 TCP_HIT FILE 1
    2012.06.27 16:52:24 OPEN /file.txt 200 0 100 2000 TCP_HIT FILE 2
    2012.06.27 16:52:24 READ /file.txt 200 1024768 100 2000 TCP_HIT FILE 2

-  ``date`` File I / O 트랜잭션이 완료된 날짜
-  ``time`` File I / O 트랜잭션이 완료된 시간
-  ``cs-method`` File I / O 액세스를 형성한다. 다음 세 가지 중 하나를 갖는다.

   -  ``ATTR`` getattr 함수를 호출합니다. 함수가 반환 된 때의 기록
   -  ``OPEN`` 파일은 열렸지만 READ하지 않습니다. 파일을 닫을 때 로그 기록
   -  ``READ`` 일을 열고 READ했다. 파일을 닫을 때 로그 기록

-  ``cs-path`` 접근 경로
-  ``sc-status`` 응답 코드. 일반 서비스 (200)를 제외한 처리에 실패 코드는 다음과 같다.

   -  ``200`` 일반 서비스
   -  ``301`` 우회가 필요
   -  ``302`` 서비스 거부
   -  ``303`` Redirect 필요
   -  ``400`` 잘못된 요청
   -  ``401`` 가상 호스트를 찾지 못한
   -  ``402`` 위안에서 초기화에 실패
   -  ``500`` 개체의 초기화에 실패
   -  ``501`` 개체 Open 실패
   -  ``502`` 저장 경로 생성에 실패
   -  ``503`` 메모리 초기화 실패
   -  ``504`` Emergency 상태
   -  ``600`` 파일 서비스 대기 Timeout
   -  ``601`` 파일의 데이터 서비스 대기 Timeout
   -  ``602`` 파일 서비스 대기중인 파일의 초기화에 실패
   -  ``603`` 파일의 데이터 서비스 대기중인 데이터의 초기화에 실패
   -  ``701`` 잘못된 Offset
   -  ``702`` 파일의 특정 영역을로드 할 수 없습니다
   -  ``703`` Not enough memory
   -  ``704`` 위안 세션의 작성에 실패

-  ``sc-bytes`` Read 된 크기
-  ``response-time`` 함수 호출 ~ 서비스 개체를 연결하는 데 필요한 된 시간
-  ``time-taken`` 함수 호출 ~ File I / O Transaction이 완료되어지기 시작한 시간
-  ``sc-cachehit`` 캐시 HIT 결과
-  ``attr`` FILE 또는 FOLDER
-  ``session-id`` File I/O 세션 ID (unsigned int64)

   .. note::

      ``session-id`` 는 Client (HTTP 또는 File I / O) Context가 생성 될 때 할당된다. 일반적인 파일 처리 과정이다 Open -> Read -> Close에서는 Open 시점에서 Client Context가 생성되고 Close 시점에서 파괴된다. 한편, getattr 함수는 원자 (Atomic) 함수이므로 매번 Client Context가 생성 / 파괴되어 항상 새로운 session-id를 할당된다.



.. _admin-log-ftp:

FTP 전송
====================================

로그 롤링되는 지정된 FTP 클라이언트를 사용하여 로그를 업로드한다.


.. _admin-log-ftpclient:

FTP 클라이언트
---------------------

FTP 클라이언트를 설정한다. 롤링 된 로그를 실시간으로 FTP 서버에 업로드한다.

.. figure:: img/conf_ftpclient.png
   :align: center

   FTP 클라이언트의 구조와 동작

FTP 클라이언트는 위 그림과 같이 STON 외부에 존재한다. STON 로컬에 존재하는 로그를 FTP 클라이언트 큐에 입력하면 FTP 작동에 관여하지 않는다. FTP 클라이언트는 자신의 설정에 따라 업로드한다.

FTP 클라이언트는 전역 설정 (server.xml)로 설정한다. ::

   # server.xml - <Server>

   <Ftp Name="backup1">
      <Mode>Passive</Mode>
      <Address>ftp.winesoft.co.kr:21</Address>
      <Account>
         <ID>test</ID>
         <Password>12345abc</Password>
      </Account>
      <ConnectTimeout>10</ConnectTimeout>
      <TransferTimeout>600</TransferTimeout>
      <TrafficCap>0</TrafficCap>
      <DeleteUploaded>OFF</DeleteUploaded>
      <BackupOnFail>OFF</BackupOnFail>
      <UploadPath>/log_backup/%v/%s-%e.%p.log</UploadPath>
      <Transfer Time="Rotate" />
   </Ftp>

   <Ftp Name="backup2">
      <Mode>Active</Mode>
      <Address>192.168.0.14:21</Address>
      <Account>
         <ID>test</ID>
         <Password>qwerty</Password>
      </Account>
      <ConnectTimeout>3</ConnectTimeout>
      <TransferTimeout>100</TransferTimeout>
      <TrafficCap>10240</TrafficCap>
      <DeleteUploaded>ON</DeleteUploaded>
      <BackupOnFail>ON</BackupOnFail>
      <Transfer Time="Static">04:00</Transfer>
   </Ftp>

-  ``<Ftp>`` FTP 클라이언트를 설정한다. ``Name`` 속성에 고유 한 이름을 설정한다.

   - ``Mode (기본: Passive)`` 연결 모드 ( ``Passive`` 또는 ``Active`` )
   - ``Address`` FTP 주소.
   - ``Account`` FTP 계정. 만약 비밀번호 (예 : qwerty)를 암호화하려면 다음 API를 사용한다. ::

        /command/encryptpassword?plain=qwerty

     암호화 된 암호는 다음과 같이 설정한다. ::

        <Password Type="enc">dXR9k0xNUZVVYQsK5Bi1cg==</Password>

   - ``ConnectTimeout`` 연결 대기 시간
   - ``TransferTimeout`` 전송 대기 시간
   - ``TrafficCap (단위: KB)`` 전송 대기 시간
   - ``DeleteUploaded (기본: OFF)`` 전송 완료시 로그를 삭제한다.
   - ``BackupOnFail (기본: OFF)`` 전송 실패시 로그가 삭제되지 않도록 로그를 다음 경로에 백업한다. ::

        /usr/local/ston/stonb/backup/

     백업 된 로그는 보내지 않고 관리자가 삭제 될 때까지 삭제되지 않는다.

   - ``UploadPath`` 업로드 경로를 설정한다. 별도로 설정하지 않으면 "/ 가상 호스트 /"에 업로드한다. example.com 로그 /example.com/ 디렉토리에 업로드된다.

     - ``%{time format}s`` 로그 시작 시간
     - ``%{time format}e`` 로그 종료 시간
     - ``%p`` prefix
     - ``%v`` 가상 호스트 이름
     - ``%h`` 기기 HOST 이름

     예를 들어, 다음과 같이 설정 한 경우 ::

        # server.xml - <Server><Ftp>

        <UploadPath>/log_backup/%v/%s-%e.%p.log</UploadPath>

     업로드 경로는 다음과 같다. ::

        /log_backup/example.com/200140722_0000-200140722_2300.access.log

   - ``Transfer`` 로그 전송 시간을 지정한다. ``Type`` 속성에 따라 값의 형식이 달라진다.

     - ``Rotate (기본)`` 롤링되면 바로 전송한다. 값을 가지지 않는다.
     - ``Static`` 하루에 한 번 지정된 시간에 전송한다. 예를 들어 04:00으로 설정한다면 오전 4시 전송을 시작한다.
     - ``Interval`` 일정 시간 간격으로 전송한다. 예를 들어, 4로 설정하면 4 시간 간격으로 로그를 전송합니다.

     전송 시간을 설정하면 그 시점에서 로그 롤링되지 않도록 적절하게 로그 관리 정책을 설정할 필요가있다.


FTP 클라이언트는 curl을 사용한다.


.. admin-log-ftplog:

FTP 로그
---------------------

FTP 로그는 /usr/local/ston/sys/stonb/stonb.log에 통합하여 저장된다. ::

    #Fields: date time local-path cs-url file-size time-taken sc-status sc-error-msg
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10006 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/access_20140423_1700.log ftp://192.168.0.14:21/winesoft.co.kr/access_20140423_1700.log 260 60 success "-"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10008 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/filesystem_20140423_080000.log ftp://192.168.0.14:21/winesoft.co.kr/filesystem_20140423_080000.log 179 60 success "-"

모든 필드는 공백으로 구분 된 각 필드의 의미는 다음과 같다.

-  ``date`` 일
-  ``time`` 시간
-  ``local-path`` 전송 로그의 로컬 경로
-  ``cs-url`` 전송 FTP 주소
-  ``file-size`` 전송 파일 크기
-  ``time-taken (단위: ms)`` 전송 시간
-  ``sc-status`` 전송 성공 / 실패 (success 또는 fail)
-  ``sc-error-msg`` 전송 실패시 curl 오류 메시지



.. admin-log-ftptransfer:

로그 FTP 전송
---------------------

로그 롤링되는 지정된 FTP 클라이언트를 통해 업로드한다. 쉼표 (,)로 구분하여 여러 FTP 클라이언트를 동시에 사용할 수있다. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Log>
      <Access Ftp="backup1, backup2">ON</Access>
      <Origin Ftp="backup_org">ON</Origin>
      <Monitoring Ftp="backup1">ON</Monitoring>
      <FileSystem Ftp="backup2">ON</FileSystem>
   </Log>

-  ``Ftp`` 사용 FTP 클라이언트

ftp://{FTP 서버 주소} / {가상 호스트 이름} / {롤링 된 로그 이름}에서 로그를 업로드한다. 예를 들어 ftp.dummy.com 서버에 가상 호스트 example.com의 롤링 된 로그 (access_20140424_0000.log)을 업로드하는 주소는 ftp://ftp.dummy.com/example.com/access_20140424_0000.log된다.
