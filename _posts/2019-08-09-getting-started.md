---
title: Buffer OverFlow
author: 100zja
date: 2020-08-16
categories: [study]
tags: [bof]
pin: true
---


# What is BOF?

**Overflow** : '넘쳐 흐르다.'의 의미   
* 버퍼 - 지정된 크기의 메모리 공간, 연산 작업을 하는 데에 있어 필요한 데이터를 일시적으로 메모리 상에 저장하는 공간, 대부분 스텍에 저장    
====> 버퍼를 가득 채우다 못해 넘쳐흘러 버퍼 이후의 공간을 침범하는 버그의 일종, 프로그램 취약점    

* 특정 함수 사용 시 입력한 문자열을 저장하기 위해 선언한 변수 크기보다 길게 입력 시 영역 값 덮어씀   

* BOF 공격에 취약한 함수 : strcpy, gets, scanf, strcat, sprintf 등  
---> 처리하는 문자열의 크기를 지정하지 않음  

## **Memory structure**

메모리는 여러 개의 프로세스가 저장되어 병렬적으로 작업을 수행한다.   
운영 체재 - 하나의 프로세스 실행 시 segment 라는 단위로 묶는다.   

![memory structure](./images/메모리_구조.png)

* 하나의 세그먼트는 code segment, data segment, stack segment라고 함, 시스템에는 최대 16383개의 segment가 생성될 수 있다.   

* **code segment** : 시스템들이 알아들을 수 있는 명령어(instruction)가 들어있음, 기계어 코드로써 컴파일러가 만든 코드, 명령을 수행하면서 많은 분기 과정, 점프, 시스템 호출 등 수행  

  - 분기 과정, 점프 : 메모리 상의 특정 위치에 있는 명령을 지정해 주어야 함   

  - 세그먼트 : 현재 메모리상의 어느 위치에 저장될지 컴파일 과정에서 알 수 없음 -> 정확한 주소 지정 불가   
=====> logical address 사용 - 실제 메모리 상의 주소(physical address)와 매핑 됨 

* segment - segment selector에 의해서 자신의 시작 위치(offset)을 찾을 수 있음, 자신의 시작 위치로부터의 위치에 있는 명령 수행 여부 결정  

* **data segment** : 프로그램 실행 시 사용되는 데이터 들어감 ==> 전역 변수   

  - 프로그램 내애서 전역 변수 선언 -> data segment에 변수 자리잡음 -> 다시 네 개의 data segment 로 나뉨 : 현재 모듈의 data segment, 상위 레벨로부터의 데이터 모듈, 동적 생성 데이터, 다른 프로그램과 공유하는 공유 데이터로 나뉨  

* **stack segment** : 현재 수행되고 있는 handler, task, program이 저장하는 데이터 영역,   
buffer - stack segment에 있음

* **stack** :   
![stack](./images/stack.png)
  - 후입선출 구조 이용    

  - 제일 위의 데이터만 알 수 있다  

  - 그래도 이 때까지의 데이터 개수 알 수 있음    

  - 중간의 데이터는 절대 알 수 없음, 알고 싶다면 제일 위에 있을거라 추정되는 그 데이터의 자료까지 모조리 꺼내봐야 함    

  - 제일 처음 들어간 자료는 모든 자료를 꺼내기 전까지 알 수 없고, 제일 마지막에 들어간 자료는 바로 알 수 있음   
     
## **Register structure**

