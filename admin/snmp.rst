.. _snmp:

제 11 장 SNMP
******************

이 장에서는 SNMP (Simple Network Management Protocol)에 대해 다룬다. 
:ref:`monitoring_stats` 의 모든 수치는 SNMP에 제공된다. 뿐만 아니라 더욱 세분화 된 시간 단위와 시스템의 상태 정보까지 제공한다. 가상 호스트에 대해 실시간 통계와 최대 60 분까지 "분"단위의 평균 통계를 제공한다.

- 다른 패키지가 필요 없다.
- snmpd를 개별적으로 실행하지 않는다.
- SNMP v1 및 v2c를 지원합니다.

.. toctree::
   :maxdepth: 2


.. _snmp-var:

변수
====================================

설정 및 사용자의 의도에 따라 변경 될 수있는 값을 변수 이름에 지정한다. 예를 들어, 디스크는 여러가 존재할 수있다. 이 경우 각 디스크를 가리키는 고유 번호가 필요하며, 입력 된 순서대로 1부터 할당된다. 이러한 변수를 ``[diskIndex]`` 에서 지정한다.

-  ``[diskIndex]``

   에서 지정한다. ::

      # server.xml - <Server><Cache>

      <Storage>
         <Disk>/cache1</Disk>
         <Disk>/cache2</Disk>
         <Disk>/cache3</Disk>
      </Storage>

   위와 같이 3 개의 디스크가 설정된 환경에서는 / cache1의
   ``[diskIndex]`` 1 / cache3의 ``[diskIndex]`` 는 3이있다. 예를 들어, / cache1의 전체 용량에 해당하는 OID는 system.diskInfo.diskInfoTotalSize.1 (1.3.6.1.4.1.40001.1.2.18.1.3) 1이된다. 마지막 .1 첫 번째 디스크를 의미한다.

-  ``[vhostIndex]``

   가상 호스트가로드 될 때 자동으로 부여된다. ::

      # vhosts.xml

      <Vhosts>
         <Vhost Status="Active" Name="kim.com"> ... </Vhost>
         <Vhost Status="Active" Name="lee.com"> ... </Vhost>
         <Vhost Status="Active" Name="park.com" StaticIndex="10300"> ... </Vhost>
      </Vhosts>

   먼저 위와 같이 3 개의 가상 호스트가로드되면 1부터 순서대로  ``[vhostIndex]`` 이 부여된다. 이후 가상 호스트는  ``[vhostIndex]`` 를 기억하고 가상 호스트가 삭제 되어도  ``[vhostIndex]`` 은 변하지 않는다. 가상 호스트의 삭제 및 추가가 동시에 발생한 경우 삭제 먼저 작동하고 신규 추가 된 가상 호스트 빈  ``[vhostIndex]`` 을 주어진다.

   .. figure:: img/snmp_vhostindex.png
      :align: center

      ``[vhostIndex]`` 동작

-  ``[diskMin]`` , ``[vhostMin]``

   시간 (분)을 의미한다. 5는 5 분의 평균을 나타내며 60는 60 분의 평균을 의미한다. 이 값은 1 (최소)에서 60 (분)까지의 범위를 가지며, 0은 실시간 (1 초)의 데이터를 의미한다.

SNMP는 동적으로 값이 변경 될 수있는 항목에 대해 Table 구조를 사용한다. 예를 들어, "전체 디스크 크기"는 디스크의 수에 따라 제공하는 데이터 수가 달라지기 때문에 Table 구조를 사용하여 표현해야한다. STON은 모든 가상 호스트에 대해 "분"단위의 통계 정보를 제공한다. 따라서 ``[vhostMin]`` . ``[vhostIndex]`` 라는 다소 모호한 표현을 제공한다.

이 표현은 가상 호스트별로 필요한 "분"단위의 통계를 볼 수있는 장점을 가지고 있지만, 변수가 2 개이므로, Table 구조로 표현하기 어려운 단점이있다. 이러한 문제를 극복하기 위해  ``[vhostMin]`` 의 기본값을 설정하여 SNMPWalk가 동작 할 수 있도록한다.


.. _snmp-conf:

유효
====================================

전역 설정 (server.xml)을 통해 SNMP 동작과 ACL을 설정한다. ::

   # server.xml - <Server><Host>

   <SNMP Port="161" Status="Inactive">
      <Allow>192.168.5.1</Allow>
      <Allow>192.168.6.0/24</Allow>
   </SNMP>

-  ``<SNMP>`` 속성을 통해 SNMP 작동을 설정한다.

   - ``Port (기본: 161)`` SNMP 서비스 포트

   - ``Status (기본: Inactive)`` NMP를 사용하려면이 값을 ``Active`` 로 설정한다.

-  ``<Allow>`` SNMP 액세스를 허용 할 IP 주소를 설정한다.。
    IP 지정 IP 주소 범위를 지정 비트 마스크 서브넷 네 가지 형식을 지원합니다. 연결 소켓이 허용 된 IP 주소가 없으면 응답을주지 않는다.



가상 호스트 / View 변수
====================================

SNMP를 통해 제공되는 가상 호스트 / View 번호와 기본 시간 (분)을 설정한다. ::

   # server.xml - <Server><Host>

   <SNMP VHostCount=0, VHostMin=5 ViewCount=0, ViewMin=5 />

-  ``VHostCount (기본: 0)`` 0의 경우 존재하는 가상 호스트까지의 응답을한다. 0보다 큰 값이면 가상 호스트의 존재 유무에 관계없이 설정된 가상 호스트까지의 응답이다.

-  ``ViewCount (기본: 0)``  View에 적용합니다. ( ``VHostCount`` 와 동일)

-  ``VHostMin (기본: 5분, 최대: 60분)``  ``[vhostMin]``  의 값을 설정한다. 0 ~ 60까지의 값을 가진다. 0의 경우 실시간 데이터를 제공하여 1-60 사이 인 경우 그만큼의 평균 값을 제공한다.

-  ``ViewMin (기본: 0)`` View에 적용합니다. ( ``VHostMin`` 와 동일)

예를 들어, 3 개의 가상 호스트가 설정되어있는 환경에서 SNMPWalk 동작이 달라진다.

- VHostCount = 0의 경우 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"

- VHostCount = 5의 경우 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.4 = ""
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.5 = ""



타 변수
---------------------

기타 변수를 설정한다. ::

   # server.xml - <Server><Host>

   <SNMP GlobalMin="5" DiskMin="5" ConfCount="10" />

-  ``GlobalMin (기본: 5분, 최대: 60분)``  ``[globalMin]``  의 값을 설정한다.

-  ``DiskMin (기본: 5分, 최대: 60분)``  ``[diskMin]``  의 값을 설정한다.

-  ``ConfCount (기본: 10)`` 설정의 목록을 n 개까지 볼 수 있습니다. 1-100 사이에서 지정 가능하다. 1은 현재의 반영 구성을 의미하고 2는 이전의 설정을 의미한다. 100은 현재를 기준으로 99 회 이전 설정을 의미한다.



Community
====================================

Community를 설정하여 허용 된 OID 만 액세스 / 차단하도록 설정한다. ::

   # server.xml - <Server><Host>

   <SNMP UnregisteredCommunity="Allow">
      <Community Name="example1" OID="Allow">
         <OID>1.3.6.1.4.1.40001.1.4.1</OID>
         <OID>1.3.6.1.4.1.40001.1.4.2</OID>
         <OID>1.3.6.1.4.1.40001.1.4.4</OID>
      </Community>
      <Community Name="example2" OID="Deny">
         <OID>1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61</OID>
      </Community>
   </SNMP>

``<SNMP>`` 의 ``UnregisteredCommunity`` 를 "Deny"로 설정하면 등록되지 않은 Community 요청을 차단한다.

-  ``<Community>`` Community를 설정한다.

   - ``Name`` Community 이름.

   - ``OID (기본: Allow)`` 서브 ``<OID>`` 태그의 값을 설정한다. 속성 값이 ``Allow`` 이면 하위 ``<OID>`` 목록 만 접근 가능하다. 반대로 속성 값이 ``Deny`` 이면 하위 <OID> 목록에는 접근이 불가능하다.


명시 적 OID (1.3.6.1.4.1.40001.1.4.4)와 범위 (OID 1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61) 표현이 가능하다. OID를 허용 / 차단하려면 하위 모든 OID에 대해 동일한 규칙이 적용된다.



.. _snmp-meta:

meta
====================================

::

   OID = 1.3.6.1.4.1.40001.1.1

메타 정보를 제공한다.

===== ============= ========= ===========================================
OID   Name          Type      Description
===== ============= ========= ===========================================
.1    manufacture   String    "WineSOFT Inc."
.2    software      String    "STON"
.3    version       String    버전
.4    hostname      String    호스트 이름
.5    state         String    "Healthy" 또는 "Inactive" 또는 "Emergency"
.6    uptime        Integer   실행 시간 (초)
.7    admin         String    <Admin> ... </Admin>
.10   Conf          OID       Conf 확장
===== ============= ========= ===========================================



.. _snmp-meta-conf:

meta.conf
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.1.10

