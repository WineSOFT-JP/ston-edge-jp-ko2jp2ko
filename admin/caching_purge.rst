.. _caching-purge:

제 5 장 Caching 해제
******************

이 장에서는 Caching 된 콘텐츠를 해제하는 방법을 설명한다. 업계 용어로 Purge로 통칭가 다양한 상황이나 환경에 따라 세분화 된 API가 필요하다.

소스에서 캐시 된 콘텐츠는 :ref:`caching-policy-ttl` 에 따라 업데이트주기를 가진다. 하지만 확실히 내용이 변경되어 관리자가이를 즉시 적용하려면 :ref:`caching-policy-ttl` 이 만료 될 때까지 기다릴 필요는 없다.
`Purge`_ / `Expire`_ / `HardPurge`_ 등을 사용하면 신속하게 콘텐츠를 무효화시킬 수있다.

비활성화 API는 단순히 브라우저에 의해 호출되는 경우도 있지만, 자동화되는 경우가 많다. 예를 들어 FTP를 통해 파일의 업로드가 완료되면 즉시 `Purge`_ 를 호출 식이다. 관리자는 다음과 같이 몇 가지 동작을 설정할 수있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>

-  ``<Purge2Expire> (기본: NONE)``

   `Purge`_ 요구를 설정에 따라 `Expire`_ 처리한다. 예를 들어, 특정 패턴(*.jpg)을 `Purge`_ 하는 경우 의도하지 않게 많은 컨텐츠가 삭제되고 소스에 과도한 부하를 발생시킬 수있다. 이러한 경우 `Expire`_ 처리하도록 설정하면 과도한 소스 부하를 방지 할 수있다.
   
   - ``NONE`` `Expire`_ 처리하지 않는다.
   - ``ROOT`` 도메인 전체 (/ *)의 `Purge`_ 를 `Expire`_ 처리한다.
   - ``PATTERN`` 모든 패턴 `Purge`_ 를 `Expire`_ 처리한다.
   - ``ALL`` 모든 `Purge`_ 를 `Expire`_ で処理する。

-  ``<RootPurgeExpire> (기본: ON)``
   
   전체 콘텐츠에 대한 의도하지 않은 `Purge`_ / `Expire`_ 는 과도한 원래 서버의 부하를 발생시킬 수있다. 이 설정을 통해 모든 콘텐츠의 `Purge`_ / `Expire`_ 을 차단할 수있다. 이 설정은 ``<Purge2Expire>`` 보다 우선합니다.
   
   - ``ON`` `Purge`_ / `Expire`_ 을 가능하게한다.
   - ``PURGE`` `Purge`_ 만 허용한다.
   - ``EXPIRE`` `Expire`_ 만 허용한다.
   - ``OFF`` 모든 `Purge`_ / `Expire`_ 을 금지한다.

-  ``<ResCodeNoCtrlTarget> (기본: 200)``

   `Purge`_ , `Expire`_ , `HardPurge`_ , `ExpireAfter`_ 의 대상 객체가 존재하지 않을 때의 HTTP 응답 코드를 설정한다.
   

