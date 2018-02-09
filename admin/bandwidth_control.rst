.. _bandwidth_control:

제 16 장 Bandwidth 
******************

이 장에서는 가상 호스트마다 다양한 방식의 Bandwidth 제한 (조절)하는 방법을 설명한다. 일단 Bandwidth가 일정 수준을 넘지 않도록 제한하는 것이 목적이었다. 지금 효과적으로 Bandwidth를 조절함으로써 그 개념이 옮겨 갔다. 또한 콘텐츠를 실시간으로 분석하여 각각에 최적화 된 Bandwidth를 사용하도록 설정할 수있다.


.. toctree::
   :maxdepth: 2

가상 호스트 Bandwidth 제한
====================================

가상 호스트의 최대 Bandwidth를 제한한다. 이것은 가장 선호하는 물리적 인 방법이다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <TrafficCap Session="0">0</TrafficCap>

-  ``<TrafficCap> (기본: 0 Mbps)`` 의 가상 호스트의 최대 Bandwidth를 Mbps 단위로 설정한다. 0으로 설정하면 Bandwidth를 제한하지 않는다. ``Session (기본: 0 Kbps)`` 속성은 클라이언트 세션 당 보낼 수있는 최대의 Bandwidth를 설정한다.

예를 들어, ``<TrafficCap>`` 50 (Mbps)로 설정 한 경우 50Mbps NIC를 설치 한 것과 같은 효과를 낸다. 가상 호스트에 액세스하는 모든 클라이언트 Bandwidth 총 50Mbps를 초과 할 수 없다.

``Session`` 은 다음과 같이 작동

1. ``Session`` 이 설정되어 있어도 모든 클라이언트 Bandwidth의 합계는 ``<TrafficCap>`` 을 초과 할 수 없다.
2. `Bandwidth Throttling`_ 을 설정해도 클라이언트 세션의 최대 속도는 ``Session`` 을 초과 할 수 없다.


.. _bandwidth-control-bt:

Bandwidth Throttling
====================================

BT (Bandwidth Throttling)과 (각 세션 당) 클라이언트의 전송 대역폭을 동적으로 조정하는 기능이다. 일반적인 미디어 파일 내부에는 다음과 같이 헤더, V (Video), A (Audio)로 구성되어있다.

.. figure:: img/conf_media_av.png
   :align: center

   헤더는 BT의 대상이 아니다.

헤더는 재생 시간이 긴 Key Frame주기가 짧을수록 커진다. 따라서 인식 할 수있는 미디어 파일이면 원활한 재생을 위해 헤더는 대역폭 제한없이 전송한다. 다음 그림과 같이 헤더가 완전히 전송 된 후 BT가 시작된다.

.. figure:: img/conf_bandwidththrottling2.png
   :align: center

   동작 시나리오

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Unit="kbps">1000</Bandwidth>
         <Ratio>100</Ratio>
         <Boost>5</Boost>
      </Settings>
      <Throttling>OFF</Throttling>
   </BandwidthThrottling>

``<BandwidthThrottling>`` 태그의 하위에 기본 동작을 설정한다.

-  ``<Settings>``

   기본 동작을 설정한다.

   -  ``<Bandwidth> (基本: 1000 Kbps)``
      클라이언트의 전송 대역폭을 설정한다. 
      ``Unit`` 속성을 통해 기본 단위位( ``kbps`` , ``mbps`` , ``bytes`` , ``kb`` , ``mb`` )를 설정한다.

   -  ``<Ratio> (기본: 100 %)``
      ``<Bandwidth>`` 설정 비율을 반영하여 대역폭을 설정한다.

   -  ``<Boost> (기본: 5 초)``
      일정 시간 만 데이터를 속도 제한없이 클라이언트로 전송한다. 데이터의 양은 ``<Boost>`` X ``<Bandwidth>`` X ``<Ratio>`` 공식 계산한다.

-  ``<Throttling>``

   -  ``OFF (기본)`` BT를 적용하지 아니한다.
   -  ``ON`` 조건 목록과 일치하면 BT를 적용한다.


Bandwidth Throttling 조건 목록
--------------------------

