﻿.. _getting-started:

제 2 장 시작하기
******************

.. note::

   - `[동영상 강좌]하자! STON Edge Server - Chapter 1. 설치 및 실행 <https://youtu.be/sMfp728pMtM?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

이 장에서는 시스템 구성에서 설치 및 샘플의 가상 호스트까지 구성 해 본다. 텍스트 편집기 만 있으면 누구나 할 수있다.

STON는 표준 Linux 서버에서 작동하도록 개발되었습니다. 개발 단계에서 HW뿐만 아니라 OS 파일 시스템 등의 종속성을 가질 수있는 요소는 최대한 배제했다. 고객이 합리적인 기기를 선택할 수 있도록 돕는 것은 매우 중요하다. 왜냐하면 서비스의 특성과 규모에 따라 적절한 서버를 구성 할 서비스의 시작이기 때문이다.


.. toctree::
   :maxdepth: 2


.. _getting-started-serverconf:

서버 구성
====================================

일반적으로 서버를 구성 할 때 CPU, 메모리, 디스크를 주로 고려하고있다. 10Gbps 급의 높은 성능을 필요로하는 서비스라면, 각 구성 요소가 서비스의 특성을 만족해야한다면 원하는 성능을 얻을 수있다.

-  **CPU**

   Quad 코어 이상을 권장한다. STON는 Many-Core의 확장 성 (Scalability)을 가진다. 코어가 많을수록 초당 처리량은 증가한다. 그러나 높은 처리량이 반드시 높은 트래픽을 의미하는 것은 아니다.

   .. figure:: img/10g_cpu.jpg
      :align: center

      클라이언트가 많을수록 많은 CPU는 힘이된다.

   4KB 파일을 약 26 만회를 보낼 수와 1GB의 파일을 한번에 보낼 수는 동일한 대역폭을 사용한다. CPU 선택의 가장 큰 기준은 얼마나 많은 동시 연결을 처리하는 방법이다.


-  **메모리**

   Memory-Indexing 방식을 사용하기 때문에 4GB 이상 권장한다. ( :ref:`adv_topics_mem` 를 참조)
   자주 액세스되는 콘텐츠는 항상 메모리에 상주가 그렇지 않은 콘텐츠는 디스크에서로드한다. 따라서 콘텐츠가 많은 집중도가 낮은 경우 (Long-tail) 디스크 부하의 증가로 성능이 저하 될 수있다. 서비스되는 콘텐츠의 수에 관계없이 콘텐츠 크기가 크고, 디스크 I / O 부하가 높은 경우 메모리를 추가하여 부하를 낮출 수있다.


-  **디스크**

   OS를 포함하여 적어도 3 개 이상을 권장하고있다. 디스크가 많을수록 많은 콘텐츠를 캐시 할 수있을뿐만 아니라 I / O 부하가 분산된다.

   .. figure:: img/02_disk.png
      :align: center

      항상 OS와 STON 콘텐츠와 다른 디스크로 구성한다.

   일반적으로 OS가 설치된 디스크에 STON를 설치한다. 로그도 동일한 디스크로 설정하는 것이 일반적이다. 로그는 서비스의 상황을 실시간으로 기록하기 위해 항상 Write 부하가 발생한다.

STON 디스크를 RAID 0과 같이 사용한다. 성능 및 RAID 관계 여부는 고객 서비스의 특성에 따라 달라진다. 그러나 파일의 변경이 빈번하지 않고 콘텐츠의 크기가 실제 메모리 크기보다 훨씬 큰 경우 RAID를 사용하여 Read 속도의 향상이 효과적 일 수있다.。


.. _getting-started-os:

OS 구성
====================================

가장 기본적인 형태로 설치한다. 표준 64bit Linux 배포판 (Cent 6.2 이상, Ubuntu 10.04 이상)이면 정상적으로 동작한다. 패키지 의존성을 가지지 않는다.


.. _getting-started-install:

설치
====================================

1. 최신 버전의 STON을 다운로드합니다. ::

      [root@localhost ~]# wget  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      --2014-06-17 13:29:14--  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      Resolving foobar.com... 192.168.0.14
      Connecting to foobar.com|192.168.0.14|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 71340645 (68M) [application/x-gzip]
      Saving to: “ston.2.0.0.rhel.2.6.32.x64.tar.gz”

      100%[===============================================>] 71,340,645  42.9M/s   in 1.6s

      2014-06-17 13:29:15 (42.9 MB/s) - “ston.2.0.0.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. 압축을 해제한다. ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. 설치 스크립트를 실행한다. ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. 설치 과정은 install.log에 기록된다. 로그를 사용하여 설치 중에 발생하는 문제를 알 수있다. ::

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


.. _getting-started-license:

라이센스 발행
====================================

신규 고객의 경우 다음 단계를 사용하여 라이센스를 발급한다.

* `신청서 <http://ston.winesoft.co.kr/EULR.doc>`_ 작성
* license@winesoft.co.kr 로 전송
* 확인 절차 후 발급

