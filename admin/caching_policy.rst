.. _caching-policy:

제 4 장 Caching 정책
******************

이 장에서는 서비스의 핵심 TTL (Time To Live)와 Caching-Key와 만료 정책에 대해 설명한다. 저장된 콘텐츠는 TTL 사이 효과적이다. HTTP 사양에서는 TTL을 설정하기 위해 특정 Cache-Control을 명시하고있다. 그러나 이것은 절대적인 것은 아니다. 다양한 방식의 TTL 정책과 :ref:`caching-purge` 를 통해 서비스의 질을 향상시킬 수있다.

HTTP 콘텐츠를 구별하는 다양한 규격이 존재한다. 그만큼 Caching-Key도 다양하게 존재할 수있다. 콘텐츠의 변경이 없을 정도 원래의 부하를 줄일 수있을뿐만 아니라 쉽게 확장 할 수있다. 서비스에 최적화 된 만료 정책을 수립하는 다양한 방법에 대해 설명한다.

앞으로 설명되는 설정을 모든 가상 호스트의 기본 설정을 적용하려면 ``<VHostDefault>`` 하위에 설정한다. 반대로, 특정 가상 호스트에만 적용 바르고 싶다면, <Vhost> 태그의 하위로 설정한다.

**Caching-Key** 와 콘텐츠를 분리하는 독특한 값이다. 파일 시스템의 파일과 구별되는 고유의 경로 (예 /usr/conf.txt)를 가지는 것과 같은 개념이다. 잘 Caching-Key는 URL과 혼동되기 쉽다. HTTP의 여러 기능에 따라 동일한 URL에도 콘텐츠가 변경 될 수 있습니다.


.. toctree::
   :maxdepth: 2



.. _caching-policy-ttl:

TTL (Time To Live)
====================================

