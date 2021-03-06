# 2주차
> p79 강의 10 ~ p125 강의 14

---
## 강의 10. 국소성(locality)을 살리는 분산
### 액세스 패턴을 고려한 분산
+ 액세스 패턴을 고려하지 않으면 서버가 다른 데이터 영역도 캐싱해야 한다.
+ 분산으로 메모리에 디스크 캐싱율을 높일 수 있다.
### 파티셔닝
+ DB 서버를 여러 대 서버로 분할하는 것.
1. 테이블별 분할
     + 테이블 크기를 고려해 더 많은 부분이 캐싱되도록 한다.
2. 하나의 테이블을 여러 테이블로 분할
     + id 순에 따라 분할하는 하테나 북마크 테이블
     + 단점은 분할의 입도시 재병합이 필요하다는 점이다.
### 용도별로 시스템을 섬으로 나누는 방법
1. 빈번하게 액세스되는 부분과
   + 정해진 테이블만 액세스하는 API
2. 검색봇과 같이 광범위한 액세스가 필요한 부분에 따라 분할을 통해 캐싱 성능을 높일 수 있다.
### 페이지 캐시를 고려한 운용
+ OS를 시작해서 기동하면 자주 사용하는 DB 파일을 한 번 cat해준다.  
  그렇게 하면 전부 메모리에 올라간다.  
  그리고 로드밸런서에 편입시켜준다.
+ 성능이나 부하시험시 기존 캐시가 최적화되었을 때 실시할 것!  
##### ※ 부하분산 → OS 동작원리를 먼저 학습할 것. 

---
##### DB 스케일 아웃 ▶ 1. 인덱스 / 2. MySQL / 3. 스케일아웃과 파티셔닝

---
## 강의 11. 인덱스를 올바르게 운용하기
### OS 캐시 활용
+ 일정 규모 이상이 되면, 칼럼 또는 스키마 변경에도 주의를 기울여야 한다.
    + 3억 레코드에서 1레코드당 1칼럼 8byte라고 하면, 8*3억 byte = 3GB
+ 스키마 설계가 데이터 크기에 미치는 영향을 고려해야 한다.
+ 정규화할 때는 속도와 데이터 상관관계를 고려하자!
### 인덱스(색인)의 중요성과 효과
+ 이분트리와 B트리
    + 이분트리는 노드의 자식이 반드시 2개 이하
    + B트리는 노드의 크기를 적당한 사이즈로 정할 수 있다. ex) m=5
      + 이분트리에 비해 디스크 seek 횟수가 적다.
      + 각 노드를 1블럭에 모아서 저장되도록 구성할 수 있다.
+ MySQL 인덱스인 B+트리 테이블 구조는 가장 최적화된 데이터 구조.
    + 노드 내 자식노드로의 포인터 값만 가지고 있으며, 맨 마지막 leaf node에만 실제값을 갖는 구조이다.
+ 계산량과 디스크 Seek 횟수 개선
    + 인덱스가 없는 선형탐색, O(n)
    + B+트리, O(log<sub>n</sub>)
+ 인덱스의 작용
    + 조건에 지정된 인덱스가 여러 개일 경우 1개의 인덱스만 작용한다.  
      즉, 복합인덱스가 필요.
    + explain 명령, Extra 열
##### ※ 파일정렬이나 임시 테이블의 문제

---
## 강의 12. MySQL의 분산
### MySQL의 레플리케이션 기능
+ 마스터, 슬레이브 서버를 나누어 마스터에 쓴 내용을 슬레이브가 폴링(polling)해서 갱신하는 것을 말한다.
+ 슬레이브는 마스터의 레플리카(replica)가 된다.
+ AP서버 - 로드밸런서 - MySQL 슬레이브 - MySQL 마스터
+ 참조쿼리는 슬레이브로, 갱신쿼리는 마스터로!
    + 갱신쿼리를 슬레이브로 던지면 마스터와 슬레이브 간 차이로 인해 레플리케이션을 중지한다.
    + O/R 매퍼 내에서 제어하는 방식도 있다.
##### ※ 로드밸런서 대신 Proxy를 사용하는 경우
### 갱신이 많은 경우의 해결방법
1. 테이블 분할, 디스크 분할
2. key-value 스토어(key-value쌍 스토리지, KVS)
    + 오버헤드가 적고 압도적으로 빠르며 확장하기 쉽다.
+ 참조쿼리가 많은 경우 슬레이브를 확장하면 되지만, 갱신쿼리가 많은 경우 마스터를 확장할 수 없다.
+ 웹 애플리케이션은 90% 이상이 참조쿼리

---
## 강의 13. MySQL의 스케일아웃과 파티셔닝
##### 스케일아웃은 메모리 증설을 하거나 파티셔닝을 이용
### 파티셔닝
+ 강의 10 참조
+ join 쿼리 대상이 아닌 테이블만을 서버 분할
+ join 대신 eid 값을 구해서 join 결과값을 원하는 다른 테이블에 where... in...을 이용해서 값을 구할 수 있다.
    + 라이브러리 오버헤드 자체가 아까운 경우 SQL을 직접 쓸 수도 있다.
+ 장점: 부하 감소, 국소성 증가, 캐시효과 증가
+ 단점:
    1. 운용이 어려워져 복구시간이 더 필요해질 수 있다.
    2. 대수 증가로 고장율이 증가한다.
    3. 경제적 비용을 고려해야 한다.
+ 대수 증가시 4대가 1세트인 이유
    + 마스터 고장시 슬레이브로 대체 가능
    + 슬레이브 고장시 정상적으로 서비스를 운영하는 서버를 놔두고 다른 1대의 서버를 중지시키고 대체 서버에 복사할 수 있다.
        + 즉, 고장서버, 서비스 운영서버, 복사용 서버와 마스터 서버 총 4대가 1세트가 된다.
+ 파티셔닝은 마지막 카드!

---
## 강의 14. 용도특화형 인덱스
### 대규모 데이터를 다뤄야 하는 경우
+ 전문 검색, 유사문서계열 탐색, 데이터마이닝
+ RDBMS 자체로는 한계가 있는 경우
    1. RDBMS 배치처리로 데이터를 추출해 인덱스 서버를 만들고, 
    2. 웹에서 RPC(remote procedure call) 등으로 액세스하는 방식 = 웹 API (JSON + HTTP)
+ 특정 목적으로 튜닝한 데이터 구조를 이용하면 RDBMS를 활용한 전부 순회방식보다 빠르게 검색할 수 있다.
### 하테나 사례
1. 하테나 키워드 링크
    + 정규표현에는 오토마톤 중에서 NFA(비결정성 유한 오토마톤) 특성으로 인해 OR로 매칭하면 계산량이 방대해져 속도가 느려진다.  
    + Trie 기반의 정규표현과 Common Prefix Search 조합을 이용한 자연언어 처리가 유용
2. 하테나 북마크 텍스트 분류
    + 출현 확률과 빈도만을 기록한 서버를 두고 Complement Naive Bayes 알고리즘을 사용
3. 전문 검색엔진
    + 검색 인덱스를 사용한 스코어링 처리

    