BT의 조건 목록을 설정한다. 조건 목록과 일치해야 BT가 적용된다. 설정된 순서대로 조건과 일치하는 검사이다. 전송 정책 / svc / {가상 호스트 이름} /throttling.txt로 설정한다. ::

   # /svc/www.example.com/throttling.txt
   # 구분 기호는 쉼표 (,)이며, {조건}, {Bandwidth}, {Ratio}, {Boost} 순으로 표기한다.
   # {조건}를 제외한 모든 필드는 선택 사항이다.
   # 생략 된 필드는 ``<Settings>`` 에 설정된 기본값이 사용된다.
   # 모든 조건식은 acl.txt 설정과 동일하다.
   # {Bandwidth} 단위는 ``<Settings>`` ``<Bandwidth>`` 의 ``Unit`` 속성을 사용한다.

   # 3 초의 데이터 속도 제한없이 전송 한 후 3Mbps（3000Kbps = 2000Kbps X 150％）에서 클라이언트로 전송한다.
   $IP[192.168.1.1], 2000, 150, 3

   # bandwidth만을 정의한다.  5 (기본) 초 데이터 속도 제한없이 전송 한 후 800 Kbps에서 클라이언트로 전송한다.
   !HEADER[referer], 800

   # boost만을 정의한다.  10 초 데이터 속도 제한없이 전송 한 후 1000 Kbps 클라이언트로 전송한다.
   HEADER[cookie], , , 10

   # 확장자가 m4a의 경우 BT를 적용하지 아니한다.
   $URL[*.m4a], no

미디어 파일 (MP4, M4A, MP3)를 분석하면 Encoding Rate에서 Bandwidth를 얻을 수있다. 액세스되는 콘텐츠의 확장자는 .mp4, .m4a, .mp3 중 하나이어야한다. 동적으로 Bandwidth를 추출하려면 다음과 같이 Bandwidth 뒤에 x를 붙인다. ::

   # /vod/*.mp4 파일에 액세스하면, bandwidth를 요구한다.  사용할 수없는 경우는 1000 bandwidthbandwidth에 사용한다.
   $URL[/vod/*.mp4], 1000x, 120, 5

   # user-agent 헤더가없는 경우는 bandwidth를 요구한다.  사용할 수없는 경우, 500 bandwidth 사용한다.
   !HEADER[user-agent], 500x

   # /low_quality/* 파일에 액세스하면, bandwidth를 요구한다.  사용할 수없는 경우 기본값을 bandwidth 사용한다.
   $URL[/low_quality/*], x, 200


QueryString 환경 설정
--------------------------

약속 된 QueryString을 사용하여 ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` 를 동적으로 설정한다. 이 설정은 BT의 조건보다 우선된다.

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Param="mybandwidth" Unit="mbps">2</Bandwidth>
         <Ratio Param="myratio">100</Ratio>
         <Boost Param="myboost">3</Boost>
      </Settings>
      <Throttling QueryString="ON">ON</Throttling>
   </BandwidthThrottling>

-  ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` 의 ``Param``

    각각의 의미에 맞게 QueryString 키를 설정한다.

-  ``<Throttling>`` 의 ``QueryString``

   - ``OFF (기본)`` QueryString 조건을 다시 정의하지 않습니다.

   - ``ON`` QueryString 조건을 다시 정의한다.

위와 같이 설정되어있는 경우는 다음과 같이 클라이언트가 요청한 URL을 기반으로 BT가 동적으로 설정된다. ::

    # 10 초의 데이터를 속도 제한없이 전송 한 후 1.3Mbps (1mbps X 130 %)에서 클라이언트로 전송한다.
    http://www.winesoft.co.kr/video/sample.wmv?myboost=10&mybandwidth=1&myratio=130

반드시 모든 매개 변수를 지정할 필요는 없다. ::

    http://www.winesoft.co.kr/video/sample.wmv?myratio=150

위와 같이 몇 가지 조건이 생략 된 경우 나머지 조건 (여기에서는 bandwidth, boost)를 결정하는 조건의 목록을 검색한다. 여기에서도 적절한 조건을 찾을 수 없으면 ``<Settings>`` 에 설정된 기본값이 사용된다. QueryString이 일부 존재도 조건 목록에서 배달 옵션 (no)으로 설정되어있는 경우 BT는 적용되지 않는다.

QueryString을 사용하기 때문에 자칫하면 :ref:`caching-policy-applyquerystring` 과 혼동을 일으킬 우려가있다.
:ref:`caching-policy-applyquerystring` 이 ``ON`` 의 경우 클라이언트가 요청 된 URL의 QueryString이 모든 인식되지만 ``BoostParam`` , ``BandwidthParam`` , ``RatioParam`` 은 제외된다. ::

   GET /video.mp4?mybandwidth=2000&myratio=130&myboost=10
   GET /video.mp4?tag=3277&myboost=10&date=20130726

예를 들어, 위와 같은 입력은 BT를 결정하는 사용 만 Caching-Key를 생성하거나 원래 서버로 요청을 보내는 경우는 삭제된다. 즉, 각각 다음과 같이 인식된다. ::

    GET /video.mp4
    GET /video.mp4?tag=3277&date=20130726
