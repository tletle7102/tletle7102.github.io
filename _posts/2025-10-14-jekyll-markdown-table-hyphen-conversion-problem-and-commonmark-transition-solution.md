---
title: "Jekyll Markdown 테이블 하이픈 변환 문제와 CommonMark 전환 해결"
categories:
  - Jekyll
tags:
  - Jekyll
  - Markdown
  - CommonMark
  - Kramdown
  - Table
last_modified_at:
---

### Jekyll에서 Markdown 테이블 하이픈 변환 문제 발생 원인과 해결  

Jekyll 블로그에서 Markdown으로 테이블을 작성할 때, 기본 파서인 Kramdown이 하이픈(-)을 자동으로 en-dash(–)나 em-dash(—)로 변환하여 테이블 구조가 깨지는 문제가 발생할 수 있음  
이로 인해 테이블 구분자(---)가 제대로 인식되지 않아 브라우저에서 테이블이 텍스트로 표시됨  
CommonMark 파서로 전환하면 이 문제를 근본적으로 해결할 수 있음  

---

#### 📌 용어 설명  
- Kramdown: Jekyll의 기본 Markdown 파서, SmartyPants 기능으로 특수 문자 자동 변환  
- SmartyPants: 텍스트를 타이포그래픽하게 변환하는 기능 (예: --- → —)  
- CommonMark: 대체 Markdown 파서, SMART 옵션 제어 가능하며 테이블 확장 지원  
- _config.yml: Jekyll 사이트 설정 파일, 파서와 플러그인 지정  
- Gemfile: Ruby gem 의존성 관리 파일, 버전 지정  

#### 📌 SmartyPants 미비활성화 시 발생 사항  
1. 기본 동작: Kramdown이 테이블 구분자(---)를 em-dash(—)로 변환  
대부분의 Markdown 에디터에서 미리보기 정상이나 빌드 후 깨짐  
2. 기능 제한: 테이블 셀 내부 --나 ---가 변환되어 내용 왜곡, 리플렉션 기반 도구 영향  
3. 경고 누락: 변환 관련 에러 표시되지 않아 디버깅 어려움  
4. 호환성 문제: GitHub Pages나 IntelliJ 빌드에서 일관성 없음  
5. Jekyll 영향: 포스트 렌더링 시 테이블 구조 인식 실패, HTML 테이블 대체 필요할 수 있음  

> 💡 Kramdown vs CommonMark  
> Kramdown은 Jekyll 기본으로 SmartyPants가 강제 적용되어 변환 발생  
> 쉽게 말하면, 텍스트를 "예쁘게" 만들려다 테이블이 깨짐  
> 
> 반대로 CommonMark은 SMART 옵션을 제거하면 변환 방지, 테이블 확장으로 구조 유지  
> 빌드 속도가 빠르고, Jekyll 플러그인과 호환성 좋음  

---

#### 📌 테이블 하이픈 변환과 Kramdown의 관련성

Kramdown의 SmartyPants 기능이 Markdown 테이블 구분자(---)를 em-dash(—)로 바꾸기 때문에 테이블이 인식되지 않음  
이로 인해 @RequestParam처럼 리플렉션 기반 바인딩이 아닌, 단순 텍스트 변환으로 테이블이 텍스트 블록으로 렌더링됨  

CommonMark으로 전환하면 SMART 옵션을 제거하여 하이픈이 그대로 유지되어 테이블 구조가 보존됨  
이는 CommonMark의 옵션 제어 기능 덕분으로, Kramdown처럼 강제 변환 없음  

---

#### 📌 설정 방법  
1. _config.yml 열기
2. 기존에 주석되어 있던 markdown: CommonMark 주석해제 처리 후 commonmark 섹션에 SMART 제거  
3. Gemfile에서 jekyll-commonmark과 commonmarker 버전 지정  
4. alembic-jekyll-theme.gemspec에서 명시적으로 고정되어 있던 jekyll-commonmark 버전 업그레이드  
5. 터미널에서 rm Gemfile.lock && bundle install 후 빌드  

#### 📌 예시 코드 (설정 파일 수정)  
##### _config.yml  
```yml  
markdown: CommonMark  
commonmark:  
  options: ["FOOTNOTES"]  
  extensions: ["strikethrough", "autolink", "table"]  
```  

##### Gemfile  
```plaintext  
gem "jekyll-commonmark", "~> 1.4.0"  
gem "commonmarker", "~> 0.23"  
```  

##### alembic-jekyll-theme.gemspec  
```gemspec  
spec.add_runtime_dependency "jekyll-commonmark", "~> 1.4.0"  
```  

---

#### 📌 추가 팁  
| 항목 | 설명 |  
| --- | --- |
| SMART 제거 | CommonMark에서 하이픈 변환 방지, 테이블 안정성 향상 |
| 버전 업그레이드 | 1.4.0으로 에러("uninitialized constant") 해결 |  