* **레지스터** : 프로세서에 위치한 고속 메모리, 극히 소량의 데이터나 처리 중인 중간 결과와도 같은 프로세서 -> 바로 사용할 수 있는 데이터를 담고 있는 영역   

  - cpu안에 존재, 연산 수행, 컴퓨터 안에서 가장 빠른 연산속도 가짐->자잘한 연산 도맡아서 함

  - 용량이 매우 작음, 레지스터에 프로그램 직접 받을 수 없음    

 * **범용 레지스터** :　논리, 수리 연산에 사용되는 피연산자, 주소를 계산하는 데 사용되는 피연산자, 메모리 포인터 저장 됨   

    - 프로그래머가 임의로 조작할 수 있게 허용되어 있는 레지스터, 32bit로 전환되면서 레지스터 이름의 앞에 E(Extended)가 붙음   

    - **EAX** : 피연산자와 연산 결과의 저장소, 누적 연산기, 곱셈과 나눗셈 연산에서 자동 사용   

    - **EBX** : 베이스 레지스터, DS segment 안의 데이터를 가리키는 포인터, 특정 주소 지정   

    - **ECX** : 문자열 처리, 수를 셈, 자동으로 루프 카운터(반복적 명령 수행시)   

    - **EDX** : 데이터 레지스터, I/O 포인터 (부호 확장 명령 등에 쓰이고 큰 수의 곱셈 또는 나눗셈 등의 연산이 이루어질 때 사용), 입출력 연산에서 반드시 간접 주소 지정에 사용   

   - **ESI** : 읽기 인덱스, DS 레지스터가 가리키는 data segment 내의 어느 데이터를 가리키는 포인터, 문자열 처리에서 source를 가리킴, 문자열 전송이나 비교에서 사용, 주로 소스 문자열 오프셋 가리킴   

    - **EDI** : 쓰기 인덱스, ES register가 가리키고 있는 data segment 내의 어느 데이터를 가리키는 포인터, 문자열 처리에서 destination 가리킴  

    - **ESP** : 스택 포인터, SS register가 가리키는 stack segment의 맨 꼭대기 가리키는 포인터 (스택의 가장 상단 주소 가짐), 현재까지 사용된 스택의 위치 저장, 스택 최상부 오프셋 가리킴   

    - **EBP** : 베이스 포인터, SS register가 가리키는 스택상의 한 데이터 가리키는 포인터(스택의 가장 밑바닥 주소 가짐), 스택의 데이터에 접근하기 위해 사용  

* **세그먼트 레지스터** : code segment, data segment, stack segment를 가리키는 주소가 들어있는 레지스터   

   - 특정 세그먼트를 가리키는 포인터 역할   

    - CS register : code segment   

    - DS, ES, FS, GS regester : data segment   

    - SS register : stack segment   
===> 세그먼트 레지스터가 가리키는 위치를 바탕으로 원하는 segment 안의 특정 데이터, 명령어들을 정확하게 열 수 있음   

* **인스트럭션 포인터** : 다음 수행해야 하는 명령(instruction)이 있는 메모리 상의 주소가 들어가 있는 레지스터    

* **플래그 레지스터** : 컨트롤 플래그 레지스터 - 상태 플래그, 컨트롤 플래그, 시스템 플래그들의 집합   
  - 1, 3, 5, 15, 22~31번 비트는 예약됨, 소프트웨어에 의해 조작 X   



## **Status Flags**

  - **CF** : carry flag, 연산을 수행하면서 carry or borrow 발생 시 1이 됨, carry, borrow는 덧셈 연산 시 bit bound를 넘어가거나 뺄셈을 하는 데 빌려오는 경우 말함  

  - **PF** : Parity flag, 연산 결과 최하위 바이트의 값이 짝수일 경우에 1이 됨, 패리티 체크를 하는 데에 사용   

  - **AF** : adjust flag, 연산 결과 carry or borrow가 3bit 이상 발생 시 1이 됨   

  - **ZF** : zero flag, 결과가 zero임을 가리킴, if문 같은 조건문 만족 시 set 됨   

  - **SF** : sign flag, 연산 결과 최상위 비트의 값과 같음, Signed 변수의 경우 양수면 0, 음수면 1   

  - **OF** : overflow flag, 정수형 결과값이 너무 큰 양수이거나 너무 작은 음수일 경우 피연산자 데이터 타입에 모두 들어가지 않을 경우 1이 됨   

  - **DF** : Direction flag, 문자열 처리에 있어서 1일 경우 -> 문자열 처리 instruction 자동 감소(문자열 처리 high address -> low address), 0일 경우 자동 증가    

