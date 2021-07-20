# 엑셀 파싱

필요 라이브러리

![image](https://user-images.githubusercontent.com/38831314/126248922-2c2498c7-b99f-4b2f-947b-608cf538a5f5.png)


추가 방법 : Add External JARs... 에다가 업로드

![image](https://user-images.githubusercontent.com/38831314/126248972-0f9cc61f-0717-4f03-98b8-5c32c4657307.png)



## 코드


```java


import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;

import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.poifs.filesystem.POIFSFileSystem;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public class Test {

	static HashMap<String, String> ori_temp = new HashMap<String, String>();
	static HashMap<String, String> ori = new HashMap<String, String>();
	static HashMap<String, String> after = new HashMap<String, String>();

	static ArrayList<String> ori_result = new ArrayList<String>();
	static ArrayList<String> after_result = new ArrayList<String>();

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {

			// 이전 파일
			File f = new File("d://Excel/temp.xlsx");
			FileInputStream fis = new FileInputStream(f);
			XSSFWorkbook work = new XSSFWorkbook(new FileInputStream(f));
			int sheetNum = work.getNumberOfSheets();

//			System.out.println("시트수 : " + sheetNum);
//			System.out.println("0번째 시트 이름" + work.getSheetName(0));

			XSSFSheet sheet = work.getSheet(work.getSheetName(0));
			int rows = sheet.getPhysicalNumberOfRows();
			int cells = sheet.getRow(0).getPhysicalNumberOfCells();
			System.out.println("해당 시트 행의 수 " + rows);

			for (int i = 0; i < rows; i++) {
				XSSFRow row = sheet.getRow(i);
				int nowcells = row.getPhysicalNumberOfCells();
				String volumeId = "";
				StringBuffer sb = new StringBuffer();
				boolean check = true;

				for (int j = 0; j < cells; j++) {

					XSSFCell cell = row.getCell(j, row.CREATE_NULL_AS_BLANK);
					String data = cell.getStringCellValue();

					if (data.equals("ok") || data.equals("warning")) {
						check = false;
						break;
					}

					if (data.length() == 0) {
						data = "-";
					}

					if (data.contains("vol-")) {
						volumeId = data;
					}

					if (j != cells - 1) {
						sb.append(data + ",");
					} else {
						sb.append(data);
					}

				}

				if (check) {
//					String temp = sb.toString();
					ori_temp.put(volumeId, sb.toString());
//					System.out.println(sb.toString());			
				}

			}

			System.out.println("가공 전 해쉬맵 사이즈 " + ori_temp.size());

			// 비교 페이지
			f = new File("d://Excel/temp.xlsx");
			fis = new FileInputStream(f);
			work = new XSSFWorkbook(new FileInputStream(f));
			sheetNum = work.getNumberOfSheets();

//			System.out.println("시트수 : " + sheetNum);
//			System.out.println("0번째 시트 이름" + work.getSheetName(0));

			sheet = work.getSheet(work.getSheetName(0));
			rows = sheet.getPhysicalNumberOfRows();
			cells = sheet.getRow(0).getPhysicalNumberOfCells();
			System.out.println("해당 시트 행의 수 " + rows);

			for (int i = 0; i < rows; i++) {
				XSSFRow row = sheet.getRow(i);
				int nowcells = row.getPhysicalNumberOfCells();
				String volumeId = "";
				StringBuffer sb = new StringBuffer();
				boolean check = true;
				for (int j = 0; j < cells; j++) {

					XSSFCell cell = row.getCell(j, row.CREATE_NULL_AS_BLANK);
					String data = cell.getStringCellValue();

					if (data.equals("ok") || data.equals("warning")) {
						check = false;
						break;
					}

					if (data.length() == 0) {
						data = "-";
					}

					if (data.contains("vol-")) {
						volumeId = data;
					}

					if (j != cells - 1) {
						sb.append(data + ",");
					} else {
						sb.append(data);
					}

				}

				if (check) {

					if (!ori_temp.containsKey(volumeId)) {
						// 기존에 없는데 조건문 통과하면
						after.put(volumeId, sb.toString());
					} else {
						// 기존에 존재해야 ori에 넣어야함
						ori.put(volumeId, sb.toString());
					}
				}

			}

			System.out.println("기존 해쉬맵 사이즈" + ori.size());
			System.out.println("신규 해쉬맵 사이즈" + after.size());

			int idx = 0;
			for (String i : ori.keySet()) {

				if (idx == 0) {
					ori_result.add("구분," + ori.get(i));
				} else {
					ori_result.add("기존," + ori.get(i));
				}
				idx++;

//			    System.out.println(i + " " + ori.get(i));
			}

			for (String i : after.keySet()) {
				after_result.add("신규," + after.get(i));
//			    System.out.println(i + " " + after.get(i));
			}

			// 정렬
			Collections.sort(ori_result);
			Collections.sort(after_result);

			System.out.println(after_result.get(0));

			// 출력 부분
			// .xlsx 확장자 지원
			XSSFWorkbook xssfWb = null; // .xlsx
			XSSFSheet xssfSheet = null; // .xlsx
			XSSFRow xssfRow = null; // .xlsx
			XSSFCell xssfCell = null;// .xlsx

			int rowNo = 0;

			xssfWb = new XSSFWorkbook();
			xssfSheet = xssfWb.createSheet("Fault_tolerance(EBS Snapshot)"); // 워크시트 이름
			xssfSheet.addMergedRegion(new CellRangeAddress(rowNo, rowNo, 0, 0)); // 첫행,마지막행,첫열,마지막열

			System.out.println(cells + "##");

			for (int i = 1; i <= cells; i++) {
				xssfSheet.setColumnWidth(i, (xssfSheet.getColumnWidth(i)) + (short) 1024);
			}
			
			for (int i = cells+1; i <= cells+3; i++) {
				xssfSheet.setColumnWidth(i, (xssfSheet.getColumnWidth(i)) + (short) 2048);
			}

			for (int i = 0; i < ori_result.size(); i++) {
				String[] temp = ori_result.get(i).split(",");
				xssfRow = xssfSheet.createRow(rowNo++); // 행 객체 추가

				// 기존 데이터 명시
				for (int j = 0; j < temp.length; j++) {
					xssfCell = xssfRow.createCell((short) j); // 추가한 행에 셀 객체 추가
					xssfCell.setCellValue(temp[j]); // 데이터 입력
				}
				// 기타  열 추가
				String[] etc = {"조치결과","미 조치 시 사유","비고"};
				
				if(i==0) {
					for (int j = cells; j <=(cells+2); j++) {
						xssfCell = xssfRow.createCell((short) j+1); // 추가한 행에 셀 객체 추가
						xssfCell.setCellValue(etc[j-cells]); // 데이터 입력
					}
				}
			}
			
			

			for (int i = 0; i < after_result.size(); i++) {
				String[] temp = after_result.get(i).split(",");
				xssfRow = xssfSheet.createRow(rowNo++); // 행 객체 추가

				// 기존 데이터 명시
				for (int j = 0; j < temp.length; j++) {
					xssfCell = xssfRow.createCell((short) j); // 추가한 행에 셀 객체 추가
					xssfCell.setCellValue(temp[j]); // 데이터 입력
				}
			}

			xssfSheet = xssfWb.createSheet("시트 이름222"); // 워크시트 이름

			String localFile = "d://Excel//" + "테스트_엑셀" + ".xlsx";

			File file = new File(localFile);
			FileOutputStream fos = null;
			fos = new FileOutputStream(file);
			xssfWb.write(fos);

			if (xssfWb != null)
				xssfWb.close();
			if (fos != null)
				fos.close();

		} catch (Exception e) {
			// TODO: handle exception
			System.out.println("error is" + e);
		}

	}

}


``
