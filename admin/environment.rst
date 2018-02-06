.. _env:

제 3 장 구성 구조
******************

.. note::

   - `[동영상 강좌]하자! STON Edge Server - Chapter 2. 설정하기 <https://youtu.be/BROcSuFyHOQ?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_
   - `[동영상 강좌]하자! STON Edge Server - Chapter 3. 가상 호스트 작성 <https://youtu.be/AvxxSWgXcqA?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

이 장에서는 설정의 구조와 변경된 설정을 적용하는 방법을 설명한다. 구조를 정확히 이해할 필요가 신속하게 서버를 배치 할 수있을뿐만 아니라 장애 상황을 유연하게 극복 할 수있다.

설정은 크게 전역 (server.xml)와 가상 호스트 (vhosts.xml)로 나뉜다.

   .. figure:: img/conf_files.png
      :align: center

      두 .xml 파일이 있습니다.

두 개의 XML 파일에서 대부분의 서비스를 구성한다. 여러 TXT 파일은 가상 호스트 고유의 예외 조건을 설정하지만, 특정 기능의 목록을 만드는 데 사용된다. 기능 설명을 위해 다음과 같이 완전한 형태의 XML을 예시 할 수하다고 할 수있다. ::

   <Server>
       <VHostDefault>
           <Options>
               <CaseSensitive>ON</CaseSensitive>
           </Options>
       </VHostDefault>
   </Server>

때문에 다음과 같이 생략하여 설명한다. ::

   # server.xml - <Server><VHostDefault><Options>

   <CaseSensitive>ON</CaseSensitive>


.. note::

   라이센스 (license.xml)는 설정이 없습니다.


.. _api-conf-reload:

설정 Reload
====================================

설정 변경 후 관리자가 명확하게 API를 호출 할 필요가 있습니다. 시스템 성능 관련 설정을 제외한 대부분의 설정은 서비스 중단없이 즉시 적용된다. ::

   http://127.0.0.1:10040/conf/reload

설정이 변경 될 때마다 :ref:`admin-log-info` 에 변경 사항이 기록된다.


server.xml 전역 설정
====================================

실행 파일과 같은 경로에있는 server.xml이 전역 설정 파일이다. XML 형식의 텍스트 파일이다. ::

    # server.xml

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
    </Server>

먼저 전역 설정의 구조와 간단한 기능을 중심으로 설명한다.
:ref:`access-control` 및 :ref:`snmp` 등의 전역 설정에 위치 덩치가 큰 기능은 각 주제를 다루는 장에서 설명한다.

.. toctree::
   :maxdepth: 2


.. _env-host:

관리자 설정
------------------------------------

관리 목적의 기능을 설정합니다. ::

    # server.xml - <Server>

    <Host>
        <Name>stream_07</Name>
        <Admin>admin@example.com</Admin>
        <Manager Port="10040" HttpMethod="ON" Role="Admin" UploadMultipartName="confile">
            <Allow>192.168.1.1</Allow>
            <Allow Role="Admin">192.168.2.1-255</Allow>
            <Allow Role="User">192.168.3.0/24</Allow>
            <Allow Role="Looker">192.168.4.0/255.255.255.0</Allow>
        </Manager>
    </Host>

-  ``<Name>``
    서버 이름을 설정한다. 이름이 입력되어 있지 않은 경우, 시스템 이름을 사용한다.

-  ``<Admin>``
    관리자 정보 (이메일과 이름)을 설정한다. 이 항목은 SNMP 조회 목적으로 만 사용된다.

-  ``<Manager>``
    관리 목적으로 사용 관리자 포트 및 ACL (Access Control List)을 설정한다. ACL은 IP, IP 주소 범위, BitMask, Subnet 이상 네 가지 형식을 지원합니다. 연결 한 세션이 Allow로 액세스가 허용 된 IP 주소가없는 경우 연결을 차단한다. API를 호출 IP가 ``<Allow>`` 목록에 반드시 설정할 필요가있다.

    액세스 조건에 따라 액세스 권한 (Role)을 설정할 수있다. 권한이없는 요구에 대해서는、 **401 Unauthorized** 로 응답한다.
    ``<Allow>`` 의 조건에 ``Role`` 속성을 명시 적으로 선언하지 않은 경우 ``<Manager>`` 의 ``Role`` 속성이 적용된다.

    - ``Admin`` 모든 API 호출이 가능하다.
    - ``User`` api_monitoring、 :ref:`api-graph` API만을 호출 할 수있다.
    - ``Looker`` :ref:`api-graph` API만을 호출 할 수있다.

    기타 다음과 같은 세세한 관리 목적의 특성을 가진다.

    - ``HttpMethod``

      - ``ON (기본)`` api-etc-httpmethod 호출시 ACL을 검사한다.

      - ``OFF`` api-etc-httpmethod 호출시 ACL을 검사하지 않습니다.

    - ``UploadMultipartName`` :ref:`api-conf-upload` 변수 이름을 설정한다.


