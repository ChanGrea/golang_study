# golang_study

## Install go language

1. [http://golang.org/dl](http://golang.org/dl)에서 OS에 해당하는 버전 다운로드
2. **/usr/local/go** 에 설치

```sh
tar -C /usr/local -xzf go{version}.linux-amd64.tar.gz
```

3. zsh에 go 실행경로 추가

```sh
sudo vi ~/.zshrc

# 맨 아래 추가
PATH=$PATH:/usr/local/go/bin
```

4. 설치 확인

```sh
go version
```
