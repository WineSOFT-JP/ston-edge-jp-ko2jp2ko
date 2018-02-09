.. _adv-vhost:

제 14 장 가상 호스트의 고급 기술
******************

이 장에서는 가상 호스트를 사용하여 서비스를 유연하게 구성하는 여러 방법에 대해 설명한다.

가상 호스트는 일반적으로 원래의 (Domain 또는 IP 목록)과 1 : 1로 구성되는 것이 기본이다. 그러나 상황에 따라 대표 가상 호스트를 여러 하위 가상 호스트에 분기하거나 반대로 독립적 인 여러 가상 호스트를 하나의 서비스로 패키지하는 경우도 발생한다. 각 기능에 따라 :ref:`monitoring_stats_vhost_client` / :ref:`admin-log-access` 의 정책이 다를 수 있다는 점에 유의해야한다.


.. toctree::
   :maxdepth: 2


.. _adv-vhost-url-rewrite:

URL 전처리
====================================

`정규표현식 <http://en.wikipedia.org/wiki/Regular_expression>`_ 을 사용하여 요청 된 URL을 변경한다. URL 전처리가 설정되어있는 경우, 모든 클라이언트 요청 (HTTP 또는 File I / O)는 반드시 URL Rewriter를 거친다.

.. figure:: img/urlrewrite1.png
   :align: center

   URL Rewriter를 통과해야한다면 가상 호스트에 갈 수있다.

만약 URL Rewriter에 의해 액세스하려는 Host의 이름이 변경된 경우 클라이언트의 HTTP 요청의 Host 헤더가 변경된 것으로 본다. URL의 전처리는 가상 호스트 설정 (vhosts.xml)로 설정한다. 대부분의 설정은 가상 호스트에 따라 다르지만, URL 전처리의 경우 클라이언트가 요구 한 Host의 이름을 변경할 수 있도록 가상 호스트와 동일한 수준으로 설정한다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost ...> ... </Vhost>
      <Vhost ...> ... </Vhost>
      <URLRewrite ...> ... </URLRewrite>
      <URLRewrite ...> ... </URLRewrite>
   </Vhosts>

멀티에 설정할 수 있으며, 순차적으로 정규 표현에 일치 여부를 비교한다. ::

   # vhosts.xml - <Vhosts>

   <URLRewrite AccessLog="Replace">
       <Pattern>www.exmaple.com/([^/]+)/(.*)</Pattern>
       <Replace>#1.exmaple.com/#2</Replace>
   </URLRewrite>

-  ``<URLRewrite>``

   URL 전처리를 설정한다.
   ``AccessLog (기본: Replace)`` 속성은 Access 로그에 기록되는 URL을 설정한다.
   ``Replace`` 경우, 변환 된 URL (/logo.jpg)를 ``Pattern`` 의 경우 변환되지 않은 URL (/baseball/logo.jpg)을 Access 기록한다.

   -  ``<Pattern>`` 일치하는 패턴을 설정한다. 하나의 패턴은 () 괄호를 사용하여 표현된다.

   -  ``<Replace>`` 변환 형식을 설정한다. 일치하는 패턴은 # 1, # 2와 같이 사용할 수있다. # 0은 요청 URL 전체를 의미한다. 패턴은 최대 9 개 (# 9)까지 지정할 수있다.

