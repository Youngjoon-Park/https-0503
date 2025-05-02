
# 📦 Kiosk 백엔드 & 프론트엔드 배포 전체 README (실행 명령어 포함)

## 📁 로컬 프로젝트 기준 경로  
- 백엔드: `C:\kiosk-project\kiosk-backend`  
- 프론트엔드: `C:\kiosk-project\kiosk-frontend`  
- PEM 파일: `C:\kiosk-project\kiosk-backend\pem\LightsailDefaultKey-ap-northeast-2.pem`  
- 서버 대상 경로: `/home/ubuntu/kiosk-system/`  

---

## ✅ 백엔드 배포 순서 (Spring Boot JAR + PM2)

### 1. 백엔드 빌드

```bash
cd C:\kiosk-project\kiosk-backend
.\gradlew clean build
```

빌드 결과:

```bash
C:\kiosk-project\kiosk-backend\build\libs\kiosk-backend-0.0.1-SNAPSHOT.jar
```

### 2. 서버로 JAR 전송

```bash
scp -i /c/kiosk-project/kiosk-backend/pem/LightsailDefaultKey-ap-northeast-2.pem \
build/libs/kiosk-backend-0.0.1-SNAPSHOT.jar \
ubuntu@3.38.6.220:/home/ubuntu/kiosk-system/
```

### 3. 서버 SSH 접속

```bash
ssh -i /c/kiosk-project/kiosk-backend/pem/LightsailDefaultKey-ap-northeast-2.pem ubuntu@3.38.6.220
```

### 4. 백엔드 실행 (최초 실행 시)

```bash
cd /home/ubuntu/kiosk-system
pm2 start kiosk-backend-0.0.1-SNAPSHOT.jar --name kiosk-backend
```

### 5. 백엔드 재시작 (업데이트 시)

```bash
pm2 restart kiosk-backend --update-env
```

### 6. 실시간 로그 확인 (에러 추적 포함)

```bash
pm2 logs kiosk-backend --lines 100
```

※ `Ctrl + C`로 종료 가능  
※ 예: `NoClassDefFoundError`, `SpringApplication` 오류 추적 시 유용

---

## ✅ 프론트엔드(Vite) 배포 순서

### 1. Vite 빌드

```bash
cd C:\kiosk-project\kiosk-frontend
npm run build
```

결과 위치:

```bash
C:\kiosk-project\kiosk-frontend\dist
```

### 2. 빌드 파일 서버로 복사

```bash
scp -i /c/kiosk-project/kiosk-backend/pem/LightsailDefaultKey-ap-northeast-2.pem \
-r dist/* \
ubuntu@3.38.6.220:/home/ubuntu/kiosk-system/src/main/resources/static/
```

### 3. 백엔드 재시작 (정적 리소스 반영)

```bash
ssh -i /c/kiosk-project/kiosk-backend/pem/LightsailDefaultKey-ap-northeast-2.pem ubuntu@3.38.6.220
pm2 restart kiosk-backend --update-env
```

---

## 📄 자주 쓰는 명령어 모음

| 항목 | 명령어 |
|------|--------|
| 백엔드 빌드 | `.\gradlew clean build` |
| JAR 업로드 | `scp -i ... kiosk-backend-0.0.1-SNAPSHOT.jar ...` |
| 서버 접속 | `ssh -i ... ubuntu@3.38.6.220` |
| 백엔드 실행 | `pm2 start ... --name kiosk-backend` |
| 백엔드 재시작 | `pm2 restart kiosk-backend --update-env` |
| 로그 보기 | `pm2 logs kiosk-backend --lines 100` |
| 프론트 빌드 | `npm run build` |
| 프론트 업로드 | `scp -i ... -r dist/* .../static/` |
