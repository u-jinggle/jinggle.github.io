---
published: true
layout: single
title: "Cloudfront를 활용한 이미지 캐싱"
category: post
comments: true
---

이전 포스트에서는 imgproxy를 활용하여 On-demand Image Resizing을 구현해 보았습니다.  
[imgproxy를 활용한 On-demand Image Resizing 포스트 보러가기](https://u-jinggle.github.io/post/imgproxy)  
    
On-demand Image Resizing방식이 따로 썸네일을 크기별로 저장할 필요가 없기 때문에 간편하지만,  
사용자가 요청할 때마다 서버에서 이미지를 Resizing 해야하는 비용은 단점이 될 수 있습니다.  

이를 해결 하기 위해 한번 호출된 내용을 Caching하여 비용을 줄이는 방법을 알아보겠습니다.   

Caching은 AWS의 **CloudFront** 서비스를 활용해 보도록 하겠습니다.    

## CloudFront Process
![CloudFront 설정 예시이미지11](/assets/images/2019-12-30_13.png){: width="100%"}  
1. User -> CloudFront에 요청
    - Cache된 내용이 있을시 CloudFront에서 바로 응답(Cache Hit)
    - Cache된 내용이 없을시 CloudFront가 Origin서버로 요청(Cache Miss)
2. CloudFront -> Origin서버로 요청
    - S3의 이미지를 Origin 서버인 EC2의 이미지 프록시가 리사이즈하여 CloudFront와 User로 전달
    - CloudFront는 해당내용을 Caching  
        
CloudFront는 크게 위의 프로세스로 실행됩니다.    

## CloudFront 설정 
### Distribution 생성
![CloudFront 설정 예시이미지1](/assets/images/2019-12-30_1.png){: width="100%"} 

CloundFront의 독립적 단위인 Distribution을 추가해줍니다.

### 전송방식 선택
![CloudFront 설정 예시이미지2](/assets/images/2019-12-30_2.png){: width="100%"}

다음으로는 전송방식을 선택해야 합니다.   
웹서비스 이용할 경우 Web을, 동영상 스트리밍 등을 사용할 경우 RTMP를 선택하면 된다고 합니다.  
저는 웹서비스를 사용할 예정이므로 Web을 선택하고 넘어가 줍니다.  

### Origin 설정
CloudFront는 요청된 url에 대해 캐싱된 내용이 없을시, 이곳에서 설정된 Origin에서 원본데이터를 찾아 제공하며 해당 내용을 캐싱합니다.    
저는 EC2에 Docker를 올려 그곳에 imgproxy를 통해 이미지 리사이즈를 제공할 예정이므로  
EC2의 퍼블릭 DNS를 Origin Domain으로 설정해 주었습니다.  

#### EC2 public Dns
![CloudFront 설정 예시이미지4](/assets/images/2019-12-30_4.png){: width="100%"} 
#### CloudFront Origin Settings  
![CloudFront 설정 예시이미지3](/assets/images/2019-12-30_3.png){: width="100%"} 

저는 최소한의 Setting으로 테스트를 해볼 예정이므로 위와같이 설정후 Create Distribution 해주었습니다.  

## CloudFront를 통한 Cache
![CloudFront 설정 예시이미지5](/assets/images/2019-12-30_5.png){: width="100%"}

CloudFront Distribution 배포가 완료되면 이미지와 같이 생성됩니다.    
Domain Name을 통해 접속할 수 있습니다.  

### CloudFront 접속
![CloudFront 설정 예시이미지6](/assets/images/2019-12-30_6.png){: width="100%"}

접속해 보면 ec2에서 imgproxy가 실행되는것과 동일한 화면을 확인 할 수 있습니다.  

### CloudFront imgproxy Resize 호출
CloudFront를 통해 imgproxy resize가 잘 동작하는지 확인해 보도록 합시다.  
![CloudFront 설정 예시이미지7](/assets/images/2019-12-30_7.png){: width="100%"}

imgproxy url 호출 형식에 맞춰 호출해 보았을 때 잘동작하는 모습을 확인 할 수 있었습니다.  
첫 호출이기 때문에 cache된 내용이 없어 "X-Cache: Miss from cloudfront"가 확인됩니다.  

![CloudFront 설정 예시이미지8](/assets/images/2019-12-30_8.png){: width="100%"}

이미지의 Size와 Time도 CloudFront적용 이전과 비슷한 정도입니다.  

### CloudFront imgproxy Resize Cache
![CloudFront 설정 예시이미지9](/assets/images/2019-12-30_9.png){: width="100%"}

같은 url을 다시 호출해 보았을 때, "X-Cache: Hit from cloudfront"가 확인됩니다.  
호출한 내용을 CloudFront에서 반환했다는 것을 확인 할 수 있습니다.

![CloudFront 설정 예시이미지10](/assets/images/2019-12-30_10.png){: width="100%"}

이미지의 Size와 Time도 대폭 줄어든 모습을 확인 할 수 있습니다.  







