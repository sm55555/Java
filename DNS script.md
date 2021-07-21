## DNS 등록 도우미

### [사용 가능한 시나리오 및 명령어]

1. A record -> CNAME : [DNS 주소] [A레코드] [CNAME] ac
2. A record -> A record : [DNS 주소] [A레코드] [CNAME] aa
3. A record 추가 : [DNS 주소] [A레코드] - a
4. CNAME  추가 : [DNS 주소] [A레코드] - c
5. A record 삭제 :  [DNS 주소] [A레코드] - da
6. CNAME 삭제 : [DNS 주소] [A레코드] - dc
7. CNAME -> CNAME : [DNS 주소] [CNAME] [CNAME] cc
8. CNAME -> A record : [DNS 주소] [CNAME] [A레코드] ca

입력 방법

```cmd
test.kkk.com [A레코드] [A레코드] aa
test.kkk.com [A레코드] [CNAME] ac
end
```

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

			if (s.equals("end")) {
				break;
			}

			String[] data = s.split(" ");
			String[] record = data[0].split("\\.");

			// A Record -> A Record
			if (data[3].equals("aa")) {

				// ------------------------------------------------------

//				dnscmd /RecordDelete koreanair.com erpprdap1.koreanair.com. A 10.7.11.29 /f
//				dnscmd /RecordDelete 11.7.10.in-addr.arpa. 29 PTR  /f
//				dnscmd /recordadd koreanair.com erpprdap1 A 10.48.43.129
//				dnscmd /recordadd 43.48.10.in-addr.arpa 129 PTR erpprdap1.koreanair.com

//				dnscmd /RecordDelete koreanair.com erpprdap1.koreanair.com. A 10.48.43.129 /f
//				dnscmd /RecordDelete 43.38.10.in-addr.arpa. 129 PTR  /f
//				dnscmd /recordadd koreanair.com erpprdap1 A 10.7.11.29
//				dnscmd /recordadd 11.7.10.in-addr.arpa 29 PTR erpprdap1.koreanair.com

				// ------------------------------------------------------

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

				// A Record -> CNAME
			} else if (data[3].equals("ac")) {

//				erpprd.koreanair.com 10.7.11.33 awsdc-nlb-erp-prd-erpap-460c371a8b59cc82.elb.ap-northeast-2.amazonaws.com ac

				// ------------------------------------------------------

//				dnscmd /RecordDelete koreanair.com erpprdap1.koreanair.com. A 10.7.11.29 /f
//				dnscmd /RecordDelete 11.7.10.in-addr.arpa. 29 PTR  /f
//				dnscmd /recordadd koreanair.com erpprdap1 CNAME awsdc-nlb-erp-prd-erpap-460c371a8b59cc82.elb.ap-northeast-2.amazonaws.com

//				dnscmd /RecordDelete koreanair.com erpprdap1.koreanair.com. CNAME awsdc-nlb-erp-prd-erpap-460c371a8b59cc82.elb.ap-northeast-2.amazonaws.com /f
//				dnscmd /recordadd koreanair.com erpprdap1 A 10.7.11.29
//				dnscmd /recordadd 11.7.10.in-addr.arpa 29 PTR erpprdap1.koreanair.com

				// ------------------------------------------------------

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

				// Add A Record
			} else if (data[3].equals("a")) {
				System.out.println("dnscmd /recordadd " + record[1] + ".com " + record[0] + " A " + data[1]);
				System.out.println("dnscmd /recordadd " + reverse(data[1]) + ".in-addr.arpa " + last(data[1]) + " "
						+ "PTR " + data[0]);

				sb.append("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " A " + data[1] + " /f\n");
				sb.append("dnscmd /RecordDelete " + reverse(data[1]) + ".in-addr.arpa. " + last(data[1]) + " PTR /f\n");
				
				//ADD CNAME
			} else if (data[3].equals("c")) {
				
				System.out.println("dnscmd /recordadd " + record[1] + ".com " + record[0] + " CNAME " + data[1]);
				sb.append("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " CNAME " + data[1] + " /f\n");
				
				// Delete A Record
			} else if (data[3].equals("da")) {				

				System.out.println("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " A " + data[1] + " /f");
				System.out.println("dnscmd /RecordDelete " + reverse(data[1]) + ".in-addr.arpa. " + last(data[1]) + " PTR /f");
				
				sb.append("dnscmd /recordadd " + record[1] + ".com " + record[0] + " A " + data[1] +"\n");
				sb.append("dnscmd /recordadd " + reverse(data[1]) + ".in-addr.arpa " + last(data[1]) + " "
						+ "PTR " + data[0] +"\n");
				// Delete CNAME
			} else if (data[3].equals("dc")) {
				
				System.out.println("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " CNAME " + data[1] + " /f");
				sb.append("dnscmd /recordadd " + record[1] + ".com " + record[0] + " CNAME " + data[1] +"\n");
				// CNAME -> CNAME
			}else if (data[3].equals("cc")) {
				
				System.out.println("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " CNAME " + data[1] + " /f");				
				System.out.println("dnscmd /recordadd " + record[1] + ".com " + record[0] + " CNAME " + data[2]);
				
				sb.append("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " CNAME " + data[2] + " /f\n");				
				sb.append("dnscmd /recordadd " + record[1] + ".com " + record[0] + " CNAME " + data[1]+"\n");
				// CNAME -> A Record
			}else if (data[3].equals("ca")) {
				
				System.out.println("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " CNAME " + data[1] + " /f");				
				System.out.println("dnscmd /recordadd " + record[1] + ".com " + record[0] + " A " + data[2]);
				System.out.println("dnscmd /recordadd " + reverse(data[2]) + ".in-addr.arpa " + last(data[2]) + " " + "PTR "+ data[0]);
				
				
				sb.append("dnscmd /RecordDelete " + record[1] + ".com " + data[0] + "." + " A " + data[2] + " /f\n");
				sb.append("dnscmd /RecordDelete " + reverse(data[2]) + ".in-addr.arpa. " + last(data[2]) + " PTR /f\n");
				sb.append("dnscmd /recordadd " + record[1] + ".com " + record[0] + " CNAME " + data[1]+"\n");
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