``[confIndex]`` 은 ``<SNMP>`` 의 ``ConfCount`` 속성에서 설정합니다.
``[confIndex]`` 가 1이면 항상 현재 적용된 설정 값을 2의 경우는 이전의 설정 값을 의미한다. 10이면 현재의 (1)에서 9 이전 설정을 의미한다.

==================== ======= ======= =============================================================================================
OID                  Name    Type    Description
==================== ======= ======= =============================================================================================
.1. ``[confIndex]``  ID      Integer 설정 ID
.2. ``[confIndex]``  Time    Integer 설정 시간 (Unix 시간)
.3. ``[confIndex]``  Type    설정 모드 (0 = Unknown, 1 = STON 시작, 2 = / conf / reload 3 = / conf / upload 4 = / conf / restore)
.4. ``[confIndex]``  Size    Integer 설정 파일 크기
.5. ``[confIndex]``  Hash    String  설정 파일 Hash 문자열
.6. ``[confIndex]``  Path    String  설정 파일의 저장 경로
.7. ``[confIndex]``  Ver     String  설정시 STON 버전
==================== ======= ======= =============================================================================================



.. _snmp-meta-system:

system
====================================

::

   OID = 1.3.6.1.4.1.40001.1.2

STON을 실행하는 시스템의 정보를 제공한다.
``[sysMin]`` 변수는 0~60 분 사이의 값을 가지며, 실시간 또는 필요한 시간 동안의 평균값을 제공한다. SNMPWalk에서  ``[sysMin]`` 는 0으로 설정되어 현재의 정보를 제공한다.

=================== ========================================= ======= ===============================================
OID                 Name                                      Type    Description
=================== ========================================= ======= ===============================================
.1. ``[sysMin]``    cpuTotal                                  Integer 전체 CPU 사용률 (100 %)
.2. ``[sysMin]``                                                      전체 CPU 사용률 (10000 %)
.3. ``[sysMin]``    cpuKernel                                 Integer	CPU (Kernel)의 사용량 (100 %)
.4. ``[sysMin]``                                                      CPU (Kernel)의 사용량 (10000 %)
.5. ``[sysMin]``    cpuUser                                   Integer CPU (User)의 사용 비율 (100 %)
.6. ``[sysMin]``                                                      CPU (User)의 사용량 (10000 %)
.7. ``[sysMin]``    cpuIdle                                   Integer CPU (Idle) 사용량 (100 %)
.8. ``[sysMin]``                                                      CPU (Idle) 사용량 (10000 %)
.9                  memTotal                                  Integer 시스템 전체 메모리 (KB)
.10. ``[sysMin]``   memUse                                    Integer 시스템의 사용 메모리 (KB)
.11. ``[sysMin]``   memFree                                   Integer 시스템의 사용 가능한 메모리 (KB)
.12. ``[sysMin]``   memSTON                                   Integer STON 사용 메모리 (KB)
.13. ``[sysMin]``   memUseRatio                               Integer 시스템 메모리 사용률 (100 %)
.14. ``[sysMin]``                                                     시스템 메모리 사용률 (10000 %)
.15. ``[sysMin]``   memSTONRatio                              Integer STON 메모리 사용률 (100 %)
.16. ``[sysMin]``                                                     STON 메모리 사용률 (10000 %)
.17                 diskCount                                 Integer disk 수
.18.1               diskInfo                                  OID     diskInfo 확장
.19.1               diskPerf                                  OID     diskPerf 확장
.20. ``[sysMin]``   cpuProcKernel                             Integer STON가 사용하는 CPU (Kernel)의 사용량 (100 %)
.21. ``[sysMin]``                                                     STON가 사용하는 CPU (Kernel)의 사용량 (10000 %)
.22. ``[sysMin]``   cpuProcUser                               Integer STON가 사용하는 CPU (User)의 사용 비율 (100 %)
.23. ``[sysMin]``                                                     STON가 사용하는 CPU (User)의 사용량 (10000 %)
.24. ``[sysMin]``   sysLoadAverage                            Integer Load Average 1 분 평균 (0.01)
.25. ``[sysMin]``                                                     Load Average 5 분 평균 (0.01)
.26. ``[sysMin]``                                                     Load Average 15 분 평균 (0.01)
.27. ``[sysMin]``   cpuNice                                   Integer CPU(Nice) (100%)
.28. ``[sysMin]``                                                     CPU(Nice) (10000%)
.29. ``[sysMin]``   cpuIOWait                                 Integer CPU(IOWait) (100%)
.30. ``[sysMin]``                                                     CPU(IOWait) (10000%)
.31. ``[sysMin]``   cpuIRQ                                    Integer CPU(IRQ) (100%)
.32. ``[sysMin]``                                                     CPU(IRQ) (10000%)
.33. ``[sysMin]``   cpuSoftIRQ                                Integer CPU(SoftIRQ) (100%)
.34. ``[sysMin]``                                                     CPU(SoftIRQ) (10000%)
.35. ``[sysMin]``   cpuSteal                                  Integer CPU(Steal) (100%)
.36. ``[sysMin]``   CPU(Steal)                                Integer (10000%)
.40. ``[sysMin]``   TCPSocket.Established. ``[globalMin]``    Integer Established 상태의 TCP 연결 수
.41. ``[sysMin]``   TCPSocket.Timewait. ``[globalMin]``       Integer TIME_WAIT 상태의 TCP 연결 수
.42. ``[sysMin]``   TCPSocket.Orphan. ``[globalMin]``         Integer 아직 file handle에 attach되어 있지 않은 TCP 연결
.43. ``[sysMin]``   TCPSocket.Alloc. ``[globalMin]``          Integer 할당 된 TCP 연결
.44. ``[sysMin]``   TCPSocket.Mem. ``[globalMin]``            Integer undocumented
=================== ========================================= ======= ===============================================



.. _snmp-meta-system-diskinfo:

system.diskInfo
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.2.18.1

디스크의 정보를 제공한다.

======================= ================== =========== =========================================
OID                     Name               Type        Description
======================= ================== =========== =========================================
.2. ``[diskIndex]``     diskInfoPath       String      디스크 경로
.3. ``[diskIndex]``     diskInfoTotalSize  Integer     전체 디스크 용량 (MB)
.4. ``[diskIndex]``     diskInfoUseSize    Integer     디스크 사용량 (MB)
.5. ``[diskIndex]``     diskInfoFreeSize   Integer     디스크의 사용 가능한 용량 (MB)
.6. ``[diskIndex]``     diskInfoUseRatio   Integer     디스크 사용률 (100 %)
.7. ``[diskIndex]``                                    디스크 사용률 (10000 %)
.8. ``[diskIndex]``     diskInfoStatus     String      "Normal"또는 "Invalid"또는 "Unmounted"
======================= ================== =========== =========================================



.. _snmp-meta-system-diskperf:

system.diskPerf
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.2.19.1

디스크 성능 상태를 제공한다.

======================================== =========================== ========== ===============================
OID                                      Name                        Type       Description
======================================== =========================== ========== ===============================
.2. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadCount           Integer    읽기 성공 횟수
.3. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadMergedCount     Integer    로드가 병합 된 횟수
.4. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadSectorsCount    Integer    읽은 섹터
.5. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadTime            Integer    읽기 시간 (ms)
.6. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteCount          Integer    쓰기 성공 횟수
.7. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteMergedCount    Integer    쓰기가 병합 된 횟수
.8. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteSectorsCount   Integer    쓰는 섹터
.9. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteTime           Integer    쓰기 시간 (ms)
.10. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOProgressCount     Integer    진행중인 IO 수
.11. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTime              Integer    IO 소요 시간 (ms)
.12. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTimeWeighted      Integer    IO 소요 시간 (ms 가중치)
======================================== =========================== ========== ===============================



.. _snmp-global:

global
====================================

::

   OID = 1.3.6.1.4.1.40001.1.3

STON의 모든 모듈이 공통으로 사용하는 리소스 정보 (소켓, 이벤트 등)를 제공한다.

-  **ServerSocket**

   클라이언트 ~STON 구간. STON가 클라이언트의 요청을 처리하는 데 사용되는 소켓

-  **ClientSocket**

   STON~ 소스 서버 구간. STON가 원래 서버로 요청을 전송하는 데 사용되는 소켓

===== =========================================== ========== ==================================================
OID   Name                                        Type       Description
===== =========================================== ========== ==================================================
.5    EQ. ``[globalMin]``                         Integer    STON Framework에서 아직 처리되지 않은 Event 수
.6    RQ. ``[globalMin]``                         Integer    최근 서비스 된 내용을 참조 큐에 저장된 Event 수
.7    waitingFiles2Write. ``[globalMin]``         Integer    쓰기 대기중인 파일 수
.10   ServerSocket.Total. ``[globalMin]``         Integer    전체 서버 소켓 수
.11   ServerSocket.Established. ``[globalMin]``   Integer    연결된 상태의 서버 소켓
.12   ServerSocket.Accepted. ``[globalMin]``      Integer    새로 연결된 서버 소켓 수
.13   ServerSocket.Closed. ``[globalMin]``        Integer    연결이 종료 된 서버 소켓
.20   ClientSocket.Total. ``[globalMin]``         Integer    모든 클라이언트 소켓 수
.21   ClientSocket.Established. ``[globalMin]``   Integer    연결된 상태의 클라이언트 소켓
.22   ClientSocket.Accepted. ``[globalMin]``      Integer    새로 연결된 클라이언트 소켓
.23   ClientSocket.Closed. ``[globalMin]``        Integer    연결이 종료 된 클라이언트 소켓
.30   ServiceAccess.Allow. ``[globalMin]``        Integer    ServiceAccess에서 허용 (Allow) 소켓 수
.31   ServiceAccess.Deny. ``[globalMin]``         Integer    ServiceAccess 의해 거부 (Deny) 소켓 수
===== =========================================== ========== ==================================================



