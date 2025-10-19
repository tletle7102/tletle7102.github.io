---
title: "리눅스에서 포어그라운드와 백그라운드 프로세스 실행"
categories:
  - Linux
tags:
  - Linux
  - Shell
  - Foreground
  - Background
  - Process
last_modified_at:
---

### 리눅스에서 Foreground와 Background 프로세스 실행

리눅스에서 Foreground와 Background는 프로세스 실행 방식  
Foreground는 터미널에서 직접 제어, Background는 터미널 자유롭게 사용 가능  
포어그라운드는 직접 후라이팬으로 김치볶음밥 볶기, 백그라운드는 쿠키가 구워지도록 오븐 돌려놓기  

---

#### 📌 용어 설명  
- 포어그라운드: 터미널에서 실행 중인 프로세스, 사용자 입력(`Ctrl + C`)으로 제어 가능  
- 백그라운드: 터미널과 독립적으로 실행, 입력 제어 안 됨  
- 프로세스: 실행 중인 프로그램  
- PID: 프로세스 식별 번호  
- `jobs`: 현재 터미널의 백그라운드 작업 목록  

---

#### 📌 포어그라운드와 백그라운드 차이  

1. 포어그라운드 특징:  
   - 터미널이 프로세스에 묶임  
   - 실행 중 터미널에서 다른 명령어 입력 불가  
   - `Ctrl + C`로 종료 가능  
   - 비유: 팬으로 볶음밥 만들며 집중  

2. 백그라운드 특징:  
   - 터미널 즉시 자유로워짐  
   - `Ctrl + C`로 종료 불가  
   - `jobs`나 `ps`로 확인 후 `kill`로 종료  
   - 비유: 오븐에서 쿠키 굽기, 다른 작업 가능  

---

#### 📌 포어그라운드 실행 예시  

1. 쉘 스크립트 실행:  
   ```bash  
   ./script.sh
   ```  
   - 터미널에서 스크립트 실행, 출력 확인  
   - `Ctrl + C`로 중단 가능  
   - 예: 크롬 탭 여는 스크립트 실행, 터미널 대기  

2. Python 프로그램 실행:  
   ```bash  
   python3 app.py
   ```  
   - 웹 서버 실행, 터미널에 로그 출력  
   - 종료하려면 `Ctrl + C`  

3. 시스템 모니터링:  
   ```bash  
   top
   ```  
   - CPU 사용량 실시간 표시, `q`로 종료  

---

#### 📌 백그라운드 실행 예시  

1. 쉘 스크립트 백그라운드 실행:  
   ```bash  
   ./script.sh &
   ```  
   - `&`로 백그라운드 실행, 터미널 즉시 사용 가능  
   - 출력 예: `[1] 12345` (PID: 12345)  

2. Python 프로그램 백그라운드 실행:  
   ```bash  
   python3 app.py &
   ```  
   - 웹 서버 백그라운드 실행, 터미널 자유로움  

3. nohup으로 실행:  
   ```bash  
   nohup ./script.sh &
   ```  
   - 터미널 닫아도 실행 지속  
   - 출력은 `nohup.out`에 저장  

4. 포어그라운드에서 백그라운드로 전환:  
   ```bash  
   ./script.sh
   Ctrl + Z  # 일시 중지
   bg        # 백그라운드로 전환
   ```  

---

#### 📌 프로세스 관리 명령어  

1. 백그라운드 프로세스 확인:  
   ```bash  
   jobs
   ```  
   - 출력 예: `[1]+  Running  ./script.sh &`  

2. 백그라운드를 포어그라운드로:  
   ```bash  
   fg %1
   ```  
   - 작업 번호 `%1`을 포어그라운드로 가져옴, `Ctrl + C` 가능  

3. 프로세스 종료:  
   ```bash  
   ps aux | grep script.sh  # PID 확인
   kill -SIGTERM 12345     # 일반 종료
   kill -SIGKILL 12345     # 강제 종료
   ```  

---

#### 📌 실전 사용 예시  

1. 간단한 스크립트로 테스트:  
   ```bash  
   # test.sh
   for i in {1..5}
   do
       echo "Running $i..."
       sleep 1
   done
   ```  
   - 포어그라운드 실행:  
     ```bash  
     chmod +x test.sh
     ./test.sh
     ```  
     - 출력 확인, `Ctrl + C`로 종료  
   - 백그라운드 실행:  
     ```bash  
     ./test.sh &
     ```  
     - 터미널 자유, `jobs`로 확인, `kill %1`로 종료  

2. 크롬 탭 여는 스크립트:  
   ```bash  
   # open_tabs.sh
   for i in {1..3}
   do
       open -a "Google Chrome" https://example.com
       sleep 1
   done
   ```  
   - 포어그라운드: `./open_tabs.sh`, 터미널 대기  
   - 백그라운드: `./open_tabs.sh &`, 터미널 사용 가능  

---

#### 📌 팁  

- 비유 활용: 포어그라운드는 TV 시청, 백그라운드는 음악 재생  
- 간단 테스트: `test.sh`로 포어그라운드/백그라운드 실행 비교  
- 종료 방법: `Ctrl + C` 안 되면 `jobs` 확인 후 `kill` 사용  
- IntelliJ 터미널: IntelliJ 내 터미널에서도 동일 명령어 사용 가능  

---

#### 📌 주의사항  

- `Ctrl + C`는 포어그라운드에서만 작동  
- `nohup` 사용 시 `nohup.out` 확인  
- `SIGKILL`은 데이터 손실 위험, `SIGTERM` 먼저 시도  
- 터미널 포커스: 크롬 같은 앱이 포어그라운드면 터미널 입력 안 먹힐 수 있음, 터미널 클릭 후 입력