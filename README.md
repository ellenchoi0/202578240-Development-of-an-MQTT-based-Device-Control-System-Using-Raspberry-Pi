# Development of an MQTT-based Device Control System Using Raspberry Pi

라즈베리파이와 MQTT 프로토콜을 활용하여 원격으로 LED 하드웨어를 제어하고, 장치 간 데이터 송수신을 처리하는 양방향 통신 시스템 구축 프로젝트입니다.

> 본 프로젝트는 Mosquitto 브로커 구축과 gpiozero 라이브러리의 의존성 해결 과정을 다루며,
> 특히 파이썬의 threading 모듈을 도입하여 메시지 수신과 데이터 발행을 동시에 처리하는 멀티태스킹 메커니즘을 구현합니다.

## 개요

라즈베리파이에 MQTT 브로커를 설치하고, PC(MQTT.fx)와 라즈베리파이 간의 발행(Publish) 및 구독(Subscribe) 메커니즘을 실습합니다. 단방향 통신의 한계를 극복하기 위해 CPU 작업 단위인 스레드(Thread)를 분리하여, 무한 루프 상태에서도 실시간 양방향 데이터 교환이 가능한 IoT 제어 환경을 학습합니다.

## 핵심 특징

- **원격 하드웨어 제어**: MQTT 프로토콜을 통한 3색 LED(Red, Green, Blue)의 실시간 원격 온/오프
- **양방향 통신 구현**: 제어 명령 수신과 동시에 센서 데이터(카운트 값)를 역전송하는 로직 구축
- **멀티태스킹 설계**: threading 모듈을 활용하여 수신 대기(Blocking) 문제를 해결하고 병렬 작업 수행
- **의존성 최적화**: 하드웨어 제어를 위한 핵심 C 라이브러리(liblgpio) 및 파이썬 바인딩 환경 구축

## 하드웨어 아키텍처

본 프로젝트는 라즈베리파이의 BCM 핀 배열을 기준으로 설계되었습니다.

## 회로 구성 및 통신 이론

- **MQTT 브로커(Mosquitto)**: 메시지 중계 역할을 수행하며, 외부 접속 허용을 위한 listener 설정 적용
- **하위 라이브러리 의존성**: gpiozero 구동을 위해 swig(빌드 도구) 및 liblgpio(C 라이브러리) 선행 설치 필요
- **스레딩(Threading)**: 파이썬의 순차적 실행 구조를 분리하여 수신(Subscribe)과 송신(Publish)을 독립된 작업 단위로 처리
  
## 동작 영상(YouTube)

실제 회로 구성 후 MQTT 통신을 통해 양방향 제어가 어떻게 이루어지는지 아래 영상에서 확인하실 수 있습니다.
- [MQTT 통신으로 제어하는 장치 만들기(202578240 최서의)](https://youtu.be/IdsUZdhM2V4)

## 실행 및 설치

### 1. 의존성 설치

가상환경 활성화 후 아래 순서대로 필수 패키지를 설치합니다.
> sudo apt install swig               # 빌드 도구 설치
> sudo apt install liblgpio-dev       # C 핵심 라이브러리 설치
> pip install lgpio                   # Python 바인딩 설치
> pip install gpiozero                # 제어 및 통신 라이브러리 설치

### 2. MQTT 브로커 설정

Mosquitto 설치 후 외부 기기 접속을 위해 /etc/mosquitto/mosquitto.conf 수정이 필요합니다.
> listener 1883 0.0.0.0 및 allow_anonymous true 설정 추가

### 3. 서버 실행

터미널에서 python3 main30-1.py 명령어를 입력하여 실행합니다.
- 메인 스레드: client.loop_forever()를 통한 메시지 수신 대기
- 서브 스레드: threading.Thread를 통한 주기적 데이터 발행

## 주요 이슈 해결
- 브로커 명칭 오타 및 경로 오류: Mosquitto(t가 두 개) 명칭 확인 및 가상환경 활성화(source) 시 경로 불일치 문제를 ls와 pwd 명령어를 통해 디버깅함.
- 순차 실행 구조의 한계 극복: loop_forever() 실행 시 프로그램이 멈추는 블로킹(Blocking) 현상을 threading 기법 도입으로 해결하여 멀티태스킹 환경을 성공적으로 구축함.
- 하드웨어 보호 및 자원 관리: GND 핀 오결선 방지를 위한 물리적 결선 재검토와 프로그램 종료 시 LED 잔광 제거를 위한 상태 초기화 로직을 강화함.
- 양방향 통신 검증: MQTT.fx를 활용하여 led 토픽으로 제어 명령을 보내는 동시에 hello 토픽으로 전송되는 라즈베리파이의 데이터를 실시간 모니터링하여 시스템 안정성을 확보함.