라이센스 파일 (license.xml)가 반드시 설치 경로에 존재해야 STON가 성공적으로 구동된다.


.. _getting-started-update:

업데이트
====================================
최신 버전이 배포되면 stonu 명령으로 업데이트 할 수있다. ::

	./stonu 2.0.1

또는 :ref:`wm` 의 :ref:`wm-update` 를 통해 쉽게 업데이트 할 수있다.

   .. figure:: img/conf_update1.png
      :align: center


.. _getting-started-update-manual:

외부 연결이되지 않는 경우
------------------------------------------------

STON가 설치된 서버에서 외부 연결이되어 있지 않으면 다음과 같이 수동 방식의 업데이트가 가능하다.


.. note::

   설치 / 업데이트는 반드시 root 권한으로하여야한다.


1. 외부 연결이 가능한 PC에서 STON을 다운로드한다. 다운로드 URL은 공식 release 이메일을 통해 배포된다.


2. 다운로드 한 파일을 PC에서 서버로 복사한다. 파일 이름의 형식은 다음과 같다.

   * RHEL/CentOS/openSUSE - ston. ``version`` .rhel.2.6.32.x64.tar.gz
   * Ubuntu - ston. ``version`` .ubuntu.2.6.32.x64.tar.gz


   CentOS 버전 ``2.4.9`` 이면 파일 이름은 ston.2.4.9.rhel.2.6.32.x64.tar.gz있다.


3. STON가 실행중인 경우 중지한다. ::

      service ston stop


4. 서버에서 복사 된 경로에 압축을 해제한다. ::

      tar zxvf ston.2.4.9.rhel.2.6.32.x64.tar.gz


   .. figure:: img/update_manual1.png
      :align: center

5. 설치 스크립트를 실행한다. STON가 실행 중이면 "yes"를 입력하고 중지한다. ::

      sh ston.2.4.9.rhel.2.6.32.x64.sh


   .. figure:: img/update_manual2.png
      :align: center

6. 설치가 완료되면 STON를 시작합니다. ::

      service ston start


   .. figure:: img/update_manual3.png
      :align: center


7. 정상 구동 된 것을 확인한다. ::

      ps -ef | grep ston


   .. figure:: img/update_manual4.png
      :align: center


.. _getting-started-run:

실행
====================================

일반적으로 기본 경로에 STON를 설치한다. ::

    /usr/local/ston/

다음 파일 중 하나가 없거나 XML 문법에 맞지 않는 경우는 실행되지 않는다.

- license.xml
- server.xml
- vhosts.xml

초기 설치시 모든 XML 파일이 존재하지 않는다. 배포 된 라이센스 파일을 설치 경로에 복사합니다. 그리고 설치 경로 server.xml.default과 vhosts.xml.default을 복사하거나 변경하여 설정하기 바란다. * .default 파일은 항상 최신 패키지와 함께 배포된다.


.. _getting-started-samplevhost:

Hello World
====================================
vhosts.xml 파일을 열고 다음과 같이 편집한다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>hello.winesoft.co.kr</Address>
            </Origin>
        </Vhost>
    </Vhosts>


.. _getting-started-runston:

STON 실행
-----------------------------------------------
1. 발행 된 license.xml 설치 경로에 복사합니다.

2. server.xml을 열고 <Storage>를 구성한다. ::

    <Server>
        <Cache>
            <Storage>
                <Disk>/cache1/</Disk>
                <Disk>/cache2/</Disk>
            </Storage>
        </Cache>
    </Server>

.. note::

   STON는 기본적으로 디스크 스토리지로 사용하기 때문에 디스크가 설정되어 있지 않은 경우, 구동되지 않는다. 자세한 내용은 다음 장에서 설명한다.

3. STON를 실행한다.  ::

      [root@localhost ~]# service ston start

   STON을 중지하려면 stop 명령을 사용합니다.  ::

      [root@localhost ~]# service ston stop


.. _getting-started-runcheck:

가상 호스트 동작 확인
-----------------------------------------------

