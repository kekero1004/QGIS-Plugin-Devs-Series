# QGIS Plugin 개발 교재 발간 사전조사
## 웹사이트 · 공식 문서 · PDF 다운로드 · 개발도구 · 출판/라이선스 소스맵

- 조사 기준일: **2026-08-27**
- 권장 교재 기준 버전: **QGIS 4.2.x + Qt 6**
- 호환성 보조 기준: **QGIS 3.44 LTR + Qt 5**
- 권장 교재 성격: **실무형 PyQGIS 플러그인 개발 + 엔지니어링 자동화 프로젝트 기반**
- 주요 언어: Python / PyQGIS / `qgis.PyQt` / Qt Designer / Processing Framework
- 대상 독자:
  - GIS·토목·측량·수자원·BIM 엔지니어
  - QGIS 사용 경험이 있고 업무자동화를 시작하려는 실무자
  - Python 초중급 개발자
  - 사내 GIS 도구를 QGIS Plugin으로 패키징하려는 개발자

---

# 1. 조사 요약

## 1.1 현재 QGIS 버전 기준

2026-08-27 기준 QGIS 공식 다운로드 페이지에서 확인되는 현황은 다음과 같다.

| 구분 | 버전 | 비고 |
|---|---:|---|
| 최신 안정 버전 | **QGIS 4.2.1 “Belém do Pará”** | 2026-07-31 릴리스 |
| 현재 공식 LTR | **QGIS 3.44.13 “Solothurn”** | 안정성 중심 |
| 개발 브랜치 | QGIS 4.3 master | 차기 기능 개발 |
| QGIS 4.0 이후 | **Qt 6 전용** | 플러그인 개발 시 핵심 변화 |

공식 다운로드:
- https://www.qgis.org/download/
- https://qgis.org/resources/installation-guide/
- https://www.qgis.org/resources/roadmap/

### 교재 발간 권장 기준

**1판의 본문은 QGIS 4.2 / Qt6 기준으로 작성하고, 각 장 하단에 “QGIS 3.44 LTR 호환성” 박스를 두는 방식을 권장한다.**

이유:

1. QGIS 4.x는 Qt6 기반으로 전환되었다.
2. 신규 플러그인은 QGIS 4 대응이 사실상 필수이다.
3. 기업·공공기관 현장에는 당분간 QGIS 3.44 LTR 사용자가 공존한다.
4. QGIS 4.2는 4.x 생태계의 안정화 기준점으로 활용하기 좋다.
5. 2026년 하반기 출판을 목표로 할 경우 QGIS 4.4가 나와도 4.2 기반 API 설명은 상당 부분 유지될 가능성이 높다.

---

# 2. 가장 중요한 공식 PDF — QGIS 4.2

QGIS 4.2 PDF 디렉터리:
- https://docs.qgis.org/4.2/pdf/
- https://docs.qgis.org/4.2/pdf/en/

현재 4.2 PDF 디렉터리에는 영문 PDF가 제공된다.

## 2.1 필수 1순위

| 우선순위 | PDF | 용도 | 직접 다운로드 |
|---|---|---|---|
| ★★★★★ | **PyQGIS 4.2 Developer Cookbook** | 플러그인·PyQGIS API·Processing·QgsTask 핵심 | https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-PyQGISDeveloperCookbook-en.pdf |
| ★★★★★ | **QGIS 4.2 Training Manual** | 교육 순서·실습 설계 참고 | https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-TrainingManual-en.pdf |
| ★★★★☆ | **QGIS 4.2 Documentation Guidelines** | 교재 문서 구조·코드 스니펫·그림·Sphinx 작성 참고 | https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DocumentationGuidelines-en.pdf |
| ★★★★☆ | **QGIS 4.2 Desktop User Guide** | UI·Processing·데이터·Plugin 사용자 관점 | https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DesktopUserGuide-en.pdf |
| ★★★☆☆ | **QGIS 4.2 Server User Guide** | QGIS Server Plugin 확장편 | https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-ServerUserGuide-en.pdf |

### PyQGIS 4.2 Developer Cookbook의 교재 활용 핵심 장

공식 PDF는 약 **176페이지**이며 플러그인 교재의 가장 중요한 1차 자료이다.

핵심 분야:

- Python Console
- Python Plugins
- Processing Plugins
- 프로젝트와 레이어
- Raster / Vector API
- Geometry
- CRS / 좌표변환
- Map Canvas
- Map Tool
- Rendering
- Expressions
- Settings
- 사용자 메시지와 로그
- Authentication
- **QgsTask 기반 Background Task**
- **Developing Python Plugins**
- **Writing a Processing Plugin**
- Plugin Layer
- Network Analysis
- QGIS Server + Python
- PyQGIS Cheat Sheet

온라인 버전:
- https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/

플러그인 장:
- https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/plugins/index.html

Processing Plugin:
- https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/processing.html

---

# 3. 한글 공식 PDF — QGIS 3.44 LTR

QGIS 3.44 한국어 PyQGIS 문서는 현재 한글 번역이 매우 충실하여 **한국어 교재 용어 통일**에 특히 유용하다.

한국어 PDF 인덱스:
- https://docs.qgis.org/3.44/pdf/ko/

## 3.1 한국어 필수 자료

