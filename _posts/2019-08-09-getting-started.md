---
title: Buffer OverFlow
author: 100zja
date: 2020-08-16
categories: [study]
tags: [bof]
pin: true
---


## What is BOF?

**Overflow** : '넘쳐 흐르다.'의 의미   
* 버퍼 - 지정된 크기의 메모리 공간  
====> 버퍼를 가득 채우다 못해 넘쳐흘러 버퍼 이후의 공간을 침범하는 버그의 일종, 프로그램 취약점    

* 특정 함수 사용 시 입력한 문자열을 저장하기 위해 선언한 변수 크기보다 길게 입력 시 영역 값 덮어씀   

* BOF 공격에 취약한 함수 : strcpy, gets, scanf, strcat, sprintf 등  
---> 처리하는 문자열의 크기를 지정하지 않음  

## Memory structure

메모리는 여러 개의 프로세스가 저장되어 병렬적으로 작업을 수행한다.   
운영 체재 - 하나의 프로세스 실행 시 segment 라는 단위로 묶는다.   

![memory structure](/image/메모리_구조.png)

* 하나의 세그먼트는 code segment, data segment, stack segment라고 함, 시스템에는 최대 16383개의 segment가 생성될 수 있다.   
* **code segment** : 시스템들이 알아들을 수 있는 명령어 - instruction이 들어있음, 기계어 코드로써 컴파일러가 만든 코드, 명령을 수행하면서 많은 분기 과정, 점프, 시스템 호출 등 수행  
  - 분기 과정, 점프 : 메모리 상의 특정 위치에 있는 명령을 지정해 주어야 함   
  - 세그먼트 : 현재 메모리상의 어느 위치에 저장될지 컴파일 과정에서 알 수 없음 -> 정확한 주소 지정 불가   
=====> logical address 사용 - 실제 메모리 상의 주소(physical address)와 매핑 됨  
* segment - segment selector에 의해서 자신의 시작 위치(offset)을 찾을 수 있음, 자신의 시작 위치로부터의 위치에 있는 명령 수행 여부 결정  
* **data segment** : 프로그램 실행 시 사용되는 데이터 들어감 ==> 전역 변수   
  - 프로그램 내애서 전역 변수 선언 -> data segment에 변수 자리잡음 -> 다시 네 개의 data segment 로 나뉨 : 현재 모듈의 data segment, 상위 레벨로부터의 데이터 모듈, 동적 생성 데이터, 다른 프로그램과 공유하는 공유 데이터로 나뉨  
* **stack segment** : 현재 수행되고 있는 handler, task, program이 저장하는 데이터 영역,   
buffer - stack segment에 있음
* **stack** : 
![stack](/image/stack.png)
## Register structure



## Status Flags

Go to the root directory of the project and start initialization:

```console
$ bash tools/init.sh
```

> If you not intend to deploy it on GitHub Pages, append parameter option `--no-gh` at the end of the above command.

What it does is:

1. Remove some files or directories from your repository:
    - `.travis.yml`
    - files under `_posts`
    - folder `docs` 

2. If you use the `--no-gh` option, the directory `.github` will be deleted. Otherwise, setup the GitHub Action workflow by removing extension `.hook` of `.github/workflows/pages-deploy.yml.hook`, and then remove the other files and directories in folder `.github`. 

3. Automatically create a commit to save the changes.

## System Flags

Generally, go to `_config.yml` and configure the variables as needed. Some of them are typical options:
* `url`
* `avatar`
* `timezone`
* `theme_mode`


## Instruction Pointer

You may want to preview the site contents before publishing, so just run it by:

```terminal
$ bash tools/run.sh
```

Then open a browser and visit to <http://localhost:4000>.

Few days later, you may find that the file changes does not refresh in real time by using `run.sh`. Don't worry, the advanced option `-r` (or `--realtime`) will solve this problem, but it requires [**fswatch**](http://emcrisostomo.github.io/fswatch/) to be installed on your machine.

## Assembly Language

Before the deployment begins, checkout the file `_config.yml` and make sure the `url` is configured correctly. Furthermore, if you prefer the [_project site_](https://help.github.com/en/github/working-with-github-pages/about-github-pages#types-of-github-pages-sites) and don't use a custom domain, or you want to visit your website with a base url on a web server other than **GitHub Pages**, remember to change the `baseurl` to your project name that starting with a slash. For example, `/project`.

Assuming you have already gone through the [initialization](#initialization), you can now choose any of the following methods to deploy your website.

## Prologue & Epilogue

For security reasons, GitHub Pages build runs on `safe` mode, which restricts us from using tool scripts to generate additional page files. Therefore, we can use GitHub Actions to build the site, store the built site files on a new branch, and use that branch as the source of the Pages service.

1. Push any commit to `origin/master` to trigger the GitHub Actions workflow. Once the build is complete, a new remote branch called `gh-pages` will appear, which is used to store the built site files.

2. Unless you prefer to project sites, rename your repository to `<username>.github.io` on GitHub.

3. Choose branch `gh-pages` as the [publishing source](https://docs.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) for your GitHub Pages site.

4. Visit your website at the address indicated by GitHub.