(Windows 7 기반) C : \ Windows \ System32 \ drivers \ etc \ hosts 파일에 다음과 같이 www.example.com 도메인을 설정한다. ::

    192.168.0.100        www.example.com

브라우저에서 www.example.com에 액세스 할 때 다음 페이지가 제대로 서비스되면 성공이다.

   .. figure:: img/helloworld3.png
      :align: center


.. _getting-started-rrderr:

WM이 느려지거나 그래프가 나오지 않는 문제
-----------------------------------------------

설치 과정에서 RRD 그래프는 동적으로 다운로드하여 설치된다. 제한된 네트워크에서 STON을 설치하는 경우 RRD가 제대로 설치되지 않을 수 있습니다. 그러면 :ref:`wm` 가 매우 느리게 작동하는지 :ref:`api-graph` 가 동작하지 않게된다. 다음과 같이 변경한다.


**1. rrdtool 설치 확인**

   다음과 같이 설치 여부를 확인한다. ::

      [root@localhost ston]# yum install rrdtool
      Loaded plugins: fastestmirror, security
      Loading mirror speeds from cached hostfile
      * base: centos.mirror.cdnetworks.com
      * elrepo: ftp.ne.jp
      * epel: mirror.premi.st
      * extras: centos.mirror.cdnetworks.com
      * updates: centos.mirror.cdnetworks.com
      Setting up Install Process
      Package rrdtool-1.3.8-6.el6.x86_64 already installed and latest version
      Nothing to do

   다음은 Ubuntu 계열의 경우이다. ::

      root@ubuntu:~# apt-get install rrdtool
      Reading package lists... Done
      Building dependency tree
      Reading state information... Done
      rrdtool is already the newest version.
      The following packages were automatically installed and are no longer required:
        libgraphicsmagick3 libgraphicsmagick++3 libgraphicsmagick1-dev libgraphics-magick-perl libgraphicsmagick++1-dev
      Use 'apt-get autoremove' to remove them.
      0 upgraded, 0 newly installed, 0 to remove and 102 not upgraded.


**2. RRD 수동 설치**

   만약 rrdtool가 yum을 사용하여 설치가되어 있지 않으면 OS 버전에 맞는 패키지를 `다운로드 <http://pkgs.repoforge.org/rrdtool/>`_ 한 후 수동으로 설치한다.

======================================== =================== ======= ============================
Name                                     Last Modified       Size    Description
======================================== =================== ======= ============================
tcl-rrdtool-1.4.7-1.el5.rf.i386.rpm      06-Apr-2012 16:57   36K     RHEL5 and CentOS-5 x86 32bit
tcl-rrdtool-1.4.7-1.el5.rf.x86_64.rpm	 06-Apr-2012 16:57   37K     RHEL5 and CentOS-5 x86 64bit
tcl-rrdtool-1.4.7-1.el6.rfx.i686.rpm     06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 32bit
tcl-rrdtool-1.4.7-1.el6.rfx.x86_64.rpm	 06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 64bit
======================================== =================== ======= ============================


.. _env-vhost-activeorigin:

소스 서버
============================================

가상 호스트는 원래의 서버를 대신하여 콘텐츠를 서비스하는 것이 목적이다. 서비스 형태에 따라 다양한 소스 서버는 다양한 접근이 가능하다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address>``
   가상 호스트가 콘텐츠를 원본 서버의 주소. 수의 제한은 없다. 주소가 2 개 이상인 경우는 Active / Active 방식 (Round-Robin)에서 선택된다. 원래 서버의 주소, 포트가 80 인 경우 생략 할 수있다.。

예를 들어, 다른 포트 (8080)에서 서비스되는 경우 1.1.1.1 : 8080과 같이 포트 번호를 명시하여야한다. 주소는 {IP | Domain} {Port} {Path}의 형식으로 8 가지 형태가 가능하다.

============================== ==========================
Address                        Hostヘッダ
============================== ==========================
1.1.1.1	                       仮想ホスト名
1.1.1.1:8080	               仮想ホスト名:8080
1.1.1.1/account/dir	       仮想ホスト名
1.1.1.1:8080/account/dir       仮想ホスト名:8080
example.com	                   example.com
example.com:8080	           example.com:8080
example.com/account/dir	       example.com
example.com:8080/account/dir   example.com:8080
============================== ==========================

:ref:`origin-httprequest` 중 Host 헤더를 별도로 설정하지 않으면 표 Host 헤더를 전송한다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>origin.com:8888/account/dir</Address>
            </Origin>
        </Vhost>
    </Vhosts>

