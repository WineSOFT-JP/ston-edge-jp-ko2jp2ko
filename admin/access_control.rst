.. _access-control:

제 15 장 액세스 제어
******************

이 장에서는 불필요한 클라이언트에 대한 액세스를 차단하는 방법을 설명한다. 액세스 블록은 일반적으로 ACL (Access Control List)에 차단 목록 (Black-list)을 만들지 만 설정 편의상 허용 목록 (White-list)을 만들 수도있다.

액세스 제어 액세스 단계에서 실행하는 서버에 대한 액세스 제어 및 가상 호스트마다 설정하는 가상 호스트의 액세스 제어로 나뉜다. 레벨마다 관점과 판단 기준이 다르므로 효과적인 차단 시점을 결정해야한다. 액세스 제어 동작은 모두 기록된다.

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

서버 액세스 제어
====================================

클라이언트가 서버에 접속 한 순간 IP 정보를 사용하여 차단 여부를 결정한다. 연결 단계에서 처리되기 때문에 가장 확실 빠르다. 전역 설정 (server.xml)로 설정하고 가장 높은 우선 순위를 갖는다. ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``
   IP 기반 ACL을 설정한다. IP, IP Range, Bitmask, Subnet 이상 네 가지 형식을 지원합니다. 순서를 인식하고 부모에 설정된 표현이 우선한다.
   ``Default (기본: Allow)`` 특성이 일치하는 조건이없는 경우의 처리 방법이다. 이 속성을 ``Deny`` 로 설정하면 하위에 ``<Allow>`` 에서 허용하는 조건을 지정해주어야한다.

차단 된 IP는 :ref:`admin-log-deny` 에 기록된다.


.. _access-control-geoip:

GeoIP
====================================

GeoIP를 사용하여 국가 별 접근을 차단할 수있다.
`GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_ 중 Binary Databases를 `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ 에 연결되어 실시간으로 변경 사항을 반영한다. ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>

``<ServiceAccess>`` 의 ``GeoIP`` 특성에 GeoIP Databases 경로를 설정한다. 국가 코드는 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ 와
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_  를 지원한다.

.. note::

   GeoIP는 파일 이름이 예약되어 있기 때문에 반드시 저장된 로컬 경로를 입력하도록 설정한다. 또한 자동으로 반영되므로 별도 설정을 Reload하고있다.


GeoIP가 설정되어있는 경우는 그 디렉토리에 저장된 파일 목록을 표시한다. 설정되어 있지 않은 경우, 404 NOT FOUND에 응답한다. ::

   http://127.0.0.1:10040/monitoring/geoiplist

결과는 JSON 형식으로 제공된다. ::

   {
       "version": "2.0.0",
       "method": "geoiplist",
       "status": "OK",
       "result":
       {
           "path" : "/usr/ston/geoip/",
           "files" :
           [
               {
                   "file" : "GeoIP.dat",
                   "size" : 766255
               },
               {
                   "file" : "GeoLiteCity.dat",
                   "size" : 12826936
               }
           ]
       }
   }


.. _access-control-vhost:

가상 호스트의 액세스 제어
====================================

가상 호스트마다 서비스 허용 / 차단 / redirect를 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <AccessControl Default="Allow" DenialCode="401">OFF</AccessControl>

-  ``<AccessControl>``

   - ``OFF (기본)`` ACL이 활성화되어 있지 않다. 모든 클라이언트의 요청을 허용한다.

   - ``ON`` ACL이 활성화된다. 차단 된 요청에는 ``DenialCode`` 속성에 설정된 응답 코드로 응답한다.
     ``Default (基本: Allow)`` 속성이 ``Allow`` 이면 ACL은 거부된다. 반대로 ``Deny`` 이라면 ACL은 허용 목록된다.

구체적인 액세스 제어 목록은 / svc / {가상 호스트 이름} /acl.txt로 설정한다.


.. _access-control-vhost_allow_deny:

