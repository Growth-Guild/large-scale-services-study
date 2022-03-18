# 2주차

---

## 3장. OS 캐시와 분산

### 강의 10. 국소성(locality)을 살리는 분산
-> 엑세스 패턴을 고려한 분산

#### 1. 파티셔닝
    1. 테이블 단위 분할
        -> 기존 테이블을 나누는 방식

    2. 테이블 데이터 분할
        -> 초성 / 날짜 단위로 나누는 방식
        -> 분할의 단위를 변경시 데이터를 한 번 병합

#### 2. 요청 패턴을 `섬`으로 분할
    1. 캐싱하기 쉬운 섬 (일반 사용자)
    2. 캐싱하기 어려운 섬 (특히 검색 봇)
    3. 이미지등 대용량 섬

#### 페이지 캐시를 고려한 운용의 기본 규칙
    1. OS 기동 직후에 서버를 투입하지 않을 것
        -> 캐시가 없기 때문에 서버가 내려가는 상황 발생 가능
        -> 자주 사용하는 DB파일등을 미리 cat해서 캐싱한 후 로드밸런서에 편입
        -> 충분히 캐시가 최적화 된 후 성능평가 / 부하성능 등을 테스트

#### 요약
    서비스의 서버를 설계할 때,
    향후 서비스의 확장도 생각하되,
    미니멈 스타트와의 최적점을 고려해서,
    최적의 scale-up / scale-out 전략을 짜고,
    scale-out일때는 국소성을 살리는 분산으로 설계한다.

---

## 4장. 분산을 고려한 MySQL 운용

#### DB scale-out 전략
    1. indexing
    2. MySQL 분산
    3. scale-out & partitioning

### 강의11. 인덱스를 올바르게 운용하기
-> 분산을 고려한 MySQL 운용의 대전제

#### 분산을 고려한 MySQL 운용의 포인트
    1. OS 캐시 활용
    2. indexing
    3. 확장을 전제로 한 설계(minimum start)

#### OS cache 활용
    1. 전체 데이터 크기에 주의
        - `데이터량 < 물리 메모리`를 유지
        - 메모리가 부족할 경우에는 증설 등
    2. 스키마 설계가 데이터 크기에 미치는 영향을 고려

#### index의 중요성 
-> B tree

    1. index = 색인
    2. B+트리
        - 외부기어장치 탐색 시에 Seek 횟수를 최소화하는 트리 구조
        - 색인의 계산량: O(n) -> O(logn) (400만 -> 25.25)
    * column을 조합해서 index를 만들기도 한다.

### 강의12. MySQL의 분산
-> 확장을 전제로 한 시스템 설계

#### MySQL의 Replication 기능
    master: CRUD
    slaves: Read only

#### master slave 
-> 참조계열(slave)은 확장하고 갱신계열(master)는 확장하지 않는다

#### PostgreSQL Replication
    master / stand by

#### PostgreSQL Replication 구축 방식
    1. WAL
        - master server의 모든 작업을 log남기고 (+ 파일로 저장됨)
        - stand by server로 전달
        - restore
    2. Log-Shipping
        - 일정 log가 쌓이면 파일 째로 전달
    3. Streaming
        - 거의 실시간
        - 파일 저장유무 x, 그냥 바로 보냄
        - 데이터 유실 우려가 있기 때문에 Log-Shipping방식으로 대처

### 강의13. MySQL의 scale-out과 partitioning
-> ROW_NUMBER() OVER (PARTITION BY ...) 아님~

#### MySQL의 scale-out 전략
    -> 메모리에 올라가지 않을 때 partitioning
    -> 국소성이 증가하는것 이외에 모든 것이 나빠짐...
    -> 첫글씨 / 날짜등으로... 
    -> 조건에 의한 캐싱데이터를 뷰테이블이나 새로운 테이블을 만들어서 관리하는 것도 방법
    -> 결국 상황에 따라서 정답은 다름

--- 

## 5장. 대규모 데이터 처리 실전 입문
-> application 개발의 급소

### 강의 14. 용도특화형 indexing
-> 대규모 데이터를 능수능란하게 다루기

#### index와 시스템 구성(RDBMS의 한계가 보일 때)
    1. 배치처리로 데이터 추출
    2. 별도 인덱스 서버를 만들어서 웹 API 등으로 쿼리

??!?!?!!? 

RDBMS 한 컬럼에 varchar 혹은 text 형식으로 json통째로 넣는 건??
ex) 동네예보 API response 

#### 용도특화형 indexing
-> Batch


### 요약
    1. GB 단위의데이터 처리
        - TB / PB와는 다름
    2. 메모리의 중요성
    3. 분산을 의식한 운용
    4. 적절한 알고리즘과 데이터 구조
