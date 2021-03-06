# 대규모 서비스를 지행하는 기술

# 대규모 데이터 처리 입문

## 대규모의 기준은?

- 하테나 북마크의 데이터 규모를 예로 들었다

| 레코드 수 | 데이터 크기 |
| --- | --- |
| entry 테이블 : 1,520만 엔트리 | 엔트리 : 3GB |
| bookmark 테이블 : 4,500만 북마크 | 북마크 : 5.5GB |
| tag 테이블 : 5,000만 태그 | 태그 : 4.8GB |
|  | HTML : 200GB 이상 |

Google이나 Yahoo와 비교햇을때는 하테나는 대규모~중규모 수준이다.

- entry 테이블을 인덱스없이 조회시 약 200초가 걸린다.

```sql
select url from entry use index(hoge) where eid = 9615899
```

## 대규모 데이터는 어떤 점이 어려운가?

- 메모리 내에서 계산이 불가할 경우, 디스크에 있는 데이터를 검색하게 된다. 이때 디스크는 I/O의 시간이 느리기때문에 조회시간이 느려질 수 밖에없는것이다.
    - “이를 어떻게 대처할 것인가?” 라는 키워드가  중요한것이다.
    - 메모리는 디스크보다 약 10의5승 ~ 10의6승배 (10만~100만배) 정도가 차이난다고한다.
    - 느린 이유는? → 디스크는 메모리와는 달리 회전과 같은 물리적인 동작을 수반하고 있기때문에
    - OS에서는 이를 어느정도 커버하는 작용 (데이터를 한꺼번에 읽어서 디스크회전수를 줄인다) 을 하고있지만, 메모리와의 속도차를 피하진 못한다.
- 전송속도 또한 차이가 난다.
    - 메모리는 7.5GB/초 의 빠른 속도가 나오지만, 디스크는 58MB/초 라는 비교적 느린 속도이기에 전송속도에서도 디스크는 느려지게 된다.

## 규모조정, 확장성

- 스케일업(scale-up) : 고가의 빠른 하드웨어를 사서 성능을 높인다.
- 스케일아웃(scale-out) : 저가이면서 일반적인 성능의 하드웨어를 많이 나열해서 시스템 전체 성능을 올린다.
    - 웹 서비스에 적합한 형태이고 비용이 저렴하는 점과 시스템 구성에 유연성이 있다는 점으로 이 전략을 주류로 사용한다.

## 웹 애플리케이션 부하

- 새로운 서버를 추가하고자 한다면 원래 있던 서버와 완전히 동일한 구성을 갖는 서버를 추가하면된다.
    - 그 이유는 웹 애플리케이션의 부하 관계를 보면 알 수 있다.
    1. 요청이 들어온다.
    2. AP서버에서 요청을 관리하여 DB로 전달
    3. DB에서 I/O 작업이 이루어짐
    4. 되돌아온 콘텐츠를 변경하여 클라이언트로 응답
    - 위 과정에서 AP서버에는 I/O 부하가 걸리지않고 CPU 부하만 걸리기때문에 분산이 간단하다
        - (데이터를 분산해서 갖고 있는 것이 아니므로 동일한 호스트가 동일하게 작업을 처리하기만 하면 분산할 수 있다)
- 하지만 I/O부하에는 문제가 있다.
    - 쓰기작업이 발생했을 경우, 데이터의 동기화가 문제가 된다.
- 서비스가 무거우면 서버를 늘리면 되는일 아닌가?
    - 서버를 늘리는것으로 해결이 된다면 그렇게 처리해도된다.
    - 하지만, 그것만으로 문제해결이 되지않기 때문에 더욱 어려운것이라고 한다.

## 대규모 데이터를 다루는 요령

1. 어떻게 하면 메모리에서 처리를 마칠 수 있을까?
    - 디스크 seek 횟수 최소화
    - 국소성을 활용한 분산 실현
2. 데이터량 증가에 강한 알고리즘을 사용하는 것
    - ex) 선형검색 → 이분검색
3. 데이터 압축이나 검색기술과 같은 테크닉 활용

## OS 페이지 캐시

- OS는 캐시 구조를 갖추고 있어서 메모리를 사용하여 디스크 액세스를 줄일수있다.
- 페이지 캐시란?
    - 커널이 한 번 할당된 메모리를 해제하지 않고 남겨두는 것
    - 남겨두었던 메모리를 사용할 수 있기 때문에 디스크를 또다시 읽으러 갈 필요가 없게된다.
- 페이지 캐시 과정
    1. 디스크의 내용을 메모리에 읽는다.
    2. 작성된 페이지는 파기되지 않고 남는다.
    3. 예외의 경우를 제외하고 모든 I/O에 투과적으로 작용한다.
- VFS(Virtual File System) : 가상파일 시스템
    - 파일시스템 구현의 추상화와 성능에 관련된 페이지 캐시 부분을 담당하는 추상화 레이어