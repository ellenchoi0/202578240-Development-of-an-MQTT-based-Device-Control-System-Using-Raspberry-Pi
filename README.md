# Development of a Bi-directional Remote LED Control System using MQTT and Multithreading

라즈베리파이와 MQTT 프로토콜을 활용하여 원격으로 LED를 제어하고, 장치 간 양방향 데이터 송수신이 가능한 IoT 통신 시스템을 구축하는 실습 프로젝트입니다.

> 본 프로젝트는 MQTT(Publish/Subscribe) 메커니즘을 기반으로 하며,
> 파이썬의 Threading 모듈을 도입하여 메시지 수신 대기와 데이터 발행이
> 동시에 이루어지는 멀티태스킹 환경 구현을 핵심으로 다룹니다.

## 개요

라즈베리파이에 Mosquitto 브로커를 구축하고, PC(MQTT.fx)와 라즈베리파이 간의 양방향 통신 환경을 조성합니다. 파이썬의 순차적 실행 구조로 인한 단방향 통신의 한계를 멀티스레딩(Multithreading) 기법으로 극복하여, 원격 제어와 실시간 데이터 전송이 동시에 가능한 시스템을 설계합니다.

## 핵심 특징

- **양방향 통신 구현**: PC에서의 LED 원격 제어(Subscribe)와 라즈베리파이의 상태 데이터 역전송(Publish) 동시 수행
- **멀티스레딩(Multithreading)**: threading 모듈을 사용하여 loop_forever()의 블로킹(Blocking) 현상을 해결하고 병렬 작업 구조 설계
- **의존성 해결 및 환경 구축**: gpiozero 구동을 위한 하위 라이브러리(swig, lgpio) 빌드 및 가상환경 최적화
- **실시간 모니터링**: MQTT.fx를 통한 데이터 누락 없는 실시간 카운트 수신 및 하드웨어 상태 검증

## 하드웨 아키텍처

본 프로젝트는 라즈베리파이 4 BCM 핀 배열을 기준으로 구성하였습니다.

## 실험 관련 이론

- **MQTT Broker (Mosquitto)**: 메시지 발행 및 구독을 중계하는 서버 소프트웨어로, 외부 접속 허용을 위한 보안 설정 수정이 필수적임
- **GPIOZERO 의존성**: 하드웨어 제어를 위해 liblgpio, lgpio, swig와 같은 C 언어 기반 핵심 라이브러리 선행 설치 필요
- **Thread(스레드)**: 프로세스 내에서 실행되는 작업의 최소 단위로, 메인 루프 외에 별도의 스레드를 생성하여 수신 대기 중에도 주기적인 데이터 송신 가능
  
## 동작 영상(YouTube)

실제 MQTT 통신을 통한 LED 제어 및 스레딩 동작 과정은 아래 영상에서 확인하실 수 있습니다.
- [라즈베리파이 MQTT 양방향 제어 실습(202578240 최서의)](https://youtu.be/z8fFxXIhTis)

## 실행 및 설치

### 1. 환경 구축 및 의존성 설치

라즈베리파이 가상환경 내에서 다음 라이브러리를 순차적으로 설치합니다.
> sudo apt-get install mosquitto mosquitto-clients swig liblgpio-dev
> pip install lgpio gpiozero paho-mqtt

### 2. 브로커 설정

mosquitto.conf 파일을 수정하여 외부 접속 및 익명 접속을 허용합니다.
> listener 1883 0.0.0.0
> allow_anonymous true

### 3. 프로그램 실행
- 텔레그램 Bot Token과 Chat ID를 소스 코드 내 해당 변수에 설정해야 합니다.
- 기상청 RSS URL의 zone 코드를 통해 원하는 지역을 설정할 수 있습니다.

## 주요 이슈 해결
- 전송 로직의 신뢰성: 초 단위 일치 방식(if hms == "HH:MM:SS")의 전송 누락 위험을 방지하기 위해, 향후 시간 범위 기반 체크 또는 전송 완료 플래그 도입의 필요성을 확인함.
- 코드 가독성 및 속도: 단순 for 루프를 리스트 컴프리헨션으로 전환하여 파이썬 인터프리터 수준에서 연산 속도를 최적화하고 가독성을 높임.
- 데이터 파싱 고도화: 정규표현식(re)의 한계를 극복하기 위해, 계층적 구조 분석에 유리한 BeautifulSoup 라이브러리 활용 방안을 검토하여 유지보수성 향상 기틀 마련.
- IoT 서비스 실증: 단순 정보 조회를 넘어 라즈베리파이가 네트워크 데이터를 스스로 판단하여 사용자에게 전달하는 능동적인 IoT 서비스 기초 역량을 확보함.