| 우선순위 | PDF | 직접 다운로드 |
|---|---|---|
| ★★★★★ | **PyQGIS 3.44 개발자 쿡북 한국어** | https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-PyQGISDeveloperCookbook-ko.pdf |
| ★★★★★ | **QGIS 3.44 교육 교재 한국어** | https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-TrainingManual-ko.pdf |
| ★★★★☆ | **QGIS 3.44 Documentation Guidelines 한국어** | https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-DocumentationGuidelines-ko.pdf |
| ★★★★☆ | **QGIS 3.44 Desktop User Guide 한국어** | https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-DesktopUserGuide-ko.pdf |
| ★★★☆☆ | **QGIS 3.44 Server User Guide 한국어** | https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-ServerUserGuide-ko.pdf |
| ★★☆☆☆ | A Gentle Introduction to GIS 한국어 | https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-GentleGISIntroduction-ko.pdf |

한국어 온라인 PyQGIS:
- https://docs.qgis.org/3.44/ko/docs/pyqgis_developer_cookbook/index.html

한국어 개발자 가이드:
- https://docs.qgis.org/3.44/ko/docs/developers_guide/index.html

### 활용 방법

QGIS 4.2 영문 문서에서 최신 API를 확인하고,
QGIS 3.44 한국어 문서에서 아래 용어의 한국어 번역을 확인하는 방식이 좋다.

예:

- feature → 피처
- layer → 레이어
- geometry → 도형/지오메트리
- coordinate reference system → 좌표계
- processing algorithm → 공간 처리 알고리즘
- provider → 제공자/프로바이더
- map canvas → 맵 캔버스
- task → 태스크
- signal / slot → 시그널 / 슬롯

단, 상용 교재에서는 기계적으로 공식 번역을 그대로 따르기보다
**영문 API 클래스명 + 실무 한국어 용어** 병기를 권장한다.

예:

> `QgsVectorLayer` — 벡터 레이어 객체  
> `QgsFeature` — 공간 객체(피처)  
> `QgsGeometry` — 지오메트리 객체

---

# 4. QGIS 3.44 영문 PDF

영문 원문 대조 및 3.x/4.x 차이 비교용으로 보관한다.

인덱스:
- https://docs.qgis.org/3.44/pdf/en/

직접 다운로드:

- PyQGIS Developer Cookbook  
  https://docs.qgis.org/3.44/pdf/en/QGIS-3.44-PyQGISDeveloperCookbook-en.pdf

- Training Manual  
  https://docs.qgis.org/3.44/pdf/en/QGIS-3.44-TrainingManual-en.pdf

- Documentation Guidelines  
  https://docs.qgis.org/3.44/pdf/en/QGIS-3.44-DocumentationGuidelines-en.pdf

- Desktop User Guide  
  https://docs.qgis.org/3.44/pdf/en/QGIS-3.44-DesktopUserGuide-en.pdf

- Server User Guide  
  https://docs.qgis.org/3.44/pdf/en/QGIS-3.44-ServerUserGuide-en.pdf

---

# 5. QGIS Testing 문서 — 교재 개정판 추적용

출판 기간이 3~6개월 이상이라면 반드시 QGIS testing 문서를 함께 추적해야 한다.

PDF 인덱스:
- https://docs.qgis.org/testing/pdf/en/

핵심 PDF:

- PyQGIS Testing Developer Cookbook  
  https://docs.qgis.org/testing/pdf/en/QGIS-testing-PyQGISDeveloperCookbook-en.pdf

- Documentation Guidelines  
  https://docs.qgis.org/testing/pdf/en/QGIS-testing-DocumentationGuidelines-en.pdf

- Desktop User Guide  
  https://docs.qgis.org/testing/pdf/en/QGIS-testing-DesktopUserGuide-en.pdf

- Training Manual  
  https://docs.qgis.org/testing/pdf/en/QGIS-testing-TrainingManual-en.pdf

- Server User Guide  
  https://docs.qgis.org/testing/pdf/en/QGIS-testing-ServerUserGuide-en.pdf

온라인:
- https://docs.qgis.org/testing/en/docs/

### 교재 집필 정책

초고 작성:
> QGIS 4.2 기준

기술 감수 시:
> QGIS testing 문서와 API diff 확인

최종 인쇄 직전:
> QGIS 4.2 최신 point release + QGIS 4.3/4.4 변화 확인

---

# 6. PyQGIS API 레퍼런스

## 6.1 Python API

현재 안정 QGIS 4.2:

- https://qgis.org/pyqgis/4.2/

QGIS 3.44:

- https://qgis.org/pyqgis/3.44/

개발 master:

- https://qgis.org/pyqgis/master/

### 교재에서 반드시 API 문서를 사용하는 방법을 가르쳐야 하는 이유

QGIS Plugin 개발에서 실제 생산성을 좌우하는 능력은
“API를 암기하는 능력”이 아니라 다음 능력이다.

1. 원하는 클래스를 찾는 방법
2. 부모 클래스 상속 관계 파악
3. 메서드의 인자/반환형 확인
4. enum 변경 확인
5. deprecated API 확인
6. 관련 signal 검색
7. QGIS C++ API와 Python binding 차이 확인

---

# 7. QGIS C++ API

PyQGIS에 원하는 설명이 부족한 경우 C++ API가 더 상세한 경우가 많다.

QGIS 4.2 C++ API:
- https://api.qgis.org/api/4.2/

QGIS API Main:
- https://api.qgis.org/

### 교재에서 포함할 내용