## **System Flags**

![system flags](./images/system_flags.png )

  - **IF** : interrupt enable flag, 프로세서에게 mask한 interrupt에 응답할 수 있게 할 시 1

  - **TF** : Trap flag, 디버깅을 할 때 single-step을 가능하게 할 시 1

  - **IOPL** : I/O privilege level field, 현재 수행 중인 프로세스 혹은 task 권한 레벨 가리킴, 현재 수행 중인 프로세스 가리키는 CPL -> I/O address 영역 접근 위해서 I/O privilege level 보다 작거나 같아야 함

  - **NT** : Nested task interrupt의 chain 제어, 1->실행 task와 현재 task가 연결되어 있음 나타냄

  - **RF** : resume flag, exception debug 하기 위해 프로세서의 응답 제어

  - **VM** : virtual-8086 mode flag, virtual-8086 모드 사용하려면 1을 줌

  - **AC** : alignment check flag, 이 비트와 CR0 레지스터(운영 모드를 제어하는 레지스터)들의 AM 비트가 set 돼 있을시 메모리 레퍼런스의 alignment checking 가능

  - **VIF** : virtual interrupt flag, IF flag의 가상 이미지, VIP flag와 결합시켜 사용

  - **VIP** : virtual interrupt pending flag, 인터럽트가 pending 되었음을 가리킴

  - **ID** : identification flag, CPUID instruction을 지원하는 CPU인지를 나타냄


## **Instruction Pointer**

: 다음 실행할 명령어가 있는 현재 code segment의 offset 값을 가짐. 하나의 명령어 범위에서 선형 명령 집합의 다음 위치 가리킬 수 있음

  - JMP, Jcc, CALL, RET, IRET instruction이 있는 주소값 가짐   

* **EIP register** - 소프트웨어에 의해 바로 엑세스 할 수 없음, 특정 명령어(JMP, Jcc, CALL, RET)을 사용하거나 인터럽트, 예외를 발생 시켜야 함, 실행할 다음 명령어 주소 포함, 명령어 포인터 레지스터   

## **Assembly Language**

* 기계어(컴퓨터가 읽을 수 있는 2진수 언어)를 사람이 보기 쉽게 문자를 기호화 하여 나타낸 것   
  - 매우 쉽고 간결함   

![Assembly](./images/어셈블리어.png)

* **push** : push %eax - eax 값을 스택에 저장

* **pop** : pop %eax - 스택 가장 상위의 값을 꺼내서 eax에 저장  

* **mov** : mov %eax, %ebx - 메모리나 레지스터의 값을 옮길 때 사용  

* **lea** : leal(%esi), %ecx - %esi의 주소값을 %ecx에 옮긴다  

* **inc** : inc %eax - %eax의 값을 1 증가시킴  

* **dec** : dec %eax - %eax의 값을 1 감소시킴 

* **add** : add %eax, %ebx - 레지스터나 메모리의 값을 덧셈할 때 쓰임 

* **sub** : sub $0x8, %esp - 레지스터나 메모리의 값을 뺄셈할 때 쓰임  

* **call** : call proc - 프로시져를 호출함  

* **ret** : ret - 호출했던 바로 다음 지점으로 이동  

* **cmp** : cmp %eax, %ebx - 레지스터와 레지스터값을 비교  

* **jmp** : jmp proc - 특정한 곳으로 분기  

* **int** : int $0x80 - OS에 할당된 인터럽트 영역을 system call  

* **nop** : nop - 아무 동작도 하지 않음, 기계어 코드가 다른 코드와 섞이지 않게 함  

- 데이터 이동 : mov, lea  

- 논리, 연산 : add, sub, inc, dec  

- 흐름 제어 : jmp, cmp  

- 프로시져 : call, ret  