.. _env-cache-storage:

Storage 구성
------------------------------------

Caching 된 콘텐츠를 저장할 Storage를 구성한다. ::

    # server.xml - <Server>

    <Cache>
        <Storage DiskFailSec="60" DiskFailCount="10" OnCrash="hang">
            <Disk>/user/cache1</Disk>
            <Disk>/user/cache2</Disk>
            <Disk Quota="100">/user/cache3</Disk>
        </Storage>
    </Cache>

-  ``<Storage>``
    콘텐츠를 저장하는 디스크를 설정한다. 서브 ``<Disk>`` 수의 제한은 없다.

    디스크 오류가 가장 많이 발생하는 기기이기 때문에 명확한 장애 조건을 설정하는 것이 좋습니다.。
    ``DiskFailSec (기본: 60초)`` 사이 ``DiskFailCount (기본: 10)`` 에만 디스크 작업이 실패 할 경우 해당 디스크는 자동으로 제거된다. 제거 된 디스크의 상태는 "Invalid"로 지정된다.

    모든 디스크가 제거 될 수 있으며,이 때의 동작은 ``OnCrash`` 속성에 설정한다.

    - ``hang (기본)`` 장애 디스크를 모두 재 투입한다. 정상적인 서비스를 기대하고있다 기보다는 원본을 보호하는 목적이 강하다.

    - ``bypass`` 모든 요청을 원래 서버로 우회한다. 디스크가 복구되면 즉시 STON이 서비스를 처리한다.

    - ``selfkill`` STON을 종료시킨다.

각 디스크마다 최대 캐시 용량을 ``Quota (단위: GB)`` 속성에 설정할 수있다. 굳이 설정하지 않아도 항상 디스크가 가득 있지 않도록 LRU (Least Recently Used) 알고리즘은 오래된 콘텐츠를 자동으로 삭제한다. 특히 호환성에 문제가있는 파일 시스템이 아니다. 따라서 관리자는 익숙한 파일 시스템을 사용하여도 성능에 큰 영향은 없다.

.. note::

   v2.5.0에서 디스크없이 작동 :ref:`adv_topics_memory_only` 가 지원된다.



.. _env-cache-resource:

메모리 제한
------------------------------------

사용 최대 메모리 및 콘텐츠 적재율을 설정한다. ::

    # server.xml - <Server>

    <Cache>
        <SystemMemoryRatio>100</SystemMemoryRatio>
        <ContentMemoryRatio>50</ContentMemoryRatio>
    </Cache>

-  ``<SystemMemoryRatio> (기본: 100%)``

   시스템 메모리에서 STON 사용 최대 메모리 비율로 설정한다. 예를 들어 16GB 장치에서는 숫자를 50 (%)로 설정하면 시스템 메모리가 8GB 인 것처럼 동작한다. 특히 :ref:`filesystem` 등을 통해 다른 프로세스와 연동 할 때 편리하다.

-  ``<ContentMemoryRatio> (기본: 50%)``

   STON 디스크에서로드 된 Body 데이터를 메모리에 최대한 Caching하고 서비스 품질을 향상시킨다. 서비스 형태에 따라이 비율을 조정하여 품질을 최적화한다.

      .. figure:: img/bodyratio1.png
         :align: center

         ContentMemoryRatio 통해 메모리의 비율을 설정한다.

   예를 들어, 게임의 다운로드와 같은 파일의 수는 많지 않지만 Contents 크기가 큰 서비스의 경우 File I / O로드가 부담 스럽다. 이런 경우 ``<ContentMemoryRatio>`` 을 높여보다 많은 Contents 데이터가 메모리에 상주하도록 설정하면 서비스의 품질을 향상시킬 수있다.

      .. figure:: img/bodyratio2.png
         :align: center

         ContentMemoryRatio을 높이면 I / O가 감소한다.



.. _env-etc:

기타 Caching 설정
------------------------------------

