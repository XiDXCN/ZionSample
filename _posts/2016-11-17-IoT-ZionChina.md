---
layout: post
title:  "ZionChina, Intelligent Diabetes Management with Azure IoT and Machine Learning"
author: "Xi Wang"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2016-11-17 20:13:28
categories: IoT
color: "blue"
#image: "{{ site.baseurl }}/images/ZionChina00_Logo.png" #should be ~350px tall
excerpt: Microsoft teamed up with ZionChina 2016 to help them envolve their solution with Azure IoT, Machine Learning and serveral key Azure technologies.
verticals: Healthcare
language: English
---



## ZionChina, Intelligent Diabetes Management with Azure IoT and Machine Learning ##

 
## Customer ##

Zion China Technology Co., Ltd. (Beijing) （www.zionchina.com）is a mobile internet healthcare company that believes that innovative technology can have a profound impact on the health and well-being of its users. It is focused on building products that improve the quality of life of patients with chronic diseases by helping them better manage their illnesses on a daily basis.
 
ZionChina created the E-Followup glucose metabolism dynamics management cloud platform. ZionChina proposes a new five-wheeled motor vehicle for diabetes management，including glucose metabolism, pharmacokinetics, dietary dynamics, exercise kinetics, and patient self-motivation. The management approach filled the gaps in the field of precision medical management in China, and pioneered the field of precision diabetes management precedent.


 
## Pain point ##

Zion China Technology Ltd. provide E-Followup Dynamic Management of Glucose Metabolism Solution E-Follow Up uses continuous glucose monitoring devices to collect patients' health data, including blood sugar, diet, exercise, medication, etc, and provide personized advices and prescription to help patients on medicine, diet and exercise.  

Their main technical and business pain points are:

1. Smart, fast and cost effective way to continuously feed data from devices to Cloud 
1. Optimize the analytic suite, moving from BI analysis to proactive predication. 
1. Persist data storage for future using with data security ensured
1. Get insight from data easier



1) Smart, fast and cost effective way to continuously feed data from devices to Cloud.

ZionChina provide services of 7-days glucose metabolism test report services and 3 months of glycated hemoglobin strengthen management services. Users wear dynamic glucose device to collect a blood glucose data in every 3 minutes, access to blood glucose changes for 7 days. At the same time collect the user's daily diet, exercise, medication, insulin and other information. ZionChina through the analysis of the data, to identify the impact of various factors for each user, to assess the current state of the user's sugar control and the risk of hidden high blood glucose, low blood glucose. At the same time, according to the user's diet preferences, to provide personalized life recommendations. 

During the wearing period, each user will generate more than 3000 blood glucose data, once every 3 minutes. High-frequency data transmission and large amounts of data storage has become a major technical problem. 

2)	Optimize the analytic suite, moving from BI analysis to proactive predication 

In the past, even with our data analytic product, each data analysis requires professional doctors to spend more than 4 hours. It takes a long time to find the correlation between the data, and it needs a lot of accumulation to find out the effective conclusion. 

3)	Persist data storage for future using with data security ensured

We need to ensure all the persist stored data, even all personal information has been removed, be stored in secure manner.


4)	Get insights from data easier

Users and doctors need to view data in many different scenarios, such as Web pages, mobile App, Pad, etc. We need a tool that can be adapted to multiple devices and is easy to develop and proficient in data analysis and presentation.


 
## Solution ##


ZionChina’s original technical solution was mostly based on traditional BI with data sourced from on premise and varies devices or Cloud storage. 

Microsoft DX China team worked with ZionChina to re-architect the solution:

- Adding IoT Hub to front-end to serve real time data transmission from device to Cloud
- Using Azure Machine Learning to generate proactive predication on glucose and patient’s data. 
- Using Azure SQL DB’s Transparent Data Encryption to secure data
- Using PowerBI and PowerBI embedded for easier insights visualization 

Key Microsoft technologies:
 
- Azure IoT Hub
- Stream Analytics
- Azure SQL Database
- Machine Learning
- Power BI

 

##Architecture##

This is the architecture of ZionChina Solution.
![](../images/ZionChina01_Arch.jpg)



##Device used & Code artifacts

Below are the key technical components with code artifacts

**IoT SDK Integration**

Using Microsoft Azure IoT Hub to real time transfer data to Cloud. In this project, the glucose device transfer data via bluetooth to mobile app on Andriod and iOS. Below are key code to integrate IoT SDK with the app.

- below code was used to register devices with IoThub

