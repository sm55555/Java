## DNS 등록 도우미

### A레코드 -> CNAME

### CNAME -> A 레코드 사용 가능

입력 방법

test.kkk.com [A레코드] [A레코드] aa
test.kkk.com [A레코드] [CNAME] ac
end

```java

import java.io.*;
import java.util.*;

public class Main {
	static StringBuffer sb = new StringBuffer();

	public static void main(String[] args) throws IOException {

		Scanner sc = new Scanner(System.in);

		System.out.println();
		while (true) {
			String s = sc.nextLine();
			
			
			if(s.equals("end")) {
				break;
			}

			String[] data = s.split(" ");
			String[] record = data[0].split("\\.");

			if (data[3].equals("aa")) {


				System.out.println(
						"dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " A " + data[1] + " /f");
				System.out.println(
						"dnscmd /RecordDelete " + reverse(data[1]) + ".in-addr.arpa. " + last(data[1]) + " PTR /f");
				System.out.println("dnscmd /recordadd " + record[1] + ".com " + record[0] + " A " + data[2]);
				System.out.println("dnscmd /recordadd " + reverse(data[2]) + ".in-addr.arpa " + last(data[2]) + " "
						+ "PTR " + data[0]);

				sb.append("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " A " + data[2] + " /f\n");
				sb.append("dnscmd /RecordDelete " + reverse(data[2]) + ".in-addr.arpa. " + last(data[2]) + " PTR /f\n");
				sb.append("dnscmd /recordadd " + record[1] + ".com " + record[0] + " A " + data[1] + "\n");
				sb.append("dnscmd /recordadd " + reverse(data[1]) + ".in-addr.arpa " + last(data[1]) + " " + "PTR "
						+ data[0] + "\n");
			} else if (data[3].equals("ac")) {

				System.out.println(
						"dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " A " + data[1] + " /f");
				System.out.println(
						"dnscmd /RecordDelete " + reverse(data[1]) + ".in-addr.arpa. " + last(data[1]) + " PTR /f");
				System.out.println("dnscmd /recordadd " + record[1] + ".com " + record[0] + " CNAME " + data[2]);

				sb.append(
						"dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " CNAME " + data[2] + " /f\n");
				sb.append("dnscmd /recordadd " + record[1] + ".com " + record[0] + " A " + data[1] + "\n");
				sb.append("dnscmd /recordadd " + reverse(data[1]) + ".in-addr.arpa " + last(data[1]) + " " + "PTR "
						+ data[0] + "\n");

			}

		}

		System.out.println();
		System.out.println("Fall Back\n");
		System.out.println(sb.toString());
	}

	private static String last(String string) {
		// TODO Auto-generated method stub
		String[] temp = string.split("\\.");

		StringBuffer sb = new StringBuffer();
		for (int i = 3; i == 3; i--) {
			sb.append(temp[i]);
		}

		return sb.toString();
	}

	private static String reverse(String string) {
		// TODO Auto-generated method stub

		String[] temp = string.split("\\.");

		StringBuffer sb = new StringBuffer();
		for (int i = 2; i >= 0; i--) {

			if (i != 0) {
				sb.append(temp[i] + ".");
			} else {
				sb.append(temp[i]);
			}
		}

		return sb.toString();
	}
}

```