“PyQGIS API에서 설명이 부족할 때 C++ API를 역추적하는 방법”을
중급 장으로 포함하는 것을 권장한다.

예:

```text
QgsVectorLayer
  ↓
Python API에서 method 확인
  ↓
C++ API에서 상세 description 확인
  ↓
PyQGIS signature와 비교
```

---

# 8. QGIS Developers Guide

QGIS 4.2:
- https://docs.qgis.org/4.2/en/docs/developers_guide/

핵심 영역:

- Coding Standards
- Human Interface Guidelines
- Development Process
- QtCreator
- Localization
- Unit Testing
- Processing Algorithm Testing
- OGC Conformance Testing

HIG 및 코딩규칙은 상용 품질 플러그인을 다루는 교재라면 반드시 반영해야 한다.

---

# 9. QGIS Plugin 구조 공식 문서

## 9.1 Python Plugin 개발

- https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/plugins/index.html

핵심 파일:

```text
my_plugin/
├── __init__.py
├── metadata.txt
├── main_plugin.py
├── LICENSE
├── README.md
├── icon.svg
├── ui/
├── processing/
├── i18n/
└── tests/
```

핵심 생명주기:

```text
QGIS
  ↓
classFactory(iface)
  ↓
PluginClass(iface)
  ↓
initGui()
  ↓
사용자 실행
  ↓
unload()
```

교재 초반에 이 생명주기를 시각적으로 설명하는 것이 중요하다.

---

# 10. Processing Plugin

공식 문서:
- https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/processing.html

QGIS 공식 Processing Script Template:
- https://github.com/qgis/QGIS/blob/master/python/plugins/processing/script/ScriptTemplate.py

QGIS Processing Plugin 소스:
- https://github.com/qgis/QGIS/tree/master/python/plugins/processing

### 교재에서 Processing을 별도 Part로 만들 것을 권장

이유:

Processing Plugin은 일반 GUI Plugin보다

- Model Designer 연동
- Batch Processing
- Processing History
- Python 호출
- `qgis_process`
- Model/Script 재활용

등의 장점을 자동으로 얻는다.

---

# 11. QGIS Plugin Repository

메인:
- https://plugins.qgis.org/

Plugin Author Resources:
- https://plugins.qgis.org/

Plugin Publishing:
- https://plugins.qgis.org/docs/publish

Plugin Approval:
- https://plugins.qgis.org/docs/approval

QGIS 4 Migration:
- https://plugins.qgis.org/docs/migrate-qgis4

### 출판 교재에서 반드시 다룰 내용

- `metadata.txt`
- plugin name
- qgisMinimumVersion
- qgisMaximumVersion
- homepage
- tracker
- repository
- license
- changelog
- external dependency
- plugin ZIP 구조
- 보안 검사
- repository 승인 절차

---

# 12. QGIS 4 / Qt6 Migration

QGIS 4는 Qt6 기반이다.

공식 Migration 안내:
- https://plugins.qgis.org/docs/migrate-qgis4

PyQGIS 4 Checker:
- https://github.com/qgis/pyqgis4-checker

Qt5 → Qt6 변환 스크립트:
- https://github.com/qgis/QGIS/blob/master/scripts/pyqt5_to_pyqt6/pyqt5_to_pyqt6.py

### 권장 코딩 규칙

가능하면 직접:

```python
from PyQt6.QtWidgets import QAction
```

형태보다 QGIS가 제공하는 compatibility layer를 우선한다.

```python
from qgis.PyQt.QtWidgets import QAction
from qgis.PyQt.QtCore import Qt
```

장점:

- QGIS 런타임과 Qt binding 정합성
- QGIS 3/4 동시 호환 코드 작성 가능성 향상
- 직접 PyQt dependency 충돌 감소

---

# 13. Plugin Builder 4

QGIS 4 신규 교재라면 **Plugin Builder 3보다 Plugin Builder 4를 기본 도구로 채택**하는 것이 적절하다.

Plugin Builder 4:
- https://plugins.qgis.org/plugins/pluginbuilder4/

문서:
- https://jonah-sullivan.github.io/Qgis-Plugin-Builder/

2026-08 기준 최신 안정 버전 계열은 QGIS 4 전용으로 제공되고 있다.

주요 기능:

- QGIS 4 / Qt6 template
- dock widget
- translation
- tests
- README / LICENSE
- pre-commit
- Ruff
- qgis-plugin-ci
- GitHub Actions
- GitLab CI
- plugin repository 배포 템플릿

### 교재 실습

**Chapter 4: Plugin Builder 4로 첫 Plugin 만들기**

예제:

```text
HelloQGIS
  ├─ 메뉴 생성
  ├─ Toolbar icon
  ├─ Dialog
  ├─ DockWidget
  ├─ metadata
  ├─ README
  └─ test
```

---

# 14. Plugin Reloader

Plugin Reloader:
- https://plugins.qgis.org/plugins/plugin_reloader/

QGIS Plugin 개발 중 코드를 수정한 뒤 QGIS를 재시작하지 않고
플러그인을 다시 로드하는 핵심 개발도구이다.

QGIS 4 호환 버전이 제공되고 있다.

### 실습 워크플로

```text
VS Code / PyCharm
       ↓
Python 코드 수정
       ↓
저장
       ↓
QGIS Plugin Reloader
       ↓
Plugin 재실행
```

---

# 15. qgis-plugin-ci

GitHub:
- https://github.com/qgis/qgis-plugin-ci

