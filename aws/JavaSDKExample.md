```java

package com.hist.macle.controller;

import java.util.ArrayList;
import java.util.Comparator;

import com.hist.macle.common.SecureUtil;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import com.hist.macle.common.ApiResponseMessage;
import com.hist.macle.common.MessageCode;
import com.hist.macle.common.UserSession;
import com.hist.macle.service.AwsChartService;
import com.hist.macle.service.AwsResourceService;
import com.hist.macle.service.CompanyService;
import com.hist.macle.vo.AwsAcoounttVo;
import com.hist.macle.vo.AwsChartVo;
import com.hist.macle.vo.AwsResourceInfoVo;
import com.hist.macle.vo.AwsResourceVo;
import com.hist.macle.vo.BillHistVo;
import com.mysql.cj.x.protobuf.MysqlxCrud.Collection;
import com.ulisesbocchio.jasyptspringboot.util.Collections;

import lombok.extern.slf4j.Slf4j;
import com.amazonaws.services.securitytoken.model.AssumeRoleRequest;

import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.auth.BasicSessionCredentials;
import com.amazonaws.auth.profile.ProfileCredentialsProvider;
import com.amazonaws.services.ec2.AmazonEC2;
import com.amazonaws.services.ec2.AmazonEC2Client;
import com.amazonaws.services.ec2.model.DescribeInstancesRequest;
import com.amazonaws.services.ec2.model.DescribeInstancesResult;
import com.amazonaws.services.ec2.model.Instance;
import com.amazonaws.services.ec2.model.Reservation;
import com.amazonaws.services.ec2.model.Tag;
import com.amazonaws.services.rds.AmazonRDS;
import com.amazonaws.services.rds.AmazonRDSClient;
import com.amazonaws.services.rds.model.DBInstance;
import com.amazonaws.services.rds.model.DescribeDBInstancesRequest;
import com.amazonaws.services.rds.model.DescribeDBInstancesResult;
import com.amazonaws.services.securitytoken.AWSSecurityTokenService;
import com.amazonaws.services.securitytoken.AWSSecurityTokenServiceClientBuilder;
import com.amazonaws.services.securitytoken.model.AssumeRoleResult;
import com.amazonaws.services.securitytoken.model.Credentials;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.dynamodbv2.document.DynamoDB;
import com.amazonaws.services.dynamodbv2.document.Table;
import com.amazonaws.services.dynamodbv2.document.TableCollection;
import com.amazonaws.services.dynamodbv2.model.ListTablesResult;

@RestController
@Slf4j
public class AwsResourceController {
	
	
	@Autowired(required = true)
	private AwsChartService awsChartService;
	
	@Autowired(required = true)
	private AwsResourceService awsResourceService;
	
	@Autowired(required = true)
	private CompanyService companyService;


	@RequestMapping(value = "/awsresource", method = RequestMethod.GET)
	public ResponseEntity<ApiResponseMessage> getAwsResource(HttpServletRequest httpServletRequest, String acct, String com, String all) throws Exception {
		long startDt = System.currentTimeMillis();

		UserSession userSession = new UserSession();
		ApiResponseMessage message = null;
		
		// 공백일때는 전체 배열을 보내야한다. 선택된것이 있으면 해당 Acct 넘겨주는 부분
		
		List<Map<String,String>> SearchAccountList = companyService.selectSearchAccountList();	
//		System.out.println("가공 전" + acct);
		String[] acctList = acct.split(",");
		
//		System.out.println("com.length() " + com.length());
//		System.out.println("com " + com);
//		System.out.println("SearchAccountList.size() " + SearchAccountList.size());
//		System.out.println("acctList.length " + acctList.length);
		
		
		// 회사 선택 후 계정 값이 전체 계정일 때 해당 회사 넣어주는 부분
		if((com.length() !=0 && !com.equals("전체 회사")) && (SearchAccountList.size()-2 ==acctList.length)) {
			acct ="";
			StringBuffer sb = new StringBuffer();			
			for(int i=0; i<SearchAccountList.size(); i++) {
				String[] customer_cd = SearchAccountList.get(i).values().toString().split(" ");		
				if(customer_cd[1].equals(com +"]")) {
					StringBuffer temp = new StringBuffer(customer_cd[0]);
				 	temp = temp.deleteCharAt(0);
				 	sb.append(temp);
				}						
			}
			
			sb = sb.deleteCharAt(sb.length()-1);
			acct = sb.toString();
			
			acctList = acct.split(",");
//			System.out.println("가공 후" + acct);
			
		}
		
	
		try {
			HttpSession session = userSession.getSession(httpServletRequest); 				
			if(session != null) {
				
				// code
				
				List<AwsChartVo> ec2_chart = new ArrayList<AwsChartVo>();
				List<AwsChartVo> rds_chart = new ArrayList<AwsChartVo>();
				List<AwsChartVo> dynamodb_chart = new ArrayList<AwsChartVo>();
				
				
				List<AwsChartVo> datatransfer_chart = new ArrayList<AwsChartVo>();
				int data_transfer_total = 0;
				
				//ec2
				
				int ec2_running = 0;
				int ec2_stopped = 0;
				List<String> ec2_detail = new ArrayList<String>();
				
				//rds
		        
		        int rds_running = 0;
				int rds_stopped = 0;
				List<String> rds_detail = new ArrayList<String>();
				
		        //dynamodb
		        
		        int dynamodb_num = 0;
		        List<String> dynamodb_detail = new ArrayList<String>();
		        
		        
		        List<AwsAcoounttVo> each_account_resource = new ArrayList<AwsAcoounttVo>();
				long step1 = System.currentTimeMillis();
				System.out.println("########################## STEP1 TIME : " + (step1 - startDt) + "ms");
					
				ec2_chart = awsChartService.getEc2Chart(acctList);
				rds_chart = awsChartService.getRdsChart(acctList);
				dynamodb_chart = awsChartService.getDynamodbChart(acctList);
				datatransfer_chart = awsChartService.getDatatransferChart(acctList);
				
				//datatransfer_chart 비교 함수
				
				datatransfer_chart.sort(new Comparator<AwsChartVo>() {

					@Override
					public int compare(AwsChartVo o1, AwsChartVo o2) {
						// TODO Auto-generated method stub
						
						int a =0;
						int b =0;
						
						for(int temp : o1.getList()) {
							a += temp;
						}
						
						for(int temp : o2.getList()) {
							b += temp;
						}
						
						if(a>b) {
							return -1;
						}else if(a<b) {
							return 1;
						}else{
							return 0;
						}
						
					}
					
				});
				
				if(datatransfer_chart.size() >5) {
					int deleteNum = datatransfer_chart.size()-5;				
					for (int i = 0; i < deleteNum; i++) {
						datatransfer_chart.remove(datatransfer_chart.size()-1);
					}										
				}
				
				// type 사용량 객체만듬, 크기 5개 안되면 그냥 넘어가고, 크기 순대로 정렬, 해당 타입이 아니면 제외
				
				for(AwsChartVo temp : datatransfer_chart) {			
					for(int index=0; index<31; index++) {
						data_transfer_total += temp.getList().get(index);				
					}
				}
					
				long step2 = System.currentTimeMillis();
				System.out.println("########################## STEP2 TIME : " + (step2 - startDt) + "ms");
				if(acctList != null && acctList.length != 0) {
					AwsResourceVo[] awsList = awsResourceService.getAwsResource(acctList);
					
					for(AwsResourceVo aws : awsList) {
						// 2021.04.23 아래 계정들 AWS API 오류로 조회 skip
						
						if(aws.getAccessKey().length() <=0) {
							//정상적인 accesskey, secrertkey가 없기에 예외처리
							continue;
						}
						
						
//						BasicAWSCredentials awsCreds = new BasicAWSCredentials(aws.getAccessKey(), SecureUtil.decodeAES(aws.getSecretKey()));
						
						AWSStaticCredentialsProvider acp = null;
						//BasicAWSCredentials awsCreds = new BasicAWSCredentials(aws.getAccessKey(), SecureUtil.decodeAES(aws.getSecretKey()));
						BasicAWSCredentials awsCreds = null;
						
						
//						if(aws.getAccount().equals("xxxxxxxxxxx") || aws.getAccount().equals("ccccc")) {
							String roleSessionName = "anyapiname";
/*							BasicAWSCredentials bac = new BasicAWSCredentials("acccesskey~~~", "secretkey~~~");


				            AWSSecurityTokenService stsClient = AWSSecurityTokenServiceClientBuilder.standard()
                                    .withCredentials(new AWSStaticCredentialsProvider(bac))
                                    .withRegion("ap-northeast-2")
                                    .build();*/
							
							
							AWSSecurityTokenService stsClient = AWSSecurityTokenServiceClientBuilder
									.standard()
//									.withEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration("sts-endpoint.amazonaws.com", "signing-region"))
									.withRegion("ap-northeast-2")
									.build();

							AssumeRoleRequest roleRequest = new AssumeRoleRequest()
							                                    .withRoleArn("arn:aws:iam::"+ aws.getAccountID() +":role/anyapiname")
							                                    .withRoleSessionName(roleSessionName);
							AssumeRoleResult roleResponse = stsClient.assumeRole(roleRequest);
							Credentials sessionCredentials = roleResponse.getCredentials();
				            BasicSessionCredentials awsCredentials = new BasicSessionCredentials(
				                    sessionCredentials.getAccessKeyId(),
				                    sessionCredentials.getSecretAccessKey(),
				                    sessionCredentials.getSessionToken());
				            acp = new AWSStaticCredentialsProvider(awsCredentials);
//						}else {
//							awsCreds = new BasicAWSCredentials(aws.getAccessKey(), SecureUtil.decodeAES(aws.getSecretKey()));
//							acp = new AWSStaticCredentialsProvider(awsCreds);
//						}
						

						int each_ec2 = 0;
						int each_rds = 0;
						int each_dynamodb = 0;
						
				        final AmazonEC2 ec2 = AmazonEC2Client.builder()
				        	    .withRegion("ap-northeast-2")
				        	    .withCredentials(acp)
				        	    .build();
				        
				        boolean done = false;
	
				        DescribeInstancesRequest request = new DescribeInstancesRequest();
				        
				        while(!done) {
				            DescribeInstancesResult response = ec2.describeInstances(request);
				           
				            for(Reservation reservation : response.getReservations()) {
	
				        for(Instance instance : reservation.getInstances()) {	     
				        	
				        		 if(instance.getState().toString().contains("running")) {
				        			 ec2_running++; 
				        			 each_ec2++;
				        		 }else if(instance.getState().toString().contains("stopped")) {
				        			 ec2_stopped++;
				        			 each_ec2++;
				        		 }
				        		 
				        		 String hostnmae = "-";
				        		 
				        		 for(Tag temp : instance.getTags()) {
				        			 if(temp.getKey().equals("Rname")) {
				        				 hostnmae = temp.getValue();
				        			 }
				        			 
				        		 }
				        		 
				        		 ec2_detail.add(instance.getState().getName());
				        		 ec2_detail.add(hostnmae);
				        		 ec2_detail.add(instance.getInstanceId().toString());
				        		 ec2_detail.add(instance.getInstanceType().toString());
				        		 ec2_detail.add(instance.getPlacement().getAvailabilityZone());
				        		 
				                }    
				            }
	
				            request.setNextToken(response.getNextToken());
	
				            if(response.getNextToken() == null) {
				                done = true;
				            }
				        }
						
				        final AmazonRDS rds = AmazonRDSClient.builder()
				    			.withRegion("ap-northeast-2")
				    			.withCredentials(acp)
				        	    .build();
				        
				        DescribeDBInstancesRequest request_rds = new DescribeDBInstancesRequest();      
				        DescribeDBInstancesResult response_rds = rds.describeDBInstances(request_rds);        	
				        List<DBInstance> instance = response_rds.getDBInstances();
				        
				        for(DBInstance temp : instance) {
				        	
					        	if(temp.getDBInstanceStatus().toString().contains("available")) {
					        		rds_running++; 
					        		each_rds++;
					        	}else if(temp.getDBInstanceStatus().toString().contains("stopped")) {
					        		rds_stopped++;
					        		each_rds++;
					   		 	}      	
					        	
					        	rds_detail.add(temp.getDBInstanceStatus());
					        	rds_detail.add(temp.getDBName());
					        	rds_detail.add(temp.getDBInstanceIdentifier());
					        	rds_detail.add(temp.getEngine());
					        	
				        }
				        
				        final AmazonDynamoDB ddb = AmazonDynamoDBClient.builder()
				        		.withRegion("ap-northeast-2")
				    			.withCredentials(acp)
				        	    .build();
				        
				        final DynamoDB dynamoDB = new DynamoDB(ddb);
				        
				        TableCollection<ListTablesResult> tables = dynamoDB.listTables();
				        Iterator<Table> iterator = tables.iterator();
	
				        while (iterator.hasNext()) {
				        	dynamodb_num++;
				        	each_dynamodb++;
				            Table table = iterator.next();
				        	dynamodb_detail.add(table.getTableName());
				        }
				        
				        
				        // 각 계정마다 account, ec2, rds, dynamodb 수 넣어주기
				        
				        each_account_resource.add(new AwsAcoounttVo(aws.getAccount(),each_ec2,each_rds,each_dynamodb));
				        
					}
				}
				long step3 = System.currentTimeMillis();
				System.out.println("########################## STEP3 TIME : " + (step3 - startDt) + "ms");
				
			
				AwsResourceInfoVo result = new AwsResourceInfoVo(ec2_running,ec2_stopped,rds_running,rds_stopped,dynamodb_num,
						ec2_detail,rds_detail,dynamodb_detail,ec2_chart,rds_chart,dynamodb_chart,datatransfer_chart,data_transfer_total,each_account_resource);

				if (result == null) {
					message = new ApiResponseMessage(HttpStatus.OK, "", "", "");
				} else {
					message = new ApiResponseMessage(HttpStatus.OK, result, "", "");
				}
							
				// code
				long endDt = System.currentTimeMillis();
				System.out.println("########################## ENDDT TIME : " + (endDt - startDt) + "ms");
			}else {					
				message = new ApiResponseMessage(HttpStatus.OK, "" , MessageCode.SESSION_IS_NULL, MessageCode.SESSION_IS_NULL_MSG);
			}
	
	}catch(Exception e) {
		message = new ApiResponseMessage(HttpStatus.OK,	"" , MessageCode.ERROR, MessageCode.ERROR_MSG);
		e.printStackTrace();
	}
		
		return new ResponseEntity<ApiResponseMessage>(message, message.getStatus());
	}


	// 중복되는 Datatransfer 합쳐주는 함수
	private void check(AwsChartVo temp ,List<AwsChartVo> chart) {
		// TODO Auto-generated method stub		
		for(int i=0; i<chart.size(); i++) {			
			//만약 같은 타입이 있다면
			if(temp.getType().equals(chart.get(i).getType())) {
				// i 인덱스에 temp list 값을 넣어준다.				
				for(int j=0; j<chart.get(i).getList().size(); j++) {
					chart.get(i).getList().set(j, chart.get(i).getList().get(j)+temp.getList().get(j));
				}				
				return;					
			}			
		}
		
		chart.add(temp);		
		
	}
}



```