허용 / 거부
---------------------

모든 클라이언트의 HTTP 요청에 대해 허용 / 거부할지 여부를 설정한다. Deny 된 요청은 :ref:`admin-log-access` 에 TCP_DENY에 기록된다.

각 조건에 대해 개별적으로 응답 코드를 설정할 수도있다. ::

   # /svc/www.example.com/acl.txt
   # 구분 기호는 쉼표 (,)이며, {조건}, {allow 또는 deny} 순으로 표기한다.
   # deny의 경우 키워드 뒤에 응답 코드를 지정할 수있다.
   # 지정하지 않으면 ``<AccessControl>`` 의 ``DenialCode`` 을 사용한다.
   # n 개의 조건을 결합 (AND)하기 위해서는 &를 사용한다.

   $IP[192.168.1.1], allow
   $IP[192.168.2.1-255]
   $IP[192.168.3.0/24], deny
   $IP[192.168.4.0/255.255.255.0]
   $IP[AP] & !HEADER[referer], allow
   $HEADER[cookie: *ILLEGAL*], deny, 404
   $HEADER[via: Apache]
   $HEADER[x-custom-header]
   !HEADER[referer] & !HEADER[user-agent] & !HEADER[host], deny
   $URL[/source/public.zip], allow
   $URL[/source/*]
   /profile.zip, deny, 500
   /secure/*.dat

허용 / 차단 조건은 IP, GeoIP, Header, URL 네 가지로 설정이 가능하다.

-  **IP**

   $IP[...]로 표기 IP, IP Range, Bitmask, Subnet 4 가지를 지원합니다.

-  **GEOIP**

   $IP[...]로 표기하고 반드시 GeoIP 설정되어 동작한다.

-  **HEADER**

   $HEADER[Key : Value]로 표기한다. 
   Value는 명확한 표현과 패턴을 인식한다. 
   $HEADER [Key :]와 같이 구분 기호가 Value가 빈 문자열이면 요청 헤더의 값이 비어있는 경우를 의미한다. 
   $HEADER [Key]와 같이 구분없이 Key 만 명시되어있는 경우 Key에 대응하는 헤더의 존재 유무를 조건으로 판단한다.

-  **URL**

   $URL[...]로 표기 생략이 가능하다. 명확한 표현과 패턴을 인식한다.

$는 "조건에 맞는다면 ~하는"을 의미하지만! 는 "조건에 맞지 않으면 ~ 할"을 의미한다. 다음과 같이 부정 조건으로 지원한다. ::

   # 국가가 JPN이없는 경우 deny한다.
   !IP[JPN], deny

   # referer 헤더가 존재하지 않는 경우 deny한다.
   !HEADER[referer], deny

   # / secure / 빠스사부가없는 경우 allow한다.
   !URL[/secure/*], allow



.. _access-control-vhost_redirect:

Redirect
---------------------

모든 클라이언트의 HTTP 요청에 대해 Redirect 여부를 설정한다. Redirect 된 요청에는 **302 Moved temporarily로** 응답한다. ::

   # /svc/www.example.com/acl.txt
   # 구분 기호는 쉼표 (,)이며, {조건}, {redirect} 순으로 표기한다.
   # redirect의 경우 키워드 다음에 이동할 URL을 지정한다. (Location 헤더의 값에 명시)

   $IP[GIN], redirect, /page/illegal_access.html
   $HEADER[referer:], redirect, http://another-site.com
   !HEADER[referer], redirect, http://example.com#URL

Redirect 할 때 클라이언트가 요청한 URL이 필요할 수있다. 이런 경우 ``#URL`` 키워드를 사용한다.

HTTPS 만 지원하는 서비스의 경우 HTTP 요청에는 다음과 같이 ``$PROTOCOL[HTTP]`` 조건에서 HTTPS를 강제하도록 redirect시킬 수있다. ::

   $PROTOCOL[HTTP], redirect, https://example.com#URL

