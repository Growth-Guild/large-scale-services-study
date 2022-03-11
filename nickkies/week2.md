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

#### Replication 구축 방식
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
