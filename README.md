Three.js 디지털 트윈 프로젝트 분석 보고서

본 문서는 smart_classroom_camera_stable.html(스마트 교실)과 ThreeJS_한국_시내버스_조립공장_3D_v2.html(시내버스 조립공장) 두 파일의 실행 방법과 코드 구조를 분석한 내용입니다.

1. 실행 방법 (How to Run)

두 파일 모두 별도의 외부 에셋(3D 모델 파일, 이미지 등) 다운로드 없이 프로시저럴(절차적) 방식으로 3D 객체를 생성하는 단일 HTML 파일(Single-File) 구조로 작성되었습니다. 다만, 브라우저 보안 정책(CORS)과 ES Module 로딩을 위해 다음과 같은 실행 환경이 필요합니다.

필수 조건

인터넷 연결: Three.js 라이브러리와 폰트를 CDN(cdn.jsdelivr.net)에서 실시간으로 불러오므로 인터넷 연결이 필수입니다.

권장 실행 방법 (로컬 웹 서버 사용)

단순히 HTML 파일을 더블클릭해서 브라우저로 열면( file:/// 프로토콜) <script type="module"> 내의 모듈(OrbitControls 등)을 불러올 때 CORS 에러가 발생하여 화면이 렌더링되지 않을 수 있습니다. 따라서 로컬 웹 서버를 구동하여 실행해야 합니다.

VS Code 사용자:

Visual Studio Code 에디터에서 해당 HTML 파일을 엽니다.

확장 프로그램인 Live Server를 설치합니다.

에디터 우측 하단의 Go Live 버튼을 누르거나, 마우스 우클릭 후 Open with Live Server를 선택합니다.

Python 설치 환경:

터미널(명령 프롬프트)을 열고 해당 파일이 있는 폴더로 이동합니다.

python -m http.server 8000 명령어를 입력합니다.

브라우저에서 http://localhost:8000으로 접속하여 파일을 실행합니다.

Node.js 사용자:

터미널에서 npx serve 또는 npm install -g http-server 후 http-server 명령어를 실행하여 접속합니다.

2. 전체적인 문법 및 아키텍처 구조

두 파일은 웹 기반 3D 인터랙티브 콘텐츠를 만들기 위한 매우 일관된 구조와 문법을 공유하고 있습니다. 외부 파일을 분리하지 않고 하나의 HTML 문서 안에 메타데이터, 스타일(CSS), UI 요소, 3D 렌더링 로직(JS)을 모두 통합한 형태입니다.

2.1. HTML 구조 (레이아웃 및 UI)

<head> 메타 및 스타일: CSS 변수(:root)를 활용한 테마 설정 및 Glassmorphism(투명/블러 효과) UI 스타일링이 정의되어 있습니다. 반응형 미디어 쿼리(@media)를 통해 모바일 환경의 UI도 대응합니다.

importmap 사용:

<script type="importmap">
  {"imports":{
    "three":"https://cdn.jsdelivr.net/npm/three@0.164.1/build/three.module.js",
    "three/addons/":"https://cdn.jsdelivr.net/npm/three@0.164.1/examples/jsm/"
  }}
</script>


이 구문을 통해 코드 내에서 import * as THREE from 'three';와 같이 모듈식으로 Three.js를 깔끔하게 불러옵니다.

<body> 오버레이 UI: <div id="world"> 또는 <div id="stage">와 같은 전체 화면 컨테이너를 바탕에 깔고, 그 위에 position: absolute 또는 fixed를 사용하여 패널, 버튼 등의 UI를 띄우는 방식입니다.

2.2. JavaScript (Three.js) 구조

두 파일 모두 <script type="module"> 안에서 핵심 로직이 실행되며, 크게 초기화 → 3D 모델링(프로시저럴) → 상태 관리 → 렌더링 루프의 단계를 거칩니다.

A. 씬 초기화 및 설정 (Scene Setup)

WebGLRenderer, Scene, PerspectiveCamera를 생성합니다.

그림자(shadowMap.enabled = true), 톤 매핑(ACESFilmicToneMapping), 물리적 기반 렌더링(PBR)에 맞는 색상 공간(SRGBColorSpace)을 설정하여 실사풍 렌더링 품질을 확보합니다.

OrbitControls를 연결하여 마우스로 화면을 회전/확대/이동할 수 있게 합니다.

B. 프로시저럴 모델링 패턴 (Procedural Generation)

외부 .gltf나 .obj 파일을 불러오지 않고, Three.js의 기본 기하학(Geometry)과 재질(Material)을 조합하여 모든 것을 코드로 그립니다.

유틸리티 함수 활용:

// 두 파일 모두 박스(bx), 원기둥(cyl), 구(sph) 등을 생성하는 래퍼 함수를 정의하여 코드를 압축합니다.
function bx(parent, w, h, d, mat, x=0, y=0, z=0) {
  const m = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), mat);
  m.position.set(x, y, z);
  parent.add(m);
  return m;
}


