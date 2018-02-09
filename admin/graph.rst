.. _api-graph:

Appendix A: Graph
******************

모든 MRTG 통계는 PNG 포맷의 그래프로 제공된다. 호출 규칙은 자원 후 단위가 붙는 형식이다. ::

    # 5つのCPUグラフ (dash, day, week, month, year)
    http://127.0.0.1:10040/graph/cpu_dash.png
    http://127.0.0.1:10040/graph/cpu_day.png
    http://127.0.0.1:10040/graph/cpu_week.png
    http://127.0.0.1:10040/graph/cpu_month.png
    http://127.0.0.1:10040/graph/cpu_year.png

모든 그래프는 5 가지 유형으로 제공된다.

============== ================= ==================== =================
유형           크기             시간 단위               기간
============== ================= ==================== =================
dash           205 X 175         5분                   12시간
day            580 X 203         5분                   2일 (48시간)
week           580 X 203         30분                  2주 (14일)
month          580 X 203         2시간                  7주
year           580 X 203         1일                  18개월
============== ================= ==================== =================

그래프는 적어도 1 개에서 최대 3 개의 선이 그려진다. Main 라인은 녹색, Sub 줄은 파란색으로 그려진다. 또한 "Week"그래프 이상에서 Peak 라인이 제공된다. Peak 라인은 이전 단위의 최대 수치를 핑크로 그린다.

.. note::

   너무 많은 그래프를 동시에 그릴 때 CPU 사용률이 지나치게 높아지면서 서비스 품질의 저하가 발생할 수있다. 이를 방지하기 위해 항상 한 번에 하나의 그래프 만 그리도록 관리한다.


.. toctree::
   :maxdepth: 2


.. _api-graph-global:

글로벌 자원
====================================

글로벌 자원 그래프는 시스템의 상태와 STON위한 리소스 서비스한다. 다음 표에서 * 는 유형 (dash, day, week, month, year) 중 하나를 의미한다.



CPU
---------------------
::

    /graph/cpu_*.png

-  ``Main`` Kernel + User
-  ``Sub`` Kernel



STON 미디어 서버 CPU
---------------------
::

    /graph/ston_media_server_cpu_*.png

-  ``Main`` Kernel + User
-  ``Sub`` Kernel



메모리
---------------------
::

    /graph/mem_*.png

-  ``Main`` 전체 사용량
-  ``Sub`` STON 미디어 서버의 사용률



IO Wait
---------------------
::

    /graph/iowait_*.png

-  ``Main`` IO Wait



Load Average
---------------------
::

    /graph/loadavg_*.png

-  ``Main`` Load Average



서버 소켓 이벤트 (클라이언트 -> STON)
---------------------
::

    /graph/ssockevent_*.png

-  ``Main`` Accepted
-  ``Sub`` Closed



서버 소켓 사용량 (클라이언트 -> STON) 
---------------------
::

    /graph/ssockusage_*.png

-  ``Main`` 전체
-  ``Sub`` Established



클라이언트 소켓 이벤트 (STON -> 소스 서버)
---------------------
::

    /graph/csockevent_*.png

-  ``Main`` Connected
-  ``Sub`` Closed



클라이언트 소켓 사용량 (STON -> 소스 서버)
---------------------
::

    /graph/csockusage_*.png

-  ``Main`` 전체
-  ``Sub`` Established



차단 된 IP 액세스
---------------------
::

    /graph/acldenied_*.png

-  ``Main`` 차단 된 클라이언트



이벤트 큐
---------------------
::

    /graph/eq_*.png

-  ``Main`` 이벤트 큐의 길이



쓰기 대기
---------------------
::

    /graph/wf2w_*.png

-  ``Main`` 쓰기 대기중인 파일 수


.. _api-graph-urlrewrite:

URL 전처리 성공
---------------------
::

    /graph/urlrewrite_*.png

-  ``Main`` 전처리 된 URL 수



TCP 소켓
---------------------
::

    /graph/tcpsocket_*.png

.. figure:: img/graph_tcpsocket_detail.png



.. _api-graph-vhost:

가상 호스트
====================================

가상 호스트의 그래프는 전체 또는 개별 가상 호스트 상태에 서비스한다. vhost 매개 변수를 사용하여 특정 가상 호스트를 지정할 수 있으며, 생략 된 경우 전체 가상 호스트의 합계를 제공한다. ::

    http://127.0.0.1:10040/graph/vhost/mem_day.png?vhost=example.com

