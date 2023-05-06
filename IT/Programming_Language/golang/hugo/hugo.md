# Hugo
- [Hugo](#hugo)
  - [1. 설치](#1-설치)
  - [2. 사용](#2-사용)
  - [3. 확인](#3-확인)
  - [4. 마치며](#4-마치며)

## 1. 설치
Hugo를 설치하기 위해서는 우선 Golang을 설치해야 한다.  
Golang은 Go의 [공식 홈페이지](https://go.dev/)에서 다운로드 받아 설치할 수 있다.  
Hugo는 Golang의 1.18버전 이상부터 지원하니 참고하도록 하자.  

Golang이 설치되었다면, 이제 GOROOT와 GOPATH를 설정해준다.  
GOROOT는 GO가 실제로 설치되는 위치이고, GOPATH는 GO를 통해 다운받는 각종 패키지들의 경로이다.  

GO에 대한 설정이 모두 완료되었다면, 이제 Hugo를 다운로드 받는다.  
Hugo는 GO를 통해 패키지를 다운로드 받거나, 직접 컴파일된 파일을 내려받아 사용하는 방법이 있다.  
후자가 더 간단하니 컴파일된 파일을 내려받아 사용하자.  
컴파일된 파일은 [Hugo의 깃허브](https://github.com/gohugoio/hugo)에서 다운로드 받을 수 있다.  
자신의 컴퓨터 OS와 CPU 아키텍처에 맞는 파일을 내려받자.  

내려받은 파일은 zip파일이거나 tar.gz파일일 것이다.  
파일을 내려받을 때에는 extended가 붙은 파일을 사용하는 것이 정신건강에 좋다.  
이 파일의 압축을 풀면 hugo 실행파일과 라이센스, 리드미 파일이 나온다.  
hugo 실행파일의 위치를 PATH에 추가해주면 사용을 위한 준비가 완료된다.  

## 2. 사용
Hugo를 사용하는 방법은 크게 어렵지 않다.  
많은 기능들을 내포하고 있지만, 그중에서 정작 Blog의 구축에 필요한 명령어는 몇개 없다.  
hugo를 구축하기 위해서는 우선 디렉토리를 하나 만들어야 한다.  
이 디렉토리는 하나의 blog를 구성하는 역할을 하게되며, hugo 엔진이 이 디렉토리의 내용들을 가지고 public 페이지의 html, js, css등 기본적인 구조를 만들어낸다.  
hugo에서는 이를 site라고 부르며, 아래의 명령어를 사용해서 생성한다.  

```bash
hugo new site [사이트이름]
```
```bash
[hello@localhost hugo]$ hugo new site test  ## 이름을 test로 지정했다.

"""
Congratulations! Your new Hugo site is created in /home/ehllo/test.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
"""

[hello@localhost hugo]$ ls -al
total 0
drwxr-xr-x.  3 kmj kmj  18 May  6 16:36 .
drwxr-xr-x.  3 kmj kmj  18 May  6 16:36 ..
drwxr-xr-x. 10 kmj kmj 141 May  6 16:36 test  ## test라는 이름의 디렉토리가 생성되었다.
```

이렇게 만들어진 디렉토리의 구조는 아래와 같다.  
우리가 주목할 디렉토리는 themes 디렉토리와 public 디렉토리이다.  
```bash
test
  |- archetypes
  |- assets
  |- config.toml ## 전반적인 설정을 진행하는 디렉토리
  |- content ## post를 저장하는 디렉토리 (post는 markdown 형식을 기본으로 한다)
  |- data
  |- layouts
  |- public ## 일반에 공개되는 디렉토리(index.html 파일이 위치)
  |- static
  |- themes ## 테마 관련 파일들이 위치하는 디렉토리
```

우선 먼저 진행해야 할 것은 themes 디렉토리에 대한 작업이다.  
hugo는 공식 홈페이지에서 여러 개발자들이 등록한 theme을 확인할 수 있게 제공하고 있으며, 각각의 theme은 저작권을 잘 확인해 사용해야 한다.  
대부분은 MIT 라이센스를 사용하기에 상업적인 사용에도 무리는 없다.  
마음에 드는 테마를 골라 Download 버튼을 클릭해보면 대부분 Github 페이지로 이동할 것이다.  
그러면 그 Github 페이지를 themes 디렉토리로 clone을 진행한다.  

대부분의 theme은 내부에 exampleSite 디렉토리를 가지고 있는데, 이는 개발자가 샘플로 올려둔 설정값이다.  
이 샘플을 사용해서 구성을 진행하는게 가장 정신건강에 이롭다.  
테마의 개발자마다 다른 설정값을 가지고 있으며, 테마에 기능이 많아지면 많아질 수록 설정은 복잡해지기 때문이다.  
심지어 아직까지는 개인 개발자들이 만들어내는 탓에 설명이나 문서화도 진행되지 않은 경우가 많아 역추적을 통해 변경해나가는 것이 가장 쉬운 방법이다.  
테마를 clone해왔고, exampleSite를 사용할 예정이라면, exampleSite 디렉토리의 모든 파일들을 themes 디렉토리가 위치한 레벨에 복사해준다.  

config.toml 파일을 사용해서 개인화를 진행해준다.  
테마에서 config 파일을 제공하고 있다면 아래의 항목들만 간단히 변경하여 사용할 수도 있다.
```toml
baseURL = 'http://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
theme = '내가 내려받은 테마'
```

theme과 config.toml에 대한 설정이 완료되었다면 이제 발행을 진행하면 된다.  
```bash
hugo -t '테마명'
```
theme 디렉토리에 내려받은 테마명을 그대로 사용하여 발행하면 된다.

```bash
Start building sites …
hugo v0.111.2-4164f8fef9d71f50ef3962897e319ab6219a1dad+extended linux/amd64 BuildDate=2023-03-05T12:32:20Z VendorInfo=gohugoio

                   | EN
-------------------+-----
  Pages            |  7
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  0
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Total in 134 ms
```
정상적으로 발행이 완료되면 이런 로그가 터미널에 뜨게된다.  
지금은 7개 페이지로 이루어진 사이트를 만들었다는 뜻이다.  

만약 여기에 추가로 post를 생성하고 싶다면 theme에서 제공하는 샘플을 변형해 사용하거나, 명령어를 통해 새로 생성해준다.  
```bash
hugo new posts/hello.md
```
이 명령어를 실행하면 content/posts/hello.md 파일이 생성된다.  
그리고 이 hello.md 파일의 내용은 다음과 같다.
```bash
---
title: "Hello"
date: 2023-05-06T17:02:17+09:00
draft: true ## 이 부분을 false로 바꿔줘야 블로그에 노출된다. 초안으로 작성중인지를 묻는 것이기에 최초 생성시에는 true로 설정되어있다.
---
```
이처럼 포스트를 위한 markdown 파일을 하나 만들고 내용을 작성하고 났다면, 발행을 다시 진행해준다.  

기본적인 사용 방법에 대한 설명이 모두 끝이났다.  

## 3. 확인
만들어진 블로그를 확인하고자 한다면, 아래의 명령어로 서버를 실행해주어야 한다.

```bash
hugo server ## 명령어는 config.toml 이 있는 위치에서 실행해주자.
```
실행이 완료되고 나면 아래와 같은 메시지가 뜨게된다.  
```bash
Start building sites …
hugo v0.111.2-4164f8fef9d71f50ef3962897e319ab6219a1dad+extended linux/amd64 BuildDate=2023-03-05T12:32:20Z VendorInfo=gohugoio
WARN 2023/05/06 17:07:38 found no layout file for "HTML" for kind "home": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2023/05/06 17:07:38 found no layout file for "HTML" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2023/05/06 17:07:38 found no layout file for "HTML" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.

                   | EN
-------------------+-----
  Pages            |  3
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  0
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Built in 9 ms
Watching for changes in /home/test/{archetypes,assets,content,data,layouts,static}
Watching for config changes in /home/test/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
이 메시지에 나와있는 것 처럼, 웹브라우저를 열고 http://localhost:1313으로 접속해보면 방금 만든 블로그를 확인할 수 있다.  
여기서 hugo의 엄청난 성능을 확인할 수 있는데, 만들어진 블로그 전체를 메모리에 올려두고 웹브라우저에게 서빙하는 방식을 사용하는데다 정적으로 구성된 통신없는 단순한 구조 덕분에 수정을 진행하면 브라우저에 바로바로 적용되는 모습을 볼 수 있다.  
단, OS의 방화벽이나 실행환경(docker, VM 등)에 따라 접속이 불가능할 수 있다.  
이런 경우에는 public 디렉토리를 git에 통째로 올리고, 이를 github page 기능을 통해 공개해 확인하는 방법도 있다.  

## 4. 마치며
hugo의 미래가 밝을지는 알 수 없다.  
워낙 다양한 웹프레임워크들이 엄청난 편의성을 가지고 시장에 나오고 있기 때문이다.  
하지만 GO를 활용한 프로젝트이며, GO를 통해 웹의 front까지 생성한다는 점에서는 굉장히 고무적인 프로젝트임에는 틀림없다.  
Go를 배워가는 예비 Goper에 입장에서는 도전해볼만 한 주제라고 생각해 시도하게 된 측면도 있다.  
hugo를 통해 여러가지 개인 프로젝트를 진행해보며 Goper에 한단계 더 가까이 다가설 수 있지 않을까?
** 참고로 hugo의 테마들 중에 중국쪽에서 만든 테마들이 많이 보이는 것을 확인할 수 있다. 그럴 가능성은 희박하겠지만, 혹시나 해당 테마들을 사용할 경우에 해킹의 위험에 대비하기 위해 민감한 내용을 담는 사이트를 만드는 용도로는 사용하지 않는 것을 추천한다.  