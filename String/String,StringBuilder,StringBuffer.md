참고 URL : https://velog.io/@dnjscksdn98/Java-String-vs-StringBuilder-vs-StringBuffer

# 정리 

### String

단순히 문자열을 참조하거나 탐색 및 검색이 잦을 때 좋습니다.

String 클래스는 immutable(불변)하다는 특성이 있습니다. String 클래스의 문자열을 저장하는 char[] 을 보면 final 로 선언되어 있다는 것을 확인할 수 있습니다.


![image](https://user-images.githubusercontent.com/38831314/117251212-4cf2eb00-ae7f-11eb-8b19-2ed7bb6e1f95.png)

더하기 연산을 하여 붙일 시 새로운 객체가 생성되어 재할당됩니다.
반복적으로 문자열을 이어 붙이면 Heap 영역에 참조를 잃은 문자열 객체가 계속해서 쌓이게 됩니다. 
물론 나중에 GC에 의해 수거가 되지만, 메모리 관리 측면에서 이러한 코드는 결코 좋다고 할 수 없습니다. 
또한 계속해서 객체를 생성하기 때문에 연산 속도적인 측면에서도 뒤떨어집니다.

![image](https://user-images.githubusercontent.com/38831314/117251641-ea4e1f00-ae7f-11eb-90ae-37e4ff1c09b2.png)

### StringBuilder

반면 StringBuilder 클래스는 mutable(가변)합니다. 상속 받고 있는 AbstractStringBuilder 클래스의 내부를 보면 변경 가능하도록 선언되어 있습니다.

런타임 때, 반복적인 문자열 추가 연산이 많을 때 좋습니다.
단일 스레드 환경이라면 StringBuffer 보다 성능이 좋을 수 있습니다.

append() 메소드를 호출하면, char[] 배열의 길이를 늘리고 같은 객체에 문자열을 더합니다. 
아래의 코드를 보면 append() 호출 이후에도 StringBuilder 객체에 변함이 없음을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/38831314/117251933-43b64e00-ae80-11eb-82c6-058e00719be9.png)

### StringBuffer

멀티 스레드 환경에서 반복적인 문자 추가 연산이 많을 때 좋습니다.