다음 표에서 * 는 유형 (dash, day, week, month, year) 중 하나를 의미한다.



적중률
---------------------
::

    /graph/vhost/hitratio_*.png

-  ``Main`` Request Hit Ratio
-  ``Sub`` Byte Hit Ratio



콘텐츠 수
---------------------
::

    /graph/vhost/filecount_*.png

.. figure:: img/graph_filecount_detail.png



컨텐츠 메모리
---------------------
::

    /graph/vhost/mem_*.png

-  ``Main`` 메모리에로드 된 컨텐츠 데이터의 양



삭제 대기
---------------------
::

    /graph/vhost/wf2d_*.png

-  ``Main`` 삭제 대기중인 파일 수



클라이언트 우회
---------------------
::

    /graph/vhost/client_httpreq_bypass_*.png

-  ``Main`` 무시 된 클라이언트의 HTTP 요청



클라이언트의 요청을 차단
---------------------
::

    /graph/vhost/client_httpreq_denied_*.png

-  ``Main`` 차단 된 클라이언트 요청



클라이언트 세션
---------------------
::

    /graph/vhost/client_http_session_*.png

-  ``Main`` 전체 클라이언트 세션
-  ``Sub`` 전송 진행중인 클라이언트 세션



클라이언트의 트래픽
---------------------
::

    /graph/vhost/client_traffic_*.png

-  ``Main`` Inbound
-  ``Sub`` Outbound



클라이언트의 응답
---------------------
::

    /graph/vhost/client_http_res_*.png

-  ``Main`` 클라이언트 HTTP 응답의 수
-  ``Sub`` 클라이언트의 HTTP 요청 수



클라이언트에 대한 자세한 응답
---------------------
::

    /graph/vhost/client_http_res_detail_*.png

.. figure:: img/graph_rescode_detail.png



클라이언트 트랜잭션의 완료
---------------------
::

    /graph/vhost/client_http_res_complete_*.png

-  ``Main`` 완료 클라이언트 HTTP 응답의 수
-  ``Sub`` 클라이언트의 HTTP 요청 수



클라이언트의 응답 시간
---------------------
::

    /graph/vhost/client_http_res_time1_*.png

-  ``Main`` 클라이언트 요청의 HTTP 응답 시간



클라이언트 완료 시간
---------------------
::

    /graph/vhost/client_http_res_time2_*.png

-  ``Main`` 클라이언트 요청의 HTTP 트랜잭션의 완료 시간



클라이언트 캐시 응답
---------------------
::

    /graph/vhost/client_http_res_hit_*.png

.. figure:: img/graph_filehit.png



클라이언트 SSL 트래픽
---------------------
::

    /graph/vhost/client_traffic_ssl_*.png

-  ``Main`` Inbound
-  ``Sub`` Outbound



소스 서버 세션
---------------------
::

    /graph/vhost/origin_http_session_*.png

-  ``Main`` 전체 원본 세션
-  ``Sub`` 전송 진행중인 소스 세션



소스 서버의 트래픽
---------------------
::

    /graph/vhost/origin_traffic_*.png

-  ``Main`` Inbound
-  ``Sub`` Outbound



소스 서버의 응답
---------------------
::

    /graph/vhost/origin_http_res_*.png

-  ``Main`` 원래 HTTP 응답의 수
-  ``Sub`` 원래 HTTP 요청의 수



소스 서버에 대한 자세한 응답
---------------------
::

    /graph/vhost/origin_http_res_detail_*.png

.. figure:: img/graph_rescode_detail.png



소스 서버 트랜잭션의 완료
---------------------
::

    /graph/vhost/origin_http_res_complete_*.png

-  ``Main`` 완료 소스 서버 HTTP 응답의 수
-  ``Sub`` 원본 서버의 HTTP 요청 수



원래 서버의 응답 시간
---------------------
::

    /graph/vhost/origin_http_res_time1_*.png

-  ``Main`` 원본 서버로 전송되는 요청의 HTTP 응답 시간



소스 서버 완료 시간
---------------------
::

    /graph/vhost/origin_http_res_time2_*.png

-  ``Main`` 원본 서버로 전송되는 요청의 HTTP 트랜잭션의 완료 시간
