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

![1](15.png)
![1](16.png)
![1](17.png)
![1](18.png)
![1](19.png)
![1](20.png)
![1](21.png)
![1](22.png)
![1](23.png)
![1](24.png)
![1](25.png)
![1](26.png)

# 6강 Working Directory 명시해주기

# 7강 어플리케이션 소스 변경으로 재빌드시 효율적으로 하는법

# 8강 Docker Volume에 대하여
