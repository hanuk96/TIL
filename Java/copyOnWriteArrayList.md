# CopyOnWriteArrayList
CopyOnWriteArrayList는 ArrayList와는 달리, List를 읽기 위해 어딘가에 전달할 때 원본이 아닌 복사본을 만들어서 전달한다.

내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어서 수행하도록 구현되어 있다.

내부의 배열은 절대 변경할 수 없으므로 순회할 때 락이 필요 없어서 속도면에서 매우 빠르다는 장점이 있다.



ConcurrentHashMap

스레드가 안전한 연산을 할 수 있게 해주는 해시맵

ConcurrentHashMap은 multi-threaded 환경에서 안정적으로 동작하고 HashTable과 synchronized map 보다 더 나은 성능을 가지고 있다. 

그 이유는, ConcurrentHashMap은 map의 일부에만 lock을 거는데 반해, 앞의 두 가지는 map 전체에 lock을 걸기 때문이다.
읽기가 쓰기보다 많을 때 가장 적합하다. 만약 쓰기가 더 많거나 쓰기가 읽기와 비슷하다면 ConcurrentHashMap의 성능은 synchronizedmap 또는 HashTable 만큼 떨어지게 된다. ConcurrentHashMap은 어플리케이션의 구동 후에 많은 요청을 처리하는 쓰레드에서 엑세스 하는 동안 초기화할 수 있기 때문에 cache 용도로 아주 적합하다. ConcurrentHashMap은 또한 HashTable의 좋은 대체안이기도 하며 가능한한 ConcurrentHashMap을 사용하라고 javadoc에 기술되어 있다. 다만 ConcurrentHashMap이 동기화 부분에는 HashTable 보다 조금 단점을 가지고 있다고 보면 되겠다.

