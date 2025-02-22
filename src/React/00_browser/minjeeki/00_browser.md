# 00_관련 CS

## 브라우저의 구조

![image.png](image.png)

### 사용자 인터페이스

: 주소 표시줄, 이전/다음 버튼, 북마크 메뉴 등 페이지를 보여주는 창 제외 사용자가 조작 가능한 부분

- 사용자가 URL을 입력할 때 렌더링 프로세스 시작의 트리거 역할을 수행

### 브라우저 엔진

: 사용자 인터페이스와 렌더링 엔진 사이 동작을 제어

- 렌더링 엔진에 문서 로딩을 지시하며, 각종 브라우저 작업의 제어와 조율을 담당함

### 렌더링 엔진

: 요청한 콘텐츠(HTML, CSS)를 화면에 표시 (파싱, 트리 생성, 레이아웃 계산, 페인팅)

- 브라우저의 가장 핵심적인 역할을 수행하는 부분
- 브라우저별 대표 엔진 : Webkit(Safari), Blink(Chrome)

### 통신

: HTTP/HTTPS 요청과 같은 네트워크 호출에 사용

- 리소스 다운로드를 담당하며 초기 불러오기 단계에서 핵심적인 역할을 수행한다
- 자바스크립트 엔진 (자바스크립트 해석기)
    
    : 자바스크립트 코드를 해석하고 실행
    
    - DOM과 CSSOM 조작 시 리렌더링을 유발하는데 사용됨
    - 비동기 리소스 로딩에도 관연한다
    - 브라우저별 대표 엔진 : V8 (Chrome), SpiderMonkey(Firefox)

### UI 백엔드

: 기본적인 위젯 (콤보 박스 등)을 그리는데 관여

- OS 사용자 인터페이스 시스템을 사용함
- 페인팅 단계에서 실제 화면 표시를 담당

### 데이터 저장소

: 자료를 저장하는 계층 (로컬 스토리지, 세션 스토리지, 쿠키 등을 관리)

- 캐시된 리소스 관리로 렌더링 최적화에 기여함

## 브라우저의 렌더링 과정
(Critical Rendering Path, 브라우저의 작동 원리)

![image.png](image%201.png)

### 네트워킹 모듈의 리소스 로드

1. 불러오기
    - 로더가 서버로부터 전달받은 리소스 스트림을 읽는 과정
    
    ---
    
    - 네트워킹 모듈이 리소스를 요청(request)/다운로드 진행
    데이터 저장소가 캐시된 리소스를 확인한 후 제공함

### 브라우저가 읽을 수 있도록 하기 : 렌더링 엔진의 파싱, 트리생성

1. 파싱 & DOM, CSSDOM 생성
(코드를 브라우저가 이해하기 쉬운 방식인 객체 모델의 형태로 변환)
    - HTML/XML 파서가 문서를 파싱해 DOM 트리 생성
        - DOM (Document Object Model) : 요소들의 위치, 배치, 모양에 관한 정보
        - <script> 태그를 만나면 파싱을 중단하고 JS 엔진에 제어권을 넘김
            
            ⇒ 자바스크립트는 파싱을 차단하기 때문에 script 위치가 중요하다
            
        - <link rel=”stylesheet”>를 만나면 CSS 파일 로드를 시작함
            
            ⇒ CSS는 렌더링을 차단하는 요소이기에 빠른 로드가 중요하다
            
        - <img>, <video> 등 외부 리소스 태그를 만나면 리소스 로드를 시작함
    - CSS 파서가 CSSOM 트리를 생성
        - CSSOM (CSS Object Model) : 요소들의 스타일과 관련된 모든 정보
        - CSS 파일과 <style> 태그 내용을 파싱해 CSSOM 트리를 구축한다
        - CSSOM에는 스타일 규칙 계산과 상속 관계가 처리되어 있다
    
    ---
    
    - 렌더링 엔진이 HTML/CSS 파서를 구동
    자바스크립트 엔진이 파싱 중 만나는 스크립트를 생성함
2. 자바스크립트 엔진이 자바스크립트 코드를 파싱하고 실행한다
    - DOM API를 이용한 DOM whwkr, CSSOM API를 이용한 스타일 조작에 관여한다
    - 이 외에도 이벤트 리스너를 등록하고 비동기 작업(AJAX, setTimeout 등)을 처리한다
    - script 태그의 async와 defer 속성
        - async : 비동기로 스크립트를 다운로드하고 즉시 실행
        - defer : 비동기로 스크립트를 다운로드하고 HTML 파싱 완료 후 실행
