.. _origin:

제 7 장 소스 서버
******************

이 장에서는 STON와 원래 서버의 관계를 설명한다. 소스 서버는 일반적으로 HTTP 사양에 준거하고있는 Web 서버를 의미한다. 관리자라면 원고를 보호하기 위해 이번 장의 모든 내용을 숙지 할 필요가있다. 이를 바탕으로 원래의 장애에도 내구성을 갖춘 유연한 서비스를 구축 할 수있다.

소스 서버는 보호되어야한다. 장애의 종류가 다양한만큼 대처 방안도 다양하다. 소스 보호 정책을 적절하게 설정하면 느긋한 점검 시간을 가질 수 있습니다.


.. toctree::
   :maxdepth: 2



.. _origin_exclusion_and_recovery:

장애 감지 및 복구
====================================

Caching 과정에서 원래의 서버에 장애가 발생하면 자동으로 제거합니다. 다시 안정화되었다고 판단하면 서비스에 투입한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ConnectTimeout>3</ConnectTimeout>
   <ReceiveTimeout>10</ReceiveTimeout>
   <Exclusion>3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>

-  ``<ConnectTimeout> (기본: 3초)``

   n 초 내에 원래 서버와의 연결이 이루어지고 있지 않은 경우, 연결에 실패 본다.

-  ``<ReceiveTimeout> (기본: 10초)``

   일반적으로 HTTP 요청에도 불구하고 원래 서버가 HTTP 응답을 n 초 동안 전송하지 않으면 전송이 실패로 간주하고있다.

-  ``<Exclusion> (기본: 3회)``

   원본 서버에서 연속적으로 n 회 장애 상황( ``<ConnectTimeout>`` 또는 ``<ReceiveTimeout>`` )가 발생했을 경우, 그 서버를 사용 소스 서버 목록에서 제거합니다. 제거 전, 통상의 통신이 행해진 경우,이 값은 0으로 초기화된다.

-  ``<Recovery> (기본: 5회)``

   ``Cycle`` 마다 ``Uri`` 에 요청하여 원본 서버가 ``ResCode`` 에 연속적으로 n 회 응답하면 해당 서버를 복구한다. 이 값을 0으로 설정하면 회복되지 않는다.

   -  ``Cycle (기본: 10초)`` 일정 시간 (초)마다하려고한다.

   -  ``Uri (기본: /)`` 요청을 보낼 Uri

   -  ``ResCode (기본: 0)`` 정상 응답으로 처리하는 응답 코드. 0의 경우, 응답 코드에 관계없이 응답이 오면 성공으로 본다. 200으로 설정하면 응답 코드가 반드시 200 아니면 그렇다면 제대로 응답으로 처리한다. 쉼표 (,)를 사용하여 유효한 응답 코드를 멀티로 설정한다. 200,206,404으로 설정하면 응답 코드가이 중 하나 인 경우 일반적으로 응답에서 처리한다.

   -  ``Log (기본: ON)`` 회복을 위해 사용 된 HTTP Transaction을 :ref:`admin-log-origin` 에 기록한다.



.. _origin-health-checker:

Health-Checker
====================================

`장애 감지 및 복구`_ 는 Caching 과정 중에 발생하는 장애에 대응한다.
``<Recovery>`` 는 응답 코드가 접수되면 HTTP Transaction을 종료한다. 그러나 Health-Checker는 HTTP Transaction이 성공하는지 확인한다. ::

   # vhosts.xml - <Vhosts><Vhost>

   <Origin>
      <Address> ... </Address>
      <HealthChecker ResCode="0" Timeout="10" Cycle="10"
                     Exclusion="3" Recovery="5" Log="ON">/</HealthChecker>
      <HealthChecker ResCode="200, 404" Timeout="3" Cycle="5"
                     Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker>
   </Origin>

