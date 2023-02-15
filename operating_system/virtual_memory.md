![](https://velog.velcdn.com/images/dodo4723/post/088063b5-bc9b-4de9-b6fd-32ab79c4ca26/image.png)

개인적으로 운영체제에서 제일 중요하다고 생각하는 부분입니다.

### 스와핑
메모리에 적재된 프로세스들 중, 현재 실행되지 않는 프로세스가 있을 수 있는데, 이들을 임시로 보조기억장치로 쫒아내고, 빈 공간에 또 다른 프로세스를 적재하는 방식

>
- `스왑 영역` : 프로세스들이 쫒겨나서 가는 보조기억장치의 일부 영역
- `스왑 아웃` : 실행되지 않는 프로세스가 메모리에서 스왑 영역으로 옮겨지는 것
- `스왑 인` : 스왑 영역에 있던 프로세스가 다시 메모리오 옮겨오는 것

<br>
<br>

### 메모리 할당

#### 메모리 공간에 프로세스를 연속적으로 할당하는 방식

>
- `최초 적합` : 메모리를 순서대로 검색하다가 적재할 수 있는 공간을 발견하면 그 공간에 배치
- `최적 적합` : 빈 공간을 모두 탐색 후, 가능한 가장 작은 공간에 배치
- `최악 적합` : 빈 공간을 모두 탐색 후, 가능한 가장 큰 공간에 배치

프로세스들이 메모리에 연속적으로 할당되는 환경에서 실행과 종료를 반복하며 메모리 사이에 빈 공간들이 생기는 현상을 **외부 단편화** 라고 합니다.

이를 해결하기 위해 **흩어져 있는 빈 공간을 하나로 압축**할 수도 있는데, 많은 오버헤드를 야기합니다.

<br>
<br>
<br>

# 가상 메모리
실행하고자 하는 프로그램을 일부만 메모리에 적재하여 실제 물리 메모리 크기보다 더 큰 프로세스를 실행할 수 있게 하는 기술

크게 **페이징**과 **세그멘테이션** 기법이 있습니다.

<br>

## 페이징 기법
프로세스의 논리 주소 공간을 **페이지**라는 일정한 단위로 자르고, 메모리 물리 주소 공간을 **프레임**이라는 페이지와 동일한 크기의 일정한 단위로 자른 뒤 페이지를 프레임에 할당하는 기법.

![](https://velog.velcdn.com/images/dodo4723/post/59e912bf-1a52-436b-975f-ee356a9b0f24/image.jpg)


외부 단편화가 발생하지 않지만, **내부 단편화**가 발생할 수 있습니다.

<br>
<br>

### 페이지 테이블
프로세스가 **물리 주소(실제 메모리 내의 주소)**에 불연속적으로 배치되더라도** 논리 주소(CPU가 바라보는 주소)**에는 연속적으로 배치되도록 **페이지 테이블**을 이용

![](https://velog.velcdn.com/images/dodo4723/post/7e3c5c92-c70c-4128-aef5-39a13804146a/image.jpg)


프로세스마다 메모리에 각자의 프로세스 테이블을 가지고 있고, CPU 내의 페이지 **테이블 베이스 레지스터(FTBR)**는 각 프로세스 페이지 테이블이 적재된 주소를 가리키고 있어 CPU가 프레임을 알 수 있음

메모리 접근을 2번 하는 문제 때문에 CPU 곁에(MMU 내에) **TLB(Translation Lookaside Buffer)**라는 페이지 테이블의 캐시 메모리를 둡니다.

<br>
<br>

### 페이징에서의 주소 변환

기본적으로 모든 논리 주소가 **페이지 번호**와 **변위**로 이루어져 있음

예) <5,2>에 접근 -> 5번 페이지가 1번 프레임에 있고 1번 프레임이 8번지부터 시작하면 10(8+2)번지 접근

<br>
<br>

### 페이지 테이블 엔트리

페이지 번호, 프레임 번호 외에도 다른 정보들도 있음

>
- `유효 비트` : 페이지가 메모리에 있으면 1, 보조기억장치면 0 - 0에 접근하려고 하면 **페이지 폴트** 예외 발생
- `보호 비트` : r(읽기), w(쓰기), x(실행) 권한을 나타냄
- `참조 비트` : 적재 이후 읽거나 쓴 적 있는 페이지는 1 아니면 0
- `수정 비트` : 변경된 적 있는 페이지는 1 아니면 0 - 페이지 아웃될 때 보조기억장치에 쓰기 작업 여부 판단

<br>
<br>
<br>
<br>

## 페이지 교체 알고리즘

**요구 페이징**은 프로세스를 메모리에 적재할 때 필요한 페이지만을 메모리에 적재하는 기법입니다.

요구된 페이지가 현재 메모리에 없을 경우(유효 비트가 1일경우) 페이지 폴트가 발생합니다. CPU는 해당 페이지를 메모리로 적재하고 유효 비트를 1로 설정합니다.

요구 페이징 시스템이 안정적으로 작동하려면 **페이지 교체**와 **프레임 할당** 문제를 해결해야 합니다.

<br>

### 1. FIFO 페이지 교체 알고리즘
- 메모리에 가장 먼저 올라온 페이지부터 내쫒는 방식

- 아이디어와 구현이 간단하지만, 프로그램 실행 내내 사용될 페이지를 내쫒을 수도 있음

<br>

### 2. 최적 페이지 교체 알고리즘
- 앞으로의 사용 빈도가 가장 낮은 페이지를 교체하는 알고리즘으로 가장 낮은 페이지 폴트율을 보장함

- 예측이 어려워 구현이 어려움. 주로 다른 페이지 교체 알고리즘의 이론상 성능을 평가하기 위한 목적으로 사용

<br>

### 3. LRU 페이지 교체 알고리즘
- Least Recently Used - 가장 오랫동안 사용되지 않은 페이지를 교체

<br>
<br>
<br>
<br>
<br>

## 스래싱과 프레임 할당
프로세스가 사용할 수 있는 프레임 수가 적어도 페이지 폴트는 자주 발생합니다. 프로세스가 실제 실행되는 시간보다 페이징에 더 많은 시간을 소요하여 성능이 저해되는 문제를 **스래싱**이라고 합니다.

스래싱이 발생하는 근본적인 원인은 각 프로세스가 필요로 하는 최소한의 프레임 수가 보장되지 않기 때문입니다. **운영체제는 프로세스들이 무리 없이 실행하기 위한 최소한의 프레임 수를 파악, 할당해줄 수 있어야 합니다.**

<br>


### 프레임 할당 방식

#### 정적 할당 방식

>
- `균등 할당` : 모든 프로세스에 균등하게 프레임을 제공
- `비례 할당` : 프로세스의 크기에 비례해 할당

<br>

#### 동적 할당 방식 - 프로세스의 실행을 보고 할당 프레임 수를 결정

>
- `작업 집합 모델` : 프로세스가 일정 기간 참조한 페이지 집합을 참고 - 3초에 7개의 페이지를 집중적으로 참조했다면 그 순간에 7개 할당
- `페이지 폴트 빈도` : 페이지 폴트율이 높으면 너무 적은 프레임을 갖고있기 때문에 더 할당하고 폴트율이 낮으면 그 반대

![](https://velog.velcdn.com/images/dodo4723/post/dfe0418c-bc72-4b4a-b5a4-42055dd8db8a/image.png)