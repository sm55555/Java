참고 URL : https://velog.io/@dnjscksdn98/Java-String-vs-StringBuilder-vs-StringBuffer

# 정리 

### String

단순히 문자열을 참조하거나 탐색 및 검색이 잦을 때 좋습니다.

String 클래스는 immutable(불변)하다는 특성이 있습니다. String 클래스의 문자열을 저장하는 char[] 을 보면 final 로 선언되어 있다는 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/38831314/117251212-4cf2eb00-ae7f-11eb-8b19-2ed7bb6e1f95.png)


### StringBuilder

반면 StringBuilder 클래스는 mutable(가변)합니다. 상속 받고 있는 AbstractStringBuilder 클래스의 내부를 보면 변경 가능하도록 선언되어 있습니다.

런타임 때, 반복적인 문자열 추가 연산이 많을 때 좋습니다.
단일 스레드 환경이라면 StringBuffer 보다 성능이 좋을 수 있습니다.



### StringBuffer

멀티 스레드 환경에서 반복적인 문자 추가 연산이 많을 때 좋습니다.