기타 Caching 서비스 기반 동작을 설정한다. ::

    # server.xml - <Server>

    <Cache>
        <Cleanup>
            <Time>02:00</Time>
            <Age>0</Age>
            <EmptyFolder>delete</EmptyFolder>
        </Cleanup>
        <Listen>0.0.0.0</Listen>
        <ConfigHistory>30</ConfigHistory>
    </Cache>

-  ``<Cleanup>``
    하루에 한 번 시스템의 최적화를 수행한다. 최적화의 대부분은 디스크 정리 작업에서 I / O 부하가 발생한다. 서비스 품질의 저하를 방지하기 위해 최적화 조금씩 단계적으로 실행된다.

    - ``<Time> (기본: AM 2)`` Cleanup 실행 시간을 설정한다. 오후 11시 10 분을 설정하려면 23:10으로 설정한다.

    - ``<Age> (기본: 0, 단위: 日)`` 0보다 큰 경우에는 일정 기간 한 번도 사용되지 않은 콘텐츠를 삭제한다. 디스크를 사전에 확보하고 서비스 시간 동안 디스크 부족이 발생할 확률을 줄이기위한 것이다.

    - ``<EmptyFolder> (기본: delete)`` Cleanup 시점에서 빈 폴더 (캐시 저장 용으로 사용)의 제거 여부를 결정한다.
      ``delete``  의 경우는 제거하고 ``keep`` 의 경우는 삭제하지 않습니다.

-  ``<Listen>``
    모든 가상 호스트가 Listen하는 IP 목록을 지정한다. 모든 가상 호스트의 기본 Listen 설정 * : 80은 0.0.0.0 : 80을 의미한다. 지정된 IP 만 열려면 다음과 같이 명확하게 설정한다. ::

       # server.xml - <Server>

       <Cache>
         <Listen>10.10.10.10</Listen>
         <Listen>10.10.10.11</Listen>
         <Listen>127.0.0.2</Listen>
       </Cache>

-  ``<ConfigHistory> (기본: 30일)``
    STON는 설정이 변경 될 때마다 모든 설정을 백업한다. 해동 후 ./conf/ 하나의 파일로 저장된다. 파일 이름은 "날짜 _ 시간 _HASH.tgz"에 생성된다. ::

       20130910_174843_D62CA26F16FE7C66F81D215D8C52266AB70AA5C8.tgz

    모든 설정이 완전히 같으면 같은 HASH 값을 가진다.
    :ref:`api-conf-restore` 이 호출되도록 새로운 설정으로 저장됩니다. 백업 된 설정은 Cleanup 시간을 기준으로 설정 한 날짜만큼 저장된다. 설정 파일의 저장 날의 제한은 없다.



강제 Cleanup
------------------------------------

API 호출에 Cleanup한다. ``<Age>`` 를 매개 변수로 입력 할 수있다. ::

   http://127.0.0.1:10040/command/cleanup
   http://127.0.0.1:10040/command/cleanup?age=10

``<Age>`` 가 0이면, 디스크 공간이 부족하다고 판단되는 경우에만 Cleanup을 실행한다.
``<Age>`` 매개 변수가 0보다 큰 경우, "일"동안 한번도 사용되지 않은 콘텐츠를 삭제한다.


.. _env-vhostdefault:

가상 호스트 설정
------------------------------------

관리자는 각각의 가상 호스트를 개별적으로 설정할 수있다. 그러나 가상 호스트를 만들 때마다 동일한 설정을 반복하는 것은 매우 소모이다. 모든 가상 호스트는 ``<VHostDefault>`` 을 상속합니다.

   .. figure:: img/vhostdefault.png
      :align: center

      단일 상속이다.

www.example.com의 경우는 별도 덮어 (Overriding) 값이 없기 때문에 A = 1, B = 2가된다. 한편, img.example.com는 B = 3으로 덮어 때문에 A = 1, B = 3이된다. 관리자는 일반적으로 동일한 서비스 특성을 가진 서비스를 서버에 구성한다. 따라서 상속은 매우 효과적인 방법이다.

``<VHostDefault>`` 는 기능별로 둘러싸인 5 개의 하위 태그를 가지고있다. ::

    # server.xml - <Server>

    <VHostDefault>
        <Options> ... </Options>
        <OriginOptions> ... </OriginOptions>
        <Media> ... </Media>
        <Stats> ... </Stats>
        <Log> ... </Log>
    </VHostDefault>

예를 들어 :ref:`media` 기능은 ``<Media>`` 하위에 설정하는 식이다.


.. _env-vhost:

vhosts.xml 가상 호스트 설정
====================================

