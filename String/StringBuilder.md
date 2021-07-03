### 삭제

```java
deleteCharAt(int);
```

### 추가

```java
sb.append();
```

### 예시

```java

import java.io.*;
import java.util.*;

public class Main {
	public static void main(String[] args) throws IOException {

		StringBuilder sb = new StringBuilder("aaa");

		sb.deleteCharAt(0);
		System.out.println(sb);
		sb.append("bb");
		System.out.println(sb);
	}
}

```
