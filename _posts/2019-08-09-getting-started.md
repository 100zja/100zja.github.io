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
버퍼 - 지정된 크기의 메모리 공간  
====> 버퍼를 가득 채우다 못해 넘쳐흘러 버퍼 이후의 공간을 침범하는 버그의 일종, 프로그램 취약점    

특정 함수 사용 시 입력한 문자열을 저장하기 위해 선언한 변수 크기보다 길게 입력 시 영역 값 덮어씀   

BOF 공격에 취약한 함수 : strcpy, gets, scanf, strcat, sprintf 등  
---> 처리하는 문자열의 크기를 지정하지 않음  

## Memory structure

메모리는 여러 개의 프로세스가 저장되어 병렬적으로 작업을 수행한다.   
운영 체재 - 하나의 프로세스 실행 시 segment 라는 단위로 묶는다.   

![memory structure](data:image/png;base64,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.jpg "memory structurue")

Before running or building for the first time, please complete the installation of the Jekyll plugins. Go to the root directory of project and run:

```terminal
$ bundle install
```

`bundle` will automatically install all the dependencies specified by `Gemfile`.

What's more, in order to generate some extra files (*categories*, *tags* and *last modified list*), we need to use some tool scripts. If your machine is running Debian or macOS, make sure that [GNU coreutils](https://www.gnu.org/software/coreutils/) is installed. Otherwise, install by:

* Debian

  ```console
  $ sudo apt-get install coreutils
  ```

* macOS

  ```console
  $ brew install coreutils
  ```


## Register structure

Running [**Chirpy**](https://github.com/cotes2020/jekyll-theme-chirpy/) requires some extra files, which cannot be generated by Jekyll native commands, so please strictly follow the methods mentioned below to run or deploy your website.

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

