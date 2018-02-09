.. _bypass:

제 8 장 우회
******************

이 장에서는 클라이언트 요청을 원래 서버에 위임 우회에 대해 설명한다. 우회 조건과 동작으로 구분된다.

바이 패스는 Caching 정책보다 우선합니다. 설계 단계에서 Edge 도입이 검토되지 않은 서비스라면 정적 리소스와 동적 리소스를 정교하게 구분 해낼 수없는 경우가 많다. 이런 경우 모든 클라이언트 요청을 무시하도록 구성한 후 로그를 기반으로 까다로운 콘텐츠 만 Caching 있도록 설정한다. 대부분의 경우 몇 시간의 로그뿐만 원래의 부하를 획기적으로 낮출 수있다.
:ref:`monitoring_stats` 에 실시간 정보를 제공하는 이유도 서비스를 실시간 튜닝 할 수 있도록한다.

바이 패스는 매우 빠른뿐만 아니라 HTTP 트랜잭션 단위로 동작한다. 아무리 개인 사이트라고해도 대부분은 메인 페이지 (.html)에만 동적으로 변경 될뿐, 나머지 99 %는 정적 리소스로 구성되어있다. 소스 서버의 동작에 맞출 수 있도록 :ref:`origin-httprequest` 우회 버전이 별도로 존재한다.


.. toctree::
   :maxdepth: 2


No-Cache 요청 우회
====================================

클라이언트가 no-cache 요청을 제출하면 우회 ::

   GET / HTTP/1.1
   cache-control: no-cache 또는 cache-control:max-age=0
   pragma: no-cache

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassNoCacheRequest>OFF</BypassNoCacheRequest>

-  ``<BypassNoCacheRequest>``

   - ``OFF (기본)`` Cache 모듈이 처리한다.

   - ``ON`` 소스 서버 우회한다.

.. note::

    이 설정은 클라이언트 동작 (아마도 ``ctrl`` + ``F5`` )에 의해 결정된다. 따라서 대량의 바이 패스가 원래에 부담을 줄 수있다.


.. _bypass-getpost:

GET / POST 우회
====================================

바이 패스가 GET / POST 요청의 기본 동작이되도록 설정할 수있다. GET과 POST 용도가 다를뿐만 기본 동작이 다르다는 점에 유의한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest>ON</BypassPostRequest>
   <BypassGetRequest>OFF</BypassGetRequest>

-  ``<BypassPostRequest>``

   - ``ON (기본)`` POST 요청을 원본 서버로 우회한다.

   - ``OFF`` POST 요청을 STON가 처리한다.

-  ``<BypassGetRequest>``

   - ``OFF (기본)`` GET 요청을 STON가 처리한다.

   - ``ON`` GET 요청을 원본 서버로 우회한다.

:ref:`access-control-vhost` 와 같은 조건을 모두 지원한다. 우회 예외 조건 / svc / {가상 호스트 이름} /bypass.txt로 설정한다. ::

   # /svc/www.example.com/bypass.txt
   $IP[192.168.2.1-255]
   /index.html

cache와 bypass 조건을 명확하게하지 않으면 기본 설정과 반대로 동작한다. 예를 들어, ``<BypassGetRequest>`` 가 ``ON`` 이면 예외 조건은 Caching 목록된다. 혼란의 여지가 많은 경우 두 번째 매개 변수를 사용하여보다 명확하게 조건을 설정할 수있다. ::

   # /svc/www.winesoft.co.kr/bypass.txt

   $HEADER[cookie: *ILLEGAL*], cache               // 항상 Caching 처리
   !HEADER[referer:]                               // 기본 설정에 따라
   !HEADER[referer] & !HEADER[user-agent], bypass  // 항상 무시
   $URL[/source/public.zip]                        // 기본 설정에 따라

정리하면, 우선 순위는 다음과 같다.

1. No-Cache 우회
2. bypass.txt에 bypass라고 명기되어
3. bypass.txt의 기본 설정



소스 서버의 고정
====================================

로그인 상태처럼 원래의 서버와 클라이언트가 반드시 1 : 1로 통신 할 필요가있다.
`GET/POST 바이패스`_ 의 특성에 원래의 서버를 고정시킬 수있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest OriginAffinity="ON">...</BypassPostRequest>
   <BypassGetRequest OriginAffinity="ON">...</BypassGetRequest>

-  ``OriginAffinity``

   - ``ON (기본)`` 클라이언트 요청이 항상 같은 서버에 바이 패스되는 것을 보장한다. 그러나 동일한 소켓임을 보증하는 것은 아니다.