문서:
- https://qgis.github.io/qgis-plugin-ci/

PyPI:
- https://pypi.org/project/qgis-plugin-ci/

주요 기능:

- plugin package 생성
- QGIS Plugin Repository 배포
- GitHub Release
- custom plugin repository
- changelog 자동화
- Transifex translation
- CI/CD

### 교재 고급편에서 반드시 포함

```text
git tag
  ↓
GitHub Actions
  ↓
pytest / lint
  ↓
qgis-plugin-ci package
  ↓
GitHub Release
  ↓
QGIS Plugin Repository
```

---

# 16. Qt 6 / GUI 개발 공식 자료

## 16.1 Qt 공식 문서

Qt 6:
- https://doc.qt.io/qt-6/

Qt Widgets:
- https://doc.qt.io/qt-6/qtwidgets-index.html

Qt Designer:
- https://doc.qt.io/qt-6/qtdesigner-manual.html

Signals & Slots:
- https://doc.qt.io/qt-6/signalsandslots.html

Model/View:
- https://doc.qt.io/qt-6/model-view-programming.html

Threading:
- https://doc.qt.io/qt-6/threads.html

---

# 17. PyQt6 공식 자료

Riverbank:
- https://www.riverbankcomputing.com/software/pyqt/
- https://www.riverbankcomputing.com/software/pyqt/download
- https://www.riverbankcomputing.com/static/Docs/PyQt6/

### 중요

QGIS Plugin에서는 일반 standalone PyQt6 예제를 그대로 복사하기보다
가능하면 다음 형태를 사용한다.

```python
from qgis.PyQt import QtCore, QtGui, QtWidgets
```

---

# 18. Python 공식 문서

QGIS가 번들한 Python 환경과 호환되는 코드를 작성해야 한다.

Python documentation:
- https://docs.python.org/3/

Python tutorial:
- https://docs.python.org/3/tutorial/

Python 3.12 docs download:
- https://docs.python.org/3.12/download.html

Python Korean:
- https://docs.python.org/ko/3/

### 주의

Python 프로젝트는 최근 버전에서 사전 빌드된 PDF 문서를 지속적으로 갱신하지 않고
HTML / EPUB / source 기반 배포를 중심으로 제공한다.

따라서 교재 참고자료는 **온라인 최신 Python 문서**를 우선하는 것이 안전하다.

---

# 19. QGIS Documentation 작성법 자체를 참고

QGIS Documentation Guidelines:
- https://docs.qgis.org/4.2/en/docs/documentation_guidelines/

PDF:
- https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DocumentationGuidelines-en.pdf

여기서 참고할 부분:

- 제목 체계
- note / warning / tip
- screenshot 규칙
- code snippet
- reference
- cross-reference
- Sphinx
- reStructuredText
- 테스트 가능한 PyQGIS 코드

### 교재 편집 디자인으로 변환

QGIS 문서 스타일을 책에 적용하면 다음과 같이 구성 가능하다.

> **TIP**  
> 빠른 개발에서는 Plugin Reloader를 사용한다.

> **WARNING**  
> Background thread에서 직접 QGIS GUI 객체를 수정하면 안 된다.

> **API**  
> `QgsProject.instance()`

> **ENGINEERING PRACTICE**  
> 1천만 feature를 반복 처리할 때 Python for-loop 대신 provider filter / spatial index / Processing을 우선 검토한다.

---

# 20. 라이선스 및 저작권 사전 검토

## 20.1 QGIS Software

QGIS:
- GNU GPL v2 or later

라이선스 정보:
- https://docs.qgis.org/4.2/en/docs/about/license/index.html

## 20.2 QGIS Documentation

QGIS 공식 4.2 문서는:
- **GNU Free Documentation License 1.3 or later**

Preamble:
- https://docs.qgis.org/4.2/en/docs/about/preamble.html

PyQGIS Cookbook 역시 문서 및 코드 조각에 GFDL 조건이 적용됨을 명시한다.

### 교재 집필 시 권장 원칙

1. 공식 문서를 **참고자료**로 사용한다.
2. 설명 문장은 직접 재구성한다.
3. API 이름과 함수 signature는 기술적 사실로 인용 가능하지만,
   문서 설명을 대량 복사하지 않는다.
4. 공식 예제 코드를 그대로 사용하는 경우 라이선스를 확인한다.
5. 출처·버전·URL을 명확히 기록한다.
6. 출판 전 출판사 또는 저작권 전문가의 라이선스 검토를 받는다.

---

# 21. QGIS Plugin Repository 콘텐츠 라이선스

QGIS Plugins 웹사이트는 사이트 콘텐츠에 대해
CC BY-SA 계열 라이선스 안내를 제공한다.

하지만:

- Plugin source 자체 라이선스
- Plugin icon
- screenshot
- external library
- sample data

는 각 프로젝트의 LICENSE를 별도로 확인해야 한다.

---

# 22. 기존 상용 QGIS / PyQGIS 서적 조사

## 22.1 PyQGIS Programmer's Guide 3

출판사:
- Locate Press

링크:
- https://locatepress.com/book/ppg3

특징:

- QGIS 3
- Python 3
- API 탐색
- Python Console
- scripting
- plugin
- standalone application
- exercise

한계:

- QGIS 3.0 시대 기반
- Qt6 / QGIS4 반영 전

### 참고 가치

**교재의 교육 순서와 exercise 구성 벤치마킹용으로 유용하다.**

