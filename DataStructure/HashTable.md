# Hashing
item을 key-indexed table에 저장하는 방식

hash function -> key로부터 array index를 계산하는 방법

hash function의 목표는 주어진 key를 table의 index에 균등하게 분배하는것

서로 다른 key가 같은 곳에 mapping되는 것 (동일한 hashcode값을 갖는것) 을 collision이 발생했다고 한다.

collision을 처리할 수 있는 방법이 있지만 무엇보다도 애초에 발생하지 않게 하는 것이 제일 좋음

hash function이 h라고 할때 h(k)를 계산하는 과정에서는 2가지의 일이 일어난다
1. hash code 생성 -> key k 를 integer값으로 변환함
2. compression function -> integer값으로 변환한 것을 [0, N-1]에 들어오도록 줄여준다.

1번을 진행하는 방식은 매우 다양함
자바 라이브러리엔 타입별로 다양한 방식으로 hashCode값을 만들어낸다.

2번도 여러 방법이 있음 단순히 mod N을 해서 얻는 방법 (N이 소수이면 좀 더 잘 퍼지게 된다),
MAD(multiply add and divide) 방법 등이 있음


Collision을 해결하는 방법
1. Separate chaining
배열의 각 위치에 리스트를 두고 추가하는 방식으로 구현 -> 동일한 부분에 mapping된다면 리스트의 형태로 이어진다. (혹은 리스트가 아닌 map을 쓸수도 있음?)

좋은 hash function을 사용한다면 잘 분배되어 (uniform 하게 분배해준다면) n개의 원소가 들어간다면 각 index의 bucket에는 n/N개만큼 들어갈것으로 예상할 수 있음

이러한 n/N의 값을 load factor라고 함 이 값은 1이하로 유지되는 것이 좋음(1에 가깝게)

separate chaining의 단점은 추가적인 자료구조를 사용함으로써 space를 더 사용한다는 점에 있음
만약 space가 굉장히 중요한 것이라면 주어진 bucket에 항상 최대 하나의 원소만을 저장하는 방식을 사용해야함

이런 방식에는 여러개가 있지만 



java의 모든 클래스는 hashCode함수를 상속받게 되는데 이는 32bit int를 반환함
만약 x.equals(y) 가 true라면 x.hashCode() == y.hashCode()여야 한다.
그렇지만 false라고 항상 x.hashCode() != y.hashCode()이라는 것은 아님 (collision은 가능함)