예를 들어 위와 같이 설정하면 원본으로 다음과 같이 요청한다. ::

   GET / HTTP/1.1
   Host: origin.com:8888

.. note::

   소스 서버에 example.com/account/dir 있도록 경로가 붙어있는 경우 요청 된 URL은 원래 서버의 주소의 경로 뒤에 붙는다. 클라이언트가 /img.jpg을 요청하면 최종 주소는 example.com/account/dir/img.jpg된다.


.. _env-vhost-standbyorigin:

보조 원본 주소
------------------------------------------------

보조 소스 서버를 설정한다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
                <Address2>1.1.1.3</Address2>
                <Address2>1.1.1.4</Address2>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address2>``

   모든 ``<Address>`` 가 제대로 작동하는 경우 ``<Address2>`` 는 서비스에 투입되지 않는다. Active 서버에 오류가 감지되면 해당 서버를 대체하기 위해 투입되고 Active 서버가 복구되면 다시 Standby 상태로 돌아갑니다. 만약 Standby 서버에 오류가 감지되면 Standby 서버가 복구 될 때까지 서비스에 투입되지 않는다.


.. _api-etc-help:

API 호출
====================================

HTTP 기반의 API를 제공한다. API 호출 권한은 :ref:`env-host` 에 영향을 받는다. 허용되지 않은 경우 즉시 연결을 종료한다.

STON 버전을 확인한다. ::

   http://127.0.0.1:10040/version

동일한 API를 Linux Shell의 명령에 실시한다. ::

   ./stonapi version

.. note::

   HTTP API는 &를 QueryString 구분 기호로 인식하고 있지만 Linux 콘솔에서는 다른 의미를 갖는다. & 들어가는 명령을 호출하는 경우 \ & 두로 기입하거나 반드시 괄호 ( "/ ... & ...")에서 호출하는 URL을 묶어야합니다.


하드웨어 정보 조회
====================================

하드웨어 정보를 조회한다. ::

   http://127.0.0.1:10040/monitoring/hwinfo

결과는 JSON 형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "hwinfo",
      "status": "OK",
      "result":
      {
         "OS" : "Linux version 3.3.0 ...(생략)...",
         "STON" : "1.1.9",
         "CPU" :
         {
            "ProcCount": "4",
            "Model": "Intel(R) Xeon(R) CPU           E5606  @ 2.13GHz",
            "MHz": "1200.000",
            "Cache": "8192 KB"
         },
         "Memory" : "8 GB",
         "NIC" :
         [
            {
               "Dev" : "eth1",
               "Model" : "Intel Corporation 82574L Gigabit Network Connection",
               "IP" : "192.168.0.13",
               "MAC" : "00:25:90:36:f4:cb"
            }
         ],
         "Disk" :
         [
            {
               "Dev" : "sda",
               "Model" : "HP DG0146FAMWL (scsi)",
               "Total" : "238787584",
               "Usage" : "40181760"
            },
            {
               "Dev" : "sdb",
               "Model" : "HITACHI HUC103014CSS600 (scsi)",
               "Total" : "144706478080",
               "Usage" : "2101075968"
            },
            {
               "Dev" : "sdc",
               "Model" : "HITACHI HUC103014CSS600 (scsi)",
               "Total" : "144706478080",
               "Usage" : "2012160000"
            }
         ]
      }
   }


다시 시작 / 종료
====================================

명령을 사용하여 STON를 다시 시작 / 종료 할 수있다. 예기치 않은 결과를 방지하기 위해 Web 페이지를 통해 확인 작업이 필요 있도록 개발되었다. ::

   http://127.0.0.1:10040/command/restart
   http://127.0.0.1:10040/command/restart?key=JUSTDOIT       // すぐに実行
   http://127.0.0.1:10040/command/terminate
   http://127.0.0.1:10040/command/terminate?key=JUSTDOIT       // すぐに実行


.. _getting-started-reset:

Caching 초기화
====================================

서비스를 중단하고 캐시 된 모든 콘텐츠를 제거한다. 설정된 모든 디스크를 포맷 작업이 완료되면 다시 서비스를 재개한다. ::

   http://127.0.0.1:10040/command/cacheclear
   http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT       // 즉시 실행

콘솔에서 다음 명령을 사용하여 전체 또는 하나의 가상 호스트를 초기화한다. ::

   ./stonapi reset
   ./stonapi reset/ston.winesoft.co.kr