.. _snmp-cache:

cache
====================================

::

    OID = 1.3.6.1.4.1.40001.1.4

캐시 서비스 통계는 가상 호스트에 대해 상세하게 수집 / 제공된다.

====== ============== ========= ============================================================
OID    Name           Type      Description
====== ============== ========= ============================================================
.1     host           OID       호스트 (확장)
.2     vhostCount     Integer   가상 호스트의 수
.3.1   vhost          OID       가상 호스트 별 통계
.4     vhostIndexMax  Integer    ``[vhostIndex]``  최대. SNMPWalk는이 수치를 기준으로 동작한다.
.10    viewCount      Integer   View 수
.11.1  view           OID       View 다른 통계
.12    viewIndexMax   Integer   [viewIndex] 최대. SNMPWalk는이 수치를 기준으로 동작한다.
====== ============== ========= ============================================================



.. _snmp-cache-host:

cache.host
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1

호스트 (= 모든 가상 호스트)의 정보를 제공한다.

===== ========= =========== =========================
OID   Name      Type        Description
===== ========= =========== =========================
.2    name      String      호스트 이름
.3    status    String      "Healthy" 또는 "Inactive"
.4    uptime    Integer     STON 실행 시간 (초)
.10   contents  OID         컨텐츠 정보 (확장)
.11   traffic   OID         통계 (확장)
===== ========= =========== =========================



.. _snmp-cache-host-contents:

cache.host.contents
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.10

호스트 (= 모든 가상 호스트)가 서비스하는 콘텐츠 정보를 제공한다.

====== ================ ========== ============================
OID    Name             Type       Description
====== ================ ========== ============================
.1     memory           Integer    메모리 캐시 크기 (KB)
.2     filesTotalCount  Integer    서비스중인 파일 수
.3     filesTotalSize   Integer    서비스중인 전체 파일 용량 (MB)
.10    filesCountU1KB   Integer    1KB미만의 파일 수
.11    filesCountU2KB   Integer    2KB미만의 파일 수
.12    filesCountU4KB   Integer    4KB미만의 파일 수
.13    filesCountU8KB   Integer    8KB미만의 파일 수
.14    filesCountU16KB  Integer    16KB미만의 파일 수
.15    filesCountU32KB  Integer    32KB미만의 파일 수
.16    filesCountU64KB  Integer    64KB미만의 파일 수
.17    filesCountU128KB Integer    128KB미만의 파일 수
.18    filesCountU256KB Integer    256KB미만의 파일 수
.19    filesCountU512KB Integer    512KB미만의 파일 수
.20    filesCountU1MB   Integer    1MB미만의 파일 수
.21    filesCountU2MB   Integer    2MB미만의 파일 수
.22    filesCountU4MB   Integer    4MB미만의 파일 수
.23    filesCountU8MB   Integer    8MB미만의 파일 수
.24    filesCountU16MB  Integer    16MB미만의 파일 수
.25    filesCountU32MB  Integer    32MB미만의 파일 수
.26    filesCountU64MB  Integer    64MB미만의 파일 수
.27    filesCountU128MB Integer    128MB미만의 파일 수
.28    filesCountU256MB Integer    256MB미만의 파일 수
.29    filesCountU512MB Integer    512MB미만의 파일 수
.30    filesCountU1GB   Integer    1GB미만의 파일 수
.31    filesCountU2GB   Integer    2GB미만의 파일 수
.32    filesCountU4GB   Integer    4GB미만의 파일 수
.33    filesCountU8GB   Integer    8GB미만의 파일 수
.34    filesCountU16GB  Integer    16GB미만의 파일 수
.35    filesCountO16GB  Integer    16GB이상의 파일 수
====== ================ ========== ============================



.. _snmp-cache-host-traffic:

cache.host.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11

호스트 (= 모든 가상 호스트)의 캐시 서비스 및 트래픽 통계를 제공한다. traffic의 모든 통계는 최대 60 분까지의 평균으로 제공한다. min은 '분'을 의미하고 최대 60까지의 값을 가진다. min이 생략되거나 0이면 실시간 정보를 제공한다.

===================== =============== ======= ==============================
OID                   Name            Type    Description
===================== =============== ======= ==============================
.1. ``[vhostMin]``    requestHitRatio Integer Request Hit Ratio(100%)
.2. ``[vhostMin]``                            Request Hit Ratio(10000%)
.3. ``[vhostMin]``    bytesHitRatio   Integer Bytes Hit Ratio(100%)
.4. ``[vhostMin]``                            Bytes Hit Ratio(10000%)
.10                   origin          OID     원래의 트래픽 정보 (확장)
.11                   client          OID     클라이언트의 트래픽 정보 (확장)
===================== =============== ======= ==============================



.. _snmp-cache-host-traffic-origin:

cache.host.traffic.origin
---------------------

::

    OID = 1.3.6.1.4.1.40001.1.4.1.11.10

소스 서버 트래픽 통계를 제공한다. 소스 서버의 트래픽은 HTTP 트래픽과 Port 우회 트래픽으로 구분한다.

