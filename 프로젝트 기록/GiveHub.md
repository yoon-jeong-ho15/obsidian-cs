# R2 사용

## s3 sdk
아마존의 AWS S3 SDK를 사용해서 R2 서비스를 이용했다.
그래서 https://developers.cloudflare.com/r2/examples/aws/aws-sdk-java/ 
공식 문서에서 제시한 코드를 살펴보았다.

### 코드1 (공식)
```java
/**
 * Client for interacting with Cloudflare R2 Storage using AWS SDK S3 compatibility
 */
public class CloudflareR2Client {
    private final S3Client s3Client;

    /**
     * Creates a new CloudflareR2Client with the provided configuration
     */
    public CloudflareR2Client(S3Config config) {
        this.s3Client = buildS3Client(config);
    }

    /**
     * Configuration class for R2 credentials and endpoint
     */
    public static class S3Config {
        private final String accountId;
        private final String accessKey;
        private final String secretKey;
        private final String endpoint;

        public S3Config(String accountId, String accessKey, String secretKey) {
            this.accountId = accountId;
            this.accessKey = accessKey;
            this.secretKey = secretKey;
            this.endpoint = String.format(
            "https://%s.r2.cloudflarestorage.com", accountId);
        }

        public String getAccessKey() { return accessKey; }
        public String getSecretKey() { return secretKey; }
        public String getEndpoint() { return endpoint; }
    }

    /**
     * Builds and configures the S3 client with R2-specific settings
     */
    private static S3Client buildS3Client(S3Config config) {
        AwsBasicCredentials credentials = AwsBasicCredentials.create(
            config.getAccessKey(),
            config.getSecretKey()
        );

        S3Configuration serviceConfiguration = S3Configuration.builder()
            .pathStyleAccessEnabled(true)
            .build();

        return S3Client.builder()
            .endpointOverride(URI.create(config.getEndpoint()))
            .credentialsProvider(StaticCredentialsProvider.create(credentials))
            .region(Region.of("auto"))
            .serviceConfiguration(serviceConfiguration)
            .build();
    }

    /**
     * Lists all buckets in the R2 storage
     */
    public List<Bucket> listBuckets() {
        try {
            return s3Client.listBuckets().buckets();
        } catch (S3Exception e) {
            throw new RuntimeException(
            "Failed to list buckets: " + e.getMessage(), e);
        }
    }

    /**
     * Lists all objects in the specified bucket
     */
    public List<S3Object> listObjects(String bucketName) {
        try {
            ListObjectsV2Request request = ListObjectsV2Request.builder()
                .bucket(bucketName)
                .build();

            return s3Client.listObjectsV2(request).contents();
        } catch (S3Exception e) {
            throw new RuntimeException(
            "Failed to list objects in bucket "
            + bucketName + ": " + e.getMessage(), e);
        }
    }

    public static void main(String[] args) {
        S3Config config = new S3Config(
            "your_account_id",
            "your_access_key",
            "your_secret_key"
        );

        CloudflareR2Client r2Client = new CloudflareR2Client(config);

        // List buckets
        System.out.println("Available buckets:");
        r2Client.listBuckets().forEach(bucket ->
            System.out.println("* " + bucket.name())
        );

        // List objects in a specific bucket
        String bucketName = "demos";
        System.out.println("\nObjects in bucket '" + bucketName + "':");
        r2Client.listObjects(bucketName).forEach(object ->
            System.out.printf("* %s (size: %d bytes, modified: %s)%n",
                object.key(),
                object.size(),
                object.lastModified())
        );
    }
}
```

이건 이제 순수 Java를 위한 코드이고 이걸 스프링 프레임워크에 맞춰서 작성하면
### 코드2 (스프링)
```java
@Component
public class CloudflareR2Client {
    private final S3Client s3Client;

    public CloudflareR2Client(S3Client s3Client){
        this.s3Client = s3Client;
    }

    public List<Bucket> listBuckets() {
        try {
            return s3Client.listBuckets().buckets();
        } catch (S3Exception e) {
            throw new RuntimeException(
	            "Failed to list buckets: " + e.getMessage(), e);
        }
    }

    public List<S3Object> listObjects(String bucketName) {
        try {
            ListObjectsV2Request request = ListObjectsV2Request.builder()
                    .bucket(bucketName)
                    .build();

            return s3Client.listObjectsV2(request).contents();
        } catch (S3Exception e) {
            throw new RuntimeException(
	            "Failed to list objects in bucket " 
	            + bucketName + ": " + e.getMessage(), e);
        }
    }
}
```

```java
@Configuration
public class S3Config {
    @Value("${cloudflare.r2.account.id}")
    private String accountId;
    @Value("${cloudflare.r2.access.key}")
    private String accessKey;
    @Value("${cloudflare.r2.secret.key}")
    private String secretKey;

    @Bean
    public S3Client buildS3Client(){
        String endpoint = String.format(
        "https://%s.r2.cloudflarestorage.com", accountId);

        AwsBasicCredentials credentials = AwsBasicCredentials.create(
                accessKey, secretKey
        );

        S3Configuration serviceConfiguration = S3Configuration.builder()
                .pathStyleAccessEnabled(true)
                .build();

        return S3Client.builder()
                .endpointOverride(URI.create(endpoint))
                .credentialsProvider(
	                StaticCredentialsProvider.create(credentials))
                .region(Region.of("auto"))
                .serviceConfiguration(serviceConfiguration)
                .build();
    }
}
```

사실 `S3Client`를 그대로 사용하는건데 왜 이렇게 하는건지는 모르겠다.
파이널 프로젝트에서 썼던 s3config 코드랑 거의 흡사다하다.

```java
@Configuration
public class S3Config {
	
	@Value("${cloud.aws.credentials.access-key}")
	private String accessKey;
	
	@Value("${cloud.aws.credentials.secret-key}")
	private String secretKey;
	
	@Value("${cloud.aws.region.static}")
	private String region;
	
	@Bean
	public AmazonS3Client amazonS3Client() {
		// AWS 인증 정보 (Access Key, Secret Key) 설정
		BasicAWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);
		
		return (AmazonS3Client) AmazonS3ClientBuilder
					.standard() // 기본설정
					.withRegion(region)
					.withCredentials(
					new AWSStaticCredentialsProvider(credentials))
					.build();
	}
}
```
### 문제점
래퍼 메소드를 계속 사용해야한다.
```java
 public void uploadImage(String bucket, String key, byte[] data){
        try{
            PutObjectRequest request = PutObjectRequest.builder()
                    .bucket(bucket)
                    .key(key)
                    .contentType("images")
                    .build();
            s3Client.putObject(request, RequestBody.fromBytes(data));
        } catch (S3Exception e){
            throw new RuntimeException("Failed to upload image"+e.getMessage());
        }
    }
```
이렇게 버켓과 연관되는 기능을 다 정의해야한다.
애초에 `s3client`만 사용한다면 service층에서 즉시 사용할 수 있겠지만.

## 에러
[[S3 에러#GiveHub]]
