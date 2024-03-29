- **21강 : 검색 기능 평가하기 - 466p~**
    - 운용하는 시스템에서 내부 검색 기능을 제공할 경우, 사용자가 어떤 검색쿼리를 입력하고 어떤 결과를 얻는지 분석하는 작업이 매우 중요하다.
    - 이번 절에서는 검색 관련 행동 로그와 평가전용 데이터를 통해 검색기능을 정량적으로 평가하고 개선하는 방법을 소개한다.
    
    🔎  검색하는 사용자의 행동(그림 21-1)
    
    - 검색하는 사용자의 **행동 패턴** 정의
        - 사용자가 **특정 검색 쿼리**를 입력하면
        - 사용자는 해당 **검색 결과**를 **출력**하는 화면으로 이동
            - 사용자는 검색 결과로 `원하는 정보`가 나오면, 해당 정보의 **상세 화면**으로 이동
            - 원하는 정보가 없다면, `다시 검색` or `이탈`
    
    - 사용자가 무엇을 검색하고
        - 그 검색 결과에 대해 **어떤 행동**을 취하는지 구분이 가능하다면
        - 검색 기능을 담당하고 있는 엔지니어에게 기능 개선 요청이 가능
    
    ### **검색 기능 개선 방법**
    
    - 1.검색 키워드의 **흔들림**을 흡수할 수 있도록 **동의어 사전 추가**
        - 사용자가 언제나 상품 이름을 `정확하게 입력하지 않음`
            - 상품의 단축된 이름 또는 별칭을 사용하는 경우 존재
            - 알파벳 이름을 한글로 검색 등
        - 검색의 흔들림을 흡수하려면
            - 공식 상품 이름과 동일한 의미를 가질 수 있는 단어를 **동의어 사전**에 추가해야 함
        
    - 2.검색 키워드를 **검색 엔진**이 이해할 수 있게 **사용자 사전 추가**
        - **검색 엔진**에 따라서는
            - 단어를 **독자적인 알고리즘**으로 분해하고,
            - 이를 기반으로 **인덱스**를 만들어 데이터 검색에 활용
        - 위와 같이 작업할 경우
            - **서비스 제공 측**이 의도한대로, 단어가 분해된다는 보장이 없음
        - 예를 들어 `위스키`라고 검색했으나,
            - `위`와 `스키`로 분해할 수 있음
        - 이런 결과를 막기 위해 `위스키`를 하나의 단어로 **사용자 사전**에 추가
    - 3.검색 결과가 사용자가 원하는 순서대로 나오도록 **정렬 순서 조정**
        - **검색 엔진**은 조건에 맞는 아이템을 **출력**할 뿐만 아니라
            - 검색 쿼리와의 연관성을 **수치화**하여
            - 점수가 높은 순서대로 정렬해주는 **순위 기능**을 가짐
        - **관련도 점수**를 계산하려면
            - 검색 키워드와 아이템에 포함된 문장의
                - `출현 위치`와 `빈도`
                - `아이템의 갱신 일자`, `접속 수`등의 다양한 요소를 조합해야 함
        - 이처럼 데이터를 조합하여 **최적의 순서**로 결과를 출력하려면
            - 적절한 평가 지표와 방법이 있어야 함
    
    <aside>
    🔎 사용할 테이블 : 데이터 21-1 : 검색 결과와 상세페이지 접근로그(acces_log) 테이블
    
    - 검색 결과 페이지가 많아서, 여러 페이지가 나올 경우
        - 두 번째 이후 페이지를 눌러도, 하나의 레코드만 로그로 저장하게 데이터를 가공
        - 이는, `검색 결과의 두번째 페이지를 열람`하는 것과 **새로운 검색**을 구분하기 위함
    - 텍스트 박스에 **검색 키워드**를 입력하여 검색하는
        - Free Keyword Search를 전제로 **리포트**와 **쿼리** 소개
    
    ![스크린샷 2022-05-01 16.04.31.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2779c885-700f-4608-9be6-f154bb3796e4/스크린샷_2022-05-01_16.04.31.png)
    
    </aside>
    
    - 1.NoMatch 비율과 키워드 집계하기 - 469p
        - 사용자가 검색했을 때, 원하는 결과가 나오지 않으면 `부정적인 인상`을 받음—>**이탈할 가능성** 존재
        - 검색 엔진이 **키워드**를 제대로 파악하지 못해, 흔들림을 흡수하지 못했다면
            - 여러 가지 대책을 통해 결과가 나오도록 변경해야 함 —> 검색 결과가 `0`인 비율(`NoMatch 비율`)을 집계하는 것이 좋음
        
        ### **NoMatch 비율 집계하기**
        
        - NoMatch 비율 :  `검색 총 수` 중에서 `검색 결과를 0`으로 리턴하는 검색 결과 비율
        - 공식
            - `NoMatch 비율 = 검색 결과가 0인 수(NoMatch 수) / 검색 총 수`
        
        ### **CODE.21.1 NoMatch 비율을 집계하는 쿼리**
        
        ```sql
        SELECT
          substring(stamp, 1, 10) AS dt
         , COUNT(1) AS search_count
         , SUM(CASE WHEN result_num = 0 THEN 1 ELSE 0 END) AS no_match_count
         , AVG(CASE WHEN result_num = 0 THEN 1.0 ELSE 0.0 END) AS no_match_rate
        FROM
          access_log
        WHERE
          action = 'search'
        GROUP BY
          dt
        ;
        ```
        
        ### 결과
        
        ![스크린샷 2022-05-01 16.52.54.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae1815d7-5248-4893-9b52-96ce6aa3d9c6/스크린샷_2022-05-01_16.52.54.png)
        
        ### **NoMatch 키워드 집계하기**
        
        - **CODE.21.2**는 `NoMatch`가 될 때의 **검색 키워드**를 추출하는 쿼리
        - 어떤 키워드가 `0`개의 결과를 내는지 집계한 뒤,
            - 동의어 사전과 사용자 사전의 추가하다고 판단되면, 이를 추가하여 검색 결과 개선
        
        ### **CODE.21.2. NoMatch 키워드를 집계하는 쿼리**
        
        - `PostgreSQL`, `Hive`, `Redshift`, `BigQuery`, `SparkSQL`
            
            ```sql
            WITH
            search_keyword_stat AS (
              -- 검색 키워드 전체 집계 결과
              SELECT
                keyword
                , result_num
                --키워드 별 검색 총 량
                , COUNT(1) AS search_count 
                --키워드 별 검색 총 량 / 전체 검색 량 = 키워드 별 검색 백분율
                , 100.0 * COUNT(1) / COUNT(1) OVER() AS search_share 
              FROM
                access_log
              WHERE
                action = 'search'
              GROUP BY
                keyword, result_num
            )
            -- NoMatch 키워드 집계 결과
            SELECT
              keyword
              , search_count
              , search_share
              -- 키워드 별 NoMatch 백분율 = 키워드별 NoMatch 수/ 총 NoMatch 수
              , 100.0 * search_count / SUM(search_count) OVER() AS no_match_share
            FROM
              search_keyword_stat
            WHERE
              -- 검색 결과가 0개인 키워드만 추출
              result_num = 0 ⭐️⭐️⭐️⭐️⭐️⭐️⭐️
            ```
            
            (with 부문)
            
            ![스크린샷 2022-05-01 17.24.53.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5c720183-8a1d-4bd6-99dd-f6ecf3a7d53d/스크린샷_2022-05-01_17.24.53.png)
            
            ### 결과
            
            ![스크린샷 2022-05-01 17.04.03.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/463dcc47-1354-4f2b-b995-20055ed57b46/스크린샷_2022-05-01_17.04.03.png)
            
        
    - 2.재검색 비율과 키워드 집계하기 - 472p
        - 검색 결과에 만족하지 못해, **새로운 키워드**로 검색한 사용자의 행동은
            - 검색을 어떻게 **개선**하면 좋을지 좋은 지표가 됨
        
        ### 1.**재검색 비율 집계하기**
        
        - **재검색 비율**이란
            - 사용자가 검색 결과의 **출력**과 관계 없이
            - 어떤 결과도 클릭하지 않고, 새로 검색을 **실행한 비율**을 나타냄
            
        - 검색 결과 출력 로그(`action=search`)와 검색 화면에서
            - 상세 화면으로의 이동 로그(`action=detail`)를 시계열 순서로 나열해서
            - 각각의 줄에 다음 줄의 액션을 기록
            
        
        ### **CODE.21.3. 검색 화면과 상세 화면의 접근 로그에 다음 줄의 액션을 기록하는 쿼리**
        
        ```sql
        WITH
        access_log_With_next_action AS (
          SELECT
            stamp
            , session
            , action
            , LEAD(action) ⭐️
            -- PostgreSQL, Hive, Redshift, BigQuery의 경우
            OVER(PARTITION BY session ORDER BY stamp ASC)
            
          FROM
            access_log
        )
        SELECT *
        FROM access_log_with_next_Action
        ORDER BY
          session, stamp
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 17.39.56.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/927cbfdb-264f-4c44-9d16-f3f6aa1c6f07/스크린샷_2022-05-01_17.39.56.png)
            
            - `action`과 `next_action` 모두가 `search`인 레코드는 **재검색 수**를 의미
            - 추가로 `action=search`인 레코드를 집계하고
                - 이를 검색 총 수로 보면, 두 값을 통해 재검색 비율 집계 가능
        
        ### **CODE.21.4. 재검색 비율을 집계하는 쿼리**
        
        ```sql
        WITH
        access_log_with_next_action AS (
          -- CODE.21.3.
        )
        SELECT
          -- PostgreSQL, Hive, Redshift, SparkSQL, substring으로 날짜 부분 추출
          substring(stamp, 1, 10) AS dt
          , COUNT(1) AS search_count
          , SUM(CASE WHEN next_action = 'search' THEN 1 ELSE 0 END) AS retry_count
          , AVG(CASE WHEN next_action = 'search' THEN 1.0 ELSE 0.0 END) AS retry_rate
        FROM
          access_log_with_next_action
        WHERE
          action = 'search'
        GROUP BY
          -- PostgreSQL, Redshift, BigQuery
          -- SELECT 구문에서 정의한 별칭을 GROUP BY 지정 가능
          dt
          
        ORDER BY
          dt
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 17.44.20.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c930f94b-262e-4764-9743-77018f0f91bb/스크린샷_2022-05-01_17.44.20.png)
            
        
        ### 2.**재검색 키워드 집계하기**
        
        - **재검색 키워드**를 집계하면
            - **동의어 사전**이 `흔들림을 잡지 못하는 범위 확인`이 가능
        - 추가로 컨텐츠 명칭의 **새로운 흔들림**과
            - 사람들이 일반적으로 해당 컨텐츠를 **부르는 이름**을 찾는 척도가 됨
            - ex) 어벤져스 : 인피니티 워 —> 사람들: 어벤져스3
        - 사용자들이
            - 어떤 검색어로 **재검색** 했는지 확인 및 집계 이후
            - 대처해야 하는 부분이 있다면 **동의어 사전 반영** 필요
        
        ### **CODE.21.5. 재검색 키워드를 집계하는 쿼리**
        
        ```sql
        WITH
        access_log_with_next_search AS (
          SELECT
            stamp
            , session
            , action
            , keyword
            , result_num
            , LEAD(action)
              -- PostgreSQL, Hive, Redshift, BigQuery
              OVER(PARTITION BY session ORDER BY stamp ASC)
              AS next_action
            , LEAD(keyword)
              -- PostgreSQL, Hive, Redshift, BigQuery
              OVER(PARTITION BY session ORDER BY stamp ASC)
              AS next_keyword
            , LEAD(result_num)
              -- PostgreSQL, Hive, Redshift, BigQuery
              OVER(PARTITION BY session ORDER BY stamp ASC)
              AS next_result_num
          FROM
            access_log
        )
        SELECT
          keyword
          , result_num
          , COUNT(1) AS retry_count
          , next_keyword
          , next_result_num
        FROM
          access_log_with_next_search
        WHERE
          action = 'search'AND next_action = 'search'  ⭐️⭐️
        GROUP BY
          keyword, result_num, next_keyword, next_result_num
        ```
        
        - 결과
        
        ![스크린샷 2022-05-01 17.54.03.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4bd3841-63b8-45f4-9979-f24ed6318a1f/스크린샷_2022-05-01_17.54.03.png)
        
    - 3.재검색 키워드를 분류해서 집계하기 - 477p
        - 사용자가 **재검색** 했다는 것은 검색 결과에 만족하지 못했으며, 다른 **동기**가 있다는 것을 의미
        
        ### **재검색의 동기**
        
        - 1.`NoMatch`에서의 조건 변경
            - 검색 결과가 `0`개 이므로, **다른 검색어**로 검색
            - 해당 키워드는 **동의어 사전**과 **사용자 사전에 추가할 키워드 후보**를 의미
            
            ### **CODE.21.6. NoMatch에서 재검색 키워드를 집계하는 쿼리**
            
            ```sql
            WITH
            access_log_with_next_search AS (
              -- CODE.21.5
            )
            SELECT
              keyword
              , result_num
              , COUNT(1) AS retry_count
              , next_keyword
              , next_result_num
            FROM
              access_log_with_next_search
            WHERE
              action = 'search'AND next_Action = 'search'-- NoMatch 로그만 필터링하기
              AND result_num = 0⭐️⭐️
            GROUP BY
              keyword, result_num, next_keyword, next_result_num
            ```
            
            - (with구문)
                
                ![스크린샷 2022-05-01 23.13.38.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/56a3bc68-dbd8-4aad-8753-bbc1d45f385b/스크린샷_2022-05-01_23.13.38.png)
                
            - 결과
                
                ![스크린샷 2022-05-01 18.05.01.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/772c839c-1e10-4824-987a-58972a9d3fca/스크린샷_2022-05-01_18.05.01.png)
                
            
        
        - 2.검색 필터링
            - 검색 결과가 너무 많으므로, 단어를 필터링
            - 재검색한 **검색 키워드**가 원래의 검색 키워드를 **포함** 하고 있다면
                - 검색을 조금 더 **필터링**하고 싶은 의미일 수 있음
            - 자주 사용되는 **필터링 키워드**가 있다면
                - 이를 원래 키워드로 검색 했을 때
                - **연관 검색어** 등으로 출력하여, 사용자가 요구하는 컨텐츠로 빠르게 유도 가능
            
            ### **CODE.21.7. 검색 결과 필터링 시의 재검색 키워드를 집계하는 쿼리**
            
            ```sql
            WITH
            access_log_with_next_search AS (
              -- CODE.21.5.
            )
            SELECT
              keyword
              , result_num
              , COUNT(1) AS retry_count
              , next_keyword
              , next_result_num
            FROM
              access_log_with_next_search
            WHERE
              action = 'search'AND next_action = 'search'
            -- 원래 키워드를 포함하는 경우만 추출하기
              AND next_keyword LIKE concat('%', keyword, '%')⭐️⭐️⭐️⭐️⭐️⭐️
            GROUP BY
              keyword, result_num, next_keyword, next_result_num
            ;
            ```
            
            - 결과
                
                ![스크린샷 2022-05-01 18.09.30.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b689b97e-4977-4c9a-90f4-8feccd92c56f/스크린샷_2022-05-01_18.09.30.png)
                
            
        
        - 3.검색 키워드 변경
            - 검색 결과가 나오기는 했으나, **다른 검색어**로 재검색
            - 완전히 다른 검색 키워드를 이용하여 재검색 했을 경우
                - 원래 검색 키워드를 사용한 검색 결과에 **원하는 내용이 없음**
            
            —>**동의어 사전**이 정상 동작하지 않았다는 것
            
            ### **CODE.21.8. 검색 키워드를 변경 때 재검색을 집계하는 쿼리**
            
            ```sql
            WITH
            access_log_with_next_search AS (
              -- CODE.21.5.
            )
            SELECT
              keyword
              , result_num
              , COUNT(1) AS retry_count
              , next_keyword
              , next_result_num
            FROM
              access_log_with_next_search
            WHERE
              action = 'search'AND next_action = 'search'
              -- 원래 키워드를 포함하지 않는 검색 결과만 추출
              -- PostgreSQL, Hive, BigQuery, SparkSQL, concat 함수 사용
              AND next_keyword NOT LIKE concat('%', keyword, '%')
              
            GROUP BY
              keyword, result_num, next_keyword, next_result_num
            ;
            ```
            
            - (with구문)
                
                ![스크린샷 2022-05-01 23.13.38.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/56a3bc68-dbd8-4aad-8753-bbc1d45f385b/스크린샷_2022-05-01_23.13.38.png)
                
            - 결과
                
                ![스크린샷 2022-05-01 18.13.40.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9a17d7f4-4383-48a9-949a-fd1240e377bb/스크린샷_2022-05-01_18.13.40.png)
                
    - 4.검색 이탈 비율과 키워드 집계하기 - 481p
        - `action=search, next_action is NULL` : 검색 이탈
        - `action=search`임을 **검색 총 수**로 활용하여
            - `검색 이탈 수 / 검색 총 수 = 검색 이탈 비율` 추산 가능
            
        
        ### **CODE.21.10. 검색 이탈 비율을 집계하는 쿼리**
        
        ```sql
        WITH
        access_log_With_next_action AS (
          -- CODE.21.3
        )
        SELECT
          -- PostgreSQL, Hive, Redshift, SparkSQL, substring으로 날짜 추출
          substring(stamp, 1, 10) AS dt
          , COUNT(1) AS search_count
          , SUM(CASE WHEN next_action IS NULL THEN 1 ELSE 0 END) AS exit_count
          , AVG(CASE WHEN next_action IS NULL THEN 1.0 ELSE 0.0 END) AS exit_rate
        FROM
          access_log_with_next_action
        WHERE
          action = 'search'
        GROUP BY
          -- PostgreSQL, Redshift, BigQuery
          -- SELECT 구문에서 정의한 별칭을 GROUP BY에 지정 가능
          dt
        ORDER BY
          dt
        ;
        ```
        
        - with 구문
            
            ![스크린샷 2022-05-01 17.39.56.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/927cbfdb-264f-4c44-9d16-f3f6aa1c6f07/스크린샷_2022-05-01_17.39.56.png)
            
        - 결과
            
            ![스크린샷 2022-05-01 18.19.34.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3dec3def-09f9-4ad8-92e6-0229975e4efc/스크린샷_2022-05-01_18.19.34.png)
            
        
        ### **검색 이탈 키워드 집계하기**
        
        - 검색 이탈이 발생했을 때 **검색한 키워드**를 추출하는 쿼리
        
        ### **CODE.21.11. 검색 이탈 키워드를 집계하는 쿼리**
        
        ```sql
        WITH
        access_log_with_next_search AS (
          -- CODE.21.5
        )
        SELECT
          keyword
          , COUNT(1) AS search_count
          , SUM(CASE WHEN next_action IS NULL THEN 1 ELSE 0 END) AS exit_count
          , AVG(CASE WHEN next_action IS NULL THEN 1.0 ELSE 0.0 END) AS exit_rate
          , result_num
        FROM
          access_log_with_next_search
        WHERE
          action='search'
        GROUP BY
          keyword, result_num
          -- 키워드 전체의 이탈률을 계산한 후, 이탈률이 0보다 큰 키워드만 추출하기⭐️⭐️⭐️
        HAVING
          SUM(CASE WHEN next_action IS NULL THEN 1 ELSE 0 END) > 0 
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 19.13.54.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6e55d0bd-2888-48e0-8e75-04d8e439a026/스크린샷_2022-05-01_19.13.54.png)
            
        
    - 5.검색 키워드 관련 지표의 집계 효율화 하기 - 484p
        - `NoMatch` 비율, `재검색` 비율, `검색 이탈` 비율
        - 각각의 지표과 **검색 키워드**를 산출하기 위해
            - 매번 비슷한 쿼리를 작성하는 것은 비효율적
        - 따라서 다음과 같이 **집계 효율화**를 위한
            - **중간 데이터 생성 SQL**을 사용
        
        ### **CODE.21.12. 검색과 관련된 지표를 집계하기 쉽게 중간 데이터를 생성하는 쿼리**
        
        ```sql
        WITH
        access_log_with_next_search AS (
          -- CODE.21.5
        )
        , search_log_with_next_action (
          SELECT *
          FROM
            access_log_with_next_search
          WHERE
            action = 'search'
        )
        SELECT *
        FROM search_log_with_next_action
        ORDER BY
          session, stamp
        ;
        ```
        
        ### **원포인트**
        
        - 앞의 출력 결과를 사용하면 `NoMatch` 수, `재검색` 수, `검색 이탈 수`를 포함해
            - 키워드 등을 간단하게 집계 가능
    - 6.검색 결과의 포괄성을 지표화 하기 - 486p
        - **검색 키워드**에 대한 지표를 사용하여
            - **검색 엔진 자체의 정밀도**를 평가하는 방법
            
        - 샘플 데이터
            - 검색 키워드에 대한 검색 결과 순위(**DATA.21.2**) : search_result
                - keyword, rank, item
            - 검색 키워드에 대한 정답 아이템(**DATA.21.3**) : correct_result
                - keyword, item
                - 검색 키워드에 대해 올바른 결과를 나타낸 아이템을 미리 정리한 것
                
        
        ### **재현율(Recall)을 사용해 검색의 포괄성 평가하기**
        
        - 검색 엔진을 **정량적**으로 평가하는 대표적인 지표로
            - 재현율(`Recall`)과 정확률(`Precision`)이 존재
        - 재현율
            - 어떤 키워드의 검색 결과에서
                - 미리 준비한 **정답 아이템**이 얼마나 나왔는지 비율로 표기
                - 재현율 : 실제로 등장한 정답수/ 준비한 정답수
            - 특정 키워드 10개의 검색 결과가 나왔으면 할때
                - 실제 4개만 나온다면, `40%`의 재현율
        - 재현율을 집계하려면
            - 일단 **검색 결과**와 **정답 아이템**을 결합하고
            - 어떤 아이템이 **정답 아이템**에 포함되는지 판단해야 함
            
        
        ### **CODE.21.13. 검색 결과와 정답 아이템을 결합하는 쿼리 ⭐️⭐️**
        
        ```sql
        WITH
        search_result_with_correct_items AS (
          SELECT
            ---실제 결과가 있으면 r.keyword, 없으면 c.keyword
            COALESCE(r.keyword, c.keyword) AS keyword
            ---실제 결과가 있으면 r.rank 없으면 빈칸
            , r.rank
            ---실제 결과가 있으면 r.item, 없으면 c.item
            , COALESCE(r.item, c.item) AS item
            ---정답아이템이 null이 아니면 1(정답플래그), null 이면 0 -->correct 컬럼
            , CASE WHEN c.item IS NOT NULL THEN 1 ELSE 0 END AS correct
          FROM
            ---실제결과 테이블(r)과 정답테이블(c)를 full outer join 
            ---을 사용해 검색 결과에 포함되지 않은 정답 아이템 레코드를 남긴다
            ---재현율을 계산하려면 정답 아이템의 총 수를 구해야하기 때문
            search_result AS r
            FULL OUTER JOIN
              correct_result AS c
              ---키워드도 같아야 하고 상품도 같아야함
              ON r.keyword = c.keyword
              AND r.item = c.item
        )
        SELECT *
        FROM
          search_Result_with_correct_items
        ORDER BY
          keyword, rank
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 19.29.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/83df94b3-7b54-43ed-8bc8-2f12804dd7a8/스크린샷_2022-05-01_19.29.27.png)
            
        
        ### **CODE.21.14. 검색 상위 n개의 재현율을 계산하는 쿼리**
        
        - 상위 `n`개의 정답 아이템 히트 수는, `SUM` 윈도 함수를 사용하여
            - 누계 정답 아이템 플래그를 모두 합하여 계산
            - 이 값을 **정답 항목 총 수**로 나누면 **재현율**을 구할 수 있음
            - 상위 n개 재현율 = 누계 정답 아이템 플래그를 모두 합/**정답 항목 총 수**
        - 검색 결과에 포함되지 않은 아이템의 레코드는
            - 재현율을 계산할 수 없음 -> 편의상 `0`으로 출력
        
        ```sql
        WITH
        search_result_with_correct_items AS (
          -- CODE.21.13.
        )
        , search_result_with_recall AS (
          SELECT
            *
            -- 검색 결과 상위에서, 정답 데이터에 포함되는 아이템 수의 누계 계산
            , SUM(correct)
            -- rank=NULL이면, 아이템의 정렬 순서에 마지막에 위치되기 때문에
            -- 편의상 가장 큰 값으로 변환 ⭐️
              OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 100000) ASC
                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_correct
            , CASE
              -- 검색 결과에 포함되지 않은 아이템은 편의상 재현율(recall)을 0으로 다루기
              WHEN rank IS NULL THEN 0.0
              ELSE
               ---재현율 공식(키워드별 현행까지 누적 정답수/키워드별 정답개수 총합)
                100.0
                * SUM(correct)
                    OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 100000) ASC
                      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
                    / SUM(correct) OVER(PARTITION BY keyword)
                END AS recall
            FROM
              search_Result_with_correct_items
        )
        SELECT *
        FROM
          search_result_with_recall
        ORDER BY
          keyword, rank
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 19.38.51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c34fe5ca-472d-42cd-aebb-a1f6bad0af86/스크린샷_2022-05-01_19.38.51.png)
            
        
            —> 4행의 recall 값 = 1(현행까지 정답수)/6(postgres의 전체 정답수)
        
        ### **재현율의 값을 집약해서 비교하기 쉽게 만들기**
        
        - 위 코드에서 구한 재현율은 **검색 결과 전체 레코드**에 대한 재현율
            - 이 값으로 **검색 엔진 평가**는 어려움
        - 검색 엔진을 **정량적**으로 평가하고
            - 여러 개의 **검색 엔진 비교**하려면
            - 하나의 **대표값**으로 집약하는 것이 바람직
        - **검색 엔진**의 일반적인 인터페이스는
            - 검색 결과 `상위 n개`를 순위 형식으로 출력
            - 각 검색 키워드에 대한 **재현율**을 사용자에게 출력되는 **아이템 개수**로 한정해서 구하는 것이 좋음
                - 첫번째 페이지에 재현되는 아이템이 몇개인지를 구하는 것이 좋다는 의미
        
        ### **CODE.21.15. 검색 결과 상위 5개의 재현율을 키워드별로 추출하는 쿼리**
        
        - 검색 결과의 출력 결과가 `default=5`로 가정하여
            - 재현율을 **키워드**별 계산
        - 검색 결과가 `5`개 이상이라면, 상위 `5`개로 재현율을 구하고
            - 검색 결과가 `5`개보다 적을 경우, **검색 결과 전체**를 기반으로 재현율을 구함
        - 검색 결과가 없다면 재현율은 `0`
        
        ```sql
        WITH
        search_result_with_correct_items AS (
          -- CODE.21.13
        )
        , search_result_with_recall AS (
          -- CODE.21.14
        )
        , recall_over_rank_5 AS (
          SELECT
            keyword
            , rank
            , recall
            -- 검색 결과 순위가 높은 순서로 번호 붙이기
            -- 검색 결과에 나오지 않는 아이템은 편의상 0으로 다루기(검색결과가 몇개가 나올지 모르기 때문)
            , ROW_NUMBER()
                OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 0) DESC)
              AS desc_number
          FROM
            search_result_with_recall
          WHERE
            -- 검색 결과 상위 5개 이하 또는 검색 결과에 포함되지 않은 아이템만 출력
            COALESCE(rank, 0) <= 5 ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️
        )
        SELECT
          keyword
          , recall AS recall_at_5
        FROM recall_over_rank_5
        -- 검색 결과 상위 5개 중에서 가장 순위가 높은 레코드 추출하기 ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️
        WHERE desc_number = 1
        ;
        ```
        
        - 결과⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️
            - hive—>검색결과 상위 5개중 정답5개,  redshift—>검색결과 상위 5개중 정답 0개
                
                ![스크린샷 2022-05-01 19.42.21.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5edb66d8-e9c3-4490-9ca8-53ab1da9a874/스크린샷_2022-05-01_19.42.21.png)
                
        
        ### **CODE.21.16. 검색 엔진 전체의 평균 재현율을 계산하는 쿼리 (쉬워서 설명 생략)**
        
        - 위 코드에서 검색 키워드의 재현율 집약 이후, 마지막으로 전체적인 **재현율 평균** 구하기
            
            ```sql
            WITH
            search_result_with_correct_items AS (
              -- CODE.21.13
            )
            , search_result_with_recall AS (
              -- CODE.21.14
            )
            , recall_over_rank_5 AS (
              -- CODE.21.15
            )
            SELECT
              avg(recall) AS average_recall_at_5
            FROM recall_over_rank_5
            -- 검색 결과 상위 5개 중에서 가장 순위가 높은 레코드 추출하기
            WHERE desc_number = 1
            ;
            ```
            
        
        ### **정리**
        
        - 재현율은 **정답 아이템**에 포함되는 아이템을
            - 어느 정도 망라할 수 있는지를 나타내는 지표
        - 재현율은 **의학** 또는 **법** 관련 정보를 다룰 때도 많이 사용됨
        
    - 7.검색 결과의 타당성을 지표화 하기 - 493p
        - 검색 결과를 평가할 때 사용하는 지표중 하나인 **정확률**(Precision)
            - 간단하게 **정밀도**라고 부르기도 함
        - **정확률**은
            - 검색 결과에 포함되는 아이템 중(재현률과 다른점), **정답 아이템**이 어느 정도 비율로 포함되는가를 나타냄
            - e.g. 검색 결과 상위 10개에 5개의 정답 아이템이 포함되어 있다면, `정확률은 50%`
            - 정확률 = 나타난 정답아이템 수/ 검색결과 수
        
        ### **정확률(Precision)을 사용해 검색의 타당성 평가하기**
        
        - 검색 결과 상위 `n`개의 정확률을 계산하는 쿼리
        - 재현율(코드 21-14)과 거의 유사하나
            - 분모 부분만 **검색 결과 순위까지의 누계 아이템 수**로 변경
            - 이전과 동일하게, 검색 결과에 포함되지 않은 아이템의 레코드도
                - 편의상 정확률을 `0`으로 계산
        
        ```sql
        **CODE.21.17. 검색 결과 상위 n개의 정확률을 계산하는 쿼리**
        ```
        
        ```sql
        WITH
        search_result_with_Correct_items AS (
          -- CODE.21.13
        )
        , search_result_with_precision AS (
          SELECT
            *
            -- 검색 결과의 상위에서 정답 데이터에 포함되는 아이템 수의 누계 구하기
            , SUM(correct)
              -- rank가 NULL이라면 정렬 순서의 마지막에 위치하므로
              -- 편의상 굉장히 큰 값으로 변환하기
              OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 100000) ASC
                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_correct
            , CASE
              -- 검색 결과에 포함되지 않은 아이템은 편의상 적합률을 0으로 다루기
                WHEN rank IS NULL THEN 0.0
                ELSE
                  100.0
                  * SUM(correct)
                      OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 100000) ASC
                        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
                  -- 재현률과 다르게, 분모에 검색 결과 순위까지의 누계 아이템 수 지정하기 ⭐️⭐️⭐️⭐️          
                    / COUNT(1)
                      OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 100000) ASC
                        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
                END AS precision
          FROM
            search_result_with_correct_items
        )
        SELECT *
        FROM
          search_result_with_precision
        ORDER BY
          keyword, rank
        ;
        ```
        
        - 결과
            - postgres 4위 —>25.00=1(현재까지 정답개수)/4(현재까지 검색결과수)
            - postgres 5위 —>20.00=1(현재까지 정답개수)/5(현재까지 검색결과수)
            - —>검색 순위에 따라 분모가 바뀜(재현율과 차이점)
                
                ![스크린샷 2022-05-01 19.53.19.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c0efc85-a99c-4932-87c3-ff17e99546d8/스크린샷_2022-05-01_19.53.19.png)
                
            
        
        ### **정확률 값을 집약해서 비교하기 쉽게 만들기**
        
        - 키워드마다 값을 집약하기
        - 검색 결과 상위 5개의 정확률을 키워드로 추출
            - 재현율을 계산할 때 사용한 쿼리(21-15)와 거의 같음
        - 검색 결과 상위 `n`개의 정확률은 `P@n`이라고 표기 ⭐️⭐️
        
        ### **CODE.21.18. 검색 결과 상위 5개의 정확률을 키워드별로 추출한 쿼리**
        
        ```sql
        WITH
        search_result_with_correct_items AS (
          -- CODE.21.13
        )
        , search_result_with_precision AS (
          -- CODE.21.17
        )
        , precision_over_rank_5 AS (
          SELECT
            keyword
            , rank
            , precision
            -- 검색 결과 순위가 높은 순서로 번호 붙이기
            -- 검색 결과에 나오지 않는 아이템은 편의상 0으로 다루기
            , ROW_NUMBER()
                OVER(PARTITION BY keyword ORDER BY COALESCE(rank, 0) DESC) AS desc_number
          FROM
            search_result_with_precision
          WHERE
            -- 검색 결과의 상위 5개 이하 또는 검색 결과에 포함되지 않는 아이템만 출력하기
            COALESCE(rank, 0) <= 5
        )
        SELECT
          keyword
          , precision AS precision_at_5
        FROM precision_over_rank_5
          -- 검색 결과의 상위 5개 중에서 가장 순위가 높은 레코드만 추출하기
          WHERE desc_number = 1;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 19.57.51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5ca7aa8e-4f0b-43e6-86c1-ac10b83d0bc5/스크린샷_2022-05-01_19.57.51.png)
            
        
        ### **CODE.21.19. 검색 엔진 전체의 평균 정확률을 계산하는 쿼리 (코드 21-16과 같음)**
        
        - 검색 키워드들의 정확률을 추출한 이후
            - 정확률의 **평균**을 구하고,
            - 검색 엔진 전체의 **평균 정확률**을 구하기
            
            ```sql
            WITH
            search_Result_With_correct_items AS (
              -- CODE.21.13
            )
            , search_Result_with_precision AS (
              -- CODE.21.17
            )
            , preceision_over_rank_5 AS (
              -- CODE.21.18
            )
            SELECT
              AVG(precision) AS average_precision_at_5
            FROM precision_over_rank_5
            -- 검색 결과 상위 5개 중에서 가장 순위가 높은 레코드만 추출하기
            WHERE desc_number=1
            ;
            ```
            
        
        ### **정리**
        
        - 정확률은 **검색 결과 상위**에 출력되는
            - **아이템의 타당성**을 나타내는 지표
        - 이 지표는 **웹 검색**등
            - 방대한 데이터 검색에서 **적절한 검색 결과**를 빠르게 찾고 싶은 경우에 중요한 지표
        
    - 8.검색 결과 순위와 관련된 지표 계산하기 - 497p
        - **재현율**과 **정확률**으로 실제 검색 엔진 검토시, 복잡한 조건 검토하게 되면 부족한 점 존재
            - 1.검색 결과의 순위는 고려하지 않음
                - 검색 상위 10개의 **적합률**이 `40%`인 검색엔진 A, B가 존재할 때,
                - A는 `1, 4`번째에 정답 아이템 존재, 검색엔진 B는 `7, 10`번째 정답 아이템 존재
                - 당연히 `1`이 더 좋은 검색 엔진이라 할 수 있으나, `P@10`을 구하면, 두 검색엔진은 **같은 성능**으로 판단
                - 검색 엔진을 고려한 지표
                    - MAP(Mean Average Precision)
                    - MRR(Mean Reciprocal Rank)
            - 2.정답과 정답이 아닌 아이템을 `0`과 `1`이라는 두가지 값으로밖에 표현할 수 없음
                - 이전 절까지 다룬 정답 아이템의 경우, 어떤 아이템이 **검색에 히트**되어도 **같은 것으로 취급**
                - 경우에 따라 **굉장히 관련 있는 아이템**, **조금 관련있는 아이템**처럼, 단계적인 점수를 사용해 정답 아이템을 다루고 싶은 경우 존재
                - 예시
                    - 사용자의 별점(5단계) 리뷰 데이터가 존재할 때,
                    - 해당 점수를 정답 데이터를 사용하여 **적합률**과 **재현률**을 구할 경우
                        - 사용자로부터 입력받은 **별점**을 활용할 수 없음
                - 단계적인 점수를 고려하여 정답 아이템을 다루는 지표
                    - DCG(Discounted Cumulated Gain)
                    - NDCG(Normalized DCG)
            - 3.모든 아이템에 대한 정답을 미리 준비하는 것은 사실 **불가능**에 가까움
                - 검색 엔진이 필요한 서비스의 경우, 검색 대상 아이템이 많을 수 있음
                - 사용자 **행동 로그**등에서, 기계적으로 정답 아이템을 만들 수 있겠으나,
                    - 사용자가 각각 확인할 수 있는 아이템의 수는 많지 않음
                - **재현율**과 **적합률**을 적용하더라도, 극단적으로 낮은 값이기 때문에 사용하기 어려움
                - 검색 대상 아이템 수에서, 정답 아이템 수가 한정된 경우에는 **BPREF**(Binary Preference)를 활용
        
        ### **MAP(Mean Average Precision)으로 검색 결과의 순위를 고려해 평가하기**
        
        - 검색 결과의 순위를 고려한 평가
        - 검색 결과 상위 `n`개의 **적합률** 평균
        - 정답 아이템이 순위에 아예 없는경우, 편의상 적합률을 `0`으로 정의
        - 예시
            - 정답 아이템 수가 `4`개라고 할때, `P@10 = 40%`
            - 상위 `1~4번째`가 모두 정답 아이템
                - `MAP =100 * ((1/1) + (2/2) + (3/3) + (4/4))/4 = 100`으로 계산
            - 상위 `7~10`번째가 정답 아이템이라면, `P@10 = 40%`
                - `MAP = 100 * ((1/7) + (2/8) + (3/9) + (4/10))/4 = 28.15`
                    - 이전보다 낮게 평가함
                    - MAP을 계산하려면, **정답 아이템**별로 **정확률** 추출 필요
                        - 이전 절의 쿼리(21-17)를 활용하여 `correct`컬럼의 플래그가 `1`인 레코드를 추출하면 됨 ⭐️⭐️⭐️⭐️⭐️⭐️
        
        ### **CODE.21.20. 정답 아이템별로 적합률을 추출하는 쿼리**
        
        ```sql
        WITH
        search_result_with_correct_items AS (
        -- CODE.21.13.
        )
        , search_result_with_precision AS (
        -- CODE.21.17
        )
        SELECT
          keyword
          , rank
          , precision
        FROM
          search_result_with_precision
        WHERE
          correct = 1 ⭐️⭐️⭐️⭐️
        ;
        ```
        
        - 결과
            - hive rank 1—> 1/1
            - postgres rank 4 —> 1/4
            - postgres rank 6—> 2/6
                
                ![스크린샷 2022-05-01 20.08.23.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5872572b-5f92-467e-986c-62877e8636f9/스크린샷_2022-05-01_20.08.23.png)
                
        
        ### **CODE.21.21. 검색 키워드별로 정확률의 평균을 계산하는 쿼리**
        
        ```sql
        WITH
        search_result_with_correct_items AS (
        -- CODE.21.13
        )
        , search_result_with_precision AS (
        -- CODE.21.17
        )
        , average_precision_for_keywords AS (
        SELECT
          keyword
          , AVG(precision) AS average_precision ⭐️⭐️⭐️⭐️⭐️
        FROM
          search_result_with_precision
        WHERE
          correct = 1
        GROUP BY
          keyword ⭐️⭐️⭐️
        )
        SELECT *
        FROM
          average_precision_for_keywords
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 20.12.47.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/839b85c5-fd57-44f7-b6a0-eb6dec65f21f/스크린샷_2022-05-01_20.12.47.png)
            
        
        ### **CODE.21.22. 검색 엔진의 MAP을 계산하는 쿼리**
        
        - 검색 키워드별로 정확률의 평균을 추출한 이후, 검색 엔진 자체의 MAP을 계산  ⭐️⭐️⭐️⭐️⭐️⭐️
            
            ```sql
            WITH
            search_result_with_correct_itmes AS (
            -- CODE.21.13
            )
            , search_result_with_precision AS (
            -- CODE.21.17
            )
            , average_precision_for_keywords AS (
            -- CODE.21.21
            )
            SELECT
              AVG(average_precision) AS mean_average_precision
            FROM
              average_precision_for_keywords
            ;
            ```
            
            - 결과
                
                ![스크린샷 2022-05-01 20.14.40.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5aaee030-13c5-43f8-9932-4c91f18eb07f/스크린샷_2022-05-01_20.14.40.png)
                
        
        ### **검색 평가와 관련한 다른 지표들**
        
        ### **정확률(Precision)**
        
        - 검색 결과 순위 중에서, 얼마나 많은 비율의 아이템이 제대로 출력되는가
        - 웹 검색에서 처럼, 방대한 **중복**을 포함한 정보에서
            - 관련 정보를 재빠르게 추출해야할 때 사용
        
        ### **재현율(Recall)**
        
        - 사용자가 원할 것으로 보이는 아이템 중에서
            - 얼마나 많은 비율의 아이템이 제대로 출력되는가
        - `법률계` 또는 `의학계`를 대상으로 하는 검색 처럼
            - 결과에 **누수**가 있으면 안되는 경우에 중요
        
        ### **P@n**
        
        - 순위 상위 `n`개를 기반으로 측정한 Precision
        
        ### **MAP(Mean Average Precision)**
        
        - 각각의 쿼리에 대한 Average Precision의 평균값
        
        ### **MRR(Mean Reciprocal Rank)**
        
        - 순위 중에서 **처음부터 정답 아이템이 나오는 순위**의 **역수**를 취한 뒤 평균을 구한 값
        - 평균역순위
        
        ### **E-Measure**
        
        - `Precision`과 `Recall`을 합쳐 만든 단일 지표
        - `Precision`과 `Recall`을 1:1로 합치면 `F-Measure`가 됨
        
        ### **F-Measure**
        
        - `Precision`과 `Recall`의 조화 평균
        
        ### **NDCG(Normalized Discounted Cumulated Gain)**
        
        - 아이템의 평가를 **여러 레벨**로 설정하고,
            - 순위 내부에서 **순위를 추가**한 지표
        - 정답 데이터를 만들기 어렵지만, **신뢰도가 매우 높음**
        
        ### **BPREF(Binary Preferences)**
        
        - 아이템 사이의 **상대적인 관심도**를 기준으로 하는 지표
        - 아이템 공간이 굉장히 거대해서
            - 평가 데이터를 만들기 힘든 경우에 사용
        
        ### **정리**
        
        - 검색 엔진을 **정량적**으로 평가하기 위한 지표는
            - 위 정리 내용 외에 많은 것이 존재
        - NDCG와 BPREF와 같은 다른 지표에 대해서도
            - 정의와 적용 범위를 확인 이후
            - 어떻게 쿼리를 작성하면 구할수 있을지 고민할 것
- **22강 : 데이터 마이닝 - 504p~**
    - 데이터 마이닝
        - 대량의 데이터에서 **특정 패턴**또는 **규칙** 등 유용한 지식을 추출하는 방법을 전반적으로 나타내는 용어
        - **상관 규칙 추출**, **클러스터링**, **상관 분석** 등이 존재
    - 데이터 마이닝의 방법 대부분은
        - `재귀 처리`와, `휴리스틱 처리` 필요
        - 단순한 `SQL`로 처리하기 어려움
    - 일반적으로 `R`이나 `Python`을 활용하는 경우가 많음
    - 이번 절에서는 **상관 규칙 추출 방법**중 하나인 **어소시에이션 분석**을 다루고
        - 이 내용을 SQL로 구현
    - 샘플데이터
        - 구매상세로그(purchase_detail_log)
            - stamp, session, purchase_id, product_id
            
            ```sql
            DROP TABLE IF EXISTS purchase_detail_log;
            CREATE TABLE purchase_detail_log(
                stamp       varchar(255)
              , session     varchar(255)
              , purchase_id integer
              , product_id  varchar(255)
            );
            
            INSERT INTO purchase_detail_log
              VALUES
                ('2016-11-03 18:10', '989004ea',  1, 'D001')
              , ('2016-11-03 18:10', '989004ea',  1, 'D002')
              , ('2016-11-03 20:00', '47db0370',  2, 'D001')
              , ('2016-11-04 13:00', '1cf7678e',  3, 'D002')
              , ('2016-11-04 15:00', '5eb2e107',  4, 'A001')
              , ('2016-11-04 15:00', '5eb2e107',  4, 'A002')
              , ('2016-11-04 16:00', 'fe05e1d8',  5, 'A001')
              , ('2016-11-04 16:00', 'fe05e1d8',  5, 'A003')
              , ('2016-11-04 17:00', '87b5725f',  6, 'A001')
              , ('2016-11-04 17:00', '87b5725f',  6, 'A003')
              , ('2016-11-04 17:00', '87b5725f',  6, 'A004')
              , ('2016-11-04 18:00', '5d5b0997',  7, 'A005')
              , ('2016-11-04 18:00', '5d5b0997',  7, 'A006')
              , ('2016-11-04 19:00', '111f2996',  8, 'A002')
              , ('2016-11-04 19:00', '111f2996',  8, 'A003')
              , ('2016-11-04 20:00', '3efe001c',  9, 'A001')
              , ('2016-11-04 20:00', '3efe001c',  9, 'A003')
              , ('2016-11-04 21:00', '9afaf87c', 10, 'D001')
              , ('2016-11-04 21:00', '9afaf87c', 10, 'D003')
              , ('2016-11-04 22:00', 'd45ec190', 11, 'D001')
              , ('2016-11-04 22:00', 'd45ec190', 11, 'D002')
              , ('2016-11-04 23:00', '36dd0df7', 12, 'A002')
              , ('2016-11-04 23:00', '36dd0df7', 12, 'A003')
              , ('2016-11-04 23:00', '36dd0df7', 12, 'A004')
              , ('2016-11-05 15:00', 'cabf98e8', 13, 'A002')
              , ('2016-11-05 15:00', 'cabf98e8', 13, 'A004')
              , ('2016-11-05 16:00', 'f3b47933', 14, 'A005')
            ;
            ```
            
    
    - 1.어소시에이션 분석 - 505p
        - **상관 규칙 추출**의 대표적인 방법
        - 예시
            - `상품 A를 구매한 사람의 60%는 상품 B도 구매 했다` 등의 상관 규칙을 데이터에서 찾아내는 것
        - 상관 규칙이란
            - `상품 A와 상품 B가 동시에 구매되는 경향이 있다`가 아닌
            - `상품 A를 구매했다면, 상품 B도 구매한다`는
                - **시간적 차이**와 **인과 관계**를 가지는 규칙 ⭐️⭐️⭐️⭐️⭐️⭐️
            - `상품 A를 구매했다면, 상품 B도 구매한다` => true,
                - `상품 B를 구매했다면, 상품 A도 구매한다` => ? (참으로 성립하지 않을 수 있음)⭐️⭐️
        - 본래 어소시에이션 분석은
            - **상품 조합**(A, B, C, ...)를 구매한 사람의 `n%`는
                - **상품 조합**(X, Y, Z, ...)도 구매한다
            - 위와 같이 **상품 조합**과 관련한 상관 규칙도 발견할 때 활용할 수 있는 **범용적인 알고리즘**
        - 하지만, 범용적인 알고리즘을 만들기 위해서는
            - **재귀적인 반복 처리**등을 포함한 많은 처리가 필요,
            - 책에서는 **2개의 상품 간의 발생하는 상관 규칙**으로 단순화
            
        
        ### **어소시에이션 분석에 사용되는 지표**
        
        - 1.지지도(Support)
        - 2.확신도/신뢰도(Confidence)
        - 3.리프트(Lift)
        
        ### 1.**지지도(Support)**
        
        - 상관 규칙이 어느 정도의 **확률**로 발생하는지 나타내는 값
        - 예시
            - 100개의 구매 로그에서 `상품 X와 상품 Y를 같이 구매한 로그`가 20개 있다면,
            - 이 규칙에 대한 지지도는 `20%`
                - 이때, 구매로그는 동시에 구매한 여러 개의 상품이 하나의 로그에 있다고 가정
        
        ### 2.**확신도/신뢰도(Confidence)**
        
        - 어떤 결과가, 어느정도의 확률로 발생하는지를 의미
        - 예시
            - 100개의 구매 로그에,
                - `상품 X 구매 레코드 50개`
                - `상품 Y 구매 레코드 20개`
                - 확신도는 `20/50 = 40%`
        
        ### 3.**리프트(Lift)**
        
        - 어떤 조건을 만족하는 경우의 확률(`확신도`)를
            - 사전 조건 없이 `해당 결과가 일어날 확률`로 나눈 값
        - 예시
            - 100개의 구매로그에
                - `상품 X : 50개`
                - `상품 X & 상품 Y : 20개`
                - `상품 Y : 20개`
                - 확신도 : `20/50 = 40%`
                - 상품 Y 구매 확률 : `20/100 = 20%`
                - 리프트 : `40% / 20% = 2.0`
            - 상품 X를 구매한 경우, 상품 Y를 구매할 확률이 2배라는 의미
        - 보통 리프트 값이 `1.0`이상이면 좋은 규칙
        
        ### **두 상품의 연관성을 어소시에이션 분석으로 찾기**
        
        - 어소시에이션 분석으로 지표를 계산하여, 두 상품의 연관성 찾기
        - `상품 A`와 `상품 B`의 어소시에이션 분석에 필요한 정보
            - 구매로그 총 수
            - 상품 A의 구매 수
            - 상품 B의 구매 수
            - 상품 A & 상품 B의 동시 구매 수
        
        ### **CODE.22.1. 구매 로그 수와 상품별 구매 수를 세는 쿼리**
        
        - `구매 로그 총 수`, `상품 X의 구매 수`를 각각 집계하여, 모든 레코드에 전개
        - 각 상품 ID를 사용하여 아래 값을 집계
            - 구매 로그 총 수(purchase_count)
            - 상품 구매 수(product_count)
                
                ```sql
                WITH
                purchase_id_count AS (
                  -- 구매 상세 로그에서 유니크한 구매 로그 수 계산하기
                  SELECT COUNT(DISTINCT purchase_id) AS purchase_count
                  FROM purchase_detail_log
                )
                , purchase_detail_log_with_counts AS (
                  SELECT
                    d.purchase_id
                    , p.purchase_count
                    , d.product_id
                    -- 상품별 구매 수 계산하기
                    , COUNT(1) OVER(PARTITION BY d.product_id) AS product_count
                  FROM
                    purchase_detail_log AS d
                    CROSS JOIN
                      -- 구매 로그 수를 모든 레코드 수와 결합하기
                      purchase_id_count AS p
                )
                SELECT
                  *
                FROM
                  purchase_detail_log_with_counts
                ORDER BY
                  product_id, purchase_id
                ;
                ```
                
                - 결과
                    
                    ![스크린샷 2022-05-01 20.53.32.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6cef6763-918d-4aad-ba89-ca3571dcbea5/스크린샷_2022-05-01_20.53.32.png)
                    
        
        ### **CODE.22.2. 상품 조합별로 구매 수를 세는 쿼리**
        
        - 위 결과에 `구매한 상품 페어`를 생성하고, 조합별로 `구매 수`를 세는 쿼리
        - 동시에 구매한 상품 페어를 생성하기 위해
            - 구매 ID에서 **자기 결합**을 한 뒤, 같은 상품 페어는 제외⭐️⭐️⭐️
            
            ```sql
            WITH
            purchase_id_count AS (
              -- CODE.22.1.
            )
            , purchase_detail_log_with_counts AS (
              -- CODE.22.1.
            )
            , product_pair_with_stat AS (
              SELECT
                l1.product_id AS p1
                , l2.product_id AS p2
                , l1.product_count AS p1_count
                , l2.product_count AS p2_count
                , COUNT(1) AS p1_p2_count
                , l1.purchase_count AS purchase_count
              FROM
                purchase_detail_log_with_counts AS l1
                JOIN
                  purchase_detail_log_with_counts AS l2
                  ON l1.purhcase_id = l2.purchase_id
              WHERE
                -- 같은 상품 조합 제외하기
                l1.product_id <> l2.product_id ⭐️⭐️⭐️
              GROUP BY
                l1.product_id
                , l2.product_id
                , l1.product_count
                , l2.product_count
                , l1.purchase_count
            )
            SELECT
              *
            FROM
            product_pair_with_stat
            ORDER BY
              p1, p2
            ;
            ```
            
            - 결과
                
                ![스크린샷 2022-05-01 21.11.03.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5fa288f0-ddf2-4d7e-997d-22588307257f/스크린샷_2022-05-01_21.11.03.png)
                
                —> 어소시에이션 분석에 필요한 다음 지표를 계산함
                
                - 구매 로그 총 수, 상품 A의 구매 수, 상품 B의 구매 수, 상품 A와 B 동시 구매수
        
        ### **CODE.22.3. 지지도, 확신도, 리프트를 계산하는 쿼리**
        
        ```sql
        WITH
        purchase_id_count AS (
          -- CODE.22.1.
        )
        , purchase_detail_log_with_counts AS (
          -- CODE.22.1.
        )
        , product_pair_with_stat AS (
          -- CODE.22.2.
        )
        SELECT
          p1
          , p2
          , 100.0 * p1_p2_count / purchase_count AS support
          , 100.0 * p1_p2_count / p1_count AS confidence
          , (100.0 * p1_p2_count / p1_count)
            / (100.0 * p2_count / purchase_count) AS lift
        FROM
          product_pair_with_stat
        ORDER BY
          p1, p2
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 21.14.52.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb044513-bbcf-4a07-ab4f-1f97728e4753/스크린샷_2022-05-01_21.14.52.png)
            
        
        ### **정리**
        
        - 이번 절의 **어소시에이션 분석**은
            - 두 상품의 **상관 규칙**을 주목한 한정적인 분석
        - 이 쿼리로 **상관 규칙 추출**을 이해했다면,
            - 다른 분석 방법도 찾아 도전해볼 것
    
- **23강 : 추천 - 513p~**
    - 추천 시스템(Recommendation System)
        - 사용자의 **흥미 기호**를 분석해서,
        - 사용자가 관심을 가질만한 정보를 추출하는 것이 아닌
            - **사용자에게 가치 있는 정보를 추천하는 것**으로 정의
        - 이를 활용하면
            - 사용자가 아직 모르는 정보를 알려주어, 구매를 유도할 수 있음
    - 이번 절에서는
        - 추천에 관련한 기본적인 내용과
        - Item to Item, User to Item 추천 리스트 만들기
    - 샘플 데이터
        - 아이템 열람/구매 로그(action_log)
            - stamp, user_id, action, product
            
            ```sql
            DROP TABLE IF EXISTS action_log;
            CREATE TABLE action_log(
                stamp   varchar(255)
              , user_id varchar(255)
              , action  varchar(255)
              , product varchar(255)
            );
            
            INSERT INTO action_log
            VALUES
                ('2016-11-03 18:00:00', 'U001', 'view'    , 'D001')
              , ('2016-11-03 18:01:00', 'U001', 'view'    , 'D002')
              , ('2016-11-03 18:02:00', 'U001', 'view'    , 'D003')
              , ('2016-11-03 18:03:00', 'U001', 'view'    , 'D004')
              , ('2016-11-03 18:04:00', 'U001', 'view'    , 'D005')
              , ('2016-11-03 18:05:00', 'U001', 'view'    , 'D001')
              , ('2016-11-03 18:06:00', 'U001', 'view'    , 'D005')
              , ('2016-11-03 18:10:00', 'U001', 'purchase', 'D001')
              , ('2016-11-03 18:10:00', 'U001', 'purchase', 'D005')
              , ('2016-11-03 19:00:00', 'U002', 'view'    , 'D001')
              , ('2016-11-03 19:01:00', 'U002', 'view'    , 'D003')
              , ('2016-11-03 19:02:00', 'U002', 'view'    , 'D005')
              , ('2016-11-03 19:03:00', 'U002', 'view'    , 'D003')
              , ('2016-11-03 19:04:00', 'U002', 'view'    , 'D005')
              , ('2016-11-03 19:10:00', 'U002', 'purchase', 'D001')
              , ('2016-11-03 19:10:00', 'U002', 'purchase', 'D005')
              , ('2016-11-03 20:00:00', 'U003', 'view'    , 'D001')
              , ('2016-11-03 20:01:00', 'U003', 'view'    , 'D004')
              , ('2016-11-03 20:02:00', 'U003', 'view'    , 'D005')
              , ('2016-11-03 20:10:00', 'U003', 'purchase', 'D004')
              , ('2016-11-03 20:10:00', 'U003', 'purchase', 'D005')
            ;
            ```
            
    
    - 1.추천시스템의 넓은 의미 - 514p
        
        ## **. 추천 시스템의 넓은 의미**
        
        - 추천 시스템의 의미는 **좁은 의미**와 **넓은 의미**가 다르기 때문에, 혼동될 수 있음
        - 넓은 의미의 추천 시스템을 사용하여
            - 추천 시스템의 종류, 모듈, 효과, 정밀도를 살펴보기
        
        ### **추천시스템의 종류**
        
        - 두가지 종류
            - `Item to Item`
                - 열람/구매한 아이템을 기반으로 다른 아이템을 추천
                - 아이템과 관련한 개별적인 아이템 제안
                - e.g. 이 상품을 본 사람들은 다음 상품도 보았습니다.
            - `User to Item`
                - 과거의 행동 또는 데모그래픽 정보를 기반으로, 흥미와 기호를 유추, 아이템 추천
                - 사용자 개인에 최적화된 아이템 제안
                - e.g. 당신을 위한 추천 아이템
        - `당신을 위한 추천 아이템`이라 적혀 있더라도, ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️
            - 내부 로직이 **최근 구매한 아이템**을 기반으로 다음 아이템을 추천하는 것이라면
            - Item to Item을 의미
        - `Item to Item`과 `User to Item`은
            - 모두 상품 또는 기사 단위의 추천이 아니더라도,
            - 상품 카테고리 또는 지역 추천 같은 **포괄적인 추천**도 추천의 대상으로 삼음
        - `User to Item`처럼 사용자 **행동**을 기반으로 흥미 기호를 추천하는 경우,
            - 데이터가 적은 사용자에 대해서는 **흥미 기호**를 제대로 판단할 수 없으므로 주의 필요
            
            ### **모듈의 종류**
            
            - 일반적으로 사용되는 `EC 사이트`에는 어떤 추천 모듈이 있는지 구분
            - 모듈의 종류
            
            | 모듈 | 설명 | 예 |
            | --- | --- | --- |
            | 리마인드 | 사용자의 과거 행동을 기반으로 아이템을 다시 제안 | 최근 보았던 상품 / 한 번 더 구매하기 |
            | 순위 | 열람 수, 구매 수 등을 기반으로 인기 있는 아이템을 제안해주는 것 | 인기 순위 / 급상승 순위 |
            | 콘텐츠베이스 | 아이템의 추가 정보를 기반으로 다른 아이템을 추천해주는 것 | 해당 배우가 출연한 다른 작품 |
            | 추천 | 서비스를 사용하는 사용자 전체의 행동 이력을 기반으로, 다음에 볼만한 아이템 또는 관련 아이템을 추측해 제안해주는 것 | 이 상품을 보았던 사람들은 이러한 상품들도 함께 보았습니다 |
            | 개별 추천 | 사용자 개인의 행동 이력을 기반으로 흥미 기호를 추천하고, 흥미 있어 할 만한 아이템을 제안해주는 것 | 당신만을 위한 추천 |
            - 어려운 계산 로직을 사용하지 않더라도
                - `과거에 출력한 정보를 여람 이력으로 출력해주는 모듈`도 `리마인드 목적의 추천 모듈`로 구분
            - 예시
                - `매출`또는 `열람 수`에서 상위를 차지하는 몇 개를 출력해주는 것 역시 **순위**를 사용한 추천
                    - 매출 또는 열람 수의 상위를 구하는 방법은 **CHAP.10**, **CHAP.14.2**에서 소개됨
                - 추가로 **계절성**을 중시하는 순위라면
                    - **CHAP.10.3(170p)**에서 소개한 **증가율**을 정렬 점수로 사용하여
                        - **급상승 순위**와 동일한 모듈 사용 가능
            - 추천/개별 추천(Personal Recommendation)은 **23.2, 23.3**에서 소개
            
            ### **추천의 효과**
            
            - 추천 시스템은
                - 정보가 과다한 상태에서 **사용자가 가치있는 정보를 쉽게 접하도록** 도와주는 기능
            - 이외에도 `EC 사이트`에서 추천의 효과는 다음과 같음
            
            | 효과 | 설명 | 출력 예 |
            | --- | --- | --- |
            | 다운셀 | 가격이 높아 구매를 고민하는 사용자에게 더 저렴한 아이템을 제안해서 구매 수를 올리는 것 | 이전 모델이 가격 할인을 시작했습니다 |
            | 크로스셀 | 관련 상품을 함께 구매하게 해서 구매 단가를 올린느 것 | 함께 구매되는 상품이 있습니다 |
            | 업셀 | 상위 모델 또는 고성능의 아이템을 제안해서 구매 단가를 올리는 것 | 이 상품의 최신 모델이 있습니다 |
            - 예시(햄버거 가게 이야기)
                - `햄버거와 함께 프렌치프라이는 어떤가요?` -> **크로스셀**
                - `500원만 추가하면 프렌치프라이 사이즈를 L로 변경할 수 있습니다` -> **업셀**
                - `작은 사이즈의 커피가 있는데, 가격이 더 저렴합니다` -> **다운셀**
            - 사용자가 한 번 방문할 때마다 **이동률**과 **열람 수**를 올리려면
                - 새로운 상품 또는 인기 상품 이외에도
                - 열람수가 적은 상품을 추천해 **새로운 발견**(의외성)을주어야 함
            - 이처럼 추천 시스템을 구축할 때는
                - 그 목적을 **명확**하게 하는 것이 중요
                
            
            ### **데이터에 따라 얻을 수 있는 정밀도와 효과의 차이**
            
            - 추천 시스템을 구축할 때 중시해야할 점은
                - **어떤 데이터**를 기반으로 **추천 시스템**을 구축할 것인가
            - 기존에는 db에 저장된 구매 데이터, 회원 데이터만 사용할 수 있었으나,
                - 최근에는 빅데이터 기술의 진보로 **상품 열람 로그**도 다룰 수 있게 됨
            - 사용할 수있는 각종 데이터를 어떻게 수집하고
                - 어떻게 사용할지에 따라 얻는 효과가 다름
            
            ### **데이터의 명시적 획득과 암묵적 획득**
            
            - 추천 시스템 구축시 사용할 데이터 수집 방법(2)
                - **사용자 행동**에서 기호를 추측하는 **암묵적인 데이터 획득**
                - 사용자의 기호를 직접 묻는 **명시적 데이터 획득**
                
                | 데이터 기반 | 설명 | 데이터 예 |
                | --- | --- | --- |
                | 암묵적 데이터 획득 | 사용자의 행동을 기반으로 기호를 추측한다데이터 양이 많지만 정확도가 떨어질 수 있다 | 구매 로그열람 로그 |
                | 명시적 데이터 획득 | 사용자에게 직접 기호를 물어본다데이터양이 적지만 평가의 정확성이 높다 | 별점 주기 |
            - 암묵적 데이터 획득은
                - 구매 했다면 **분명 이상품에 기호가 있을 것이다**라는 전제를 기반으로
                - **구매 데이터**를 추천 시스템의 데이터 소스로 활용
                - 단, 상품 구매가 **반드시 사용자의 기호와 이어지는지는 불분명**
                    - e.g. DVD를 구매했으나, 재미 없었음 등
            - 명시적 데이터 획득
                - `별점 주기`등을 사용해서, 사용자의 기호를 정확하게 수집하는 방법
                - 사람들이 별점을 주지 않는 경우도 많으므로, 암묵적 데이터 획득보다 **데이터 수가 적음**
            - 추가로 추천 시스템 중에는 **무엇을 기반으로 추천**하는지 알려주어
                - 사용자를 납득시키는 경우도 존재
            - 단, 명시적으로 획득한 데이터는 **근거**를 보여도 문제가 없으나
                - 암묵적 데이터의 경우, 사용자가 **사생활 침해**로 여길 수 있음 ⭐️⭐️⭐️⭐️⭐️
            
            ### **열람 로그와 구매 로그 차이**
            
            - 암묵적 데이터를 사용할 경우, **열람 로그**와 **구매 로그**중에
                - 무엇을 사용할지에 따라 추천되는 정보가 다를 수 있음
            
            | 데이터 소스 | 내용 | 데이터 예 |
            | --- | --- | --- |
            | 열람 로그 | 특정 아이템과 유사한 아이템을 추천해서 사용자의 선택지를 늘릴 수 있음 | 여행용 가방을 구매한다고 할 때 크기, 형태, 색이 다른것을 추천한다 |
            | 구매 로그 | 함께 구매 가능한 상품을 추천해서 구매 단가를 끌어 올릴 수 있음 | 여행용 가방을 구매한다고 할 때 함께 구매하면 좋은 여권 케이스 등을 추천 |
            
            ### **정리**
            
            - 추천 시스템에는 다양한 목적, 효과 모듈이 존재
            - 추천 시스템의 정의를 확실하게 하고
                - 어떤 효과를 기대하는지 등을 구체화 한 뒤, 시스템을 구축하는 것을 추천
            
        
    - 2.특정 아이템에 흥미가 있는 사람이 함께 찾아보는 아이템 검색 - 519p
        - 사용자의 **접근 로그**를 사용해서 만들 수 있는 추천 시스템 중에서도,
            - 간단하고, 효과가 높은 추천시스템은 **Item to Item**
        - Item to Item 추천
            - `User to Item` 추천과 비교하여, 구현이 쉽고 효과적인 이유는
            - 일반적인 시스템은 **사용자 유동성 > 아이템 유동성**으로, 데이터를 축적하기 더 쉽기 때문
        - 사용자의 흥미/관심은 꾸준히 변화하나,
            - 아이템의 연관성은 시간이 지나도 **크게 변하지 않음**
        - 오래된 로그를 활용해도, 큰 문제가 되지 않음
        - 다만, 뉴스 기사 처럼 아이템 자체가 유동적인 경우에는
            - 이러한 장점을 누릴 수 없으므로 유의
            
        
        ### **접근 로그를 사용해 아이템의 상관도 계산하기**
        
        - **접근 로그**에는
            - **사용자별 아이템 열람**과 **구매 액션**이 저장되어 있음
        - 다음 코드는 **로그를 기반으로** 사용자와 아이템 **조합**을 구하고 점수를 계산하는 쿼리
            - 사용자 단위의 `아이템 열람 수 : 구매 수 = 3:7` 비율로 가중치를 두어, 평균을 구하고
                - 이를 사용자의 아이템에 대한 관심도 점수로 사용
        
        ### **CODE.23.1. 열람 수와 구매 수를 조합한 점수를 계산하는 쿼리**
        
        ```sql
        WITH
        ratings AS (
          SELECT
            user_id
            , product
            -- 상품 열람 수
            , SUM(CASE WHEN action = 'view' THEN 1 ELSE 0 END) AS view_count
            -- 상품 구매 수
            , SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END) AS purchase_count
            -- ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️열람 수 : 구매 수 = 3:7 비율의 가중치 주어 평균 계산
            , 0.3 * SUM(CASE WHEN action='view' THEN 1 ELSE 0 END)
              + 0.7 * SUM(CASE WHEN action='purchase' THEN 1 ELSE 0 END)
              AS score
          FROM
            action_log
          GROUP BY
            user_id, product
        )
        SELECT *
        FROM
          ratings
        ORDER BY
          user_id, score DESC
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 21.43.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7aa8e0f7-b44d-4bff-a1b0-2de16932528d/스크린샷_2022-05-01_21.43.27.png)
            
            - 사용자의 아이템에 관한 흥미를 **수치화**했다면, 그 점수를 조합하여, 아이템 사이의 **유사도**를 계산가능
            - 다음 코드는 **아이템 사이의 유사도**를 계산하고, 순위에 따라 출력하는 쿼리
            
        
        ### **CODE.23.2. 아이템 사이의 유사도를 계산하고 순위를 생성하는 쿼리**
        
        - 아이템들의 유사도를 계산하려면 : 양쪽 아이템을 **모두 열람**하거나 **구매한 사용자**가 필요
        - 따라서 `ratings` 테이블을 **사용자 ID**로 자기 결합하고, 공통 사용자가 존재하는 **아이템 페어**를 생성
        - 이후, 같은 아이템으로 만들어지는 페어는 **제외**하고, 아이템 페어를 집약한 뒤, **유사도**를 계산
        - 2개의 아이템에 대한 **사용자의 점수**를 ⭐️곱하여⭐️ **유사도**를 집계
            
            —>**사용자의 점수**를 ⭐️곱하여⭐️ : **벡터의 내적**
            
            - 벡터 : 어떤 ⭐️아이템에 대해 부여된⭐️ 사용자의 점수를 나열한 **숫자의 집합—>크기와 방향있음**
            - 곱한 점수가 높을수록 **유사도**가 높다고 할 수 있다.
        
        ```sql
        WITH
        ratings AS (
          -- CODE.23.1.
        )
        SELECT
          r1.product AS target
          , r2.product AS related
          -- 두 아이템을 모두 열람/구매한 사용자 수
          , COUNT(r1.user_id) AS users
          -- 스코어들을 곱하고 합계를 구해 유사도 계산
          , SUM(r1.score * r2.score) AS score
          -- 상품의 유사도 순위 계산
          , ROW_NUMBER()
              OVER(PARTITION BY r1.product ORDER BY SUM(r1.score * r2.score) DESC)
            AS rank
        FROM
          ratings AS r1
          JOIN
            ratings AS r2
            -- 공통 사용자가 존재하는 상품의 페어 만들기⭐️⭐️⭐️
            ON r1.user_id = r2.user_id
        WHERE
          -- 같은 아이템의 경우에는 페어 제외하기
          r1.product <> r2.product
        GROUP BY
          r1.product, r2.product
        ORDER BY
          target, rank
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-01 22.13.12.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f2a146a0-76c9-4c48-b424-92299f03aa38/스크린샷_2022-05-01_22.13.12.png)
            
        
        ### ⭐️**점수 정규화 하기**⭐️
        
        - 단순히 **백터 내적**을 사용한 유사도는, 실적용시 **정밀도 문제**가 발생
            - 접근수가 많은 아이템의 유사도가 **상대적으로 높게 평가되기 때문!**
            - 점수의 값이 **어느정도의 유사도**를 나타내는지 점수만으로 확인이 어려움
                - 해당 점수가 높은 것인지, 낮은 것인지 판단이 어려움
                - 다른 아이템과 비교해서 상대적인 점수의 위치를 확인해야 함
                    - 이에 해결책은 ⭐️**벡터 정규화**⭐️
        - 벡터 정규화
            - 벡터를 모두 **같은 길이**로 만든다는 의미
            - 벡터의 각 수치를 **제곱** 후 더한 뒤, 제곱근을 취한 것 -> **벡터의 길이**
                - 두 점간의 거리를 계산할 때 사용하는 `Euclidean distance`(유클리드 거리) 정의와 동일
            - norm : 벡터의 크기
                - 노름으로 벡터의 각 수치를 나누면, 벡터의 `norm = 1`⭐️⭐️⭐️
                - 이렇게 정규화 하는 방식을 ⭐️`L2 정규화`⭐️ 또는 `2 norm 정규화`라고 부름
                
                —> 혹쉬 잘 이해가 안된다면.......
                
                [벡터 크기의 이해와 정규화 과정](https://ballbot.tistory.com/41)
                
            
            다음 코드는~
            
            - 사용자의 아이템에 대한 점수 집합을 **아이템 벡터**로 만든 이후, **L2 정규화**하는 쿼리
            - 벡터 norm의 계산에는 `SUM()`, `OVER()`함수와 `SQRT()`함수를 사용
            - 이렇게 구한 `norm`으로 각각의 점수를 나누어 `norm_score`를 계산
            
            ### **CODE.23.3. 아이템 벡터를 L2 정규화하는 쿼리**
            
            ```sql
            WITH
            ratings AS (
              -- CODE.23.1.
            )
            , product_base_normalized_ratings AS (
              -- 아이템 벡터 정규화하기
              SELECT
                user_id
                , product
                 --벡터
                , score
                 --벡터의 크기
                , SQRT(SUM(score * score) OVER(PARTITION BY product)) AS norm
                 --벡터정규화(벡터/벡터의 크기)
                , score / SQRT(SUM(score * score) OVER(PARTITION BY product)) AS norm_score
              FROM
                ratings
            )
            SELECT *
            FROM
              prodduct_base_normalized_ratings
            ;
            ```
            
            - 결과 —>상품별로 norm이 같음,
                
                ![스크린샷 2022-05-01 22.21.51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1dc8adf4-177f-4b11-b221-70beec397f7f/스크린샷_2022-05-01_22.21.51.png)
                
            
            - 정규화 시에 무조건 벡터의 크기는 **1 ( 벡터의 값이 한개밖에 없는 D002의 norm_score는 1.0으로 정규화 되어있다, 상품별로 norm_score제곱을 다 더하면 1 )**
            
            이어서, 정규화된 점수를 사용하여 **아이템의 유사도**를 계산
            
            - 코드 예에서는 **같은 아이템 페어**도 함께 유사도 계산
            
            ### **CODE.23.4. 정규화된 점수로 아이템의 유사도를 계산하는 쿼리**
            
            ```sql
            WITH
            ratings AS (
              -- CODE.23.1.
            )
            , product_base_normalized_ratings AS (
              -- CODE.23.3.
            )
            SELECT
              r1.product AS target
              , r2.product AS related
              -- 모든 아이템을 열람/구매 한 사용자 수
              , COUNT(r1.user_id) AS users
              -- 스코어들을 곱하고 합계를 구해 유사도 계산하기
              , SUM(r1.norm_score * r2.norm_score) As score
              -- 상품의 유사도 순위 구하기
              , ROW_NUMBER()
                OVER(PARTITION BY r1.product ORDER BY SUM(r1.norm_score * r2.norm_score) DESC)
                AS rank
            FROM
              product_base_normalized_ratings AS r1
              JOIN
                product_base_normalized_ratings AS r2
                -- 공통 사용자가 존재하면 상품 페어 만들기
                ON r1.user_id = r2.user_id
            GROUP BY
              r1.product, r2.product
            ORDER BY
              target, rank
            ;
            ```
            
            - 위 쿼리의 결과로
                - 같은 아이템 페어의 유사도는 **반드시 1**
                - 벡터를 정규화 하면, 자기 자신과의 내적값은 반드시 `1`이 되며, 내적의 최댓값 역시 `1`
            
            ![스크린샷 2022-05-01 22.39.38.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c0fd4f5-1758-4b30-9358-970f78d0e8bd/스크린샷_2022-05-01_22.39.38.png)
            
            - 모든 벡터의 값이 **양수**라면,
                - **내적의 최솟값은 0**
            - 값의 의미
                - 내적의 값이 0; 유사성이 없는 아이템
                - 내적의 값이 1; 완전히 일치하는 아이템
                
                —> 유사 이미지 매핑도 이런 방식으로 설계된듯 하다[Croquis-lens 대시보드 구축](https://www.notion.so/Croquis-lens-ae1d0975ee85412fb9186f71db52dd2e) 
                
            - 벡터를 `L2 정규화`하여 내적한 값
                - `2개의 벡터를` `n`차원에 매핑하였을 때, 이루는 각도에 `cos`값을 취한 값과 동일
                - `코사인 유사도`라고 부름 ( 크기 둘다 1인 벡터를 한쪽에 코사인으로 내려 꽂은 후 곱한것)
                
                  —> 혹쉬 이해가 잘 안된다면....
                
                [코사인 유사도 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EC%BD%94%EC%82%AC%EC%9D%B8_%EC%9C%A0%EC%82%AC%EB%8F%84)
                
            
            ### **정리**
            
            - 아이템 유사도를 계산하는 방법, **벡터의 내적**
            - 이때 벡터로 `아이템 조회 수`와 `구매 수`를 사용하였는데,
                - 실제 서비스 추천시 사용하려면, **서비스 특성에 고려하여 벡터 구성**
            
    - 3.당신을 위한 추천 상품 - 526p
        - 당신을 위한 추천 상품(User to Item)
            - 사용자와 관련된 추천
            - 웹사이트 최상위 페이지
            - 사용자의 마이 페이지
            - 검색 결과가 나오지 않았을 경우 출력하는 페이지는 물론이고,
                - 메일 매거진, 푸시 통지 등의 다양한 상황에 활용할 수 있음
        - `Item to Item`은 아이템의 유사도만 계산하면 되지만
            - `User to Item`은
                - **사용자와 사용자의 유사도**를 계산하고,
                - **유사 사용자가 흥미를 가진 아이템**을 구매해야 함
        - 다음 코드는 **사용자들의 유사도**를 계산하는 쿼리
            - `L2 정규화`를 적용하기는 했으나,
            - 사용자의 **백터 내적**을 계산할 수 있게, 사용자 별로 **벡터 노름**을 계산
        
        ### **CODE.23.5. 사용자끼리의 유사도를 계산하는 쿼리**
        
        ```sql
        WITH
        ratings AS (
          -- CODE.23.1.
        )
        , user_base_normalized_ratings AS (
          -- 사용자 벡터 정규화하기
          SELECT
            user_id
            , product
            , score
            -- PARTITION BY user_id으로 사용자별 벡터 노름 계산하기
            , SQRT(SUM(score * score) OVER(PARTITION BY user_id)) AS norm
            , score / SQRT(SUM(score * score) OVER(PARTITION BY user_id)) AS norm_score
          FROM
            ratings
        )
        , related_users AS (
          -- 경향이 비슷한 사용자 찾기
          SELECT
            r1.user_id
            , r2.user_id AS related_user
            , COUNT(r1.product) AS products
            , SUM(r1.norm_score * r2.norm_score) AS score
            , ROW_NUMBER()
                OVER(PARTITION BY r1.user_id ORDER BY SUM(r1.norm_score * r2.norm_score) DESC) AS rank
          FROM
            user_base_normalized_ratings AS r1
            JOIN
              user_base_normalized_ratings AS r2
              ON r1.product = r2.product
          WHERE
            r1.user_id <> r2.user_id
          GROUP BY
            r1.user_id, r2.user_id
        )
        SELECT *
        FROM
          related_users
        ORDER BY
          user_id, rank
        ;
        ```
        
        - with
            
            ![스크린샷 2022-05-01 21.43.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7aa8e0f7-b44d-4bff-a1b0-2de16932528d/스크린샷_2022-05-01_21.43.27.png)
            
        - 결과
            
            ![스크린샷 2022-05-08 16.53.42.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4fa98565-74ce-400b-9ace-7da9fc7f5ff1/스크린샷_2022-05-08_16.53.42.png)
            
        - 유사 사용자 순위를 계산했다면,
            - 유사 사용자가 흥미 있어 하는 아이템 목록을 추출하고 **아이템 순위**를 만듭니다.
        - 다음 코드는 **높은 순위**에 있는 유사 사용자를 기반으로 **추천 아이템**을 추천하는 쿼리
        - 유사 사용자 전체를 사용해 **아이템 목록**을 추출하면 계산량이 너무 많아짐
        - 낮은 순위의 **유사 사용자**는 큰 의미가 없는 데이터를 만들어낼 가능성이 있습니다.
            - 유사도 상위 `n`명을 사용해 추천 상품을 계산하고 있습니다.
        - 추천 상품의 아이템에는 사용자가
            - 기존에 구매한 상품이 포함될 수 있음
        - 이 코드에 구매한 상품을 제외하는 코드를 넣는다면 더욱 효율적으로 추천할 수 있음
        
        ### **CODE.23.6. 순위가 높은 유사 사용자를 기반으로 추천 아이템을 추출하는 쿼리**
        
        ```sql
        WITH
        ratings AS (
          -- CODE.23.1.
        )
        , user_base_normalized_ratings AS (
          -- CODE.23.5.
        )
        , related_users AS (
          -- CODE.23.5.
        )
        , related_user_base_products AS (
          SELECT
            u.user_id
            , r.product
            , SUM(u.score * r.score) * AS score
            , ROW_NUMBER()
                OVER(PARTITION BY u.user_id ORDER BY SUM(u.score * r.score) DESC)
            AS rank
          FROM
            related_users AS u
            JOIN
              ratings AS r
              ON u.related_user = r.user_id
          WHERE
        ----⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️
            u.rank <= 1
          GROUP BY
            u.user_id, r.product
        )
        SELECT *
        FROM
          related_user_base_products
        ORDER BY
          user_id
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 17.04.06.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e6c01186-e8aa-4f18-a017-62f946434b03/스크린샷_2022-05-08_17.04.06.png)
            
        
        - 사용자의 **아이템 구매 수**를 활용해 이미 구매한 아이템을 필터링하는 쿼리는 다음과 같음
        - 구매하지 않은 상품만 결과로 출력할 것이므로
            - `User to Item` 결과에 `LEFT JOIN`으로 구매 수를 결합하고,
                - 아이템의 구매가 `0`또는 `NULL`인 아이템을 압축한 뒤 순위를 생성합니다
        
        ### **CODE.23.7. 이미 구매한 아이템을 필터링하는 쿼리**
        
        ```sql
        WITH
        ratings AS (
          -- CODE.23.1.
        )
        , user_base_normalized_ratings AS (
          -- CODE.23.5.
        )
        , related_users AS (
          -- CODE.23.5.
        )
        , related_user_base_products AS (
          -- CODE.23.6.
        )
        SELECT
          p.user_id
          , p.product
          , p.score
          , ROW_NUMBER()
              OVER(PARTITION BY p.user_id ORDER BY p.score DESC) AS rank
        FROM
          related_user_base_products AS p
          LEFT JOIN
            ratings AS r
            ON p.user_id = r.user_id
            AND p.product = r.product
        WHERE 
          -- 대상 사용자가 구매하지 않은 상품만 추천하기 ⭐️⭐️⭐️⭐️⭐️⭐️⭐️⭐️
          COALESCE(r.purchase_count, 0) = 0
        ORDER BY
          p.user_id
        ;
        ```
        
        - `User to Item`의 경우 등록한지 얼마 안 된
            - 사용자와 게스트 사용자는, 데이터가 충분하지 않으므로, 제대로 예측하기가 어려움
        - 따라서 등록하지 얼마 안 되는 사용자는 `User to Item`이 아닌
            - 다른 추천 로직(랭킹 또는 콘텐츠 기반)을 적용하는 것이 좋음
        
        ### **정리**
        
        - 이번 절에서 소개한 **사용자 행동 로그**를 기반으로 **유사 아이템**을 찾고
            - 추천하는 등의 기능은 **협업 필터링**이라고 부르는 개념의 한가지 구현 방법
        
    - 4.추천 시스템을 개선할 때의 포인트 - 532p
        
        다들 한번씩 읽어보세용 ㅎㅎ passss!
        
    - 5.출력할 때 포인트 -534p
        
        다들 한번씩 읽어보세용 ㅎㅎ passss!
        
    - 6.추천과 관련된 지표 - 536p
        
        다들 한번씩 읽어보세용 ㅎㅎ passss!
        
- ***24강 : 점수 계산하기 - 538p~***
    - 여러 지표들을 조합하여,
        - 범위를 변화 시켜 순위를 구할때 활용할 수 있는 다양한 계산 방법 소개
    - 1.여러 값을 균형있게 조합해서 점수 계산하기 - 538p
        - 데이터 분석시에는 **여러 지표**를 비교하여 종합적으로 판단 필요
            - **재현율**과 **적합률**의 균형 잡기
            - `CTR`, `CTR`의 관계
            - `지지도`와 `확산도` 등
        - 여러 값을 **조합**한 뒤, **집약**해서 쉽게 비교, 검토하는 방법 소개
            - **재현율**과 **적합률**을 조합하고
            - 이를 기반으로 검색 엔진의 **정밀도**를 정량적으로 평가
        - 데이터
            - 세로로 저장한 데이터(데이터 24-1)
                - `path`, `recall`, `precision`
            - 가로로 저장한 데이터(데이터 24-2)
                - `path`, `index`, `value`
        
        ### **세 종류의 평균**
        
        - 여러 값을 집약하는 **가장 대표적인 방법**
        - **산술 평균**
            - 각각의 모두 값을 더하여, 전체 개수로 나누기
            - 가장 기본적인 평균
        - **기하 평균**
            - 각각의 값을 곱한 뒤, 개수만큼 제곱근을 한 값
            - `n제곱근`을 걸기 때문에, 실수값을 얻기 위해, 각 값은 양수여야 함
            - **성장률**과 **이율**등을 구할 때 사용
            - 여러 지표를 여러번 **곱해야하는 경우**, 기하평균이 적합함
        - **조화 평균**
            - 각 값의 **역수의 산술평균**을 구하고, 다시 **역수**를 취한 값
            - 역수를 취해야 하기 때문에, 각 값이 `0`이 되면 안됨
            - **비율을 나타내는 값의 평균**을 계산할 때 유용하게 활용할 수 있음
            - **평균 속도 계산**에서 많이 사용
        
        ### **평균 계산하기**
        
        - 세로 데이터의 평균을 구하는 쿼리
        - **기하 평균**과 **조화 평균**을 동시에 구할 수 있도록
            - `WHERE`구문에서 **재현율**과 **적합률**이 `0`이 아니고, 곱했을 때 `0`보다 큰 것만 추출하도록 함⭐️
        - **기하 평균** 계산시에 `SQRT`를 사용하여 제곱근을 구해도 되지만,
            - 3개의 값 이상의 평균에서도 대응할 수 있도록 `POWER`함수에 매개변수 `1/2`를 넣어 계산
        
        ### **CODE.24.1. 세로 데이터의 평균을 구하는 쿼리**
        
        ```sql
        SELECT
          *
          -- 산술 평균
          , (recall + precision) / 2 AS arithmetic_mean
          -- 기하 평균
          , POWER(recall * precision, 1.0 / 2) AS geometric_mean
          -- 조화 평균
          , 2.0 / ((1.0 / recall) + (1.0 / precision)) AS harmonic_mean
        FROM
          search_evaluation_by_col
          -- 값이 0보다 큰 것만으로 한정하기
        WHERE recall * precision > 0
        ORDER BY path
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 17.29.46.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0ea28f60-3617-4315-8742-7b8600c111fc/스크린샷_2022-05-08_17.29.46.png)
            
        - **재현율**과 **적합률**이 모두 `50.0`으로 같은 값이라면
            - **산술 평균**도 모두 `50.0`
        - 두 값이 다를 경우는 반드시 `산술 평균 > 기하 평균 > 조화 평균`순
        
        - 가로 데이터의 산술 평균, 기하 평균, 조화 평균을 구하려면
            - 값이 `0`이 되는 `poath`를 제외할 수 있게 `WHERE` 구문으로 `value > 0`으로 한정
            - `HAVING` 구문으로, 값의 결손이 없는 `path`로 한정
        - **산술 평균**은 일반적인 집약 함수인 `AVG` 사용
            - **조화 평균**도 `AVG`와 **역수 계산**만 유의하여 사용
        - **기하 평균**은 별도의 함수를 제공하지 않음
            - `log`를 활용하여 **값의 곱셈 -> 로그 덧셈**으로 변환
            
            `# 2와 8의 기하 평균
            b(log_b(2) + log_b(8))/2`
            
        
        ### **CODE.24.2. 가로 기반 데이터의 평균을 계산하는 쿼리**
        
        ```sql
        SELECT
          path
          -- 산술 평균
          , AVG(value) AS arithmetic_mean
          -- 기하 평균(대수의 산술 평균)
          -- PostgreSQL, Redshift, 상용 로그로 log함수 사용하기
          , POWER(10, AVG(log(value))) AS geometric_mean
          -- Hive, BigQuery, SparkSQL, 상용 로그로 log10 함수 사용
          , POWER(10, AVG(log10(value))) AS geometric_mean
          -- 조화 평균
          , 1.0 / (AVG(1.0 / value)) AS harmonic_mean
        FROM
          search_evaluation_by_row
          -- 값이 0보다 큰 것만으로 한정
        WHERE value > 0
        GROUP BY path
        -- 빠진 데이터가 없게 path로 한정
        HAVING COUNT(*) = 2
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 17.33.41.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2548f2b7-f0b6-49d1-8fbd-92ec736bdedd/스크린샷_2022-05-08_17.33.41.png)
            
        - **재현율**과 **적합률**의 조화 평균은
            - 두 값을 **통합적으로 평가**할 때 자주 사용
            - `F 척도`(f-measure)또는 `F1 스코어`라는 별도의 이름으로 지칭
        
        ### **가중 평균**
        
        - **재현율**과 **적합률**에 가중치의 비율을 다르게 하기
            - 검색 결과의 포괄성을 고려하면서도,
            - 검색 결과 상위에 있는 요소들의 타당성을 중요시하여
            - 검색 엔진을 평가하기위해
                - `재현율 : 적합률 = 3:7`
        - 가중치가 부여된 평균 계산 방법
            - `산술 평균` = 각 값에 가중치 곱하고, 가중치의 합으로 나누기
            - `기하 평균` = 가중치만큼 제곱, 합계만큼 제곱근
            - `조화 평균` = 각 값의 역수에 가중치 곱하여 산술 평균, 이후 역수
        - 가중치의 합이 `1.0`이 되면
            - 가중치의 합으로 나누거나, 제곱근 구하는 등의 계산이 **필요 없음⭐️**
        - 가중치의 합계를 `1.0`으로 설정할 수 있게,
            - 가중치 : `재현율 = 0.3, 적합률 = 0.7`
            
        
        ### **CODE.24.3. 세로 기반 데이터의 가중 평균을 계산하는 쿼리**
        
        ```sql
        SELECT
          *
          -- 가중치가 추가된 산술 평균
          , 0.3 * recall + 0.7 * precision AS weighted_a_mean
          -- 가중치가 추가된 기하 평균
          , POWER(recall, 0.3) * POWER(precision, 0.7) AS weighted_g_mean
          -- 가중치가 추가된 조화 평균
          , 1.0 / (((0.3) / recall) + (0.7 / precision)) AS weighted_h_mean
        FROM
          search_evaluation_by_col
        -- 값이 0보다 큰 것만으로 한정
        WHERE recall * precision > 0
        ORDER BY path
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 17.43.33.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a37b2efb-446f-437d-a4b0-712b3763363b/스크린샷_2022-05-08_17.43.33.png)
            
        
        - 가로 기반 테이블 데이터의 가중 평균 계산
            - 무게를 `SELECT ... CASE`로 나타낼 수 있지만, 쿼리가 길어지므로,
                - 임시 테이블`(weight)`을 사용하여, 가중치 **마스터 테이블**을 생성하여
                - 이를 결합하는 방식으로 사용
        - `AVG`를 사용하면, **레코드 행 수로 나눈 평균**을 구하므로
            - 대신 `SUM`함수를 사용
        
        ### **CODE.24.4. 가로 기반 테이블의 가중 평균을 계산하는 쿼리**
        
        ```sql
        WITH
        weights AS (
          -- 가중치 마스터 테이블(가중치의 합계가 1.0이 되도록 설정)
                    SELECT 'recall'   AS index, 0.3 AS weight
          UNION ALL SELECT 'precision' AS index, 0.7 AS weight
        )
        SELECT
          e.path
          -- 가중치가 추가된 산술 평균
          , SUM(w.weight * e.value) AS weighted_a_mean
          -- 가중치가 추가된 기하 평균
          -- PosgreSQL, Redshift, log 사용
          , POWER(10, SUM(w.weight * log(e.value))) AS weighted_g_mean
          -- Hive, BigQuery, SparkSQL, log10 함수 사용
          , POWER(10, SUM(w.weight * log10(e.value))) AS weighted_g_mean
        
          -- 가중치가 추가된 조화 평균
          , 1.0 / (SUM(w.weight / e.value)) AS weighted_h_mean
        FROM
          search_evaluation_by_row AS e
          JOIN
            weights AS w
            ON e.index = w.index
          -- 값이 0보다 큰 것만으로 한정하기
        WHERE e.value > 0
        GROUP BY e.path
        -- 빠진 데이터가 없도록 path로 한정
        HAVING COUNT(*) = 2
        ORDER BY e.path
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 17.45.00.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f2ffa0de-2b09-4ae0-94f0-d77946455329/스크린샷_2022-05-08_17.45.00.png)
            
        
        ### **정리**
        
        - `3개 이상`의 값에 대해서도 같은 쿼리로 평균을 구할 수 있음
        - `3`개의 평균을 사용하면, 다양한 지표의 데이터 활용에 도움이 됨
    - 2.값의 범위가 다른 지표를 정규화해서 비교 가능한 상태로 만들기 - 546p
        - 값의 범위(스케일)이 다른 지표를 결합할 때는⭐️⭐️⭐️
            - ⭐️정규화⭐️라는 ⭐️**전처리**⭐️를 해주어야 함
        - 샘플
            - 상품 열람 수, 구매 수 데이터(데이터 24-3)
                - `user_id, product, view_count, purchase_count`
            - 각각의 값을 정규화 해야함
            - 상품 구매수는 `0 | 1`이지만, 열람 수는 `21, 49`와 같은 비교적 큰 수
        - `열람 수`와 `구매 수`를 모두 ⭐️⭐️ `0 ~ 1`의 실수로 변환⭐️⭐️해야 함
        
        ### **Min-Max 정규화**
        
        - ⭐️서로 다른 값의 **폭**을 가지는 각 지표를 `0~1`의 스케일로 정규화 하는 방법⭐️
        - 각 지표의 `min`과 `max`를 구하고, 변환후의 최소 최대를 `0.0 ~ 1.0`이 되게 반영
        - ⭐️공식 : 각각의 지표를 `min`으로 뺀 뒤,`max - min`을 나누는 방식⭐️
        
        - `열람 수`와 `구매 수`에 `Min-Max` 정규화를 적용하는 쿼리
        - 레코드 전체의 `min`, `max`를 구하기 위해 `min/max 윈도 함수 사용`
        
        ### **CODE.24.5. 열람 수와 구매 수에 Min-Max 정규화를 적용하는 쿼리**
        
        ```sql
        SELECT
          user_id
          , product
          , view_count AS v_count
          , purchase_count AS p_count
          , 1.0 * (view_count - MIN(view_count) OVER())
            -- PostgreSQL, Redshift, BigQuery, SparkSQL의 경우 `NULLIF`로 0으로 나누는 것 피하기
            --nullif(a,b)-->a,b가 같으면 null값,다르면 a
            / NULLIF((MAX(view_count) OVER() - MIN(view_count) OVER()), 0)
            AS norm_v_count
          , 1.0 * (purchase_count - MIN(purchase_count) OVER())
            -- PostgreSQL, Redshift, BigQuery, SparkSQL의 경우 `NULLIF`로 0으로 나누는 것 피하기
            / NULLIF((MAX(purchase_count) OVER() - MIN(purchase_count) OVER()), 0)
            AS norm_p_count
        FROM action_counts
        ORDER BY user_id, product;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 18.03.32.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13d5fafb-b1e8-471c-8c64-43a5e235faca/스크린샷_2022-05-08_18.03.32.png)
            
        - `norm_v_count` : 정규화 후의 열람 수
        - `norm_p_count` : 정규화 후의 구매 수
            - 구매 수의 경우, 최솟값과 최대값이 기존 `0, 1`이므로, 값이 변화되지 않음
        
        ### **시그모이드 함수로 변환하기**
        
        - ⭐️`Min-Max 정규화`의 문제⭐️
            - **모집단**의 변화에 따라, **정규화 후**의 값이 모두 변경 됨
                - ex) 최댓값 또는 최솟값이 변하는 경우
            - 정규화 계산시, 데이터의 수에 따라 계산 비용이 의존적
        - ⭐️이 문제를 극복하기 위해 ⭐️**결과값이 0~1**인 함수를 사용하여⭐️, 데이터를 변환하는 정규화 방법 사용
        - 시그모이드 함수 : `0`이상의 값을 `0~1`의 범위로 변환해주는 `S`자 곡선을 그리는 함수
        - 그 중에서도 `y=1/(1+e^(-ax))` 함수가 많이 사용됨
            - 이 때, `a=1`인 함수를 **표준 시그모이드 함수**로 부름
        - 이러한 시그모이드 함수를
            - ⭐️`x=0`일 때, `y=0`⭐️
            - ⭐️`x=`무한대, `y=1`⭐️이 되도록 변경
            - 매개변수 `a`(gain)에 적당한 값을 넣어 지표 조정
            
        - 시그모이드 함수를 사용해 변환하는 쿼리
            - 열람수에는 적당하게 `gain 0.1`
            - 구매수에는 적당하게 `gain 10`을 적용
        - `gain`을 크게 적용할경우, **입력 값**이 조금 커지는 것만으로도, 변환후의 값이 `1`에 가까워짐
        
        ### **CODE.24.6. 시그모이드 함수를 사용해 변환하는 쿼리**
        
        ```sql
        SELECT
          user_id
          , product
          , view_count AS v_count
          , purchase_count AS p_count
          -- gain을 0.1로 사용한 sigmoid 함수
          , 2.0 / (1 + exp(-0.1 * view_count)) - 1.0 AS sigm_v_count
          -- gain을 10으로 사용한 sigmoid 함수
          , 2.0 / (1 + exp(-10 * purchase_count)) - 1.0 AS sigm_p_count
        FROM action_counts
        ORDER BY user_id, product;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 18.26.35.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2889fe0c-1dbc-48d5-a166-9f6d32c7b5a0/스크린샷_2022-05-08_18.26.35.png)
            
        - `sigmoid`를 적용한
            - **열람 수**
                - `10 -> 0.4621`
                - `49 -> 0.9852`
            - **구매 수**
                - gain이 `10`이므로, 기존 값(`0 ~ 1`)과 큰 차이가 없음
        - ⭐️`Min-Max 정규화`와 다르게 값이 `2`배가 된다고, 변환된 값이 `2`배가 되는 것이 아님⭐️
            - sigmoid 자체가 곡선이므로,
            - ⭐️`0`에 가까울 수록⭐️ **변환 후**의 값에 미치는 **영향이 큼**
        - 위 특징은 **열람 수**를 다룰 때 매우 좋음⭐️
            - 열람 수가 `1`회인 상품과, `10`회인 상품은 **관심도**의 차이가 매우 크지만
            - 열람 수가 `10001`회인 상품과, `1010`회인 상품은, 크게 관심도 차이가 나지 않음
        - 즉, **sigmoid** 함수를 사용해 **비선형 변환**을 하는 것이 더 직관적으로 이해할 수 있음
        
        ### **정리**
        
        - **값의 범위**(스케일)이 다른 지표를 다루는 여러가지 방법
        - 어떤 방법이 적합할지는 **데이터의 특성**에 따라 다름
        - 직접 사용해보면서, 어느쪽이 더 효율적일지 확인 필요
    - 3.각 데이터의 편차 값 계산하기 - 551p
        - 데이터의 **분포**가 **정규 분포**에 가깝다는게 어느정도 확실하다면,
            - **정규값**과 **편차값**을 이용하는 경우가 많음
        - 일반적인 **편차값**의 정의와
            - 이를 `SQL`로 구하는 방법
        
        ### **표준편차, 정규값, 편차값**
        
        ### **표준편차**
        
        - 데이터의 **쏠림 상태**을 나타내는 값
            
            `# 모집단의 표준편차를 구할 때
            # SQL : stddev_pop
            표준편차 = v((각 데이터 - 평균)^2을 모두 더한 것 / 데이터의 수)
            
            # 모집단의 일부(표본)를 기반으로 표준편차를 구할 때
            # SQL : sqddev
            표준편차 = v((각 데이터 - 평균)^2를 모두 더한것 / 데이터의 개수 - 1`⭐️`)`
            
        - 표준편차의 `min = 0`
        - 표준편차가 커질수록 **데이터에 쏠림**이 존재
        
        ### **정규값**
        
        - 평균으로부터 **얼마나 떨어져 있는지**
        - 데이터의 **쏠림 정도**를 기반으로 **점수의 가치**를 평가하기 쉽게
            - 데이터를 변화하는 것 -> **정규화**
        - 정규화된 데이터를 ⭐️**정규값**⭐️이라고 함
            
            ⭐️`정규값 = (각각의 데이터 - 평균) / 표준편차`
            
        - 정규값의 평균은 `0`, 표준 편차는 `1`
        - 이 특징을 활용하면 ⭐️서로 다른 데이터, 단위가 다른 데이터(`e.g. CTR, CVR`)도 쉽게 비교 가능
        
        ### **편차값**
        
        - **조건**이 다른 데이터를 쉽게 비교하고 싶을 때
        - **정규값**을 사용하여 계산
            
            ⭐️`편차값 = 각 정규화된 데이터 * 10 + 50`
            
        - 편차값의 평균은 `50`, 표준편차는 `10`
        
        ### **편차 계산하기**
        
        - 편차값은 `10 * (각각의 값 - 평균 값 / 표준편차) + 50`으로 계산 가능
        - **평균값**과 **표준편차**를 사용해 각 값을 계산하므로 **윈도 함수**를 사용해 구해야 함
        
        - 윈도 함수를 사용하여 과목들의 **표준편차**, **기본값/편차값**을 계산하는 쿼리
        
        ### **CODE.24.7. 표준편차, 기본값, 편차값을 계산하는 쿼리**
        
        ```sql
        SELECT
          subject
          , name
          , score
          -- 과목별로 표준편차 구하기
          , stddev_pop(score) OVER(PARTITION BY subject) AS stddev_pop
          -- 과목별 평균 점수 구하기
          , AVG(score) OVER(PARTITION BY subject) AS avg_score
          -- 점수별로 기준 점수 구하기 - 과목별 정규화
          , (score - AVG(score) OVER(PARTITION BY subject))
            / stddev_pop(score) OVER(PARTITION BY subject) AS std_value
          -- 점수별로 편차값 구하기
          , 10.0 * (score - AVG(score) OVER(PARTITION BY subject))
            / stddev_pop(score) OVER(PARTITION BY subject) + 50 AS deviation
        FROM exam_scores
        ORDER BY subject, name;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 18.32.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0cf51385-d9d8-483c-95fc-4c38d47cf48c/스크린샷_2022-05-08_18.32.27.png)
            
        
        ### **CODE.24.8. 표준편차를 따로 계산하고, 편차값을 계산하는 쿼리—>패스**
        
        ```sql
        WITH
        exam_stddev_pop AS (
          -- 다른 테이블에서 과목별로 표준편차 구해두기
          SELECT
            subject
            , stddev_pop(score) AS stddev_pop
          FROM exam_scores
          GROUP BY subject
        )
        SELECT
          s.subject
          , s.name
          , s.score
          , d.stddev_pop
          , AVG(s.score) OVER(PARTITION BY s.subject) AS avg_score
          , (s.score - AVG(s.score) OVER(PARTITION BY s.subject)) / d.stddev_pop AS std_value
          , 10.0 * (s.score - AVG(s.score) OVER(PARTITION BY s.subject)) / d.stddev_pop + 50 AS deviation
        FROM
          exam_scores AS s
          JOIN
            exam_stddev_pop AS d
            ON s.subject = d.subject
        ORDER BY s.subject, s.name;
        ```
        
        ### **정리**
        
        - **기본값**과 **편차값** 등의 지표를 `SQL`로 계산하는 방법은
            - **윈도 함수**의 등장 덕분에, 매우 쉬워짐
        - **지표 정의**와 **의미**를 확실하게 확인하여, **의미 있는 데이터 분석**을 할 것
    - 4.거대한 숫자 지표를 직감적으로 이해하기 쉽게 가공하기 - 556p
        - 사람이 직감적으로 이해하기 쉬운 값으로 변형하는 **비선형적인 함수**로 **로그**를 사용하는 방법
        - `사용자 액션 수`, `경과일 수`처럼
            - 값이 크면 클수록 **값 차이**가 큰 영향을 주지 않는경우,
            - **로그**를 사용해 데이터를 변환하면, 이해하기 쉽게 가공 가능
        - 샘플(데이터24-4)
            - `dt`, `user_id`, `product`, `v_count`, `p_count`
                - 사용자별 상품 열람 수(`v_count`)
                - 구매 수(`p_count`)
        - **23강 2절**에서는
            - 모든 기간의 `열람 수 + 구매 수`를 하여, **상품 점수**를 계산
            - 이번절에서는
                - **오래된** 날짜의 액션일수록, **가중치를 적게** 주는 방법 사용
        - 사용자별로 **최종 액션일**, **각각의 레코드와 최종 액션일과의 날짜 차이** 계산
        
        ### **CODE.24.9. 사용자들의 최종 접근일과 각 레코드와의 날짜 차이를 계산하는 쿼리**
        
        ```sql
        WITH
        action_counts_with_diff_date AS (
          SELECT *
            -- 사용자별로 최종 접근일과 각 레코드의 날짜 차이 계산
            -- PostfreSQL, Redshift의 경우 날짜끼리 빼기 연산 가능
            , MAX(dt::date) OVER(PARTITION BY user_id) AS last_access
            , MAX(dt::date) OVER(PARTITION BY user_id) - dt::date AS diff_date
          FROM
            action_counts_with_date
        )
        SELECT *
        FROM action_counts_with_diff_date;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 18.38.53.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c099d689-a812-446a-86d7-b08700abb8e3/스크린샷_2022-05-08_18.38.53.png)
            
        - `최종 액션일`과의 차이가 크면 클수록, 가중치를 낮게 하는 방법
            - 날짜 차이에 **역수** 취하기?
                - 단순하게 역수를 취하게 되면 `2일 전, 1/2`, `30일 전, 1/30`이 되기 때문에, 극단적으로 작아짐
                - 따라서 **날짜 차이**에 **로그**를 취해
                    - 날짜가 증가할수록 가중치가 **완만하게 감소**하게 변경
            - 추가로, 날짜 차이가 `0`일 때, 가중치의 최댓값이 `1`이 될 수 있게 하고
            - 추가적인 **매개 변수**를 사용해, **가중치의 변화율**을 조정할 수 있도록 매개변수 a를 둔다
        - 수식
            
            `y = 1/(log2(ax+2)) {0 <= x}`
            
        - 매개변수가 `0`에 가까워질수록, 그래프가 **완만**해지며
        - `a=0`이 되어버리면, 가중치가 항상 `1`이 된다
        - 매개 변수를 크게 할수록 **가중치의 감소가 눈에 띄게 변화**
        
        ### **CODE.24.10. 날짜 차이에 따른 가중치를 계산하는 쿼리**
        
        ```sql
        WITH
        action_counts_with_diff_date AS (
          -- CODE.24.9.
        ), action_counts_with_weight AS (
          SELECT *
            -- 날짜 차이에 따른 가중치 계산하기(a = 0.1)
            -- PostgreSQL, Hive, SparkSQL, log(<밑수>, <진수>) 함수 사용
            , 1.0 / log(2, 0.1 * diff_date + 2) AS weight
          FROM action_counts_with_diff_date
        )
        SELECT
          user_id
          , product
          , v_count
          , p_count
          , diff_date
          , weight
        FROM action_counts_with_weight
        ORDER BY
          user_id, product, diff_date DESC
        ;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 18.41.41.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4b4cca67-a923-4911-88cb-ee4b3966be0d/스크린샷_2022-05-08_18.41.41.png)
            
        
        - 최종 접근일과의 날짜 차이가 `0`일때, 가중치가 `1.0`으로 가장 크고,
            - 최종 접근일과의 날짜 차이가 커질수록, 가중치가 완만하게 줄어듦
        - 지금까지 구한 과정으로 구한 **가중치**를
            - **열람 수**와 **구매 수**에 곱해 점수를 계산
        - 날짜별로 **열람 수**와 **구매 수**에 해당 날짜의 **가중치**를 곱한 뒤 더해 점수 계산
        
        ### **CODE.24.11. 일수차에 따른 중첩을 사용해 열람 수와 구매 수 점수를 계산하는 쿼리**
        
        ```sql
        WITH
        action_counts_with_date AS (
          -- CODE.24.9.
        )
        , action_counts_with_weight AS (
          -- CODE.24.10.
        )
        , action_scores AS (
          SELECT
            user_id
            , product
            , SUM(v_count) AS v_count
            , SUM(v_count * weight) AS v_score
            , SUM(p_count) AS p_count
            , SUM(p_count * weight) AS p_score
          FROM action_counts_with_weight
          GROUP BY
            user_id, product
        )
        SELECT *
        FROM action_scores
        ORDER BY
          user_id, product;
        ```
        
        - 결과
            
            ![스크린샷 2022-05-08 18.44.16.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79b04888-54e9-4b62-9c92-2998e216288f/스크린샷_2022-05-08_18.44.16.png)
            
        - 날짜에 따라 가중치가 적용된 점수 산출 가능
        - 같은 구매 수의 상품이더라도, 날짜가 오래되었다면
            - 점수가 낮게 측정
        
        ### **정리**
        
        - `열람 수`와 `구매 수`처럼 **어떤 것을 세어 집계한 숫자**는
            - 점수를 계산할 때, **로그**를 취해서 값의 변화를 완만하게 표시가능
        - 이를 활용하면 사람이 더 **직감적으로** 쉽게 값 변화 인지 가능
        - 로그는 수학적 요소지만, **그 활용범위가 넓음**
    - 5.독자적인 점수 계산 방법을 정의해서 순위 작성하기 - 562p
    - 예시 데이터
        - 월별 상품 매출 수(`monthly_sales`)
        - `year_month, item, amount`
    - **순서 생성** 방침은
        - 1년 동안의 계절마다 **주기적으로 팔리는 상품**과
        - **최근 트랜드 상품**이 상위에 오도록 순위를 구성
    - 따라서⭐️`1년 전의 매출과`, `최근 1개월`의 매출 값으로 **가중 평균**을 내서 점수를 사용
    - `4분기별 상품 매출액과 합계`를 구하는 쿼리
        - 상품별 `2016 1q`, `4q`의 매출액을 사용하여 순위를 구함
    
    ### **CODE.24.12. 분기별 상품 매출액과 매출 합계를 집계하는 쿼리**
    
    ```sql
    WITH
    item_sales_per_quarters AS (
      SELECT item
        -- 2016.1q의 상품 매출 모두 더하기
        , SUM(
          CASE WHEN year_month IN ('2016-01', '2016-02', '2016-03') THEN amount ELSE 0 END
        ) AS sales_2016_q1
        -- 2016.4q의 상품 매출 모두 더하기
        , SUM(
          CASE WHEN year_month IN ('2016-10', '2016-11', '2016-12') THEN amount ELSE 0 END
        ) AS sales_2016_q4
      FROM monthly_sales
      GROUP BY item
    )
    SELECT
      item
      -- 2016.1q의 상품 매출
      , sales_2016_q1
      -- 2016.1q의 상품 매출 합계
      , SUM(sales_2016_q1) OVER() AS sum_sales_2016_q1
      -- 2016.4q의 상품 매출
      , sales_2016_q4
      -- 2016.4q의 상품 매출 합계
      , SUM(sales_2016_q4) OVER() AS sum_sales_2016_q4
    FROM item_sales_per_quaters
    ;
    ```
    
    ![스크린샷 2022-05-08 18.56.42.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0cc936f-3274-4dad-b391-63a3ec05f6d2/스크린샷_2022-05-08_18.56.42.png)
    
    - 위 출력 결과에서 `2016.1q`와 `4q`의 합계 매출액을 확인하면,
        - 서비스 전체의 매출이 상승 경향이 있는 것을 알 수 있음(각각 `254,000`, `550,000`)
    - 따라서 1분기 매출액과 4분기 매출액이 같다고 해도, **의미가 다름**
    
    - 다음 코드는 `1q`와 `4q`의 매출액을 정규화하여 점수를 계산하는 쿼리
        - 분모가 다른 2개의 지표를 비교할 수 있도록 ⭐️`Mix-Max 정규화` 사용
    
    ### **CODE.24.13. 분기별 상품 매출액을 기반으로 점수를 계산하는 쿼리**
    
    ```sql
    WITH
    item_sales_per_quarters AS (
      -- CODE.24.12
    )
    , item_scores_per_quarters AS (
      SELECT
        item
        , sales_2016_q1
        , 1.0
          * (sales_2016_q1 - MIN(sales_2016_q1) OVER())
          -- PostgreSQL, Redshift, BigQuery, SparkSQL, NULLIF로 divide 0 회피
          / NULLIF(MAX(sales_2016_q1) OVER() - MIN(sale_2016_q1) OVER(), 0)
          AS score_2016_q1
        , sales_2016_q4
        , 1.0
          * (sales_2016_q4 - MIN(sales_2016_q4) OVER())
          -- PostgreSQL, Redshift, BigQuery, SparkSQL, NULLIF로 divide 0 회피
          / NULLIF(MAX(sales_2016_q4) OVER() - MIN(sales_2016_q4) OVER(), 0)
          AS score_2016_q4
      FROM item_sales_per_quarters
    )
    SELECT *
    FROM item_scores_per_quarters
    ;
    ```
    
    ![스크린샷 2022-05-08 19.02.16.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/285f2a35-3998-45a2-8cbc-2b7b27c59c77/스크린샷_2022-05-08_19.02.16.png)
    
    - 다음으로, `1q`와 `4q`의 매출을 기반으로 **정규화된 점수**를 계산했다면,
        - `1q`와 `4q`의 **가중 평균**을 구하고, 하나의 점수로 집약하기
    - `1q : 4q = 7 : 3`으로 지정하고 가중 평균을 계산할때 다음과 같이 순위 생성
    
    ### **CODE.24.14. 분기별 상품 점수 가중 평균으로 순위를 생성하는 쿼리**
    
    ```sql
    WITH
    item_sales_per_quarters AS (
      -- CODE.24.12
    ),
    item_scores_per_quarters AS (
      -- CODE.24.13
    )
    SELECT
      item
      , 0.7 * score_2016_q1 + 0.3 * score_2016_q4 AS score
      , ROW_NUMBER()
          OVER(ORDER BY 0.7 * score_2016_q1 + 0.3 * score_2016_q4 DESC)
        AS rank
    FROM item_scores_per_quarters
    ORDER BY rank
    ;
    ```
    
    ![스크린샷 2022-05-08 19.03.51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a89711e6-4ad6-42c5-a91e-4d820e614d09/스크린샷_2022-05-08_19.03.51.png)
    
    ### **정리**
    
    - **주기 요소가 있는 상품**과 **트렌드 상품**을 균형 있게 조합한 **순위**를 생성할 수 있음
    - 추가로 **여러 가지 점수 계산 방식**을 조합하면 다양한 분야에 활용