========================== =================================== ========== ===================================================================
OID                        Name                                Type       Description
========================== =================================== ========== ===================================================================
.1. ``[vhostMin]``         inbound                             Integer    소스 서버에서받은 평균 트래픽 (Bytes)
.2. ``[vhostMin]``         outbound                            Integer    소스 서버로 전송 평균 트래픽 (Bytes)
.3. ``[vhostMin]``         sessionAverage                      Integer    전체 원본 서버의 평균 방문수
.4. ``[vhostMin]``         activesessionAverage                Integer    전체 원본 서버 세션에서 전송되는 평균 세션 수 
.10                        http                                OID        소스 서버의 HTTP 트래픽 정보
.10.1. ``[vhostMin]``      http.inbound                        Integer    소스 서버에서받은 평균 HTTP 트래픽 (Bytes)
.10.2. ``[vhostMin]``      http.outbound                       Integer    소스 서버로 전송 평균 HTTP 트래픽 (Bytes)
.10.3. ``[vhostMin]``      http.sessionAverage                 Integer    소스 서버의 평균 HTTP 세션 수
.10.4. ``[vhostMin]``      http.reqHeaderSize                  Integer    소스 서버로 전송 평균 HTTP Header 트래픽 (Bytes)
.10.5. ``[vhostMin]``      http.reqBodySize                    Integer    소스 서버로 전송 평균 HTTP Body 트래픽 (Bytes)
.10.6. ``[vhostMin]``      http.resHeaderSize                  Integer    소스 서버에서받은 평균 HTTP Header 트래픽 (Bytes)
.10.7. ``[vhostMin]``      http.resBodySize                    Integer    소스 서버에서받은 평균 HTTP Body 트래픽 (Bytes)
.10.8. ``[vhostMin]``      http.reqAverage                     Integer    소스 서버로 전송되는 평균 HTTP 요청 수
.10.9. ``[vhostMin]``      http.reqCount                       Integer    소스 서버로 전송되는 HTTP 요청 수
.10.10. ``[vhostMin]``     http.resTotalAverage                Integer    원본 서버가 보낸 전체 평균 HTTP 응답 수
.10.11. ``[vhostMin]``     http.resTotalCompleteAverage        Integer    원본 서버에서 성공적으로 평균 HTTP 트랜잭션 수
.10.12. ``[vhostMin]``     http.resTotalTimeRes                Integer    원래 서버에서 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.13. ``[vhostMin]``     http.resTotalTimeComplete           Integer    원본 서버에서 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.14. ``[vhostMin]``     http.resTotalCount                  Integer    원본 서버가 보낸 전체 HTTP 응답 수
.10.15. ``[vhostMin]``     http.resTotalCompleteCount          Integer    원본 서버에서 성공적으로 HTTP 트랜잭션 수
.10.20. ``[vhostMin]``     http.res2xxAverage                  Integer    원본 서버가 보낸 평균 2xx 응답 수
.10.21. ``[vhostMin]``     http.res2xxCompleteAverage          Integer    원본 서버에서 성공적으로 평균 2xx 트랜잭션 수
.10.22. ``[vhostMin]``     http.res2xxTimeRes                  Integer    원본 서버에서 2xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.23. ``[vhostMin]``     http.res2xxTimeComplete             Integer    원본 서버에서 2xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms))
.10.24. ``[vhostMin]``     http.res2xxCount                    Integer    원본 서버가 보낸 2xx 응답 수
.10.25. ``[vhostMin]``     http.res2xxCompleteCount            Integer    원본 서버에서 성공한 2xx 트랜잭션 수
.10.30. ``[vhostMin]``     http.res3xxAverage                  Integer    원본 서버가 보낸 평균 3xx 응답 수
.10.31. ``[vhostMin]``     http.res3xxCompleteAverage          Integer    원본 서버에서 성공적으로 평균 3xx 트랜잭션 수
.10.32. ``[vhostMin]``     http.res3xxTimeRes                  Integer    원본 서버에서 3xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.33. ``[vhostMin]``     http.res3xxTimeComplete             Integer    원본 서버에서 3xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.34. ``[vhostMin]``     http.res3xxCount                    Integer    원본 서버가 보낸 3xx 응답 수
.10.35. ``[vhostMin]``     http.res3xxCompleteCount            Integer    원본 서버에서 성공한 3xx 트랜잭션 수
.10.40. ``[vhostMin]``     http.res4xxAverage                  Integer    원본 서버가 보낸 평균 4xx 응답 수
.10.41. ``[vhostMin]``     http.res4xxCompleteAverage          Integer    원본 서버에서 성공적으로 평균 4xx 트랜잭션 수
.10.42. ``[vhostMin]``     http.res4xxTimeRes                  Integer    원본 서버에서 4xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.43. ``[vhostMin]``     http.res4xxTimeComplete             Integer    원본 서버에서 4xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.44. ``[vhostMin]``     http.res4xxCount                    Integer    원본 서버가 보낸 4xx 응답 수
.10.45. ``[vhostMin]``     http.res4xxCompleteCount            Integer    원본 서버에서 성공한 4xx 트랜잭션 수
.10.50. ``[vhostMin]``     http.res5xxAverage                  Integer    원본 서버가 보낸 평균 5xx 응답 수
.10.51. ``[vhostMin]``     http.res5xxCompleteAverage          Integer    원본 서버에서 성공적으로 평균 5xx 트랜잭션 수
.10.52. ``[vhostMin]``     http.res5xxTimeRes                  Integer    원본 서버에서 5xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.53. ``[vhostMin]``     http.res5xxTimeComplete             Integer    원본 서버에서 5xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.54. ``[vhostMin]``     http.res5xxCount                    Integer    원본 서버가 보낸 5xx 응답 수
.10.55. ``[vhostMin]``     http.res5xxCompleteCount            Integer    원본 서버에서 성공한 5xx 트랜잭션 수
.10.60. ``[vhostMin]``     http.connectTimeoutAverage          Integer    평균 소스 서버 연결 실패 횟수
.10.61. ``[vhostMin]``     http.receiveTimeoutAverage          Integer    평균 원본 서버 전송에 실패한 횟수
.10.62. ``[vhostMin]``     http.connectAverage                 Integer    평균 소스 서버 접속 성공 횟수
.10.63. ``[vhostMin]``     http.dnsQueryTime                   Integer    소스 서버 연결시 평균 DNS 쿼리의 소요 시간
.10.64. ``[vhostMin]``     http.connectTime                    Integer    소스 서버의 평균 접속 시간 (0.01ms)
.10.65. ``[vhostMin]``     http.connectTimeoutCount            Integer    소스 서버 연결 실패 횟수
.10.66. ``[vhostMin]``     http.receiveTimeoutCount            Integer    소스 서버의 전송에 실패한 횟수
.10.67. ``[vhostMin]``     http.connectCount                   Integer    소스 서버 접속 성공 횟수
.10.68. ``[vhostMin]``     http.closeAverage                   Integer    전송 중에 소스 서버에서 먼저 소켓을 종료 한 평균 횟수
.10.69. ``[vhostMin]``     http.closeCount                     Integer    전송 중에 소스 서버에서 먼저 소켓을 종료 한 횟수
.11                        portbypass                          OID        Port 우회 원본 서버의 트래픽 정보
.11.1. ``[vhostMin]``      portbypass.inbound                  Integer    Port 우회를 통해 원본 서버에서받는 평균 트래픽 (Bytes)
.11.2. ``[vhostMin]``      portbypass.outbound                 Integer    Port 우회를 통해 원래의 서버로 전송하는 평균 트래픽 (Bytes)
.11.3. ``[vhostMin]``      portbypass.sessionAverage           Integer    Port 우회 동안 평균 원본 서버의 세션 수
.11.4. ``[vhostMin]``      portbypass.closedAverage            Integer    Port 우회중인 원본 서버가 연결을 종료 한 평균 횟수
.11.5. ``[vhostMin]``      portbypass.connectTimeoutAverage    Integer    Port 우회하는 서버의 평균 연결 실패 횟수
.11.6. ``[vhostMin]``      portbypass.closedCount              Integer    Port 우회중인 원본 서버가 연결을 종료 한 횟수
.11.7. ``[vhostMin]``      portbypass.connectTimeoutCount      Integer    Port 우회하는 서버 연결 실패 횟수
========================== =================================== ========== ===================================================================



.. _snmp-cache-host-traffic-client:

cache.host.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.11

클라이언트 트래픽 통계를 제공한다. 클라이언트의 트래픽은 HTTP 트래픽은 SSL 트래픽, Port 우회 트래픽으로 구분된다. SNMP는 디렉토리 별 통계를 제공하지 않는다. 비록 디렉토리 통계가 설정되어있다하더라도 합산되어 있습니다.

