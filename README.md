# WebGL Basic
브라우저에서 GPU 프로그래밍을 가능하게 하는 플랫폼

## CPU vs GPU

### Core
![CPU & GPU cores](https://kr.nvidia.com/content/tesla/images/cpu-and-gpu.jpg)
- CPU : 직렬처리 고성능 core ( 1,2,4~ )
- GPU : 병렬처리 저성능 core ( 수천~ )

### GPU 구조
![CPU&GPU](./img/webgl_cpu_gpu.png)
GPU에서 화면을 렌더링 하기 위한 필요 요소
- 색상버퍼 : 색상 정보를 담고 있는 스크린상의 각 픽젤 정보를 저장핸 직사각형 형태의 메모리 구조(RGB/RGBA)
- Z버퍼 : 픽셀이 갖는 z index 값을 저장하며, 색상버퍼와 같은 크기
- 스텐실버퍼 : 특정위치에 오브젝트가 그려지는 모양을 조정( 그림자 표현 )
- 텍스쳐메모리 : GPU용 이미지 메모리
- 비디오컨트롤러 : 주기적으로 라인단위 화면갱신( 초당60회/60Hz )

## WebGL 파이프라인
![CPU&GPU](./img/webgl_gpu_pipeline.png)

### 버텍스셰이더
- 3D모델링 데이터가 처음으로 진입하는 단계
- 정점의 변환이 이루어짐 ( [변환행렬](https://www.google.co.kr/imgres?imgurl=https://upload.wikimedia.org/wikipedia/commons/thumb/2/23/2D_affine_transformation_matrix-ko-001.svg/1200px-2D_affine_transformation_matrix-ko-001.svg.png&imgrefurl=https://ko.wikipedia.org/wiki/%25EB%25B3%2580%25ED%2599%2598%25ED%2596%2589%25EB%25A0%25AC&h=1600&w=1200&tbnid=aUFBWArxnKq_QM:&q=%ED%96%89%EB%A0%AC%EB%B3%80%ED%99%98&tbnh=160&tbnw=119&usg=AI4_-kSomHnfAkwEpSE10HFm5CG1pAHXdQ&vet=12ahUKEwi29Yv8qcXeAhXExLwKHTKxBhoQ_B0wF3oECAUQEA..i&docid=s3MLlgHSQw77OM&itg=1&sa=X&ved=2ahUKEwi29Yv8qcXeAhXExLwKHTKxBhoQ_B0wF3oECAUQEA) )
- 사용자에 의해 정의 됨

#### 버텍스셰이더 구조
![vertex shader](./img/webgl_vertex_shader.png)

#### 버텍스셰이더 변수
- attribute : 사용자가 버텍스셰이더에 전달하는 변수
- uniform : 사용자가 셰이더에 전달하는 전역변수
- varying : 버텍스셰이더가 프래그먼트에서 값을 전달할때 사용.

#### 버텍스셰이더 소스코드
```
attribute vec4 a_position;
void main(){
    gl_Position = a_position;
}
```

#### 버텍스셰이터 처리 흐름
![vertex shader flow](https://webgl2fundamentals.org/webgl/lessons/resources/vertex-shader-anim.gif)

### 프리미티브 어셈블리
![view frustum](./img/webgl_view_frustum.png)
- 배치된 버텍스를 이용해 삼각형 구성
- 뷰절두체 생성

### 단편화(레스터화)
- 연속적인 데이터를 컴퓨터가 표현할 수 있는 단위로 변환

### 프래그먼트셰이더
- 픽셀에 색을 채우는 단계

#### 프래그먼트셰이더 소스코드
```
precision mediump float;
void main() {
  gl_FragColor = vec4(1, 0, 0.5, 1);
}
```
### 프래그먼트 후처리
- 가위테스트
- 멀티샘플 프래그먼트 연산
- 스탠실 테스트
- 깊이버퍼 테스트
- 블랜딩
- 디더링

## 삼각형 출력
![view frustum](./img/webgl_triangle.png)

1. webgl 컨텍스트 생성
2. 버텍스 버퍼 생성
3. 셰이더소스 작성
4. 셰이더 컴파일
5. 셰이더 프로그램 생성
6. 그리기
### 1. webgl 컨텍스트 생성
``` javascript
const canvas = document.getElementById('gl-canvas');
const gl = canvas.getContext('webgl');
```
### 2. 버텍스 버퍼 생성
``` javascript
const vertexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER,  vertexBuffer);
gl.bufferData(gl.ARRAY_BUFFER, 
              new Float32Array([
                  0, .5, 0,
                  -.5, -.5, 0,
                  .5, -.5, 0]), 
                  gl.STATIC_DRAW);
```
### 3. 셰이더소스 작성
``` javascript
const vertexShaderSource = `
    attribute vec3 aVertexPosition;
    void main(){
        gl_Position = vec4(aVertexPosition, 1);
    }
`;

const fragmentShaderSource = `
    precision mediump float;
    void main(){
        gl_FragColor = vec4(1, 1, 1, 1);
    }
`;
```
### 4. 셰이더 컴파일
``` javascript
const vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertexShaderSource);
gl.compileShader(vertexShader);
if(!gl.getShaderParameter(vertexShader, gl.COMPILE_STATUS)){
    console.log( '버텍스 셰이더 컴파일 실패' );
    gl.deleteShader(vertexShader);
}

const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragmentShaderSource);
gl.compileShader(fragmentShader);
if(!gl.getShaderParameter(fragmentShader, gl.COMPILE_STATUS)){
    console.log( '프레그먼트 셰이더 컴파일 실패' );
    gl.deleteShader(fragmentShader);
}
```
### 5. 셰이더 프로그램 생성
``` javascript
const shaderProgram = gl.createProgram();
gl.attachShader(shaderProgram, vertexShader);
gl.attachShader(shaderProgram, fragmentShader);
gl.linkProgram(shaderProgram);
if(!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)){
    console.log( '셰이더 프로그렘 링크 실패' );
}
gl.useProgram(shaderProgram);
```
### 6. 그리기
``` javascript
gl.clearColor(0, 0, 0, 1);
gl.viewport(0, 0, canvas.width, canvas.height);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.vertexAttribPointer(
    gl.getAttribLocation(shaderProgram, 'aVertexPosition'), 
    3, 
    gl.FLOAT, 
    false, 
    0, 
    0 
);
gl.enableVertexAttribArray(gl.getAttribLocation(shaderProgram, 'aVertexPosition'));
gl.drawArrays(gl.TRIANGLES, 0, 3 );
```

