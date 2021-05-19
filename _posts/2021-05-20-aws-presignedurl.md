---
layout: post
title:  "AWS Presigned url 그리고 Signed Url"
date:   2021-05-19 10:00
author: twosunny
tags:	[aws,presignedurl,signedurl,cdn,cloudfront]
---

## 들어가며
이미지/동영상 업로드 기능 및 CDN 은 이미지나 동영상을 제공하는 서비스에서는 너무나 필수적으로      
사용되고 있는 플랫폼이다.  
다만 인증된 사용자만 업로드 기능하게 한다던지, 넷플릭스 처럼 유료 회원만 특정 컨텐츠를 시청하게  
한다던지 하려면 꽤 많은 부분을 고려하여 개발을 해야 한다.  
AWS에서는 presinged url / signed Url 을 제공하여 이러한 니즈들을 보다 빠르게 구현할수 있다.  


### Presigned URL
>미리 서명된 URL의 생성자가 해당 객체에 대한 액세스 권한을 보유할 경우, 미리 서명된 URL은 URL에서  
>식별된 객체에 대한 액세스를 부여합니다. 즉, 객체를 업로드하기 위해 미리 서명된 URL을 수신하는 경우,  
>미리 서명된 URL의 생성자가 해당 객체를 업로드하는 데 필요한 권한을 보유하는 경우에만 객체를 업로드할 수 있습니다.  
>
>모든 객체 및 버킷은 기본적으로 비공개입니다. 사용자/고객이 특정 객체를 버킷에 업로드할 수 있기를 원하는 경우 미리   
>서명된 URL이 유용하지만 AWS 보안 자격 증명 또는 권한을 요구하지는 않습니다.  
>
>미리 서명된 URL을 생성하는 경우 보안 자격 증명을 제공한 후 버킷 이름, 객체 키, HTTP 메서드(객체 업로드 PUT)  
>및 만료 날짜/시간을 지정해야 합니다. 사전 서명된 URL은 지정된 기간 동안만 유효합니다. 즉, 만료 날짜 및 시간 전에  
>작업을 시작해야 합니다. 작업이 멀티파트 업로드와 같이 여러 단계로 구성된 경우 만료 전에 모든 단계를 시작해야 합니다.  

만약 비공개 S3에 특정 파일을 업로드 하려고 하는 경우 Presigned url은 꽤 유용하다.
물론 인증을 각 서비스별로 진행해야 하고(로그인 및 특정 권한을 가진 유저는 서비스에서 판단),  
인증된 사용자에게 Presigned url을 부여하여 파일을 업로드 하게 하면 된다.  

아래는 aws 가이드에서 발췌한 소스 중 주요 소스이다.(java)
```java
AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
                    .withCredentials(new ProfileCredentialsProvider())
                    .withRegion(clientRegion)
                    .build();

            // Set the pre-signed URL to expire after one hour.
            java.util.Date expiration = new java.util.Date();
            long expTimeMillis = expiration.getTime();
            expTimeMillis += 1000 * 60 * 60;
            expiration.setTime(expTimeMillis);

            // Generate the pre-signed URL.
            System.out.println("Generating pre-signed URL.");
            GeneratePresignedUrlRequest generatePresignedUrlRequest = new GeneratePresignedUrlRequest(bucketName, objectKey)
                    .withMethod(HttpMethod.PUT)
                    .withExpiration(expiration);
            URL url = s3Client.generatePresignedUrl(generatePresignedUrlRequest);

```  

저 소스를 돌리기 전에 미리 s3버킷 접근이 가능한 access key 및 secret key을 발급받아    
소스상으로 인증 처리는 사전에 해 놓아야 한다.   
또 하나 큰 장점은 파일 업로드 할때 네트워크 대역폭이나 별도 앱을 제작하지 않는다는게 큰 장점이다.  
자체 구현한다고 가정하면, 오리진 서버 / 업로드 서버 및 네트워크 대역폭 등 신경 쓸게 많지만  
미리 서명된 URL 을 사용하면 앱 서버에서는 Presign url만 클라이언트에게 던져주고, 실제 파일 업로드는  
클라이언트에서 진행되기 때문에 트래픽 및 대역폭을 신경쓰지 않아도 된다.  

만약 파일 업로드 구현을 처음부터 시작해야 하는 경우에는 S3와 Presinged URL 조합을 적극 추천한다.  
  

### Signed URL
>인터넷을 통해 콘텐츠를 배포하는 많은 기업에서는 유료 사용자 등 일부 사용자용으로 제작된 각종 문서,   
>비즈니스 데이터, 미디어 스트림 또는 콘텐츠에 대한 액세스를 제한하고자 합니다.   
>CloudFront를 통해 이러한 프라이빗 콘텐츠를 안전하게 제공하려면 다음과 같이 하세요.
>
>사용자가 특별한 CloudFront 서명된 URL 또는 서명된 쿠키를 사용하여 프라이빗 콘텐츠에 액세스하도록 합니다.  
>사용자가 오리진 서버(예: Amazon S3 또는 프라이빗 HTTP 서버)에서 직접 콘텐츠에 액세스하는 URL이 아닌   
>CloudFront URL을 사용하여 콘텐츠에 액세스해야 합니다. 반드시 CloudFront URL을 사용해야 하는 것은 아니지만,   
>서명된 URL 또는 서명된 쿠키에 지정한 제약 조건을 사용자가 우회하지 못하도록 하려면 사용하는 것이 좋습니다.
>

Signed Url 도 Presinged Url과 구조는 비슷한다.  
다만 방식이 조금 다른데, 서명된 URL로 접근하게 할것인지 서명된 쿠키로 접근하게 할 것인지  
이건 서비스를 어떻게 제공할지 방식에 따라 선택하면 된다.  

>다음과 같은 경우 서명된 URL을 사용합니다.  
>애플리케이션 설치 파일을 다운로드하는 등 개별 파일에 대한 액세스를 제한하고 싶은 경우  
>사용자가 쿠키를 지원하지 않는 클라이언트(예: 사용자 지정 HTTP 클라이언트)를 사용하는 경우  
>
>다음과 같은 경우 서명된 쿠키를 사용합니다.  
>HLS 형식의 비디오 파일 전체 또는 웹 사이트의 구독자 영역에 있는 파일 전체 등 제한된 파일 여러 개에 대한   
>액세스 권한을 제공하려는 경우, 현재의 URL을 변경하고 싶지 않은 경우  

예를 들어, 어떤 미디어 서비스가 오픈하기 전 베타 테스트를 진행한다고 가정하자.    
베타 서비스는 특정 유저들에게만 특정 시간대에 컨텐츠를 보여주고 싶을때  
Signed URL을 사용하면 자체 구현보다 쉽게 구현을 할수가 있다.   


## 마치며
파일 업로드나 미디어 활용은 우리 일상 생활속에서 꽤 깊숙히 자리 잡았다.    
다만 구현하려면 은근히 신경쓸 부분이 많고, 특히 보안적인 부분이나 대역폭 등 여러가지가 고려되어야 한다.  
파일 업로드나 CDN은 비용적인 면이나 개발 공수적인 면에서 aws를 활용하는건 꽤 매리트가 많은것 같다.    


## 참고사이트 
* https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/private-content-choosing-signed-urls-cookies.html
* https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/PresignedUrlUploadObject.html
 