========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    클라이언트에서받은 평균 트래픽 (Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    클라이언트로 전송 평균 트래픽 (Bytes)
.3. ``[vhostMin]``         sessionAverage                             Integer    전체 평균 클라이언트 세션 수
.4. ``[vhostMin]``         activesessionAverage                       Integer    모든 클라이언트에 전송되는 평균 세션 수
.10                        http                                       OID        클라이언트의 HTTP 트래픽 정보
.10.1. ``[vhostMin]``      http.inbound                               Integer    클라이언트에서받은 평균 HTTP 트래픽 (Bytes)
.10.2. ``[vhostMin]``      http.outbound                              Integer    클라이언트로 전송 평균 HTTP 트래픽 (Bytes)
.10.3. ``[vhostMin]``      http.sessionAverage                        Integer    클라이언트의 평균 HTTP 세션 수
.10.4. ``[vhostMin]``      http.reqHeaderSize                         Integer    클라이언트에서받은 평균 HTTP Header 트래픽 (Bytes)
.10.5. ``[vhostMin]``      http.reqBodySize                           Integer    클라이언트에서받은 평균 HTTP Body 트래픽 (Bytes)
.10.6. ``[vhostMin]``      http.resHeaderSize                         Integer    클라이언트로 전송 평균 HTTP Header 트래픽 (Bytes)
.10.7. ``[vhostMin]``      http.resBodySize                           Integer    클라이언트로 전송 평균 HTTP Body 트래픽 (Bytes)
.10.8. ``[vhostMin]``      http.reqAverage                            Integer    클라이언트에서받은 평균 HTTP 요청 수
.10.9. ``[vhostMin]``      http.reqCount                              Integer    클라이언트로부터 수신 한 HTTP 요청 수
.10.10. ``[vhostMin]``     http.resTotalAverage                       Integer    클라이언트로 전송 평균 전체 응답 수
.10.11. ``[vhostMin]``     http.resTotalCompleteAverage               Integer    클라이언트가 완료 한 평균 HTTP 트랜잭션 수
.10.12. ``[vhostMin]``     http.resTotalTimeRes                       Integer    클라이언트의 응답의 평균 소요 시간 (0.01ms)
.10.13. ``[vhostMin]``     http.resTotalTimeComplete                  Integer    클라이언트 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.14. ``[vhostMin]``     http.resTotalCount                         Integer    클라이언트에 보낼 전체 응답 수
.10.15. ``[vhostMin]``     http.resTotalCompleteCount                 Integer    클라이언트가 완료된 HTTP 트랜잭션 수
.10.20. ``[vhostMin]``     http.res2xxAverage                         Integer    클라이언트로 전송 평균 2xx 응답 수
.10.21. ``[vhostMin]``     http.res2xxCompleteAverage                 Integer    클라이언트가 완료 한 평균 2xx 트랜잭션 수
.10.22. ``[vhostMin]``     http.res2xxTimeRes                         Integer    클라이언트 2xx 응답의 평균 소요 시간 (0.01ms)
.10.23. ``[vhostMin]``     http.res2xxTimeComplete                    Integer    클라이언트 2xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.24. ``[vhostMin]``     http.res2xxCount                           Integer    클라이언트로 전송 2xx 응답 수
.10.25. ``[vhostMin]``     http.res2xxCompleteCount                   Integer    클라이언트가 완료된 2xx 트랜잭션 수
.10.30. ``[vhostMin]``     http.res3xxAverage                         Integer    클라이언트로 전송 평균 3xx 응답 수
.10.31. ``[vhostMin]``     http.res3xxCompleteAverage                 Integer    클라이언트가 완료 한 평균 3xx 트랜잭션 수
.10.32. ``[vhostMin]``     http.res3xxTimeRes                         Integer    클라이언트 3xx 응답의 평균 소요 시간 (0.01ms)
.10.33. ``[vhostMin]``     http.res3xxTimeComplete                    Integer    클라이언트 3xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.34. ``[vhostMin]``     http.res3xxCount                           Integer    클라이언트로 전송 3xx 응답 수
.10.35. ``[vhostMin]``     http.res3xxCompleteCount                   Integer    클라이언트가 완료된 3xx 트랜잭션 수
.10.40. ``[vhostMin]``     http.res4xxAverage                         Integer    클라이언트로 전송 평균 4xx 응답 수
.10.41. ``[vhostMin]``     http.res4xxCompleteAverage                 Integer    클라이언트가 완료 한 평균 4xx 트랜잭션 수
.10.42. ``[vhostMin]``     http.res4xxTimeRes                         Integer    클라이언트 4xx 응답의 평균 소요 시간 (0.01ms)
.10.43. ``[vhostMin]``     http.res4xxTimeComplete                    Integer    클라이언트 4xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.44. ``[vhostMin]``     http.res4xxCount                           Integer    클라이언트로 전송 4xx 응답 수
.10.45. ``[vhostMin]``     http.res4xxCompleteCount                   Integer    클라이언트가 완료된 4xx 트랜잭션 수
.10.50. ``[vhostMin]``     http.res5xxAverage                         Integer    클라이언트로 전송 평균 5xx 응답 수
.10.51. ``[vhostMin]``     http.res5xxCompleteAverage                 Integer    클라이언트가 완료 한 평균 5xx 트랜잭션 수
.10.52. ``[vhostMin]``     http.res5xxTimeRes                         Integer    클라이언트 5xx 응답의 평균 소요 시간 (0.01ms)
.10.53. ``[vhostMin]``     http.res5xxTimeComplete                    Integer    클라이언트 5xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.54. ``[vhostMin]``     http.res5xxCount                           Integer    클라이언트로 전송 5xx 응답 수
.10.55. ``[vhostMin]``     http.res5xxCompleteCount                   Integer    클라이언트가 완료된 5xx 트랜잭션 수
.10.60. ``[vhostMin]``     http.reqDeniedAverage                      Integer    차단 된 요청의 평균
.10.61. ``[vhostMin]``     http.reqDeniedCount                        Integer    차단 된 요청 수
.11                        portbypass                                 OID        Port 우회 클라이언트의 트래픽 정보
.11.1. ``[vhostMin]``      portbypass.inbound                         Integer    Port 우회를 통해 클라이언트로부터받는 평균 트래픽 (Bytes)
.11.2. ``[vhostMin]``      portbypass.outbound                        Integer    Port 우회를 통해 클라이언트로 전송 평균 트래픽 (Bytes)
.11.3. ``[vhostMin]``      portbypass.sessionAverage                  Integer    Port 우회하는 클라이언트의 평균 방문수
.11.4. ``[vhostMin]``      portbypass.closedAverage                   Integer    Port 우회중인 클라이언트가 연결을 종료 한 평균 횟수
.11.5. ``[vhostMin]``      portbypass.closedCount                     Integer    Port 우회중인 클라이언트가 연결을 종료 한 횟수
.12                        ssl                                        OID        SSL 클라이언트의 트래픽 정보
.12.2. ``[vhostMin]``      ssl.inbound                                Integer    SSL을 통해 클라이언트로부터받는 평균 트래픽 (Bytes)
.12.3. ``[vhostMin]``      ssl.outbound                               Integer    SSL을 통해 클라이언트로 전송 평균 트래픽 (Bytes)
.13                        requestHitAverage                          OID        평균 캐시 HIT 결과
.13.1. ``[vhostMin]``      requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.13.2. ``[vhostMin]``      requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.13.3. ``[vhostMin]``      requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.13.4. ``[vhostMin]``      requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.13.5. ``[vhostMin]``      requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.13.6. ``[vhostMin]``      requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.13.7. ``[vhostMin]``      requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.13.8. ``[vhostMin]``      requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.13.9. ``[vhostMin]``      requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.13.10. ``[vhostMin]``     requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.13.11. ``[vhostMin]``     requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.14                        requestHitCount                            OID        캐시 HIT 결과 수
.14.1. ``[vhostMin]``      requestHitCount.TCP_HIT                    Integer    TCP_HIT
.14.2. ``[vhostMin]``      requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.14.3. ``[vhostMin]``      requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.14.4. ``[vhostMin]``      requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.14.5. ``[vhostMin]``      requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.14.6. ``[vhostMin]``      requestHitCount.TCP_MISS                   Integer    TCP_MISS
.14.7. ``[vhostMin]``      requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.14.8. ``[vhostMin]``      requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.14.9. ``[vhostMin]``      requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.14.10. ``[vhostMin]``     requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.14.11. ``[vhostMin]``     requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
========================== ========================================== ========== =============================================================



.. _snmp-cache-host-traffic-filesystem:

cache.host.traffic.filesystem
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.20

Host의 File I / O 통계를 제공한다.

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requestHitRatio                              Integer    Request Hit Ratio(100%)
.2. ``[vhostMin]``                                                               Request Hit Ratio(10000%)
.3. ``[vhostMin]``       byteHitRatio                                 Integer    Byte Hit Ratio(100%)
.4. ``[vhostMin]``                                                               Byte Hit Ratio(10000%)
.5. ``[vhostMin]``       outbound                                     Integer    File I / O로 보내는 평균 트래픽 (Bytes)
.6. ``[vhostMin]``       session                                      Integer    File I / O를 수행중인 평균 Thread 수
.7                       requestHitAverage                            OID        평균 캐시 HIT 결과
.7.1. ``[vhostMin]``     requestHitAverage.TCP_HIT                    Integer    TCP_HIT
.7.2. ``[vhostMin]``     requestHitAverage.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]``     requestHitAverage.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]``     requestHitAverage.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]``     requestHitAverage.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]``     requestHitAverage.TCP_MISS                   Integer    TCP_MISS
.7.7. ``[vhostMin]``     requestHitAverage.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]``     requestHitAverage.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]``     requestHitAverage.TCP_DENIED                 Integer    TCP_DENIED
.7.10. ``[vhostMin]``    requestHitAverage.TCP_ERROR                  Integer    TCP_ERROR
.7.11. ``[vhostMin]``    requestHitAverage.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.8                       requestHitCount                              OID        캐시 HIT 결과 수
.8.1. ``[vhostMin]``     requestHitCount.TCP_HIT                      Integer    TCP_HIT
.8.2. ``[vhostMin]``     requestHitCount.TCP_IMS_HIT                  Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]``     requestHitCount.TCP_REFRESH_HIT              Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]``     requestHitCount.TCP_REF_FAIL_HIT             Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]``     requestHitCount.TCP_NEGATIVE_HIT             Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]``     requestHitCount.TCP_MISS                     Integer    TCP_MISS
.8.7. ``[vhostMin]``     requestHitCount.TCP_REFRESH_MISS             Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]``     requestHitCount.TCP_CLIENT_REFRESH_MISS      Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]``     requestHitCount.TCP_DENIED                   Integer    TCP_DENIED
.8.10. ``[vhostMin]``    requestHitCount.TCP_ERROR                    Integer    TCP_ERROR
.8.11. ``[vhostMin]``    requestHitCount.TCP_REDIRECT_HIT             Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]``      getattr.filecount                            Integer    (getattr 함수 호출) FILE로 응답 한 횟수
.11. ``[vhostMin]``      getattr.dircount                             Integer    (getattr 함수 호출) DIR에 응답 한 횟수
.12. ``[vhostMin]``      getattr.failcount                            Integer    (getattr 함수 호출) 실패에 응답 한 횟수
.13. ``[vhostMin]``      getattr.timeres                              Integer    (getattr 함수의 호출)의 반응 시간 (0.01ms)
.14. ``[vhostMin]``      open.count                                   Integer    open 함수의 호출 횟수
.15. ``[vhostMin]``      open.timeres                                 Integer    open 함수의 반응 시간 (0.01ms)
.16. ``[vhostMin]``      read.count                                   Integer    read 함수 호출 횟수
.17. ``[vhostMin]``      read.timeres                                 Integer    read 함수의 반응 시간 (0.01ms)
.18. ``[vhostMin]``      read.buffersize                              Integer    read 함수에서 요구 된 버퍼 크기 (Bytes)
.19. ``[vhostMin]``      read.bufferfilled                            Integer    read 함수에서 요구 된 버퍼에 포장 크기 (Bytes)
======================== ============================================ ========== =============================================



.. _snmp-cache-host-traffic-dims:

cache.host.traffic.dims
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.21

Host의 DIMS 변환 통계를 제공한다.

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requests                                     Integer    DIMS 변환 요청 수
.2. ``[vhostMin]``       converted                                    Integer    변환에 성공 횟수
.3. ``[vhostMin]``       failed                                       Integer    변환에 실패한 횟수
.4. ``[vhostMin]``       avgsrcsize                                   Integer    원본 이미지의 평균 크기 (Bytes)
.5. ``[vhostMin]``       avgdestsize                                  Integer    변환 된 이미지의 평균 크기 (Bytes)
.6. ``[vhostMin]``       avgtime                                      Integer    변환 시간 (ms)
======================== ============================================ ========== =============================================