-  ``<HealthChecker> (기본: /)``

   Health-Checker를 구성한다. 멀티로 구성이 가능하다. 값으로 Uri를 지정하고 XML 예외 문자의 경우 CDATA를 사용한다.

   -  ``ResCode (기본: 0)`` 올바른 응답 코드 (쉼표로 다중 구성 가능)

   -  ``Timeout (기본: 10초)`` 소켓 연결에서 HTTP Transaction이 완료 될 때까지 유효 시간

   -  ``Cycle (기본: 10초)`` 실행주기

   -  ``Exclusion (기본: 3회)`` 연속 n 회 실패하면 해당 서버 제거

   -  ``Recovery (기본: 5회)`` 연속 n 회 성공하면 해당 서버의 재 투입

   -  ``Log (기본: ON)`` HTTP Transaction을 :ref:`admin-log-origin` 에 기록한다.

Health-Checker는 멀티로 구성 할 수 있으며, 고객의 요구에 관계없이 독립적으로 실행된다.
`장애 감지 및 복구`_ 와 다른 Health-Checker와도 정보를 공유하지 않고 자신 만의 기준에서 배제 및 책임을 결정한다.


.. _origin-use-policy:

보낸 사람 주소를 사용 정책
====================================

원본 주소 (IP)는 다음의 요소에 의해 어떻게 사용되는지 결정된다.

-  :ref:`env-vhost-activeorigin` 주소 형식 (IP 주소 또는 Domain)과 보조 주소
-  `障害の検出と回復`_
-  `Health-Checker`_

서비스를 운영하고 원래의 주소가 제거 / 회복되는 것은 빈번하다. STON는 IP 테이블을 기반으로 원래 주소를 사용하여 `origin-status`_ API를 통해 정보를 제공한다.

보낸 사람 주소를 IP로 설정하면 매우 간단하다.

-  설정의 변경뿐만 아니라, IP 목록을 변화시키는 요인은 아니다.
-  TTL은 IP 주소가 만료되지 않는다.
-  장애 / 복구를 모두 설정 (IP 주소)에 따라 동작한다.

보낸 사람 주소를 Domain으로 설정하면 Resolving て IP 주소를 얻어야한다. 
( :ref:`admin-log-dns` 에 기록된다.) IP 목록은 동적으로 변경 될 수 있으며, 모든 IP는 TTL (Time To Live) 동안 만 유효하다.

-  Domain은 정기적으로 (1 ~ 10 초) Resolving한다.
-  Resolving을 통해 사용하는 IP 테이블을 구성한다.
-  모든 IP는 TTL만큼 효과적이며, TTL이 만료되면 사용하지 않는다.
-  동일한 IP 주소가 다시 Resolving되면 TTL을 업데이트한다.
-  IP 테이블은 비어 있지. (TTL이 만료되고도) 마지막 IP는 삭제되지 않는다.

IP의 TTL이 너무 긴 경우 지나치게 많은 IP 주소를 사용하도록되어 의도하지 않은 결과를 만들 수있다. 이를 방지하기 위해 IP의 최대 TTL을 제한 할 수있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <DnsMaxTTL>60</DnsMaxTTL>

-  ``<DnsMaxTTL> (기본: 60초)`` Resolving 된 IP 주소의 최대 사용 시간 (초)을 설정한다. 이 값이 0의 경우 DNS에서 제공 한 TTL을 그대로 사용한다.


.. note::

   보낸 사람 주소를 Domain에 설정해도 장애 / 복구는 IP 기반에서 동작한다. Domain 주소 장애 / 복구 정책은 다음과 같다.

   -  (Domain에 대해) 아는 모든 IP 주소가 배제 (Inactive)과 그 Domain 주소가 배제된다.
   -  신규 IP가 Resolving도 Domain이 배제되는 경우는 IP 주소는 처음부터 배제된다.
   -  모든 IP 주소는 TTL이 만료도 배제 된 Domain 상태는 풀리지 않는다.
   -  배제 된 Domain에 속한 IP 주소 중 하나가 회복되어야한다는 Domain은 다시 활성화된다.

   다소 복잡한 내용이기 때문에 `origin-status`_ API를 사용하여 서비스의 작동 상태에 대한 이해를 높이는 것이 좋다.



.. _origin-status:

원래 상태 모니터링
====================================

