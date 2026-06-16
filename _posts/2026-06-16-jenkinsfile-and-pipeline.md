---
title: "Jenkinsfile과 Pipeline (빌드 단계를 코드로 정의하기)"
categories:
  - CI/CD
tags:
  - Jenkins
  - Pipeline
last_modified_at:
---

# Jenkinsfile과 Pipeline

Jenkins가 빌드·배포를 한다는데, "무슨 순서로 뭘 할지"는 어디에 적어두는지 궁금했음. 그게 Jenkinsfile이라고 해서 정리함


## 1. Jenkins한테 순서를 알려줘야 한다

Jenkins는 도구일 뿐이라, 알아서 다 해주진 않음. "이런 순서로 이런 걸 해라"를 사람이 적어줘야 함.

예를 들면:

```
1. 코드 받아오기
2. 빌드하기
3. 테스트하기
4. 도커 이미지 만들기
5. 배포하기
```

이 단계들을 적어두는 파일이 **Jenkinsfile**임.


## 2. Jenkinsfile은 프로젝트 안에 같이 둔다

Jenkinsfile은 보통 프로젝트 루트(코드랑 같은 곳)에 둠. 그래서 코드랑 배포 과정이 같이 버전 관리됨. "이 버전은 이렇게 배포했다"가 기록으로 남음.


## 3. Pipeline 기본 구조

Jenkinsfile은 Pipeline(파이프라인) 문법으로 씀. 핵심은 `stage`(단계)로 나누는 거임.

```groovy
pipeline {
    agent any                 // 아무 작업 공간에서나 실행

    stages {
        stage('빌드') {        // 1단계
            steps {
                sh 'npm run build'
            }
        }
        stage('테스트') {      // 2단계
            steps {
                sh 'npm test'
            }
        }
        stage('배포') {        // 3단계
            steps {
                sh 'docker compose up -d --build'
            }
        }
    }
}
```

- `stage('이름')` : 하나의 단계. 빌드, 테스트, 배포처럼 나눔.
- `steps` : 그 단계에서 실제로 실행할 명령어들.
- `sh '...'` : 셸 명령어를 실행하라는 뜻.


## 4. stage로 나누면 좋은 점

Jenkins 화면에서 단계별로 진행 상황이 보임.

```
[빌드 ✅] → [테스트 ✅] → [배포 ❌]
```

이렇게 어느 단계에서 실패했는지 한눈에 보임. 위 그림이면 "배포에서 터졌구나"를 바로 알 수 있음. 한 덩어리로 적었으면 어디서 깨졌는지 찾기 힘들었을 거임.


## 5. 정리

- Jenkinsfile = Jenkins한테 "무슨 순서로 뭘 할지" 알려주는 파일.
- Pipeline 문법으로 작성하고, `stage`로 단계를 나눔.
- 단계로 나누면 진행 상황과 실패 지점이 한눈에 보임.

> Webhook이 Jenkins를 깨우면, Jenkins는 이 Jenkinsfile에 적힌 stage 순서대로 빌드·배포를 진행함.