타겟팅은 URL 패턴 2 가지로 표현한다. ::

   example.com/logo.jpg      // URL
   example.com/img/          // URL
   example.com/img/*.jpg     // 패턴
   example.com/img/*         // 패턴
   
명확한 URL을 다른 패턴 ( * .jpg)로 무효화가 가능하다. 그러나 작업을 수행 할 때까지 대상 수를 명확하게 알 수 없다. 이것은 자칫하면 관리자의 의도와는 다르기 때문에 많은 대상을 지정할 수있다. 이것은 실제로 CPU 리소스를 많이 소비하게 전체 시스템에 부담을 줄 수있다.

따라서 실제 서비스에 명확한 URL 만 사용하는 것이 좋습니다한다. 패턴 표현은 서비스에서 배제 된 상태에서 관리 목적으로 사용하기 때문이다.


.. note::

   보안상의 이유로 example.com/files/ 등의 특정 디렉토리에 액세스은 403 FORBIDDEN 등으로 차단된다. 그러나 루트 디렉토리는 예외를 가진다. 예를 들어, 사용자가 example.com에 액세스하면 브라우저는 루트 디렉토리 (/)를 요청한다. ::
   
      GET / HTTP/1.1
      Host: example.com
   
   이에 대해 Web Server는 관리자가 설정 한 기본 페이지 (아마 index.html 또는 index.htm)로 응답한다. 물론, Web 서비스의 구성은 루트 디렉토리 (/) 디렉토리가 아닌 페이지에서 동작한다.

하지만 Cache 서버 루트 디렉토리 (/)에 접속했는데, 200 OK 페이지가 온 이해한다. 또한 원래 서버가 어떤 페이지를 응답 한 모른다. 간단하게 정리하면 Cache 서버의 관점에서는 디렉토리 표현도 URL의 일종에 불과하다. ::
   
      example.com/img/          // example.com 가상 호스트의 / img /를 방문한 결과 페이지
      example.com/              // example.com의 가상 호스트의 기본 페이지 (/)
      example.com/img/*         // example.com 가상 호스트의 / img 디렉토리 및 하위 페이지
      example.com/*             // example.com example.com 가상 호스트의 모든 콘텐츠
         


.. toctree::
   :maxdepth: 2


.. _api-cmd-purge:

Purge
====================================

대상 콘텐츠를 해제시켜 원래의 서버에서 콘텐츠를 다시 다운로드 받게한다. Purge 후 처음 액세스 할 때 원본 서버에서 콘텐츠를 다시 캐시한다. 만약 원래 서버에 장애가 발생하여 콘텐츠를 검색 할 수없는 경우에는 비활성화 된 콘텐츠를 복원시켜 서비스에 장애가 없도록 처리한다. 이렇게 복원 된 콘텐츠는 해당 시점에서 ConnectTimeout 설정 만 뒤에 업데이트한다. ::

    http://127.0.0.1:10040/command/purge?url=...
    
대상 콘텐츠는 URL 패턴으로 지정할 수있을뿐만 아니라, "|"(Vertical Bar)를 구분 기호를 사용하여 여러 도메인에 여러 대상을 지정할 수있다. 만약 도메인 이름이 생략 된 경우, 최근 사용 된 도메인을 사용한다. ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
결과는 JSON 형식으로 제공된다. 대상 콘텐츠 수 / 용량과 처리 시간 (단위 : ms)이 명시된다. 이미 Purge 된 내용은 다시 Purge되지 않는다. ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
``<Purge2Expire>`` 을 통해 특정 조건의 Purge를 Expire에서 작동하도록 설정할 수있다. 결과없는 응답에는 ``<ResCodeNoCtrlTarget>`` 에서 HTTP 응답 코드를 설정할 수있다.


.. note::
   
   원본 서버가 장애로 인해 모두 제거 된 경우 콘텐츠를 업데이트 할 수 없기 때문에 Purge가 작동하지 않습니다.
   

.. _api-cmd-expire:
   
Expire
====================================

대상 콘텐츠의 TTL을 즉시 만료시킨다. Expire 후 처음 액세스 할 때 원본 서버에서 변경 여부를 확인한다. 변경되지 않은 경우 TTL 연장이있는만큼 콘텐츠의 다운로드가 발생하지 ::

    http://127.0.0.1:10040/command/expire?url=...
    
다른 모든 동작은 `Purge`_ 과 같다.


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

대상 콘텐츠의 TTL 유효 기간을 현재 (API 호출 시점)에서 입력 된 시간 (초) 만 뒤에 설정한다. ExpireAfter 만료 앞당겨 콘텐츠를보다 신속하게 업데이트하거나 반대로 유효 기간을 늘려 소스 서버의 부하를 줄일 수있다. ::

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

함수 호출의 사양은 `Purge`_ / `Expire`_ 와 비슷하지만 sec 매개 변수 (단위 : 초)를 사용하여 TTL 만료를 지정할 수있다. sec를 생략하면 기본값은 1 일 (86400 초)으로 설정되고 0을 입력하면 실패한다. 결과는 `Purge`_ / `Expire`_ 과 같지만 원래 서버의 장애의 유무에 관계없이 작동합니다. 결과없는 응답에는 ``<ResCodeNoCtrlTarget>`` 에서 HTTP 응답 코드를 설정할 수있다.

.. note::
   ExpireAfter는 캐시 된 콘텐츠의 현재 유효 기간 만 설정하면 사용자 TTL과 설정된 기본 TTL을 변경하는 API는 없다. ExpireAfter 호출하면 캐시 된 콘텐츠는 영향을받지 않는다.

   url 매개 변수를 먼저 입력 한 경우 sec 매개 변수 url 매개 변수의 QueryString으로 인식 될 수있다. 따라서 sec 매개 변수를 먼저 입력되어있는 것이 안전하다.
   
   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ 이상의 API는 원래 서버의 장애 상황에서도 콘텐츠가 사라지지 않고 제대로 작동합니다. 그러나 HardPurge 콘텐츠의 완전한 제거를 의미한다. HardPurge 가장 강력한 제거 방법이 삭제 된 콘텐츠는 원래 서버에 장애가 발생해도 복귀 할 수 없다. 결과없는 응답에는 ``<ResCodeNoCtrlTarget>`` 에서 HTTP 응답 코드를 설정할 수있다. ::

    http://127.0.0.1:10040/command/hardpurge?url=...


Purge의 기본 동작
====================================

Purge API를 호출하면 콘텐츠의 회복 여부를 선택한다. ::

   # server.xml - <Server><Cache>
   
   <Purge>Normal</Purge>
      
-  ``<Purge>`` 
   
   - ``Normal (기본)`` `Purge`_ で動作する。 에서 동작한다. (원래 재해 복구)
   
   - ``Hard`` `HardPurge`_ 에서 동작한다. (원래 재해 복구되지 않음)


HTTP Method
====================================

비활성화 API를 확장 HTTP Method를 호출 할 수 있습니다. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP Method는 기본적으로 Manager 포트와 서비스 (80) 포트로 동작합니다. 서비스 포트에 요구되는 HTTP Method의 :ref:`env-host` 에서 설정한다.


.. _api-etc-post:

POST 규격
====================================

비활성화 API를 다음과 같이 POST로 호출 할 수 있습니다. ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    

