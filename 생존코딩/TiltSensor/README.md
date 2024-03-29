# chapter08 TiltSensor

난이도 : ★★☆

프로젝트 명 : TiltSensor

기능

- 기기를 기울이면 수평을 측정할 수 있다. 화면에 표시되는 원이 가운데로 이동하면 수평이다.

핵심 구성요소

- SensorManager : 센서 관리자
- SensorEventListener : 센서 이벤트를 수신하는 리스너
- 커스텀 뷰 : 나만의 새로운 뷰를 만드는 방법

라이브러리 설정 : 없음





## 0) 해법요약

가속도 센서를 사용하여 수평 측정기를 만든다. 가로 모드로 고정하고 사용 중에는 화면이 꺼지지 않도록 한다. 수평 측정기 화면은 커스텀 뷰를 작성하고 그래픽 조작 API를 사용하여 그림을 그린다.

구현 순서

1. 준비하기 : 프로젝트 생성 및 안드로이드 설정
2. 스텝 1 : 액티비티의 생명주기 알아보기
3. 스텝 2 : 센서 사용하기
4. 스텝 3 : 커스텀 뷰 작성하기



## 1) 준비하기

프로젝트를 생성한다. Anko 라이브러리는 이 프로젝트에서는 필요 없다.

- 프로젝트명 : TiltSensor
- minSdkVersion : 19
- 기본 액티비티 : EmptyActivity





## 2) 액티비티의 생명주기(Lifecycle) 알아보기

센서를 다루기 전에 액티비티의 생명주기를 먼저 알아야 한다. 액티비티에는 특정 시점에 호출되는 여러 메서드가 있다. 예를 들어 onCreate()는 생성 시점에 호출된다. 이렇게 특정한 타이밍에 호출되는 메서드를 콜백 메서드라고 한다. 액티비티의 생명주기는 다음과 같다.