```python
def generate_sas_token(uri, key, policy_name='device', expiry=3600):
	"""
	generate authorization
	uri: 主机名
	key: Iothub中共享访问策略中的主密钥
	policy_name: 共享策略名称
	expiry: 超时时间
	"""
    ttl = time() + expiry
    sign_key = "%s\n%d" % (uri, int(ttl))
    signature = b64encode(HMAC(b64decode(key), sign_key, sha256).digest())

    return 'SharedAccessSignature ' + urlencode({
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl)),
        'skn': policy_name
    })


class DeviceIdentify:
	'''
	Iot device register
	'''
	def __init__(self, uri, deviceId, version, authorization):
		"""
		uri: 'https://'' + 主机名 + '/'
		deviceId: 表示用户的硬件设备的一个字符串
		version: '2016-02-03'
		authorization: sas_token
		"""
		self.uri = uri
		self.deviceId = deviceId
		self.version = version
		self.url = self.uri + self.deviceId + "?api-version=" + self.version
		self.values = {
			"deviceId": self.deviceId
		}
		self.authorization = authorization

	def identifyDevice(self):
		data = json.dumps(self.values)
		#context = ssl._create_unverified_context()#python 2.7之后, 需要验证ssl

		request = urllib2.Request(self.url, data, {"Authorization": self.authorization, "Content-Type": "application/json"})
		request.get_method = lambda: "PUT"

		try:
			response = urllib2.urlopen(request)###http error 409 conflict 重复注册的问题
			#response = urllib2.urlopen(request, context=context)###python 2.7之后, 这样使用
			result = response.read()
		except Exception as err:
			return {
				'status': 'unabled',
				'error': {
					'description': str(err),
					'code': 95000
				}
			}
		else:
			return json.loads(result)




```

- Below code was used to send messagr from Andriod to IoTHub

```java


public class SendMessage2Iothub{

    // 发送信息到iothub
	/**
	 * sendIOTMessage 向iothub发送消息
	 * @param connString 连接字符串,如：HostName=*******;DeviceId=*****;SharedAccessKey=*******"
	 * @param obj 发送对象：字符串、json等
	 * @return
	 * @throws Exception
	 */
	private void sendIOTMessage(String connString, Object obj) throws URISyntaxException, IOException {
        IotHubClientProtocol protocol = IotHubClientProtocol.AMQPS;
        DeviceClient client = new DeviceClient(connString, protocol);
        try {
            client.open();
        } catch(IOException e1) {
            System.out.println("Exception while opening IoTHub connection: " + e1.toString());
        } catch(Exception e2) {
            System.out.println("Exception while opening IoTHub connection: " + e2.toString());
        }
        try {
            String msgStr = obj.toString();
			Message msg = new Message(msgStr);
            msg.setProperty("messageCount", protocol.name());
            System.out.println(msgStr);
            EventCallback eventCallback = new EventCallback();
            client.sendEventAsync(msg, eventCallback, cGlucoseExchangeEntity.getPid());
        } catch (Exception e) {
        }

    }
	
}

```

- below code was used to send messagr from iOS to IoTHub

```
//
//  LotHubSingleton.m
//  diabetesDataCenter
//
//  Created by 将离。 on 2016/10/19.
//  Copyright © 2016年 zionchina. All rights reserved.
//

#import "LotHubSingleton.h"

#import <AFNetworking.h>

#define HOSTNAME @""
#define HOSTURL [NSString stringWithFormat:@"https://%@",HOSTNAME]

#define SendMessagePath @"/messages/events"
#define ReciveMessagePath @"/messages/devicebound"

#define DeiviceId @""
#define DevicePrimaryKey @""
#define DeviceSasToken @""

@interface LotHubSingleton()

@end

@implementation LotHubSingleton

// 发送信息到iothub
/**
 * sendSendMessageToLotHubRequestWithDicitionary 向iothub发送消息
 * @param messageDictionary 发送对象：字符串、json等
 * @return
 * @throws Exception
 */

- (void)sendSendMessageToLotHubRequestWithDicitionary:(NSDictionary *)messageDictionary
{
    AFSecurityPolicy * securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];
    securityPolicy.allowInvalidCertificates = YES;
    securityPolicy.validatesDomainName = NO;
    
    NSString *postUrl = [NSString stringWithFormat:@"%@/devices/%@%@?api-version=2016-02-03",HOSTURL,DeviceId,SendMessagePath];
    
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"text/html",@"application/json", @"text/json", @"text/javascript",@"application/javascript",nil];
    manager.securityPolicy = securityPolicy;
    manager.requestSerializer.timeoutInterval = 10.f;
    
    NSMutableURLRequest *mUrlRequest = [[AFJSONRequestSerializer serializer] requestWithMethod:@"POST" URLString:postUrl parameters:nil error:nil];
    mUrlRequest.HTTPBody = [NSJSONSerialization dataWithJSONObject:messageDictionary options:NSJSONWritingPrettyPrinted error:nil];
    [mUrlRequest setValue:DeviceSasToken forHTTPHeaderField:@"authorization"];
    
    [[manager dataTaskWithRequest:mUrlRequest completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
        if (!error) {
            NSInteger statusCode = [(NSHTTPURLResponse *)response statusCode];
            NSLog(@"statusCode: %ld responseObject: %@",statusCode, responseObject);
        } else {
            NSLog(@"Error: %@, %@, %@", error, response, responseObject);
        }
    }] resume];
    
}






@end

```





**Stream Analytics**

Microsoft Azure Stream Analytics was used to real time process ingested data on IoT hub and streamly put useful data for analyzing onto Azure SQL DB.

![](../images/ZionChina03_SA_input.png)

Figure: Azure stream analytics configure of input



![](../images/ZionChina04_SA_output.png)

Figure:Azure stream analytics configure of output

    


##Opportunities going forward

This section is optional, but if you have details on how the customer plans to proceed or what more they hope to accomplish, please include.

##Conclusion##



