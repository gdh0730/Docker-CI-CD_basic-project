# Docker & CI/CD

---

## ⚙️ 기술 스택 및 버전
- **언어**: JavaScript (Node.js / React)
- **컨테이너**: Docker (Dockerfile, Docker Compose)
- **CI/CD**: GitHub Actions
- **환경**: Linux(Ubuntu), macOS, Windows에서 모두 동작 가능
- **클라우드(옵션)**: AWS EC2 또는 Docker 지원 호스팅 서비스

---

## 📂 디렉토리 구조
본 레포지토리는 강의 실습별로 분리된 예시 폴더 구조를 갖습니다:

```
.
├── dockerfile-folder
│   └── Dockerfile
├── nodejs-docker-app
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── docker-compose-app
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── docker-react-gh-app
    ├── Dockerfile
    ├── Dockerfile.dev
    ├── docker-compose.yml
    ├── src/
    └── .github/workflows/deploy.yaml
```

각 폴더는 독립적인 실습 예제며, Node.js 서버, React 프론트엔드, Docker Compose 구성, 그리고 GitHub Actions 연동 등의 주제를 포괄합니다.

---

## 1️⃣ Docker 환경 구성
### 왜 Docker를 사용하는가?
1. **환경 일관성**: 개발 환경과 배포 환경이 달라 발생하는 이슈 감소
2. **빠른 배포**: 애플리케이션 전체를 컨테이너로 패키징하여 배포 과정 단순화
3. **확장성**: 필요 시 손쉽게 컨테이너를 늘리거나 줄일 수 있음

### Dockerfile 핵심
- **Base Image**: 경량화를 위해 `node:alpine` 사용 (이미지 크기 최소화)
- **작업 디렉토리 설정**: `WORKDIR /app`을 지정해 코드 가독성 향상
- **의존성 설치**: `npm install` 실행 후 애플리케이션 소스 복사
- **다단계 빌드**: 빌드와 런타임 이미지를 분리하여 최종 이미지 크기/보안 최적화

```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .

# 빌드 단계 (React 등 빌드 필요한 경우)
RUN npm run build

# 배포 단계
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**사용 이유**: 빌드 환경과 실행 환경 분리를 통해 **보안**, **성능** 모두 확보.

---

## 2️⃣ Docker Compose로 다중 컨테이너 관리
### 왜 Compose를 사용하는가?
1. **개발 생산성 향상**: 여러 컨테이너(웹, DB, 캐시 등)를 단일 커맨드(`docker-compose up -d`)로 실행 가능
2. **확장 및 로드 밸런싱**: Scale 기능을 통해 컨테이너 개수를 쉽게 조절
3. **환경 변수 관리**: `.env` 파일을 통해 포트, 데이터베이스 URL 등 관리 간편

### docker-compose.yml 예시
```yaml
version: '3.7'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
  db:
    image: postgres
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

**사용 이유**: **애플리케이션 간 의존성을 명확히** 관리하고, **컨테이너를 코드로서 정의(IaC)** 하기 위해.

---

## 3️⃣ Node.js 서버 실습
### 왜 Node.js인가?
1. **빠른 I/O 처리**: 비동기 이벤트 기반 구조로 요청 처리에 유리
2. **방대한 생태계**: npm 라이브러리로 다양한 기능 손쉽게 구현
3. **경량화**: Docker와 결합 시 작은 메모리로도 웹 서버 구동 가능

```js
// server.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Dockerized Node.js Server!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

**사용 이유**: 간단한 REST API 서버 또는 웹 서버 구축에 적합하며, Docker로 컨테이너화 시 **핫 리로딩**(개발 모드)과 **경량화**가 쉬움.

---

## 4️⃣ React 애플리케이션 실습
### 왜 React인가?
1. **SPA 구조**: 단일 페이지 내에서 빠른 화면 전환
2. **컴포넌트화**: UI 재사용성과 유지보수성 높음
3. **생태계**: 다양한 서드파티 라이브러리를 통한 기능 확장 용이

```dockerfile
# Dockerfile.dev
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "start"]
```

**사용 이유**: `Dockerfile.dev`를 통해 **개발환경용 이미지**를 별도로 관리, 개발 시 코드 변경사항이 즉시 반영.

---

## 5️⃣ GitHub Actions 기반 CI/CD
### 파이프라인 동작 원리
1. **코드 푸시 시 이벤트 트리거**: main 브랜치에 변경이 생기면 워크플로우 실행
2. **자동 빌드 및 테스트**: Docker 환경에서 빌드 후 단위/통합 테스트 수행
3. **이미지 빌드 & 푸시**: 성공 시 Docker Hub 등에 빌드된 이미지를 업로드
4. **배포**(옵션): 별도의 서버나 클라우드 환경에 이미지 배포/롤아웃 자동화

```yaml
# .github/workflows/deploy.yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Build Docker Image
        run: docker build -t your-dockerhub-username/your-app:latest .
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      - name: Push Docker Image
        run: docker push your-dockerhub-username/your-app:latest

