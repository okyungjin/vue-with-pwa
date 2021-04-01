# [PWA] Vue.js 프로젝트에 PWA 적용하기

## Project
* Project setup `npm install`
* Compiles and hot-reloads for development `npm run serve`
* Compiles and minifies for production `npm run build`
* ints and fixes files `npm run lint`

## PWA의 필수 항목 3가지
아래 3가지를 충족해야 PWA가 될 수 있다.

1. 웹 앱 매니페스트 (Web App Manifest)
2. 서비스 워커 (Service Worker)
3. HTTPS 프로토콜

## 1. Vue 프로젝트 생성하기
Vue 프로젝트가 없다면 Vue CLI를 통해 프로젝트를 생성한다.
```bash
npm install -g @vue/cli // Vue CLI를 전역으로 설치
vue create vue-with-pwa
```
필자는 `Default ([Vue 2] babel, eslint)` 프리셋을 선택했다.

## 2. Manifest.json 작성
>웹 앱 매니페스트란 **앱 소개 정보와 기본 설정을 담은 json 파일**을 말한다.
브라우저는 manifest.json 파일을 읽어 **"넌 일반 웹이 아니라 PWA구나"**라고 인식하게 된다.

직접 작성할 수 있으나, [SimiCart에서 제공하는 Manifest Genertor](https://www.simicart.com/manifest-generator.html/)를 이용하면 여러 사이즈의 아이콘과 `manifest.json` 파일을 얻을 수 있다.

### (1) manifest.json
Manifest Genertor를 사용하여 만들어진 `manifest.json` 파일이다.
```json
{
  "theme_color": "#3eaf7c",
  "background_color": "#3eaf7c",
  "display": "standalone",
  "scope": "/",
  "start_url": "/",
  "name": "Vue PWA",
  "short_name": "PWA",
  "icons": [
    {
      "src": "icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icons/icon-256x256.png",
      "sizes": "256x256",
      "type": "image/png"
    },
    {
      "src": "icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```
#### display 옵션
`manifest.json`의 `display` 옵션에는 4가지 종류가 있다.
* fullscreen : 기기의 최대 화면으로 보여준다
* standalone : 웹 브라우저의 주소, 상태 표시줄 등을 모두 제거한 화면을 보여준다 (가장 많이 사용)
* minimal-ui : 상단에 주소 표시줄만 추가한다
* browser : 웹 브라우저와 똑같은 모습으로 실행된다


#### manifest.json 경로
`manifest.json`은 `public` 안에 생성해주고, `public`에 `icons` 디렉토리를 생성하여 아이콘들을 넣어준다.
4가지의 아이콘을 모두 넣어주어야 한다.
```
public 
  ㄴ icons
    ㄴ icon-192x192.png
    ㄴ icon-256x256.png
    ㄴ icon-384x384.png
    ㄴ icon-512x512.png
  ㄴ index.html
  ㄴ manifest.json
```
### (2) index.html
```html
<html lang="en">
  <head>
    <link rel="manifest" href="/manifest.json" />
    <link rel="apple-touch-icon" sizes="192x192" href="/icons/icon-192x192.png">
    <link rel="apple-touch-icon" sizes="256x256" href="/icons/icon-256x256.png">
    <link rel="apple-touch-icon" sizes="384x384" href="/icons/icon-384x384.png">
    <link rel="apple-touch-icon" sizes="512x512" href="/icons/icon-512x512.png">
  </head>
```
`index.html` 의 `<head>` 태그에 위의 코드를 넣어준다.
매니페스트를 추가하고, 아이폰 사파리의 경우 아이콘을 불러오지 못해 또 import를 해주는 부분이다.

### (3) manifest 등록 확인
크롬 개발자모드를 통해 manifest가 잘 등록된 것을 확인할 수 있다.
![](https://images.velog.io/images/okyungjin/post/4f3d8c43-4d9a-4f9a-8ac7-b9ac84094058/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%204.48.25.png)


## 3. Service Worker 등록
서비스 워커는 PWA의 핵심이라고 할 수 있다.
>서비스 워커란 **웹 브라우저 안에 있지만 웹 페이지와는 분리되어 항상 실행되는 백그라운드 프로그램**을 말한다.
서비스 워커는 브라우저와 서버 사이에서 상탯값의 변경을 감시하고 푸시 알림으로 사용자에게 특정 메시지와 댓글 알림을 보낸다.

**웹 브라우저에 있지만 웹 페이지와는 분리되어 실행된다는 것이 핵심이다.**

### (1) registerServiceWorker.js
먼저, npm 모듈을 설치한다.
```bash
npm i register-service-worker
npm i @vue/cli-plugin-pwa
```

설치가 완료되면 `src` 에 `registerServiceWorker.js` 를 생성하여 아래 코드를 적어준다.
```js
import { register } from 'register-service-worker'

if (process.env.NODE_ENV === 'production') {
  register(`${process.env.BASE_URL}service-worker.js`, {
    ready () {
      console.log(
        'App is being served from cache by a service worker.\n' +
        'For more details, visit https://goo.gl/AFskqB'
      )
    },
    registered () {
      console.log('Service worker has been registered.')
    },
    cached () {
      console.log('Content has been cached for offline use.')
    },
    updatefound () {
      console.log('New content is downloading.')
    },
    updated () {
      console.log('New content is available; please refresh.')
    },
    offline () {
      console.log('No internet connection found. App is running in offline mode.')
    },
    error (error) {
      console.error('Error during service worker registration:', error)
    }
  })
}
```
### (2) main.js
해당 파일을 `main.js` 에서 import 해준다.
```js
// Service Worker
import './registerServiceWorker'
```
>* 필자처럼 localhost로 실행한다면 `if (process.env.NODE_ENV === 'production')` 코드를 주석처리 하면 된다. 물론 배포할 때는 주석을 제거한다.
* 혹시 오류가 난다면 종료하고 `npm run serve` 를 다시 실행해본다.

### (3) Service Worker 등록 확인
서비스 워커 등록이 된 것을 확인할 수 있다.
![](https://images.velog.io/images/okyungjin/post/46933088-9129-4299-a462-2f3b1229d51f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%205.23.34.png)
## 4. HTTPS 프로토콜
HTTPS(Hypertext transfer over secure socket layer)는 일반 텍스트를 통신하는 HTTP 프로토콜에 비해 **암호화와 인증을 거쳐 보안을 강화한 웹 통신 규약**이다. HTTPS는 웹 서버와 브라우저 사이에 주고받는 데이터가 암호화되므로, 해커가 데이터를 가로채도 어떤 내용인 지 알 수 없다.

>서비스 워커를 이용해서 PWA를 배포할 때는 반드시 HTTPS 프로토콜을 사용해야 한다.

왜냐하면
1. HTTPS 프로토콜은 Lighthouse라는 PWA 성능 평가 프로그램에서 인증을 받기 위한 의무 사항이다
2. `홈 화면에 추가` 기능은 HTTPS에서만 지원하기 때문이다