.. _snmp-cache-host-traffic-compression:

cache.host.traffic.compression
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.22

Host 압축 통계를 제공한다.

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requests                                     Integer    압축 요청 수
.2. ``[vhostMin]``       converted                                    Integer    압축 성공 횟수
.3. ``[vhostMin]``       failed                                       Integer    압축 실패 횟수
.4. ``[vhostMin]``       avgsrcsize                                   Integer    원본 파일의 평균 크기 (Bytes)
.5. ``[vhostMin]``       avgdestsize                                  Integer    압축 된 파일의 평균 크기 (Bytes)
.6. ``[vhostMin]``       avgtime                                      Integer    압축 시간 (ms)
======================== ============================================ ========== =============================================





.. _snmp-cache-vhost:

cache.vhost
====================================

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1

가상 호스트의 정보를 제공한다. ``[vhostIndex]`` 1에서 가상 호스트 수의 범위를 가진다.

======================= ========= ========== ============================================
OID                     Name      Type       Description
======================= ========= ========== ============================================
.2. ``[vhostIndex]``    name      String     가상 호스트 이름
.3. ``[vhostIndex]``    status    String     "Healthy" 또는 "Inactive" 또는 "Emergency"
.4. ``[vhostIndex]``    uptime    Integer    가상 호스트의 실행 시간 (초)
.10                     contents  OID        컨텐츠 정보 (확장)
.11                     traffic   OID        통계 (확장)
======================= ========= ========== ============================================



.. _snmp-cache-vhost-contents:

cache.vhost.contents
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.10

가상 호스트가 서비스하는 콘텐츠 정보를 제공한다.

========================= =================== ========== =============================
OID                       Name                Type       Description
========================= =================== ========== =============================
.1. ``[vhostIndex]``      memory              Integer    메모리 캐시 크기 (KB)
.2. ``[vhostIndex]``      filesTotalCount     Integer    서비스중인 파일 수
.3. ``[vhostIndex]``      filesTotalSize      Integer    서비스중인 전체 파일 용량 (MB)
.10. ``[vhostIndex]``     filesCountU1KB      Integer    1KB미만의 파일 수
.11. ``[vhostIndex]``     filesCountU2KB      Integer    2KB미만의 파일 수
.12. ``[vhostIndex]``     filesCountU4KB      Integer    4KB미만의 파일 수
.13. ``[vhostIndex]``     filesCountU8KB      Integer    8KB미만의 파일 수
.14. ``[vhostIndex]``     filesCountU16KB     Integer    16KB미만의 파일 수
.15. ``[vhostIndex]``     filesCountU32KB     Integer    32KB미만의 파일 수
.16. ``[vhostIndex]``     filesCountU64KB     Integer    64KB미만의 파일 수
.17. ``[vhostIndex]``     filesCountU128KB    Integer    128KB미만의 파일 수
.18. ``[vhostIndex]``     filesCountU256KB    Integer    256KB미만의 파일 수
.19. ``[vhostIndex]``     filesCountU512KB    Integer    512KB미만의 파일 수
.20. ``[vhostIndex]``     filesCountU1MB      Integer    1MB미만의 파일 수
.21. ``[vhostIndex]``     filesCountU2MB      Integer    2MB미만의 파일 수
.22. ``[vhostIndex]``     filesCountU4MB      Integer    4MB미만의 파일 수
.23. ``[vhostIndex]``     filesCountU8MB      Integer    8MB미만의 파일 수
.24. ``[vhostIndex]``     filesCountU16MB     Integer    16MB미만의 파일 수
.25. ``[vhostIndex]``     filesCountU32MB     Integer    32MB미만의 파일 수
.26. ``[vhostIndex]``     filesCountU64MB     Integer    64MB미만의 파일 수
.27. ``[vhostIndex]``     filesCountU128MB    Integer    128MB미만의 파일 수
.28. ``[vhostIndex]``     filesCountU256MB    Integer    256MB미만의 파일 수
.29. ``[vhostIndex]``     filesCountU512MB    Integer    512MB미만의 파일 수
.30. ``[vhostIndex]``     filesCountU1GB      Integer    1GB미만의 파일 수
.31. ``[vhostIndex]``     filesCountU2GB      Integer    2GB미만의 파일 수
.32. ``[vhostIndex]``     filesCountU4GB      Integer    4GB미만의 파일 수
.33. ``[vhostIndex]``     filesCountU8GB      Integer    8GB미만의 파일 수
.34. ``[vhostIndex]``     filesCountU16GB     Integer    16GB미만의 파일 수
.35. ``[vhostIndex]``     filesCountO16GB     Integer    16GB이상의 파일 수
========================= =================== ========== =============================



.. _snmp-cache-vhost-traffic:

cache.vhost.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11

가상 호스트 캐시 서비스 및 트래픽 통계를 제공한다. traffic의 모든 통계는 최대 60 분까지의 평균 제공된다. min은 '분'을 의미하고 최대 60까지의 값을 가진다. min이 생략되거나 0이면 실시간 정보를 제공한다.

