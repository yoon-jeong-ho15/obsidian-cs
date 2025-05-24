# 개념
## S3Configuration
공식문서 : https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/s3/S3Configuration.html
### chuckedEncodingEnabled
public boolean chunkedEncodingEnabled()
Returns whether the *client should use chunked encoding* when *signing the payload body*.
This option only currently applies to [`PutObjectRequest`](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/s3/model/PutObjectRequest.html "class in software.amazon.awssdk.services.s3.model") and [`UploadPartRequest`](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/s3/model/UploadPartRequest.html "class in software.amazon.awssdk.services.s3.model").
Returns:
True if chunked encoding should be used.
#### 노트
[[S3 에러#GiveHub]]이 에러의 원인이 된 설정.
이걸 false설정을 주지 않았더니 데이터가 여러 조각으로 나눠져서 보내지는데 그때마다 검증을 할 수 없어서 에러가 발생했던것.