- 스택 조작 :　push pop  

- 인터럽트 : int  

프로시져 : 해결해야할 복잡한 문제들을 작은 부분들로 쪼개어 하나하나 풀어가는 것==> 프로그래밍이 한결 쉬워짐  

## **Prologue & Epilogue**

* **프롤로그** :  
함수 내에서 사용할 스택 프레임 설정, 함수가 시작될 때 stack pointer와 base pointer를 새로 지정하는 과정      
    - push %ebp  
	- mov %ebp, %esp  
====> ebp에 스택을 저장하고, esp를 ebp에 저장   

* **에필로그** :  
수행을 마치고 처음 호출한 지점으로 돌아가기 위해 스택 복원  
leave, ret

  - **leave** :   
	  mov %ebp, %esp   
	  pop %ebp  
	  ===> mov %ebp, %esp - esp레지스터에 ebp주소 넣기  
		pop %ebp - ebp를 스택 프레임에서 꺼내기 

  - **ret** :  
    pop %eip    
	  jmp %eip   
	  ===>  pop %eip - eip 레지스터를 스택에서 꺼냄  
		jmp %eip - 다음 명령의 주소로 점프  
===> 현재 esp를 ebp로 복구해주고 ret을 통해 다음 가야할 address로 점프  

## **Byte Order**

* **Big Endian** :  
바이트 순서가 낮은 메모리 주소에서 높은 메모리 주소로 정렬  
ex)74E3FF59(16) -> 74E3FF59  

* **Little Endian** :  
바이트 순서가 높은 메모리 주소에서 낮은 메모리 주소로 됨  
ex)74E3FF59(16) -> 59FFE374  
===> 수를 더하거나 빼는 셈을 할 때 낮은 메모리 주소 영역의 변화는 수의 크기 변화에서 더 적음 

## **Other**

* **return address** :   
3개의 인자를 스택에 넣은 뒤 function 호출, call이라는 명령어가 호출됨과 동시에 스택에는 IP가 쌓이는 것 ===>buffer overflow에서 가장 중요함   

* **Leave instruction** :   
함수 프롤로그 작업을 되돌리는 일, 함수 프롤로그
  - push %ebp  
  mov %esp, %ebp  
  프롤로그를 되돌리는 작업  
  - mov %ebp, %esp  
  pop %ebp  
===> Leave instruction 하나가 위의 두 가지 일을 한꺼번에 함  

* **shell code**  :   
쉘 - 터미널에 입력한 명령을 해석하고 관리하는 프로그램, 사용자와 리눅스 커널 사이에 중간자 역할  
쉘을 실행시키는 코드, 일반적으로 기계어 코드로 작성되어 있음  
만드는 이유 - 실행중인 프로세스에 어떠한 동작을 하도록 코드를 넣어 그 실행 흐름을 조작하기 때문에 실행 가능한 상태의 명령어를 만들어야함  

## **BOF method of attack**

* **고전적 방법** : 쉘 코드가 있는 곳의 address 추측 -> buffer 배열의 정확한 address 알 수 없기 때문, 몇 번의 시행착오를 거치면서 쉘이 떨어질 때까지 계속 공격 시도해야함   
==> 실행 확률 높이고 buffer를 채우기 위해 nop를 사용, 보통 0x90값 사용  
  - 노가다성이 짙고 힘들게 나와 지금은 거의 사용되지 않음, 훨씬 효과적이고 쉬운 방법이 많이 나왔기 때문

* **환경변수를 이용하는 방법** :  
환경변수 - 응용프로그램에서 참조하여 사용할 수 있음 --> 공격자는 환경 변수를 만든 뒤 이 안에 쉘 코드를 넣고 취약한 프로그램에 환경변수의 address를 retrun address에 넣음 --> 쉘 코드 실행  
===> overflow 되는 버퍼의 크기가 쉘 코드에 들어갈 만큼 넉넉하지 못할 경우에 매우 유용함  
  - **<u>대부분의 프로그램의 스텍 시작점은 같다</u>**  
  ===> 같은 쉘 환경에서 프로그램이 실행되면 main 함수에서 만나는 스텍 포인터는 같을 수 밖에 없을 것, 공격자는 스텍 포인터의 주소값을 알아내어 거기서부터 return address 유추  

