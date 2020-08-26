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

![1](/images/구1.png) 

힌트가 있음을 확인하고 힌트를 읽는다 

![2](/images/구2.png)  
* char형 변수 buf, buf2가 10 크기의 배열로 각각 선언되고  
* 40만큼의 문자열을 fgets가 입력받아 buf에 저장한다  
* buf2에 저장된 문자가 go와 일치한다면 "Good skill!" 문구 출력과 3010의 level10 권한을 설정하고
* /bin/bash 쉘을 실행한다  

![3](/images/구3.png)  

gdb를 실행할 수 있는 권한이 있는 tmp로 이동한 뒤에 실행 파일을 생성한다

![4](/images/구4.png)  

gdb를 실행해서 메모리를 확인한다  
main+3에서 40바이트의 메모리가 할당된다  
main+43에서 eax에 ebp-40의 주소를 저장함 (buf)    
main+65에서 eax에 ebp-24의 주소를 저장함(buf2)  
===> buf에서 입력 시에 ebp-40을 넘겨 16바이트를 넘기면 ebp-24 공간을 넘어가 버퍼 오버플로우 발생가능  

![5](/images/구5.png)  
/usr/bin으로 이동해 bof 소스를 실행시켜 버퍼 오버플로우를 발생 시키고 level10의 권한을 얻어 my-pass로 확인한다