# 2주차
- - -
## 강의 10 - 국소성을 살리는 분산

### 국소성을 고려한 분산이란?
* 데이터에 대한 액세스 패턴을 고려해서 분산시키는 것을 국소성을 고려한 분산이라고 한다.
* '인기 엔트리', '자신의 북마크' 등 여러 액세스 패턴에 따라 접근하는 서버를 분산시킨다.

### 파티셔닝 (Partitioning)
* 파티셔닝은 DB 서버를 여러 대의 서버로 분할하는 방법이다.
* 가장 간단한 방법은 '테이블 단위 분할'이다.
  * 요청에 따라 어떤 DB 서버에서 데이터를 읽을지 애플리케이션에서 로직을 구현해야한다.
* '테이블 데이터 분할'은 테이블 하나를 여러 개의 작은 테이블로 분할하는 방법이다.

### 요청 패턴을 '섬'으로 분할
* 앞서 살펴봤던 DB 의 테이블이나 DB 내의 데이터 패턴에 따라 DB 서버를 선택하는 방법과는 다르게 '용도별로 시스템을 섬으로 나누는 방법'이다.

### 페이지 캐시를 고려한 운용의 기본 규칙
1. OS 기동 직후에는 서버를 투입시키지 않는다.
   1. OS가 기동된 직후에는 캐시가 적재되어 있지 않기 때문이다.
2. 성능평가나 부하시험 시, 최초의 캐시가 최적화되어 있지 않기 때문에 초깃값은 버린다.

- - -
## 강의 11 - 인덱스를 올바르게 운용하기

### OS 캐시 활용
* 초기의 스키마는 보통 그다지 신경쓰지 않고 설계하지만, 테이블 규모가 매우 크다면 스키마를 조금 변경하는 것만으로 기가바이트 단위로 데이터가 증가할 수 있다.
* 어느정도 규모가 있는 서비스가 되면 컬럼 변경, 스키마 변경에도 그에 상응하는 주의를 기울여야만 한다.
* 대량의 데이터를 저장하는 테이블은 레코드가 가능한 한 작아지도록 설계해야 한다.

### 인덱스의 중요성
* 인덱스는 탐색을 빠르게 하기 위한 트리 구조이다.
* MySQL의 인덱스는 기본적으로 B+Tree 구조이다.
* B+Tree 는 B Tree 에서 파생된 데이터 구조이며, B Tree 는 트리를 구성하는 각 노드가 여러 개의 자식을 가질 수 있는 '다분 트리'이다.
* B는 Balanced 의 약자로, 데이터의 삽입이나 삭제가 반복되어도 트리가 한 쪽으로 치우치지 않고 균형을 이룬다.
* 탐색할 때는 데이터 건수 n 에 대해서 log n 이 되므로 계산량은 O(log n) 이다.

### 이분트리와 B 트리 비교
* 이분트리는 부모 노드가 반드시 하나이고, 자식 노드는 2개로 정해져있다.
* B 트리는 개수를 조정할 수 있다. 즉, 노드의 크기를 4KB 등으로 정할 수 있다.
  * B 트리의 이러한 특징은 OS가 디스크에서 데이터를 읽는 블록 단위로 읽어내면 디스크 Seek 발생횟수를 최소화 할 수 있다.
  * 따라서 같은 노드 내의 데이터는 디스크 Seek 없이 탐색이 가능하다.
* B 트리의 파생인 B+트리는 각 노드 내에 자식 노드로의 포인터만 가지고 있고, 실제 데이터의 값은 리프노드에만 가지고 있는 구조이다.

### 인덱스의 작용
* 인덱스는 하나의 쿼리에서 하나의 인덱스만 사용할 수 있으므로 여러 컬럼에 대해서 인덱스를 태우고자 할 경우는 복합 인덱스를 사용해야한다.
* 쿼리가 어떤 인덱스를 사용하는지 확인하려면 explain 명령어를 이용한다.

- - -
** 강의 12 - MySQL의 분산

### MySQL 의 레플리케이션 기능
* MySQL 에는 레플리케이션 기능이 있다.
  * 레플리케이션은 슬레이브에서 마스터의 데이터를 복제하는 것이다.
* 조회성 쿼리들은 슬레이브 노드로 분산시켜서 조회 성능을 끌어올릴 수 있다.
* 갱신 쿼리는 마스터에서 처리한다.

### 마스터/슬레이브의 특징
* 참조계열 쿼리는 확장을 위해 서버를 늘리면 되지만, 서버를 단순히 늘린다기 보다는 메모리에 초점을 맞추는 것이 중요하다.
* 갱신계열 쿼리는 마스터에서 처리하지만, 마스터는 확장하기 어렵다. 
* 웹 애플리케이션 계열에서는 대략 90% 이상이 참조계열 쿼리다.

### 갱신/쓰기 계열 쿼리에 부하가 발생할 때는?
* 테이블을 분할하여 테이블 크기를 작게 줄여주는 방법이 있다.
  * 테이블 파일이 분산되면 동일 호스트 내에서 여러 디스크를 가지고 분산시킬 수 있고, 서로 다른 서버로 분산할 수도 있다.
  * 만약 서로 다른 서버로 분산한다면 보상 트랜잭션을 잘 고려해 주어야 한다.
* RDBMS 를 사용하지 않는 방법을 고려해본다.
  * key-value 스토어를 통해서 값을 저장하고 꺼낼 뿐이므로 RDB가 갖는 복잡한 통계처리나 범용적인 정렬처리가 필요하지 않다면 key-value 스토어는 오버헤드가 적고 압도적으로 빠르며, 확장하기도 쉽다.

- - -
## 강의 13 - MySQL의 스케일 아웃과 파티셔닝

### 파티셔닝
* 파티셔닝은 테이블 A와 테이블 B를 서로 다른 서버에 놓아서 분산하는 방법이다.
* 파티셔닝은 국소성을 활용해서 분산할 수 있으므로 캐시를 더욱 효율적으로 사용할 수 있다.
* 파티셔닝은 운영이 복잡해진다.
* 관리해야 하는 서버 수가 늘어나는 만큼 고가용성이 낮아진다.
* 다중화할 때는 마스터 + 슬레이브로 4대로 구성하여, 한 대에 장애가 발생하더라도 서비스가 무중단으로 운영할 수 있도록 고려한다.

- - -
## 강의 14 - 용도 특화형 인덱싱

### 인덱스와 시스템 구성
* 전문 검색, 유사문서계열 탐색, 데이터 마이닝 등 대규모 데이터를 다룰 때, RDBMS 로는 한계가 있다.
  * 이러한 경우 RDBMS 에서 데이터를 추출하여 별도의 인덱스 서버를 구축하여 인덱스 서버에 웹 애플리케이션에서 RPC 등으로 액세스하는 방법을 이용할 수 있다.
* RPC 는 Remote Procedure Call 을 의미한다.
* 인덱스를 위한 전용 서버와 AP 서버를 분리함으로써 더 효율적인 메모리 사용이 가능하다.

### 용도특화형 인덱싱
* RDBMS는 데이터를 정렬, 통계처리가 가능하고, JOIN 하는 등 범용적인 용도로 쓰인다.
* Elastic Search 와 같은 역인덱싱 구조의 데이터베이스는 전문 검색 등 대량의 문서 데이터를 검색하는데 특화되어 있다.

- - -
## 강의 15 - 이론과 실전 양쪽과의 싸움

### 대규모 웹 애플리케이션에 있어서 이론과 실전
* 한쪽으로 치우치지 않고 이론과 실전 사이에서 균형을 잘 맞춰 실행해가는 것이 중요하다.