그룹화(THREE.Group): 버스의 부품 모듈이나 교실의 책상 세트처럼 관련된 객체들을 그룹으로 묶어 배치와 애니메이션을 용이하게 합니다.

C. 상태 관리 및 이벤트 바인딩 (State & Events)

데이터 객체 활용: 스마트 교실은 sim (시뮬레이션 상태), 버스 공장은 state 객체를 두어 현재 상태(온도, 조립 단계, 속도 등)를 중앙에서 관리합니다.

HTML DOM 이벤트 연결: 버튼 클릭(addEventListener('click')), 슬라이더 조절(input) 등의 이벤트가 발생하면 상태 객체를 업데이트하고, UI 텍스트를 갱신한 뒤, 3D 씬(재질 속성, 객체 위치 등)에 즉각 반영합니다.

D. 애니메이션 루프 (Render Loop)

requestAnimationFrame (또는 renderer.setAnimationLoop)을 사용하여 매 프레임마다 화면을 다시 그립니다.

THREE.Clock()을 사용해 DeltaTime을 구하고, 이를 기반으로 부드러운 애니메이션(부품 조립 이동, 카메라 전환, 환풍기 회전 등)을 계산합니다.

TWEEN 라이브러리 없이 자체적인 선형 보간(lerp, MathUtils.lerp) 수식을 사용하여 카메라와 부품을 부드럽게 이동시킵니다.

3. 파일별 세부 특징 분석

3.1. 스마트 교실 디지털 트윈 (smart_classroom_camera_stable.html)

데이터 기반(JSON) 아키텍처: <script type="application/json" id="sceneSpec"> 형태를 사용하여 공간의 크기, 책상 배치, 초기 센서 값 등을 JSON 데이터로 분리했습니다. 이는 하드코딩을 줄이고 유지보수를 쉽게 합니다.

IoT 센서 시뮬레이션: 온도, CO2, 조도 등의 가상 데이터를 수식 기반으로 시뮬레이션합니다. (예: 인원이 늘어나면 CO2가 증가, 환기를 켜면 감소).

동적 캔버스 텍스처: 칠판(Smart Board)이나 정보 태그(Sprite)를 이미지 파일 대신 document.createElement('canvas')를 이용해 실시간으로 텍스트와 그래프를 그려 Three.js의 재질로 맵핑(CanvasTexture)하는 고급 기법을 사용했습니다.

성능 최적화(Occlusion): 카메라가 벽 바깥으로 나갈 때 벽을 투명하게 하거나 숨겨서 내부를 볼 수 있게 하는 절개(Cutaway) 로직이 포함되어 있습니다.

3.2. 시내버스 조립공장 (ThreeJS_한국_시내버스_조립공장_3D_v2.html)

단계별 애니메이션 (Step-by-step): 9단계의 조립 공정 데이터를 배열(stageData)로 관리합니다. 각 부품 객체에 조립 단계(stage), 초기 위치(explode), 최종 위치(from) 데이터를 부여하여 조립 과정을 시각화합니다.

시각적 효과 (X-Ray & Explode):

X-Ray: 전체 재질(Material)을 순회하며 transparent와 opacity 속성을 조작하여 반투명 효과를 줍니다.

분해 보기 (Explode): 각 그룹의 위치 좌표(Position)를 미리 정의된 확산 좌표로 보간(lerp) 이동시킵니다.

카메라 제어 확장: OrbitControls 외에도 키보드(W, A, S, D, Q, E)로 공간을 비행하듯 이동할 수 있는 FlyControls 형태의 커스텀 로직을 자체 구현하여 키보드 이벤트와 카메라 좌표를 연동했습니다.

Raycaster 활용: 마우스로 3D 공간의 특정 부품(버스 타이어, 도어 등)을 클릭하면 정보를 우측 패널에 표시하기 위해 THREE.Raycaster로 교차 판정(Intersection)을 수행합니다.

결론

두 파일은 외부 에셋 종속성 없이 순수 코드로만 고품질 3D 환경을 구현하는 '단일 파일 기반 프로시저럴 디지털 트윈'이라는 매우 진보적이고 독특한 구조를 취하고 있습니다. 프론트엔드 프레임워크(React, Vue 등) 없이 바닐라 JavaScript만으로 DOM 조작과 상태 관리, Three.js 렌더링을 훌륭하게 동기화하고 있습니다.