API를 통해 가상 호스트 소스의 상태를 모니터링합니다. ::

   http://127.0.0.1:10040/monitoring/origin       // 모든 가상 호스트
   http://127.0.0.1:10040/monitoring/origin?vhost=www.example.com

결과는 JSON 형식으로 제공된다. ::

   {
       "origin" :
       [
           {
               "VirtualHost" : "example.com",
               "Address" :
               [
                   { "1.1.1.1" : "Active" },
                   { "1.1.1.2" : "Active" }
               ],
               "Address2" : [  ],
               "ActiveIP" :
               [
                   { "1.1.1.1" : 0 },
                   { "1.1.1.2" : 0 }
               ] ,
               "InactiveIP" : [ ]
           },
           {
               "VirtualHost" : "foobar.com",
               "Address" :
               [
                   { "origin.foobar.com" : "Active" }
               ],
               "Address2" : [  ],
               "ActiveIP" :
               [
                   { "5.5.5.5" : 21 },
                   { "5.5.5.6" : 60 },
                   { "5.5.5.7" : 37 }
               ],
               "InactiveIP" :
               [
                   { "5.5.5.8" : 10 },
                   { "5.5.5.9" : 184 }
               ]
           }
       ]
   }

-  ``VirtualHost`` 가상 호스트 이름

-  ``Address`` :ref:`env-vhost-activeorigin` . 설정 주소가 사용중인 경우 ``Active`` , (장애가 발생) 사용하지 않는 경우 ``Inactive`` 로 표시된다.

-  ``Address2`` :ref:`env-vhost-standbyorigin` . 설정 주소를 사용중인 경우 ``Active`` , 사용하지 않는 경우 ``Inactive`` 로 표시된다.

-  ``ActiveIP`` 사용중인 IP 목록 및 TTL. 원본 서버를 IP로 설정하면 ``Address`` 와 동일한 IP에 TTL은 0으로 표시된다. Domain으로 설정하면 Resolving 결과에 따른다. 다양한 IP 및 TTL을 사용한다.

-  ``InactiveIP`` 사용하지 않는 IP 목록 및 TTL. 사용하지 않아도 복구하고 있는지 HealthChecker에 의해 관리 할 수있다. 그 주소는 TTL 동안 회복되지 않으면 제거된다



.. _origin-status-reset:

원래 상태로 초기화
====================================

API를 통해 가상 호스트의 소스 서버 제거 / 복구를 초기화한다. 또한 현재 사용중인 세션을 재사용하지 않고 새로운 연결을 생성합니다. ::

   http://127.0.0.1:10040/command/resetorigin       // 모든 가상 호스트
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com



.. _origin-busysessioncount:

과부하 판단
====================================

처음에 요구되는 콘텐츠는 항상 원래 서버로 요청해야한다. 그러나 이미 Caching 된 컨텐츠이면 더 유연하게 대응할 수있다. 원본 서버가 과부하 상태라고 판단되면 업데이트를 밀어서 원래로드 높지 않다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BusySessionCount>100</BusySessionCount>

-  ``<BusySessionCount> (기본: 100개)``
   소스 서버와 HTTP 트랜잭션을 진행중인 세션의 수가 일정 수를 넘으면 과부하 상태라고 판단한다. 과부하 상태에서 만료 된 컨텐츠를 업데이트하기 위해 원본 서버에 연결하지 않도록 TTL을 :ref:`caching-policy-ttl` 의 ``<OriginBusy>`` 만 연장한다. 무조건 원래 서버로 요청이 가게하려면이 값을 매우 크게 설정하면된다.


.. _origin-balancemode:

소스 선택
====================================

원래 서버의 주소가 멀티 (2 개 이상)로 구성되어있는 경우 원본 서버 선택 정책을 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>

