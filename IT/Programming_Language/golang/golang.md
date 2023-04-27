
## 기본구조

다음은 Golang 프로그래밍의 기본 구조입니다.

코드 조각

[Project Root]
  |-- [src]
    |-- [my_package]
      |-- [main.go]
      |-- [other_files.go]
    |-- [other_package]
      |-- [main.go]
      |-- [other_files.go]
  |-- [go.mod]
  |-- [go.sum]
  |-- [Makefile]
  |-- [other_files]

코드를 주의해서 사용하십시오. 더 알아보기
디렉토리 src에는 프로젝트의 모든 소스 코드가 포함되어 있습니다.
각 패키지는 src.
각 패키지의 파일 main.go은 해당 패키지의 진입점입니다.
패키지의 다른 파일은 함수, 유형, 변수, 상수 등이 될 수 있습니다.
파일 go.mod은 프로젝트의 Go 모듈입니다. 여기에는 모듈 이름, 버전 및 종속성이 포함됩니다.
이 go.sum파일은 프로젝트의 종속 항목에 대한 체크섬입니다. 종속성이 변경되지 않았는지 확인하는 데 사용됩니다.
Makefile빌드 자동화 파일입니다 . 프로젝트를 컴파일, 테스트 및 실행하는 데 사용할 수 있습니다.
프로젝트 루트 디렉터리의 다른 파일은 Go 컴파일러에서 무시됩니다.
다음은 파일, 모드 및 빌드 정렬 개념에 대한 자세한 설명입니다.

파일 정렬: Golang 프로그램은 패키지로 구성됩니다. 패키지는 함께 컴파일되는 소스 파일의 모음입니다. 패키지 main는 프로그램의 진입점입니다. 다른 모든 패키지는 패키지에서 사용할 수 있는 라이브러리입니다 main.
모듈: 모듈은 함께 릴리스되는 관련 Go 패키지 모음입니다. Go 리포지토리에는 일반적으로 리포지토리의 루트에 위치한 하나의 모듈만 포함됩니다. 파일 go.mod은 프로젝트의 Go 모듈입니다. 여기에는 모듈 이름, 버전 및 종속성이 포함됩니다.
빌드: 빌드는 Go 소스 코드를 실행 가능한 바이너리로 컴파일하고 연결하는 프로세스입니다. 이 go build명령은 Go 소스 코드를 컴파일하고 실행 가능한 바이너리로 연결합니다. 이 go test명령은 프로젝트에 대한 단위 테스트를 실행합니다. 이 go run명령은 Go 소스 코드를 컴파일하고 실행합니다.