---

## 22.2 QGIS Python Programming Cookbook — Second Edition

출판사:
- Packt

링크:
- https://www.packtpub.com/en-us/product/qgis-python-programming-cookbook-second-edition-9781787121102

Example Code:
- https://github.com/PacktPublishing/QGIS-Python-Programming-Cookbook-Second-Edition

특징:

- recipe 방식
- 자동화 사례가 많음

한계:

- QGIS 2.18 기반으로 매우 오래됨

### 활용

기술 소스보다는
**“문제 → 짧은 recipe → 결과”의 편집 구조**를 참고한다.

---

# 23. QGIS 공식 Books 페이지

QGIS Books:
- https://www.qgis.org/resources/books/

용도:

- 기존 QGIS 책의 시장 조사
- 출간 연도 확인
- 주제 중복 검토
- 차별화 포인트 발굴

---

# 24. 무료 PDF와 유료 PDF 구분 정책

교재 리서치 파일에서는 반드시 다음처럼 구분할 것을 권장한다.

## A. 공식 무료 PDF

예:

- QGIS PyQGIS Developer Cookbook
- QGIS Training Manual
- QGIS Desktop User Guide
- QGIS Server Guide
- Documentation Guidelines

→ QGIS 공식 서버에서 직접 다운로드.

## B. 출판사 판매 PDF

예:

- Locate Press
- Packt

→ 구매 링크만 기록.

## C. 비공식 PDF 공유사이트

→ **교재 조사자료에 포함하지 않는 것을 권장.**

이유:

- 저작권 위험
- 변조 가능성
- 버전 불명
- 악성파일 가능성
- 출판사 법무 검토 시 문제

---

# 25. 추천 교재 제목 후보

1. **QGIS 4 Plugin Development**
   - PyQGIS · Qt6 · Processing · Automation

2. **실무자를 위한 QGIS Plugin 개발**
   - Python으로 만드는 GIS 업무자동화

3. **PyQGIS 4 실전 프로그래밍**
   - QGIS Plugin부터 GeoAI까지

4. **엔지니어를 위한 QGIS 자동화 개발**
   - Plugin · Processing · Spatial Database · AI

5. **QGIS Plugin Engineering**
   - Production-Grade GIS Automation with Python

---

# 26. 권장 교재 전체 목차

## Part I. QGIS Plugin Development Foundations

### Chapter 1. QGIS를 개발 플랫폼으로 이해하기

- QGIS Desktop architecture
- QGIS Core / GUI
- PyQGIS
- Qt
- Processing
- Plugin
- Provider
- GDAL / PROJ / GEOS
- Python Runtime

### Chapter 2. QGIS 4 개발환경

- QGIS 4.2 설치
- OSGeo4W
- Python environment
- Qt6
- VS Code
- PyCharm
- Python Console
- PYTHONPATH
- QGIS profile

### Chapter 3. PyQGIS API 읽는 법

- Python API
- C++ API
- class hierarchy
- method
- enum
- signals
- deprecation

---

# Part II. 첫 QGIS Plugin

## Chapter 4. Plugin Builder 4

- 설치
- template 생성
- metadata
- `__init__.py`
- `classFactory`
- `initGui`
- `unload`

## Chapter 5. 메뉴와 Toolbar

- QAction
- icon
- signal
- callback
- status message

## Chapter 6. Qt Designer와 Dialog

- `.ui`
- dialog
- widget
- layout
- signal/slot
- QGIS widget

## Chapter 7. Dock Widget

- QgsDockWidget
- layer selection
- live update
- settings

---

# Part III. PyQGIS Core Programming

## Chapter 8. QgsProject와 Layer

- QgsProject
- mapLayers
- layer tree
- add/remove layer

## Chapter 9. Vector Layer

- QgsVectorLayer
- QgsFeature
- field
- selection
- edit buffer
- provider

## Chapter 10. Geometry

- point
- line
- polygon
- multipart
- WKT / WKB
- spatial predicate
- geometry operation

## Chapter 11. CRS

- QgsCoordinateReferenceSystem
- QgsCoordinateTransform
- EPSG
- PROJ
- datum transformation

## Chapter 12. Raster / DEM

- QgsRasterLayer
- raster provider
- pixel query
- GDAL integration

---

# Part IV. Map Canvas Development

## Chapter 13. Map Canvas

- QgsMapCanvas
- extent
- refresh
- coordinate transform

## Chapter 14. Custom Map Tool

- QgsMapTool
- identify
- mouse event
- snapping
- rubber band
- vertex marker

## Chapter 15. Symbology

- renderer
- symbols
- categorized
- graduated
- labeling

---

# Part V. Processing Plugin

## Chapter 16. QgsProcessingAlgorithm

## Chapter 17. Processing Provider

## Chapter 18. Batch / Model Builder / qgis_process

---

# Part VI. Production-Grade Plugin

## Chapter 19. QgsTask와 Background Processing

- UI freeze
- QgsTask
- cancel
- progress
- thread safety

## Chapter 20. Settings

- QgsSettings
- user profile
- project settings

## Chapter 21. Logging / Error Handling

- QgsMessageLog
- Python logging
- exception
- log file

## Chapter 22. Network / API

- QgsNetworkAccessManager
- REST
- JSON
- authentication

## Chapter 23. Database

- PostgreSQL
- PostGIS
- GeoPackage
- SQLite

---

# Part VII. Software Engineering

