# 步骤
1. 于AccessKey管理开通RAM服务，创建用户。
2. 复制粘贴用户的accessKeyId和accessKeySecret到代码中。
3. 给予用户权限

# 普通上传方式
## 原生代码

```xml
<dependency>  
    <groupId>com.aliyun.oss</groupId>  
    <artifactId>aliyun-sdk-oss</artifactId>  
    <version>3.15.1</version>  
</dependency>
```

```java
// Endpoint以华东1（杭州）为例，其它Region请按实际情况填写（换掉hangzhou）。  
String endpoint = "https://oss-cn-hangzhou.aliyuncs.com";  
// 阿里云账号AccessKey拥有所有API的访问权限，风险很高。强烈建议您创建并使用RAM用户进行API访问或日常运维，请登录RAM控制台创建RAM用户。  
String accessKeyId = "yourAccessKeyId";  
String accessKeySecret = "yourAccessKeySecret";  
// 填写Bucket名称，例如examplebucket。  
String bucketName = "examplebucket";  
// 填写Object完整路径，完整路径中不能包含Bucket名称，例如exampledir/exampleobject.txt。  
String objectName = "exampledir/exampleobject.txt";  
// 填写本地文件的完整路径，例如D:\\localpath\\examplefile.txt。  
// 如果未指定本地路径，则默认从示例程序所属项目对应本地路径中上传文件流。  
String filePath= "D:\\localpath\\examplefile.txt";  
  
// 创建OSSClient实例。  
OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);  
  
try {  
    InputStream inputStream = new FileInputStream(filePath);  
    // 创建PutObjectRequest对象。  
    PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, objectName, inputStream);  
    // 设置该属性可以返回response。如果不设置，则返回的response为空。  
    putObjectRequest.setProcess("true");  
    // 创建PutObject请求。  
    PutObjectResult result = ossClient.putObject(putObjectRequest);  
    // 如果上传成功，则返回200。  
    System.out.println(result.getResponse().getStatusCode());  
} catch (OSSException oe) {  
    System.out.println("Caught an OSSException, which means your request made it to OSS, "  
            + "but was rejected with an error response for some reason.");  
    System.out.println("Error Message:" + oe.getErrorMessage());  
    System.out.println("Error Code:" + oe.getErrorCode());  
    System.out.println("Request ID:" + oe.getRequestId());  
    System.out.println("Host ID:" + oe.getHostId());  
} catch (ClientException ce) {  
    System.out.println("Caught an ClientException, which means the client encountered "  
            + "a serious internal problem while trying to communicate with OSS, "  
            + "such as not being able to access the network.");  
    System.out.println("Error Message:" + ce.getMessage());  
} finally {  
    if (ossClient != null) {  
        ossClient.shutdown();  
    }  
}
```

## 整合Alibaba

```xml
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alicloud-oss</artifactId>  
    <version>2.2.0.RELEASE</version>  
</dependency>
```

`bucket`不是内部配置项，是用来通过`@Value`获取的。
`endpoint`如果加上http头可能会导致后面拼接host的时候多余了个http。

```yml
spring:
  cloud:
    alicloud:
      accesskey: yourAccessKeyId
      secretkey: yourAccessKeySecret
      oss:
        endpoint: oss-cn-beijing.aliyuncs.com
        bucket: gulimall-ayana
```

```java
@Autowired  
OSS ossClient;  
  
@Test  
public void textUploadOSS() throws FileNotFoundException {  
  
    InputStream inputStream = new FileInputStream("D:\\AAAAA\\BGImage\\神子.jpg");  
    ossClient.putObject("gulimall-ayana", "testdir/神子3.jpg", inputStream);  
    ossClient.shutdown();  
}
```

# 服务端签名后直传

文档：[服务端签名后直传](https://help.aliyun.com/document_detail/31926.html)

后端签名方法：

```java
@Autowired  
OSS ossClient;  

@Value("${spring.cloud.alicloud.oss.bucket}")  
private String bucket;  

@Value("${spring.cloud.alicloud.oss.endpoint}")  
private String endpoint;  

@Value("${spring.cloud.alicloud.accesskey}")  
private String accessId;  

@RequestMapping("/oss/policy")  
public Map<String, String> policy() {  

	// 填写Host地址，格式为https://bucketname.endpoint。  
	String host = "https://" + bucket + "." + endpoint;  
	// 设置上传回调URL，即回调服务器地址，用于处理应用服务器与OSS之间的通信。OSS会在文件上传完成后，把文件上传信息通过此回调URL发送给应用服务器。  
	//String callbackUrl = "https://192.168.0.0:8888";  
	// 设置上传到OSS文件的前缀。取当前日期。  
	String format = new SimpleDateFormat("yyyy-MM-dd").format(new Date());  
	String dir = format + "/";  

	Map<String, String> respMap=null;  
	try {  
		long expireTime = 30;  
		long expireEndTime = System.currentTimeMillis() + expireTime * 1000;  
		Date expiration = new Date(expireEndTime);  
		PolicyConditions policyConds = new PolicyConditions();  
		policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, 1048576000);  
		policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, dir);  

		String postPolicy = ossClient.generatePostPolicy(expiration, policyConds);  
		byte[] binaryData = postPolicy.getBytes("utf-8");  
		String encodedPolicy = BinaryUtil.toBase64String(binaryData);  
		String postSignature = ossClient.calculatePostSignature(postPolicy);  

		respMap = new LinkedHashMap<String, String>();  
		respMap.put("accessId", accessId);  
		respMap.put("policy", encodedPolicy);  
		respMap.put("signature", postSignature);  
		respMap.put("dir", dir);  
		respMap.put("host", host);  
		respMap.put("expire", String.valueOf(expireEndTime / 1000));  
		// respMap.put("expire", formatISO8601Date(expiration));  
	} catch (Exception e) {  
		// Assert.fail(e.getMessage());  
		System.out.println(e.getMessage());  
	}  

	return respMap;  
}
```






