실행 파일과 같은 경로에 존재하는 vhosts.xml 파일을 가상 호스트 설정 파일로 인식합니다. 가상 호스트의 수에는 제한이 없다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>


.. _env-vhost-create-destroy:

생성 / 파괴
------------------------------------

``<Vhosts>`` 서브 ``<Vhost>`` 가상 호스트를 설정한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` 가상 호스트를 설정한다.

   - ``Status (기본: Active)`` nactive 인 경우는 그 가상 호스트를 서비스하지 않습니다. 캐시 된 콘텐츠는 유지된다.
   - ``Name`` 가상 호스트 이름. 중복 될 수 없다.

``<Vhost>`` 을 삭제하면 해당 가상 호스트가 제거된다. 삭제 된 가상 호스트의 모든 콘텐츠는 삭제 대상이된다. 다시 추가해도 콘텐츠가 되살아나는 없다.


.. _env-vhost-find:

검색
------------------------------------

다음은 가장 간단한 형태의 HTTP 요청이다. ::

    GET / HTTP/1.1
    Host: www.example.com

일반적인 Web 서버는 Host 헤더에 가상 호스트를 찾는다. 하나의 가상 호스트를 여러 이름으로 서비스하려면  ``<Alias>`` 를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
        <Alias>another.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   가상 호스트 별명을 설정한다. 수는 제한이 없다. 명확한 표현 (another.com)과 패턴 표현 ( * .sub.example.com)를 지원한다. 패턴은 복잡한 정규식이 아니라 prefix에 * 표현을 하나만 넣을 수있는 간단한 형식만을 지원하고있다.

가상 호스트 검색 순서는 다음과 같다.

1. ``<Vhost>`` 의 ``Name`` 과 일치 하는가?
2. 명시적인 ``<Alias>`` 와 일치 하는가?
3. 패턴 ``<Alias>`` 을 만족 하는가?


.. _env-vhost-defaultvhost:

Default 가상 호스트
------------------------------------

요청을 처리하는 가상 호스트를 찾을 수없는 경우, 선택되는 가상 호스트를 지정할 수있다. 요청을 처리하고 싶지 않다면 설정하고있다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   기본 가상 호스트 이름을 설정한다. 반드시 ``<Vhost>`` 의 ``Name`` 속성과 동일한 문자열로 설정해야한다.


.. _env-vhost-listen:

서비스 주소
------------------------------------
서비스의 주소를 설정한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>

-  ``<Listen> (기본: *:80)``

   {IP} : {Port}의 형식으로 서비스의 주소를 설정한다. * : 80 표현은 모든 NIC에서 80 포트에서 요청을 처리한다는 뜻이다. 예를 들어, 특정 IP (1.1.1.1)의 90 포트로 서비스하고 싶다면 다음과 같이 설정한다. ::

       # vhosts.xml - <Vhosts>

       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>

.. note::

   서비스 포트를 열지 않으려면 ``OFF`` 로 설정한다. ::

      # vhosts.xml - <Vhosts>

      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>


.. _env-vhost-txt:

가상 호스트 - 예외 조건 (.txt)
---------------------------------------

서비스 다음과 같이 예외적 인 상황이 필요한 때가있다.

- 모든 POST 요청은 허용하지 않지만 특정 URL에 POST 요청은 허용한다.
- 모든 GET 요청은 STON가 응답하지만 특정 IP 대역은 원래 서버로 우회한다.
- 특정 국가에는 전송 속도를 제한한다.

이러한 예외 사항은 XML로 설정하고 있지 않다. 모든 가상 호스트는 별도의 예외 조건이있다. 예외 조건은 ./svc/ 가상 호스트 / 디렉토리 하위에 TXT 존재한다. 관련 기능에 대해 설명하면 예외 조건도 함께 다룬다.


가상 호스트의 목록을 확인
====================================

가상 호스트의 목록을 조회한다. ::

   http://127.0.0.1:10040/monitoring/vhostslist

결과는 JSON 형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ]
   }


.. _api-conf-show:

설정을 확인하기
====================================

서비스중인 설정 파일을 확인한다. txt 파일은 가상 호스트 (vhost)를 명확하게 지정해야한다. ::

    http://127.0.0.1:10040/conf/server.xml
    http://127.0.0.1:10040/conf/vhosts.xml
    http://127.0.0.1:10040/conf/querystring.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/bypass.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/ttl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/expires.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/acl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/headers.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/throttling.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/postbody.txt?vhost=www.example.com


.. _api-conf-history:

설정 목록
====================================

백업 된 설정의 목록을 볼 수 있습니다. ::

    http://127.0.0.1:10040/conf/latest
    http://127.0.0.1:10040/conf/history

결과는 JSON 형식으로 제공된다. 고속 마지막 설정 상태 만 확인하려면 / conf / latest을 사용하는 것을 권장한다. ::

    {
        "history" :
        [
            {
                "id" : "5",
                "conf-date" : "2013-11-06",
                "conf-time" : "15:26:37",
                "type" : "loaded",
                "size" : "16368",
                "hash" : "D62CA26F16FE7C66F81D215D8C52266AB70AA5C8",
                "ver": "1.2.8"
            },
            {
                "id" : "6",
                "conf-date" : "2013-11-07",
                "conf-time" : "07:02:21",
                "type" : "modified",
                "size" : "27544",
                "hash" : "F81D215D8C52266AB70AA5C8D62CA26F16FE7C66",
                "ver": "1.2.8"
            }
        ]
    }

-  ``id`` 설정의 고유 ID (Reload 때마다 +1)
-  ``conf-date`` 설정 변경 날짜
-  ``conf-time`` 설정을 변경 시간
-  ``type`` 의 설정이 반영 된 형태
   - ``loaded`` STON가 시작된다
   - ``modified`` 설정 (관리자 또는 WM 의해) 변경된 때
   - ``uploaded`` 설정 파일 API를 통해 업로드 된 때
   - ``restored`` 설정 파일이 API를 통해て 회복 된 때
-  ``size`` 설정 파일 크기
-  ``hash`` 설정 파일을 SHA-1에서 hash 값


.. _api-conf-restore:

설정 복원
====================================

hash 값 또는 id에 따라 언제든지 설정으로 되 돌린다. hash와 id가 모두 포함되어있는 경우 hash 값이 우선된다. 제대로 Rollback 된 경우 200 OK 실패한 경우 500 Internal Error로 응답한다. ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _api-conf-download:

설정 다운로드
====================================

hash 값 또는 id에 따라 언제든지 설정을 다운로드한다. Content-Type은 "application / x-compressed"로 지정된다. hash와 id가 모두 포함되어있는 경우 hash 값이 우선 당시의 설정이없는 경우, 404 NOT FOUND에 응답한다. ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...



.. _api-conf-upload:

설정 업로드
====================================

API를 이용하여 설정을 변경합니다. 제대로 반영하면 200 OK 응답하지만, 실패하면 500 Internal Server Error로 응답한다. ::

   {
      "version": "2.5.10",
      "method": "uploadconfig",
      "status": "Fail",
      "result": "E0001"
    }

"result"오류 코드는 다음과 같다.

======= ===============================
result  설명
======= ===============================
E0000   설정을 적용 완료
E0001   업로드 한 파일이 존재하지 않는다.
E0002   tgz 파일을 압축 해제에 실패했습니다.
E0003   업로드 된 xml 파일이로드되지 않는다.
E0004   업로드 된 xml 파일의 내용이 잘못되었습니다.
======= ===============================


.. _api-conf-upload-tgz:

전체 설정 업로드
------------------------------------

전체 설정 압축 파일을 HTTP Post 방식 (Multipart 지원)에 업로드한다. ::

    http://127.0.0.1:10040/conf/upload

다음과 같이 주소, Content-Length, Content-Type (= "multipart / form-data")가 명확하게 선언되어 있어야한다. ::

    POST /conf/upload
    Content-Length: 16455
    Content-Type: multipart/form-data; boundary=......

업로드가 완료되면 압축을 해제 한 후 전체 설정을 업데이트한다.

Multipart 방식은 "confile"을 기본 이름으로 사용한다. 이 값은 ``<Manager>`` 의 ``UploadMultipartName`` 속성에서 설정할 수있다. ::


    <form enctype="multipart/form-data" action="http://127.0.0.1:10040/conf/upload" method="POST">
        <input name="confile" type="file" />
        <input type="submit" value="Upload" />
    </form>


.. _api-conf-upload-xml:

XML 구성 업로드
------------------------------------

XML 개별 설정 압축 파일을 HTTP Post 방식 (Multipart과 SOAP 방식을 모두 지원)에 업로드한다. ::

    http://127.0.0.1:10040/conf/upload/server.xml
    http://127.0.0.1:10040/conf/upload/vhosts.xml


.. note::
   
   server.xml을 업로드 할 경우 전체 설정을 업데이트하지만 vhosts.xml 만 업로드 할 경우 가상 호스트에만 설정을 업데이트한다.