TTL과 저장된 콘텐츠 사용 시간이다. TTL을 길게 설정하면 원래 서버의 부하는 줄어들지 만 변경이 늦게 반영된다. 반대로 짧게 설정하면 너무 잦은 변경 확인 요청에 원래 서버의 부하가 높아진다. 운영의 묘미는 TTL을 적절하게 설정하여 원래의 부하를 줄일 수있다. TTL은 일단 설정되면 만료까지 변하지 않는다. 새로운 TTL은 파일이 만료 때 적용된다. 관리자는 :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-expireafter` , :ref:`api-cmd-hardpurge` 등의 API를 사용하여 TTL을 변경할 수있다.


기본 TTL
---------------------

기본적으로 TTL은 원래 서버의 응답에 따라 결정된다. TTL이 만료 될 때까지 저장된 콘텐츠에 서비스된다. TTL이 만료되면 원래의 서버에 콘텐츠의 변경 여부( **If-Modified-Since** 또는 **If-None-Match** )를 확인한다. 원본 서버가 **304 Not Modified** 응답을 주면 TTL은 연장된다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTL>
        <Res2xx Ratio="20" Max="86400">1800</Res2xx>
        <NoCache Ratio="0" Max="5" MaxAge="0">5</NoCache>
        <Res3xx>300</Res3xx>
        <Res4xx>30</Res4xx>
        <Res5xx>30</Res5xx>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>3</ReceiveTimeout>
        <OriginBusy>3</OriginBusy>
    </TTL>

``Ratio`` （0〜100）를 제외한 모든 설정 단위는 초 (sec)이다.

-  ``<Res2xx> (기본: 1800초, Ratio: 20, Max=86400)``
   원본 서버가 200 OK로 응답했을 때 TTL을 설정한다. 콘텐츠를 처음 저장할 때 ``<Res2xx>`` 초 후 콘텐츠가 만료 (TTL)되도록 설정한다. (TTL 만료 후) 원본 서버에서 변경되지 않은 경우 (304 Not Modified) ``Ratio`` 비율 (0 ~ 100) 만 TTL을 연장한다. TTL은 최대 ``Max`` 까지 증가한다.

-  ``<NoCache> (기본: 5초, Ratio: 0, Max=5, MaxAge=0)``
   ``<Res2xx>`` 과 같거나 원래 서버가 no-cache 응답하는 경우에만 적용된다. ::

      cache-control: no-cache 또는 private 또는 must-revalidate

   ``MaxAge`` 가 0보다 크면 max-age를 줄 수있다.

   .. figure:: img/nocache_maxage.png
      :align: center

      Max-Age 만 클라이언트에서 Caching된다

-  ``<Res3xx> (기본: 300초)``
   원래 서버가 3xx로 응답 한 경우 TTL을 설정한다. Redirect 용도로 사용되는 경우가 많다.

-  ``<Res4xx> (기본: 30초)``
   원래 서버가 4xx에 응답 할 때 TTL을 설정한다.
   **404 Not Found** 경우가 많다.

-  ``<Res5xx> (기본: 30초)``
   원래 서버가 5xx 응답 한 경우 TTL을 설정한다. 소스 서버의 내부 결함 상태 인 경우가 많다.

-  ``<ConnectTimeout> (기본: 3초)``
   원래 서버에 연결할 수없는 경우 TTL을 설정한다. 내용이 이미 저장되어있는 경우 ``<ConnectTimeout>`` 초만 TTL을 연장한다. 컨텐츠가 저장되어 있지 않은 경우 ``<ConnectTimeout>`` 초 정도의 장애 상황에 응답한다. 이것은 장애 상황을 서비스한다는 의미가 아니라, TTL 시간 (아마도 장애 상황 일) 전 서버에 부담을주지 않기 때문이다.

-  ``<ReceiveTimeout> (기본: 3초)``
   연결은되었지만 데이터를 수신하지 않은 경우 TTL을 설정한다.
   ``<ConnectTimeout>`` 과 의미 적 동일하다.

-  ``<OriginBusy> (기본: 3초)``
   :ref:`origin-busysessioncount` 조건을 충족하면 원래 서버의 요청없이 만료 된 콘텐츠의 TTL을 설정된 시간만큼 연장한다. 이것은 원래 서버의 부하를 가중시키지 않기 때문이다.

.. note::

   TTL 값을 0으로 설정하면 서비스 직후 짧습니다. 만약 모든 요청에 ​​대해 원래 서버의 응답을주는 경우 우회 할 것을 권장한다.


.. _caching-policy-customttl:

Custom TTL
---------------------

URL에 대해 개별적으로 TTL을 설정한다. 명확한 URL 또는 패턴의 URL 매칭되는 콘텐츠마다 고정 된 TTL을 설정할 수있다. / svc / {가상 호스트 이름} /ttl.txt로 설정한다. ::

    # /svc/www.example.com/ttl.txt
    # 구분 기호는 쉼표 (,)이며, 시간의 단위는 초이다.

    *.jsp, 10
    /,5
    /index.html, 5
    /script/*.js, 300
    /image/ad.jpg, 1800


모든 페이지 (html, php, jsp 등)에 별도의 TTL을 설정하기 위해 * .html을 추가 하요토도라도 첫 페이지 (/)는 설정되지 않는다. 원래 서버가 첫 번째 페이지를 어떤 페이지 (예를 들어 index.php에 default.jsp 등)에 설정했는지 HTTP 프로토콜은 알 수 없다. 따라서 모든 페이지에 다른 TTL을 설정하려면 /를 추가 할 필요가있다.


TTL의 우선 순위
---------------------

적용 TTL 설정의 우선 순위를 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (생략) ...
    </TTL>

``<TTL>`` 의 ``Priority (기본: cc_nocache, custom, cc_maxage, rescode)`` 속성에 설정한다.

- ``cc_nocache`` 소스가 Cache-Control : no-cache 응답 한 경우
- ``custom`` `caching-policy-customttl`
- ``cc_maxage`` 소스가 Cache-Control에 maxage을 명시하는 경우
- ``rescode`` 원래 응답 코드 별 기본 TTL


이상 TTL 연장
---------------------

소스 서버의 종료에 대한 응답이 오지 않는 경우에는 장애의 판단이 명확하지만, 가끔 제대로 응답하고 장애 상황 인 경우가 발생한다. 예를 들어 콘텐츠를 저장하는 Storage와의 연결을 잃거나 뭔가 정상적인 처리가 불가능하다고 판단하는 경우가 있습니다. 전자의 경우 4xx 응답 (주로 **404 Not Found** ), 후자는 5xx 응답 (주로 **500 Internal Error** )를 받게된다.

그러나 이미 관련 콘텐츠가 저장되어있는 경우 텍스트 응답을 믿는 것보다 TTL을 연장시켜 서비스 전체에 장애가 발생하지 않도록하는 것이 효과적이다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTLExtensionBy4xx>OFF</TTLExtensionBy4xx>
    <TTLExtensionBy5xx>ON</TTLExtensionBy5xx>

-  ``<TTLExtensionBy4xx>``

   -  ``OFF (기본)`` 4xx 응답으로 콘텐츠를 업데이트한다.

   -  ``ON`` 304 not modified을받은 것처럼 동작한다.

의도 된 4xx 응답이 없는지주의해야한다.

-  ``<TTLExtensionBy5xx>``

   -  ``ON (기본)`` **304 Not Modified** 을받은 것처럼 동작한다.

   -  ``OFF`` 5xx응답으로 콘텐츠를 업데이트한다.

일반 서버라면 5xx 응답하지 않는다. 주로 서버의 일시적인 장애의 콘텐츠를 비활성화하여 원래의 부하를 가중시키지 않기위한 용도로 사용된다.。



.. _caching-policy-unvalidatable:

업데이트 보이지 않는 정책
---------------------

콘텐츠의 TTL은 만료되었지만, 원래 서버가 모두 제거되고 성공적으로 콘텐츠를 업데이트 (Revalidate) 할 수없는 경우 정책을 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <UnvalidatableObjectResCode>0</UnvalidatableObjectResCode>

-  ``<UnvalidatableObjectResCode>``

   -  ``0 (기본)`` 만료 된 콘텐츠의 TTL을 ``<ConnectTimeout>`` 만 연장한다.

   -  ``HTTP 응답코드`` 업데이트 할 수없는 경우에는 설정된 응답 코드로 응답한다. 
   
예를 들어, 다음과 같이 설정되어있는 경우 콘텐츠를 업데이트 할 수 없을 때 만료 된 컨텐츠를 404 Not Found로 응답한다.。 ::

   <UnvalidatableObjectResCode>404</UnvalidatableObjectResCode>



.. _caching-policy-renew:

업데이트 정책
====================================

TTL이 만료 콘텐츠의 경우 원본 서버에서 업데이트 여부를 확인 한 후 서비스가 이루어집니다.

   .. figure:: img/perf_refreshexpired.jpg
      :align: center

      변경 확인 후 응답

1. TTL이 효과적이다. 즉시 응답한다.

#. TTL이 만료 원래 서버에 변경 확인 (If-Modified-Since)을 요청한다. 변경의 확인이 될 때까지 클라이언트에 응답하지 않습니다.

#. 원래 서버에서 응답이 오면 TTL을 연장하거나 내용을 변경 (Swap)한다. 원본 서버에서 확인 되었기 때문에 클라이언트에 응답한다.

#. 변경 확인 된 내용이므로 다음의 TTL 만료시까지 즉시 응답한다.

고화질 동영상이나 게임과 같은 비교적 반응성보다 전송 속도가 중요한 서비스에서는이 방법이 큰 문제가되지 않는다. 대용량 데이터의 경우 원본 서버가 10 초에 업데이트 응답을 보낼 몇 분 정도 걸릴 때문에 비교적 원래의 반응성이 크게 중요하지 않다. 오히려 액세스 빈도가 높지 않은 내용이므로 반드시 업데이트 확인한다.

그러나, 쇼핑몰 같은 경우 상황은 다르다. Web 페이지가 빠르게로드되는 것이 무엇보다 중요하다. 1 ~ 2 초에서 클라이언트의 화면 구성이 모두이어야한다. 전송 속도보다 반응 속도가 더 중요하다는 것이다.

이 이때 TTL이 만료 원래 서버에 업데이트를 확인할 필요하다면 매우 큰 지연이 발생할 수있다. 일반 쇼핑몰 수백만 개의 콘텐츠를 동시에 서비스하는 것을 생각하면 항상 원래 서버에서 업데이트 확인 작업이 발생하고 있다고 생각해야한다. 자칫하면 원래 서버에 장애가 발생하거나 네트워크 장애가 발생하면 최악이다.

우리가 원하는 것은 어떤 소스 서버의 장애 나 지연에서 캐시 된 콘텐츠가 안전하게 전송 될 것이다.。

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center

      장애 두렵지 않다!

이러한 차이가 있기 때문에 백그라운드 콘텐츠 업데이트 기능이 개발되었다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <RefreshExpired>ON</RefreshExpired>

-  ``<RefreshExpired>``

   -  ``ON (기본)`` 변경 확인 후 응답한다.

   -  ``OFF`` 변경의 승인을 기다리지 않고 응답한다. 새로운 콘텐츠의 다운로드가 완료되면 변경 (Swap)한다.

``OFF`` 설정의 더 큰 이유는 콘텐츠가 거의 변경 빈도 없기 때문이다.

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center

      변경에 민감하지 않으면 기다리지 않는다.

위의 그림은 원래 서버의 업데이트 작업이 모두 백그라운드에서 진행되기 때문에 캐시 된 콘텐츠는 기다리지 않고 즉시 클라이언트에 서비스된다. 원본 서버가 **304 Not Modified** 응답하는 경우 TTL 만 연장된다. 파일이 업데이트 된 원본 서버에서 200 OK를 응답 한 경우 해당 파일이 완전히 다운로드 된 후, 파일이 원활하게 전환된다. 내용이 바뀌어도 이전 콘텐츠 (녹색)을 다운로드받은 사용자는 일반적으로 다운로드가 열린다. 파일 교환 후 액세스 된 사용자 (겨자 색상) 변경 한 파일에 서비스된다. 콘텐츠 업데이트, 네트워크 장애, 원래 서버 장애 등 어떠한 변수도 콘텐츠 업데이트는 백그라운드에서 이루어 지므로 실제 서비스에는 전혀 지연이 없다.


클라이언트 no-cache 요구 TTL 유효 기간이
---------------------

클라이언트의 HTTP 요청에 no-cache 설정이 여러 기재되어있는 경우 그 내용을 즉시 만료시킬 수있다. ::

    GET /logo.jpg HTTP/1.1
    ...
    cache-control: no-cache または cache-control:max-age=0
    pragma: no-cache
    ...

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <NoCacheRequestExpire>OFF</NoCacheRequestExpire>

-  ``<NoCacheRequestExpire>``

   -  ``OFF (기본)``  은 무시한다.

   -  ``ON``  TTL을 즉시 만료한다.

만료 된 컨텐츠는 `갱신정책`_ 에 따른다.


.. _caching-policy-accept-encoding:

Accept-Encoding 헤더
====================================

동일한 URL에 HTTP 요청에도 Accept-Encoding 헤더의 존재 유무에 따라 다른 콘텐츠를 캐시 할 수있다. 소스 서버에 요청을 보낼 때 압축 여부를 알 수 없다. 응답을 받았다하더라도 압축할지 여부를 매번 비교할 수도 없다.

   .. figure:: img/acceptencoding.png
      :align: center

      원래 서버가 어떤 해답을 줄 알 수 없다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <AcceptEncoding>ON</AcceptEncoding>

-  ``<AcceptEncoding>``

   -  ``ON (기본)`` HTTP 클라이언트가 보낸 Accept-Encoding 헤더를 인식한다.

   -  ``OFF`` HTTP 클라이언트가 보낸 Accept-Encoding 헤더를 무시한다.

원본 서버에서 압축을 지원하지 않거나 압축이 필요한 대용량 파일의 경우 ``OFF`` 로 설정하는 것이 바람직하다.


.. _caching-policy-casesensitive:

대소 문자 구분
====================================

소스 서버의 대문자와 소문자의 구별을 능동적으로 알 수 없다.

   .. figure:: img/casesensitive.png
      :align: center

      아마 그런 내용인지 404가 발생한다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <CaseSensitive>ON</CaseSensitive>

-  ``<CaseSensitive>``

   -  ``ON (기본)`` URL 대소 문자를 구문이다.

   -  ``OFF`` URL 대소 문자를 구별하지 않는다. 모두 소문자로 처리된다.


.. _caching-policy-applyquerystring:

QueryString 구분
====================================

QueryString에서 동적으로 생성 된 컨텐츠가없는 경우 QueryString을 인식하는 것은 불필요하다. 아무 의미없는 Random 값과 항상 변화하는 시간의 값이 매번 붙는 경우 원래의 엄청난 부하가 발생할 수있다.

   .. figure:: img/querystring.png
      :align: center

      동적 콘텐츠가없는 경우에는 같은 내용 일 가능성이 높다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <ApplyQueryString Collective="OFF">ON</ApplyQueryString>

-  ``<ApplyQueryString>``

   -  ``ON (기본)`` QueryString을 인식한다. 예외 조건에 만족하는 QueryString이 무시된다.

   -  ``OFF`` QueryString을 무시한다. 예외 조건에 만족하는 QueryString을 인식한다.

QueryString- 예외 조건 / svc / {가상 호스트 이름} /querystring.txt로 설정한다. ::

    # ./svc/www.example.com/querystring.txt

    /private/personal.jsp?login=ok*
    /image/ad.jpg

예외 조건이 ``<ApplyQueryString>`` 의 설정에 따라 의미가 다르다는 점에 유의한다. 명확한 URL 또는 패턴 ( * 만 가능)로 설정이 가능하다.

``Collective`` 특성 :ref:`api-cmd-purge` API를 호출 할 때 대상을 지정한다.

-  ``Collective``

   -  ``OFF (기본)`` 매개 변수 URL만을 대상으로하고있다.

   -  ``ON`` 매개 변수 URL뿐만 아니라 URL의 QueryString 존재하는 모든 콘텐츠를 대상으로 지정한다.

``Collective`` 속성이 ON이며, 파일이 많을수록 :ref:`api-cmd-purge` API 실행 CPU 부하가 높아진다. 관련 파일을 검색하는 시간이 길어질 수 있습니다 예기치 않은 문제가 발생할 수있다. 가급적 QueryString까지 붙은 명확한 URL을 :ref:`api-cmd-purge` API를 호출하는 것이 좋습니다.




Vary 헤더
====================================

Vary 헤더를 인식하여 콘텐츠를 구분한다. 일반적으로 Vary 헤더는 Cache 서버의 성능을 크게 떨어 뜨리는 원흉이다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <VaryHeader />

-  ``<VaryHeader>``

   원래 서버가 응답 한 Vary 헤더 지원 헤더의 목록을 설정한다. 구분 기호는 쉼표 (,)를 사용한다.

예를 들어 원본 서버가 다음과 같이 Vary 헤더를 보냈다해도 ``<VaryHeader>`` 가 설정되어 있지 않으면 무시한다. ::

    Vary: Accept-Encoding, Accept, User-Agent

User-Agent를 제외 Accept-Encoding Accept 헤더만을 인식하게하려면 다음과 같이 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <VaryHeader>Accept-Encoding, Accept</VaryHeader>

원본 서버가 보낸 모든 Vary 헤더를 인식하도록하려면 다음과 같이 설정한다.。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <VaryHeader>*</VaryHeader>


.. _caching-policy-post-method-caching:

POST 요청의 캐시
====================================

POST 요청을 Caching하도록 설정한다. POST 요청의 특성상 URL은 동일하지만 Body 데이터가 다를 수 있습니다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>

-  ``<PostRequest>``

   -  ``OFF (기본)``  POST 요청이 오면 세션을 종료한다.

   -  ``ON`` POST 요청을 Caching한다.

실제로 POST 요청을 처리하는 대부분의 경우 Body 데이터를 Caching-Key로 사용한다.
``BodySensitive`` 특성과 예외 조건을 사용하여 정교한 설정이 가능하다.

-  ``BodySensitive``

    -  ``ON (기본)`` Body 데이터까지 Caching-Key로 인식한다. 최대 길이는  ``MaxContentLength (기본: 102400 Bytes)`` 속성으로 제한한다. 예외 조건에 만족하는 Body 데이터를 무시한다.

    -  ``OFF`` Body 데이터는 무시한다. 예외 조건에 만족하는 Body 데이터를 인식한다.

POST 요청의 예외 조건은 / svc / {가상 호스트 이름} /postbody.txt로 설정한다. ::

    # /svc/www.example.com/postbody.txt

    /bigsale/*.php?nocache=*
    /goods/search.php

예외 조건이 ``BodySensitive`` 설정에 따라 의미가 다르다는 점에 유의한다. 명확한 URL 또는 패턴 ( * 만 허용)로 설정이 가능하다.

이 설정은 :ref:`bypass-getpost` 과 정책적 혼란 스러울 수있다.
``<BypassPostRequest> (기본: ON)`` 에 의해 POST 요청이 캐시되지 않을 수 있습니다. 따라서 POST 요청을 캐시하기 위해서는 ``<BypassPostRequest>`` 를 ``OFF`` 또는 예외 조건을 설정할 필요가있다. 정리하면, 우선 순위는 다음과 같다.

* 우회 조건( :ref:`bypass-getpost` )에 만족 한 경우 원래 서버에 우회.
* Content-Length 헤더가없는 경우 연결을 종료한다.
* ``PostRequest`` 가  ``ON`` 으로 설정되어 있으며, Content-Length가 ``MaxContentLength`` 속성 값을 넘지 않으면 캐시 모듈에 의해 처리된다.
* 이상의 시나리오에서는 처리되지 않은 요청은 종료한다.

.. note::

    ``MaxContentLength`` 특성을 크게 설정하면 Caching-Key 관리에 많은 메모리가 필요하다. 가능한 한 작게 설정하는 것이 좋다.