* **Return Into Libc 기법** : 
스텍 영역의 코드를 실행하지 못하게 하는 non-executable stack 보호 기법이나 일부 IDS에서 네트워크를 통해 쉘 코드가 유입되는 것을 차단하는 보호 기법을 뚫기 위한 방법으로 제안됨  
리눅스의 메모리 보호 기법 중 하나인 NX bit를 우회하기 위해 사용되는 공격 기법  
  - NX bit :  
  stack segment의 execute 권한을 제한함으로써, stack에 쉘 코드를 저장하고 실행해 악용되는 것을 막기 위한 메모리 보호 기법  
  - **non-executable stack** :  
  스텍 영역에 있는 코드를 실행하지 못하게 하는 것, CPU 레벨에서의 보호 방법이기도 하고 운영체제 영역에서의 보호 방법이기도 함, EIP 레지스터에 stack segment 영역의 주소가 들어가면 에러 메시지 출력, 실행 종료 or 에러 메시지 없이 실행 멈춤  

  - **IDS** :  
  일부 IDS는 네트워크 인터페이스에서 수신되는 데이터에 쉘 코드가 포함되어 있거나 비정상적으로 많은 양의 NOP가 포함되어 있다면 침입으로 간주해 네트워크 연결을 종료함
===> 이러한 공격기법 등장으로 고전 기법 먹혀들지 X  

* **beist's execl 방법** :  
root의 권한을 얻어오기 위해서는 excel() 함수를 사용할 수 있음  
buffer overflow 취약점을 가진 프로그램을 return-into-libc 기법으로 overflow 시켜 공격  
==> non-executable stack 보호 기법을 회피하여 특정 명령을 수행할 수 있음, 스텍 영역에 쉘 코드를 집어넣지 않고 실행 가능, 버퍼 크기에 제약을 받지 않아 쉘 코드를 넣을 충분한 버퍼를 가지지 않은 취약 프로그램도 충분히 공격 가능함

* **Eggshell 기법** :  
  - **Eggshell** : bof에서 버퍼의 크기가 쉘코드의 크기보다 작아서 쉘코드를 써넣을 수 없을 때 환경변수에 쉘을 등록해둔 뒤 그 주소값만 가져와서 쉘을 띄울 수 있게 해주는 프로그램  
  - 환경변수에 등록 후 등록된 주소의 값을 불러오는 프로그램(getegg)을 작성해야 제대로 된 사용 가능 -> 코드 작성 후 한번 실행하여 "Egg"라는 이름으로 쉘코드를 환경 변수에 넣어줘야 프로그램으로 에그쉘 주소 가져오기 가능  
  - 에그쉘 소스를 바로 vi 편집기에 붙여넣으면 깨진 상태에서 붙여넣기가 되고 컴파일도 제대로 되지 않음 -> 붙여넣기 전에 :set paste 로 설정을 바꿔줘야 함  
  - **환경 변수** :  
  프로세스가 컴퓨터에서 동작하는 방식에 영향을 미치는, 동적인 값들의 모임, 쉘에서 정의되고 실행하는 동안 프로그램에 필요한 변수를 나타낸다  
    - **로컬 환경 변수** : 현재 진행중인 세션에서만 동작하는 환경 변수  
    - **사용자 환경 변수** : 특정 사용자에 대해서만 정의된 환경 변수, 로컬 터미널 세션, 원격 로그으니 세션을 사용하여 로그인할 때마다 로드 됨  
    - **시스템 전체 환경 변수** : 해당 시스템에 존재하는 모든 사용자가 사용할 수 있는 환경 변수  