AppTree Web SDK 가이드.
----

### 개요
모바일 웹 사이트에서 AppTree 광고를 사용하기 위한 Web SDK 가이드 문서입니다.

### 1. 사전 안내
- 한 페이지에 AppTree 광고가 여러 개 일 경우 DIV 태그의 id 는 중복되지 않도록 주의해주십시오.

### 2-1. 초기화 파라메터 정의
|파라메터 이름|파라메터 설명|필수 여부|
|:---:|:---:|:---:|
|appId|<a href="http://www.apptree.co.kr" target="_blank">AppTree</a> 에서 발급 받은 appId|O|
|unitId|<a href="http://www.apptree.co.kr" target="_blank">AppTree</a> 에서 발급 받은 unitId|O|
|userId|사용자 고유값|O|
|adid|android의 경우 adid, ios의 경우에는 idfa, 혹은 idfv 값|O| 
|clientIp|광고 노출 지면 ID (Default:1)|O| 

### 2-2. 광고 load 함수 호출 후 광고를 성공적으로 가져온 경우 넘어오는 객체
넘어오는 객체는 CpmResponseDto의 객체가 넘어오고, 아래 설명에는 CpmResponseDto 안에 있는 객체의 세부 내용까지 
포함하여 NativeAd의 구조까지 포함하였습니다. 
```typescript
class CpmResponseDto {
  statusCode: number; // 200인 경우만 성공 그외에는 실패
  message: string[] = []; // 메시지는 배열의 형태로 넘어온다. 성공과 실패에 대한 내용만 존재
  ad?: NativeAd; // 일반적인 광고의 경우에는 이쪽으로 데이터가 넘어온다. 
  ads?: NativeAd[]; // 롤링배너의 경우, NativeAd가 배열로 넘어온다.
}

class NativeAd {
  adTitle!: string; // 광고 제목
  adDesc!: string;  // 광고 상세설명
  link!: string;    // 광고링크 이 url로 광고를 클릭시 이동한다. 
  impressTrackers: string[] = []; // 직접적으로 사용할 일은 없음
  clickTrackers: string[] = [];   // 직접적으로 사용할 일은 없음
  cta!: string; // 버튼이 존재하는 광고에서 버튼의 이름
  iconUrl?: string; // 아이콘 광고의 경우 icon image url
  iconWidth?: number; // 아이콘 가로
  iconHeight?: number; // 아이콘 높이
  imageUrl!: string; // 광고 이미지 url
  imageWidth!: number; // 광고 가로 사이즈
  imageHeight!: number; // 광고 세로 사이즈 
}
```

### 

### 3. 작성 예제

#### 1) 배너 (Banner)
AppTree 배너 광고를 사용하기 위해서는  
아래와 같이 위에서 정의한 필요한 파라메터들과 함께 AppTree.Ads 클래스를 초기화 하도록 합니다.
```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>web sdk example</title>
  <script src="//code.jquery.com/jquery-3.6.0.min.js"></script>
  <script src="//web-sdk.apptree.co.kr/apptree.sdk.min.js"></script>
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/swiper@11/swiper-bundle.min.css" />
  <script src="//cdn.jsdelivr.net/npm/swiper@11/swiper-bundle.min.js"></script>
</head>
<style>
  .row {
    height: 50vh;
    /* 세로 높이를 화면의 1/3로 설정 */
    width: 100%;
    /* 가로 폭을 화면 전체로 설정 */
    touch-action: none;
  }
</style>

<body>
  <script>
    function createBannerItem(ad, impressFun, clickFun) {
      var aElement = document.createElement("a");
      var imgElement = document.createElement("img");

      aElement.href = ad.link;
      aElement.addEventListener("click", clickFun);

      imgElement.src = ad.imageUrl;
      imgElement.style.maxWidth = '100%';
      imgElement.style.width = ad.imageWidth;
      imgElement.style.height = 'auto';
      imgElement.addEventListener("load", impressFun);

      aElement.appendChild(imgElement);
      return aElement;
    }

    function loadBanner() {
      const ads = new Apptree.Ads({
        appId: "com.ebcard.bustago",
        unitId: "lc258",
        adid: "1bbe89be-1c50-4667-b120-e5516d2c4ef1",
        userId: "1",
        clientIp: "192.168.0.1"
      });
      let adsData = {};
      ads.load((res) => {
        adsData = res;
        console.log(adsData);

        try {
          const banner1 = createBannerItem(res.ad, () => ads.impress(adsData.ad), (event) => {
            event.preventDefault();
            ads.click(adsData.ad);
            window.open(adsData.ad.link);
          });
          const banner1Div = document.querySelector('#banner1Div');
          banner1Div.appendChild(banner1);
        } catch (error) {
          console.log(error);
        }
      }, () => {
        // 광고를 가져온데 실패를 하게 되면 호출이 된다. 
      });
    }

    function loadRollingBanner() {
      const rollingAds = new Apptree.Ads({
        appId: "com.ebcard.bustago",
        unitId: "lc45",
        adid: "1bbe89be-1c50-4667-b120-e5516d2c4ef1",
        userId: "1",
        clientIp: "192.168.0.1"
      });

      let rollingAdsData = {};
      rollingAds.load((res) => {
        rollingAdsData = res;
        console.log(res);
        const rollingBanner = document.querySelector('.swiper-wrapper');
        for (let index = 0; index < res.ads.length; index++) {
          const ad = res.ads[index];
          var liElement = document.createElement("div");
          liElement.setAttribute('class', 'swiper-slide');
          const bannerItem = createBannerItem(ad, () => {
            setTimeout(() => {
              rollingAds.impress(ad);
            }, index * 100);
          }, (event) => {
            event.preventDefault();
            rollingAds.click(ad);
            window.open(ad.link);
          });
          liElement.appendChild(bannerItem);
          rollingBanner.appendChild(liElement);
        }

        var swiper = new Swiper('.swiper-container', {
          nested: true,
          loop: true, // 루프 활성화
          autoplay: {
            deplay: 3000,
            // disableOnInteraction: false,
          },
          navigation: {
            nextEl: '.swiper-button-next',
            prevEl: '.swiper-button-prev',
          },
        });
      }, () => { });
    }

    $(document).ready(function () {
      loadBanner();
      loadRollingBanner();
    });
  </script>
  <!-- 배너 광고 구현 -->
  <div id="banner1Div" class="row"></div>
  <!-- 롤링배너 광고 구현 -->
  <div class="row">
    <div class="swiper-container">
      <div class="swiper-wrapper"></div>
    </div>
  </div>

</body>

</html>
```
모바일 웹페이지를 WebView 를 통해 표시하는 Hybrid 앱에서는 디바이스 광고 식별자,  
즉 GAID (Google Advertiser Indentifier) 혹은 IDFA (Apple Identifier For Advertising) 를 AppTree 로 전달해야 합니다. 흐름은 다음과 같습니다.  

a) Hybrid 앱에서 에서 먼저 GAID 혹은 IDFA 를 직접 추출합니다.  

* 디바이스 광고 식별자 추출 방법 : [Android](https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient) / [iOS](https://developer.apple.com/kr/app-store/user-privacy-and-data-use)
* WebView 내의 Javascript 와 네이티브 코드 간 데이터 전달 방법 : [Android](https://developer.android.com/guide/webapps/webview?hl=ko#BindingJavaScript) / [iOS](https://developer.apple.com/documentation/webkit/wkusercontentcontroller?language=objc)
