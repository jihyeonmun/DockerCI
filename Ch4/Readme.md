# 1강 섹션 설명

# 2강 Node.js 앱 만들기

[nodejs](https://nodejs.org/fr/docs/guides/nodejs-docker-webapp/)
![1](1.png)
package.json(프로젝트의 정보와 프로젝트에서 사용 중인 패키지의 의존성을 관리하는 곳)
-> server.js (시작점(entry point)로서 가장 먼저 시작되는 파일)

1. package.json 만들기
   npm init

2. express 설치하기
   JavaScript와 jQuery의 관계철머 Node.js의 API를 단순화하고 새로운 기능을 추가해서 Node.js를 더 쉽고 유용하게 사용할 수 있게 해줍니다.

# 3강 DockerFile 작성하기

```
FROM node:10
RUN npm install
CMD ["node", "server.js"]

```

3. docker build ./

# 4강 Package.json 파일이 없다고 나오는 이유

package.json 파일이 컨테이너 안에 들어있지가 않다!
server.js도 동일함!

이러한 에러를 해결해야 함.!

![2](2.png)
![3](3.png)
![1](4.png)

1. npm install

- 어플리케이션에 필요한 종속성을 다운받아 줌.
- 이렇게 다운 받을 때 먼저 package.json을 보고 그곳에 명시된 종속성들을 다운받아서 설치해줌.
- 하지만 package.json 컨테이너 안에 없기에 찾을 수 없다는 에러가 발생하게 됨.

![1](6.png)
![1](7.png)
![1](8.png)
![1](9.png)
![1](10.png)

> 이러한 이유 때문에 COPY를 이용해서 Package.json을 컨테이너 안으로 넣어주어야 한다.

COPY package.json(로컬에 있는 이 파일을 복사할 파일 경로) ./ (도커 컨테이너의 지정된 장소에 복사해주기, 컨테이너내에서 파일이 복사될 경로)

![1](11.png)
![1](12.png)
![1](13.png)

### 다시 빌드하기

```
docker build -t moons94/nodejs ./
```

### 생성된 이미지 기반으로 실행하기

```
docker run moons94/node.js:lates ./
```

> 작동하지 않는다. 그 이유는 서버 파일을 찾지 못해서...

#### docker

```
FROM node:10
COPY ./ ./
RUN npm install
CMD ["node", "server.js"]

```

이제는 잘 작동되지만, 반대로 localhost:8080에서든 실행이 되지 않는 것을 알 수 있다. 다음강에서 확인!

# 5강 생성한 이미지로 어플리케이션 실행시 접근이 안 되는 이유

![1](14.png)
우리가 이미지를 만들 때 로컬에 있던 파일(package.json)등을 컨테이너에 복사해줘야 했었다.
그것과 비슷하게 네트워크도 로컬 네트워크에 있던 것을 컨테이너 내부에 있는 네트워크에 연결을 시켜주어야 한다.

> 포트 옵션을 설정해야 한다! (49160과 8080간에 연결이 되어야 한다. )

```
docker run -p 5000:8080 moons94/nodejs
```

localhost:5000에 들어가면 실행가능한 것을 확인할 수 있음.("Hello World")

![1](15.png)

5강 완료!

# 6강 Working Directory 명시해주기

![1](16.png)
우선 정의를 보면..
이미지 안에서 어플리케이션 소스 코드를 갖고 있을 디렉토리를 생성하는 것임.
그리고 이 디렉토리가 어플리케이션에 working 디렉토리가 된다.

그런데 왜 따로 working 디렉토리가 있어야 하나요?

```
docker run -it moons94/nodejs ls
```

> 다양하게 있다는 것을 확인할 수가 있다.

```
Dockerfile  dev   lib    mnt           package-lock.json  root  server.js  tmp
bin         etc   lib64  node_modules  package.json       run   srv        usr
boot        home  media  opt           proc               sbin  sys        var
```

> 여기 파일 등 중에서 노드 앱을 위해서 COPY로 이미지안으로 들어온 것들
> Dockerfile package.json package-lock.json node_module

이렇게 workdir를 지정하지 않고 그냥 COPY할 때 생기는 문제점

1. 혹시 이 중에서 원래 이미지에 있던 파일과 이름이 같다면?
   ex) 베이스 이미지에 이미 home이라는 폴더가 있고 COPY를 하므로서 새로 추가되는 폴더 중에 home이라는 폴더가 있다면 중복이 되므로 원래있던 폴더가 덮어씌어져 버린다.

2. 그리고 모든 파일이 한 디렉토리에 들어가버리니 너무 정리정돈이 안되어읶ㅆ다.

그래서 모든 어플리케이션을 위한 소스들은 WORK 디렉토리를 따로 만들어서 보관한다.

만드는 방법

![1](17.png)
![1](18.png)