![01](https://user-images.githubusercontent.com/49340180/62274547-0f2b8c00-b47b-11e9-9c62-95669ed51cfe.PNG)

센서처럼 화면이 꺼져 있을 때는 센서가 동작하지 않고 화면이 켜져 있을 때만 동작하는 처리를 하는 경우가 있다. 만약 계속 센서가 동작한다면 배터리가 빨리 소모된다. 이처럼 적절한 타이밍에 필요한 코드를 작성하려면 생명주기를 잘 알아야 한다. 생명주기를 몇 가지 구간으로 나누어서 보면 좀 더 이해가 쉽다.





### 2-1) 액티비티 시작

액티비티가 시작되면 가장 먼저 onCreate() 메서드가 호출된다. 즉 onCreate() 메서드를 오버라이드하여 프로그램을 작성하면 액티비티가 시작되면서 작성한 프로그램이 자동으로 시작된다. 그 다음에는 onStart() 메서드와 onResume() 메서드 순으로 호출된다.

![01](https://user-images.githubusercontent.com/49340180/62274830-9bd64a00-b47b-11e9-970e-df494fa12140.PNG)



### 2-2) 액티비티 종료

액티비티가 종료될 때는 화면에서 보이지 않게 되는 순간 제일 먼저 onPause() 메서드가 호출되고 완전히 보이지 않게 되면 onStop() 메서드가 호출되며 마지막으로 onDestroy() 메서드가 호출된다. 간단한 앱을 작성할 때는 사용할 일이 거의 없지만, 복잡한 앱을 작성하면 액티비티가 종료될 때 메모리에서 해제하는 객체가 있을 수 있다. 이때 onDestroy() 메서드를 오버라이드 한다.

![01](https://user-images.githubusercontent.com/49340180/62275011-f40d4c00-b47b-11e9-824e-56b57e1ada2d.PNG)



### 2-3) 액티비티 재개

앱을 실행 중에 종료하지 않고 백그라운드에 대기하는 경우가 있다. 예를 들어 다른 앱이 실행되거나 홈 키를 누르거나 전원 버튼을 눌러 화면을 끄는 경우다. 이때는 onPause() , onStop()까지만 호출되고 대기하게 된다. 이때 다시 앱이 전면으로 나오는 경우, 즉 화면을 다시 켜거나 최근 실행 앱에서 다시 이 앱을 실행하는 경우다. 이때는 onRestart(),onStart(),onResume() 순으로 호출된다

![01](https://user-images.githubusercontent.com/49340180/62275151-4ea6a800-b47c-11e9-8013-b5317cdcf96a.PNG)



### 2-4) 프로세스 강제 종료

안드로이드의 모든 앱은 백그라운드 실행 중에는 메모리 부족 등으로 강제로 종료될 수 있다. 이 경우 앱을 다시 실행하면 onCreate() 메서드부터 호출한다.

![01](https://user-images.githubusercontent.com/49340180/62275263-8b729f00-b47c-11e9-8225-de52fb9805d3.PNG)





## 3) 센서 사용하기

가속도 센서의 사용 방벙에 관해서 알아본다. 센서의 사용 방법은 대부분 비슷해서 하나의 센서만 잘 사용하면 나머지도 쉽게 응용이 가능하다.

알아볼 순서

1. 센서 사용 준비
2. 가속도 센서 사용 방법
3. 가로 모드로 고정하기
4. 화면이 꺼지지 않게 하기





### 3-1) 센서

안드로이드 기기에는 많은 센서가 내장되어 있다. 다음은 Sensor 클래스에 정의된 안드로이드에서 공식으로 지원하는 센서 목록이다.

| 센서                     | 설명             | 용도                         |
| ------------------------ | ---------------- | ---------------------------- |
| TYPE_ACCELEROMETER       | 가속도 센서      | 동작 감지(흔들림, 기울임 등) |
| TYPE_AMBIENT_TEMPERATURE | 주변 온도 센서   | 대기 온도 모니터링           |
| TYPE_GRAVITY             | 중력 센서        | 동작 감지(흔들림, 기울임 등) |
| TYPE_GYROSCOPE           | 자이로 센서      | 회전 감지                    |
| TYPE_LIGHT               | 조도 센서        | 화면 밝기 제어               |
| TYPE_LINER_ACCELERATION  | 선형 가속도 센서 | 단일 축을 따라 가속 모니터링 |
| TYPE_MAGNETIC_FIELD      | 자기장 센서      | 나침반                       |
| TYPE_ORIENTATION         | 방향 센서        | 장치 위치 결정               |
| TYPE_PRESSURE            | 기압 센서        | 공기압 모니터링 변화         |
| TYPE_PROXIMITY           | 근접 센서        | 통화 중인지 검사             |
| TYPE_RELATIVE_HUMIDITY   | 상대 습도 센서   | 이슬점, 상대 습도를 모니터링 |
| TYPE_ROTATION_VECTOR     | 회전 센서        | 모션 감지, 회전 감지         |
| TYPE_TEMPERATURE         | 온도 센서        | 장치의 온도 감지             |

모든 기기에 이 센서가 다 내장된 것은 아니다. 제조사에 따라 여기에 포함되지 않은 센서를 쓰기도 한다. 프로젝트에서는 대부분의 기기에 기본으로 내장되어 있는 가속도 센서를 사용한다.



### 3-2) 좌표 시스템

일반적으로 센서 프레임워크에서는 센서값을 나타내는 데 x, y, z 표준 3축 좌표계를 사용한다. 일반적으로 기기를 정면으로 봤을 때 x축은 수평이며 오른쪽을 가리키고, y축은 수직이며 위쪽을 가리키고, z축은 화면의 바깥쪽을 향한다.

![01](https://user-images.githubusercontent.com/49340180/62281487-d4c8eb80-b488-11e9-9c6f-c6bb1583af0e.PNG)

이 좌표는 다음 센서에서 사용한다.

- 가속도 센서
- 중력 센서
- 자이로 스코프
- 선형 가속도 센서
- 자기장 센서





## 4) 커스텀 뷰 작성하기

수평계를 화면에 나타내려면 기존에 없는 뷰를 만들 필요가 있다. 없는 뷰를 만드는 것을 커스텀 뷰를 작성한다고 한다. 그래픽 API들을 사용하여 수평계를 나타내고 센서와 연동되는 뷰를 작성한다.

구현 순서

1. 수평계 그래픽 구상
2. 커스텀 뷰 작성하기
3. 그래픽 API를 사용해서 뷰에 그리기
4. 센서값을 뷰에 반영하기



## 6) 마치며

이번 프로젝트에서는 센서를 사용하는 방법과 커스텀 뷰를 작성하는 방법을 배웠다. 모든 센서는 다루는 방법이 비슷하기 때문에 한 번만 익혀두면 쉽게 다른 센서도 사용 가능하다.

- SensorManager 객체를 사용해 센서를 사용할 수 있다.
- View를 상속하여 커스텀 뷰를 작성할 수 있다. onDraw() 메서드는 뷰의 모양을 그리는 메서드다. Canvas, Paint 같은 그래픽 API를 사용하여 뷰의 모양을 그릴 수 있다.