## Chapter 24. Plugin Architecture

권장 구조:

```text
plugin/
├── __init__.py
├── plugin.py
├── metadata.txt
├── gui/
├── core/
├── services/
├── repositories/
├── processing/
├── tasks/
├── utils/
├── tests/
└── resources/
```

설계 패턴:

- MVC/MVP
- Service Layer
- Repository
- Dependency Injection
- Adapter
- Strategy

## Chapter 25. Testing

- unittest
- pytest
- headless test
- Processing algorithm test

## Chapter 26. Code Quality

- Ruff
- pre-commit
- type hints
- mypy 선택적 활용
- docstring

## Chapter 27. Git / GitHub

- branch
- PR
- issue
- release

## Chapter 28. CI/CD

- GitHub Actions
- qgis-plugin-ci
- semantic versioning
- release

---

# Part VIII. QGIS 4 Migration

## Chapter 29. Qt5 → Qt6

- enum
- API 변경
- qgis.PyQt
- deprecated API

## Chapter 30. PyQGIS 4 Checker

- static migration
- Docker
- CI integration

---

# Part IX. 엔지니어링 Plugin 실전 프로젝트

이 부분이 기존 해외 PyQGIS 서적과 차별화할 수 있는 핵심이다.

## Project 1. Coordinate Transformer

기능:

- EPSG 선택
- CSV coordinate import
- CRS transform
- validation
- export

관련 API:

- QgsCoordinateReferenceSystem
- QgsCoordinateTransform

---

## Project 2. Survey GCP / Benchmark QA Plugin

기능:

- GCP CSV
- SHP / GPKG
- tolerance
- XY/Z error
- outlier
- map marker
- QA report

---

## Project 3. DEM QA Plugin

기능:

- NoData
- min/max
- slope
- coverage
- hole
- abnormal elevation
- statistics

---

## Project 4. HEC-RAS Geometry Preparation Plugin

기능:

- centerline
- cross-section
- bank line
- levee
- DEM sampling
- stationing
- CSV export

---

## Project 5. Drone Survey QA Plugin

기능:

- orthophoto
- DSM/DTM
- LAS/LAZ metadata
- flight boundary
- GCP residual
- coordinate system QA

---

## Project 6. Point Cloud QA Plugin

- LAS
- LAZ
- COPC
- density
- classification
- elevation statistics
- tile index

---

## Project 7. GIS–BIM QA Plugin

- IFC
- CRS
- bounding box
- attribute mapping
- GeoPackage/PostGIS export

---

## Project 8. GeoAI Plugin

- segmentation
- detection
- AI inference
- GeoTIFF
- vectorization
- model parameter UI

---

# 27. 교재 차별화 전략

기존 PyQGIS 자료의 주요 약점은 다음과 같다.

1. API 설명 위주
2. QGIS 2/3 기반 구자료가 많음
3. 소프트웨어 엔지니어링 설명 부족
4. Test/CI/CD 부족
5. 실제 엔지니어링 프로젝트 부족
6. 대용량 GIS 데이터 처리 부족
7. Qt6/QGIS4 내용 부족
8. AI coding workflow 부족

따라서 신규 교재는 아래 영역을 강화해야 한다.

```text
PyQGIS
   +
Qt6 GUI
   +
Processing
   +
Software Architecture
   +
Testing
   +
CI/CD
   +
Large GIS Data
   +
Civil / Survey / Drone Engineering
   +
AI Assisted Development
```

---

# 28. AI 기반 QGIS Plugin 개발 챕터 제안

2026년 교재라면 AI 개발 워크플로를 빼기 어렵다.

## 포함 권장 내용

- Claude Code
- ChatGPT
- Codex
- Gemini
- Cursor
- VS Code Agent
- GitHub Copilot

### Workflow

```text
Requirement
   ↓
PRD
   ↓
Plugin Architecture
   ↓
AI Code Generation
   ↓
PyQGIS API Verification
   ↓
Run QGIS
   ↓
Plugin Reloader
   ↓
Test
   ↓
Git
```

### 반드시 강조할 점

AI가 생성한 PyQGIS 코드는
**QGIS Python API 문서로 반드시 검증**해야 한다.

특히 흔한 오류:

- 존재하지 않는 class
- QGIS 2 API
- Qt5 enum
- PyQt5 direct import
- deprecated method
- wrong Processing parameter
- GUI thread violation

---

# 29. PDF 폴더 구성 권장

```text
QGIS_PLUGIN_BOOK_RESEARCH/
│
├── 00_INDEX/
│
├── 01_QGIS_4_2/
│   ├── PyQGIS_Cookbook/
│   ├── Training/
│   ├── User_Guide/
│   ├── Server/
│   └── Documentation_Guidelines/
│
├── 02_QGIS_3_44_KO/
│
├── 03_QGIS_TESTING/
│
├── 04_QT6/
│
├── 05_PYQT6/
│
├── 06_PYTHON/
│
├── 07_PLUGIN_EXAMPLES/
│
├── 08_BOOKS/
│
├── 09_LICENSE/
│
└── 10_BOOK_DRAFT/
```

---

# 30. Windows PowerShell — 필수 PDF 일괄 다운로드