처리량은 :ref:`monitoring_stats` 로 볼 :ref:`api-graph-urlrewrite` 하지만 확인할 수있다. URL 전처리는 :ref:`media-trimming` , :ref:`media-hls` 등의 다른 기능과 결합하여 표현을 간결하게한다. ::

   # vhosts.xml - <Vhosts>

   <URLRewrite>
       <Pattern>example.com/([^/]+)/(.*)</Pattern>
       <Replace>example.com/#1.php?id=#2</Replace>
   </URLRewrite>
   // Pattern : example.com/releasenotes/1.3.4
   // Replace : example.com/releasenotes.php?id=1.3.4

   <URLRewrite>
       <Pattern>example.com/download/(.*)</Pattern>
       <Replace>download.example.com/#1</Replace>
   </URLRewrite>
   // Pattern : example.com/download/1.3.4
   // Replace : download.example.com/1.3.4

   <URLRewrite>
       <Pattern>example.com/img/(.*\.(jpg|png).*)</Pattern>
       <Replace>example.com/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : example.com/img/image.jpg?date=20140326
   // Replace : example.com/image.jpg?date=20140326/STON/composite/watermark1

   <URLRewrite>
       <Pattern>example.com/preview/(.*)\.(mp3|mp4|m4a)$</Pattern>
       <Replace><![CDATA[example.com/#1.#2?&end=30&boost=10&bandwidth=2000&ratio=100]]></Replace>
   </URLRewrite>
   // Pattern : example.com/preview/audio.m4a
   // Replace : example.com/audio.m4a?end=30&boost=10&bandwidth=2000&ratio=100

   <URLRewrite>
       <Pattern>example.com/(.*)\.mp4\.m3u8$</Pattern>
       <Replace>example.com/#1.mp4/mp4hls/index.m3u8</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4.m3u8
   // Replace : example.com/video.mp4/mp4hls/index.m3u8

   <URLRewrite>
       <Pattern>example.com/(.*)_(.*)_(.*)</Pattern>
       <Replace>example.com/#0/#1/#2/#3</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4_10_20
   // Replace : example.com/example.com/video.mp4_10_20/video.mp4/10/20

패턴의 표현에 XML의 5 개의 특수 문자 ( "&"<>)이 들어가는 경우는 <! [CDATA [...]]>로 묶어 올바르게 설정되어있다.
:ref:`wm` 을 사용하여 설정하면 모든 패턴은 CDATA로 처리된다.



.. _adv-vhost-facadevhost:

Facade 가상 호스트
====================================

``<Alias>`` 가상 호스트 별명 만 추가하고 있기 때문에, 통계 및 로그가 분리되지 않는다. 가상 호스트는 공유가 도메인에 따라 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 를 분리하려면 Facade 가상 호스트를 설정한다.

.. figure:: img/adv_vhost_facade.png
   :align: center

   facade는 통계 및 로그를 수집합니다.

::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
       ...
    </Vhost>

    <Vhost Name="another.com" Status="facade:example.com">
       ...
    </Vhost>

``Status``  속성의 값을 ``facade:`` + ``仮想ホスト`` 설정한다. 예의 경우 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 example.com 대신 클라이언트가 요청한 도메인 인 another.com에 수집된다.



.. _adv-vhost-sub-path:

Sub-Path 지정
====================================

가상 호스트에서 경로를 따라 다른 가상 호스트가 처리하도록 설정할 수있다.

.. figure:: img/adv_vhost_subpath.png
   :align: center

   통계 / 로그 요청을 최종 처리하는 각 가상 호스트에 기록된다.


::

   # vhosts.xml - <Vhosts>

   <Vhost Name="sports.com">
     <Sub Status="Active">
       <Path Vhost="baseball.com">/baseball/<Path>
       <Path Vhost="football.com">/football/<Path>
       <Path Vhost="photo.com">/*.jpg<Path>
     </Sub>
   </Vhost>

   <Vhost Name="baseball.com" />
   <Vhost Name="football.com" />
   <Vhost Name="photo.com" />

-  ``<Sub>`` 경로 또는 패턴이 일치하면 해당 요청을 다른 가상 호스트에 보낸다. 일치하지 않는 경우에만 현재 가상 호스트가 처리한다.

   - ``Status (기본: Active)`` Inactive의 경우는 무시한다.

   -  ``<Path>`` 클라이언트가 요청한 URI와 경로가 일치하는 경우 ``Vhost`` 에 요청을 보낸다. 값은 경로 또는 패턴 만 가능하다. ::

         <Path Vhost="baseball.com">baseball<Path>
         <Path Vhost="photo.com">*.jpg<Path>

      위와 같이 입력하면 각각 / baseball /와 / * .jpg로 인식된다.

예를 들어, 클라이언트가 다음과 같이 요청하면 그 요청은 가상 호스트의 football.com가 처리한다. ::

   GET /football/rank.html HTTP/1.1
   Host: sports.com


.. _adv-vhost-redirection-trace:

Redirect 추적
====================================

원본 서버에서 Redirect 계 (301,302,303,307)로 응답하는 경우 Location 헤더를 추적하여 콘텐츠를 요청한다.

   .. figure:: img/conf_redirectiontrace.png
      :align: center

      클라이언트는 Redirect 여부를 모른다.

::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <RedirectionTrace>OFF</RedirectionTrace>

-  ``<RedirectionTrace>``

   - ``OFF (기본)`` 3xx 응답으로 저장된다.

   - ``ON`` Location 헤더에 기재된 주소에서 콘텐츠를 다운로드한다. 형식에 맞지 않을 경우 Location 헤더가 없으면 작동하지 않습니다. 무한 Redirect되는 것을 방지하기 위해 한 번만 추적한다.



.. _adv-vhost-link:

가상 호스트의 링크
====================================

콘텐츠가 여러 소스에 분산되어있는 경우 가상 호스트의 링크를 이용하여 컨텐츠가 통합되어있는 것처럼 서비스가 가능하다. 특히 On-Premise에서 클라우드 스토리지 마이그레이션하거나 스토리지 용량, 비용 등의 이유로 콘텐츠가 분산되어있는 환경에서 유용하다.

.. figure:: img/adv_vhost_link.png
   :align: center

   cloud.com에없는 내용은 nas.com가 처리한다.

::

   # vhosts.xml - <Vhosts><Vhost>

   <VhostLink Condition="...">...</VhostLink>

-  ``<VhostLink>`` 요청을 위임 가상 호스트 이름. 컨텐츠 소스의 응답이 ``Condition`` 을 만족하면 지정된 가상 호스트에 요청을 위임합니다. 그러나 하나만 설정할 수있다.

   - ``Condition`` HTTP 응답 코드 / 패턴 (1xx, 2xx, 3xx, 4xx, 5xx), fail (소스에서 캐시되지 않은 경우)

클라이언트의 요구가 다른 가상 호스트에 위임하더라도 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 클라이언트가 액세스 한 가상 호스트에 기록된다.

.. note::

   링크 관계에있는 가상 호스트의 설정이 다른 경우 의도하지 않은 동작하는 것을주의한다. 가상 호스트의 링크가 A (단순 캐시) -> B (영상 압축)로 연결되어있는 경우는 A에서 처리 된 이미지는 압축되지 않지만 B로 처리 된 이미지는 압축된다 .

예를 들어 nas.com 콘텐츠를 cloud.com로 이전하는 경우, cloud.com없는 (= 404 Not Found) 콘텐츠에만 nas.com에 요청을 보낼 수있다. 아래의 경우 요청이 nas.com 의해 처리 되어도 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 cloud.com에 기록된다.

::

   # vhosts.xml - <Vhosts>

   // cloud.comに 없는(=404 Not Found) 의 콘텐츠는 nas.  com에서 서비스한다.
   <Vhost Name="cloud.com">
     <VhostLink Condition="404">nas.com</VhostLink>
   </Vhost>

   <Vhost Name="nas.com">
   </Vhost>


:ref:`admin-log-access` 의 vhostlink 필드를 통해 클라이언트의 요구가 어떤 가상 호스트에서 처리 된 것을 알 수있다. 「 - 」는 요청이 연결되어 있지 않다는 것을 의미한다 "nas.com"는 요청이 링크되어 nas.com에서 처리 된 것을 의미한다. ::

    #Fields: date time s-ip cs-method cs-uri-stem ...(중략)... vhostlink
    2016.11.24 16:52:24 220.134.10.5 GET /web/h.gif ...(중략)... -
    2016.11.24 16:52:26 220.134.10.5 GET /favicon.ico ...(중략)... nas.com

링크가 여러 번 발생하면 「+」를 구분자로 연결된 모든 가상 호스트는 명시된다. 이 경우 마지막 가상 호스트가 마지막 요청을 처리 한 가상 호스트이다.

다음과 같이 여러 가상 호스트를 다른 기준으로 연결될 수있다.


::

   # vhosts.xml - <Vhosts>

   //  원본 서버가 5 xx에 응답하거나 캐시하지 않은 경우 (= fail) 요청을 bar.com에게 위임한다.
   <Vhost Name="foo.com">
     <VhostLink Condition="5xx,fail">bar.com</VhostLink>
   </Vhost>

   // 원본 서버가 4 xx에 응답 할 때 요청을 helloworld.com에게 위임한다.
   <Vhost Name="bar.com">
     <VhostLink Condition="4xx">helloworld.com</VhostLink>
   </Vhost>

   // 원본 서버에서 403,404 또는 5 xx에 응답 할 때 요청을 example.com에게 위임한다.
   <Vhost Name="helloworld.com">
     <VhostLink Condition="403,404,5xx">example.com</VhostLink>
   </Vhost>

   // 더 이상 위임하지 않는다.
   <Vhost Name="example.com">
   </Vhost>

.. figure:: img/adv_vhost_link_worst.png
   :align: center

   억제하면서 가능하다.

위의 예제의 경우 foo.com의 :ref:`admin-log-access` 는 다음과 같다. ::

   #Fields: date time s-ip cs-method cs-uri-stem ...(중략)... vhostlink
   2016.11.24 16:52:24 220.134.10.5 GET /test.jpg ...(중략)... bar.com+helloworld.com+example.com

다음의 경우 링크는 즉시 중단된다.

* 대상 가상 호스트가 존재하지 않는 경（foo.com - >？）
* 자신을 대상 가상 호스트에 지정한 경우（foo.com - > foo.com）
* 재귀 링크 (Recursive Link)가 발생한 경우（foo.com - > bar.com - > foo.com）