```
docker build -t moons94/nodejs ./
```

> 새로운 dockerfile로 이미지 다시 생성후 컨테이너에서 실행
> docker build <dockerhub 아이디/앱이름>

> docker run -it <이미지 이름> sh
> 워크 디렉토리에 파일이 잘 들어가있는지 확인

> 명령어 ls 입력
> WORKDIR 설정후 터미널에 들어오면 기본적으로 work 디렉토리에서 시작하게 됨.

![1](19.png)

복사된 파일이 /app에 들어있는 것이고, 루트 경로에 있는 것을 확인할 수 있다.

# 7강 어플리케이션 소스 변경으로 재빌드시 효율적으로 하는법

> 어플리케이션을 만들다 보면 소스 코드를 계속 변경시켜줘야 하면 그에 따라서 변경된 부분을 확인하면서 개발을 해나가야 합니다. 그래서 도커를 이요해서 어떻게 실시간으로 소스가 반영되게 하는지 알아보겠음.

현재까지 만든 어플을 소스 변경시 변경 된 소스를 반영시켜주기 위해서는, 현재까지 도커 컨테이너로 어플을 실행한 순서를 보면...
![1](20.png)
소스코드를 계속 변경시켜 변경된 부분을 어플리케이션에 반영시키려면 표시된 이 구간 전체를 다시 실행시켜줘야 함.

> 소스를 바꿔도 화면은 반영이 되지 않고, 바뀐 소스 파일로 이미지를 재생성해야 하고, docker build <도커아디이>/앱 이름을 해야함.

매우 비효율적이다는 것을 알 수 있음.

docker run -d -p 5000:8080 <이미지 이름>

실제로 이러한 방식으로 할 떄 비효율적인 점들

- COPY ./ ./ 이 부분으로 인해서 소스를 변화시킨 부분은 server.js 뿐인데 모든 node_module에 있는 종속성들까지 다시 다운받아주어야 함.

- 그리고 소스하나 변경시켰을 뿐인데 이미지를 다시 생성하고 다시 컨테이너를 실행시켜주어야 함.

##### 그렇다면 이러한 비효율적인 부분을 어떻게 하면 해결할수 있을까?

# 8강 어플리케이션 소스 변경으로 재빌드 시 효율적으로 하는 법

![1](21.png)

![1](22.png)

### "COPY package.json ./"이 핵심이다.

1. 복사를 먼저해주고, npm install을 통해서 종속성을 해결하고,

2. 전체 코드를 복사한다.

#### 그 이유는 ?

npm install 할때 불 필요한 다운로드를 피하기 위해서임. 원래 모듈을 다시 받는 것은 모듈에 변화가 생겨야만 다시 받아야 하는데, 소스 코드에 조금의 변화만 생겨도 모듈 전체를 다시 받는 문제점이 있음.

-> npm install 을 하는 부분이 상당히 비효율을 야기한다. (종속성 다운은 한번이면 족함! )

> 오른쪽을 방식대로 해주게 되면 모듈은 모듈에 변화가 생길때만 다시 다운받아주며, 소스 코드에 변화가 생길때 모듈을 다시 받는 현상을 없애줄 수 있음.

아주 빠르다

# 9강 Docker Volume에 대하여

이제는 npm install 전에 package.json만 따로 변경을 해줘서 쓸떼없이 모듈을 다시 받지는 않아도 되게 됐습니다.

하지만 아직도 소스를 변경할때마다 변경된 소스 부분은 COPY한 후 이미지를 다시 빌드를 해주고 컨테이너를 다시 실행해줘야지 변경된 소스가 화면에 반영이 됨.
이러한 작업은 너무나 시간 소요가 크며 이미지도 너무나 많이 빌드하게 됨.
이러한 문제점을 해결하기 위해서 Volume을 사용하게 한다.

### Docker와 COPY 비교!

![1](23.png)
![1](24.png)

COPY는 이미지를 다 변경해줘야 했다!
Volume은 도커 컨테이너에서 볼륨을 이용해서 로컬에 있는 파일을 참조(Mapping)하게 한다.

![1](25.png)

노란 부분이 추가된 것을 확인할 수 있다.

PWD : print working directory
(현재 작업중인 디렉터리의 이름을 출력하는 데 쓰인다.)
![1](26.png)
nodemodule 없기 떄문에 -v 부분이 참조하지 말자는 것이다 !

> 이렇게 volume을 이용해서 키면 빌드할 떄 소스를 바꾸더라도 바꾼 코드가 적용이 된다.

> 이렇게 copy로만 만든 이미지를 사용해서 어플리케이션을 키면 소스 코드를 바꿔서 해도 바꾼 소스가 적용 되어 있지 않다.

```
docker run -d -p 5000:8080 -v /usr/src/app/node_modules -v $(pwd):/usr/src/app moons94/nodejs
```