-  ``<BalanceMode> (기본: RoundRobin)``

   -  ``RoundRobin (기본)``
      모든 소스 서버가 균등하게 요청을 수신하도록 RoundRobin에서 동작한다. 연결된 Idle 세션은 해당 서버에 요청이 필요한 경우에만 사용한다.

   -  ``Session``
      재사용 가능한 세션이있는 경우 먼저 사용한다. 새 세션이 필요한 경우 Round-Robin하게한다.

   -  ``Hash``
      콘텐츠를 `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_ 알고리즘에 따라 원래의 서버에 분산하여 요청한다. 서버가 선택되면 이미 연결된 세션을 재사용하지 않는 경우, 새로 연결한다.


=========== =================================================================== =====================================================
/           RoundRobin                                                          Session
=========== =================================================================== =====================================================
부하 (요청)	 모든 서버가 부하를 균등하게 분배	                                반응성 및 재사용 성이 좋은 서버에 부하가 가중되는
연결 비용	 고 (서버의 순서가되면 연결된 세션을 찾고 있지 않으면 연결 시도)   낮은 (재사용 가능한 세션이없는 경우에만 연결)
재사용	 낮은 (서버 분배 우선)	                                            고 (항상 연결된 세션을 우선 사용)
세션 수	    많이 (각 서버마다 동시에 진행되는 HTTP 트랜잭션의 합계)          적은 (동시에 진행되는 HTTP 트랜잭션 만 세션이 존재)
=========== =================================================================== =====================================================


세션의 재사용
====================================

원본 서버가 Keep-Alive를 지원하는 경우 연결된 세션은 항상 다시 사용된다. 그러나 세션을 재사용하여 전송 요청에 대해 원래 서버가 일방적으로 연결을 종료 할 수있다. 때문에 연결을 회복하기 위해 사용자의 반응이 늦어 질 가능성이있다. 특히 오랫동안 다시 사용하지 않는 세션의 경우, 이러한 가능성은 더욱 높다. 이를 방지하기 위해 n 초 동안 재사용되지 않는 세션에 자동으로 연결을 종료하도록 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ReuseTimeout>60</ReuseTimeout>

-  ``<ReuseTimeout> (기본: 60초)``
   일정 시간 동안 사용되지 않은 원래의 세션은 종료한다. 0으로 설정하면 원래 서버의 세션을 재사용하지 않는다.


.. _origin_partsize:

Range 요구
====================================

한 번 다운로드 된 콘텐츠의 크기를 설정한다. 동영상처럼 앞부분 만 주로 소비되는 콘텐츠의 경우 다운로드 크기를 제한하면 불필요한 소스 트래픽을 줄일 수있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>

-  ``<PartSize> (기본: 0 MB)``
   0보다 큰 경우, 클라이언트가 요구 한 시점에서 설정 크기 (MB) 만 Range 요구에 다운로드한다.


``<PartSize>`` 를 사용하는 또 다른 이유는 디스크 공간을 절약하기 때문이다. 기본적으로 STON 원래 크기의 파일을 디스크에 생성한다. 그러나 ``<PartSize>`` 가 0이 아닌 경우는 다운로드되는만큼 파일을 분할하여 저장한다.

예를 들어, 1 시간의 영상 (600MB)을 1 분 (10MB) 만 시청 한 경우에 디스크 공간 10MB만을 사용한다. 공간을 절약하는 장점은 있지만, 파일이 분할되어 저장되기 때문에 디스크 부하가 약간 높아진다.

.. note::

   첫 번째 콘텐츠를 다운로드 할 때 Content-Length를 알 수 없기 때문에, Range 요구를 할 수 없다. 때문에 ``<PartSize>`` 가 설정되어있는 경우 설정 크기만큼 다운로드하여 연결을 종료한다.




전체 Range 초기화
====================================

일반적으로 원본 서버에서 처음 파일을 다운로드하거나 업데이트 확인하면 다음과 같이 간단한 형태의 GET 요청을 보냅니다. ::

    GET /file.dat HTTP/1.1

그러나 원래 서버가 일반적인 GET 요청에 대해 항상 파일을 변조하도록 설정되어있는 경우 원본 파일을 그대로 Caching 수없는 문제가 될 수 있습니다.

가장 대표적인 예는 Apache Web 서버가 mod_h.264_streaming 등의 외부 모듈과 같이 구동되는 경우이다. Apache Web 서버는 GET 요청에 대해 항상 mod_h.264_streaming 모듈을 통해 응답한다. 클라이언트 (이 경우에는 STON)은 원본 파일의 상태가 아니라 모듈에 의해 변조 된 파일을 서비스받을 수 있습니다.

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center

      mod_h.264_streaming 모듈은 항상 소스를 변조한다.

Range 요청을 사용하면 모듈을 무시하고 원본을 다운로드 할 수있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <FullRangeInit>OFF</FullRangeInit>

-  ``<FullRangeInit>``

   - ``OFF (기본)`` 일반적인 HTTP 요청을 보낸다.。

   - ``ON`` 0부터 시작 Range 요구를 보낸다. Apache의 경우 Range 헤더가 명시되면 모듈을 우회한다. ::

        GET /file.dat HTTP/1.1
        Range: bytes=0-

     먼저 파일 Caching 때 콘텐츠의 Range를 모르기 때문에 Full-Range (= 0부터 시작)를 요청한다. 원본 서버가 Range 요구에 제대로 응답 (206 OK) 여부를 반드시 확인해야한다.

콘텐츠를 업데이트 할 때 다음과 같이 **If-Modified-Since** 헤더가 함께 제공된다. 소스 서버가 제대로 **304 Not Modified** 응답해야한다. ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT

.. note::

   ``<FullRangeInit>`` 이 제대로 작동하는지 확인하여 Web 서버의 목록입니다.

   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22


.. _origin-wholeclientrequest:

클라이언트의 요구를 유지
====================================

원래 요청하면 클라이언트가 보낸 요청을 유지하도록 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <WholeClientRequest>OFF</WholeClientRequest>

-  ``<WholeClientRequest>``

   - ``OFF (기본)`` Caching-Key를 다시 요청하는 URL에 사용한다.

   - ``ON`` 클라이언트가 요청 된 URL에 원래 요청한다.

Hit Ratio를 높이기 위해 다음 설정을 사용하여 Caching-Key를 결정한다.

- :ref:`caching-policy-casesensitive`
- :ref:`caching-policy-applyquerystring`
- :ref:`caching-policy-post-method-caching`

그러면 원래 서버에 요청한 URL과 Caching-Key는 다음과 같이 결정된다.

============================================== ======================= ============================
설정                                           클라이언트 요청 URL       원래 요청 URL / Caching-Key
============================================== ======================= ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png         /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png         /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A   /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A   /view/list.php?type=A
============================================== ======================= ============================

``<WholeClientRequest>`` 를 ``ON`` 으로 설정하면 다음과 같이 Caching-Key에 관계없이 클라이언트가 전송 한 URL을 그대로 바탕으로 보낸다.

============================================== =================================== ============================
設定                                            クライアント/元の要求URL           Caching-Key
============================================== =================================== ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png                     /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png                     /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A               /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A               /view/list.php?type=A
============================================== =================================== ============================

POST 요청을 캐시하는 경우 원래 서버에 요청할 때 클라이언트가 보낸 POST 요청의 Body 데이터가 변경없이 전송된다.

.. note::

   클라이언트가 보낸 URL을 그대로 보내 :ref:`media-trimming` 있도록 추가 기능을 위해 붙여진 QueryString 그대로 원래의 서버로 전송된다.


.. _origin-httprequest:

원래 요청의 기본 Header
====================================

Host 헤더
---------------------

소스 서버로 전송 HTTP 요청의 Host 헤더를 설정한다. 별도로 설정하지 않은 경우 가상 호스트 이름이 명시된다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Host />

-  ``<Host>``
   원래 서버로 전송 Host 헤더를 설정한다. 원래 서버에서 80 포트 이외의 포트에서 서비스하는 경우 반드시 포트 번호를 명시하여야한다. ::

      # server.xml - <Server><VHostDefault><OriginOptions>
      # vhosts.xml - <Vhosts><Vhost><OriginOptions>

      <Host>www.example2.com:8080</Host>


클라이언트가 보낸 Host 헤더를 다시 보내려면 *로 설정한다.


User-Agent 헤더
---------------------

소스 서버로 전송 HTTP 요청의 User-Agent 헤더를 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent>STON</UserAgent>

-  ``<UserAgent> (기본: STON)``
   원래 서버로 전송 UserAgent 헤더를 설정한다.


클라이언트가 전송 한 User-Agent 헤더를 다시 보내려면 *로 설정한다.


XFF (X-Forwarded-For) 헤더
---------------------

클라이언트와 원래 서버 사이에 STON가 위치하면 원래 서버는 클라이언트의 IP 주소를 얻을 수 없다. 위해 STON 원래 서버로 전송되는 모든 HTTP 요청에 X-Forwarded-For 헤더를 명시한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <XFFClientIPOnly>OFF</XFFClientIPOnly>

-  ``<XFFClientIPOnly>``

   - ``OFF (기본)`` 클라이언트 IP (128.134.9.1)가 전송 XFF 헤더에 클라이언트의 IP 주소를 추가합니다. 클라이언트가 XFF 헤더를 전송하지 않은 경우, 클라이언트의 IP 만 명시된다. ::

        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1

   - ``ON`` XFF 헤더의 첫 번째 주소 만 원래 서버에 전송한다. ::

        X-Forwarded-For: 220.61.7.150


ETag 헤더 인식
---------------------

원본 서버에서 응답 ETag 인식할지 여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>

-  ``<OriginalETag>``

   - ``OFF (기본)``  ETag 헤더를 무시한다.

   - ``ON`` ETag를 인식하고 콘텐츠 업데이트시 If-None-Match 헤더를 추가합니다.



.. _origin_url_rewrite:

원래 요청 URL 변경
====================================

캐시를 목적으로 원본에 보내는 HTTP 요청의 URL을 변경한다. ::

   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <URLRewrite>
      <Pattern>/download/(.*)</Pattern>
      <Replace>/#1</Replace>
   </URLRewrite>
   // Pattern : /download/1.3.4
   // Replace : /1.3.4

   <URLRewrite>
      <Pattern>/img/(.*\.(jpg|png).*)</Pattern>
      <Replace>/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : /img/image.jpg?date=20140326
   // Replace : /image.jpg?date=20140326/STON/composite/watermark1

:ref:`handling_http_requests_url_rewrite` 같은 표현을 사용하지만 가상 호스트마다 독립적으로 설정하는 가상 호스트 이름을 입력하지 않았습니다.

.. note::

   무시되는 HTTP 요청의 URL을 수정할 수 없습니다.
   ``<WholeClientRequest>`` 보다 우선합니다.



.. _origin_modify_client:

원래 요청 헤더의 변경 
====================================

원래에 HTTP 요청을 보낼 때의 조건에 따라 HTTP 헤더를 변경한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>

-  ``<ModifyHeader>``

   -  ``OFF (기본)`` 변경하지 않는다.

   -  ``ON`` 헤더 변경 조건에 따라 헤더를 변경한다.

헤더 변경 시점은 HTTP 요청 패킷이 완성되어 원래의 서버로 전송되기 직전에 실행된다. 그러나 Range 헤더는 변조 할 수 없다.

이 기능은 :ref:`handling_http_requests_modify_client` 의 하위 기능이다. 헤더 변경은 $ ORGREQ 키워드를 사용한다. ::

   # /svc/www.example.com/headers.txt

   $URL[/*.mp4], $ORGREQ[x-media-type: video/mp4], set
   $IP[1.1.1.1], $ORGREQ[user-agent: media_probe], put
   *, $ORGREQ[If-Modified-Since], unset
   *, $ORGREQ[If-None-Match], unset

   # #PROTOCOL 키워드를 통해 클라이언트가 요청한 프로토콜을 헤더에 추가합니다.
   $URL[*], $ORGREQ[X-Forwarded-Proto: #PROTOCOL], set


.. note::

   If-Modified-Since 헤더와 If-None-Match 헤더를 ``unset`` 하는 TTL이 만료 콘텐츠는 항상 다시 다운로드한다.
