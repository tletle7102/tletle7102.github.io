---
title: "iTerm + Oh My Zsh + Powerlevel10k 초기화부터 설치까지"
categories:
  - macOS
tags:
  - iTerm
  - zsh
  - Oh My Zsh
  - Powerlevel10k
last_modified_at:
---

### iTerm + Oh My Zsh + Powerlevel10k를 한 번에 깔끔하게 세팅하는 방법

macOS에서 터미널을 꾸미다 보면  
이미 Oh My Zsh가 설치되어 있거나  
예전에 시도하다 남은 설정 때문에 에러를 경험하는 경우가 있음
그래서 처음부터 순서대로 따라할 수 있게 문서로 정리

---

#### 📌 전체 진행 순서

1. 기존 설정 및 찌꺼기 초기화  
2. iTerm 설치  
3. Oh My Zsh 재설치  
4. Powerlevel10k 설치  
5. 테마 적용 및 설정 마법사 실행  

---

#### 📌 1. 설치 전 초기화 (가장 중요)

이미 Oh My Zsh나 zsh 설정이 있는 경우도 있기 때문에, 이 단계부터 진행

##### 1️⃣ 기존 Oh My Zsh 폴더 제거
```
rm -rf ~/.oh-my-zsh
```

##### 2️⃣ 기존 zsh 설정 백업
```
mv ~/.zshrc ~/.zshrc.backup
```

설정 파일이 없으면 에러 나도 무시해도 됨  
나중에 참고용으로 필요하면 백업 파일 확인 가능  

---

#### 📌 2. iTerm 설치

기본 터미널 대신 iTerm 사용 권장  

```
brew install --cask iterm2
```

설치 후 iTerm 실행  

---

#### 📌 3. Oh My Zsh 설치

초기화된 상태에서 새로 설치  

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

설치 중 질문:
- 기본 쉘 변경 → Yes  
- `.zshrc` 덮어쓰기 → Yes  

---

#### 📌 4. Powerlevel10k 설치

Oh My Zsh 설치 **이후에 반드시 실행**  

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
~/.oh-my-zsh/custom/themes/powerlevel10k
```

---

#### 📌 5. 테마 적용

`.zshrc` 파일 수정  

```
vim ~/.zshrc
```

아래 줄 찾기  
```
ZSH_THEME="robbyrussell"
```

아래처럼 변경  
```
ZSH_THEME="powerlevel10k/powerlevel10k"
```

저장 후 종료  
```
Esc → :wq → Enter
```

---

#### 📌 6. 적용 및 설정 마법사 실행

```
exec zsh
```

정상이라면  
Powerlevel10k 설정 마법사가 자동 실행됨  

---

#### 📌 verbose 옵션이란? (Powerlevel10k 설정에서 자주 보이는 개념)
터미널에서 자주 보이는 --verbose는 영어로 “말이 많은, 장황한(verbose)”에서 온 표현으로,
“자세하게, 상세하게 출력해 달라”는 의미로 쓰이는 옵션임.
​
##### 일반 의미

기본 모드: 중요한 결과만 간단히 출력

--verbose 모드: 중간 과정, 디버그 정보, 소요 시간 등 추가 정보까지 상세하게 출력

Powerlevel10k 설치/설정 마법사에서

“설정 과정을 더 자세히 볼래?”

“로그를 더 많이 보여줄까?” 같은 문맥에서 verbose 관련 문구가 등장할 수 있음

즉, 터미널에서 --verbose를 보게 되면
“결과만 간단히”가 아니라 “과정과 디테일까지 보고 싶다”는 옵션이라고 이해하면 됨

---

#### 📌 아이콘 깨질 때 필수 설정

Powerlevel10k는 Nerd Font 필요  

```
brew install --cask font-meslo-lg-nerd-font
```

iTerm 설정:
- Preferences  
- Profiles  
- Text  
- Font → MesloLGS NF 선택  

---

#### 📌 자주 발생하는 에러와 원인

- oh-my-zsh.sh not found  
  → Oh My Zsh 폴더가 깨진 상태  

- theme 'powerlevel10k/powerlevel10k' not found  
  → Oh My Zsh 삭제 후 테마 재설치 누락  

이 문서처럼 초기화부터 진행하면 대부분 발생하지 않음  

---

#### 📌 요약

- 설치 전 초기화가 가장 중요  
- Oh My Zsh → Powerlevel10k 순서 필수  
- 중간에 꼬였으면 다시 초기화가 가장 빠른 해결책  
--verbose는 “상세 출력 모드”라는 뜻으로, 설정/디버깅 시 유용하게 활용 가능  
- 설정 마법사로 원하는 스타일 쉽게 선택 가능  

터미널은 매일 쓰는 도구라  
한 번만 제대로 세팅해두면 만족도 높음  