우회해야한다면 소스 서버와 연결된 모든 소켓이 끊어지는 상황이 발생하기도한다. 그러나 이러한 경우에도 서버에 새로운 소켓 접속을 요구한다.

     .. figure:: img/private_bypass3.jpg
        :align: center

        항상 같은 서버에 무시된다.

     무시했던 소스 서버가 장애를 제거하거나 DNS에 빠진 경우 새 서버로 우회된다.

   - ``OFF`` 클라이언트의 요청이 어떤 서버에 바이 패스되는 것을 보장 할 수 없다.

     .. figure:: img/private_bypass1.jpg
        :align: center

        :ref:`origin-balancemode` 에 의해 따른다.



원래 세션 고정
====================================

클라이언트 소켓마다 원래의 서버와 1 : 1로 우회 세션을 사용한다.

.. figure:: img/private_bypass2.jpg
   :align: center

   클라이언트가 원래 세션을 소유한다.

`GET/POST 바이패스`_ 의 특성에 원래 세션을 고정 할 수있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest Private="OFF">...</BypassPostRequest>
   <BypassGetRequest Private="OFF">...</BypassGetRequest>

-  ``Private``

   - ``ON`` 클라이언트 세션이 원래 서버 전용의 세션을 사용하도록 동작한다. 항상 같은 서버로 요청이 무시된다. 클라이언트와 원래 서버 중 어느 하나의 세션이 종료 된 순간, 상대의 세션도 종료된다.

   - ``OFF (기본)`` 전용 세션을 사용하지 않는다.

원본 서버가 사용자의 로그인 정보를 세션에 따라 유지하는 경우처럼 클라이언트의 요구가 반드시 동일한 소켓에 처리되어야 할 경우에 편리하다.

.. note::

   자칫하면 너무 많은 요구를 ``Private`` 우회하는 경우 클라이언트의 수만큼 원본 서버에 연결되어 엄청난 부하를 줄 수있다. 또한 이렇게 연결된 소스 세션은 클라이언트가 소유하게되기 때문에 악의적 인 공격 상황에서 위험을 초래할 수있다.


Timeout
-----------------------

바이 패스는 원래 서버에서 동적으로 처리 한 결과를 응답하는 경우가 많다. 이로 인해 처리 속도가 정적 콘텐츠보다 느린 경우가 많다. 바이 패스 전용 Timeout을 설정하고 서투른 장애의 판단이되지 않도록한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BypassConnectTimeout>5</BypassConnectTimeout>
   <BypassReceiveTimeout>300</BypassReceiveTimeout>

-  ``<BypassConnectTimeout> (기본: 5초)``
   우회를 위해 n 초 내에 원래 서버와의 연결이되지 않은 경우 연결이 실패로 처리한다.


-  ``<BypassReceiveTimeout> (기본: 5초)``
   우회 중에 소스 서버에서 응답이 n 초 없으면 전송 실패로 처리한다.



우회 헤더
====================================

:ref:`origin-httprequest` 설정을 우회하는 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent Bypass="OFF">...</UserAgent>
   <Host Bypass="ON"/>
   <XFFClientIPOnly Bypass="ON">...</XFFClientIPOnly>

-  ``Bypass`` 특성

   - ``ON`` 으로 설정되고 헤더를 지정한다.。

   - ``OFF`` 클라이언트가 보낸 관련 헤더를 명시한다.


.. _bypass-port:

Port 우회
====================================

특정 TCP 포트의 모든 패킷을 원본 서버로 우회한다. 가상 호스트에만 설정이다. ::

   # vhosts.xml - <Vhosts>

   <Vhost Name="www.example">
      <PortBypass>443</PortBypass>
      <PortBypass Dest=”1935”>1935</PortBypass>
   </Vhost>

-  ``<PortBypass>``
   지정된 포트에서 입력 된 모든 패킷을 원래 서버의 같은 포트에 우회. 
   ``Dest`` 특성에 소스 서버의 포트를 설정한다.

예를 들어, 포트 443를 우회하는 경우 클라이언트는 원래 서버와 직접 SSL 통신을하는 효과를 갖는다. 무시되는 포트는 절대로 중복 할 수 없다.

.. note::

   구조적으로 Port 우회는 HTTP보다 하위 Layer 인 TCP에서 열린다. 특정 가상 호스트의 하위 Port 바이 패스를 설정하는 이유는 통계 정보를 수집하는 주체가 필요하기 때문이다.