========================================= ================= =========== ==============================
OID                                       Name              Type        Description
========================================= ================= =========== ==============================
.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitRatio   Integer     Request Hit Ratio(100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                   Request Hit Ratio(10000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``     bytesHitRatio     Integer     Bytes Hit Ratio(100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                   Bytes Hit Ratio(10000%)
.10                                       origin            OID         원래의 트래픽 정보 (확장)
.11                                       client            OID         클라이언트의 트래픽 정보 (확장)
========================================= ================= =========== ==============================



.. _snmp-cache-vhost-traffic-origin:

cache.vhost.traffic.origin
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.10

소스 서버 트래픽 통계를 제공한다. 소스 서버의 트래픽은 HTTP 트래픽과 Port 우회 트래픽으로 구분된다.

============================================= ===================================== ========== =================================================================
OID                                           Name                                  Type       Description
============================================= ===================================== ========== =================================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                               Integer    소스 서버에서받은 평균 트래픽 (Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                              Integer    소스 서버로 전송 평균 트래픽 (Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         sessionAverage                        Integer    전체 원본 서버의 평균 방문수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activesessionAverage                  Integer    전체 원본 서버 세션에서 전송되는 평균 세션 수
.10                                           http                                  OID        소스 서버의 HTTP 트래픽 정보
.10.1. ``[vhostMin]`` . ``[vhostIndex]``      http.inbound                          Integer    소스 서버에서받은 평균 HTTP 트래픽 (Bytes)
.10.2. ``[vhostMin]`` . ``[vhostIndex]``      http.outbound                         Integer    소스 서버로 전송 평균 HTTP 트래픽 (Bytes)
.10.3. ``[vhostMin]`` . ``[vhostIndex]``      http.sessionAverage                   Integer    소스 서버의 평균 HTTP 세션 수
.10.4. ``[vhostMin]`` . ``[vhostIndex]``      http.reqHeaderSize                    Integer    소스 서버로 전송 평균 HTTP Header 트래픽 (Bytes)
.10.5. ``[vhostMin]`` . ``[vhostIndex]``      http.reqBodySize                      Integer    소스 서버로 전송 평균 HTTP Body 트래픽 (Bytes)
.10.6. ``[vhostMin]`` . ``[vhostIndex]``      http.resHeaderSize                    Integer    소스 서버에서받은 평균 HTTP Header 트래픽 (Bytes)
.10.7. ``[vhostMin]`` . ``[vhostIndex]``      http.resBodySize                      Integer    소스 서버에서받은 평균 HTTP Body 트래픽 (Bytes)
.10.8. ``[vhostMin]`` . ``[vhostIndex]``      http.reqAverage                       Integer    소스 서버로 전송되는 평균 HTTP 요청 수
.10.9. ``[vhostMin]`` . ``[vhostIndex]``      http.reqCount                         Integer    소스 서버로 전송되는 HTTP 요청 수
.10.10. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalAverage                  Integer    원본 서버가 보낸 전체 평균 HTTP 응답 수
.10.11. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteAverage          Integer    원본 서버에서 성공적으로 평균 HTTP 트랜잭션 수
.10.12. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeRes                  Integer    원래 서버에서 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.13. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeComplete             Integer    원본 서버에서 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.14. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCount                    Integer    원본 서버가 보낸 전체 HTTP 응답 수
.10.15. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteCount            Integer    원본 서버에서 성공적으로 HTTP 트랜잭션 수
.10.20. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxAverage                    Integer    원본 서버가 보낸 평균 2xx 응답 수
.10.21. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteAverage            Integer    원본 서버에서 성공적으로 평균 2xx 트랜잭션 수
.10.22. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeRes                    Integer    원본 서버에서 2xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.23. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeComplete               Integer    원본 서버에서 2xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.24. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCount                      Integer    원본 서버가 보낸 2xx 응답 수
.10.25. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteCount              Integer    원본 서버에서 성공한 2xx 트랜잭션 수
.10.30. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxAverage                    Integer    원본 서버가 보낸 평균 3xx 응답 수
.10.31. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteAverage            Integer    원본 서버에서 성공적으로 평균 3xx 트랜잭션 수
.10.32. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeRes                    Integer    원본 서버에서 3xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.33. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeComplete               Integer    원본 서버에서 3xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.34. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCount                      Integer    원본 서버가 보낸 3xx 응답 수
.10.35. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteCount              Integer    원본 서버에서 성공한 3xx 트랜잭션 수
.10.40. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxAverage                    Integer    원본 서버가 보낸 평균 4xx 응답 수
.10.41. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteAverage            Integer    원본 서버에서 성공적으로 평균 4xx 트랜잭션 수
.10.42. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeRes                    Integer    원본 서버에서 4xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.43. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeComplete               Integer    원본 서버에서 4xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.44. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCount                      Integer    원본 서버가 보낸 4xx 응답 수
.10.45. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteCount              Integer    원본 서버에서 성공한 4xx 트랜잭션 수
.10.50. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxAverage                    Integer    원본 서버가 보낸 평균 5xx 응답 수
.10.51. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteAverage            Integer    원본 서버에서 성공적으로 평균 5xx 트랜잭션 수
.10.52. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeRes                    Integer    원본 서버에서 5xx 응답 헤더를 수신 할 때까지의 평균 소요 시간 (0.01ms)
.10.53. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeComplete               Integer    원본 서버에서 5xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.54. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCount                      Integer    원본 서버가 보낸 5xx 응답 수
.10.55. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteCount              Integer    원본 서버에서 성공한 5xx 트랜잭션 수
.10.60. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTimeoutAverage            Integer    평균 소스 서버 연결 실패 횟수
.10.61. ``[vhostMin]`` . ``[vhostIndex]``     http.receiveTimeoutAverage            Integer    평균 원본 서버 전송에 실패한 횟수
.10.62. ``[vhostMin]`` . ``[vhostIndex]``     http.connectAverage                   Integer    평균 소스 서버 접속 성공 횟수
.10.63. ``[vhostMin]`` . ``[vhostIndex]``     http.dnsQueryTime                     Integer    소스 서버 연결시 평균 DNS 쿼리의 소요 시간
.10.64. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTime                      Integer    소스 서버의 평균 접속 시간 (0.01ms)
.10.65. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTimeoutCount              Integer    소스 서버 연결 실패 횟수
.10.66. ``[vhostMin]`` . ``[vhostIndex]``     http.receiveTimeoutCount              Integer    소스 서버의 전송에 실패한 횟수
.10.67. ``[vhostMin]`` . ``[vhostIndex]``     http.connectCount                     Integer    소스 서버 접속 성공 횟수
.10.68. ``[vhostMin]`` . ``[vhostIndex]``     http.closeAverage                     Integer    전송 중에 소스 서버에서 먼저 소켓을 종료 한 평균 횟수
.10.69. ``[vhostMin]`` . ``[vhostIndex]``     http.closeCount                       Integer    전송 중에 소스 서버에서 먼저 소켓을 종료 한 횟수
.11                                           portbypass                            OID        Port 우회 원본 서버의 트래픽 정보
.11.1. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.inbound                    Integer    Port 우회를 통해 원본 서버에서받는 평균 트래픽 (Bytes)
.11.2. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.outbound                   Integer    Port 우회를 통해 원래의 서버로 전송하는 평균 트래픽 (Bytes)
.11.3. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.sessionAverage             Integer    Port 우회 동안 평균 원본 서버의 세션 수
.11.4. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedAverage              Integer    Port 우회중인 원본 서버가 연결을 종료 한 평균 횟수
.11.5. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.connectTimeoutAverage      Integer    Port 우회하는 서버의 평균 연결 실패 횟수
.11.6. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedCount                Integer    Port 우회중인 원본 서버가 연결을 종료 한 횟수
.11.7. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.connectTimeoutCount        Integer    Port 우회하는 서버 연결 실패 횟수
============================================= ===================================== ========== =================================================================



.. _snmp-cache-vhost-traffic-client:

cache.vhost.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.11

클라이언트 트래픽 통계를 제공한다. 클라이언트의 트래픽은 HTTP 트래픽은 SSL 트래픽, Port 우회 트래픽으로 구분된다. SNMP는 디렉토리 별 통계를 제공하지 않는다. 비록 디렉토리 통계가 설정되어있다하더라도 합산되어 있습니다.

============================================= ========================================= ========== ==============================================================
OID                                           Name                                      Type       Description
============================================= ========================================= ========== ==============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                   Integer    클라이언트에서받은 평균 트래픽 (Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                  Integer    클라이언트로 전송 평균 트래픽 (Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         sessionAverage                            Integer    전체 평균 클라이언트 세션 수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activesessionAverage                      Integer    모든 클라이언트에 전송되는 평균 방문수 균 세션 수
.10                                           http                                      OID        클라이언트의 HTTP 트래픽 정보
.10.1. ``[vhostMin]`` . ``[vhostIndex]``      http.inbound                              Integer    클라이언트에서받은 평균 HTTP 트래픽 (Bytes)
.10.2. ``[vhostMin]`` . ``[vhostIndex]``      http.outbound                             Integer    클라이언트로 전송 평균 HTTP 트래픽 (Bytes)
.10.3. ``[vhostMin]`` . ``[vhostIndex]``      http.sessionAverage                       Integer    클라이언트의 평균 HTTP 세션 수
.10.4. ``[vhostMin]`` . ``[vhostIndex]``      http.reqHeaderSize                        Integer    클라이언트에서받은 평균 HTTP Header 트래픽 (Bytes)
.10.5. ``[vhostMin]`` . ``[vhostIndex]``      http.reqBodySize                          Integer    클라이언트에서받은 평균 HTTP Body 트래픽 (Bytes)
.10.6. ``[vhostMin]`` . ``[vhostIndex]``      http.resHeaderSize                        Integer    클라이언트로 전송 평균 HTTP Header 트래픽 (Bytes)
.10.7. ``[vhostMin]`` . ``[vhostIndex]``      http.resBodySize                          Integer    클라이언트로 전송 평균 HTTP Body 트래픽 (Bytes)
.10.8. ``[vhostMin]`` . ``[vhostIndex]``      http.reqAverage                           Integer    클라이언트에서받은 평균 HTTP 요청 수
.10.9. ``[vhostMin]`` . ``[vhostIndex]``      http.reqCount                             Integer    클라이언트로부터 수신 한 HTTP 요청 수
.10.10. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalAverage                      Integer    클라이언트로 전송 평균 전체 응답 수
.10.11. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteAverage              Integer    클라이언트가 완료 한 평균 HTTP 트랜잭션 수
.10.12. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeRes                      Integer    클라이언트의 응답의 평균 소요 시간 (0.01ms)
.10.13. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeComplete                 Integer    클라이언트 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.14. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCount                        Integer    클라이언트에 보낼 전체 응답 수
.10.15. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteCount                Integer    클라이언트가 완료된 HTTP 트랜잭션 수
.10.20. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxAverage                        Integer    클라이언트로 전송 평균 2xx 응답 수
.10.21. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteAverage                Integer    클라이언트가 완료 한 평균 2xx 트랜잭션 수
.10.22. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeRes                        Integer    클라이언트 2xx 응답의 평균 소요 시간 (0.01ms)
.10.23. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeComplete                   Integer    클라이언트 2xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.24. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCount                          Integer    클라이언트로 전송 2xx 응답 수
.10.25. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteCount                  Integer    클라이언트가 완료된 2xx 트랜잭션 수
.10.30. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxAverage                        Integer    클라이언트로 전송 평균 3xx 응답 수
.10.31. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteAverage                Integer    클라이언트가 완료 한 평균 3xx 트랜잭션 수
.10.32. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeRes                        Integer    클라이언트 3xx 응답의 평균 소요 시간 (0.01ms)
.10.33. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeComplete                   Integer    클라이언트 3xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.34. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCount                          Integer    클라이언트로 전송 3xx 응답 수
.10.35. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteCount                  Integer    클라이언트가 완료된 3xx 트랜잭션 수
.10.40. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxAverage                        Integer    클라이언트로 전송 평균 4xx 응답 수
.10.41. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteAverage                Integer    클라이언트가 완료 한 평균 4xx 트랜잭션 수
.10.42. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeRes                        Integer    클라이언트 4xx 응답의 평균 소요 시간 (0.01ms)
.10.43. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeComplete                   Integer    클라이언트 4xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.44. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCount                          Integer    클라이언트로 전송 4xx 응답 수
.10.45. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteCount                  Integer    클라이언트가 완료된 4xx 트랜잭션 수
.10.50. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxAverage                        Integer    클라이언트로 전송 평균 5xx 응답 수
.10.51. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteAverage                Integer    클라이언트가 완료 한 평균 5xx 트랜잭션 수
.10.52. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeRes                        Integer    클라이언트 5xx 응답의 평균 소요 시간 (0.01ms)
.10.53. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeComplete                   Integer    클라이언트 5xx 응답 HTTP Transaction 평균 완료 시간 (0.01ms)
.10.54. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCount                          Integer    클라이언트로 전송 5xx 응답 수
.10.55. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteCount                  Integer    클라이언트가 완료된 5xx 트랜잭션 수
.10.60. ``[vhostMin]`` . ``[vhostIndex]``     http.reqDeniedAverage                     Integer    차단 된 요청의 평균
.10.61. ``[vhostMin]`` . ``[vhostIndex]``     http.reqDeniedCount                       Integer    차단 된 요청 수
.11                                           portbypass                                OID        Port 우회 클라이언트의 트래픽 정보
.11.1. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.inbound                        Integer    Port 우회를 통해 클라이언트로부터받는 평균 트래픽 (Bytes)
.11.2. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.outbound                       Integer    Port 우회를 통해 클라이언트로 전송 평균 트래픽 (Bytes)
.11.3. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.sessionAverage                 Integer    Port 우회하는 클라이언트의 평균 방문수
.11.4. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedAverage                  Integer    Port 우회중인 클라이언트가 연결을 종료 한 평균 횟수
.11.5. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedCount                    Integer    Port 우회중인 클라이언트가 연결을 종료 한 횟수
.12                                           ssl                                       OID        SSL 클라이언트의 트래픽 정보
.12.2. ``[vhostMin]`` . ``[vhostIndex]``      ssl.inbound                               Integer    SSL을 통해 클라이언트로부터받는 평균 트래픽 (Bytes)
.12.3. ``[vhostMin]`` . ``[vhostIndex]``      ssl.outbound                              Integer    SSL을 통해 클라이언트로 전송 평균 트래픽 (Bytes)
.13                                           requestHitAverage                         OID        평균 캐시 HIT 결과
.13.1. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_HIT                 Integer    TCP_HIT
.13.2. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_IMS_HIT             Integer    TCP_IMS_HIT
.13.3. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REFRESH_HIT         Integer    TCP_REFRESH_HIT
.13.4. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REF_FAIL_HIT        Integer    TCP_REF_FAIL_HIT
.13.5. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_NEGATIVE_HIT        Integer    TCP_NEGATIVE_HIT
.13.6. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_MISS                Integer    TCP_MISS
.13.7. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REFRESH_MISS        Integer    TCP_REFRESH_MISS
.13.8. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_CLIENT_REFRESH_MISS Integer    TCP_CLIENT_REFRESH_MISS
.13.9. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_DENIED              Integer    TCP_DENIED
.13.10. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_ERROR               Integer    TCP_ERROR
.13.11. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REDIRECT_HIT        Integer    TCP_REDIRECT_HIT
.14                                           requestHitCount                           OID        캐시 HIT 결과 수
.14.1. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_HIT                   Integer    TCP_HIT
.14.2. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_IMS_HIT               Integer    TCP_IMS_HIT
.14.3. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REFRESH_HIT           Integer    TCP_REFRESH_HIT
.14.4. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REF_FAIL_HIT          Integer    TCP_REF_FAIL_HIT
.14.5. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_NEGATIVE_HIT          Integer    TCP_NEGATIVE_HIT
.14.6. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_MISS                  Integer    TCP_MISS
.14.7. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REFRESH_MISS          Integer    TCP_REFRESH_MISS
.14.8. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_CLIENT_REFRESH_MISS   Integer    TCP_CLIENT_REFRESH_MISS
.14.9. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_DENIED                Integer    TCP_DENIED
.14.10. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_ERROR                 Integer    TCP_ERROR
.14.11. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REDIRECT_HIT          Integer    TCP_REDIRECT_HIT
============================================= ========================================= ========== ==============================================================



.. _snmp-cache-vhost-traffic-filesystem:

cache.vhost.traffic.filesystem
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.20

가상 호스트의 File I / O 통계를 제공한다.

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitRatio                             Integer    Request Hit Ratio(100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                                              Request Hit Ratio(10000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``       byteHitRatio                                Integer    Byte Hit Ratio(100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                                              Byte Hit Ratio(10000%)
.5. ``[vhostMin]`` . ``[vhostIndex]``       outbound                                    Integer    File I / O로 보내는 평균 트래픽 (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       session                                     Integer    File I / O를 수행중인 평균 Thread 수
.7                                          requestHitAverage                           OID        평균 캐시 HIT 결과
.7.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_HIT                   Integer    TCP_HIT
.7.2. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_IMS_HIT               Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REFRESH_HIT           Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REF_FAIL_HIT          Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_NEGATIVE_HIT          Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_MISS                  Integer    TCP_MISS
.7.7. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REFRESH_MISS          Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_CLIENT_REFRESH_MISS   Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_DENIED                Integer    TCP_DENIED
.7.10. ``[vhostMin]`` . ``[vhostIndex]``    requestHitAverage.TCP_ERROR                 Integer    TCP_ERROR
.7.11. ``[vhostMin]`` . ``[vhostIndex]``    requestHitAverage.TCP_REDIRECT_HIT          Integer    TCP_REDIRECT_HIT
.8                                          requestHitCount                             OID        캐시 HIT 결과 수
.8.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_HIT                     Integer    TCP_HIT
.8.2. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_IMS_HIT                 Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REFRESH_HIT             Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REF_FAIL_HIT            Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_NEGATIVE_HIT            Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_MISS                    Integer    TCP_MISS
.8.7. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REFRESH_MISS            Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_CLIENT_REFRESH_MISS     Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_DENIED                  Integer    TCP_DENIED
.8.10. ``[vhostMin]`` . ``[vhostIndex]``    requestHitCount.TCP_ERROR                   Integer    TCP_ERROR
.8.11. ``[vhostMin]`` . ``[vhostIndex]``    requestHitCount.TCP_REDIRECT_HIT            Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]`` . ``[vhostIndex]``      getattr.filecount                           Integer    (getattr 함수 호출) FILE로 응답 한 횟수
.11. ``[vhostMin]`` . ``[vhostIndex]``      getattr.dircount                            Integer    (getattr 함수 호출) DIR에 응답 한 횟수
.12. ``[vhostMin]`` . ``[vhostIndex]``      getattr.failcount                           Integer    (getattr 함수 호출) 실패에 응답 한 횟수
.13. ``[vhostMin]`` . ``[vhostIndex]``      getattr.timeres                             Integer    (getattr 함수의 호출)의 반응 시간 (0.01ms)
.14. ``[vhostMin]`` . ``[vhostIndex]``      open.count                                  Integer    open 함수의 호출 횟수
.15. ``[vhostMin]`` . ``[vhostIndex]``      open.timeres                                Integer    open 함수의 반응 시간 (0.01ms)
.16. ``[vhostMin]`` . ``[vhostIndex]``      read.count                                  Integer    read 함수 호출 횟수
.17. ``[vhostMin]`` . ``[vhostIndex]``      read.timeres                                Integer    read 함수의 반응 시간 (0.01ms)
.18. ``[vhostMin]`` . ``[vhostIndex]``      read.buffersize                             Integer    read 함수에서 요구 된 버퍼 크기 (Bytes)
.19. ``[vhostMin]`` . ``[vhostIndex]``      read.bufferfilled                           Integer    read 함수에서 요구 된 버퍼에 포장 크기 (Bytes)
=========================================== =========================================== ========== ==============================================



.. _snmp-cache-vhost-traffic-dims:

cache.vhost.traffic.dims
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.21

가상 호스트의 DIMS 변환 통계를 제공한다.

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requests                                    Integer    DIMS 변환 요청 수
.2. ``[vhostMin]`` . ``[vhostIndex]``       converted                                   Integer    변환에 성공 횟수
.3. ``[vhostMin]`` . ``[vhostIndex]``       failed                                      Integer    변환에 실패한 횟수
.4. ``[vhostMin]`` . ``[vhostIndex]``       avgsrcsize                                  Integer    원본 이미지의 평균 크기 (Bytes)
.5. ``[vhostMin]`` . ``[vhostIndex]``       avgdestsize                                 Integer    변환 된 이미지의 평균 크기 (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       avgtime                                     Integer    변환 시간 (ms)
=========================================== =========================================== ========== ==============================================



.. _snmp-cache-vhost-traffic-compression:

cache.vhost.traffic.compression
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.22

가상 호스트의 압축 통계를 제공한다.

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requests                                    Integer    압축 요청 수
.2. ``[vhostMin]`` . ``[vhostIndex]``       converted                                   Integer    압축 성공 횟수
.3. ``[vhostMin]`` . ``[vhostIndex]``       failed                                      Integer    압축 실패 횟수
.4. ``[vhostMin]`` . ``[vhostIndex]``       avgsrcsize                                  Integer    원본 파일의 평균 크기 (Bytes)
.5. ``[vhostMin]`` . ``[vhostIndex]``       avgdestsize                                 Integer    압축 된 파일의 평균 크기 (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       avgtime                                     Integer    압축 시간 (ms)
=========================================== =========================================== ========== ==============================================



.. _snmp-cache-view:

cache.view
====================================

::

   OID = 1.3.6.1.4.1.40001.1.4.11.1

가상 호스트의 통계와 같은 정보를 제공한다.
``[viewIndex]`` 1에서 View 개수의 범위를 가진다.

- 1.3.6.1.4.1.40001.1.4.3 - 가상 호스트의 통계

- 1.3.6.1.4.1.40001.1.4.11 - View 통계
