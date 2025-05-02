
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

# 1단계: 빌드
cd C:/kiosk-project/kiosk-backend
./gradlew clean build

# 2단계: 서버에 jar 복사
scp -i C:/kiosk-project/kiosk-backend/pem/LightsailDefaultKey-ap-northeast-2.pem \
build/libs/kiosk-backend-0.0.1-SNAPSHOT.jar \
ubuntu@3.38.6.220:/home/ubuntu/kiosk-system/

# 3단계: PM2 재시작
ssh -i C:/kiosk-project/kiosk-backend/pem/LightsailDefaultKey-ap-northeast-2.pem ubuntu@3.38.6.220
pm2 delete kiosk-backend
pm2 start "java -jar /home/ubuntu/kiosk-system/kiosk-backend-0.0.1-SNAPSHOT.jar" --name kiosk-backend

# 4단계: 로그 확인
pm2 logs kiosk-backend

# 📘 13장: 실전 HTTPS 구축 및 인증서 발급 + 실습 리드미 (AWS 기준)

---

## 🧠 1. HTTPS란?

* HTTPS는 HTTP + SSL/TLS입니다.
* 사용자의 **브라우저와 서버 간의 데이터 통신을 암호화**하여 도청, 위조 방지를 목표로 합니다.
* 실무에서는 `https://` 접속이 필수가 되었고, 카카오페이/네이버페이 등 외부 API도 **HTTPS만 허용**합니다.

* 

---

## 🔐 2. SSL 인증서 발급 도구: Certbot과 Let's Encrypt

* Let's Encrypt는 무료로 SSL 인증서를 발급해주는 CA(인증기관)
* Certbot은 해당 인증서를 발급하고 자동으로 관리해주는 클라이언트
* 설치 후 단 한 줄로 HTTPS 인증서 발급 가능

```bash
sudo apt install certbot
sudo certbot certonly --standalone -d kiosktest.shop
```

---

## ⚙️ 3. 인증서 발급 후 생성되는 파일들

| 파일명             | 설명                         |
| --------------- | -------------------------- |
| `privkey.pem`   | 비밀 키. 절대 외부 노출 금지          |
| `cert.pem`      | 공개 인증서 (서버 도메인 인증)         |
| `chain.pem`     | 중간 인증기관 연결용 체인             |
| `fullchain.pem` | cert.pem + chain.pem 묶음 파일 |

---

## 🔑 4. EC2 접속용 PEM vs HTTPS 인증용 PEM 차이

| 구분            | 설명                                            |
| ------------- | --------------------------------------------- |
| EC2 접속용 PEM   | AWS Lightsail에서 수동 생성. `.pem` 확장자 하나          |
| HTTPS 인증용 PEM | certbot이 생성. `/etc/letsencrypt/live/도메인명/` 경로 |
| 목적 차이         | 하나는 SSH 접속용, 하나는 인증서용                         |

---

## 🧭 5. HTTPS 인증서 자동 갱신 설정

* 인증서는 **90일 유효기간**이므로 자동 갱신이 필수
* `crontab`에 아래 스케줄 등록

```bash
sudo crontab -e
```

```
0 4 * * * /usr/bin/certbot renew --quiet
```

> 매일 새벽 4시에 자동으로 인증서 갱신 시도

---

## 🧪 6. 실습용 PEM 설정 예시 (Nginx)

```nginx
server {
    listen 443 ssl;
    server_name kiosktest.shop;

    ssl_certificate     /etc/letsencrypt/live/kiosktest.shop/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/kiosktest.shop/privkey.pem;

    location / {
        proxy_pass http://localhost:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🧾 7. 실습 과정 요약: HTTPS 적용

### ✅ \[1단계] 인증서 발급 (certbot)

```bash
sudo apt update
sudo apt install certbot
sudo certbot certonly --standalone -d kiosktest.shop
```

### ✅ \[2단계] Nginx 설정에 적용

```bash
sudo vi /etc/nginx/sites-available/default
# 위의 ssl_certificate, ssl_certificate_key 설정 추가
```

### ✅ \[3단계] Nginx 재시작

```bash
sudo systemctl restart nginx
```

---

## 🚨 8. HTTPS 적용 후 발생 가능한 에러와 해결법

| 에러                      | 원인                            | 해결법                                        |
| ----------------------- | ----------------------------- | ------------------------------------------ |
| `NET::ERR_CERT_INVALID` | 도메인 불일치                       | 인증서 다시 발급 (`-d 도메인명` 확인)                   |
| QR코드 X                  | 백엔드가 http로 응답                 | Spring Boot에서 모든 `redirect:` URL https로 수정 |
| Post 요청 막힘              | Mixed content (http↔https 혼용) | axios 또는 API Base URL 모두 `https://`로 고정    |

---

## 💡 9. 실무 기준: HTTPS 배포 전체 흐름 정리

### ✅ 백엔드 빌드 및 배포

```bash
cd C:\kiosk-project\kiosk-backend
./gradlew clean build
```

```bash
scp -i ./pem/LightsailDefaultKey.pem build/libs/kiosk-backend-0.0.1-SNAPSHOT.jar \
ubuntu@3.38.6.220:/home/ubuntu/kiosk-system/
```

```bash
ssh -i ./pem/LightsailDefaultKey.pem ubuntu@3.38.6.220
pm2 restart kiosk-backend
```

### ✅ 에러 확인

```bash
pm2 logs kiosk-backend
# 또는
cat /home/ubuntu/.pm2/logs/kiosk-backend-error.log
```

---

## 🎨 10. 프론트엔드 HTTPS 기반 배포 시 주의사항

* `axiosInstance.js`

```js
const api = axios.create({
  baseURL: 'https://kiosktest.shop',
  withCredentials: true,
});
```

* QR코드 URL 생성 시 반드시 `https://` 기반 redirect URL을 사용해야 실제 모바일 결제가 됨

---

## 🔁 11. 프론트엔드 수정 및 반영 절차

```bash
cd C:\kiosk-project\kiosk-frontend
npm run build
scp -i ../kiosk-backend/pem/LightsailDefaultKey.pem -r dist/* \
ubuntu@3.38.6.220:/home/ubuntu/kiosk-system/src/main/resources/static/
```

---

## 🧭 12. 결제 실패, 로그인 실패 이유 분석 및 실무 대응

### ❌ 로그인 실패

* JWT 토큰 누락 → localStorage 확인 필요
* 서버에서 403 응답 → Security 설정 확인

### ❌ 결제 승인 실패

* pg\_token 전달 오류 → `/payment/success?pg_token=...` 확인
* 카카오페이 서버에서 리디렉션 실패 → approval\_url, fail\_url 올바른 도메인인지 점검

---

## 🔚 13. 정리 및 결론

* HTTPS는 실무에서 필수이며, 인증서는 무료로도 발급 가능
* AWS에서는 **SSH용 PEM**과 **HTTPS 인증서 PEM**을 명확히 구분해야 함
* 인증서 발급 후 Nginx 또는 Spring Boot에 반영, 자동 갱신도 crontab으로 필수 설정
* 실무에서는 발생 가능한 문제(토큰, http↔https 혼용 등)를 빠르게 로그 확인으로 해결하는 역량이 중요

---

**👉 이 장은 실제 배포, 인증서 적용, Nginx 연동까지 모두 포함되며 실습과 개념을 통합한 가장 핵심 실무 단원입니다.**

