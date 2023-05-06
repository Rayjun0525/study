# Linux
리눅스 사전 보면서 공부하기.  

# A
1. access
```bash
## rockylinux 9.1 minimal에는 존재하지 않음
# 지정한 파일의 존재 유무와 권한 확인
access [모드] [파일명]
access rw testfile
echo $?
```
2. alias
```bash
# 복잡한 명령어와 옵션을 짧은 문자열로 변환
#alias만 사용시 현재 적용된 alias 확인 가능
alias [명칭]=[값]
```