3. 생성된 DOM과 CSSOM으로 렌더 트리 생성 (브라우저 내 렌더링 엔진)
    - 렌더 트리 : 웹 페이지의 청사진 (설계도) / 화면에 렌더링되어야 하는 요소들의 모든 정보
    (화면에 표시될 요소만을 포함하며, 각 요소는 스타일 계산이 완료된 상태)
    - DOM 트리와 렌더 트리의 관계
        - DOM 트리는 문서의 모든 요소를 포함하지만, 렌더 트리는 실제 화면에 표시되는 요소만 포함한다 (`display: none`의 경우 렌더 트리에서 제외되나 `visibility: hidden` 은 포함)
        - 렌더 트리에는 <head> 요소와 <script> 태그, display: contents 속성을 가진 요소는 제외된다
    - 렌더링 엔진이 DOM과 CSSOM을 결합시켜 표시될 요소를 결정함

### 레이아웃, 페인트 : 화면에 보일 수 있게 하기

1. 레이아웃 : 렌더 트리를 토대로 노드와 스타일, 크기를 계산하는 것 (요소의 배치를 잡는 과정)
    - 렌더링 엔진이 뷰포트를 기반으로 요소의 위치와 크기를 계산함
    - % 단위를 px 단위로 변환하는 것 역시 이 작업에서 진행된다.
2. 페인트 : 렌더트리의 각 노드를 픽셀로 변환해 실제로 그리는 작업을 실행하는 것
(실제로 화면에 그려내는 과정)
    - 렌더링 엔진이 픽셀 변환 작업을 진행
    UI 백엔드가 레이어를 생성하고 실제 화면에 그리기 작업을 수행
3. 컴포지팅 (Compositing) : 페인팅된 레이어들을 합성하는 최종 단계
    - GPU 가속을 통한 성능 최적화 기능
    - transform: translateZ(0) 또는 will-change 속성으로 레이어를 생성할 수 있음

## 브라우저 화면 업데이트 과정 (리플로우와 리페인트, 리렌더링)

### 동적 변경 처리 (화면이 업데이트되는 상황)

- 자바스크립트에 의한 DOM/CSSOM 변경
    - DOM 변경 : 렌더링 엔진이 렌더 트리 재구성부터 진행
    - 레이아웃 변경 : 렌더링 엔진이 리플로우부터 수행
    - 스타일만 변경 : 렌더링 엔진이 리페인트 작업을 수행
- 비동기 리소스 로드 완료 시
    - 해당 요소를 렌더 트리에 추가한다
    - 필요한 경우에는 레이아웃 재계산이 이뤄진다

### 리렌더링, 리플로우트

1. 리렌더링 : 자바스크립트가 DOM, CSSOM을 변경하는 경우 → 렌더트리 재생성
2. 리플로우 : 재결합된 렌더트리를 기반으로 레이아웃을 다시 계산하는 것
3. 리페인트 : 재결합된 렌더트리를 기반으로 다시 페인트하는 것

---

- DOM 및 CSSOM 수정에는 자바스크립트 엔진이 관여하며,
렌더링 엔진이 수정된 내용을 감지해 필요한 단계부터 재실행한다
    
    이 전체 프로세스에는 브라우저 엔진이 조율한다
    

### 레이아웃과 리플로우는 계산 비용이 높은 작업이기에 렌더링 최적화에 신경써야 한다

- 자바스크립트는 파싱을 차단하기 때문에 script 태그의 위치가 중요하며
CSS는 렌더링을 차단하는 리소스이기 때문에 빠른 로드가 중요하다
- CSS 애니메이션은 가능한 transform, opacity을 사용해 리플로우를 최소화하는 것이 좋다
- 자바스크립트에서 DOM을 조작할 경우에는 일괄 처리해서 리플로우를 최소화하는 것이 좋다
(동시에 발생한 다양한 업데이트를 모으고, 다 모였을 경우 한번에 DOM을 수정)
- 레이아웃 스래싱(Layout Thrashing) 방지
    - 강제 동기 레이아웃(forced synchronous layout) 피하기
    - offsetWidth, offsetHeight 등의 레이아웃 속성 접근 최소화