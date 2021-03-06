---
description: 컴퓨터 공학의 기초 과목인 컴퓨터구조론에서 'CPU의 구조와 기능'에 대해 알아봅니다
---

이 장에서는 CPU의 구조와 동작 원리에 대해 설명한다. CPU가 **프로그램 수행**을 하기 위해 수행해야 하는 세부적인 동작들을 보자:

1. 명령어 인출(Instruction Fetch): 기억장치로부터 **명령어를 읽어온다**.
2. 명령어 해독(Instruction Decode): **명령어를 해독**한다.
3. 데이터 인출(Data Fetch): 명령어 실행을 위해 **데이터가 필요한 경우, 기억장치 / I/O장치에서 데이터를 읽어온다.**
4. 데이터 처리(Data Process): 데이터에 대한 **산술적 / 논리적 연산을 수행**한다.
5. 데이터 저장(Data Store): 수행한 **결과를 저장**한다.

1, 2: 모든 명령어에 대해 **공통적으로 수행**.

3, 4, 5: 명령어에 따라 **필요한 경우에만 수행**.

## CPU의 기본 구조

### 산술논리연산장치(ALU)

각종 산술 연산 / 논리 연산을 수행하는 회로들로 구성.

### 레지스터 세트

- 액세스 속도 **가장 빠름**
- **내부 회로 복잡**해 큰 공간 차지 -> 많은 수의 레지스터를 CPU내에 포함시킬 수 없음.
- 따라서 **특수 목적용 레지스터들과 적은 수의 일반 목적용 레지스터들만이 포함**.

### 제어 유니트

- 프로그램 **코드(명령어) 해석**.
- 명령어를 실행하기 위한 **제어 신호들을 순차적으로 발생**하는 하드웨어 모듈.

### CPU 내부 버스

- **ALU와 레지스터 간 데이터 이동**을 위한 데이터 선, 주소 선.
- **제어 유니트에서 발생된 제어 신호를 전송하는 선**들로 구성.
- 외부의 시스템 버스와 **직접 연결되지 않음**. 반드시 **버퍼 레지스터**, **시스템 버스 인터페이스 회로**를 통해 접속.