```powershell
$Root = "$PWD\QGIS_PLUGIN_BOOK_RESEARCH"

New-Item -ItemType Directory -Force "$Root\01_QGIS_4_2" | Out-Null
New-Item -ItemType Directory -Force "$Root\02_QGIS_3_44_KO" | Out-Null
New-Item -ItemType Directory -Force "$Root\03_QGIS_TESTING" | Out-Null

$QGIS42 = @(
  "https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-PyQGISDeveloperCookbook-en.pdf",
  "https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-TrainingManual-en.pdf",
  "https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DocumentationGuidelines-en.pdf",
  "https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DesktopUserGuide-en.pdf",
  "https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-ServerUserGuide-en.pdf"
)

foreach ($url in $QGIS42) {
    $name = Split-Path $url -Leaf
    Invoke-WebRequest -Uri $url -OutFile "$Root\01_QGIS_4_2\$name"
}

$QGIS344KO = @(
  "https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-PyQGISDeveloperCookbook-ko.pdf",
  "https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-TrainingManual-ko.pdf",
  "https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-DocumentationGuidelines-ko.pdf",
  "https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-DesktopUserGuide-ko.pdf",
  "https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-ServerUserGuide-ko.pdf"
)

foreach ($url in $QGIS344KO) {
    $name = Split-Path $url -Leaf
    Invoke-WebRequest -Uri $url -OutFile "$Root\02_QGIS_3_44_KO\$name"
}

$Testing = @(
  "https://docs.qgis.org/testing/pdf/en/QGIS-testing-PyQGISDeveloperCookbook-en.pdf",
  "https://docs.qgis.org/testing/pdf/en/QGIS-testing-DocumentationGuidelines-en.pdf"
)

foreach ($url in $Testing) {
    $name = Split-Path $url -Leaf
    Invoke-WebRequest -Uri $url -OutFile "$Root\03_QGIS_TESTING\$name"
}

Write-Host "Download Complete: $Root"
```

---

# 31. 반드시 북마크할 URL — Top 20

1. QGIS Download  
   https://www.qgis.org/download/

2. QGIS Roadmap  
   https://www.qgis.org/resources/roadmap/

3. QGIS 4.2 Documentation  
   https://docs.qgis.org/4.2/en/docs/

4. PyQGIS 4.2 Cookbook  
   https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/

5. PyQGIS 4.2 API  
   https://qgis.org/pyqgis/4.2/

6. QGIS C++ API  
   https://api.qgis.org/api/4.2/

7. Python Plugin Development  
   https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/plugins/index.html

8. Processing Plugin  
   https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/processing.html

9. QGIS Developers Guide  
   https://docs.qgis.org/4.2/en/docs/developers_guide/

10. QGIS Plugin Repository  
    https://plugins.qgis.org/

11. Publishing Plugin  
    https://plugins.qgis.org/docs/publish

12. Approval Guide  
    https://plugins.qgis.org/docs/approval

13. QGIS4 Migration  
    https://plugins.qgis.org/docs/migrate-qgis4

14. Plugin Builder 4  
    https://plugins.qgis.org/plugins/pluginbuilder4/

15. Plugin Builder Docs  
    https://jonah-sullivan.github.io/Qgis-Plugin-Builder/

16. Plugin Reloader  
    https://plugins.qgis.org/plugins/plugin_reloader/

17. qgis-plugin-ci  
    https://github.com/qgis/qgis-plugin-ci

18. PyQGIS4 Checker  
    https://github.com/qgis/pyqgis4-checker

19. Qt6  
    https://doc.qt.io/qt-6/

20. PyQt6  
    https://www.riverbankcomputing.com/static/Docs/PyQt6/

---

# 32. 자료 신뢰도 등급

## S — 교재의 기술적 기준

- QGIS official documentation
- PyQGIS API
- QGIS API
- QGIS source
- QGIS Plugin Repository rules
- Qt official docs
- Python official docs

## A — 개발도구 공식 프로젝트

- Plugin Builder 4
- Plugin Reloader
- qgis-plugin-ci
- pyqgis4-checker

## B — 유명 개발자/출판사

- Locate Press
- Packt
- 검증된 QGIS 개발자 blog/tutorial

## C — 일반 블로그

API가 공식 문서와 일치하는지 검증 후 사용.

## D — AI 생성 답변

아이디어 및 초안 생성에만 사용하고
반드시 S/A 등급 자료로 검증.

---

# 33. 교재 집필 시 Citation Database 권장

예:

```yaml
id: qgis42-pyqgis-cookbook
title: PyQGIS 4.2 Developer Cookbook
organization: QGIS Project
version: 4.2
url: https://docs.qgis.org/4.2/en/docs/pyqgis_developer_cookbook/
pdf: https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-PyQGISDeveloperCookbook-en.pdf
license: GNU FDL 1.3+
accessed: 2026-08-27
priority: S
```

모든 자료를 YAML/CSV/BibTeX로 관리하면
향후 GitBook·PDF·출판용 참고문헌 생성이 쉬워진다.

---

# 34. 교재 집필용 Git Repository 제안

```text
qgis-plugin-development-book/
│
├── README.md
├── LICENSE
├── CITATIONS.md
├── CHANGELOG.md
│
├── book/
│   ├── part01/
│   ├── part02/
│   ├── part03/
│   └── appendices/
│
├── examples/
│   ├── 01_hello_qgis/
│   ├── 02_dock_widget/
│   ├── 03_map_tool/
│   ├── 04_processing/
│   └── engineering/
│
├── assets/
│   ├── figures/
│   └── screenshots/
│
├── tests/
│
├── references/
│   ├── sources.yaml
│   └── bibliography.bib
│
└── scripts/
    ├── check_links.py
    ├── build_book.py
    └── capture_qgis_version.py
```