```

**사용 이유**:
- 자동화된 빌드/테스트/배포로 **개발 사이클 단축**
- GitHub Secrets를 활용한 **보안** 유지(Docker Hub 자격증명 등)

---

## 6️⃣ 실행 방법
1. **Docker 설치**: [Docker 공식 문서](https://docs.docker.com/get-docker/) 참고
2. **(옵션) Docker Compose 설치**: [Compose 설치 가이드](https://docs.docker.com/compose/install/)
3. **프로젝트 클론**: `git clone https://github.com/your-repo/docker-portfolio.git`
4. **폴더 진입**: 예) `cd nodejs-docker-app`
5. **이미지 빌드**: `docker build -t node-docker .`
6. **컨테이너 실행**: `docker run -p 3000:3000 node-docker`
7. **브라우저 열기**: `http://localhost:3000`에서 결과 확인

**Compose** 사용 시:
```bash
docker-compose up -d
```

---

## 7️⃣ 보안 및 성능 고려
- **이미지 최소화**: `alpine` 기반 이미지 사용으로 빌드 시간 단축 및 크기 절감
- **Secret 관리**: GitHub Actions Secrets, .env 파일 등 활용
- **테스트 자동화**: CI 단계에서 단위/통합 테스트를 실행해 코드 품질 보장
- **다단계 빌드**: 불필요한 빌드 종속성을 최종 이미지에 포함시키지 않음

---

## 8️⃣ 아키텍처 & 다이어그램
1. **개발** → Node.js & React 애플리케이션
2. **Dockerization** → Dockerfile(Docker Compose 포함)로 컨테이너화
3. **CI/CD** → GitHub Actions가 main 브랜치 푸시 시 빌드 & 테스트 수행, Docker Hub 배포
4. **배포** → 서버에서 `docker pull` 후 `docker run` or `docker-compose up`

**다이어그램 예시** (파일 참고):
```
개발자 → (코드 푸시) → GitHub → (CI/CD) → Docker Hub → (Pull & Deploy) → Production Server
```

---

## 9️⃣ 배운 점 및 결론
- **Docker의 가치**: 컨테이너화를 통해 어플리케이션 이식성과 확장성을 극대화
- **CI/CD의 중요성**: 자동화된 배포 파이프라인으로 **개발 속도**와 **품질** 모두 향상
- **DevOps 문화**: 개발과 운영을 긴밀히 연결하며, 반복 가능한 인프라를 코드로 관리

> **"개발자로서 Docker와 CI/CD는 필수 역량이며, 본 프로젝트를 통해 실무에서 활용 가능한 구조를 경험하였습니다."**

---

## 📝 참고 및 추가 개발 사항
- **테스트 프레임워크**(Jest, Mocha)와 결합해 CI 단계에서 자동화 테스트 강화
- **인프라 프로비저닝**(Terraform, Ansible 등)과 연동 가능
- **모니터링/로그 관리**(Prometheus, ELK 등) 추가로 운영 환경 안정성 확보
- **CDN 사용**(CloudFront, Netlify 등)으로 프론트엔드 자원 배포 최적화

---

## 🏆 최종 정리
이 저장소는 도커 컨테이너 환경 구축부터 CI/CD 파이프라인 운영까지 DevOps 핵심 개념을 종합적으로 다루고 있습니다. **어떤 기술을 사용했는지보다 "왜" 사용했는지**를 중점적으로 기술했으며, 이를 통해 **DevOps 개발자로서 한 단계 성장**할 수 있었습니다.

프로젝트를 직접 실행해보며 더 나은 구조나 자동화 방식이 있다면 자유롭게 Issue나 PR을 보내주세요! 😄

