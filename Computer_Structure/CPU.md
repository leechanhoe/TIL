![](https://velog.velcdn.com/images/dodo4723/post/088063b5-bc9b-4de9-b6fd-32ab79c4ca26/image.png)

저는 이전부터 컴퓨터구조는 어떻게 이루어져있는지 무척 궁금했습니다. 어떻게 이렇게 빠를수가 있고, 어떻게 우리에게 화면을 보여주며, 어떻게 우리의 삶을 편안하게 해주는지 궁금했습니다.

저번학기(2-2)에 들었던 디시털시스템설계 과목에서 이런 궁금증중 하나인 컴퓨터는 어떻게 정보를 저장할수 있을까에 대한 해답을 찾을 수 있었습니다. **래치 -> 플립플롭 -> 레지스터**까지 발전해 온 과정을 알 수 있었습니다. 하지만 `레지스터`의 작동 방식은 알아도, 어떻게 사용되는지는 알지 못했습니다.

이 책을 2주간 읽고 제가 궁금했던 내용들을 많이 알게 되었습니다. 인상깊은 내용이나 중요한 내용들을 정리해보았습니다. 컴퓨터구조 파트와 운영체제 파트가 있는데 먼저 앞 파트인 컴퓨터구조  중 CPU부터 살펴보겠습니다.

<br>
<br>
<br>

## 명령어

명령어는 연산 코드와 오퍼랜드로 구성되어 있습니다.

### 1. 연산 코드
명령어가 수행할 연산을 의미
<br>

**연산 코드 유형**
>
- 데이터 전송 - move, store, load등
- 산술/논리 연산 - add, divide, and/or/not, increment등
- 제어 흐름 변경 - jump, halt, call등
- 입출력 제어 - read, write, start io등

<br>
<br>

### 2. 오퍼랜드
`연산에 사용할 데이터` 또는 `연산에 사용할 데이터가 저장된 위치`를 의미. 

보통 메모리 주소나 레지스터 이름이 담겨 **주소 필드**라고도 부름

명령어 안에 하나도 없을 수도 있고, 여러개가 있을 수도 있음.

<br>
<br>

### 3. 주소 지정 방식
오퍼랜드에 데이터 대신 해당 데이터의 메모리 주소(**유효 주소**)를 명시하면 표현할 수 있는 정보의 크기가 더 커집니다.

유효 주소를 찾는 방법을 주소 지정 방식 이라고 합니다.
>
- `즉시 주소 지정 방식` - 데이터를 오퍼랜드 필드에 직접 명시
- `직접 주소 지정 방식` - 유효 주소를 직접적으로 명시
- `간접 주소 지정 방식` - 유효 주소의 주소를 명시 - 직접 주소 
지정 방식보다 연산 코드의 크기만큼 표현 정보가 늘어남
- `레지스터 주소 지정 방식` - 직접 주소 지정 방식과 비슷하나 데이터가 레지스터에 저장
- `레지스터 간접 주소 지정 방식` - 데이터는 메모리에, 유효 주소는 레지스터에, 그 레지스터 주소를 명시

<br>
<br>
<br>


# 1. CPU
메모리에 저장된 명령어를 읽어 들이고, 해석하고, 실행하는 장치

`ALU`, `제어장치`, `레지스터`로 구성되어 있습니다.

### ALU
- 계산하는 부품입니다

- 레지스터를 통해 피연산자를 받아들이고, 제어장치로부터 수행할 연산을 알려주는 제어 신호를 받아들입니다.

- 연산을 수행한 결과는 특정 숫자나 문자, 주소가 될 수 있고 레지스터에 저장됩니다.

- 연산 결과에 대한 추가적인 정보인 **플래그**도 플래그 레지스터에 저장합니다.(부호 플래그, 오버플로우 플래그, 제로 플래그 등)

![](https://velog.velcdn.com/images/dodo4723/post/55268dcb-9c6b-449c-bb95-d31d854039ad/image.jpg)


<br>

### 제어장치
- 제어 신호를 내보내고, 명령어를 해석하는 부품으로 일종의 전기 신호입니다.

- `클럭 신호`, `해석해야할 명령어`, `플래그 값`, 제어 버스를 통해 `제어 신호`를 받아들입니다.

- 제어 버스로 **CPU 외부**(`메모리`, `입출력장치`)에 제어 신호를 전달합니다

- **CPU 내부**(`ALU` - 수행할 연산 지시, `레지스터` - 데이터 이동이나 명령어 해석)에도 제어 신호를 보냅니다.

![](https://velog.velcdn.com/images/dodo4723/post/413be8b4-c49b-4330-9114-db332aab2f55/image.jpg)


<br>

### 레지스터
프로그램 속 명령어와 데이터는 실행 전후로 반드시 레지스터에 저장됩니다.

**레지스터의 종류**
>
- `프로그램 카운터` : 메모리에서 읽어 들일 명령어의 주소 저장
- `명령어 레지스터` : 방금 메모리에서 읽어 들인 명령어를 저장하는 레지스터
- `메모리 주소 레지스터` : 메모리의 주소를 저장하는 레지스터 - CPU가 읽어 들이고자 하는 주소 값을 주소 버스로 보낼 때 거침
- `메모리 버퍼 레지스터` : 메모리와 주고받을 값(데이터와 명령어)를 저장
- `범용 레지스터` : 데이터와 주소를 모두 저장가능
- `플래그 레지스터` : 플래그 값을 저장
- `스택 포인터` : 스택 주소 지정 방식에 사용되어 스택의 꼭대기를 가리킴
- `베이스 레지스터` : 베이스 레지스터 주소 지정 방식에서 오퍼랜드와 이 값을 더하여 유효 주소를 얻음

<br>

### 인터럽트
CPU의 작업을 방해하는 신호로 동기 인터럽트인 예외와 비동기 인터럽트인 하드웨어 인터럽트가 있습니다.

**하드웨어 인터럽트 처리 순서**
>
1. 입출력 장치는 CPU에 **인터럽트 요청 신호**를 보냄
2. CPU는 실행 사이클이 끝나고 명령어를 인출 전 항상 인터럽트 여부 확인
3. **인터럽트 플래그**를 통해 현재 인터럽트를 받아들일 수 있는지 여부 확인
4. 받아들일 수 있으면 CPU는 지금까지의 작업을 백업
5. CPU는 인터럽트 벡터를 참조하여 **인터럽트 서비스 루틴**을 실행
6. 인터럽트 서비스 루틴이 끝나면 백업을 복구하여 실행재개

- `인터럽트 벡터` : 인터럽트 서비스 루틴의 시작 주소를 포함하는 인터럽트 서비스 루틴의 식별 정보
- `인터럽트 서비스 루틴` : 인터럽트를 처리하는 프로그램

![](https://velog.velcdn.com/images/dodo4723/post/0a15d5d4-e64f-4440-9969-74cf34492333/image.jpg)


<br>
<br>
<br>
<br>
<br>
<br>



## 코어
명령어를 실행하는 부품(ALU, 레지스터, 제어장치를 가지고 있음)

CPU는 단순히 명령어를 실행하는 부품에서 **명령어를 실행하는 부품을 여러 개 포함하는 부품**으로 확장됐습니다. 코어가 여러개면 **멀티코어 CPU**라고 부릅니다.

<br>

## 스레드
실행 흐름의 단위로 하드웨어적 스레드와 소프트웨어적 스레드로 나뉩니다.

#### 하드웨어적 스레드
- 하나의 코어가 동시에 처리하는 명령어 단위
- 2코어 4스레드는 한번에 네 개의 명령어를 처리가능 - 이런 CPU를 **멀티스레드 프로세서**라고 함
- 프로그램 입장에선 한번에 하나의 명령어를 처리하는 CPU가 4개 있는 것처럼 보임
- **논리 프로세서** 라고 부르기도 함

#### 소프트웨어적 스레드
- 하나의 프로그램에서 독립적으로 실행되는 단위

<br>

**1코어 1스레드 CPU도 소프트웨어적 스레드를 수십 개 실행할 수 있습니다.**

<br>
<br>
<br>

## 명령어 병렬 처리 기법

### 명령어 파이프라인
명령어 처리 과정을 클럭 단위로 나누면 다음과 같이 나눌 수 있습니다.
>
1. 명령어 인출
2. 명령어 해석
3. 명령어 실행
4. 결과 저장

![](https://velog.velcdn.com/images/dodo4723/post/781d0931-b2c3-4538-9496-a8ce5f372a93/image.jpg)


같은 단계가 겹치지만 않는다면 CPU는 각 단계를 동시에 실행할 수 있습니다. 이것을 **명령어 파이프라이닝** 이라고 합니다.

파이프라이닝은 특정 상황에서는 성능 향상에 실패하는 경우도 있습니다. 이러한 상황을 **파이프라인 위험** 이라고 합니다. 크게 `데이터 위험`, `제어 위험`, `구조적 위험`이 있습니다

저번 학기 시스템 프로그래밍 시간에 배운 내용들이 나왔습니다.

#### 1. 데이터 위험
>
R1 <- R2 + R3<br>
R4 <- R1 + R5

위의 경우 첫번째를 수행해야 두번째를 수행할 수 있습니다.

이처럼 데이터 의존적인 두 명령어를 무작정 동시에 실행하려고 하면 제데로 작동하지 않는 것을 **데이터 위험** 이라고 합니다.

#### 2. 제어 위험
프로그램 카운터의 갑작스러운 변화로 명령어 파이프라인에 미리 가지고 와서 처리 중이었던 명령어들이 쓸모가 없어지는 것을 **제어 위험**이라고 합니다.

#### 3. 구조적 위험
서로 다른 명령어가 동시에 CPU부품을 사용하려고 할 때 발생합니다. **자원 위험**이라고도 합니다.

파이프라인이 여러개인 구조를 **슈퍼스칼라** 라고 합니다.

<br>
<br>

### 비순차적 명령어 처리 기법
명령어를 순차적으로만 실행하지 않고 순서를 바꿔 실행해도 무방한 명령어를 먼저 실행하여 명령어 파이프라인이 멈추는 것을 방지하는 기법입니다.

<br>
<br>

### 명령어 집합(ISA)
- CPU가 이해할 수 있는 명령어들의 모음
- CPU마다 다를 수 있습니다.
- 크게 CISC와 RISC가 있습니다.

![](https://velog.velcdn.com/images/dodo4723/post/ae7b0bc4-39ed-448d-a678-37a7474f0e90/image.jpg)