---

# 35. 출판 전 기술 검수 체크리스트

- [ ] QGIS 4.2.x에서 모든 예제 실행
- [ ] QGIS 3.44 LTR 호환 예제 확인
- [ ] Qt5/Qt6 차이 표시
- [ ] `qgis.PyQt` 사용 여부 확인
- [ ] Deprecated API 검사
- [ ] PyQGIS4 Checker 실행
- [ ] Plugin Builder 4 최신 template 비교
- [ ] metadata validation
- [ ] Plugin ZIP validation
- [ ] Plugin Repository 정책 최신화
- [ ] Windows 테스트
- [ ] Linux 테스트
- [ ] macOS 가능하면 테스트
- [ ] Processing Modeler 연동 검증
- [ ] QgsTask cancel 검증
- [ ] 대용량 레이어 테스트
- [ ] Python exception 처리 확인
- [ ] screenshot QGIS 버전 일치
- [ ] 코드 GitHub 공개
- [ ] LICENSE 확인
- [ ] 공식 문서 출처 정리
- [ ] PDF URL dead-link 검사

---

# 36. 최종 권고

## 교재 기술 기준

```text
QGIS 4.2+
Python bundled with QGIS
Qt 6
qgis.PyQt
PyQGIS
Processing Framework
Plugin Builder 4
Plugin Reloader
Git
pytest/unittest
Ruff
pre-commit
GitHub Actions
qgis-plugin-ci
```

## 핵심 차별화 문장

> 단순히 QGIS Plugin을 만드는 방법을 설명하는 책이 아니라  
> **“GIS 엔지니어가 반복 업무를 Production-grade QGIS Plugin으로 제품화하는 방법”**을 설명하는 교재로 설계한다.

특히 토목·측량·드론·DEM·Point Cloud·좌표계·수자원·GeoAI 실전 프로젝트를 포함하면
기존 PyQGIS 교재와 상당히 차별화할 수 있다.

---

# 37. 즉시 다운로드 권장 PDF 8개

가장 먼저 다음 8개를 다운로드하면 된다.

1. PyQGIS 4.2 Developer Cookbook  
   https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-PyQGISDeveloperCookbook-en.pdf

2. QGIS 4.2 Training Manual  
   https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-TrainingManual-en.pdf

3. QGIS 4.2 Documentation Guidelines  
   https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DocumentationGuidelines-en.pdf

4. QGIS 4.2 Desktop User Guide  
   https://docs.qgis.org/4.2/pdf/en/QGIS-4.2-DesktopUserGuide-en.pdf

5. PyQGIS 3.44 Developer Cookbook 한국어  
   https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-PyQGISDeveloperCookbook-ko.pdf

6. QGIS 3.44 Training Manual 한국어  
   https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-TrainingManual-ko.pdf

7. QGIS 3.44 Documentation Guidelines 한국어  
   https://docs.qgis.org/3.44/pdf/ko/QGIS-3.44-DocumentationGuidelines-ko.pdf

8. PyQGIS Testing Developer Cookbook  
   https://docs.qgis.org/testing/pdf/en/QGIS-testing-PyQGISDeveloperCookbook-en.pdf

---

# 38. 다음 작업 권장

이 사전조사 이후 다음 산출물을 순서대로 만드는 것을 권장한다.

```text
01. BOOK_PRD.md
        ↓
02. BOOK_TOC.md
        ↓
03. AUTHORING_GUIDELINE.md
        ↓
04. QGIS_PLUGIN_TEMPLATE/
        ↓
05. chapter_01 ~ chapter_30
        ↓
06. engineering project source
        ↓
07. GitBook HTML
        ↓
08. PDF / ePub
```

---

## References / Verification URLs

- QGIS Download: https://www.qgis.org/download/
- QGIS Roadmap: https://www.qgis.org/resources/roadmap/
- QGIS 4.2 Docs: https://docs.qgis.org/4.2/en/docs/
- QGIS 4.2 PDF index: https://docs.qgis.org/4.2/pdf/en/
- QGIS 3.44 KO PDF index: https://docs.qgis.org/3.44/pdf/ko/
- QGIS 3.44 KO PyQGIS: https://docs.qgis.org/3.44/ko/docs/pyqgis_developer_cookbook/index.html
- PyQGIS 4.2 API: https://qgis.org/pyqgis/4.2/
- QGIS API 4.2: https://api.qgis.org/api/4.2/
- QGIS Plugin Repository: https://plugins.qgis.org/
- Plugin Builder 4: https://plugins.qgis.org/plugins/pluginbuilder4/
- Plugin Reloader: https://plugins.qgis.org/plugins/plugin_reloader/
- qgis-plugin-ci: https://github.com/qgis/qgis-plugin-ci
- PyQGIS4 Checker: https://github.com/qgis/pyqgis4-checker
- Qt 6 Docs: https://doc.qt.io/qt-6/
- PyQt6 Docs: https://www.riverbankcomputing.com/static/Docs/PyQt6/
- Python Docs: https://docs.python.org/3/
- Locate Press PyQGIS Programmer's Guide 3: https://locatepress.com/book/ppg3
- QGIS Books: https://www.qgis.org/resources/books/

---

**End of pre-research document**
# QGIS-Plugin-Devs-Series
QGIS Plugins Devs Series....
