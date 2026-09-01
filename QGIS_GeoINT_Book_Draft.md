# QGIS 기반 GeoINT 솔루션 개발

### QGIS 3.6부터 4.x까지 아우르는 지리공간정보 분석 플러그인 구축 실무

- 초고 작성일: 2026-08-31
- 대상 QGIS: **3.6 이상 ~ 4.x** (Qt5 / Qt6 동시 지원)
- 예제 플러그인 코드명: **QGeoINT** (패키지명 `qgeoint`)
- 주요 언어/프레임워크: Python 3.7+ / PyQGIS / `qgis.PyQt` / Processing Framework / GDAL
- 시리즈: 인프라 엔지니어를 위한 QGIS 플러그인 개발 교재 (제4권)

---

## 이 책에 대하여

### 대상 독자

이 책은 다음과 같은 독자를 전제로 한다.

- 공간정보를 다루지만 "지도 제작"이 아니라 **판단 근거 생산**이 목적인 실무자
- 재난 대응, 국토·시설물 감시, 환경 변화 추적, 공개출처정보 검증 업무 담당자
- 사내에서 반복되는 영상·지형 분석 절차를 도구로 제품화해야 하는 GIS 개발자
- 이미 QGIS를 사용해 왔고, PyQGIS 기초 문법을 어느 정도 알고 있는 엔지니어
- 폐쇄망(오프라인) 환경에서 동작해야 하는 분석 도구를 만들어야 하는 개발자

Python 문법 자체는 설명하지 않는다. 대신 **PyQGIS API를 어떻게 읽고 검증하는지**,
그리고 **분석 결과를 어떻게 방어 가능한 형태로 남기는지**에 지면을 집중한다.

### 이 책이 다루지 않는 것

- 특정 국가·기관의 보안 규정 또는 비공개 자료 처리 절차
- 개인 식별을 목적으로 하는 위치 추적 기법
- 상용 위성영상 사업자별 계약 조건 및 가격 정책
- 딥러닝 모델의 학습(training) 과정 — 이 책은 **추론(inference) 통합**만 다룬다

법·윤리 경계는 부록이 아니라 본문 Chapter 29에 배치했다. 뒤로 미룰 수 없는 주제이기 때문이다.

---

## 이 책의 세 가지 설계 원칙

이 책의 모든 코드는 아래 세 가지 기준을 만족하도록 작성되었다.
30개 장 전체를 관통하는 검증 기준이므로 먼저 명확히 해 둔다.

### 원칙 1 — 재현성 (Reproducibility)

> 같은 입력과 같은 파라미터를 넣으면 6개월 뒤 다른 사람이 실행해도 같은 결과가 나와야 한다.

이를 위해 QGeoINT의 모든 분석은 **실행 레시피(recipe)** 를 JSON으로 남긴다.
마우스로 조작한 임계값이라도 반드시 직렬화되어 저장된다.

### 원칙 2 — 추적성 (Provenance / Traceability)

> 생성된 모든 레이어는 "어떤 원본에서, 어떤 처리를 거쳐, 언제 만들어졌는지"를 스스로 답할 수 있어야 한다.

QGeoINT는 모든 산출 레이어에 `qgeoint:provenance` 커스텀 프로퍼티를 부착한다.
출처 정보가 없는 레이어는 산출물로 인정하지 않는다.

### 원칙 3 — 방어가능성 (Defensibility)

> 분석 결론에는 항상 신뢰도 등급과 불확실성 범위가 함께 붙어야 한다.

단정형 산출물은 금지한다. "변화 있음"이 아니라
"변화 탐지됨 / 신뢰도 B2 / 오탐 가능 요인: 계절 차이, 그림자 각도" 형태로 표현한다.

> **ENGINEERING PRACTICE**
> 이 세 원칙은 코드 리뷰 체크리스트로 그대로 사용할 수 있다.
> 새 기능을 머지하기 전에 "재현되는가 / 추적되는가 / 방어되는가" 세 질문을 던진다.

---

## 버전 정책 — 왜 QGIS 3.6까지 내려가는가

2026년 기준 최신 QGIS는 4.2이고, LTR은 3.44이다.
그럼에도 이 책이 하한을 **3.6**으로 잡은 이유는 다음과 같다.

| 환경 | 실제 관측되는 QGIS 버전 | 이유 |
|---|---|---|
| 개인·연구 | 4.x | 최신 기능 필요 |
| 기업 표준 PC | 3.22 / 3.28 / 3.34 LTR | IT 정책상 고정 |
| 폐쇄망 분석망 | **3.6 ~ 3.16** | 보안 검증 통과 버전만 반입 가능 |
| 공공 납품 환경 | 3.10 / 3.16 | 조달 시점에 고정된 버전 |

표 0-1. 실무 환경별 QGIS 버전 분포

폐쇄망 분석 환경은 소프트웨어 반입 절차가 무겁기 때문에
버전이 수년간 고정되는 일이 흔하다. 3.6은 그러한 환경에서 여전히 관측되는 하한선이며,
동시에 `QgsProcessingAlgorithm` 및 `QgsTask` API가 현재와 거의 동일한 형태로 안정화된
첫 세대이기도 하다.

> **호환성 원칙**
> 본문 코드는 **4.x를 기준**으로 작성하고,
> 3.6~3.16에서 동작하지 않는 부분은 `compat.py`를 통해 흡수한다.
> 각 장 말미의 "버전 호환 노트"에 차이를 정리한다.

---

## 전체 구성

```text
Part I    GeoINT와 QGIS 플랫폼            Ch 1  ~ Ch 3
Part II   플러그인 골격과 호환성 계층      Ch 4  ~ Ch 7
Part III  데이터 인제스트와 정규화          Ch 8  ~ Ch 11
Part IV   영상 분석                        Ch 12 ~ Ch 15
Part V    지형 분석                        Ch 16 ~ Ch 18
Part VI   공개출처정보(OSINT) 융합         Ch 19 ~ Ch 21
Part VII  산출물과 시각화                  Ch 22 ~ Ch 24
Part VIII 운영 품질과 배포                 Ch 25 ~ Ch 28
Part IX   법·윤리와 종합 프로젝트          Ch 29 ~ Ch 30
부록      A ~ E
```

---
---

# Part I. GeoINT와 QGIS 플랫폼

---

# Chapter 1. GeoINT란 무엇인가

## 1.1 정의

GeoINT(Geospatial Intelligence, 지리공간정보 분석)는
**영상·지형·지도 데이터를 결합하여 특정 장소에서 일어나는 일에 대한 판단을 생산하는 활동**이다.

핵심은 "데이터"가 아니라 "판단"이다. 위성영상 한 장은 GeoINT가 아니다.
그 영상을 지난달 영상과 비교하고, 지형 조건과 대조하고,
공개된 보도 자료와 교차 확인한 뒤 도출된 **"이 지역의 침수 면적은 약 4.2 km², 신뢰도 중"** 이라는 문장이 GeoINT다.

| 구분 | 산출물 | 예시 |
|---|---|---|
| 데이터 (Data) | 원시 파일 | Sentinel-2 L2A 타일 |
| 정보 (Information) | 정리된 값 | NDWI 래스터, 침수 폴리곤 |
| 판단 (Intelligence) | 근거 있는 결론 | "침수 4.2 km², 도로 3개소 단절, 신뢰도 B2" |

표 1-1. 데이터·정보·판단의 구분

> **ENGINEERING PRACTICE**
> 플러그인을 설계할 때 "우리 도구의 최종 출력은 세 단계 중 어디인가"를 먼저 정한다.
> 대부분의 GIS 도구는 2단계에서 멈춘다. GeoINT 도구는 3단계까지 책임진다.

## 1.2 분석 순환 구조

GeoINT 업무는 일반적으로 다음 순환을 따른다.

```text
   요구 정의 (Requirement)
        ↓
   수집 계획 (Collection Planning)
        ↓
   수집 / 인제스트 (Ingest)
        ↓
   처리 / 정규화 (Processing)
        ↓
   분석 (Exploitation)
        ↓
   산출 / 배포 (Production)
        ↓
   피드백 → 요구 정의로 회귀
```

그림 1-1. GeoINT 분석 순환

이 책의 Part 구성은 이 순환에 대응한다.

| 순환 단계 | 해당 Part | 주요 챕터 |
|---|---|---|
| 수집·인제스트 | Part III | Ch 8~11 |
| 처리·정규화 | Part III, IV | Ch 10, 12, 14 |
| 분석 | Part IV, V, VI | Ch 13, 15, 17, 20 |
| 산출·배포 | Part VII | Ch 22~24 |
| 품질 보증 | Part VIII | Ch 25~28 |

표 1-2. 분석 순환과 이 책의 대응 구조

## 1.3 정보 출처의 분류

GeoINT는 단일 출처로 성립하지 않는다. 실무에서 다루는 출처는 대략 다음과 같다.

- **영상 출처**: 위성, 항공, 드론 정사영상, SAR
- **지형 출처**: DEM, DSM, 수치지형도, 점군(LiDAR)
- **공개출처(OSINT)**: 뉴스, 소셜미디어 지오태그, 공공 통계, OSM
- **센서 출처**: AIS, ADS-B, 기상 관측망, IoT 수위계
- **문서 출처**: 보고서, 인허가 문서, 사업 계획서

이 책에서는 **합법적으로 접근 가능한 공개·상용 출처**만을 다룬다.
비공개 자료의 취급 절차는 각 기관 규정을 따라야 하며 이 책의 범위 밖이다.

## 1.4 신뢰도 표기 — Admiralty 등급

다중 출처를 융합할 때 가장 흔한 실패는 "모든 출처를 동등하게 취급하는 것"이다.
공개적으로 널리 쓰이는 Admiralty(NATO) 등급 체계를 QGeoINT의 기본 표기로 채택한다.

| 출처 신뢰도 | 의미 | | 정보 확실성 | 의미 |
|---|---|---|---|---|
| A | 완전 신뢰 | | 1 | 타 출처로 확인됨 |
| B | 대체로 신뢰 | | 2 | 개연성 높음 |
| C | 보통 | | 3 | 개연성 있음 |
| D | 신뢰 낮음 | | 4 | 의심스러움 |
| E | 비신뢰 | | 5 | 개연성 없음 |
| F | 판단 불가 | | 6 | 판단 불가 |

표 1-3. Admiralty 출처·확실성 등급

따라서 산출물의 결론에는 항상 `B2`, `C3` 같은 두 글자 코드가 붙는다.
Chapter 21에서 이 등급을 레이어 속성으로 관리하는 구현을 다룬다.

## 1.5 왜 QGIS인가

상용 GeoINT 플랫폼이 존재함에도 QGIS를 선택하는 실무적 이유는 다음과 같다.

1. **오프라인 설치 가능** — 폐쇄망 반입이 상대적으로 수월하다.
2. **GDAL/PROJ/GEOS 직결** — 포맷·좌표계 처리에서 우회로가 필요 없다.
3. **Processing Framework** — 분석 절차를 모델로 저장하고 배치 실행할 수 있다. 재현성 원칙과 직결된다.
4. **플러그인 확장성** — 기관 고유 절차를 도구로 캡슐화할 수 있다.
5. **라이선스 비용 없음** — 분석관 수 증가에 비용이 비례하지 않는다.
6. **산출물 투명성** — 알고리즘이 공개되어 있어 결과를 설명할 수 있다. 방어가능성 원칙과 직결된다.

> **WARNING**
> 반대로 QGIS의 한계도 분명하다. 대규모 동시 사용자 관리, 자료 접근 통제,
> 감사 로그 중앙화 기능은 기본 제공되지 않는다.
> Chapter 26에서 플러그인 수준의 감사 추적 구현으로 이를 부분적으로 보완한다.

## 1.6 이 책에서 만들 것 — QGeoINT

Part II부터 Part IX까지 하나의 플러그인을 점진적으로 완성한다.

```text
qgeoint/
├── __init__.py
├── metadata.txt
├── plugin.py
├── compat.py              ← Qt5/Qt6 · QGIS 3/4 흡수 계층
├── core/
│   ├── provenance.py      ← 출처 추적
│   ├── recipe.py          ← 실행 레시피 직렬화
│   ├── confidence.py      ← Admiralty 등급
│   └── grid.py            ← MGRS 격자
├── ingest/
├── analysis/
│   ├── change.py
│   ├── terrain.py
│   └── inference.py
├── fusion/
├── production/
├── processing/            ← Processing Provider
├── tasks/                 ← QgsTask 백그라운드
├── gui/
├── ext_libs/              ← 벤더링 의존성
└── tests/
```

그림 1-2. QGeoINT 최종 디렉터리 구조

각 장에서 하나씩 채워 나가며, Chapter 30에서 전체를 결합한 종합 프로젝트를 수행한다.

### 버전 호환 노트 (Chapter 1)

이 장은 개념 설명이므로 버전 의존성이 없다.

---

# Chapter 2. QGIS를 GeoINT 플랫폼으로 이해하기

## 2.1 계층 구조

플러그인 개발자가 반드시 구분해야 할 QGIS 내부 계층은 다음과 같다.

```text
┌──────────────────────────────────────┐
│  Plugin (QGeoINT)                    │  ← 우리가 만드는 것
├──────────────────────────────────────┤
│  qgis.gui      qgis.analysis         │  QGIS GUI / 분석 API
├──────────────────────────────────────┤
│  qgis.core                           │  프로젝트·레이어·지오메트리
├──────────────────────────────────────┤
│  qgis.PyQt  →  PyQt5 또는 PyQt6      │  Qt 바인딩 추상화
├──────────────────────────────────────┤
│  GDAL / OGR / PROJ / GEOS / SpatiaLite│  네이티브 라이브러리
└──────────────────────────────────────┘
```

그림 2-1. QGIS 계층 구조와 플러그인의 위치

> **API**
> 플러그인에서는 `from PyQt5...` 또는 `from PyQt6...`를 **직접 쓰지 않는다.**
> 반드시 `from qgis.PyQt...`를 사용한다. 이것이 3.x/4.x 동시 지원의 출발점이다.

## 2.2 GeoINT 관점에서 중요한 QGIS 서브시스템

| 서브시스템 | 핵심 클래스 | GeoINT에서의 역할 |
|---|---|---|
| 프로젝트 | `QgsProject` | 분석 세션 상태 보관, 커스텀 프로퍼티로 출처 저장 |
| 래스터 | `QgsRasterLayer`, `QgsRasterDataProvider` | 영상 접근, 밴드 통계, 픽셀 조회 |
| 벡터 | `QgsVectorLayer`, `QgsFeature` | 탐지 결과, 관심지역(AOI), 주석 |
| 좌표계 | `QgsCoordinateReferenceSystem`, `QgsCoordinateTransform` | 다중 출처 정합의 기준 |
| Processing | `QgsProcessingAlgorithm` | 재현 가능한 분석 절차 |
| 시간 | `QgsTemporalController` (3.14+) | 시계열 영상 재생 |
| 태스크 | `QgsTask`, `QgsTaskManager` | 대용량 처리 중 UI 유지 |
| 레이아웃 | `QgsLayout`, `QgsLayoutExporter` | 브리핑 산출물 자동 생성 |
| 로그 | `QgsMessageLog` | 감사 추적의 1차 저장소 |

표 2-1. GeoINT 개발에서 자주 사용하는 서브시스템

## 2.3 폐쇄망 운용을 전제로 한 설계

일반 QGIS 플러그인과 GeoINT 플러그인의 가장 큰 차이는 **네트워크 가정**이다.

일반 플러그인은 인터넷 연결을 가정한다. 배경지도를 XYZ 타일로 불러오고,
`pip install`로 의존성을 해결하고, 온라인 지오코딩 API를 호출한다.
분석망에서는 이 중 어느 것도 성립하지 않는다.

### 설계 규칙

1. **런타임 네트워크 호출은 선택 기능으로 격리한다.**
   네트워크 없이도 핵심 분석은 전부 동작해야 한다.
2. **의존성은 `ext_libs/`에 벤더링한다.** (Chapter 6)
3. **배경지도는 로컬 MBTiles/GeoPackage 대체 경로를 제공한다.**
4. **좌표 변환은 PROJ 격자 파일 부재 상황을 처리한다.** (Chapter 10)
5. **AI 모델은 파일로 반입되며 다운로드하지 않는다.** (Chapter 15)

```python
# core/runtime.py
from qgis.core import QgsNetworkAccessManager, QgsSettings

def network_allowed() -> bool:
    """폐쇄망 모드에서는 모든 외부 호출을 차단한다."""
    return not QgsSettings().value("qgeoint/offline_mode", False, type=bool)


def require_network(feature_name: str) -> None:
    if not network_allowed():
        raise RuntimeError(
            f"'{feature_name}' 기능은 네트워크가 필요합니다. "
            f"오프라인 모드가 켜져 있어 실행할 수 없습니다."
        )
```

> **TIP**
> 오프라인 모드를 기본값 `True`로 두고 개발하면
> 무심코 들어간 네트워크 의존성을 개발 중에 즉시 발견할 수 있다.

## 2.4 프로파일 분리

분석 업무는 여러 사업·과제가 병행되는 경우가 많다.
QGIS 프로파일(`--profile` 옵션)로 과제별 환경을 분리하면
플러그인 설정, 처리 이력, 좌표계 즐겨찾기가 섞이지 않는다.

```bash
# Windows (OSGeo4W)
qgis-bin.exe --profile flood2026

# Linux
qgis --profile flood2026
```

프로파일 경로는 플러그인 내부에서 다음과 같이 조회한다.

```python
from qgis.core import QgsApplication
from pathlib import Path

def profile_dir() -> Path:
    return Path(QgsApplication.qgisSettingsDirPath())

def workspace_dir() -> Path:
    d = profile_dir() / "qgeoint_workspace"
    d.mkdir(parents=True, exist_ok=True)
    return d
```

### 버전 호환 노트 (Chapter 2)

- `QgsTemporalController`는 QGIS **3.14 이상**에서만 사용 가능하다. Chapter 11에서 폴백을 다룬다.
- `QgsApplication.qgisSettingsDirPath()`는 3.6부터 동일하게 동작한다.

---

# Chapter 3. 개발환경 구성과 버전 전략

## 3.1 다중 버전 개발 환경

3.6부터 4.x까지 지원하려면 **최소 세 개의 QGIS 설치본**이 필요하다.

| 역할 | 버전 | 용도 |
|---|---|---|
| 하한 검증기 | 3.6 또는 3.10 | 최소 지원 버전 회귀 확인 |
| 주력 개발기 | 3.34 또는 3.44 LTR | 일상 개발 |
| 상한 검증기 | 4.2 이상 | Qt6 및 신규 API 확인 |

표 3-1. 권장 다중 설치 구성

Windows에서는 OSGeo4W 네트워크 설치 관리자로 여러 버전을 병존시킬 수 있으나,
구버전(3.6~3.16)은 현재 OSGeo4W 저장소에서 내려가 있는 경우가 있다.
이때는 아카이브 독립 설치본 또는 Docker 이미지를 사용한다.

```bash
# Docker로 하한 버전 검증 (headless 테스트용)
docker run --rm -it \
  -v "$PWD":/plugin \
  qgis/qgis:release-3_10 \
  bash -c "xvfb-run -s '+extension GLX -screen 0 1024x768x24' \
           python3 -m pytest /plugin/tests -q"
```

> **WARNING**
> Docker 이미지의 Python 버전이 데스크톱 설치본과 다를 수 있다.
> QGIS 3.6은 Python 3.7, 3.28은 3.9, 4.x는 3.12를 번들하는 경우가 많다.
> 의존성 휠(wheel) 선택 시 이 차이가 결정적이다. (Chapter 6)

## 3.2 Python 버전 대응표

| QGIS | 번들 Python | Qt | 주요 제약 |
|---|---|---|---|
| 3.6 | 3.7 | Qt5 | f-string 사용 가능, `:=` 불가, `dataclasses` 사용 가능 |
| 3.10 | 3.7 / 3.8 | Qt5 | 동일 |
| 3.16 | 3.9 | Qt5 | `dict` 병합 연산자 사용 가능 |
| 3.28 | 3.9 | Qt5 | |
| 3.34 / 3.44 | 3.9 ~ 3.12 | Qt5 | |
| 4.x | 3.12 이상 | Qt6 | `match` 문 사용 가능 |

표 3-2. QGIS 버전별 Python 런타임

### 코딩 하한선 결정

이 책의 코드는 **Python 3.7 문법**을 하한으로 삼는다.

금지 문법:

- 대입 표현식 `:=` (3.8+)
- 위치 전용 매개변수 `/` (3.8+)
- `dict1 | dict2` (3.9+)
- `list[str]` 형태의 내장 제네릭 어노테이션 (3.9+)
- `match` 문 (3.10+)

허용 대안:

```python
# 나쁜 예 (3.9+ 전용)
def load(paths: list[str]) -> dict[str, int]:
    ...

# 좋은 예 (3.7 호환)
from typing import List, Dict

def load(paths: List[str]) -> Dict[str, int]:
    ...
```

> **TIP**
> `from __future__ import annotations`를 파일 상단에 두면
> 어노테이션 평가가 지연되어 3.7에서도 `list[str]` 표기를 쓸 수 있다.
> 다만 `typing.get_type_hints()`를 호출하는 코드에서는 여전히 실패하므로
> 이 책에서는 안전하게 `typing` 모듈 표기를 사용한다.

## 3.3 편집기 설정

VS Code에서 PyQGIS 자동완성을 활성화하려면 QGIS의 Python 경로를 지정한다.

```jsonc
// .vscode/settings.json
{
  "python.defaultInterpreterPath": "C:/OSGeo4W/apps/Python39/python.exe",
  "python.analysis.extraPaths": [
    "C:/OSGeo4W/apps/qgis-ltr/python",
    "C:/OSGeo4W/apps/qgis-ltr/python/plugins"
  ],
  "python.analysis.typeCheckingMode": "basic",
  "files.exclude": { "**/__pycache__": true }
}
```

## 3.4 개발 반복 루프

```text
코드 수정 (VS Code)
     ↓
저장
     ↓
QGIS — Plugin Reloader (Ctrl+F5)
     ↓
동작 확인 / 로그 확인
     ↓
pytest (headless)
     ↓
git commit
```

그림 3-1. 개발 반복 루프

Plugin Reloader는 QGIS 재시작 없이 플러그인을 다시 불러온다.
다만 다음 경우에는 재시작이 필요하다.

- `ext_libs`에 새 패키지를 추가했을 때 (`sys.path` 캐시)
- C 확장 모듈을 교체했을 때
- `metadata.txt`의 의존성 항목을 변경했을 때

## 3.5 심볼릭 링크로 개발본 연결

```powershell
# Windows (관리자 PowerShell)
New-Item -ItemType SymbolicLink `
  -Path "$env:APPDATA\QGIS\QGIS3\profiles\default\python\plugins\qgeoint" `
  -Target "D:\dev\qgeoint\qgeoint"
```

```bash
# Linux / macOS
ln -s ~/dev/qgeoint/qgeoint \
      ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/qgeoint
```

이렇게 하면 저장소에서 바로 편집하고 QGIS에서 즉시 확인할 수 있다.

### 버전 호환 노트 (Chapter 3)

- QGIS 4.x의 프로파일 경로는 `QGIS3` 대신 `QGIS4`를 사용한다.
  스크립트에서 경로를 하드코딩하지 말고 `QgsApplication.qgisSettingsDirPath()`로 조회한다.
- Plugin Reloader는 3.6~4.x 전 구간에서 동작하나, QGIS 4용 버전을 별도로 설치해야 한다.

---
---

# Part II. 플러그인 골격과 호환성 계층

---

# Chapter 4. QGeoINT 플러그인 골격

## 4.1 최소 골격

QGIS 플러그인은 다음 세 요소만 있으면 로드된다.

```text
qgeoint/
├── __init__.py      ← classFactory 제공
├── metadata.txt     ← QGIS가 읽는 명세
└── plugin.py        ← 실제 구현
```

### `__init__.py`

```python
# qgeoint/__init__.py
def classFactory(iface):
    """QGIS가 플러그인을 로드할 때 호출하는 진입점."""
    from .plugin import QGeoINTPlugin
    return QGeoINTPlugin(iface)
```

> **WARNING**
> `__init__.py` 최상단에서 무거운 모듈(numpy, GDAL 등)을 import 하지 않는다.
> QGIS 시작 시간이 직접적으로 늘어나고, import 실패 시 플러그인 목록 자체가 깨진다.
> 반드시 `classFactory` 내부에서 지연 import 한다.

### `metadata.txt`

```ini
[general]
name=QGeoINT
qgisMinimumVersion=3.6
qgisMaximumVersion=4.99
description=지리공간정보 분석(GeoINT) 워크플로 도구 모음
about=영상 변화탐지, 지형 분석, 다중출처 융합, 브리핑 산출물 자동화를 제공합니다.
version=0.1.0
author=Bim
email=author@example.org

tracker=https://example.org/qgeoint/issues
repository=https://example.org/qgeoint
homepage=https://example.org/qgeoint

category=Plugins
icon=resources/icon.svg
experimental=False
deprecated=False

tags=geoint,remote sensing,change detection,terrain,osint
hasProcessingProvider=yes

changelog=0.1.0
    - 최초 골격
```

핵심 항목 해설:

| 항목 | 주의점 |
|---|---|
| `qgisMinimumVersion` | 3.6으로 두면 3.6 미만에서는 설치가 차단된다 |
| `qgisMaximumVersion` | **반드시 명시한다.** 생략하면 3.99가 기본값이 되어 QGIS 4에서 설치되지 않는다 |
| `hasProcessingProvider` | `yes`로 두면 QGIS가 Processing Provider 등록을 기대한다 (Chapter 25) |
| `category` | `Plugins`, `Raster`, `Vector`, `Database`, `Web` 중 선택 |

표 4-1. `metadata.txt` 주요 항목

> **호환성**
> QGIS 4 대응에서 가장 흔한 실수가 `qgisMaximumVersion` 누락이다.
> 3.x 시절 작성된 플러그인이 QGIS 4 저장소에 나타나지 않는 이유의 대부분이 이것이다.

## 4.2 플러그인 본체

```python
# qgeoint/plugin.py
from __future__ import annotations

import os
from typing import List, Optional

from qgis.PyQt.QtWidgets import QAction, QToolBar
from qgis.PyQt.QtGui import QIcon
from qgis.core import QgsApplication, QgsMessageLog, Qgis

from .compat import qgis_version_int, add_action_shortcut

PLUGIN_DIR = os.path.dirname(__file__)
LOG_TAG = "QGeoINT"


class QGeoINTPlugin:
    def __init__(self, iface):
        self.iface = iface
        self.actions: List[QAction] = []
        self.toolbar: Optional[QToolBar] = None
        self.provider = None

    # ------------------------------------------------------------------
    # 생명주기
    # ------------------------------------------------------------------
    def initGui(self) -> None:
        self.toolbar = self.iface.addToolBar("QGeoINT")
        self.toolbar.setObjectName("QGeoINTToolbar")

        self._add_action(
            "resources/icon_ingest.svg",
            "데이터 인제스트",
            self.on_ingest,
        )
        self._add_action(
            "resources/icon_change.svg",
            "변화 탐지",
            self.on_change_detection,
        )
        self._add_action(
            "resources/icon_report.svg",
            "브리핑 생성",
            self.on_report,
        )

        self._init_processing_provider()
        self.log(f"QGeoINT 로드 완료 (QGIS {qgis_version_int()})")

    def unload(self) -> None:
        for action in self.actions:
            self.iface.removePluginMenu("&QGeoINT", action)
            self.iface.removeToolBarIcon(action)
        self.actions.clear()

        if self.toolbar is not None:
            del self.toolbar
            self.toolbar = None

        if self.provider is not None:
            QgsApplication.processingRegistry().removeProvider(self.provider)
            self.provider = None

    # ------------------------------------------------------------------
    # 내부 헬퍼
    # ------------------------------------------------------------------
    def _add_action(self, icon_rel: str, text: str, callback) -> QAction:
        icon = QIcon(os.path.join(PLUGIN_DIR, icon_rel))
        action = QAction(icon, text, self.iface.mainWindow())
        action.triggered.connect(callback)
        self.toolbar.addAction(action)
        self.iface.addPluginToMenu("&QGeoINT", action)
        self.actions.append(action)
        return action

    def _init_processing_provider(self) -> None:
        from .processing.provider import QGeoINTProvider
        self.provider = QGeoINTProvider()
        QgsApplication.processingRegistry().addProvider(self.provider)

    @staticmethod
    def log(msg: str, level=Qgis.Info) -> None:
        QgsMessageLog.logMessage(msg, LOG_TAG, level)

    # ------------------------------------------------------------------
    # 콜백 (각 장에서 구현)
    # ------------------------------------------------------------------
    def on_ingest(self) -> None:
        from .gui.ingest_dialog import IngestDialog
        IngestDialog(self.iface.mainWindow()).exec_()

    def on_change_detection(self) -> None:
        from .gui.change_dialog import ChangeDialog
        ChangeDialog(self.iface.mainWindow()).exec_()

    def on_report(self) -> None:
        from .production.briefing import run_briefing_wizard
        run_briefing_wizard(self.iface)
```

## 4.3 `unload()`를 제대로 쓰는 이유

`unload()`는 단순한 정리 함수가 아니다. 다음이 누락되면 실무에서 실제 문제가 발생한다.

| 누락 항목 | 증상 |
|---|---|
| 액션 제거 누락 | Plugin Reloader 후 메뉴 항목 중복 누적 |
| Provider 제거 누락 | Processing 툴박스에 같은 알고리즘이 여러 번 표시 |
| 시그널 연결 해제 누락 | 언로드 후에도 콜백이 호출되어 `RuntimeError: wrapped C/C++ object deleted` |
| 실행 중 태스크 미취소 | QGIS 종료 시 크래시 |
| Map Tool 미해제 | 마우스 커서가 이전 도구에 고정 |

표 4-2. `unload()` 누락 시 증상

```python
def unload(self) -> None:
    # 실행 중인 태스크 정리
    from .tasks.registry import cancel_all_tasks
    cancel_all_tasks()

    # 맵툴 해제
    if getattr(self, "map_tool", None) is not None:
        self.iface.mapCanvas().unsetMapTool(self.map_tool)
        self.map_tool = None

    # 프로젝트 시그널 해제
    try:
        QgsProject.instance().layersAdded.disconnect(self._on_layers_added)
    except (TypeError, RuntimeError):
        pass
    ...
```

> **TIP**
> 시그널 해제는 `try/except (TypeError, RuntimeError)`로 감싼다.
> 연결된 적이 없거나 이미 파괴된 객체에 대한 `disconnect()`는 예외를 던진다.

### 버전 호환 노트 (Chapter 4)

- `QDialog.exec_()`는 Qt6에서 `exec()`로 바뀌었다. Chapter 5의 `compat.py`에서 흡수한다.
- `Qgis.Info`는 4.x에서 `Qgis.MessageLevel.Info`로 이동했으나 하위 별칭이 유지된다.
  단 IDE 경고를 피하려면 `compat.py`에 상수를 재수출하는 편이 낫다.

---

# Chapter 5. compat.py — Qt5/Qt6 · QGIS 3/4 호환 계층

## 5.1 왜 별도 모듈이 필요한가

`qgis.PyQt`가 Qt 바인딩 차이를 상당 부분 흡수해 주지만,
그것만으로는 3.6~4.x 전 구간을 덮을 수 없다. 남는 차이는 세 종류다.

1. **Qt6에서 사라진 메서드** — `exec_()`, `QDesktopWidget` 등
2. **enum 스코프 강화** — Qt6는 `Qt.AlignLeft` 대신 `Qt.AlignmentFlag.AlignLeft`를 요구
3. **QGIS API 자체의 변경** — 클래스 이동, 시그니처 변경, 신규 API 추가

이 세 가지를 코드 곳곳에서 `if` 분기로 처리하면 유지보수가 불가능해진다.
**모든 분기를 `compat.py` 한 곳에 가둔다.** 이것이 이 시리즈의 일관된 전략이다.

> **ENGINEERING PRACTICE**
> 규칙: `compat.py` 이외의 파일에서 `QGIS_VERSION_INT`를 참조하면 코드 리뷰에서 반려한다.
> 버전 분기는 반드시 이 모듈이 제공하는 함수·상수를 통해서만 노출한다.

## 5.2 전체 구현

```python
# qgeoint/compat.py
"""QGIS 3.6 ~ 4.x / Qt5 ~ Qt6 호환 계층.

이 모듈 밖에서는 버전 분기를 하지 않는다.
"""
from __future__ import annotations

from typing import Any, Callable, Optional

from qgis.core import Qgis
from qgis.PyQt import QtCore, QtGui, QtWidgets
from qgis.PyQt.QtCore import QT_VERSION_STR

# ----------------------------------------------------------------------
# 버전 판별
# ----------------------------------------------------------------------
QGIS_VERSION_INT = Qgis.QGIS_VERSION_INT          # 예: 30616, 40201
QT6 = QT_VERSION_STR.startswith("6")
QGIS4 = QGIS_VERSION_INT >= 40000


def qgis_version_int() -> int:
    return QGIS_VERSION_INT


def at_least(major: int, minor: int) -> bool:
    """QGIS 버전이 major.minor 이상인지 확인."""
    return QGIS_VERSION_INT >= (major * 10000 + minor * 100)


# ----------------------------------------------------------------------
# 1. 다이얼로그 실행
# ----------------------------------------------------------------------
def exec_dialog(dialog: "QtWidgets.QDialog") -> int:
    """Qt5의 exec_() / Qt6의 exec() 차이를 흡수."""
    runner = getattr(dialog, "exec", None) or getattr(dialog, "exec_")
    return runner()


# ----------------------------------------------------------------------
# 2. enum 접근
# ----------------------------------------------------------------------
def _enum(owner: Any, scope: str, name: str) -> Any:
    """Qt6의 스코프드 enum과 Qt5의 평면 enum을 모두 지원."""
    scoped = getattr(owner, scope, None)
    if scoped is not None and hasattr(scoped, name):
        return getattr(scoped, name)
    return getattr(owner, name)


Qt = QtCore.Qt

ALIGN_LEFT = _enum(Qt, "AlignmentFlag", "AlignLeft")
ALIGN_RIGHT = _enum(Qt, "AlignmentFlag", "AlignRight")
ALIGN_CENTER = _enum(Qt, "AlignmentFlag", "AlignCenter")
ALIGN_VCENTER = _enum(Qt, "AlignmentFlag", "AlignVCenter")

KEY_ESCAPE = _enum(Qt, "Key", "Key_Escape")
KEY_RETURN = _enum(Qt, "Key", "Key_Return")

CURSOR_CROSS = _enum(Qt, "CursorShape", "CrossCursor")
CURSOR_WAIT = _enum(Qt, "CursorShape", "WaitCursor")

MOUSE_LEFT = _enum(Qt, "MouseButton", "LeftButton")
MOUSE_RIGHT = _enum(Qt, "MouseButton", "RightButton")

CHECKED = _enum(Qt, "CheckState", "Checked")
UNCHECKED = _enum(Qt, "CheckState", "Unchecked")

ITEM_SELECTABLE = _enum(Qt, "ItemFlag", "ItemIsSelectable")
ITEM_ENABLED = _enum(Qt, "ItemFlag", "ItemIsEnabled")

USER_ROLE = _enum(Qt, "ItemDataRole", "UserRole")
DISPLAY_ROLE = _enum(Qt, "ItemDataRole", "DisplayRole")


# ----------------------------------------------------------------------
# 3. 지오메트리 타입 / 레이어 타입 enum
# ----------------------------------------------------------------------
try:                                   # QGIS 3.30+
    from qgis.core import QgsWkbTypes
    GEOM_POINT = Qgis.GeometryType.Point
    GEOM_LINE = Qgis.GeometryType.Line
    GEOM_POLYGON = Qgis.GeometryType.Polygon
except AttributeError:                 # QGIS 3.6 ~ 3.28
    from qgis.core import QgsWkbTypes
    GEOM_POINT = QgsWkbTypes.PointGeometry
    GEOM_LINE = QgsWkbTypes.LineGeometry
    GEOM_POLYGON = QgsWkbTypes.PolygonGeometry


# ----------------------------------------------------------------------
# 4. 메시지 레벨
# ----------------------------------------------------------------------
try:
    LEVEL_INFO = Qgis.MessageLevel.Info
    LEVEL_WARNING = Qgis.MessageLevel.Warning
    LEVEL_CRITICAL = Qgis.MessageLevel.Critical
except AttributeError:
    LEVEL_INFO = Qgis.Info
    LEVEL_WARNING = Qgis.Warning
    LEVEL_CRITICAL = Qgis.Critical


# ----------------------------------------------------------------------
# 5. 화면 정보 (QDesktopWidget 제거 대응)
# ----------------------------------------------------------------------
def screen_geometry() -> "QtCore.QRect":
    app = QtWidgets.QApplication.instance()
    screen = app.primaryScreen()
    if screen is not None:
        return screen.availableGeometry()
    return QtCore.QRect(0, 0, 1280, 800)


# ----------------------------------------------------------------------
# 6. 단축키 등록
# ----------------------------------------------------------------------
def add_action_shortcut(action: "QtWidgets.QAction", sequence: str) -> None:
    action.setShortcut(QtGui.QKeySequence(sequence))


# ----------------------------------------------------------------------
# 7. 좌표변환 컨텍스트 (3.8+에서 도입)
# ----------------------------------------------------------------------
def transform_context():
    from qgis.core import QgsProject
    project = QgsProject.instance()
    getter = getattr(project, "transformContext", None)
    if getter is not None:
        return getter()
    return None


def make_transform(src_crs, dst_crs):
    """QgsCoordinateTransform을 버전 무관하게 생성."""
    from qgis.core import QgsCoordinateTransform, QgsProject
    ctx = transform_context()
    if ctx is not None:
        return QgsCoordinateTransform(src_crs, dst_crs, ctx)
    return QgsCoordinateTransform(src_crs, dst_crs, QgsProject.instance())


# ----------------------------------------------------------------------
# 8. 임시 레이어 URI 생성
# ----------------------------------------------------------------------
def memory_uri(geometry: str, crs_authid: str, fields: str = "") -> str:
    base = "{0}?crs={1}".format(geometry, crs_authid)
    if fields:
        base += "&" + fields
    base += "&index=yes"
    return base
```

## 5.3 사용 예

호환 계층이 있으면 실제 코드는 버전을 의식하지 않는다.

```python
from .compat import exec_dialog, ALIGN_CENTER, GEOM_POLYGON, make_transform

label.setAlignment(ALIGN_CENTER)
result = exec_dialog(dialog)
transform = make_transform(src_crs, dst_crs)
```

## 5.4 호환 계층 자체를 테스트하기

`compat.py`는 버전마다 다른 경로를 타므로 **가장 먼저 테스트해야 할 모듈**이다.

```python
# tests/test_compat.py
import pytest
from qgeoint import compat


def test_enum_constants_resolve():
    """모든 재수출 상수가 None이 아니어야 한다."""
    names = [n for n in dir(compat) if n.isupper()]
    assert names, "재수출된 상수가 없다"
    for name in names:
        assert getattr(compat, name) is not None, f"{name} 해석 실패"


def test_version_helpers():
    assert compat.qgis_version_int() > 30000
    assert compat.at_least(3, 6) is True


def test_transform_creation():
    from qgis.core import QgsCoordinateReferenceSystem as CRS
    t = compat.make_transform(CRS("EPSG:4326"), CRS("EPSG:5186"))
    assert t.isValid()
```

이 테스트를 CI 매트릭스(3.6 / LTR / 4.x)에서 모두 통과시키는 것이
Part VIII에서 다룰 **버전 호환 게이트**의 1차 관문이다.

### 버전 호환 노트 (Chapter 5)

- `Qgis.QGIS_VERSION_INT`는 3.6부터 4.x까지 동일하게 제공된다.
- `Qgis.GeometryType`은 3.30에서 도입되었다. 그 이전은 `QgsWkbTypes.*Geometry`.
- Qt6에서 `QDesktopWidget`은 완전히 제거되었으므로 `QScreen`을 사용해야 한다.

---

# Chapter 6. ext_libs — 폐쇄망 의존성 벤더링

## 6.1 문제 정의

QGeoINT는 다음 외부 패키지를 필요로 한다.

| 패키지 | 용도 | 필수 여부 |
|---|---|---|
| `numpy` | 래스터 배열 연산 | QGIS 번들 (설치 불필요) |
| `osgeo.gdal` | 영상 I/O | QGIS 번들 |
| `mgrs` 또는 자체 구현 | MGRS 격자 변환 | 벤더링 |
| `pyproj` | 측지 계산 보조 | QGIS 번들(3.10+) / 3.6은 부재 가능 |
| `onnxruntime` | AI 추론 | 벤더링 (선택) |
| `python-pptx` | 브리핑 산출 | 벤더링 (선택) |

표 6-1. QGeoINT 의존성 목록

분석망에서는 `pip install`을 실행할 수 없으므로,
필요한 패키지를 플러그인 ZIP 안에 함께 넣어야 한다. 이를 **벤더링(vendoring)** 이라 한다.

## 6.2 디렉터리 전략

Python 버전이 QGIS 버전마다 다르므로 단일 디렉터리로는 부족하다.

```text
qgeoint/ext_libs/
├── common/          ← 순수 Python, 버전 무관
│   ├── mgrs/
│   └── pptx/
├── py37/            ← QGIS 3.6 ~ 3.10 (Python 3.7)
│   └── onnxruntime/
├── py39/            ← QGIS 3.16 ~ 3.34
│   └── onnxruntime/
└── py312/           ← QGIS 4.x
    └── onnxruntime/
```

그림 6-1. Python ABI별 분기 구조

> **WARNING**
> C 확장을 포함하는 패키지(`onnxruntime`, `shapely`, `rasterio` 등)는
> Python 마이너 버전 + 플랫폼별로 휠이 다르다.
> `cp37-win_amd64` 휠을 Python 3.12에서 import하면 즉시 실패한다.
> 순수 Python 패키지만 `common/`에 둔다.

## 6.3 경로 주입

```python
# qgeoint/ext_libs/__init__.py
"""벤더링된 의존성을 sys.path에 등록한다."""
from __future__ import annotations

import os
import sys
import platform

_HERE = os.path.dirname(__file__)


def _abi_dir() -> str:
    major, minor = sys.version_info[:2]
    return "py{0}{1}".format(major, minor)


def _candidates():
    yield os.path.join(_HERE, "common")
    yield os.path.join(_HERE, _abi_dir())
    # 근접 ABI 폴백 (3.8은 py37 휠이 대체로 동작)
    fallback = {"py38": "py37", "py310": "py39", "py311": "py39"}
    alt = fallback.get(_abi_dir())
    if alt:
        yield os.path.join(_HERE, alt)


_INSTALLED = False


def install() -> None:
    """플러그인 로드 시 1회 호출."""
    global _INSTALLED
    if _INSTALLED:
        return
    for path in _candidates():
        if os.path.isdir(path) and path not in sys.path:
            sys.path.insert(0, path)
    _INSTALLED = True


def report() -> str:
    return "Python {0}.{1} / {2} / ABI dir: {3}".format(
        sys.version_info[0], sys.version_info[1],
        platform.machine(), _abi_dir()
    )
```

`plugin.py`의 `initGui()` 최상단에서 호출한다.

```python
def initGui(self) -> None:
    from .ext_libs import install as install_ext_libs, report
    install_ext_libs()
    self.log("의존성 경로 등록: " + report())
    ...
```

## 6.4 선택적 의존성의 우아한 실패

필수가 아닌 패키지는 없어도 플러그인 전체가 죽으면 안 된다.

```python
# core/optional.py
from typing import Optional, Any


def try_import(module_name: str) -> Optional[Any]:
    try:
        return __import__(module_name, fromlist=["__name__"])
    except Exception:
        return None


class FeatureGate:
    """선택 기능의 가용 여부를 한곳에서 관리."""

    def __init__(self):
        self._cache = {}

    def available(self, module_name: str) -> bool:
        if module_name not in self._cache:
            self._cache[module_name] = try_import(module_name) is not None
        return self._cache[module_name]

    def require(self, module_name: str, feature: str) -> None:
        if not self.available(module_name):
            raise RuntimeError(
                "'{0}' 기능에는 {1} 패키지가 필요합니다. "
                "ext_libs에 포함되어 있는지 확인하세요.".format(feature, module_name)
            )


GATES = FeatureGate()
```

GUI에서는 사용 불가한 기능의 버튼을 비활성화하고 이유를 툴팁으로 표시한다.

```python
from .core.optional import GATES

btn_ai = self.ui.btnRunInference
if not GATES.available("onnxruntime"):
    btn_ai.setEnabled(False)
    btn_ai.setToolTip("onnxruntime이 설치되어 있지 않아 AI 추론을 사용할 수 없습니다.")
```

## 6.5 라이선스 관리

벤더링은 **재배포**다. 따라서 각 패키지의 라이선스를 반드시 동봉한다.

```text
qgeoint/ext_libs/LICENSES/
├── mgrs-MIT.txt
├── python-pptx-MIT.txt
└── onnxruntime-MIT.txt
```

그리고 `THIRD_PARTY.md`에 목록을 정리한다.

| 패키지 | 버전 | 라이선스 | 출처 |
|---|---|---|---|
| mgrs | 1.4.x | MIT | PyPI |
| python-pptx | 0.6.x | MIT | PyPI |
| onnxruntime | 1.x | MIT | PyPI |

표 6-2. 재배포 의존성 라이선스

> **WARNING**
> GPL 계열 패키지를 벤더링하면 플러그인 전체의 라이선스 의무가 달라질 수 있다.
> QGIS 자체가 GPL v2+ 이므로 플러그인도 GPL 호환 라이선스를 택하는 편이 안전하다.
> 사내 비공개 배포라면 문제가 적지만, 공개 저장소 등록 시에는 반드시 검토한다.

### 버전 호환 노트 (Chapter 6)

- QGIS 3.6의 번들 Python은 3.7이며 `pyproj`가 포함되지 않은 배포본이 있다.
  측지 계산은 가능하면 `QgsDistanceArea`로 대체한다.
- QGIS 4.x는 Python 3.12 기반이므로 `py312` 휠이 필요하다.
  구버전 휠만 준비했다면 AI 기능은 자동 비활성화된다.

---

# Chapter 7. 플러그인 아키텍처 설계

## 7.1 계층 분리

GeoINT 플러그인은 기능이 빠르게 늘어난다. 초기에 계층을 나누지 않으면
1년 뒤 `plugin.py`가 3천 줄이 된다.

```text
gui/         ← 위젯, 다이얼로그. 로직 없음
   ↓ 호출
services/    ← 유스케이스 조합. 트랜잭션 경계
   ↓ 호출
core/        ← 순수 도메인 로직. QGIS GUI 의존 없음
   ↓ 사용
repositories/← 레이어·파일 입출력
```

그림 7-1. 계층 의존 방향

**의존은 한 방향으로만 흐른다.** `core`가 `gui`를 import 하면 설계 오류다.

## 7.2 core는 QGIS GUI를 모른다

이 규칙의 실질적 이익은 **테스트 가능성**이다.
`core`가 `iface`나 위젯을 참조하지 않으면 headless 환경에서 그대로 테스트할 수 있다.

```python
# core/change.py  — GUI 의존 없음
from __future__ import annotations
from dataclasses import dataclass
from typing import Optional
import numpy as np


@dataclass
class ChangeParams:
    threshold: float = 0.15
    min_area_px: int = 25
    smooth_radius: int = 1


@dataclass
class ChangeResult:
    mask: np.ndarray
    changed_px: int
    total_px: int

    @property
    def changed_ratio(self) -> float:
        return self.changed_px / float(self.total_px) if self.total_px else 0.0


def detect_change(before: np.ndarray,
                  after: np.ndarray,
                  params: ChangeParams,
                  nodata_mask: Optional[np.ndarray] = None) -> ChangeResult:
    """두 정규화 배열의 차분 기반 변화 탐지. 순수 함수."""
    if before.shape != after.shape:
        raise ValueError("입력 배열의 크기가 다릅니다: "
                         "{0} vs {1}".format(before.shape, after.shape))

    diff = np.abs(after.astype("float32") - before.astype("float32"))
    mask = diff >= params.threshold

    if nodata_mask is not None:
        mask &= ~nodata_mask

    total = int(mask.size)
    if nodata_mask is not None:
        total = int((~nodata_mask).sum())

    return ChangeResult(mask=mask, changed_px=int(mask.sum()), total_px=total)
```

이 함수는 QGIS 없이도 테스트된다.

```python
# tests/test_change_core.py
import numpy as np
from qgeoint.core.change import detect_change, ChangeParams


def test_detects_uniform_shift():
    before = np.zeros((10, 10), dtype="float32")
    after = np.full((10, 10), 0.5, dtype="float32")
    r = detect_change(before, after, ChangeParams(threshold=0.2))
    assert r.changed_px == 100
    assert r.changed_ratio == 1.0


def test_respects_nodata():
    before = np.zeros((4, 4), dtype="float32")
    after = np.ones((4, 4), dtype="float32")
    nodata = np.zeros((4, 4), dtype=bool)
    nodata[0, :] = True
    r = detect_change(before, after, ChangeParams(threshold=0.5), nodata)
    assert r.changed_px == 12
    assert r.total_px == 12
```

> **ENGINEERING PRACTICE**
> `core/` 디렉터리의 어떤 파일에도 `from qgis.PyQt` import가 없어야 한다.
> CI에서 이를 자동 검사할 수 있다 (Chapter 28).

## 7.3 서비스 계층

서비스는 "하나의 사용자 의도"를 처리한다.

```python
# services/change_service.py
from __future__ import annotations
from typing import Optional

from qgis.core import QgsRasterLayer, QgsProject

from ..core.change import detect_change, ChangeParams
from ..core.provenance import Provenance, attach_provenance
from ..core.recipe import Recipe
from ..repositories.raster_repo import read_band, write_mask_raster


class ChangeDetectionService:
    def __init__(self, project: Optional[QgsProject] = None):
        self.project = project or QgsProject.instance()

    def run(self,
            before_layer: QgsRasterLayer,
            after_layer: QgsRasterLayer,
            params: ChangeParams,
            output_path: str) -> QgsRasterLayer:

        before, meta_b = read_band(before_layer, band=1)
        after, meta_a = read_band(after_layer, band=1)

        result = detect_change(before, after, params)

        out_layer = write_mask_raster(result.mask, meta_b, output_path)

        recipe = Recipe(
            operation="change_detection",
            inputs={
                "before": before_layer.source(),
                "after": after_layer.source(),
            },
            params={
                "threshold": params.threshold,
                "min_area_px": params.min_area_px,
            },
        )
        attach_provenance(out_layer, Provenance.from_recipe(recipe))
        self.project.addMapLayer(out_layer)
        return out_layer
```

서비스가 담당하는 것:

- 입출력 조립
- 도메인 함수 호출
- **출처 기록** (원칙 2)
- **레시피 직렬화** (원칙 1)
- 프로젝트 등록

## 7.4 출처(Provenance) 모듈

```python
# core/provenance.py
from __future__ import annotations

import json
import getpass
import socket
from dataclasses import dataclass, asdict, field
from datetime import datetime
from typing import Any, Dict, List, Optional

PROP_KEY = "qgeoint/provenance"


@dataclass
class Provenance:
    operation: str
    created_at: str
    created_by: str
    host: str
    inputs: Dict[str, str] = field(default_factory=dict)
    params: Dict[str, Any] = field(default_factory=dict)
    confidence: Optional[str] = None     # Admiralty 코드 (예: "B2")
    notes: List[str] = field(default_factory=list)
    parent_ids: List[str] = field(default_factory=list)

    @classmethod
    def from_recipe(cls, recipe, confidence: Optional[str] = None) -> "Provenance":
        return cls(
            operation=recipe.operation,
            created_at=datetime.now().isoformat(timespec="seconds"),
            created_by=getpass.getuser(),
            host=socket.gethostname(),
            inputs=dict(recipe.inputs),
            params=dict(recipe.params),
            confidence=confidence,
        )

    def to_json(self) -> str:
        return json.dumps(asdict(self), ensure_ascii=False, indent=2)

    @classmethod
    def from_json(cls, text: str) -> "Provenance":
        return cls(**json.loads(text))


def attach_provenance(layer, prov: Provenance) -> None:
    layer.setCustomProperty(PROP_KEY, prov.to_json())


def read_provenance(layer) -> Optional[Provenance]:
    raw = layer.customProperty(PROP_KEY)
    if not raw:
        return None
    try:
        return Provenance.from_json(raw)
    except Exception:
        return None
```

> **TIP**
> `setCustomProperty()`로 저장한 값은 QGIS 프로젝트(.qgz) 안에 함께 저장된다.
> 프로젝트 파일만 전달해도 출처가 따라간다.
> 다만 레이어를 GeoPackage로 내보낼 때는 별도로 메타데이터에 기록해야 한다 (Chapter 24).

## 7.5 레시피 직렬화

```python
# core/recipe.py
from __future__ import annotations

import json
import hashlib
from dataclasses import dataclass, field, asdict
from typing import Any, Dict

SCHEMA_VERSION = 1


@dataclass
class Recipe:
    operation: str
    inputs: Dict[str, str] = field(default_factory=dict)
    params: Dict[str, Any] = field(default_factory=dict)
    schema: int = SCHEMA_VERSION

    def fingerprint(self) -> str:
        """동일 입력·파라미터면 동일 지문. 재현성 검증에 사용."""
        payload = json.dumps(asdict(self), sort_keys=True, ensure_ascii=False)
        return hashlib.sha256(payload.encode("utf-8")).hexdigest()[:16]

    def save(self, path: str) -> None:
        with open(path, "w", encoding="utf-8") as f:
            json.dump(asdict(self), f, ensure_ascii=False, indent=2)

    @classmethod
    def load(cls, path: str) -> "Recipe":
        with open(path, "r", encoding="utf-8") as f:
            data = json.load(f)
        if data.get("schema", 1) > SCHEMA_VERSION:
            raise ValueError("이 레시피는 더 최신 버전의 QGeoINT가 필요합니다.")
        return cls(**data)
```

`fingerprint()`는 회귀 테스트에서 유용하다.
같은 레시피를 두 번 실행했을 때 산출물 해시가 같은지 확인하면 재현성을 자동 검증할 수 있다.

### 버전 호환 노트 (Chapter 7)

- `dataclasses`는 Python 3.7부터 표준이므로 3.6 QGIS(Python 3.7)에서도 사용 가능하다.
- `datetime.isoformat(timespec=...)`은 Python 3.6+에서 지원된다.
- `QgsMapLayer.setCustomProperty()`는 3.6~4.x 전 구간 동일하다.

---
---

# Part III. 데이터 인제스트와 정규화

---

# Chapter 8. 영상 데이터 인제스트

## 8.1 인제스트의 정의

인제스트는 "파일을 여는 것"이 아니다. 다음 다섯 가지를 모두 수행해야 인제스트가 끝난다.

1. 포맷 판별 및 유효성 검사
2. 좌표계 확정 (미정의 시 사용자 확인)
3. 메타데이터 추출 (촬영일시, 센서, 해상도, 밴드 구성)
4. 카탈로그 등록 (검색 가능한 형태로)
5. 출처 정보 부착

## 8.2 다루는 포맷

| 포맷 | 확장자 | 특징 | 주의점 |
|---|---|---|---|
| GeoTIFF | `.tif` | 표준 | 압축·타일링 여부 확인 |
| COG | `.tif` | 클라우드 최적화 | 오버뷰 내장 필수 |
| JPEG2000 | `.jp2` | 위성 배포 표준 | GDAL 드라이버 의존 |
| NITF | `.ntf` | 영상 배포 컨테이너 | 메타데이터가 TRE에 존재 |
| ECW / MrSID | `.ecw`, `.sid` | 항공영상 압축 | 라이선스 제약 |
| HDF5 / NetCDF | `.h5`, `.nc` | 다차원 과학 자료 | 서브데이터셋 지정 필요 |

표 8-1. GeoINT에서 자주 다루는 영상 포맷

> **WARNING**
> ECW와 MrSID 드라이버는 배포판에 따라 포함되지 않는다.
> 플러그인은 드라이버 부재를 감지해 안내 메시지를 띄워야 한다.

## 8.3 드라이버 가용성 점검

```python
# ingest/drivers.py
from osgeo import gdal
from typing import Dict, List

REQUIRED = ["GTiff", "JP2OpenJPEG", "HFA", "VRT"]
OPTIONAL = ["JP2ECW", "ECW", "MrSID", "NITF", "netCDF"]


def driver_report() -> Dict[str, List[str]]:
    available = set()
    for i in range(gdal.GetDriverCount()):
        available.add(gdal.GetDriver(i).ShortName)

    return {
        "missing_required": [d for d in REQUIRED if d not in available],
        "missing_optional": [d for d in OPTIONAL if d not in available],
        "available_count": [str(len(available))],
    }
```

## 8.4 메타데이터 추출

```python
# ingest/imagery.py
from __future__ import annotations

from dataclasses import dataclass, field
from datetime import datetime
from typing import Dict, List, Optional

from osgeo import gdal, osr


@dataclass
class ImageryMeta:
    path: str
    driver: str
    width: int
    height: int
    band_count: int
    crs_authid: Optional[str]
    pixel_size_x: float
    pixel_size_y: float
    nodata: Optional[float]
    acquired: Optional[str] = None
    sensor: Optional[str] = None
    cloud_cover: Optional[float] = None
    extra: Dict[str, str] = field(default_factory=dict)

    @property
    def gsd(self) -> float:
        """지상표본거리 근사값 (미터 단위 좌표계 가정)."""
        return (abs(self.pixel_size_x) + abs(self.pixel_size_y)) / 2.0


_DATE_KEYS = [
    "ACQUISITIONDATETIME", "TIFFTAG_DATETIME", "NITF_IDATIM",
    "ACQUISITION_DATE", "DATE_ACQUIRED", "SENSING_TIME",
]
_SENSOR_KEYS = ["SENSOR_ID", "NITF_ISORCE", "SPACECRAFT_ID", "PLATFORM"]
_CLOUD_KEYS = ["CLOUD_COVER", "CLOUDY_PIXEL_PERCENTAGE"]


def _pick(md: Dict[str, str], keys: List[str]) -> Optional[str]:
    upper = {k.upper(): v for k, v in md.items()}
    for k in keys:
        if k in upper and upper[k]:
            return upper[k]
    return None


def read_imagery_meta(path: str) -> ImageryMeta:
    ds = gdal.Open(path, gdal.GA_ReadOnly)
    if ds is None:
        raise IOError("영상을 열 수 없습니다: {0}".format(path))

    gt = ds.GetGeoTransform()
    md = {}
    for domain in (None, "IMAGERY", "TRE", "NITF_METADATA"):
        try:
            md.update(ds.GetMetadata(domain) or {})
        except Exception:
            pass

    authid = None
    wkt = ds.GetProjection()
    if wkt:
        srs = osr.SpatialReference()
        srs.ImportFromWkt(wkt)
        auth_name = srs.GetAuthorityName(None)
        auth_code = srs.GetAuthorityCode(None)
        if auth_name and auth_code:
            authid = "{0}:{1}".format(auth_name, auth_code)

    band1 = ds.GetRasterBand(1)
    cloud_raw = _pick(md, _CLOUD_KEYS)

    meta = ImageryMeta(
        path=path,
        driver=ds.GetDriver().ShortName,
        width=ds.RasterXSize,
        height=ds.RasterYSize,
        band_count=ds.RasterCount,
        crs_authid=authid,
        pixel_size_x=gt[1],
        pixel_size_y=gt[5],
        nodata=band1.GetNoDataValue(),
        acquired=_pick(md, _DATE_KEYS),
        sensor=_pick(md, _SENSOR_KEYS),
        cloud_cover=float(cloud_raw) if cloud_raw else None,
        extra={k: v for k, v in md.items() if len(v) < 200},
    )
    ds = None
    return meta
```

> **TIP**
> `ds = None`으로 명시적으로 닫는다. GDAL 데이터셋은 참조가 남아 있으면
> 파일 잠금이 유지되어 Windows에서 후속 쓰기 작업이 실패한다.

## 8.5 좌표계 미정의 영상 처리

실무에서 자주 만나는 문제다. 드론 정사영상이나 스캔 지도는 CRS가 없거나 잘못돼 있다.

```python
# ingest/crs_resolver.py
from typing import Optional
from qgis.core import QgsCoordinateReferenceSystem, QgsRasterLayer


CANDIDATE_KR = [
    ("EPSG:5186", "중부원점 (GRS80, Korea 2000)"),
    ("EPSG:5187", "동부원점"),
    ("EPSG:5185", "서부원점"),
    ("EPSG:5179", "UTM-K (통합원점)"),
    ("EPSG:4326", "WGS84 경위도"),
    ("EPSG:32652", "UTM Zone 52N"),
]


def guess_crs_by_extent(layer: QgsRasterLayer) -> Optional[str]:
    """좌표 범위로 좌표계를 추정한다. 확정이 아니라 '후보 제시'용."""
    ext = layer.extent()
    x, y = ext.center().x(), ext.center().y()

    if -180 <= x <= 180 and -90 <= y <= 90:
        return "EPSG:4326"
    if 100000 <= x <= 400000 and 100000 <= y <= 900000:
        return "EPSG:5186"      # 한국 평면직각 계열
    if 700000 <= x <= 1500000 and 1500000 <= y <= 2200000:
        return "EPSG:5179"      # UTM-K
    if 200000 <= x <= 800000 and 3500000 <= y <= 4500000:
        return "EPSG:32652"     # UTM 52N
    return None
```

> **WARNING**
> 좌표계 자동 추정은 **절대 무단 적용하면 안 된다.**
> 반드시 사용자에게 후보를 제시하고 확인을 받은 뒤,
> 그 선택을 출처 정보에 `"crs_assigned_by": "user"`로 기록한다.
> 자동으로 붙인 좌표계는 방어가능성 원칙을 위반한다.

## 8.6 인제스트 카탈로그

인제스트된 영상은 GeoPackage 테이블에 등록하여 검색 가능하게 만든다.

```python
# ingest/catalog.py
from __future__ import annotations
import sqlite3
from typing import List, Optional
from .imagery import ImageryMeta

DDL = """
CREATE TABLE IF NOT EXISTS qgeoint_catalog (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    path        TEXT NOT NULL UNIQUE,
    driver      TEXT,
    width       INTEGER,
    height      INTEGER,
    band_count  INTEGER,
    crs_authid  TEXT,
    gsd         REAL,
    acquired    TEXT,
    sensor      TEXT,
    cloud_cover REAL,
    ingested_at TEXT DEFAULT (datetime('now','localtime'))
);
CREATE INDEX IF NOT EXISTS idx_catalog_acquired ON qgeoint_catalog(acquired);
CREATE INDEX IF NOT EXISTS idx_catalog_sensor   ON qgeoint_catalog(sensor);
"""


class Catalog:
    def __init__(self, gpkg_path: str):
        self.path = gpkg_path

    def _conn(self):
        return sqlite3.connect(self.path)

    def init(self) -> None:
        with self._conn() as c:
            c.executescript(DDL)

    def register(self, meta: ImageryMeta) -> None:
        with self._conn() as c:
            c.execute(
                """INSERT OR REPLACE INTO qgeoint_catalog
                   (path, driver, width, height, band_count, crs_authid,
                    gsd, acquired, sensor, cloud_cover)
                   VALUES (?,?,?,?,?,?,?,?,?,?)""",
                (meta.path, meta.driver, meta.width, meta.height,
                 meta.band_count, meta.crs_authid, meta.gsd,
                 meta.acquired, meta.sensor, meta.cloud_cover),
            )

    def find(self, sensor: Optional[str] = None,
             max_cloud: Optional[float] = None) -> List[tuple]:
        sql = "SELECT path, acquired, sensor, gsd, cloud_cover FROM qgeoint_catalog WHERE 1=1"
        args = []
        if sensor:
            sql += " AND sensor = ?"
            args.append(sensor)
        if max_cloud is not None:
            sql += " AND (cloud_cover IS NULL OR cloud_cover <= ?)"
            args.append(max_cloud)
        sql += " ORDER BY acquired DESC"
        with self._conn() as c:
            return c.execute(sql, args).fetchall()
```

### 버전 호환 노트 (Chapter 8)

- GDAL 버전에 따라 NITF TRE 메타데이터 도메인 이름이 다르다.
  `GetMetadataDomainList()`로 먼저 확인하는 방어 코드를 권장한다.
- QGIS 3.6의 GDAL은 2.x 계열이므로 COG 드라이버(`COG`)가 없다.
  COG 생성은 `GTiff` + `COPY_SRC_OVERVIEWS=YES`로 대체한다.

---

# Chapter 9. 벡터·공개데이터 인제스트

## 9.1 관심지역(AOI) 관리

GeoINT 작업의 시작은 거의 항상 AOI 정의다.

```python
# core/aoi.py
from __future__ import annotations
from dataclasses import dataclass
from typing import Optional

from qgis.core import (
    QgsVectorLayer, QgsFeature, QgsGeometry, QgsField,
    QgsCoordinateReferenceSystem, QgsRectangle,
)
from qgis.PyQt.QtCore import QVariant

from ..compat import memory_uri


@dataclass
class AOI:
    name: str
    geometry: QgsGeometry
    crs: QgsCoordinateReferenceSystem
    note: str = ""

    def bbox(self) -> QgsRectangle:
        return self.geometry.boundingBox()

    def area_km2(self) -> float:
        from qgis.core import QgsDistanceArea, QgsUnitTypes
        da = QgsDistanceArea()
        da.setSourceCrs(self.crs, __import__(
            "qgis.core", fromlist=["QgsProject"]).QgsProject.instance().transformContext())
        da.setEllipsoid("WGS84")
        m2 = da.measureArea(self.geometry)
        return da.convertAreaMeasurement(m2, QgsUnitTypes.AreaSquareKilometers)


def aoi_to_layer(aoi: AOI) -> QgsVectorLayer:
    uri = memory_uri("Polygon", aoi.crs.authid())
    layer = QgsVectorLayer(uri, "AOI - {0}".format(aoi.name), "memory")
    provider = layer.dataProvider()
    provider.addAttributes([
        QgsField("name", QVariant.String),
        QgsField("note", QVariant.String),
        QgsField("area_km2", QVariant.Double),
    ])
    layer.updateFields()

    feat = QgsFeature(layer.fields())
    feat.setGeometry(aoi.geometry)
    feat.setAttributes([aoi.name, aoi.note, aoi.area_km2()])
    provider.addFeature(feat)
    layer.updateExtents()
    return layer
```

> **호환성**
> `QgsField(name, QVariant.String)` 형식은 3.6~4.x에서 모두 동작한다.
> QGIS 3.36부터 `QMetaType` 기반 생성자가 추가되었으나 기존 형식도 유지된다.

## 9.2 CSV 좌표 인제스트

현장에서 가장 흔한 입력이다. 헤더가 제각각이라 유연한 매핑이 필요하다.

```python
# ingest/csv_points.py
from __future__ import annotations
import csv
from typing import Dict, List, Optional, Tuple

X_ALIASES = ["x", "lon", "longitude", "경도", "east", "easting", "e"]
Y_ALIASES = ["y", "lat", "latitude", "위도", "north", "northing", "n"]
Z_ALIASES = ["z", "elev", "elevation", "height", "표고", "고도"]


def sniff_columns(header: List[str]) -> Dict[str, Optional[str]]:
    low = {h.strip().lower(): h for h in header}

    def find(aliases):
        for a in aliases:
            if a in low:
                return low[a]
        return None

    return {"x": find(X_ALIASES), "y": find(Y_ALIASES), "z": find(Z_ALIASES)}


def read_points(path: str,
                encoding: str = "utf-8-sig") -> Tuple[List[dict], Dict[str, Optional[str]]]:
    with open(path, "r", encoding=encoding, newline="") as f:
        sample = f.read(4096)
        f.seek(0)
        try:
            dialect = csv.Sniffer().sniff(sample, delimiters=",;\t|")
        except csv.Error:
            dialect = csv.excel
        reader = csv.DictReader(f, dialect=dialect)
        rows = list(reader)
        mapping = sniff_columns(reader.fieldnames or [])
    return rows, mapping
```

> **TIP**
> 한국어 환경의 CSV는 CP949(EUC-KR)인 경우가 많다.
> `utf-8-sig`로 실패하면 `cp949`로 재시도하는 폴백을 넣는다.
> BOM 처리를 위해 `utf-8`이 아니라 `utf-8-sig`를 기본값으로 둔다.

```python
def read_points_auto(path: str):
    for enc in ("utf-8-sig", "cp949", "euc-kr", "latin-1"):
        try:
            return read_points(path, enc) + (enc,)
        except UnicodeDecodeError:
            continue
    raise IOError("인코딩을 판별할 수 없습니다: {0}".format(path))
```

## 9.3 OSM 데이터 활용

OpenStreetMap은 폐쇄망에서도 사전 추출본(`.osm.pbf` → GeoPackage)으로 반입해 쓸 수 있다.

주요 활용:

| 레이어 | GeoINT 용도 |
|---|---|
| `highway` | 접근로 분석, 경로 계산 |
| `building` | 시설물 밀도, 변화 대조 기준 |
| `waterway` / `natural=water` | 침수 판정 기준면 |
| `landuse` | 오탐 필터링 (농경지 계절 변화 등) |
| `amenity` | 주요 시설 위치 참조 |

표 9-1. OSM 레이어의 GeoINT 활용

> **WARNING**
> OSM은 ODbL 라이선스다. 파생 데이터베이스를 배포하면 동일 조건 공유 의무가 발생할 수 있다.
> 산출 지도에는 반드시 `© OpenStreetMap contributors` 출처를 표기한다. (Chapter 23)

## 9.4 속성 스키마 정규화

여러 출처의 벡터를 하나로 합칠 때는 공통 스키마로 강제한다.

```python
# ingest/schema.py
from typing import Dict, List
from qgis.PyQt.QtCore import QVariant
from qgis.core import QgsField, QgsFields

CANONICAL_FIELDS = [
    ("obj_id",      QVariant.String,  "고유 식별자"),
    ("obj_class",   QVariant.String,  "객체 분류"),
    ("source",      QVariant.String,  "출처"),
    ("source_grade", QVariant.String, "Admiralty 출처 등급 (A~F)"),
    ("cert_grade",  QVariant.String,  "확실성 등급 (1~6)"),
    ("observed_at", QVariant.String,  "관측 일시 ISO8601"),
    ("gsd_m",       QVariant.Double,  "지상표본거리(m)"),
    ("analyst",     QVariant.String,  "분석자"),
    ("remark",      QVariant.String,  "비고"),
]


def canonical_fields() -> QgsFields:
    fields = QgsFields()
    for name, vtype, comment in CANONICAL_FIELDS:
        f = QgsField(name, vtype)
        f.setComment(comment)
        fields.append(f)
    return fields


def missing_fields(layer) -> List[str]:
    existing = {f.name() for f in layer.fields()}
    return [n for n, _, _ in CANONICAL_FIELDS if n not in existing]
```

모든 탐지 결과 레이어는 이 스키마를 만족해야 한다.
Chapter 21의 융합 단계와 Chapter 24의 산출물 생성은 이 필드명을 전제로 동작한다.

### 버전 호환 노트 (Chapter 9)

- `QgsField.setComment()`는 3.6부터 지원된다.
- `QgsVectorLayer` 메모리 provider의 `index=yes` 옵션은 3.6에서도 유효하다.
- QGIS 4에서 `QVariant` 기반 필드 타입은 여전히 동작하지만,
  신규 코드에서는 `QMetaType.Type.QString` 형식이 권장된다.
  `compat.py`에 필드 팩토리를 두면 양쪽을 흡수할 수 있다.

---

# Chapter 10. 좌표계와 격자 체계

## 10.1 GeoINT에서 좌표계가 특별히 중요한 이유

일반 GIS에서는 좌표계 오류가 "지도가 이상하게 보인다"로 끝난다.
GeoINT에서는 **잘못된 위치 판단**으로 이어진다. 수십 미터의 오차가 결론을 뒤집는다.

주요 실패 유형:

| 실패 | 원인 | 결과 |
|---|---|---|
| 수백 미터 어긋남 | 측지계 불일치 (Bessel vs GRS80) | 오판 |
| 수 미터 어긋남 | 격자 변환 파일 부재 | 정밀 분석 실패 |
| 완전히 다른 위치 | 축척계수/원점 혼동 | 즉시 발견 (그나마 다행) |
| 고도 오차 수십 미터 | 타원체고 vs 표고(지오이드) 혼동 | DEM 분석 왜곡 |

표 10-1. 좌표계 관련 실패 유형

## 10.2 한국 좌표계 정리

| EPSG | 명칭 | 원점 | 비고 |
|---|---|---:|---|
| 5186 | Korea 2000 / Central Belt 2010 | 127°E, 38°N | 현행 중부원점 |
| 5185 | 서부원점 | 125°E | |
| 5187 | 동부원점 | 129°E | |
| 5188 | 동해원점 | 131°E | |
| 5179 | Korea 2000 / Unified CS (UTM-K) | 127.5°E | 국가기본도 |
| 5174 | Korean 1985 / Modified Central Belt | 127°E (구) | **구 베셀 측지계** |
| 4326 | WGS84 | — | GPS 원본 |
| 32651/32652 | WGS84 / UTM 51N, 52N | — | 국제 호환 |

표 10-2. 한국에서 사용되는 주요 좌표계

> **WARNING**
> EPSG:5174(구 베셀 측지계)와 EPSG:5186(세계측지계)의 차이는 약 **160~360 m**다.
> 과거 자료에는 5174가 흔하게 남아 있다. 변환 없이 겹치면 확연히 어긋나므로
> 오히려 발견하기 쉽지만, 5174의 여러 변형(중부원점 보정값 차이)이 섞이면
> 수 미터 오차로 나타나 발견이 어렵다.

## 10.3 안전한 변환 래퍼

```python
# core/crs.py
from __future__ import annotations
from typing import Optional, Tuple

from qgis.core import (
    QgsCoordinateReferenceSystem, QgsCoordinateTransform,
    QgsPointXY, QgsProject, QgsGeometry, QgsDatumTransform,
)
from ..compat import make_transform


class CrsError(RuntimeError):
    pass


def crs(authid: str) -> QgsCoordinateReferenceSystem:
    c = QgsCoordinateReferenceSystem(authid)
    if not c.isValid():
        raise CrsError("유효하지 않은 좌표계: {0}".format(authid))
    return c


def transform_point(pt: QgsPointXY, src: str, dst: str) -> QgsPointXY:
    tr = make_transform(crs(src), crs(dst))
    return tr.transform(pt)


def transform_geometry(geom: QgsGeometry, src: str, dst: str) -> QgsGeometry:
    g = QgsGeometry(geom)
    tr = make_transform(crs(src), crs(dst))
    if g.transform(tr) != 0:
        raise CrsError("지오메트리 변환 실패: {0} → {1}".format(src, dst))
    return g


def roundtrip_error_m(pt: QgsPointXY, src: str, dst: str) -> float:
    """왕복 변환 오차. 변환 경로 건전성 확인용."""
    fwd = transform_point(pt, src, dst)
    back = transform_point(fwd, dst, src)
    dx = back.x() - pt.x()
    dy = back.y() - pt.y()
    return (dx * dx + dy * dy) ** 0.5
```

`roundtrip_error_m()`은 자체 진단 기능으로 유용하다.
1 mm 이상 오차가 나면 변환 경로에 문제가 있다는 신호다.

## 10.4 PROJ 격자 파일 부재 처리

정밀 변환에는 격자 이동 파일(`.gsb`, `.tif`)이 필요하다.
폐쇄망에서는 자동 다운로드가 불가능하므로 사전 반입해야 한다.

```python
# core/proj_check.py
import os
from typing import List
from qgis.core import QgsApplication


def proj_search_paths() -> List[str]:
    paths = []
    env = os.environ.get("PROJ_LIB") or os.environ.get("PROJ_DATA")
    if env:
        paths.extend(env.split(os.pathsep))
    paths.append(os.path.join(QgsApplication.prefixPath(), "share", "proj"))
    return [p for p in paths if os.path.isdir(p)]


def has_grid(filename: str) -> bool:
    return any(os.path.exists(os.path.join(p, filename))
               for p in proj_search_paths())


def missing_grids(required: List[str]) -> List[str]:
    return [g for g in required if not has_grid(g)]
```

시작 시 점검하고 결과를 로그에 남긴다.

```python
REQUIRED_GRIDS = ["kr_ngii_KNGeoid18.tif"]   # 예시: 지오이드 모델

missing = missing_grids(REQUIRED_GRIDS)
if missing:
    self.log("PROJ 격자 파일 누락: {0}. 정밀 변환 정확도가 저하될 수 있습니다."
             .format(", ".join(missing)), LEVEL_WARNING)
```

## 10.5 MGRS 격자

MGRS(Military Grid Reference System)는 GeoINT 실무에서 위치를 구술·기록하는 표준 표기다.
UTM 기반이며 문자열 하나로 정밀도까지 표현한다.

```text
52S CH 12345 67890
│   │  │     └── 북쪽 좌표 (5자리 = 1 m 정밀도)
│   │  └──────── 동쪽 좌표
│   └─────────── 100 km 격자 식별 문자
└─────────────── UTM 존 + 위도대
```

그림 10-1. MGRS 표기 구조

| 자릿수 | 정밀도 | 예 |
|---|---|---|
| 0 | 100 km | 52S CH |
| 2 | 10 km | 52S CH 1 6 |
| 4 | 1 km | 52S CH 12 67 |
| 6 | 100 m | 52S CH 123 678 |
| 8 | 10 m | 52S CH 1234 6789 |
| 10 | 1 m | 52S CH 12345 67890 |

표 10-3. MGRS 정밀도 단계

```python
# core/grid.py
from __future__ import annotations
from typing import Optional, Tuple

from qgis.core import QgsPointXY
from .crs import transform_point
from .optional import GATES


def to_mgrs(pt_wgs84: QgsPointXY, precision: int = 5) -> Optional[str]:
    """WGS84 경위도 → MGRS 문자열. precision은 자릿수(1~5)."""
    if not GATES.available("mgrs"):
        return None
    import mgrs
    conv = mgrs.MGRS()
    return conv.toMGRS(pt_wgs84.y(), pt_wgs84.x(), MGRSPrecision=precision)


def from_mgrs(text: str) -> Optional[QgsPointXY]:
    if not GATES.available("mgrs"):
        return None
    import mgrs
    conv = mgrs.MGRS()
    lat, lon = conv.toLatLon(text.replace(" ", ""))
    return QgsPointXY(lon, lat)


def layer_point_to_mgrs(pt: QgsPointXY, layer_crs_authid: str,
                        precision: int = 5) -> Optional[str]:
    wgs = transform_point(pt, layer_crs_authid, "EPSG:4326")
    return to_mgrs(wgs, precision)
```

> **TIP**
> MGRS 표기는 **정밀도가 곧 주장의 강도**다.
> GSD 10 m 영상에서 1 m 정밀도(10자리) MGRS를 기록하면
> 데이터가 뒷받침하지 않는 정밀도를 주장하는 셈이다.
> QGeoINT는 영상의 GSD에 따라 MGRS 자릿수를 자동 제한한다.

```python
def precision_for_gsd(gsd_m: float) -> int:
    if gsd_m <= 1.0:
        return 5      # 1 m
    if gsd_m <= 10.0:
        return 4      # 10 m
    if gsd_m <= 100.0:
        return 3      # 100 m
    return 2          # 1 km
```

### 버전 호환 노트 (Chapter 10)

- `QgsCoordinateTransform` 생성자는 3.8에서 `QgsCoordinateTransformContext` 인자를 받도록 확장되었다.
  3.6에서는 `QgsProject`를 넘겨야 한다 → `compat.make_transform()`이 처리한다.
- `QgsDatumTransform` API는 3.8에서 대폭 개편되었다. 3.6 지원 시 직접 사용을 피한다.
- QGIS 3.10 이상은 PROJ 6+를 사용하며 격자 파일 이름 체계가 바뀌었다.

---

# Chapter 11. 시간 정보 모델

## 11.1 GeoINT는 4차원이다

변화 탐지, 활동 패턴 분석, 사건 재구성 — 모두 시간 축 없이는 성립하지 않는다.
그럼에도 대부분의 GIS 데이터는 시간을 부실하게 다룬다.

QGeoINT의 시간 처리 규칙:

1. 모든 시각은 **ISO 8601 문자열**로 저장한다 (`2026-08-31T09:15:00+09:00`).
2. 시간대(timezone)를 반드시 포함한다. 영상 메타데이터는 대개 UTC다.
3. 시각이 불확실하면 **구간**으로 표현한다 (`observed_from`, `observed_to`).
4. "관측 시각"과 "사건 발생 시각"을 구분한다.

## 11.2 시간 파서

```python
# core/timeutil.py
from __future__ import annotations
import re
from datetime import datetime, timezone, timedelta
from typing import Optional

KST = timezone(timedelta(hours=9))

_PATTERNS = [
    ("%Y-%m-%dT%H:%M:%S", r"^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}"),
    ("%Y-%m-%d %H:%M:%S", r"^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}"),
    ("%Y:%m:%d %H:%M:%S", r"^\d{4}:\d{2}:\d{2} \d{2}:\d{2}:\d{2}"),   # EXIF
    ("%Y%m%d%H%M%S",      r"^\d{14}$"),                                # NITF IDATIM
    ("%Y-%m-%d",          r"^\d{4}-\d{2}-\d{2}$"),
    ("%Y%m%d",            r"^\d{8}$"),
]


def parse_datetime(text: Optional[str],
                   assume_tz: timezone = timezone.utc) -> Optional[datetime]:
    if not text:
        return None
    s = text.strip().replace("Z", "+00:00")

    # ISO 8601 오프셋 포함 우선 시도
    try:
        return datetime.fromisoformat(s)
    except (ValueError, AttributeError):
        pass

    for fmt, pattern in _PATTERNS:
        if re.match(pattern, s):
            try:
                dt = datetime.strptime(s[:len(datetime.now().strftime(fmt))], fmt)
                return dt.replace(tzinfo=assume_tz)
            except ValueError:
                continue
    return None


def to_kst(dt: datetime) -> datetime:
    if dt.tzinfo is None:
        dt = dt.replace(tzinfo=timezone.utc)
    return dt.astimezone(KST)


def iso(dt: Optional[datetime]) -> Optional[str]:
    return dt.isoformat(timespec="seconds") if dt else None
```

> **WARNING**
> `datetime.fromisoformat()`은 Python 3.7부터 사용 가능하지만
> 3.11 이전에는 `Z` 접미사를 파싱하지 못한다. 위 코드처럼 사전 치환이 필요하다.

## 11.3 시간 구간 모델

```python
# core/timespan.py
from __future__ import annotations
from dataclasses import dataclass
from datetime import datetime
from typing import Optional


@dataclass
class TimeSpan:
    """관측 시각의 불확실성을 구간으로 표현."""
    start: datetime
    end: Optional[datetime] = None

    @property
    def is_instant(self) -> bool:
        return self.end is None or self.end == self.start

    @property
    def uncertainty_seconds(self) -> float:
        if self.is_instant:
            return 0.0
        return (self.end - self.start).total_seconds()

    def overlaps(self, other: "TimeSpan") -> bool:
        a_end = self.end or self.start
        b_end = other.end or other.start
        return self.start <= b_end and other.start <= a_end

    def label(self) -> str:
        if self.is_instant:
            return self.start.strftime("%Y-%m-%d %H:%M")
        return "{0} ~ {1}".format(
            self.start.strftime("%Y-%m-%d %H:%M"),
            self.end.strftime("%Y-%m-%d %H:%M"),
        )
```

변화 탐지 결과의 시간 표현은 항상 구간이다.
"8월 3일 영상과 8월 20일 영상 사이에 변화 발생" — 정확한 발생 시각은 알 수 없다.

```python
def change_window(before_time: datetime, after_time: datetime) -> TimeSpan:
    """변화가 발생했을 수 있는 시간 구간."""
    return TimeSpan(start=before_time, end=after_time)
```

## 11.4 QGIS 시간 컨트롤러 연동

QGIS 3.14부터 `QgsTemporalController`가 도입되어 시계열 재생이 가능하다.

```python
# gui/temporal.py
from qgis.core import QgsDateTimeRange
from ..compat import at_least


def enable_layer_temporal(layer, field_start: str,
                          field_end: str = None) -> bool:
    """벡터 레이어에 시간 속성을 설정한다. 3.14 미만은 False 반환."""
    if not at_least(3, 14):
        return False

    from qgis.core import QgsVectorLayerTemporalProperties as TP

    props = layer.temporalProperties()
    if field_end:
        props.setMode(TP.ModeFeatureDateTimeStartAndEndFromFields)
        props.setStartField(field_start)
        props.setEndField(field_end)
    else:
        props.setMode(TP.ModeFeatureDateTimeInstantFromField)
        props.setStartField(field_start)
    props.setIsActive(True)
    layer.triggerRepaint()
    return True
```

3.14 미만 환경에서는 폴백으로 **속성 필터 기반 시간 슬라이더**를 자체 구현한다.

```python
def apply_time_filter(layer, field: str, start_iso: str, end_iso: str) -> None:
    """모든 버전에서 동작하는 폴백. subsetString 사용."""
    expr = "\"{0}\" >= '{1}' AND \"{0}\" <= '{2}'".format(field, start_iso, end_iso)
    layer.setSubsetString(expr)
```

> **TIP**
> `setSubsetString()`은 provider 수준 필터라 성능이 좋다.
> 다만 메모리 레이어에서는 지원되지 않는 버전이 있으므로
> 시계열 대상 레이어는 GeoPackage로 저장한 뒤 필터를 건다.

## 11.5 시계열 스택 관리

여러 시점의 영상을 하나의 분석 단위로 묶는다.

```python
# core/stack.py
from __future__ import annotations
from dataclasses import dataclass, field
from datetime import datetime
from typing import List, Optional

from .timeutil import parse_datetime


@dataclass
class StackEntry:
    path: str
    acquired: Optional[datetime]
    sensor: Optional[str] = None
    gsd_m: Optional[float] = None
    cloud_cover: Optional[float] = None


@dataclass
class TimeStack:
    name: str
    entries: List[StackEntry] = field(default_factory=list)

    def sorted_entries(self) -> List[StackEntry]:
        dated = [e for e in self.entries if e.acquired]
        return sorted(dated, key=lambda e: e.acquired)

    def pairs(self):
        """인접 시점 쌍을 순서대로 생성 (변화 탐지 입력)."""
        s = self.sorted_entries()
        for i in range(len(s) - 1):
            yield s[i], s[i + 1]

    def best_pair(self, max_cloud: float = 20.0):
        """구름이 적은 최초/최종 시점 쌍을 선택."""
        clean = [e for e in self.sorted_entries()
                 if e.cloud_cover is None or e.cloud_cover <= max_cloud]
        if len(clean) < 2:
            raise ValueError("조건을 만족하는 영상이 2장 미만입니다.")
        return clean[0], clean[-1]

    def gaps_days(self) -> List[float]:
        s = self.sorted_entries()
        return [(s[i + 1].acquired - s[i].acquired).total_seconds() / 86400.0
                for i in range(len(s) - 1)]
```

`gaps_days()`는 관측 공백을 드러낸다.
"6개월 공백이 있었다"는 사실 자체가 분석 결론의 신뢰도에 영향을 준다.

### 버전 호환 노트 (Chapter 11)

- `QgsTemporalController`, `QgsVectorLayerTemporalProperties`: **3.14 이상**.
- `QgsRasterLayerTemporalProperties`: 3.14 이상.
- 3.6~3.12에서는 `setSubsetString()` 기반 폴백을 사용한다.
- `datetime.fromisoformat()`의 `Z` 지원은 Python 3.11 이상이다.

---
---

# Part IV. 영상 분석

---

# Chapter 12. 래스터 처리 기초

## 12.1 QGIS 래스터 읽기 전략

QGIS에서 래스터 픽셀에 접근하는 방법은 세 가지이며, 목적에 따라 선택이 다르다.

| 방법 | API | 적합한 경우 |
|---|---|---|
| 블록 읽기 | `QgsRasterDataProvider.block()` | 소~중 규모, QGIS 렌더링 파이프라인 활용 |
| GDAL 직접 | `gdal.Open().ReadAsArray()` | 대규모, numpy 연산 중심 |
| Processing | `processing.run()` | 재현성 필요, 표준 알고리즘 존재 |

표 12-1. 래스터 접근 방법 비교

GeoINT에서는 **GDAL 직접 + 타일 단위 처리**를 기본으로 한다. 대용량이 전제이기 때문이다.

## 12.2 타일 단위 읽기

```python
# repositories/raster_repo.py
from __future__ import annotations

from dataclasses import dataclass
from typing import Iterator, Optional, Tuple

import numpy as np
from osgeo import gdal, osr


@dataclass
class RasterMeta:
    width: int
    height: int
    geotransform: Tuple[float, ...]
    projection: str
    nodata: Optional[float]
    dtype: int


def open_raster(path: str) -> "gdal.Dataset":
    ds = gdal.Open(path, gdal.GA_ReadOnly)
    if ds is None:
        raise IOError("래스터를 열 수 없습니다: {0}".format(path))
    return ds


def raster_meta(ds) -> RasterMeta:
    b = ds.GetRasterBand(1)
    return RasterMeta(
        width=ds.RasterXSize,
        height=ds.RasterYSize,
        geotransform=ds.GetGeoTransform(),
        projection=ds.GetProjection(),
        nodata=b.GetNoDataValue(),
        dtype=b.DataType,
    )


def iter_tiles(width: int, height: int,
               tile: int = 1024, overlap: int = 0
               ) -> Iterator[Tuple[int, int, int, int]]:
    """(xoff, yoff, xsize, ysize) 타일을 생성한다."""
    step = tile - overlap
    for y in range(0, height, step):
        ysize = min(tile, height - y)
        if ysize <= 0:
            break
        for x in range(0, width, step):
            xsize = min(tile, width - x)
            if xsize <= 0:
                break
            yield x, y, xsize, ysize


def read_band(layer_or_path, band: int = 1
              ) -> Tuple[np.ndarray, RasterMeta]:
    path = layer_or_path if isinstance(layer_or_path, str) \
        else layer_or_path.source()
    ds = open_raster(path)
    meta = raster_meta(ds)
    arr = ds.GetRasterBand(band).ReadAsArray().astype("float32")
    if meta.nodata is not None:
        arr = np.where(arr == meta.nodata, np.nan, arr)
    ds = None
    return arr, meta
```

> **WARNING**
> `ReadAsArray()`로 전체 영상을 한 번에 읽으면
> 30000×30000 float32 영상은 약 3.6 GB를 점유한다.
> 실무 코드는 항상 `iter_tiles()` 기반으로 작성하고,
> 전체 읽기는 소규모 AOI에만 사용한다.

## 12.3 정규화 지수 계산

```python
# analysis/indices.py
from __future__ import annotations
import numpy as np


def _safe_ratio(a: np.ndarray, b: np.ndarray) -> np.ndarray:
    """(a-b)/(a+b)를 0 나눗셈 없이 계산."""
    num = a - b
    den = a + b
    out = np.full(a.shape, np.nan, dtype="float32")
    valid = np.isfinite(den) & (np.abs(den) > 1e-10)
    out[valid] = num[valid] / den[valid]
    return out


def ndvi(nir: np.ndarray, red: np.ndarray) -> np.ndarray:
    """식생지수. 식생 존재·활력 판단."""
    return _safe_ratio(nir, red)


def ndwi(green: np.ndarray, nir: np.ndarray) -> np.ndarray:
    """수분지수(McFeeters). 수체 추출."""
    return _safe_ratio(green, nir)


def mndwi(green: np.ndarray, swir: np.ndarray) -> np.ndarray:
    """수정 수분지수(Xu). 도시 지역 수체 추출에 유리."""
    return _safe_ratio(green, swir)


def ndbi(swir: np.ndarray, nir: np.ndarray) -> np.ndarray:
    """건조지/시가화지수."""
    return _safe_ratio(swir, nir)


def nbr(nir: np.ndarray, swir2: np.ndarray) -> np.ndarray:
    """정규화 연소비. 산불 피해 범위 산정."""
    return _safe_ratio(nir, swir2)
```

| 지수 | 밴드 조합 | GeoINT 활용 |
|---|---|---|
| NDVI | (NIR−RED)/(NIR+RED) | 식생 변화, 위장 탐지 보조 |
| NDWI | (GREEN−NIR)/(GREEN+NIR) | 침수 범위 |
| MNDWI | (GREEN−SWIR)/(GREEN+SWIR) | 도시 침수 (건물 오탐 감소) |
| NDBI | (SWIR−NIR)/(SWIR+NIR) | 신규 조성지 탐지 |
| NBR | (NIR−SWIR2)/(NIR+SWIR2) | 산불 피해 등급 |

표 12-2. 주요 정규화 지수

## 12.4 밴드 매핑

센서마다 밴드 순서가 다르다. 하드코딩하면 다른 위성 자료에서 즉시 깨진다.

```python
# analysis/bandmap.py
from typing import Dict, Optional

SENSOR_BANDS: Dict[str, Dict[str, int]] = {
    "SENTINEL2_L2A_10M": {"blue": 1, "green": 2, "red": 3, "nir": 4},
    "SENTINEL2_L2A_20M": {"swir1": 5, "swir2": 6},
    "LANDSAT8_OLI":      {"coastal": 1, "blue": 2, "green": 3, "red": 4,
                          "nir": 5, "swir1": 6, "swir2": 7},
    "RGB":               {"red": 1, "green": 2, "blue": 3},
    "RGBN":              {"red": 1, "green": 2, "blue": 3, "nir": 4},
}


def resolve_band(profile: str, name: str) -> Optional[int]:
    return SENSOR_BANDS.get(profile, {}).get(name)


def require_bands(profile: str, *names: str) -> Dict[str, int]:
    mapping = {}
    for n in names:
        idx = resolve_band(profile, n)
        if idx is None:
            raise ValueError(
                "'{0}' 프로파일에 '{1}' 밴드가 정의되어 있지 않습니다.".format(profile, n))
        mapping[n] = idx
    return mapping
```

사용자가 프로파일을 선택하도록 GUI에 노출하고, 선택값을 레시피에 기록한다.

## 12.5 래스터 쓰기

```python
def write_array(arr: np.ndarray, meta: RasterMeta, path: str,
                dtype: int = gdal.GDT_Float32,
                nodata: Optional[float] = None,
                cog: bool = True) -> str:
    driver = gdal.GetDriverByName("GTiff")
    options = ["COMPRESS=DEFLATE", "PREDICTOR=2", "TILED=YES",
               "BLOCKXSIZE=512", "BLOCKYSIZE=512", "BIGTIFF=IF_SAFER"]

    ds = driver.Create(path, meta.width, meta.height, 1, dtype, options)
    ds.SetGeoTransform(meta.geotransform)
    ds.SetProjection(meta.projection)

    band = ds.GetRasterBand(1)
    if nodata is not None:
        band.SetNoDataValue(nodata)
        arr = np.where(np.isnan(arr), nodata, arr)
    band.WriteArray(arr)
    band.FlushCache()

    if cog:
        ds.BuildOverviews("AVERAGE", [2, 4, 8, 16])

    ds = None
    return path
```

> **호환성**
> QGIS 3.6의 GDAL 2.x에는 전용 `COG` 드라이버가 없다.
> 위처럼 `GTiff` + 타일링 + 오버뷰로 사실상 동등한 결과를 만든다.
> GDAL 3.1 이상에서는 `gdal.GetDriverByName("COG")`를 쓸 수 있으므로
> 드라이버 존재 여부로 분기한다.

### 버전 호환 노트 (Chapter 12)

- `gdal.Dataset.BuildOverviews()`는 전 버전 공통이다.
- `PREDICTOR=2`는 정수형에만 유효하다. 실수형은 `PREDICTOR=3`을 사용한다.
- QGIS 4.x 번들 GDAL은 3.8 이상이므로 `COG` 드라이버 사용 가능.

---

# Chapter 13. 변화 탐지

## 13.1 변화 탐지의 실제 난이도

"두 영상을 빼면 변화가 나온다"는 설명은 교과서에만 존재한다.
실무에서 차분 결과의 대부분은 **변화가 아니라 잡음**이다.

| 오탐 원인 | 현상 | 대응 |
|---|---|---|
| 기하 오정합 | 경계선 전체가 변화로 표시 | 사전 정합 (Chapter 14) |
| 조도·계절 차이 | 전체적 밝기 이동 | 상대 방사 정규화 |
| 그림자 각도 차이 | 건물 북측 변화 오탐 | 태양각 기반 마스킹 |
| 구름·구름 그림자 | 대규모 허위 변화 | 구름 마스크 |
| 식생 계절 변화 | 농경지 전체 변화 | 토지피복 마스크 |
| 수위 변동 | 하천 경계 변화 | 상시수역 마스크 |
| 센서 차이 | 밴드 특성 불일치 | 동일 센서 쌍 사용 원칙 |

표 13-1. 변화 탐지 오탐 원인과 대응

> **ENGINEERING PRACTICE**
> 변화 탐지 도구의 품질은 "얼마나 잘 찾느냐"가 아니라
> **"얼마나 적게 틀리느냐"** 로 평가된다.
> 분석관이 검토해야 할 후보가 5000개면 그 도구는 쓰이지 않는다.

## 13.2 상대 방사 정규화

절대 보정 정보가 없을 때, 두 영상의 통계를 맞춘다.

```python
# analysis/radiometry.py
from __future__ import annotations
import numpy as np
from typing import Optional, Tuple


def histogram_match(source: np.ndarray, reference: np.ndarray,
                    mask: Optional[np.ndarray] = None) -> np.ndarray:
    """source의 히스토그램을 reference에 맞춘다."""
    s = source.copy()
    valid = np.isfinite(s) & np.isfinite(reference)
    if mask is not None:
        valid &= ~mask
    if valid.sum() < 100:
        return s

    s_vals = s[valid].ravel()
    r_vals = reference[valid].ravel()

    s_sorted, s_idx, s_counts = np.unique(s_vals, return_inverse=True,
                                          return_counts=True)
    r_sorted, r_counts = np.unique(r_vals, return_counts=True)

    s_cdf = np.cumsum(s_counts).astype("float64") / s_vals.size
    r_cdf = np.cumsum(r_counts).astype("float64") / r_vals.size

    interp = np.interp(s_cdf, r_cdf, r_sorted)
    out = s.copy()
    out[valid] = interp[s_idx]
    return out.astype("float32")


def linear_normalize(source: np.ndarray, reference: np.ndarray,
                     mask: Optional[np.ndarray] = None
                     ) -> Tuple[np.ndarray, float, float]:
    """평균·표준편차 기반 선형 정규화. 계수를 함께 반환(추적성)."""
    valid = np.isfinite(source) & np.isfinite(reference)
    if mask is not None:
        valid &= ~mask

    s_mean, s_std = float(np.mean(source[valid])), float(np.std(source[valid]))
    r_mean, r_std = float(np.mean(reference[valid])), float(np.std(reference[valid]))

    gain = r_std / s_std if s_std > 1e-9 else 1.0
    offset = r_mean - gain * s_mean
    return (source * gain + offset).astype("float32"), gain, offset
```

`gain`과 `offset`을 반환하는 이유는 원칙 2(추적성) 때문이다.
어떤 보정이 적용되었는지 레시피에 남겨야 한다.

## 13.3 임계값 결정

고정 임계값은 영상마다 다르게 동작한다. 통계 기반 자동 임계값을 기본으로 한다.

```python
# analysis/threshold.py
from __future__ import annotations
import numpy as np
from typing import Tuple


def sigma_threshold(diff: np.ndarray, k: float = 2.5) -> float:
    """평균 + k·표준편차. 가장 단순하고 설명하기 쉽다."""
    v = diff[np.isfinite(diff)]
    return float(np.mean(v) + k * np.std(v))


def mad_threshold(diff: np.ndarray, k: float = 3.0) -> float:
    """중앙값 절대편차 기반. 이상치에 강건하다."""
    v = diff[np.isfinite(diff)]
    med = np.median(v)
    mad = np.median(np.abs(v - med))
    return float(med + k * 1.4826 * mad)


def otsu_threshold(diff: np.ndarray, bins: int = 256) -> float:
    """Otsu 이진화. 이봉 분포일 때 효과적."""
    v = diff[np.isfinite(diff)]
    hist, edges = np.histogram(v, bins=bins)
    hist = hist.astype("float64")
    total = hist.sum()
    if total == 0:
        return float(np.nanmax(diff))

    centers = (edges[:-1] + edges[1:]) / 2.0
    w0 = np.cumsum(hist) / total
    w1 = 1.0 - w0
    m0 = np.cumsum(hist * centers) / np.maximum(np.cumsum(hist), 1)
    total_mean = (hist * centers).sum() / total
    m1 = (total_mean - m0 * w0) / np.maximum(w1, 1e-12)

    between = w0 * w1 * (m0 - m1) ** 2
    return float(centers[int(np.argmax(between))])


def choose_threshold(diff: np.ndarray, method: str = "mad",
                     k: float = 3.0) -> Tuple[float, str]:
    if method == "sigma":
        return sigma_threshold(diff, k), "mean+{0}sigma".format(k)
    if method == "otsu":
        return otsu_threshold(diff), "otsu"
    return mad_threshold(diff, k), "median+{0}MAD".format(k)
```

> **TIP**
> 자동 임계값의 결과는 반드시 사용자에게 **히스토그램과 함께** 제시한다.
> 숫자만 보여주면 분석관이 판단할 수 없다.
> QGeoINT의 변화 탐지 다이얼로그는 차분 히스토그램 위에 임계선을 겹쳐 그린다.

## 13.4 후처리 — 잡음 제거

```python
# analysis/morphology.py
from __future__ import annotations
import numpy as np
from typing import Tuple


def _binary_dilate(mask: np.ndarray) -> np.ndarray:
    out = mask.copy()
    out[1:, :] |= mask[:-1, :]
    out[:-1, :] |= mask[1:, :]
    out[:, 1:] |= mask[:, :-1]
    out[:, :-1] |= mask[:, 1:]
    return out


def _binary_erode(mask: np.ndarray) -> np.ndarray:
    return ~_binary_dilate(~mask)


def opening(mask: np.ndarray, iterations: int = 1) -> np.ndarray:
    """침식 후 팽창. 고립된 점 잡음 제거."""
    m = mask
    for _ in range(iterations):
        m = _binary_erode(m)
    for _ in range(iterations):
        m = _binary_dilate(m)
    return m


def closing(mask: np.ndarray, iterations: int = 1) -> np.ndarray:
    """팽창 후 침식. 내부 구멍 메움."""
    m = mask
    for _ in range(iterations):
        m = _binary_dilate(m)
    for _ in range(iterations):
        m = _binary_erode(m)
    return m


def label_components(mask: np.ndarray) -> Tuple[np.ndarray, int]:
    """4-연결 연결요소 라벨링. scipy 없이 순수 numpy로 구현."""
    labels = np.zeros(mask.shape, dtype="int32")
    current = 0
    h, w = mask.shape
    for y in range(h):
        for x in range(w):
            if not mask[y, x] or labels[y, x]:
                continue
            current += 1
            stack = [(y, x)]
            labels[y, x] = current
            while stack:
                cy, cx = stack.pop()
                for ny, nx in ((cy-1, cx), (cy+1, cx), (cy, cx-1), (cy, cx+1)):
                    if 0 <= ny < h and 0 <= nx < w and mask[ny, nx] \
                            and labels[ny, nx] == 0:
                        labels[ny, nx] = current
                        stack.append((ny, nx))
    return labels, current


def filter_by_area(mask: np.ndarray, min_px: int) -> np.ndarray:
    labels, n = label_components(mask)
    if n == 0:
        return mask
    counts = np.bincount(labels.ravel())
    keep = np.zeros(counts.size, dtype=bool)
    keep[1:] = counts[1:] >= min_px
    return keep[labels]
```

> **WARNING**
> 위 `label_components()`는 순수 Python 루프라 대형 영상에서 매우 느리다.
> `scipy.ndimage.label`이 있으면 그것을 쓰고, 없을 때만 폴백으로 사용한다.
> QGIS 배포본에 scipy가 포함되지 않는 경우가 있으므로 폴백이 필요하다.

```python
def label_components_fast(mask):
    from ..core.optional import GATES
    if GATES.available("scipy"):
        from scipy import ndimage
        return ndimage.label(mask)
    return label_components(mask)
```

## 13.5 변화 후보의 벡터화

래스터 마스크를 폴리곤으로 변환해야 분석관이 검토·주석할 수 있다.

```python
# analysis/vectorize.py
from osgeo import gdal, ogr, osr
from typing import Optional


def polygonize(mask_path: str, out_gpkg: str, layer_name: str = "change",
               crs_wkt: Optional[str] = None) -> str:
    src = gdal.Open(mask_path)
    band = src.GetRasterBand(1)

    driver = ogr.GetDriverByName("GPKG")
    if not gdal.VSIStatL(out_gpkg):
        dst = driver.CreateDataSource(out_gpkg)
    else:
        dst = ogr.Open(out_gpkg, update=1) or driver.CreateDataSource(out_gpkg)

    srs = osr.SpatialReference()
    srs.ImportFromWkt(crs_wkt or src.GetProjection())

    layer = dst.CreateLayer(layer_name, srs=srs, geom_type=ogr.wkbPolygon)
    layer.CreateField(ogr.FieldDefn("value", ogr.OFTInteger))

    gdal.Polygonize(band, band.GetMaskBand(), layer, 0, [], callback=None)

    # value=0 (배경) 제거
    layer.SetAttributeFilter("value = 0")
    for feat in layer:
        layer.DeleteFeature(feat.GetFID())
    layer.SetAttributeFilter(None)

    dst = None
    src = None
    return out_gpkg
```

## 13.6 변화 후보에 속성 부여

벡터화된 후보에 판단 근거를 붙인다.

```python
# analysis/enrich.py
from qgis.core import QgsVectorLayer, QgsField, edit
from qgis.PyQt.QtCore import QVariant

from ..core.grid import layer_point_to_mgrs, precision_for_gsd


def enrich_change_features(layer: QgsVectorLayer,
                           gsd_m: float,
                           observed_span_label: str,
                           source_grade: str = "B",
                           cert_grade: str = "3") -> None:
    new_fields = [
        QgsField("area_m2", QVariant.Double),
        QgsField("mgrs", QVariant.String),
        QgsField("observed_at", QVariant.String),
        QgsField("source_grade", QVariant.String),
        QgsField("cert_grade", QVariant.String),
        QgsField("review", QVariant.String),   # 분석관 검토 결과
    ]
    existing = {f.name() for f in layer.fields()}
    to_add = [f for f in new_fields if f.name() not in existing]
    if to_add:
        layer.dataProvider().addAttributes(to_add)
        layer.updateFields()

    prec = precision_for_gsd(gsd_m)
    authid = layer.crs().authid()

    with edit(layer):
        for feat in layer.getFeatures():
            geom = feat.geometry()
            centroid = geom.centroid().asPoint()
            feat["area_m2"] = geom.area()
            feat["mgrs"] = layer_point_to_mgrs(centroid, authid, prec) or ""
            feat["observed_at"] = observed_span_label
            feat["source_grade"] = source_grade
            feat["cert_grade"] = cert_grade
            feat["review"] = "미검토"
            layer.updateFeature(feat)
```

`review` 필드가 핵심이다. 자동 탐지 결과는 **후보**일 뿐이며,
분석관 검토를 거치지 않은 항목은 산출물에 포함하지 않는다.

| review 값 | 의미 |
|---|---|
| 미검토 | 자동 탐지 직후 상태 |
| 확인 | 실제 변화로 판단 |
| 오탐 | 잡음으로 판단 |
| 보류 | 추가 자료 필요 |

표 13-2. 검토 상태 값

### 버전 호환 노트 (Chapter 13)

- `gdal.Polygonize()`는 GDAL 2.x/3.x 모두 동일 시그니처다.
- `with edit(layer)` 컨텍스트 매니저는 QGIS 3.0부터 제공된다.
- `numpy.unique(return_counts=True)`는 numpy 1.9+ 이므로 문제없다.

---

# Chapter 14. 영상 정합과 기하 품질

## 14.1 정합이 선행되지 않으면 변화 탐지는 무의미하다

두 영상이 1픽셀만 어긋나도 모든 경계선이 변화로 검출된다.
GSD 0.5 m 영상에서 1픽셀 오차는 실제로는 대수롭지 않지만,
차분 결과에서는 도로·건물 윤곽 전체를 뒤덮는다.

## 14.2 정합 품질 진단

```python
# analysis/coregistration.py
from __future__ import annotations
import numpy as np
from typing import Tuple, Optional


def phase_correlation_shift(a: np.ndarray, b: np.ndarray
                            ) -> Tuple[float, float, float]:
    """위상상관법으로 (dy, dx) 이동량과 상관 피크 강도를 추정."""
    a = np.nan_to_num(a - np.nanmean(a))
    b = np.nan_to_num(b - np.nanmean(b))

    # 해닝 창으로 경계 효과 억제
    wy = np.hanning(a.shape[0])[:, None]
    wx = np.hanning(a.shape[1])[None, :]
    a = a * wy * wx
    b = b * wy * wx

    fa = np.fft.fft2(a)
    fb = np.fft.fft2(b)
    cross = fa * np.conj(fb)
    denom = np.abs(cross)
    denom[denom < 1e-12] = 1e-12
    r = np.fft.ifft2(cross / denom).real

    peak = np.unravel_index(np.argmax(r), r.shape)
    dy, dx = peak
    if dy > a.shape[0] // 2:
        dy -= a.shape[0]
    if dx > a.shape[1] // 2:
        dx -= a.shape[1]

    strength = float(r.max() / (r.std() + 1e-12))
    return float(dy), float(dx), strength


def assess_coregistration(before: np.ndarray, after: np.ndarray,
                          tile: int = 512, samples: int = 9):
    """여러 타일에서 이동량을 측정해 전역/국부 오정합을 진단."""
    h, w = before.shape
    step_y = max(1, (h - tile) // 3)
    step_x = max(1, (w - tile) // 3)

    shifts = []
    for y in range(0, h - tile + 1, step_y):
        for x in range(0, w - tile + 1, step_x):
            a = before[y:y + tile, x:x + tile]
            b = after[y:y + tile, x:x + tile]
            if not np.isfinite(a).any() or not np.isfinite(b).any():
                continue
            dy, dx, s = phase_correlation_shift(a, b)
            shifts.append((dy, dx, s))
            if len(shifts) >= samples:
                break

    if not shifts:
        return None

    arr = np.array(shifts)
    return {
        "median_dy": float(np.median(arr[:, 0])),
        "median_dx": float(np.median(arr[:, 1])),
        "spread_dy": float(np.std(arr[:, 0])),
        "spread_dx": float(np.std(arr[:, 1])),
        "mean_strength": float(np.mean(arr[:, 2])),
        "n_samples": len(shifts),
    }
```

해석 지침:

| 지표 | 판정 |
|---|---|
| median 이동 < 0.5 px | 정합 양호 |
| median 이동 0.5~2 px | 전역 평행이동 보정 필요 |
| median 이동 > 2 px | 재정사 또는 GCP 기반 재정합 필요 |
| spread > 1 px | 국부 왜곡 존재. 단순 평행이동으로 해결 불가 |
| mean_strength < 5 | 상관 신뢰 부족. 자동 판정 불가 |

표 14-1. 정합 진단 지표 해석

> **WARNING**
> `spread`가 크면 지형 기복에 의한 시차(parallax)일 가능성이 높다.
> 이 경우 DEM을 이용한 정사보정을 다시 수행해야 하며,
> 평행이동 보정을 강행하면 일부 지역만 맞고 나머지는 더 어긋난다.

## 14.3 평행이동 보정

```python
def apply_shift(arr: np.ndarray, dy: float, dx: float) -> np.ndarray:
    """정수 픽셀 평행이동. 부분 픽셀은 반올림."""
    iy, ix = int(round(dy)), int(round(dx))
    out = np.full(arr.shape, np.nan, dtype="float32")

    ys_src = slice(max(0, -iy), arr.shape[0] - max(0, iy))
    ys_dst = slice(max(0, iy), arr.shape[0] - max(0, -iy))
    xs_src = slice(max(0, -ix), arr.shape[1] - max(0, ix))
    xs_dst = slice(max(0, ix), arr.shape[1] - max(0, -ix))

    out[ys_dst, xs_dst] = arr[ys_src, xs_src]
    return out
```

부분 픽셀 정밀도가 필요하면 `gdal.Warp()`의 GCP 기반 변환을 사용한다.

```python
from osgeo import gdal

def warp_with_shift(src_path: str, out_path: str,
                    shift_x_m: float, shift_y_m: float) -> str:
    ds = gdal.Open(src_path)
    gt = list(ds.GetGeoTransform())
    gt[0] += shift_x_m
    gt[3] += shift_y_m

    driver = gdal.GetDriverByName("GTiff")
    out = driver.CreateCopy(out_path, ds,
                            options=["COMPRESS=DEFLATE", "TILED=YES"])
    out.SetGeoTransform(tuple(gt))
    out = None
    ds = None
    return out_path
```

## 14.4 GCP 기반 정합 품질 보고

지상기준점이 있으면 정량적 품질 보고가 가능하다.

```python
# analysis/gcp_qa.py
from __future__ import annotations
from dataclasses import dataclass
from typing import List
import math


@dataclass
class GcpResidual:
    gcp_id: str
    dx: float
    dy: float

    @property
    def dist(self) -> float:
        return math.hypot(self.dx, self.dy)


def rmse(residuals: List[GcpResidual]) -> dict:
    n = len(residuals)
    if n == 0:
        return {"n": 0}
    sx = sum(r.dx ** 2 for r in residuals) / n
    sy = sum(r.dy ** 2 for r in residuals) / n
    dists = sorted(r.dist for r in residuals)
    return {
        "n": n,
        "rmse_x": math.sqrt(sx),
        "rmse_y": math.sqrt(sy),
        "rmse_total": math.sqrt(sx + sy),
        "max": dists[-1],
        "p95": dists[int(0.95 * (n - 1))],
    }


def ce90(residuals: List[GcpResidual]) -> float:
    """수평 원형오차 90%. 영상 위치정확도의 표준 표기."""
    r = rmse(residuals)
    if r["n"] == 0:
        return float("nan")
    return 2.146 * (r["rmse_x"] + r["rmse_y"]) / 2.0
```

> **API**
> CE90은 "이 영상에서 읽은 좌표는 90% 확률로 실제 위치의 X m 이내"를 뜻한다.
> 산출물 메타데이터에 반드시 포함해야 하는 값이다.
> CE90이 15 m인 영상에서 MGRS 1 m 정밀도를 기록하면 방어할 수 없다.

## 14.5 정합 품질을 레시피에 기록

```python
from ..core.recipe import Recipe

def build_coreg_recipe(before: str, after: str, diag: dict,
                       applied_shift: tuple) -> Recipe:
    return Recipe(
        operation="coregistration",
        inputs={"before": before, "after": after},
        params={
            "median_dy_px": diag["median_dy"],
            "median_dx_px": diag["median_dx"],
            "spread_dy_px": diag["spread_dy"],
            "spread_dx_px": diag["spread_dx"],
            "correlation_strength": diag["mean_strength"],
            "applied_shift_px": list(applied_shift),
            "method": "phase_correlation",
        },
    )
```

### 버전 호환 노트 (Chapter 14)

- `numpy.fft` 기반 구현이라 QGIS 버전 의존성이 없다.
- `gdal.Warp()` 옵션 중 `transformerOptions`는 GDAL 2.3 이상 필요.
- QGIS 3.6 번들 numpy는 1.16 계열이며 위 코드는 모두 호환된다.

---

# Chapter 15. AI 추론 통합

## 15.1 범위 한정

이 장은 **학습된 모델을 플러그인에서 실행하는 방법**만 다룬다.
모델 학습, 데이터 라벨링, 아키텍처 설계는 범위 밖이다.

전제:

- 모델은 ONNX 형식으로 반입된다.
- 추론은 CPU에서 수행한다 (분석망 GPU 부재 가정).
- 모델 카드(입력 크기, 정규화 방식, 클래스 정의)가 함께 제공된다.

> **WARNING**
> 모델 카드 없는 모델은 사용하지 않는다.
> 입력 정규화 방식을 모르면 추론 결과가 조용히 틀린다.
> 오류가 나지 않고 "그럴듯한 잘못된 결과"가 나오므로 가장 위험하다.

## 15.2 모델 기술 파일

```json
{
  "name": "building-seg-v3",
  "task": "segmentation",
  "input": {
    "shape": [1, 3, 512, 512],
    "layout": "NCHW",
    "dtype": "float32",
    "normalize": {"mean": [0.485, 0.456, 0.406],
                  "std": [0.229, 0.224, 0.225]},
    "band_order": ["red", "green", "blue"],
    "value_range": [0, 255]
  },
  "output": {
    "shape": [1, 2, 512, 512],
    "classes": ["background", "building"],
    "activation": "softmax"
  },
  "training": {
    "gsd_range_m": [0.3, 0.8],
    "regions": ["temperate urban"],
    "date": "2025-11"
  },
  "limitations": [
    "GSD 1 m 이상 영상에서 성능 급감",
    "적설 조건 미학습",
    "고밀도 저층 주거지 과소 탐지 경향"
  ]
}
```

`limitations` 항목이 방어가능성 원칙과 직결된다.
GeoINT 산출물에는 "이 탐지는 적설 조건에서 검증되지 않았음" 같은 한계가 반드시 따라붙어야 한다.

```python
# analysis/model_card.py
from __future__ import annotations
import json
from dataclasses import dataclass
from typing import Any, Dict, List, Optional


@dataclass
class ModelCard:
    name: str
    task: str
    input_spec: Dict[str, Any]
    output_spec: Dict[str, Any]
    training: Dict[str, Any]
    limitations: List[str]

    @classmethod
    def load(cls, path: str) -> "ModelCard":
        with open(path, "r", encoding="utf-8") as f:
            d = json.load(f)
        return cls(
            name=d["name"], task=d["task"],
            input_spec=d["input"], output_spec=d["output"],
            training=d.get("training", {}),
            limitations=d.get("limitations", []),
        )

    def check_applicability(self, gsd_m: float) -> List[str]:
        """적용 가능성 경고 목록을 반환한다."""
        warnings = []
        rng = self.training.get("gsd_range_m")
        if rng and not (rng[0] <= gsd_m <= rng[1]):
            warnings.append(
                "영상 GSD {0:.2f} m가 학습 범위 {1}~{2} m를 벗어납니다."
                .format(gsd_m, rng[0], rng[1]))
        return warnings
```

## 15.3 타일 기반 추론

대형 영상은 모델 입력 크기로 잘라 처리하고 다시 합친다.
경계 아티팩트를 줄이기 위해 겹침(overlap)과 가중 병합을 사용한다.

```python
# analysis/inference.py
from __future__ import annotations
import numpy as np
from typing import Callable, Optional

from ..core.optional import GATES
from .model_card import ModelCard


class OnnxRunner:
    def __init__(self, model_path: str, card: ModelCard, threads: int = 4):
        GATES.require("onnxruntime", "AI 추론")
        import onnxruntime as ort

        opts = ort.SessionOptions()
        opts.intra_op_num_threads = threads
        opts.graph_optimization_level = \
            ort.GraphOptimizationLevel.ORT_ENABLE_ALL

        self.session = ort.InferenceSession(
            model_path, sess_options=opts, providers=["CPUExecutionProvider"])
        self.card = card
        self.input_name = self.session.get_inputs()[0].name

    def _preprocess(self, tile: np.ndarray) -> np.ndarray:
        spec = self.card.input_spec
        lo, hi = spec.get("value_range", [0, 255])
        x = np.clip(tile.astype("float32"), lo, hi) / float(hi - lo)

        norm = spec.get("normalize")
        if norm:
            mean = np.array(norm["mean"], dtype="float32").reshape(-1, 1, 1)
            std = np.array(norm["std"], dtype="float32").reshape(-1, 1, 1)
            x = (x - mean) / std
        return x[None, ...]           # NCHW

    def run_tile(self, tile: np.ndarray) -> np.ndarray:
        out = self.session.run(None, {self.input_name: self._preprocess(tile)})
        return out[0][0]              # (C, H, W)


def _blend_weights(size: int, overlap: int) -> np.ndarray:
    """가장자리로 갈수록 가중치가 낮아지는 창."""
    w = np.ones(size, dtype="float32")
    if overlap > 0:
        ramp = np.linspace(0.0, 1.0, overlap, dtype="float32")
        w[:overlap] = ramp
        w[-overlap:] = ramp[::-1]
    return w


def infer_large(image: np.ndarray,
                runner: OnnxRunner,
                tile: int = 512,
                overlap: int = 64,
                class_index: int = 1,
                progress: Optional[Callable[[int], bool]] = None
                ) -> np.ndarray:
    """(C,H,W) 입력 영상에 대해 타일 추론 후 가중 병합."""
    _, h, w = image.shape
    acc = np.zeros((h, w), dtype="float32")
    wsum = np.zeros((h, w), dtype="float32")

    step = tile - overlap
    ys = list(range(0, max(1, h - overlap), step))
    xs = list(range(0, max(1, w - overlap), step))
    total = len(ys) * len(xs)
    done = 0

    wy = _blend_weights(tile, overlap)[:, None]
    wx = _blend_weights(tile, overlap)[None, :]
    weight = wy * wx

    for y in ys:
        for x in xs:
            y1, x1 = min(y + tile, h), min(x + tile, w)
            patch = np.zeros((image.shape[0], tile, tile), dtype="float32")
            patch[:, :y1 - y, :x1 - x] = image[:, y:y1, x:x1]

            prob = runner.run_tile(patch)[class_index]

            acc[y:y1, x:x1] += prob[:y1 - y, :x1 - x] * weight[:y1 - y, :x1 - x]
            wsum[y:y1, x:x1] += weight[:y1 - y, :x1 - x]

            done += 1
            if progress is not None:
                if not progress(int(done * 100 / total)):
                    raise InterruptedError("사용자가 추론을 취소했습니다.")

    wsum[wsum < 1e-6] = 1.0
    return acc / wsum
```

## 15.4 AI 결과의 표기 원칙

AI 추론 결과는 다른 분석보다 **더 강한 표기 의무**가 있다.

```python
def build_inference_provenance(card: ModelCard, gsd_m: float,
                               threshold: float, source_path: str):
    from ..core.provenance import Provenance
    from datetime import datetime
    import getpass, socket

    notes = list(card.limitations)
    notes.extend(card.check_applicability(gsd_m))
    notes.append("자동 추론 결과이며 분석관 검토 전 상태입니다.")

    return Provenance(
        operation="ai_inference",
        created_at=datetime.now().isoformat(timespec="seconds"),
        created_by=getpass.getuser(),
        host=socket.gethostname(),
        inputs={"image": source_path, "model": card.name},
        params={"threshold": threshold, "gsd_m": gsd_m,
                "task": card.task},
        confidence="C3",     # 자동 추론의 기본 등급
        notes=notes,
    )
```

> **ENGINEERING PRACTICE**
> 자동 추론 결과의 기본 신뢰도는 **C3(보통 출처 / 개연성 있음)** 을 넘지 않는다.
> 분석관이 검토하고 다른 출처로 교차 확인해야 B2 이상으로 승격할 수 있다.
> 이 승격 이력도 출처 정보에 남긴다.

## 15.5 심볼로지로 불확실성 표현

확률값을 이진 마스크로 눌러버리면 정보가 사라진다.
QGeoINT는 확률 래스터를 함께 보존하고, 벡터 결과에 신뢰 구간을 붙인다.

```python
# analysis/uncertainty.py
import numpy as np
from typing import Dict


def confidence_bands(prob: np.ndarray) -> Dict[str, float]:
    """확률 분포에서 확신도 구간별 픽셀 비율."""
    valid = np.isfinite(prob)
    total = float(valid.sum()) or 1.0
    return {
        "high (>=0.8)":   float((prob >= 0.8).sum()) / total,
        "medium (0.5-0.8)": float(((prob >= 0.5) & (prob < 0.8)).sum()) / total,
        "low (0.3-0.5)":  float(((prob >= 0.3) & (prob < 0.5)).sum()) / total,
    }


def ambiguity_index(prob: np.ndarray) -> float:
    """0.5 근처 픽셀 비율. 높을수록 모델이 헤맸다는 뜻."""
    valid = np.isfinite(prob)
    if not valid.any():
        return float("nan")
    near = np.abs(prob[valid] - 0.5) < 0.15
    return float(near.sum()) / float(valid.sum())
```

`ambiguity_index`가 0.3을 넘으면 자동으로 경고를 띄운다.
"모델이 이 영상에서 확신하지 못하고 있다"는 것은 분석관이 반드시 알아야 할 정보다.

### 버전 호환 노트 (Chapter 15)

- `onnxruntime`는 Python 3.7용 마지막 버전이 1.10 계열이다.
  QGIS 3.6 지원 시 이 버전 휠을 `py37/`에 넣는다.
- QGIS 4.x(Python 3.12)는 onnxruntime 1.17 이상이 필요하다.
- 두 버전 간 `SessionOptions` API는 호환되므로 코드 분기는 불필요하다.

---
---

# Part V. 지형 분석

---

# Chapter 16. DEM 처리와 파생 산출물

## 16.1 표고 자료의 구분

| 구분 | 의미 | GeoINT 용도 |
|---|---|---|
| DEM | 일반적 수치표고모델 | 총칭 |
| DTM | 지표면(구조물·식생 제거) | 침수, 유역, 기동성 분석 |
| DSM | 최상단 표면(건물·수목 포함) | 가시권, 통신, 그림자 분석 |
| nDSM | DSM − DTM (물체 높이) | 건물 높이, 수목 높이 |

표 16-1. 표고 자료 유형

> **WARNING**
> 가시권 분석에 DTM을 쓰면 건물이 없는 것으로 계산되어
> 실제와 전혀 다른 결과가 나온다. 반드시 DSM을 사용한다.
> 반대로 침수 분석에 DSM을 쓰면 건물 지붕이 지면으로 취급된다.
> **어떤 표고를 쓸 것인가는 분석 목적이 결정한다.**

## 16.2 수직 기준면

```text
타원체고 (h)  ─ GPS가 직접 측정
     │
     ├─ 지오이드고 (N) ─ 지오이드 모델에서 조회
     │
표고 (H) = h − N     ← 지도·설계에서 쓰는 높이
```

그림 16-1. 수직 기준면 관계

한국의 지오이드고는 대략 20~30 m 범위다. 혼동하면 수십 미터 오차가 발생한다.

```python
# analysis/vertical.py
from typing import Optional
from qgis.core import QgsPointXY


def ellipsoid_to_orthometric(h: float, geoid_undulation: float) -> float:
    """타원체고 → 표고."""
    return h - geoid_undulation


def sample_geoid(pt: QgsPointXY, geoid_layer) -> Optional[float]:
    """지오이드 모델 래스터에서 값을 조회."""
    from qgis.core import QgsRaster
    provider = geoid_layer.dataProvider()
    result = provider.identify(pt, QgsRaster.IdentifyFormatValue)
    if not result.isValid():
        return None
    values = result.results()
    return float(list(values.values())[0]) if values else None
```

## 16.3 지형 파생 계산

```python
# analysis/terrain.py
from __future__ import annotations
import numpy as np
from typing import Tuple


def _gradients(dem: np.ndarray, cellsize: float) -> Tuple[np.ndarray, np.ndarray]:
    """Horn 방식 3x3 경사 계산."""
    z = dem
    p = np.pad(z, 1, mode="edge")

    dzdx = ((p[:-2, 2:] + 2 * p[1:-1, 2:] + p[2:, 2:]) -
            (p[:-2, :-2] + 2 * p[1:-1, :-2] + p[2:, :-2])) / (8.0 * cellsize)
    dzdy = ((p[2:, :-2] + 2 * p[2:, 1:-1] + p[2:, 2:]) -
            (p[:-2, :-2] + 2 * p[:-2, 1:-1] + p[:-2, 2:])) / (8.0 * cellsize)
    return dzdx, dzdy


def slope_degrees(dem: np.ndarray, cellsize: float) -> np.ndarray:
    dzdx, dzdy = _gradients(dem, cellsize)
    return np.degrees(np.arctan(np.sqrt(dzdx ** 2 + dzdy ** 2))).astype("float32")


def aspect_degrees(dem: np.ndarray, cellsize: float) -> np.ndarray:
    """북쪽 0°, 시계방향."""
    dzdx, dzdy = _gradients(dem, cellsize)
    a = np.degrees(np.arctan2(dzdy, -dzdx))
    a = np.where(a < 0, a + 360.0, a)
    a = np.where((dzdx == 0) & (dzdy == 0), -1.0, a)   # 평지
    return a.astype("float32")


def hillshade(dem: np.ndarray, cellsize: float,
              azimuth: float = 315.0, altitude: float = 45.0) -> np.ndarray:
    slope = np.radians(slope_degrees(dem, cellsize))
    aspect = np.radians(aspect_degrees(dem, cellsize))
    az = np.radians(360.0 - azimuth + 90.0)
    alt = np.radians(altitude)

    shaded = (np.sin(alt) * np.cos(slope) +
              np.cos(alt) * np.sin(slope) * np.cos(az - aspect))
    return (np.clip(shaded, 0, 1) * 255).astype("uint8")


def tri(dem: np.ndarray) -> np.ndarray:
    """지형 기복 지수 (Terrain Ruggedness Index)."""
    p = np.pad(dem, 1, mode="edge")
    center = p[1:-1, 1:-1]
    acc = np.zeros(center.shape, dtype="float32")
    for dy in (-1, 0, 1):
        for dx in (-1, 0, 1):
            if dy == 0 and dx == 0:
                continue
            n = p[1 + dy:p.shape[0] - 1 + dy, 1 + dx:p.shape[1] - 1 + dx]
            acc += (n - center) ** 2
    return np.sqrt(acc).astype("float32")
```

## 16.4 DEM 품질 점검

인제스트 단계에서 DEM 자체의 품질을 반드시 확인한다.

```python
# analysis/dem_qa.py
from __future__ import annotations
import numpy as np
from typing import Dict


def dem_quality_report(dem: np.ndarray, cellsize: float,
                       nodata_mask: np.ndarray = None) -> Dict[str, float]:
    valid = np.isfinite(dem)
    if nodata_mask is not None:
        valid &= ~nodata_mask

    v = dem[valid]
    if v.size == 0:
        return {"error": 1.0}

    from .terrain import slope_degrees
    slp = slope_degrees(np.nan_to_num(dem, nan=float(np.nanmean(dem))), cellsize)

    return {
        "coverage_ratio": float(valid.sum()) / float(dem.size),
        "min": float(v.min()),
        "max": float(v.max()),
        "mean": float(v.mean()),
        "std": float(v.std()),
        "p1": float(np.percentile(v, 1)),
        "p99": float(np.percentile(v, 99)),
        "slope_gt_60_ratio": float((slp[valid] > 60).sum()) / float(valid.sum()),
        "spike_ratio": float((np.abs(v - np.median(v)) > 6 * v.std()).sum())
                       / float(v.size),
    }
```

경고 기준:

| 지표 | 임계 | 의미 |
|---|---|---|
| `coverage_ratio` < 0.95 | 결측 과다 | 보간 또는 보완 자료 필요 |
| `min` < −100 m | 이상값 | NoData 미설정 의심 |
| `max` > 3000 m (국내) | 이상값 | 단위 혼동(피트/미터) 의심 |
| `slope_gt_60_ratio` > 0.05 | 급경사 과다 | 잡음 또는 건물 포함(DSM) |
| `spike_ratio` > 0.001 | 스파이크 | 필터링 필요 |

표 16-2. DEM 품질 경고 기준

### 버전 호환 노트 (Chapter 16)

- `QgsRaster.IdentifyFormatValue`는 3.6~4.x 동일하다.
- QGIS 내장 `native:slope`, `native:aspect` Processing 알고리즘도 사용 가능하나,
  타일 처리와 numpy 통합을 위해 자체 구현을 사용했다.
- `np.percentile`의 `interpolation` 인자는 numpy 1.22에서 `method`로 바뀌었다.
  기본값 사용 시 문제없다.

---

# Chapter 17. 가시권과 통달 분석

## 17.1 활용 맥락

가시권(viewshed) 분석은 다음 업무에 쓰인다.

- CCTV·관측소 설치 위치 선정 및 사각지대 확인
- 통신 중계기 커버리지 설계 (전파 통달)
- 경관 영향 평가 (풍력발전기, 송전탑 가시 범위)
- 재난 상황에서 관측 가능 지점 산정
- 촬영 위치 검증 — 특정 지점에서 그 장면이 실제로 보이는지 (Chapter 20)

## 17.2 시선(LOS) 계산

```python
# analysis/viewshed.py
from __future__ import annotations
import numpy as np
from typing import Optional, Tuple


def line_of_sight(dem: np.ndarray, cellsize: float,
                  y0: int, x0: int, y1: int, x1: int,
                  observer_h: float = 1.7,
                  target_h: float = 0.0,
                  earth_curvature: bool = True,
                  refraction_k: float = 0.13) -> bool:
    """관측점(y0,x0)에서 목표점(y1,x1)이 보이는지 판정."""
    n = int(max(abs(y1 - y0), abs(x1 - x0)))
    if n == 0:
        return True

    ys = np.linspace(y0, y1, n + 1)
    xs = np.linspace(x0, x1, n + 1)
    yi = np.clip(np.round(ys).astype(int), 0, dem.shape[0] - 1)
    xi = np.clip(np.round(xs).astype(int), 0, dem.shape[1] - 1)

    profile = dem[yi, xi].astype("float64")
    dist = np.hypot((ys - y0) * cellsize, (xs - x0) * cellsize)

    if earth_curvature:
        R = 6371000.0
        drop = (dist ** 2) / (2.0 * R) * (1.0 - refraction_k)
        profile = profile - drop

    z_obs = profile[0] + observer_h
    z_tgt = profile[-1] + target_h
    total = dist[-1]
    if total <= 0:
        return True

    # 중간 지점들의 필요 고도 (직선 보간)
    required = z_obs + (z_tgt - z_obs) * (dist[1:-1] / total)
    return bool(np.all(profile[1:-1] <= required + 1e-6))


def viewshed(dem: np.ndarray, cellsize: float,
             y0: int, x0: int,
             radius_px: int,
             observer_h: float = 1.7,
             target_h: float = 0.0,
             progress=None) -> np.ndarray:
    """R2 방식 방사형 가시권. 원주 방향으로 광선을 쏜다."""
    h, w = dem.shape
    visible = np.zeros((h, w), dtype=bool)
    visible[y0, x0] = True

    R = 6371000.0
    k = 0.13
    z0 = dem[y0, x0] + observer_h

    n_rays = int(2 * np.pi * radius_px)
    n_rays = max(n_rays, 360)

    for i in range(n_rays):
        theta = 2.0 * np.pi * i / n_rays
        dy, dx = np.sin(theta), np.cos(theta)

        max_slope = -np.inf
        for step in range(1, radius_px + 1):
            yy = int(round(y0 + dy * step))
            xx = int(round(x0 + dx * step))
            if not (0 <= yy < h and 0 <= xx < w):
                break

            d = step * cellsize
            z = dem[yy, xx]
            if not np.isfinite(z):
                continue
            z -= (d * d) / (2.0 * R) * (1.0 - k)

            slope_to_target = (z + target_h - z0) / d
            if slope_to_target >= max_slope:
                visible[yy, xx] = True
            slope_to_surface = (z - z0) / d
            if slope_to_surface > max_slope:
                max_slope = slope_to_surface

        if progress is not None and i % 32 == 0:
            if not progress(int(i * 100 / n_rays)):
                raise InterruptedError("가시권 계산이 취소되었습니다.")

    return visible
```

> **TIP**
> R2 방식은 방사형으로 광선을 쏘기 때문에 먼 거리에서 광선 사이 간격이 벌어져
> 계산되지 않는 픽셀이 생긴다. `n_rays`를 반경에 비례해 늘리는 것이 위 코드의 대응이다.
> 정밀도가 더 필요하면 QGIS의 Visibility Analysis 플러그인이나
> GRASS `r.viewshed`를 Processing으로 호출한다.

## 17.3 GRASS 알고리즘 호출

정확도가 중요한 경우 검증된 구현을 쓰는 편이 낫다.

```python
# analysis/viewshed_grass.py
import processing
from typing import Optional


def run_grass_viewshed(dem_layer, x: float, y: float,
                       observer_h: float, max_dist: float,
                       output: str) -> Optional[str]:
    try:
        result = processing.run("grass7:r.viewshed", {
            "input": dem_layer,
            "coordinates": "{0},{1}".format(x, y),
            "observer_elevation": observer_h,
            "target_elevation": 0.0,
            "max_distance": max_dist,
            "refraction_coeff": 0.14286,
            "-c": True,      # 지구 곡률 보정
            "-b": True,      # 이진 출력
            "output": output,
        })
        return result["output"]
    except Exception:
        return None
```

> **호환성**
> QGIS 3.x는 provider ID가 `grass7`, QGIS 4.x부터는 `grass`로 바뀐 경우가 있다.
> 다음처럼 레지스트리에서 조회해 분기한다.

```python
from qgis.core import QgsApplication

def grass_prefix() -> str:
    reg = QgsApplication.processingRegistry()
    for pid in ("grass7", "grass"):
        if reg.providerById(pid) is not None:
            return pid
    raise RuntimeError("GRASS Processing 공급자를 찾을 수 없습니다.")
```

## 17.4 다중 관측점 통합 커버리지

```python
def cumulative_viewshed(dem: np.ndarray, cellsize: float,
                        observers, radius_px: int) -> np.ndarray:
    """여러 관측점의 가시권을 누적. 값 = 몇 개 지점에서 보이는가."""
    count = np.zeros(dem.shape, dtype="int16")
    for (y0, x0, h) in observers:
        v = viewshed(dem, cellsize, y0, x0, radius_px, observer_h=h)
        count += v.astype("int16")
    return count


def coverage_gaps(count: np.ndarray, aoi_mask: np.ndarray) -> dict:
    """사각지대 통계."""
    inside = aoi_mask
    total = float(inside.sum()) or 1.0
    return {
        "covered_ratio": float((count[inside] > 0).sum()) / total,
        "redundant_ratio": float((count[inside] >= 2).sum()) / total,
        "gap_ratio": float((count[inside] == 0).sum()) / total,
    }
```

`redundant_ratio`는 중복 관측 비율이다.
관측 자원 배치 검토에서 "중복이 과한 구역"과 "사각지대"를 동시에 볼 수 있다.

## 17.5 프레넬 존 — 전파 통달 판단

시선이 통해도 전파는 통하지 않을 수 있다.
제1 프레넬 존의 60% 이상이 확보되어야 실용적 통신이 가능하다.

```python
# analysis/fresnel.py
import numpy as np


def fresnel_radius(d1_m: float, d2_m: float, freq_hz: float, n: int = 1) -> float:
    """n차 프레넬 존 반경(m)."""
    c = 299792458.0
    wavelength = c / freq_hz
    total = d1_m + d2_m
    if total <= 0:
        return 0.0
    return float(np.sqrt(n * wavelength * d1_m * d2_m / total))


def fresnel_clearance(profile_z: np.ndarray, dist: np.ndarray,
                      z_tx: float, z_rx: float, freq_hz: float) -> float:
    """경로상 최소 프레넬 여유 비율. 0.6 이상이면 양호."""
    total = dist[-1]
    if total <= 0:
        return 1.0

    los = z_tx + (z_rx - z_tx) * (dist / total)
    clearance = los - profile_z

    ratios = []
    for i in range(1, len(dist) - 1):
        r = fresnel_radius(dist[i], total - dist[i], freq_hz)
        if r > 1e-6:
            ratios.append(clearance[i] / r)
    return float(min(ratios)) if ratios else 1.0
```

| 프레넬 여유 비율 | 판정 |
|---|---|
| ≥ 1.0 | 완전 확보 |
| 0.6 ~ 1.0 | 실용적 통신 가능 |
| 0.2 ~ 0.6 | 감쇠 상당, 불안정 |
| < 0.2 | 사실상 불통 |

표 17-1. 프레넬 여유 비율 판정

### 버전 호환 노트 (Chapter 17)

- GRASS 알고리즘 ID는 QGIS 버전에 따라 `grass7:`/`grass:` 접두어가 다르다.
- QGIS 3.6에서는 `processing.run()` 호출 전 `Processing.initialize()`가 필요한 경우가 있다.
- 순수 numpy 구현은 버전 의존성이 없다.

---

# Chapter 18. 접근성과 경로 분석

## 18.1 문제 정의

재난 대응과 현장 조사 계획에서 반복되는 질문은 다음과 같다.

- 이 지점까지 차량으로 몇 분 걸리는가
- 도로가 끊겼을 때 대체 경로는 무엇인가
- 30분 내에 도달 가능한 범위는 어디까지인가
- 하천을 건널 수 있는 지점은 어디인가

이 질문들은 모두 **비용면(cost surface) → 누적비용 → 경로** 라는 동일한 구조로 푼다.

## 18.2 비용면 구성

```python
# analysis/cost.py
from __future__ import annotations
import numpy as np
from typing import Dict, Optional


def slope_cost(slope_deg: np.ndarray,
               max_slope: float = 30.0) -> np.ndarray:
    """경사에 따른 통행 비용 배수. 초과 시 통행 불가(inf)."""
    cost = 1.0 + (slope_deg / 10.0) ** 2
    cost = np.where(slope_deg > max_slope, np.inf, cost)
    return cost.astype("float32")


LANDCOVER_COST = {
    1: 1.0,    # 도로
    2: 1.5,    # 나지
    3: 2.5,    # 초지
    4: 5.0,    # 산림
    5: np.inf, # 수역
    6: 8.0,    # 시가지(건물 밀집)
}


def landcover_cost(landcover: np.ndarray) -> np.ndarray:
    out = np.ones(landcover.shape, dtype="float32")
    for code, c in LANDCOVER_COST.items():
        out[landcover == code] = c
    return out


def combine_cost(*layers: np.ndarray) -> np.ndarray:
    out = np.ones(layers[0].shape, dtype="float32")
    for l in layers:
        out = out * l
    return out
```

## 18.3 누적비용 계산

다익스트라 방식으로 시작점에서의 누적 비용을 계산한다.

```python
# analysis/accumulate.py
from __future__ import annotations
import heapq
import numpy as np
from typing import List, Tuple, Optional


NEIGHBORS = [(-1, 0, 1.0), (1, 0, 1.0), (0, -1, 1.0), (0, 1, 1.0),
             (-1, -1, 1.41421356), (-1, 1, 1.41421356),
             (1, -1, 1.41421356), (1, 1, 1.41421356)]


def accumulate_cost(cost: np.ndarray, cellsize: float,
                    sources: List[Tuple[int, int]],
                    max_cost: Optional[float] = None,
                    progress=None) -> np.ndarray:
    """다익스트라 누적비용. 반환 단위는 cost x 거리(m)."""
    h, w = cost.shape
    acc = np.full((h, w), np.inf, dtype="float32")
    visited = np.zeros((h, w), dtype=bool)

    heap = []
    for (y, x) in sources:
        acc[y, x] = 0.0
        heapq.heappush(heap, (0.0, y, x))

    processed = 0
    total = h * w

    while heap:
        c, y, x = heapq.heappop(heap)
        if visited[y, x]:
            continue
        visited[y, x] = True
        processed += 1

        if max_cost is not None and c > max_cost:
            continue

        base = cost[y, x]
        if not np.isfinite(base):
            continue

        for dy, dx, diag in NEIGHBORS:
            ny, nx = y + dy, x + dx
            if not (0 <= ny < h and 0 <= nx < w) or visited[ny, nx]:
                continue
            step_cost = cost[ny, nx]
            if not np.isfinite(step_cost):
                continue
            edge = (base + step_cost) / 2.0 * diag * cellsize
            nc = c + edge
            if nc < acc[ny, nx]:
                acc[ny, nx] = nc
                heapq.heappush(heap, (nc, ny, nx))

        if progress is not None and processed % 20000 == 0:
            if not progress(min(99, int(processed * 100 / total))):
                raise InterruptedError("누적비용 계산이 취소되었습니다.")

    return acc
```

## 18.4 최소비용 경로 역추적

```python
def least_cost_path(acc: np.ndarray, target: Tuple[int, int]
                    ) -> List[Tuple[int, int]]:
    """누적비용면에서 목표점 → 시작점 경로를 역추적."""
    h, w = acc.shape
    y, x = target
    if not np.isfinite(acc[y, x]):
        raise ValueError("목표점에 도달할 수 없습니다.")

    path = [(y, x)]
    guard = 0
    while acc[y, x] > 0 and guard < h * w:
        guard += 1
        best = None
        best_val = acc[y, x]
        for dy, dx, _ in NEIGHBORS:
            ny, nx = y + dy, x + dx
            if 0 <= ny < h and 0 <= nx < w and acc[ny, nx] < best_val:
                best_val = acc[ny, nx]
                best = (ny, nx)
        if best is None:
            break
        y, x = best
        path.append((y, x))
    return path[::-1]
```

## 18.5 도달권(isochrone) 산출

```python
def isochrone_bands(acc: np.ndarray, breaks: List[float]) -> np.ndarray:
    """누적비용을 구간으로 분류. breaks는 오름차순."""
    out = np.zeros(acc.shape, dtype="uint8")
    prev = 0.0
    for i, b in enumerate(breaks, start=1):
        out[(acc > prev) & (acc <= b)] = i
        prev = b
    out[np.isinf(acc)] = 255      # 도달 불가
    return out
```

시간 기준 도달권을 만들려면 비용을 시간 단위로 정의한다.

```python
def travel_time_cost(speed_kmh: np.ndarray) -> np.ndarray:
    """속도(km/h) 래스터 → 단위거리당 소요시간(초/m)."""
    speed_ms = speed_kmh / 3.6
    out = np.full(speed_kmh.shape, np.inf, dtype="float32")
    valid = speed_ms > 0.01
    out[valid] = 1.0 / speed_ms[valid]
    return out
```

## 18.6 도로망 기반 경로 — QgsGraph

지형 기반이 아니라 실제 도로망을 따라야 하는 경우 QGIS 네트워크 분석 API를 쓴다.

```python
# analysis/network.py
from qgis.core import (
    QgsVectorLayer, QgsPointXY, QgsGeometry, QgsFeature,
)
from qgis.analysis import (
    QgsVectorLayerDirector, QgsNetworkDistanceStrategy,
    QgsGraphBuilder, QgsGraphAnalyzer,
)
from typing import List, Optional


def shortest_path(road_layer: QgsVectorLayer,
                  start: QgsPointXY, end: QgsPointXY,
                  direction_field: int = -1) -> Optional[QgsGeometry]:
    director = QgsVectorLayerDirector(
        road_layer, direction_field, "1", "-1", "both",
        QgsVectorLayerDirector.DirectionBoth)
    director.addStrategy(QgsNetworkDistanceStrategy())

    builder = QgsGraphBuilder(road_layer.crs())
    tied = director.makeGraph(builder, [start, end])
    graph = builder.graph()

    start_id = graph.findVertex(tied[0])
    end_id = graph.findVertex(tied[1])
    if start_id < 0 or end_id < 0:
        return None

    tree, cost = QgsGraphAnalyzer.dijkstra(graph, start_id, 0)
    if tree[end_id] == -1:
        return None

    points = []
    cur = end_id
    while cur != start_id:
        points.append(graph.vertex(cur).point())
        cur = graph.edge(tree[cur]).fromVertex()
    points.append(graph.vertex(start_id).point())
    points.reverse()

    return QgsGeometry.fromPolylineXY(points)
```

> **호환성**
> `QgsGraph.edge().fromVertex()`는 QGIS 3.12에서 시그니처가 변경되었다.
> 3.6~3.10은 `outVertex()`/`inVertex()`를 사용한다. `compat.py`에 래퍼를 둔다.

```python
# compat.py 추가분
def edge_from_vertex(graph, edge_id: int) -> int:
    edge = graph.edge(edge_id)
    getter = getattr(edge, "fromVertex", None) or getattr(edge, "outVertex")
    return getter()


def edge_to_vertex(graph, edge_id: int) -> int:
    edge = graph.edge(edge_id)
    getter = getattr(edge, "toVertex", None) or getattr(edge, "inVertex")
    return getter()
```

### 버전 호환 노트 (Chapter 18)

- `qgis.analysis` 네트워크 API는 3.6부터 존재하나 메서드명이 3.12에서 변경되었다.
- `QgsGraphAnalyzer.dijkstra()`는 전 버전 동일하다.
- 누적비용 순수 구현은 대형 래스터에서 느리다.
  실무에서는 GRASS `r.cost` 또는 SAGA 알고리즘 호출을 우선 검토한다.

---
---

# Part VI. 공개출처정보(OSINT) 융합

---

# Chapter 19. OSINT 수집 파이프라인

## 19.1 이 파트의 전제

Part VI은 **공개된 정보를 이용해 장소와 사건을 검증하는 방법**을 다룬다.
언론사 팩트체크 팀, 재난 대응 기관, 인권 조사 기관이 사용하는 것과 같은 방법론이다.

동시에 다음은 이 책의 범위에서 명시적으로 제외한다.

- 특정 개인의 위치·이동 경로 추적
- 비공개 계정 접근, 인증 우회, 스크래핑 차단 회피
- 서비스 이용약관을 위반하는 자동 수집
- 개인 식별이 가능한 자료의 무단 축적

> **WARNING**
> "기술적으로 가능한가"와 "해도 되는가"는 다른 질문이다.
> 이 파트의 모든 기법은 **장소와 사건**을 대상으로 하며,
> **사람**을 대상으로 삼는 순간 다른 법·윤리 체계가 적용된다. Chapter 29를 반드시 함께 읽는다.

## 19.2 공개 출처의 유형

| 유형 | 예 | 지리 정보 |
|---|---|---|
| 공공 데이터 | 통계청, 공공데이터포털, 기상청 | 행정구역, 관측소 |
| 지도 데이터 | OSM, 국가공간정보포털 | 도로, 건물, 지명 |
| 위성영상 | Sentinel, Landsat 공개 아카이브 | 좌표 포함 |
| 뉴스·보도 | 언론 기사, 보도자료 | 지명 텍스트 |
| 소셜 미디어 | 공개 게시물 | 지오태그, 배경 단서 |
| 센서 공개망 | AIS, ADS-B, 지진계 | 좌표 스트림 |
| 기업 공시 | 사업보고서, 인허가 | 주소 |

표 19-1. 공개 출처 유형과 지리 정보

## 19.3 수집 인터페이스 추상화

폐쇄망에서는 온라인 수집이 불가능하므로,
**수집기(collector)와 적재기(loader)를 분리**한다.

```python
# fusion/collector.py
from __future__ import annotations
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime
from typing import Any, Dict, List, Optional


@dataclass
class Observation:
    """출처 무관 공통 관측 레코드."""
    obs_id: str
    source: str                     # 출처 식별자
    source_grade: str = "C"         # Admiralty A~F
    cert_grade: str = "3"           # 1~6
    lon: Optional[float] = None
    lat: Optional[float] = None
    position_error_m: Optional[float] = None
    observed_at: Optional[str] = None
    text: str = ""
    url: str = ""
    attrs: Dict[str, Any] = field(default_factory=dict)

    @property
    def has_position(self) -> bool:
        return self.lon is not None and self.lat is not None


class Collector(ABC):
    """온라인 환경에서 실행되는 수집기."""

    name = "base"

    @abstractmethod
    def collect(self, **kwargs) -> List[Observation]:
        ...


class Loader(ABC):
    """폐쇄망에서 실행되는 적재기. 파일만 읽는다."""

    @abstractmethod
    def load(self, path: str) -> List[Observation]:
        ...
```

수집은 인터넷 구간에서 수행해 JSONL 파일로 내보내고,
분석망에서는 그 파일만 적재한다.

```python
# fusion/jsonl_io.py
import json
from dataclasses import asdict
from typing import Iterable, List
from .collector import Observation


def write_jsonl(observations: Iterable[Observation], path: str) -> int:
    n = 0
    with open(path, "w", encoding="utf-8") as f:
        for obs in observations:
            f.write(json.dumps(asdict(obs), ensure_ascii=False) + "\n")
            n += 1
    return n


def read_jsonl(path: str) -> List[Observation]:
    out = []
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if line:
                out.append(Observation(**json.loads(line)))
    return out
```

## 19.4 지명 → 좌표 (지오코딩)

폐쇄망에서는 온라인 지오코딩 API를 쓸 수 없다.
지명 사전을 사전 반입해 로컬 매칭한다.

```python
# fusion/gazetteer.py
from __future__ import annotations
import sqlite3
from typing import List, Optional, Tuple

DDL = """
CREATE TABLE IF NOT EXISTS gazetteer (
    id       INTEGER PRIMARY KEY,
    name     TEXT NOT NULL,
    name_norm TEXT NOT NULL,
    kind     TEXT,
    admin1   TEXT,
    admin2   TEXT,
    lon      REAL NOT NULL,
    lat      REAL NOT NULL,
    accuracy_m REAL
);
CREATE INDEX IF NOT EXISTS idx_gaz_norm ON gazetteer(name_norm);
"""


def normalize(name: str) -> str:
    return "".join(name.split()).lower()


class Gazetteer:
    def __init__(self, db_path: str):
        self.path = db_path

    def init(self):
        with sqlite3.connect(self.path) as c:
            c.executescript(DDL)

    def lookup(self, name: str, admin1: Optional[str] = None
               ) -> List[Tuple[float, float, str, float]]:
        sql = ("SELECT lon, lat, kind, COALESCE(accuracy_m, 1000) "
               "FROM gazetteer WHERE name_norm = ?")
        args = [normalize(name)]
        if admin1:
            sql += " AND admin1 = ?"
            args.append(admin1)
        with sqlite3.connect(self.path) as c:
            return c.execute(sql, args).fetchall()

    def resolve(self, name: str, admin1: Optional[str] = None):
        """모호성을 숨기지 않고 후보 수와 함께 반환."""
        hits = self.lookup(name, admin1)
        if not hits:
            return None, 0
        return hits[0], len(hits)
```

> **ENGINEERING PRACTICE**
> `resolve()`가 후보 개수를 함께 반환하는 이유는 방어가능성 때문이다.
> "신촌"처럼 동일 지명이 여러 곳에 있으면 자동 선택하지 않고
> 후보 수를 기록해 확실성 등급을 낮춘다.

```python
def cert_from_ambiguity(candidate_count: int) -> str:
    if candidate_count == 1:
        return "2"      # 개연성 높음
    if candidate_count <= 3:
        return "3"      # 개연성 있음
    return "4"          # 의심스러움
```

## 19.5 좌표 문자열 추출

텍스트에 좌표가 직접 포함된 경우가 있다.

```python
# fusion/coord_parse.py
from __future__ import annotations
import re
from typing import List, Tuple, Optional

DD = re.compile(
    r"(-?\d{1,3}\.\d{3,})\s*[,/]\s*(-?\d{1,3}\.\d{3,})")
DMS = re.compile(
    r"(\d{1,3})[°\s]+(\d{1,2})['\s]+(\d{1,2}(?:\.\d+)?)[\"\s]*([NSEW])",
    re.IGNORECASE)
MGRS_RE = re.compile(
    r"\b(\d{1,2}[C-HJ-NP-X])\s?([A-HJ-NP-Z]{2})\s?(\d{2,10})\b")


def dms_to_dd(deg: float, minute: float, sec: float, hemi: str) -> float:
    val = deg + minute / 60.0 + sec / 3600.0
    return -val if hemi.upper() in ("S", "W") else val


def extract_coordinates(text: str) -> List[Tuple[str, float, float]]:
    """(형식, lon, lat) 목록을 반환."""
    found = []

    for m in DD.finditer(text):
        a, b = float(m.group(1)), float(m.group(2))
        # 위경도 순서 추정: |lat| <= 90
        if abs(a) <= 90 and abs(b) <= 180:
            found.append(("DD", b, a))
        elif abs(b) <= 90 and abs(a) <= 180:
            found.append(("DD", a, b))

    dms_hits = DMS.findall(text)
    if len(dms_hits) >= 2:
        vals = {}
        for d, mm, s, hemi in dms_hits:
            v = dms_to_dd(float(d), float(mm), float(s), hemi)
            vals["lat" if hemi.upper() in "NS" else "lon"] = v
        if "lat" in vals and "lon" in vals:
            found.append(("DMS", vals["lon"], vals["lat"]))

    return found


def extract_mgrs(text: str) -> List[str]:
    return ["".join(m) for m in MGRS_RE.findall(text)]
```

> **WARNING**
> 위경도 순서 자동 추정은 위험하다. 한국(위도 33~38, 경도 124~132)에서는
> 두 값 모두 90 이하가 될 수 있어 순서를 뒤바꿔도 유효한 좌표가 된다.
> 결과가 예상 지역을 벗어나면 반드시 사용자에게 확인을 요청한다.

### 버전 호환 노트 (Chapter 19)

- `sqlite3`, `re`, `json`은 표준 라이브러리이므로 전 버전 호환된다.
- `dataclasses.asdict()`는 Python 3.7+.

---

# Chapter 20. 사진 기반 위치 검증

## 20.1 목적과 경계

이 장의 기법은 **"이 사진이 정말 이 장소에서 찍혔는가"를 검증**하기 위한 것이다.
재난 보도 검증, 허위정보 대응, 현장 조사 기록 확인 등에 쓰인다.

명확한 경계:

- 대상은 **장소**다. 사진 속 인물의 신원을 추정하는 데 사용하지 않는다.
- 검증은 **주장에 대한 반증 시도**다. "여기가 맞다"를 증명하려 들면 확증편향에 빠진다.
- 결론은 항상 등급으로 표현한다. "확정"이라는 결론은 거의 나오지 않는다.

## 20.2 검증 절차

```text
1. 주장 정리      "이 사진은 A 지점에서 B 방향을 촬영했다고 주장됨"
        ↓
2. 단서 추출      지형 윤곽, 건축물, 표지판, 식생, 그림자, 도로 형태
        ↓
3. 후보 지점 설정  지도·영상에서 조건을 만족하는 지점 목록화
        ↓
4. 기하 대조      시선 방향, 가시선, 상대 각도 확인
        ↓
5. 시간 대조      그림자 방향과 태양 위치 계산
        ↓
6. 반증 시도      "이 후보가 틀렸다면 어떤 증거가 보여야 하는가"
        ↓
7. 등급 부여      A1 ~ F6
```

그림 20-1. 위치 검증 절차

## 20.3 태양 위치 계산

그림자 방향은 촬영 시각과 위치를 강하게 제약한다.

```python
# fusion/solar.py
from __future__ import annotations
import math
from datetime import datetime, timezone
from typing import Tuple


def julian_day(dt: datetime) -> float:
    dt = dt.astimezone(timezone.utc)
    y, m = dt.year, dt.month
    d = (dt.day + dt.hour / 24.0 + dt.minute / 1440.0 + dt.second / 86400.0)
    if m <= 2:
        y -= 1
        m += 12
    a = y // 100
    b = 2 - a + a // 4
    return (math.floor(365.25 * (y + 4716)) +
            math.floor(30.6001 * (m + 1)) + d + b - 1524.5)


def solar_position(dt: datetime, lat: float, lon: float) -> Tuple[float, float]:
    """(방위각°, 고도각°). 방위각은 북=0, 시계방향."""
    jd = julian_day(dt)
    n = jd - 2451545.0

    L = (280.460 + 0.9856474 * n) % 360.0            # 평균황경
    g = math.radians((357.528 + 0.9856003 * n) % 360.0)  # 평균근점이각
    lam = math.radians(L + 1.915 * math.sin(g) + 0.020 * math.sin(2 * g))

    eps = math.radians(23.439 - 0.0000004 * n)
    ra = math.atan2(math.cos(eps) * math.sin(lam), math.cos(lam))
    dec = math.asin(math.sin(eps) * math.sin(lam))

    gmst = (18.697374558 + 24.06570982441908 * n) % 24.0
    lst = math.radians((gmst * 15.0 + lon) % 360.0)
    ha = lst - ra

    latr = math.radians(lat)
    alt = math.asin(math.sin(latr) * math.sin(dec) +
                    math.cos(latr) * math.cos(dec) * math.cos(ha))
    az = math.atan2(-math.sin(ha),
                    math.tan(dec) * math.cos(latr) - math.sin(latr) * math.cos(ha))

    return (math.degrees(az) % 360.0, math.degrees(alt))


def shadow_bearing(dt: datetime, lat: float, lon: float) -> float:
    """그림자가 뻗는 방향(태양 방위각의 반대)."""
    az, _ = solar_position(dt, lat, lon)
    return (az + 180.0) % 360.0


def shadow_length_ratio(dt: datetime, lat: float, lon: float) -> float:
    """물체 높이 대비 그림자 길이 비율."""
    _, alt = solar_position(dt, lat, lon)
    if alt <= 0.5:
        return float("inf")
    return 1.0 / math.tan(math.radians(alt))
```

> **TIP**
> 이 계산은 근사식이며 오차가 수 분의 1도 수준이다.
> 위치 검증 용도로는 충분하지만, 정밀 천문 계산에는 사용하지 않는다.
> 정밀도가 필요하면 `pyephem`이나 `skyfield`를 벤더링한다.

## 20.4 가능 촬영 시각 범위 역산

그림자 방향으로부터 촬영 시각 후보를 좁힌다.

```python
from datetime import timedelta


def candidate_times(date: datetime, lat: float, lon: float,
                    observed_shadow_bearing: float,
                    tolerance_deg: float = 10.0,
                    step_minutes: int = 5):
    """관측된 그림자 방향과 일치하는 시각 후보."""
    results = []
    t = date.replace(hour=0, minute=0, second=0, microsecond=0)
    end = t + timedelta(days=1)

    while t < end:
        az, alt = solar_position(t, lat, lon)
        if alt > 3.0:
            bearing = (az + 180.0) % 360.0
            diff = abs((bearing - observed_shadow_bearing + 180) % 360 - 180)
            if diff <= tolerance_deg:
                results.append((t, bearing, alt, diff))
        t += timedelta(minutes=step_minutes)
    return results
```

하루에 두 번(오전/오후) 후보가 나오는 것이 일반적이다.
그림자 **길이** 비율을 함께 대조하면 그중 하나를 배제할 수 있다.

## 20.5 시선 방향과 지형 윤곽 대조

DSM에서 특정 지점·방향의 스카이라인을 추출해 사진과 비교한다.

```python
# fusion/skyline.py
from __future__ import annotations
import math
import numpy as np
from typing import List, Tuple


def extract_skyline(dsm: np.ndarray, cellsize: float,
                    y0: int, x0: int, observer_h: float,
                    bearing_from: float, bearing_to: float,
                    step_deg: float = 0.5,
                    max_dist_px: int = 500) -> List[Tuple[float, float]]:
    """방위각별 최대 앙각(스카이라인 프로파일)."""
    h, w = dsm.shape
    z0 = dsm[y0, x0] + observer_h
    out = []

    b = bearing_from
    while b <= bearing_to:
        theta = math.radians(90.0 - b)      # 북=0 → 수학각 변환
        dx, dy = math.cos(theta), -math.sin(theta)

        max_angle = -90.0
        for step in range(1, max_dist_px + 1):
            yy = int(round(y0 + dy * step))
            xx = int(round(x0 + dx * step))
            if not (0 <= yy < h and 0 <= xx < w):
                break
            z = dsm[yy, xx]
            if not np.isfinite(z):
                continue
            d = step * cellsize
            angle = math.degrees(math.atan2(z - z0, d))
            if angle > max_angle:
                max_angle = angle
        out.append((b, max_angle))
        b += step_deg
    return out


def skyline_similarity(profile_a: List[Tuple[float, float]],
                       profile_b: List[Tuple[float, float]]) -> float:
    """두 스카이라인의 RMS 차이(도). 낮을수록 유사."""
    if len(profile_a) != len(profile_b):
        raise ValueError("프로파일 길이가 다릅니다.")
    diffs = [(a[1] - b[1]) ** 2 for a, b in zip(profile_a, profile_b)]
    return math.sqrt(sum(diffs) / len(diffs))
```

## 20.6 검증 결과 기록

```python
# fusion/verification.py
from __future__ import annotations
from dataclasses import dataclass, field, asdict
from typing import List, Optional
import json


@dataclass
class LocationClaim:
    claim_id: str
    asserted_lon: float
    asserted_lat: float
    asserted_time: Optional[str] = None
    asserted_bearing: Optional[float] = None
    source_url: str = ""


@dataclass
class VerificationResult:
    claim: LocationClaim
    supporting: List[str] = field(default_factory=list)
    contradicting: List[str] = field(default_factory=list)
    unresolved: List[str] = field(default_factory=list)
    source_grade: str = "C"
    cert_grade: str = "4"
    analyst: str = ""
    checked_at: str = ""

    @property
    def verdict(self) -> str:
        if self.contradicting:
            return "반증됨"
        if len(self.supporting) >= 3 and not self.unresolved:
            return "일치"
        if self.supporting:
            return "부분 일치"
        return "판단 불가"

    def to_json(self) -> str:
        d = asdict(self)
        d["verdict"] = self.verdict
        return json.dumps(d, ensure_ascii=False, indent=2)
```

> **ENGINEERING PRACTICE**
> `contradicting` 항목이 하나라도 있으면 다른 근거가 아무리 많아도 "반증됨"이다.
> 이것이 확증편향을 코드 수준에서 막는 장치다.
> 단, 반증 근거 자체의 신뢰도도 기록해야 하므로
> 실제 구현에서는 각 항목에 근거 등급을 함께 저장한다.

### 버전 호환 노트 (Chapter 20)

- 순수 Python/numpy 구현이므로 QGIS 버전 의존성이 없다.
- `datetime.timezone`은 Python 3.2+ 이므로 문제없다.

---

# Chapter 21. 다중 출처 융합과 신뢰도 평가

## 21.1 융합의 세 가지 층위

| 층위 | 내용 | 예 |
|---|---|---|
| 위치 융합 | 서로 다른 출처의 같은 대상을 하나로 병합 | 영상 탐지 + 보도 지명 |
| 시간 융합 | 시간 구간의 교집합 산출 | 관측 창 좁히기 |
| 판단 융합 | 등급 결합 | C3 + C3 → B2 (독립 출처 2건) |

표 21-1. 융합의 층위

## 21.2 위치 융합 — 공간 클러스터링

```python
# fusion/cluster.py
from __future__ import annotations
import math
from typing import Dict, List, Tuple

from .collector import Observation


def haversine_m(lon1: float, lat1: float, lon2: float, lat2: float) -> float:
    R = 6371000.0
    p1, p2 = math.radians(lat1), math.radians(lat2)
    dp = p2 - p1
    dl = math.radians(lon2 - lon1)
    a = (math.sin(dp / 2) ** 2 +
         math.cos(p1) * math.cos(p2) * math.sin(dl / 2) ** 2)
    return 2 * R * math.asin(math.sqrt(a))


def cluster_observations(obs: List[Observation],
                         radius_m: float = 250.0) -> List[List[Observation]]:
    """단순 거리 기반 응집 클러스터링."""
    positioned = [o for o in obs if o.has_position]
    unassigned = list(positioned)
    clusters: List[List[Observation]] = []

    while unassigned:
        seed = unassigned.pop(0)
        group = [seed]
        changed = True
        while changed:
            changed = False
            for cand in list(unassigned):
                for member in group:
                    if haversine_m(cand.lon, cand.lat,
                                   member.lon, member.lat) <= radius_m:
                        group.append(cand)
                        unassigned.remove(cand)
                        changed = True
                        break
        clusters.append(group)
    return clusters


def cluster_centroid(group: List[Observation]) -> Tuple[float, float, float]:
    """위치 오차의 역수를 가중치로 사용한 가중 평균과 산포."""
    weights = []
    for o in group:
        err = o.position_error_m or 500.0
        weights.append(1.0 / max(err, 1.0))
    total = sum(weights) or 1.0

    lon = sum(o.lon * w for o, w in zip(group, weights)) / total
    lat = sum(o.lat * w for o, w in zip(group, weights)) / total

    spread = max((haversine_m(lon, lat, o.lon, o.lat) for o in group),
                 default=0.0)
    return lon, lat, spread
```

## 21.3 등급 결합 규칙

```python
# fusion/confidence.py
from __future__ import annotations
from typing import List, Tuple

SOURCE_ORDER = "ABCDEF"
CERT_ORDER = "123456"


def source_value(grade: str) -> int:
    g = (grade or "F")[0].upper()
    return SOURCE_ORDER.index(g) if g in SOURCE_ORDER else 5


def cert_value(grade: str) -> int:
    g = (grade or "6")[0]
    return CERT_ORDER.index(g) if g in CERT_ORDER else 5


def combine(gradings: List[Tuple[str, str]],
            independent: bool = True) -> Tuple[str, str, List[str]]:
    """복수 평가를 하나로 결합. (출처등급, 확실성등급, 근거설명)."""
    if not gradings:
        return "F", "6", ["평가 자료 없음"]

    notes = []
    best_src = min(source_value(s) for s, _ in gradings)
    best_cert = min(cert_value(c) for _, c in gradings)

    if independent and len(gradings) >= 2:
        distinct = len({s for s, _ in gradings})
        if distinct >= 2 and best_cert > 0:
            best_cert -= 1
            notes.append("독립 출처 {0}건 교차 확인으로 확실성 1단계 상향"
                         .format(distinct))
    else:
        notes.append("출처 독립성이 확인되지 않아 상향 미적용")

    return SOURCE_ORDER[best_src], CERT_ORDER[best_cert], notes
```

> **WARNING**
> **출처 독립성 판단이 가장 어렵다.**
> 여러 언론사가 같은 통신사 기사를 받아쓴 것은 독립 출처가 아니다.
> 자동 판정은 불가능하며, 분석관이 명시적으로 표시해야 한다.
> QGeoINT는 기본값을 `independent=False`로 두고, 사용자가 켜야만 상향한다.

## 21.4 모순 탐지

```python
# fusion/conflict.py
from __future__ import annotations
from typing import List, Dict
from .collector import Observation
from .cluster import haversine_m


def detect_conflicts(group: List[Observation],
                     max_spread_m: float = 1000.0) -> List[str]:
    issues = []

    # 1. 공간 모순
    for i, a in enumerate(group):
        for b in group[i + 1:]:
            if a.has_position and b.has_position:
                d = haversine_m(a.lon, a.lat, b.lon, b.lat)
                tol = (a.position_error_m or 200) + (b.position_error_m or 200)
                if d > max(tol, max_spread_m):
                    issues.append(
                        "{0}과(와) {1}의 위치가 {2:.0f} m 떨어져 있습니다."
                        .format(a.obs_id, b.obs_id, d))

    # 2. 시간 모순
    times = [o.observed_at for o in group if o.observed_at]
    if len(set(times)) > 1:
        issues.append("관측 시각이 서로 다릅니다: {0}".format(sorted(set(times))))

    # 3. 등급 편차
    grades = {o.source_grade for o in group}
    if "A" in grades and ("E" in grades or "F" in grades):
        issues.append("신뢰도 편차가 큰 출처가 혼재합니다. 개별 검토가 필요합니다.")

    return issues
```

## 21.5 융합 결과 레이어 생성

```python
# fusion/to_layer.py
from qgis.core import (
    QgsVectorLayer, QgsFeature, QgsGeometry, QgsPointXY, QgsField,
)
from qgis.PyQt.QtCore import QVariant

from ..compat import memory_uri
from ..core.provenance import Provenance, attach_provenance
from .cluster import cluster_observations, cluster_centroid
from .confidence import combine
from .conflict import detect_conflicts


def build_fusion_layer(observations, radius_m: float = 250.0,
                       independent: bool = False) -> QgsVectorLayer:
    layer = QgsVectorLayer(memory_uri("Point", "EPSG:4326"),
                           "융합 결과", "memory")
    dp = layer.dataProvider()
    dp.addAttributes([
        QgsField("cluster_id", QVariant.Int),
        QgsField("n_sources", QVariant.Int),
        QgsField("spread_m", QVariant.Double),
        QgsField("source_grade", QVariant.String),
        QgsField("cert_grade", QVariant.String),
        QgsField("conflicts", QVariant.String),
        QgsField("notes", QVariant.String),
    ])
    layer.updateFields()

    clusters = cluster_observations(observations, radius_m)
    feats = []
    for cid, group in enumerate(clusters, start=1):
        lon, lat, spread = cluster_centroid(group)
        src, cert, notes = combine(
            [(o.source_grade, o.cert_grade) for o in group],
            independent=independent)
        conflicts = detect_conflicts(group)

        f = QgsFeature(layer.fields())
        f.setGeometry(QgsGeometry.fromPointXY(QgsPointXY(lon, lat)))
        f.setAttributes([cid, len(group), spread, src, cert,
                         " / ".join(conflicts), " / ".join(notes)])
        feats.append(f)

    dp.addFeatures(feats)
    layer.updateExtents()

    attach_provenance(layer, Provenance(
        operation="multi_source_fusion",
        created_at="", created_by="", host="",
        params={"radius_m": radius_m, "independent": independent},
        notes=["자동 융합 결과. 분석관 검토 필요."],
    ))
    return layer
```

## 21.6 신뢰도 심볼로지

등급을 색과 크기로 즉시 읽을 수 있게 한다.

```python
# fusion/style.py
from qgis.core import (
    QgsCategorizedSymbolRenderer, QgsRendererCategory,
    QgsMarkerSymbol,
)

GRADE_COLORS = {
    "A": "#1a9641", "B": "#a6d96a", "C": "#ffffbf",
    "D": "#fdae61", "E": "#d7191c", "F": "#808080",
}


def apply_confidence_style(layer, field: str = "source_grade") -> None:
    categories = []
    for grade, color in GRADE_COLORS.items():
        symbol = QgsMarkerSymbol.createSimple({
            "name": "circle",
            "color": color,
            "outline_color": "black",
            "outline_width": "0.3",
            "size": "3.5",
        })
        categories.append(QgsRendererCategory(grade, symbol,
                                              "{0} 등급".format(grade)))
    layer.setRenderer(QgsCategorizedSymbolRenderer(field, categories))
    layer.triggerRepaint()
```

### 버전 호환 노트 (Chapter 21)

- `QgsMarkerSymbol.createSimple()`은 3.6~4.x 동일하다.
- QGIS 3.26부터 `QgsSymbol` 관련 일부 enum이 `Qgis` 네임스페이스로 이동했으나
  문자열 딕셔너리 기반 생성은 영향을 받지 않는다.

---
---

# Part VII. 산출물과 시각화

---

# Chapter 22. GeoINT 심볼로지

## 22.1 심볼로지가 판단을 바꾼다

같은 데이터라도 표현 방식에 따라 읽는 사람의 결론이 달라진다.
GeoINT 심볼로지의 목표는 "예쁜 지도"가 아니라 **오독 방지**다.

원칙:

1. **불확실성은 반드시 시각적으로 구분된다.** 확정과 추정을 같은 색으로 그리지 않는다.
2. **색상만으로 정보를 전달하지 않는다.** 색각 이상 사용자와 흑백 인쇄를 고려한다.
3. **시간 정보를 함께 표현한다.** 언제 관측된 것인지 지도에서 읽혀야 한다.
4. **범례에 등급 정의를 포함한다.** 지도가 단독으로 유통되기 때문이다.

## 22.2 불확실성 표현 패턴

| 정보 | 시각 변수 | 구현 |
|---|---|---|
| 확실성 등급 | 외곽선 스타일 | 실선(확인) / 파선(추정) / 점선(미확인) |
| 위치 오차 | 원 반경 | 오차 반경 원을 함께 표시 |
| 관측 경과 시간 | 채도 | 오래될수록 흐리게 |
| 출처 등급 | 색상 | 녹색~적색 단계 |
| 검토 상태 | 심볼 형태 | 원(미검토) / 사각(확인) / 삼각(보류) |

표 22-1. 불확실성 시각화 패턴

## 22.3 규칙 기반 렌더러

```python
# production/symbology.py
from qgis.core import (
    QgsRuleBasedRenderer, QgsFillSymbol, QgsLineSymbol,
    QgsSymbol, QgsWkbTypes,
)
from ..compat import GEOM_POLYGON


def _fill(color: str, outline_style: str, width: str = "0.5") -> QgsFillSymbol:
    return QgsFillSymbol.createSimple({
        "color": color,
        "outline_color": "black",
        "outline_style": outline_style,   # solid / dash / dot
        "outline_width": width,
        "style": "solid",
    })


RULES = [
    # (라벨, 표현식, 색상, 외곽선 스타일)
    ("확인된 변화",   "\"review\" = '확인'",   "255,0,0,120",   "solid"),
    ("검토 보류",     "\"review\" = '보류'",   "255,170,0,100", "dash"),
    ("미검토 후보",   "\"review\" = '미검토'", "160,160,160,80", "dot"),
    ("오탐 처리",     "\"review\" = '오탐'",   "200,200,255,40", "dot"),
]


def apply_change_rules(layer) -> None:
    root = QgsRuleBasedRenderer.Rule(None)

    for label, expr, color, style in RULES:
        symbol = _fill(color, style)
        rule = QgsRuleBasedRenderer.Rule(symbol, filterExp=expr, label=label)
        root.appendChild(rule)

    # 소면적 후보는 축척 제한
    small = QgsRuleBasedRenderer.Rule(
        _fill("160,160,160,60", "dot"),
        filterExp="\"area_m2\" < 100",
        label="소면적(100 m² 미만)")
    small.setMaximumScale(1)
    small.setMinimumScale(10000)
    root.appendChild(small)

    layer.setRenderer(QgsRuleBasedRenderer(root))
    layer.triggerRepaint()
```

## 22.4 시간 경과에 따른 감쇠

```python
def age_based_opacity_expression(date_field: str = "observed_at",
                                 half_life_days: float = 7.0) -> str:
    """관측 후 경과일에 따라 투명도를 조절하는 표현식."""
    return (
        "set_color_part("
        "  @symbol_color, 'alpha',"
        "  255 * exp(-0.693 * "
        "    (day(age(now(), to_datetime(\"{0}\"))) / {1}))"
        ")"
    ).format(date_field, half_life_days)
```

데이터 정의 속성으로 적용한다.

```python
from qgis.core import QgsProperty, QgsSymbolLayer

def apply_age_fade(layer, date_field: str = "observed_at") -> None:
    symbol = layer.renderer().symbol()
    sl = symbol.symbolLayer(0)
    sl.setDataDefinedProperty(
        QgsSymbolLayer.PropertyFillColor,
        QgsProperty.fromExpression(age_based_opacity_expression(date_field)))
    layer.triggerRepaint()
```

> **호환성**
> `QgsSymbolLayer.PropertyFillColor`는 QGIS 4에서
> `QgsSymbolLayer.Property.PropertyFillColor`로 이동했다.
> `compat.py`의 `_enum()` 헬퍼로 처리한다.

## 22.5 표준 심볼 체계의 활용

시설·사건 유형을 표준화된 기호로 표기하면 조직 간 소통 비용이 줄어든다.
공개 표준 기호집(예: 재난 관리 분야의 공통 픽토그램, 국가 지리원 지형도 기호)을
SVG 심볼로 등록해 사용한다.

```python
# production/symbol_registry.py
import os
from qgis.core import QgsApplication, QgsSvgMarkerSymbolLayer, QgsMarkerSymbol


def register_svg_path(plugin_dir: str) -> None:
    """플러그인의 SVG 경로를 QGIS SVG 검색 경로에 추가."""
    svg_dir = os.path.join(plugin_dir, "resources", "svg")
    paths = QgsApplication.svgPaths()
    if svg_dir not in paths:
        paths.append(svg_dir)
        QgsApplication.setDefaultSvgPaths(paths)


def svg_symbol(svg_name: str, size: float = 6.0,
               color: str = "#000000") -> QgsMarkerSymbol:
    layer = QgsSvgMarkerSymbolLayer(svg_name)
    layer.setSize(size)
    layer.setFillColor(_qcolor(color))
    symbol = QgsMarkerSymbol()
    symbol.changeSymbolLayer(0, layer)
    return symbol


def _qcolor(hex_str: str):
    from qgis.PyQt.QtGui import QColor
    return QColor(hex_str)
```

> **WARNING**
> 특정 기관의 기호집은 저작권 또는 사용 제한이 있을 수 있다.
> 플러그인에 SVG를 포함해 배포하기 전에 반드시 사용 조건을 확인한다.
> 공개 라이선스가 확인된 기호만 동봉하고, 나머지는 사용자가 자체 경로에 두도록 안내한다.

## 22.6 색각 안전 팔레트

```python
# production/palette.py

# ColorBrewer 계열 — 색각 이상에서도 구분 가능
SEQUENTIAL_BLUE = ["#eff3ff", "#bdd7e7", "#6baed6", "#3182bd", "#08519c"]
DIVERGING_BROWN_TEAL = ["#a6611a", "#dfc27d", "#f5f5f5", "#80cdc1", "#018571"]
QUALITATIVE_SAFE = ["#1b9e77", "#d95f02", "#7570b3", "#e7298a",
                    "#66a61e", "#e6ab02"]


def check_contrast(hex_a: str, hex_b: str) -> float:
    """WCAG 명도 대비 비율. 3.0 이상 권장."""
    def lum(h):
        h = h.lstrip("#")
        rgb = [int(h[i:i + 2], 16) / 255.0 for i in (0, 2, 4)]
        rgb = [c / 12.92 if c <= 0.03928 else ((c + 0.055) / 1.055) ** 2.4
               for c in rgb]
        return 0.2126 * rgb[0] + 0.7152 * rgb[1] + 0.0722 * rgb[2]

    la, lb = lum(hex_a), lum(hex_b)
    hi, lo = max(la, lb), min(la, lb)
    return (hi + 0.05) / (lo + 0.05)
```

### 버전 호환 노트 (Chapter 22)

- `QgsRuleBasedRenderer.Rule(symbol, filterExp=..., label=...)` 키워드 인자는 3.6부터 지원된다.
- `QgsApplication.setDefaultSvgPaths()`는 3.6~4.x 공통이다.
- `set_color_part()` 표현식 함수는 3.0부터 존재한다.

---

# Chapter 23. 지도 레이아웃 자동화

## 23.1 산출 지도의 필수 요소

GeoINT 산출 지도는 단독으로 유통된다. 지도만 보고도 다음을 알 수 있어야 한다.

- [ ] 제목과 목적
- [ ] 작성 기관·작성자·작성일시
- [ ] 좌표계 및 격자 표기
- [ ] 축척 막대 (숫자 축척은 인쇄 크기 변경 시 무효)
- [ ] 방위 표시
- [ ] 원본 자료 출처 및 관측 일시
- [ ] 범례 (등급 정의 포함)
- [ ] 취급 주의 표기 (해당 시)
- [ ] 면책 및 한계 사항

## 23.2 레이아웃 생성

```python
# production/layout.py
from __future__ import annotations
from datetime import datetime
from typing import Optional

from qgis.core import (
    QgsProject, QgsPrintLayout, QgsLayoutItemMap, QgsLayoutItemLabel,
    QgsLayoutItemLegend, QgsLayoutItemScaleBar, QgsLayoutItemPicture,
    QgsLayoutPoint, QgsLayoutSize, QgsUnitTypes, QgsLayoutExporter,
    QgsLayoutItemPage, QgsRectangle,
)
from qgis.PyQt.QtGui import QFont


MM = QgsUnitTypes.LayoutMillimeters


class BriefingLayout:
    def __init__(self, name: str, project: Optional[QgsProject] = None):
        self.project = project or QgsProject.instance()
        self.layout = QgsPrintLayout(self.project)
        self.layout.initializeDefaults()
        self.layout.setName(name)
        self._setup_page()

    def _setup_page(self) -> None:
        page = self.layout.pageCollection().page(0)
        page.setPageSize("A3", QgsLayoutItemPage.Landscape)

    # --------------------------------------------------------------
    def add_map(self, extent: QgsRectangle,
                x=10, y=25, w=270, h=250) -> QgsLayoutItemMap:
        m = QgsLayoutItemMap(self.layout)
        m.setRect(0, 0, w, h)
        m.setExtent(extent)
        m.attemptMove(QgsLayoutPoint(x, y, MM))
        m.attemptResize(QgsLayoutSize(w, h, MM))
        m.setFrameEnabled(True)
        self.layout.addLayoutItem(m)
        self.map_item = m
        return m

    def add_title(self, text: str, subtitle: str = "") -> None:
        label = QgsLayoutItemLabel(self.layout)
        label.setText(text)
        f = QFont("Noto Sans KR", 20)
        f.setBold(True)
        label.setFont(f)
        label.adjustSizeToText()
        label.attemptMove(QgsLayoutPoint(10, 8, MM))
        self.layout.addLayoutItem(label)

        if subtitle:
            sub = QgsLayoutItemLabel(self.layout)
            sub.setText(subtitle)
            sub.setFont(QFont("Noto Sans KR", 11))
            sub.adjustSizeToText()
            sub.attemptMove(QgsLayoutPoint(10, 18, MM))
            self.layout.addLayoutItem(sub)

    def add_legend(self, x=290, y=25, w=100, h=120) -> None:
        legend = QgsLayoutItemLegend(self.layout)
        legend.setTitle("범례")
        legend.setLinkedMap(self.map_item)
        legend.setLegendFilterByMapEnabled(True)
        legend.attemptMove(QgsLayoutPoint(x, y, MM))
        legend.attemptResize(QgsLayoutSize(w, h, MM))
        legend.setFrameEnabled(True)
        self.layout.addLayoutItem(legend)

    def add_scalebar(self, x=10, y=280) -> None:
        bar = QgsLayoutItemScaleBar(self.layout)
        bar.setStyle("Single Box")
        bar.setLinkedMap(self.map_item)
        bar.applyDefaultSize()
        bar.attemptMove(QgsLayoutPoint(x, y, MM))
        self.layout.addLayoutItem(bar)

    def add_metadata_block(self, sources, analyst: str,
                           crs_authid: str, x=290, y=150) -> None:
        lines = [
            "작성일시: {0}".format(datetime.now().strftime("%Y-%m-%d %H:%M")),
            "작성자: {0}".format(analyst),
            "좌표계: {0}".format(crs_authid),
            "",
            "자료 출처:",
        ]
        lines.extend("  - {0}".format(s) for s in sources)
        lines.extend([
            "",
            "한계 사항:",
            "  - 자동 탐지 결과 포함. 등급 표기 참조.",
            "  - 관측 시점 이후 변화는 반영되지 않음.",
        ])

        label = QgsLayoutItemLabel(self.layout)
        label.setText("\n".join(lines))
        label.setFont(QFont("Noto Sans KR", 8))
        label.attemptMove(QgsLayoutPoint(x, y, MM))
        label.attemptResize(QgsLayoutSize(100, 110, MM))
        label.setFrameEnabled(True)
        self.layout.addLayoutItem(label)

    def add_handling_notice(self, text: str) -> None:
        """취급 주의 표기. 상하단 양쪽에 배치."""
        for y in (2, 292):
            label = QgsLayoutItemLabel(self.layout)
            label.setText(text)
            f = QFont("Noto Sans KR", 12)
            f.setBold(True)
            label.setFont(f)
            label.adjustSizeToText()
            label.attemptMove(QgsLayoutPoint(160, y, MM))
            self.layout.addLayoutItem(label)

    # --------------------------------------------------------------
    def export_pdf(self, path: str) -> bool:
        exporter = QgsLayoutExporter(self.layout)
        settings = QgsLayoutExporter.PdfExportSettings()
        settings.dpi = 300
        settings.rasterizeWholeImage = False
        result = exporter.exportToPdf(path, settings)
        return result == QgsLayoutExporter.Success

    def export_image(self, path: str, dpi: int = 200) -> bool:
        exporter = QgsLayoutExporter(self.layout)
        settings = QgsLayoutExporter.ImageExportSettings()
        settings.dpi = dpi
        return exporter.exportToImage(path, settings) == QgsLayoutExporter.Success
```

## 23.3 MGRS 격자 오버레이

```python
# production/grid_overlay.py
from qgis.core import (
    QgsLayoutItemMapGrid, QgsCoordinateReferenceSystem,
)
from qgis.PyQt.QtGui import QColor


def add_utm_grid(map_item, interval_m: float = 1000.0,
                 utm_authid: str = "EPSG:32652") -> None:
    grid = map_item.grid()
    grid.setEnabled(True)
    grid.setCrs(QgsCoordinateReferenceSystem(utm_authid))
    grid.setIntervalX(interval_m)
    grid.setIntervalY(interval_m)
    grid.setStyle(QgsLayoutItemMapGrid.Solid)

    line_symbol = grid.lineSymbol()
    line_symbol.setColor(QColor(0, 0, 0, 80))
    line_symbol.setWidth(0.15)

    grid.setAnnotationEnabled(True)
    grid.setAnnotationPrecision(0)
    grid.setAnnotationFormat(QgsLayoutItemMapGrid.CustomFormat)
    grid.setAnnotationExpression(
        "right(to_string(round(@grid_number/1000)), 2)")   # 1000 m 격자 하위 2자리
    map_item.updateBoundingRect()
```

> **TIP**
> 격자 주기는 축척에 맞춰 자동 결정하는 편이 낫다.
> 1:25,000 지도에 100 m 격자를 그리면 지도가 격자로 뒤덮인다.

```python
def auto_grid_interval(scale: float) -> float:
    if scale <= 5000:
        return 100.0
    if scale <= 25000:
        return 1000.0
    if scale <= 100000:
        return 5000.0
    return 10000.0
```

## 23.4 배치 지도 생성 (아틀라스)

여러 AOI에 대해 동일 양식의 지도를 자동 생산한다.

```python
# production/atlas.py
from qgis.core import QgsLayoutExporter


def export_atlas(layout, coverage_layer, out_dir: str,
                 filename_expr: str = "'map_' || \"name\"") -> int:
    atlas = layout.atlas()
    atlas.setEnabled(True)
    atlas.setCoverageLayer(coverage_layer)
    atlas.setFilenameExpression(filename_expr)
    atlas.setSortFeatures(True)
    atlas.setSortExpression("\"name\"")

    exporter = QgsLayoutExporter(layout)
    settings = QgsLayoutExporter.PdfExportSettings()
    settings.dpi = 300

    result, error = QgsLayoutExporter.exportToPdfs(
        atlas, out_dir, settings)
    if result != QgsLayoutExporter.Success:
        raise RuntimeError("아틀라스 내보내기 실패: {0}".format(error))
    return atlas.count()
```

### 버전 호환 노트 (Chapter 23)

- `QgsLayoutExporter.exportToPdfs(atlas, ...)`는 QGIS **3.4 이상**에서 제공된다.
- `QgsLayoutItemMapGrid.setAnnotationExpression()`은 3.10 이상.
  3.6에서는 `setAnnotationFormat(DecimalWithSuffix)` 등 사전 정의 형식만 사용 가능하다.
- QGIS 4에서 `QgsLayoutItemPage.Landscape`는 `QgsLayoutItemPage.Orientation.Landscape`로 이동했다.

---

# Chapter 24. 브리핑 산출물 자동화

## 24.1 산출물 3종

| 산출물 | 형식 | 용도 |
|---|---|---|
| 상황도 | PDF / PNG | 지도 중심. 현장 배포 |
| 브리핑 슬라이드 | PPTX | 보고 발표 |
| 분석 보고서 | DOCX / PDF | 근거 기록·보존 |

표 24-1. 산출물 유형

세 가지 모두 **같은 데이터 소스에서 자동 생성**되어야 한다.
수동으로 옮겨 적는 순간 재현성이 깨진다.

## 24.2 산출물 데이터 모델

```python
# production/report_model.py
from __future__ import annotations
from dataclasses import dataclass, field
from datetime import datetime
from typing import Any, Dict, List, Optional


@dataclass
class Finding:
    """하나의 분석 결론."""
    finding_id: str
    summary: str                     # 한 문장 결론
    location_mgrs: str
    location_desc: str
    source_grade: str
    cert_grade: str
    evidence: List[str] = field(default_factory=list)
    counter_evidence: List[str] = field(default_factory=list)
    observed_span: str = ""
    image_paths: List[str] = field(default_factory=list)

    @property
    def grade(self) -> str:
        return "{0}{1}".format(self.source_grade, self.cert_grade)


@dataclass
class BriefingPackage:
    title: str
    aoi_name: str
    analyst: str
    created_at: str = field(
        default_factory=lambda: datetime.now().isoformat(timespec="minutes"))
    key_judgement: str = ""
    findings: List[Finding] = field(default_factory=list)
    sources: List[str] = field(default_factory=list)
    limitations: List[str] = field(default_factory=list)
    map_image: Optional[str] = None

    def sorted_findings(self) -> List[Finding]:
        order = "ABCDEF"
        return sorted(self.findings,
                      key=lambda f: (order.index(f.source_grade[0]),
                                     f.cert_grade))
```

## 24.3 보고서 생성 (DOCX)

```python
# production/docx_report.py
from __future__ import annotations
import os
from typing import Optional

from ..core.optional import GATES
from .report_model import BriefingPackage


def build_docx(pkg: BriefingPackage, out_path: str) -> str:
    GATES.require("docx", "DOCX 보고서 생성")
    from docx import Document
    from docx.shared import Pt, Cm, RGBColor
    from docx.enum.text import WD_ALIGN_PARAGRAPH

    doc = Document()

    # 표지
    h = doc.add_heading(pkg.title, level=0)
    h.alignment = WD_ALIGN_PARAGRAPH.CENTER

    meta = doc.add_paragraph()
    meta.alignment = WD_ALIGN_PARAGRAPH.CENTER
    meta.add_run("대상지역: {0}\n작성자: {1}\n작성일시: {2}".format(
        pkg.aoi_name, pkg.analyst, pkg.created_at))

    doc.add_page_break()

    # 핵심 판단
    doc.add_heading("1. 핵심 판단", level=1)
    p = doc.add_paragraph(pkg.key_judgement or "(작성 필요)")
    p.runs[0].bold = True

    # 상황도
    if pkg.map_image and os.path.exists(pkg.map_image):
        doc.add_heading("2. 상황도", level=1)
        doc.add_picture(pkg.map_image, width=Cm(16))

    # 세부 발견사항
    doc.add_heading("3. 세부 발견사항", level=1)
    for i, f in enumerate(pkg.sorted_findings(), start=1):
        doc.add_heading("3.{0} {1}".format(i, f.summary), level=2)

        table = doc.add_table(rows=0, cols=2)
        table.style = "Light Grid Accent 1"
        rows = [
            ("위치 (MGRS)", f.location_mgrs),
            ("위치 설명", f.location_desc),
            ("관측 시간대", f.observed_span),
            ("신뢰도 등급", f.grade),
        ]
        for k, v in rows:
            cells = table.add_row().cells
            cells[0].text = k
            cells[1].text = v or "-"

        if f.evidence:
            doc.add_paragraph("근거:", style="Intense Quote")
            for e in f.evidence:
                doc.add_paragraph(e, style="List Bullet")

        if f.counter_evidence:
            doc.add_paragraph("반대 근거 / 대안 해석:", style="Intense Quote")
            for e in f.counter_evidence:
                doc.add_paragraph(e, style="List Bullet")

        for img in f.image_paths:
            if os.path.exists(img):
                doc.add_picture(img, width=Cm(14))

    # 출처
    doc.add_heading("4. 자료 출처", level=1)
    for s in pkg.sources:
        doc.add_paragraph(s, style="List Number")

    # 한계
    doc.add_heading("5. 한계 및 유의사항", level=1)
    for l in pkg.limitations:
        doc.add_paragraph(l, style="List Bullet")

    doc.save(out_path)
    return out_path
```

> **ENGINEERING PRACTICE**
> `counter_evidence`(반대 근거) 섹션을 템플릿에 **고정으로 포함**한다.
> 비워 두면 "반대 근거를 검토하지 않았다"는 사실 자체가 드러난다.
> 이것이 서식으로 사고를 강제하는 방법이다.

## 24.4 슬라이드 생성 (PPTX)

```python
# production/pptx_briefing.py
from ..core.optional import GATES
from .report_model import BriefingPackage


def build_pptx(pkg: BriefingPackage, out_path: str) -> str:
    GATES.require("pptx", "PPTX 브리핑 생성")
    from pptx import Presentation
    from pptx.util import Inches, Pt

    prs = Presentation()
    prs.slide_width = Inches(13.333)
    prs.slide_height = Inches(7.5)

    # 표지
    slide = prs.slides.add_slide(prs.slide_layouts[0])
    slide.shapes.title.text = pkg.title
    slide.placeholders[1].text = "{0} | {1} | {2}".format(
        pkg.aoi_name, pkg.analyst, pkg.created_at)

    # 핵심 판단
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    slide.shapes.title.text = "핵심 판단"
    tf = slide.placeholders[1].text_frame
    tf.text = pkg.key_judgement
    tf.paragraphs[0].font.size = Pt(24)

    # 상황도
    if pkg.map_image:
        slide = prs.slides.add_slide(prs.slide_layouts[5])
        slide.shapes.title.text = "상황도"
        slide.shapes.add_picture(pkg.map_image, Inches(0.5), Inches(1.3),
                                 height=Inches(5.8))

    # 발견사항
    for f in pkg.sorted_findings():
        slide = prs.slides.add_slide(prs.slide_layouts[5])
        slide.shapes.title.text = "[{0}] {1}".format(f.grade, f.summary)

        box = slide.shapes.add_textbox(Inches(0.5), Inches(1.4),
                                       Inches(6.0), Inches(5.5))
        tf = box.text_frame
        tf.word_wrap = True
        tf.text = "위치: {0}".format(f.location_mgrs)
        for e in f.evidence:
            p = tf.add_paragraph()
            p.text = "· " + e
            p.level = 1
        for e in f.counter_evidence:
            p = tf.add_paragraph()
            p.text = "× " + e
            p.level = 1

        if f.image_paths:
            slide.shapes.add_picture(f.image_paths[0], Inches(6.8),
                                     Inches(1.4), width=Inches(6.0))

    prs.save(out_path)
    return out_path
```

## 24.5 산출물에 출처 정보 동봉

산출물 폴더에는 항상 기계 판독 가능한 출처 파일을 함께 둔다.

```python
# production/package.py
from __future__ import annotations
import json
import os
import zipfile
from dataclasses import asdict
from typing import List

from .report_model import BriefingPackage


def export_package(pkg: BriefingPackage, out_dir: str,
                   extra_files: List[str] = None) -> str:
    os.makedirs(out_dir, exist_ok=True)

    manifest = {
        "title": pkg.title,
        "aoi": pkg.aoi_name,
        "analyst": pkg.analyst,
        "created_at": pkg.created_at,
        "findings": [asdict(f) for f in pkg.findings],
        "sources": pkg.sources,
        "limitations": pkg.limitations,
        "generator": "QGeoINT",
    }
    manifest_path = os.path.join(out_dir, "manifest.json")
    with open(manifest_path, "w", encoding="utf-8") as f:
        json.dump(manifest, f, ensure_ascii=False, indent=2)

    zip_path = os.path.join(out_dir, "briefing_package.zip")
    with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
        z.write(manifest_path, "manifest.json")
        for path in (extra_files or []):
            if os.path.exists(path):
                z.write(path, os.path.basename(path))
    return zip_path
```

### 버전 호환 노트 (Chapter 24)

- `python-docx`, `python-pptx`는 순수 Python이므로 `ext_libs/common/`에 둘 수 있다.
- 두 라이브러리 모두 Python 3.7에서 동작한다.
- 한글 폰트가 없는 환경에서 DOCX/PPTX를 열면 글꼴이 대체된다.
  템플릿에서 폰트를 지정할 때는 대체 가능한 범용 폰트를 함께 명시한다.

---
---

# Part VIII. 운영 품질과 배포

---

# Chapter 25. Processing 통합과 백그라운드 처리

## 25.1 왜 Processing 알고리즘으로 만드는가

GUI 다이얼로그만 제공하면 다음을 잃는다.

- 배치 실행
- 모델 디자이너 연동
- 실행 이력 자동 기록
- `qgis_process` 명령행 실행
- Python 스크립트에서의 재사용

**재현성 원칙 때문에 GeoINT 플러그인의 모든 분석은 Processing 알고리즘이어야 한다.**
GUI는 알고리즘을 편하게 호출하는 껍데기일 뿐이다.

## 25.2 Provider 구현

```python
# processing/provider.py
from qgis.core import QgsProcessingProvider
from qgis.PyQt.QtGui import QIcon
import os

from .alg_change_detection import ChangeDetectionAlgorithm
from .alg_viewshed import ViewshedAlgorithm
from .alg_dem_qa import DemQaAlgorithm
from .alg_fusion import FusionAlgorithm


class QGeoINTProvider(QgsProcessingProvider):

    def loadAlgorithms(self) -> None:
        for alg in (ChangeDetectionAlgorithm(),
                    ViewshedAlgorithm(),
                    DemQaAlgorithm(),
                    FusionAlgorithm()):
            self.addAlgorithm(alg)

    def id(self) -> str:
        return "qgeoint"

    def name(self) -> str:
        return "QGeoINT"

    def longName(self) -> str:
        return "QGeoINT — 지리공간정보 분석"

    def icon(self) -> QIcon:
        return QIcon(os.path.join(os.path.dirname(__file__),
                                  "..", "resources", "icon.svg"))
```

## 25.3 알고리즘 구현

```python
# processing/alg_change_detection.py
from __future__ import annotations

from qgis.core import (
    QgsProcessingAlgorithm, QgsProcessingParameterRasterLayer,
    QgsProcessingParameterNumber, QgsProcessingParameterEnum,
    QgsProcessingParameterRasterDestination,
    QgsProcessingParameterVectorDestination,
    QgsProcessingException, QgsProcessingContext, QgsProcessingFeedback,
)
from qgis.PyQt.QtCore import QCoreApplication


class ChangeDetectionAlgorithm(QgsProcessingAlgorithm):

    BEFORE = "BEFORE"
    AFTER = "AFTER"
    METHOD = "METHOD"
    K = "K"
    MIN_AREA = "MIN_AREA"
    OUTPUT_MASK = "OUTPUT_MASK"
    OUTPUT_POLY = "OUTPUT_POLY"

    METHODS = ["MAD (강건)", "sigma (단순)", "Otsu (이봉분포)"]
    METHOD_KEYS = ["mad", "sigma", "otsu"]

    # ------------------------------------------------------------------
    def initAlgorithm(self, config=None):
        self.addParameter(QgsProcessingParameterRasterLayer(
            self.BEFORE, self.tr("이전 시점 영상")))
        self.addParameter(QgsProcessingParameterRasterLayer(
            self.AFTER, self.tr("이후 시점 영상")))
        self.addParameter(QgsProcessingParameterEnum(
            self.METHOD, self.tr("임계값 산정 방법"),
            options=self.METHODS, defaultValue=0))
        self.addParameter(QgsProcessingParameterNumber(
            self.K, self.tr("임계 계수 (k)"),
            type=QgsProcessingParameterNumber.Double,
            defaultValue=3.0, minValue=0.5, maxValue=10.0))
        self.addParameter(QgsProcessingParameterNumber(
            self.MIN_AREA, self.tr("최소 면적 (픽셀)"),
            type=QgsProcessingParameterNumber.Integer,
            defaultValue=25, minValue=1))
        self.addParameter(QgsProcessingParameterRasterDestination(
            self.OUTPUT_MASK, self.tr("변화 마스크")))
        self.addParameter(QgsProcessingParameterVectorDestination(
            self.OUTPUT_POLY, self.tr("변화 후보 폴리곤")))

    # ------------------------------------------------------------------
    def processAlgorithm(self, parameters, context, feedback):
        import numpy as np

        from ..repositories.raster_repo import read_band, write_array
        from ..analysis.radiometry import linear_normalize
        from ..analysis.threshold import choose_threshold
        from ..analysis.morphology import closing, filter_by_area
        from ..analysis.coregistration import assess_coregistration
        from ..analysis.vectorize import polygonize

        before_layer = self.parameterAsRasterLayer(parameters, self.BEFORE, context)
        after_layer = self.parameterAsRasterLayer(parameters, self.AFTER, context)
        method = self.METHOD_KEYS[
            self.parameterAsEnum(parameters, self.METHOD, context)]
        k = self.parameterAsDouble(parameters, self.K, context)
        min_area = self.parameterAsInt(parameters, self.MIN_AREA, context)
        mask_path = self.parameterAsOutputLayer(parameters, self.OUTPUT_MASK, context)
        poly_path = self.parameterAsOutputLayer(parameters, self.OUTPUT_POLY, context)

        feedback.pushInfo("영상 읽는 중...")
        before, meta = read_band(before_layer)
        after, meta_a = read_band(after_layer)

        if before.shape != after.shape:
            raise QgsProcessingException(
                "두 영상의 크기가 다릅니다: {0} vs {1}. "
                "사전에 동일 격자로 리샘플링하세요."
                .format(before.shape, after.shape))

        feedback.setProgress(10)
        feedback.pushInfo("정합 상태 진단 중...")
        diag = assess_coregistration(before, after)
        if diag:
            feedback.pushInfo(
                "  중앙 이동량: dy={0:.2f} px, dx={1:.2f} px (산포 {2:.2f}/{3:.2f})"
                .format(diag["median_dy"], diag["median_dx"],
                        diag["spread_dy"], diag["spread_dx"]))
            if max(abs(diag["median_dy"]), abs(diag["median_dx"])) > 2.0:
                feedback.reportError(
                    "경고: 정합 오차가 2픽셀을 초과합니다. "
                    "결과에 대량의 오탐이 포함될 수 있습니다.", fatalError=False)

        if feedback.isCanceled():
            return {}

        feedback.setProgress(25)
        feedback.pushInfo("방사 정규화 중...")
        after_norm, gain, offset = linear_normalize(after, before)
        feedback.pushInfo("  gain={0:.4f}, offset={1:.4f}".format(gain, offset))

        feedback.setProgress(45)
        diff = np.abs(after_norm - before)
        threshold, label = choose_threshold(diff, method, k)
        feedback.pushInfo("임계값: {0:.4f} ({1})".format(threshold, label))

        mask = diff >= threshold
        mask &= np.isfinite(diff)

        feedback.setProgress(65)
        feedback.pushInfo("잡음 제거 중...")
        mask = closing(mask, iterations=1)
        mask = filter_by_area(mask, min_area)

        changed = int(mask.sum())
        total = int(np.isfinite(diff).sum())
        feedback.pushInfo("변화 픽셀: {0:,} / {1:,} ({2:.3f}%)".format(
            changed, total, 100.0 * changed / max(total, 1)))

        if changed == 0:
            feedback.pushWarning("탐지된 변화가 없습니다. 임계 계수를 낮춰 보세요.")

        feedback.setProgress(80)
        write_array(mask.astype("uint8"), meta, mask_path,
                    dtype=1, nodata=0, cog=False)

        feedback.setProgress(90)
        feedback.pushInfo("벡터화 중...")
        polygonize(mask_path, poly_path, "change", meta.projection)

        feedback.setProgress(100)
        return {self.OUTPUT_MASK: mask_path, self.OUTPUT_POLY: poly_path}

    # ------------------------------------------------------------------
    def name(self):
        return "change_detection"

    def displayName(self):
        return self.tr("변화 탐지 (차분 기반)")

    def group(self):
        return self.tr("영상 분석")

    def groupId(self):
        return "imagery"

    def shortHelpString(self):
        return self.tr(
            "두 시점 영상의 차분으로 변화 후보를 탐지합니다.\n\n"
            "결과는 자동 탐지 후보이며 분석관 검토가 필요합니다. "
            "정합 오차가 큰 영상 쌍에서는 오탐이 급증합니다.")

    def createInstance(self):
        return ChangeDetectionAlgorithm()

    def tr(self, text):
        return QCoreApplication.translate("QGeoINT", text)
```

> **TIP**
> `feedback.pushInfo()`로 남긴 내용은 Processing 실행 이력에 저장된다.
> gain/offset/임계값을 모두 출력해 두면 사후에 "왜 이런 결과가 나왔는가"를 답할 수 있다.
> 이것만으로도 추적성 요구의 상당 부분이 충족된다.

## 25.4 QgsTask 백그라운드 처리

Processing이 아닌 GUI 작업(예: 대화형 가시권 계산)은 `QgsTask`로 처리한다.

```python
# tasks/base.py
from __future__ import annotations
from typing import Any, Callable, Optional

from qgis.core import QgsTask, QgsMessageLog, QgsApplication
from ..compat import LEVEL_INFO, LEVEL_CRITICAL


class QGeoINTTask(QgsTask):
    """공통 예외 처리·로깅을 갖춘 기반 태스크."""

    def __init__(self, description: str):
        super().__init__(description, QgsTask.CanCancel)
        self.exception: Optional[Exception] = None
        self.result: Any = None

    # 하위 클래스가 구현
    def execute(self) -> Any:
        raise NotImplementedError

    def run(self) -> bool:
        try:
            self.result = self.execute()
            return True
        except InterruptedError:
            return False
        except Exception as exc:      # 태스크 내 예외는 QGIS를 죽인다
            self.exception = exc
            return False

    def finished(self, success: bool) -> None:
        if success:
            QgsMessageLog.logMessage(
                "{0} 완료".format(self.description()), "QGeoINT", LEVEL_INFO)
        elif self.exception is not None:
            QgsMessageLog.logMessage(
                "{0} 실패: {1}".format(self.description(), self.exception),
                "QGeoINT", LEVEL_CRITICAL)
        else:
            QgsMessageLog.logMessage(
                "{0} 취소됨".format(self.description()), "QGeoINT", LEVEL_INFO)

    def progress_callback(self) -> Callable[[int], bool]:
        """분석 함수에 넘길 진행률 콜백. False 반환 시 취소."""
        def cb(percent: int) -> bool:
            self.setProgress(float(percent))
            return not self.isCanceled()
        return cb
```

```python
# tasks/viewshed_task.py
import numpy as np
from .base import QGeoINTTask
from ..analysis.viewshed import viewshed


class ViewshedTask(QGeoINTTask):
    def __init__(self, dem: np.ndarray, cellsize: float,
                 y0: int, x0: int, radius_px: int, observer_h: float):
        super().__init__("가시권 계산")
        self.dem = dem
        self.cellsize = cellsize
        self.y0, self.x0 = y0, x0
        self.radius_px = radius_px
        self.observer_h = observer_h

    def execute(self):
        return viewshed(self.dem, self.cellsize, self.y0, self.x0,
                        self.radius_px, self.observer_h,
                        progress=self.progress_callback())
```

## 25.5 태스크 레지스트리

`unload()` 시 실행 중인 태스크를 정리해야 한다.

```python
# tasks/registry.py
from typing import List
from qgis.core import QgsApplication, QgsTask

_ACTIVE: List[QgsTask] = []


def submit(task: QgsTask) -> None:
    _ACTIVE.append(task)

    def _cleanup():
        if task in _ACTIVE:
            _ACTIVE.remove(task)

    task.taskCompleted.connect(_cleanup)
    task.taskTerminated.connect(_cleanup)
    QgsApplication.taskManager().addTask(task)


def cancel_all_tasks() -> int:
    n = 0
    for task in list(_ACTIVE):
        if task.canCancel():
            task.cancel()
            n += 1
    _ACTIVE.clear()
    return n
```

> **WARNING**
> `QgsTask.run()` 안에서 GUI 객체(레이어 렌더링, 위젯, `iface`)를 건드리면
> QGIS가 크래시한다. 태스크는 순수 계산만 하고,
> 레이어 추가·화면 갱신은 반드시 `finished()`에서 수행한다.

### 버전 호환 노트 (Chapter 25)

- `QgsProcessingFeedback.pushWarning()`은 QGIS **3.16 이상**이다.
  3.6~3.14에서는 `pushInfo("경고: ...")`로 폴백한다.
- `reportError(msg, fatalError=...)`의 두 번째 인자는 3.14부터 지원된다.
- `QgsTask.CanCancel` 플래그는 3.0부터 동일하다.

```python
# compat.py 추가분
def push_warning(feedback, text: str) -> None:
    fn = getattr(feedback, "pushWarning", None)
    if fn is not None:
        fn(text)
    else:
        feedback.pushInfo("경고: " + text)
```

---

# Chapter 26. 로깅과 감사 추적

## 26.1 GeoINT에서 로그의 의미

일반 소프트웨어에서 로그는 디버깅 수단이다.
GeoINT에서 로그는 **"누가 언제 어떤 자료로 무엇을 했는가"에 대한 기록**이며,
분석 결론의 방어 근거가 된다.

두 가지를 분리해 관리한다.

| 구분 | 대상 | 저장 위치 | 보존 |
|---|---|---|---|
| 진단 로그 | 개발·오류 추적 | QGIS 메시지 로그 + 파일 | 단기 |
| 감사 로그 | 분석 행위 기록 | SQLite (append-only) | 장기 |

표 26-1. 로그의 두 층위

## 26.2 진단 로그

```python
# core/logging_setup.py
from __future__ import annotations
import logging
import os
from logging.handlers import RotatingFileHandler

from qgis.core import QgsMessageLog
from ..compat import LEVEL_INFO, LEVEL_WARNING, LEVEL_CRITICAL

TAG = "QGeoINT"

_LEVEL_MAP = {
    logging.DEBUG: LEVEL_INFO,
    logging.INFO: LEVEL_INFO,
    logging.WARNING: LEVEL_WARNING,
    logging.ERROR: LEVEL_CRITICAL,
    logging.CRITICAL: LEVEL_CRITICAL,
}


class QgisLogHandler(logging.Handler):
    """Python logging → QGIS 메시지 패널."""

    def emit(self, record):
        try:
            QgsMessageLog.logMessage(
                self.format(record), TAG,
                _LEVEL_MAP.get(record.levelno, LEVEL_INFO))
        except Exception:
            pass


def setup_logging(log_dir: str, level: int = logging.INFO) -> logging.Logger:
    logger = logging.getLogger("qgeoint")
    if logger.handlers:
        return logger

    logger.setLevel(level)
    logger.propagate = False

    fmt = logging.Formatter(
        "%(asctime)s [%(levelname)s] %(name)s:%(lineno)d — %(message)s")

    qgis_handler = QgisLogHandler()
    qgis_handler.setFormatter(logging.Formatter("%(message)s"))
    logger.addHandler(qgis_handler)

    os.makedirs(log_dir, exist_ok=True)
    file_handler = RotatingFileHandler(
        os.path.join(log_dir, "qgeoint.log"),
        maxBytes=5 * 1024 * 1024, backupCount=3, encoding="utf-8")
    file_handler.setFormatter(fmt)
    logger.addHandler(file_handler)

    return logger
```

## 26.3 감사 로그

```python
# core/audit.py
from __future__ import annotations
import getpass
import json
import os
import socket
import sqlite3
from datetime import datetime
from typing import Any, Dict, List, Optional

DDL = """
CREATE TABLE IF NOT EXISTS audit_log (
    seq         INTEGER PRIMARY KEY AUTOINCREMENT,
    ts          TEXT NOT NULL,
    actor       TEXT NOT NULL,
    host        TEXT NOT NULL,
    action      TEXT NOT NULL,
    target      TEXT,
    params_json TEXT,
    result      TEXT,
    prev_hash   TEXT,
    entry_hash  TEXT
);
CREATE INDEX IF NOT EXISTS idx_audit_ts ON audit_log(ts);
CREATE INDEX IF NOT EXISTS idx_audit_action ON audit_log(action);
"""


def _hash(*parts: str) -> str:
    import hashlib
    h = hashlib.sha256()
    for p in parts:
        h.update((p or "").encode("utf-8"))
    return h.hexdigest()


class AuditLog:
    """추가 전용 감사 로그. 해시 체인으로 변조를 탐지한다."""

    def __init__(self, db_path: str):
        self.path = db_path
        os.makedirs(os.path.dirname(db_path), exist_ok=True)
        with sqlite3.connect(self.path) as c:
            c.executescript(DDL)

    def record(self, action: str, target: str = "",
               params: Optional[Dict[str, Any]] = None,
               result: str = "ok") -> None:
        ts = datetime.now().isoformat(timespec="seconds")
        actor = getpass.getuser()
        host = socket.gethostname()
        params_json = json.dumps(params or {}, ensure_ascii=False,
                                 sort_keys=True)

        with sqlite3.connect(self.path) as c:
            row = c.execute(
                "SELECT entry_hash FROM audit_log ORDER BY seq DESC LIMIT 1"
            ).fetchone()
            prev = row[0] if row else ""
            entry = _hash(prev, ts, actor, host, action, target,
                          params_json, result)
            c.execute(
                """INSERT INTO audit_log
                   (ts, actor, host, action, target, params_json,
                    result, prev_hash, entry_hash)
                   VALUES (?,?,?,?,?,?,?,?,?)""",
                (ts, actor, host, action, target, params_json,
                 result, prev, entry))

    def verify_chain(self) -> List[int]:
        """변조된 항목의 seq 목록을 반환. 비어 있으면 정상."""
        broken = []
        prev = ""
        with sqlite3.connect(self.path) as c:
            rows = c.execute(
                """SELECT seq, ts, actor, host, action, target,
                          params_json, result, prev_hash, entry_hash
                   FROM audit_log ORDER BY seq""").fetchall()
        for r in rows:
            (seq, ts, actor, host, action, target,
             params_json, result, prev_hash, entry_hash) = r
            expected = _hash(prev, ts, actor, host, action, target,
                             params_json, result)
            if prev_hash != prev or expected != entry_hash:
                broken.append(seq)
            prev = entry_hash
        return broken

    def export_csv(self, out_path: str) -> str:
        import csv
        with sqlite3.connect(self.path) as c:
            rows = c.execute("SELECT * FROM audit_log ORDER BY seq").fetchall()
            cols = [d[0] for d in c.execute(
                "SELECT * FROM audit_log LIMIT 0").description]
        with open(out_path, "w", encoding="utf-8-sig", newline="") as f:
            w = csv.writer(f)
            w.writerow(cols)
            w.writerows(rows)
        return out_path
```

> **TIP**
> 해시 체인은 암호학적 보증이 아니라 **실수·사고에 의한 변조 탐지** 수단이다.
> DB 파일에 쓰기 권한이 있는 사람은 체인 전체를 재계산할 수 있다.
> 강한 보증이 필요하면 로그를 외부 시스템으로 실시간 전송해야 한다.

## 26.4 예외 처리 정책

```python
# core/errors.py
from __future__ import annotations


class QGeoINTError(Exception):
    """사용자에게 보여줄 수 있는 오류."""
    def __init__(self, message: str, hint: str = ""):
        super().__init__(message)
        self.hint = hint


class DataError(QGeoINTError):
    pass


class ConfigError(QGeoINTError):
    pass


class DependencyError(QGeoINTError):
    pass
```

```python
# gui/error_handler.py
import traceback
import logging
from qgis.PyQt.QtWidgets import QMessageBox

from ..core.errors import QGeoINTError

logger = logging.getLogger("qgeoint")


def handle(parent, exc: Exception, context: str = "") -> None:
    if isinstance(exc, QGeoINTError):
        text = str(exc)
        detail = exc.hint
        logger.warning("%s: %s", context, text)
    else:
        text = "예상치 못한 오류가 발생했습니다."
        detail = "".join(traceback.format_exception(
            type(exc), exc, exc.__traceback__))
        logger.error("%s: %s", context, detail)

    box = QMessageBox(parent)
    box.setIcon(QMessageBox.Warning)
    box.setWindowTitle("QGeoINT")
    box.setText(text)
    if detail:
        box.setDetailedText(detail)
    box.exec_() if hasattr(box, "exec_") else box.exec()
```

핵심 정책: **사용자에게 보여줄 오류와 개발자용 오류를 구분한다.**
스택 트레이스를 다이얼로그 본문에 그대로 띄우면 사용자가 무엇을 해야 할지 알 수 없다.

### 버전 호환 노트 (Chapter 26)

- `QMessageBox.Warning`은 Qt6에서 `QMessageBox.Icon.Warning`이다.
  `compat._enum()`으로 처리한다.
- `sqlite3`는 전 버전 동일하다.
- `logging.handlers.RotatingFileHandler`의 `encoding` 인자는 Python 3.x 공통이다.

---

# Chapter 27. 테스트 전략

## 27.1 세 층위의 테스트

| 층위 | 대상 | QGIS 필요 | 실행 속도 |
|---|---|---|---|
| 단위 | `core/`, `analysis/` 순수 함수 | 불필요 | 빠름 |
| 통합 | `services/`, `repositories/` | 필요 (headless) | 보통 |
| 알고리즘 | Processing 알고리즘 | 필요 | 느림 |

표 27-1. 테스트 층위

Chapter 7의 계층 분리가 여기서 보상을 준다.
`core/`가 QGIS에 의존하지 않으므로 테스트의 70% 이상을 일반 Python 환경에서 돌릴 수 있다.

## 27.2 pytest 설정

```python
# tests/conftest.py
import os
import sys
import pytest

QGIS_AVAILABLE = True
try:
    from qgis.core import QgsApplication
except ImportError:
    QGIS_AVAILABLE = False

requires_qgis = pytest.mark.skipif(
    not QGIS_AVAILABLE, reason="QGIS 런타임이 필요합니다.")


@pytest.fixture(scope="session")
def qgis_app():
    if not QGIS_AVAILABLE:
        pytest.skip("QGIS 없음")
    from qgis.core import QgsApplication
    QgsApplication.setPrefixPath(os.environ.get("QGIS_PREFIX_PATH", "/usr"), True)
    app = QgsApplication([], False)
    app.initQgis()

    # Processing 초기화
    sys.path.append(os.path.join(app.prefixPath(), "python", "plugins"))
    from processing.core.Processing import Processing
    Processing.initialize()

    yield app
    app.exitQgis()


@pytest.fixture
def sample_dem():
    import numpy as np
    y, x = np.mgrid[0:64, 0:64]
    return (100.0 + 20.0 * np.sin(x / 8.0) + 15.0 * np.cos(y / 6.0)
            ).astype("float32")


@pytest.fixture
def tmp_gpkg(tmp_path):
    return str(tmp_path / "test.gpkg")
```

```ini
; pytest.ini
[pytest]
testpaths = tests
markers =
    slow: 실행 시간이 긴 테스트
    qgis: QGIS 런타임 필요
addopts = -q --strict-markers
```

## 27.3 골든 데이터 회귀 테스트

재현성 원칙을 자동 검증하는 핵심 장치다.

```python
# tests/test_golden.py
import json
import os
import numpy as np
import pytest

from qgeoint.core.change import detect_change, ChangeParams
from qgeoint.core.recipe import Recipe

GOLDEN_DIR = os.path.join(os.path.dirname(__file__), "golden")


def _load_golden(name):
    with open(os.path.join(GOLDEN_DIR, name), "r", encoding="utf-8") as f:
        return json.load(f)


@pytest.mark.parametrize("case", ["urban_small", "rural_flood", "forest_cut"])
def test_change_detection_regression(case):
    fixture = _load_golden("{0}.json".format(case))

    before = np.array(fixture["before"], dtype="float32")
    after = np.array(fixture["after"], dtype="float32")
    params = ChangeParams(**fixture["params"])

    result = detect_change(before, after, params)

    assert result.changed_px == fixture["expected"]["changed_px"], (
        "{0}: 변화 픽셀 수가 골든값과 다릅니다. "
        "알고리즘 변경이 의도된 것이라면 골든 데이터를 갱신하세요.".format(case))
    assert abs(result.changed_ratio -
               fixture["expected"]["changed_ratio"]) < 1e-6


def test_recipe_fingerprint_stability():
    """같은 레시피는 항상 같은 지문을 만들어야 한다."""
    r1 = Recipe("change_detection",
                inputs={"a": "x.tif", "b": "y.tif"},
                params={"threshold": 0.15, "k": 3.0})
    r2 = Recipe("change_detection",
                params={"k": 3.0, "threshold": 0.15},
                inputs={"b": "y.tif", "a": "x.tif"})
    assert r1.fingerprint() == r2.fingerprint()
```

> **ENGINEERING PRACTICE**
> 골든 데이터가 깨졌을 때 **무조건 갱신하지 않는다.**
> "왜 결과가 바뀌었는가"를 먼저 설명할 수 있어야 갱신한다.
> 설명 없는 골든 갱신은 회귀 테스트를 무력화한다.

## 27.4 Processing 알고리즘 테스트

```python
# tests/test_algorithms.py
import os
import numpy as np
import pytest
from .conftest import requires_qgis


@requires_qgis
def test_change_detection_algorithm(qgis_app, tmp_path, sample_dem):
    import processing
    from qgis.core import QgsApplication, QgsRasterLayer
    from qgeoint.processing.provider import QGeoINTProvider
    from qgeoint.repositories.raster_repo import write_array, RasterMeta

    provider = QGeoINTProvider()
    QgsApplication.processingRegistry().addProvider(provider)

    meta = RasterMeta(width=64, height=64,
                      geotransform=(0, 1, 0, 64, 0, -1),
                      projection="", nodata=None, dtype=6)

    before_path = str(tmp_path / "before.tif")
    after_path = str(tmp_path / "after.tif")
    after = sample_dem.copy()
    after[20:30, 20:30] += 50.0        # 인공 변화 삽입

    write_array(sample_dem, meta, before_path, cog=False)
    write_array(after, meta, after_path, cog=False)

    result = processing.run("qgeoint:change_detection", {
        "BEFORE": before_path,
        "AFTER": after_path,
        "METHOD": 0,
        "K": 3.0,
        "MIN_AREA": 4,
        "OUTPUT_MASK": str(tmp_path / "mask.tif"),
        "OUTPUT_POLY": str(tmp_path / "poly.gpkg"),
    })

    assert os.path.exists(result["OUTPUT_MASK"])
    layer = QgsRasterLayer(result["OUTPUT_MASK"], "mask")
    assert layer.isValid()
```

## 27.5 호환성 매트릭스 테스트

```python
# tests/test_version_matrix.py
import pytest
from qgeoint import compat


def test_minimum_version_supported():
    assert compat.QGIS_VERSION_INT >= 30600, \
        "QGeoINT는 QGIS 3.6 이상을 요구합니다."


def test_no_direct_pyqt_imports():
    """core/ 아래에 직접 PyQt import가 없어야 한다."""
    import os
    import re

    root = os.path.join(os.path.dirname(__file__), "..", "qgeoint", "core")
    pattern = re.compile(r"^\s*from\s+PyQt[56]|^\s*import\s+PyQt[56]",
                         re.MULTILINE)

    offenders = []
    for dirpath, _, files in os.walk(root):
        for fn in files:
            if not fn.endswith(".py"):
                continue
            path = os.path.join(dirpath, fn)
            with open(path, "r", encoding="utf-8") as f:
                if pattern.search(f.read()):
                    offenders.append(path)

    assert not offenders, \
        "직접 PyQt import 발견 (qgis.PyQt 사용 필요): {0}".format(offenders)


def test_no_version_branching_outside_compat():
    """compat.py 외부에서 QGIS_VERSION_INT를 참조하지 않아야 한다."""
    import os
    root = os.path.join(os.path.dirname(__file__), "..", "qgeoint")
    offenders = []
    for dirpath, _, files in os.walk(root):
        for fn in files:
            if not fn.endswith(".py") or fn == "compat.py":
                continue
            path = os.path.join(dirpath, fn)
            with open(path, "r", encoding="utf-8") as f:
                if "QGIS_VERSION_INT" in f.read():
                    offenders.append(path)
    assert not offenders, \
        "compat.py 외부의 버전 분기: {0}".format(offenders)
```

이 두 테스트가 Chapter 5와 Chapter 7의 아키텍처 규칙을 자동으로 강제한다.

## 27.6 headless 실행

```bash
# Linux
xvfb-run -s "+extension GLX -screen 0 1024x768x24" \
  python3 -m pytest tests/ -v

# Windows (OSGeo4W Shell)
python -m pytest tests/ -v -m "not qgis"
```

```bash
# 환경변수
export QGIS_PREFIX_PATH=/usr
export PYTHONPATH=/usr/share/qgis/python:/usr/share/qgis/python/plugins:$PYTHONPATH
export QT_QPA_PLATFORM=offscreen
```

> **TIP**
> `QT_QPA_PLATFORM=offscreen`을 설정하면 xvfb 없이도 상당수 테스트가 통과한다.
> 다만 레이아웃 렌더링 테스트는 여전히 실제 화면 버퍼가 필요하다.

### 버전 호환 노트 (Chapter 27)

- `Processing.initialize()` 경로는 3.6~4.x 동일하다.
- QGIS 4의 `QgsApplication` 초기화 시 `setPrefixPath` 인자는 변경되지 않았다.
- pytest 7 이상은 Python 3.7을 지원하지 않으므로,
  3.6 환경 CI에서는 pytest 6.2 계열을 고정해야 한다.

---

# Chapter 28. CI/CD와 배포

## 28.1 버전 매트릭스 CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.9"
      - run: pip install ruff
      - run: ruff check qgeoint/
      - run: ruff format --check qgeoint/

  core-tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python: ["3.7", "3.9", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python }}
      - run: pip install pytest numpy
      - name: core 단위 테스트 (QGIS 불필요)
        run: python -m pytest tests/ -m "not qgis" -q

  qgis-matrix:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        qgis: ["release-3_10", "release-3_28", "ltr", "latest"]
    container:
      image: qgis/qgis:${{ matrix.qgis }}
    steps:
      - uses: actions/checkout@v4
      - name: 호환성 게이트
        run: |
          xvfb-run -s "+extension GLX -screen 0 1024x768x24" \
            python3 -m pytest tests/test_compat.py \
                              tests/test_version_matrix.py -v
      - name: 전체 테스트
        run: |
          xvfb-run -s "+extension GLX -screen 0 1024x768x24" \
            python3 -m pytest tests/ -v
```

> **TIP**
> `fail-fast: false`가 중요하다. 3.10에서 실패했다고 4.x 결과를 못 보면
> 버전별 문제를 한 번에 파악할 수 없다.

## 28.2 정적 검사

```toml
# pyproject.toml
[tool.ruff]
line-length = 100
target-version = "py37"
exclude = ["qgeoint/ext_libs", "qgeoint/resources_rc.py"]

[tool.ruff.lint]
select = ["E", "F", "W", "I", "UP", "B", "C4", "SIM"]
ignore = [
    "E501",    # line-length는 formatter가 처리
    "UP006",   # py37 호환을 위해 typing.List 유지
    "UP007",   # Optional[X] 유지
    "UP035",   # typing 모듈 import 유지
]

[tool.ruff.lint.per-file-ignores]
"tests/*" = ["B011"]
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: [--maxkb=2000]
```

## 28.3 PyQGIS4 Checker 통합

```yaml
  migration-check:
    runs-on: ubuntu-latest
    continue-on-error: true      # 경고 수준
    steps:
      - uses: actions/checkout@v4
      - name: QGIS 4 마이그레이션 점검
        run: |
          pip install pyqgis4-checker || true
          pyqgis4-checker qgeoint/ --format json > migration_report.json || true
      - uses: actions/upload-artifact@v4
        with:
          name: migration-report
          path: migration_report.json
```

## 28.4 패키징

```python
# scripts/build_plugin.py
"""플러그인 ZIP을 생성한다. ext_libs 포함/제외 옵션 지원."""
from __future__ import annotations

import argparse
import os
import shutil
import zipfile

EXCLUDE_DIRS = {"__pycache__", ".git", ".pytest_cache", "tests", ".ruff_cache"}
EXCLUDE_EXT = {".pyc", ".pyo", ".log", ".qgs~"}


def should_skip(path: str, include_ext_libs: bool) -> bool:
    parts = path.replace("\\", "/").split("/")
    if any(p in EXCLUDE_DIRS for p in parts):
        return True
    if not include_ext_libs and "ext_libs" in parts:
        return True
    return os.path.splitext(path)[1] in EXCLUDE_EXT


def build(src_dir: str, out_zip: str, include_ext_libs: bool = True) -> str:
    plugin_name = os.path.basename(src_dir.rstrip("/\\"))
    with zipfile.ZipFile(out_zip, "w", zipfile.ZIP_DEFLATED) as z:
        for root, dirs, files in os.walk(src_dir):
            dirs[:] = [d for d in dirs if d not in EXCLUDE_DIRS]
            for fn in files:
                full = os.path.join(root, fn)
                rel = os.path.relpath(full, os.path.dirname(src_dir))
                if should_skip(rel, include_ext_libs):
                    continue
                z.write(full, rel)
    return out_zip


def verify(zip_path: str) -> None:
    """ZIP 구조 검증. 최상위에 플러그인 폴더 하나만 있어야 한다."""
    with zipfile.ZipFile(zip_path) as z:
        names = z.namelist()
        tops = {n.split("/")[0] for n in names}
        assert len(tops) == 1, "ZIP 최상위 항목이 여러 개입니다: {0}".format(tops)
        top = tops.pop()
        assert "{0}/metadata.txt".format(top) in names, "metadata.txt 누락"
        assert "{0}/__init__.py".format(top) in names, "__init__.py 누락"
    print("검증 통과: {0}".format(zip_path))


if __name__ == "__main__":
    ap = argparse.ArgumentParser()
    ap.add_argument("--src", default="qgeoint")
    ap.add_argument("--out", default="dist/qgeoint.zip")
    ap.add_argument("--slim", action="store_true",
                    help="ext_libs 제외 (온라인 환경용)")
    args = ap.parse_args()

    os.makedirs(os.path.dirname(args.out), exist_ok=True)
    build(args.src, args.out, include_ext_libs=not args.slim)
    verify(args.out)
```

두 종류의 배포본을 만든다.

| 배포본 | ext_libs | 크기 | 대상 |
|---|---|---|---|
| full | 포함 | 수십~수백 MB | 폐쇄망 |
| slim | 제외 | 수 MB | 인터넷 환경 |

표 28-1. 배포본 종류

## 28.5 릴리스 자동화

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags: ["v*"]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: 버전 일치 확인
        run: |
          TAG=${GITHUB_REF#refs/tags/v}
          META=$(grep '^version=' qgeoint/metadata.txt | cut -d= -f2)
          if [ "$TAG" != "$META" ]; then
            echo "태그($TAG)와 metadata.txt($META) 버전이 다릅니다."
            exit 1
          fi

      - name: 패키징
        run: |
          python scripts/build_plugin.py --out dist/qgeoint-full.zip
          python scripts/build_plugin.py --out dist/qgeoint-slim.zip --slim

      - name: 릴리스 생성
        uses: softprops/action-gh-release@v2
        with:
          files: |
            dist/qgeoint-full.zip
            dist/qgeoint-slim.zip
          generate_release_notes: true
```

## 28.6 사내 플러그인 저장소

폐쇄망에서는 QGIS 공식 저장소를 쓸 수 없다. 사내 XML 저장소를 만든다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugins>
  <pyqgis_plugin name="QGeoINT" version="0.1.0">
    <description>지리공간정보 분석 워크플로 도구 모음</description>
    <about>영상 변화탐지, 지형 분석, 다중출처 융합, 브리핑 자동화</about>
    <version>0.1.0</version>
    <qgis_minimum_version>3.6</qgis_minimum_version>
    <qgis_maximum_version>4.99</qgis_maximum_version>
    <homepage>http://intra.example/qgeoint</homepage>
    <file_name>qgeoint-full.zip</file_name>
    <download_url>http://intra.example/plugins/qgeoint-full.zip</download_url>
    <author_name>GIS팀</author_name>
    <experimental>False</experimental>
    <deprecated>False</deprecated>
  </pyqgis_plugin>
</plugins>
```

사용자는 QGIS 플러그인 관리자에서 이 XML URL을 저장소로 추가한다.

```python
# scripts/gen_repo_xml.py
import configparser
import os
from xml.etree import ElementTree as ET


def generate(metadata_path: str, download_base: str,
             zip_name: str, out_path: str) -> str:
    cfg = configparser.ConfigParser()
    cfg.read(metadata_path, encoding="utf-8")
    g = cfg["general"]

    root = ET.Element("plugins")
    p = ET.SubElement(root, "pyqgis_plugin",
                      {"name": g["name"], "version": g["version"]})

    mapping = {
        "description": g.get("description", ""),
        "about": g.get("about", ""),
        "version": g["version"],
        "qgis_minimum_version": g.get("qgisMinimumVersion", "3.6"),
        "qgis_maximum_version": g.get("qgisMaximumVersion", "4.99"),
        "homepage": g.get("homepage", ""),
        "file_name": zip_name,
        "download_url": download_base.rstrip("/") + "/" + zip_name,
        "author_name": g.get("author", ""),
        "experimental": g.get("experimental", "False"),
        "deprecated": g.get("deprecated", "False"),
    }
    for k, v in mapping.items():
        ET.SubElement(p, k).text = v

    ET.ElementTree(root).write(out_path, encoding="UTF-8",
                               xml_declaration=True)
    return out_path
```

### 버전 호환 노트 (Chapter 28)

- `qgis/qgis` Docker 이미지의 태그 체계는 시기에 따라 변한다.
  CI 정의는 주기적으로 점검한다.
- QGIS 3.6 컨테이너에서는 pytest 최신 버전이 설치되지 않는다.
  `pip install "pytest<7"`로 고정한다.
- 사내 저장소 XML 스키마는 3.x/4.x 동일하다.

---
---

# Part IX. 법·윤리와 종합 프로젝트

---

# Chapter 29. 법적·윤리적 프레임워크

## 29.1 이 장을 마지막에 두지 않은 이유

법·윤리를 부록으로 밀어내면 "개발이 끝난 뒤 검토하는 것"이 된다.
실제로는 **설계 단계에서 결정되는 사안**이다.
어떤 필드를 저장할지, 로그를 얼마나 남길지, 결과를 누구에게 보여줄지는
코드를 쓰기 전에 정해야 한다.

## 29.2 개인정보 관련 원칙

GeoINT 도구는 의도하지 않아도 개인정보를 다루게 되기 쉽다.

| 상황 | 개인정보 해당 여부 | 대응 |
|---|---|---|
| 고해상도 영상 속 차량 번호판 | 해당 가능 | 마스킹 또는 해상도 저하 |
| 소셜 게시물의 지오태그 + 계정명 | 해당 | 계정명 미저장, 게시물 ID만 |
| 건물 단위 주소 | 상황에 따라 해당 | 목적 범위 내에서만 |
| 개인 소유 토지 경계 | 해당 가능 | 공개 지적 정보 범위 준수 |
| 이동 궤적 데이터 | 해당 | 익명화·집계 후 사용 |

표 29-1. GeoINT에서 개인정보 판단이 필요한 상황

### 설계 원칙

1. **최소 수집** — 분석 목적에 필요한 필드만 저장한다.
2. **목적 구속** — 다른 목적으로 재사용하지 않는다.
3. **보유 기간 제한** — 자동 삭제 정책을 코드로 구현한다.
4. **접근 통제** — 산출물에 취급 범위를 표기한다.
5. **가명·익명 처리** — 개인 식별자를 저장하지 않는다.

```python
# core/privacy.py
from __future__ import annotations
import hashlib
import os
import re
from datetime import datetime, timedelta
from typing import Dict, List, Optional

# 개인 식별 가능성이 높은 필드명 패턴
SENSITIVE_PATTERNS = [
    r"name", r"이름", r"성명",
    r"phone", r"tel", r"전화", r"휴대",
    r"email", r"이메일",
    r"resident", r"주민",
    r"account", r"계정", r"user_?id", r"handle",
    r"address_detail", r"상세주소",
]

_COMPILED = [re.compile(p, re.IGNORECASE) for p in SENSITIVE_PATTERNS]


def flag_sensitive_fields(field_names: List[str]) -> List[str]:
    """개인정보 가능성이 있는 필드를 표시한다."""
    return [f for f in field_names
            if any(p.search(f) for p in _COMPILED)]


def pseudonymize(value: str, salt: str) -> str:
    """복원 불가 가명 처리. salt는 프로젝트 단위로 관리."""
    h = hashlib.sha256((salt + "|" + value).encode("utf-8"))
    return "ANON-" + h.hexdigest()[:12]


def coarsen_position(lon: float, lat: float,
                     grid_m: float = 500.0) -> tuple:
    """위치를 격자 중심으로 뭉갠다. 집계 공개용."""
    deg_per_m_lat = 1.0 / 111320.0
    deg_per_m_lon = deg_per_m_lat / max(0.1, abs(__import__("math")
                                                 .cos(__import__("math")
                                                      .radians(lat))))
    gy = grid_m * deg_per_m_lat
    gx = grid_m * deg_per_m_lon
    return (round(lon / gx) * gx + gx / 2.0,
            round(lat / gy) * gy + gy / 2.0)
```

인제스트 시 자동 점검한다.

```python
def audit_layer_privacy(layer) -> Dict[str, List[str]]:
    names = [f.name() for f in layer.fields()]
    flagged = flag_sensitive_fields(names)
    return {
        "flagged_fields": flagged,
        "recommendation": (
            ["해당 필드가 분석에 필요한지 재검토하고, "
             "불필요하면 제거하거나 가명 처리하십시오."] if flagged else []
        ),
    }
```

## 29.3 보유 기간 관리

```python
# core/retention.py
from __future__ import annotations
import os
import sqlite3
from datetime import datetime, timedelta
from typing import List


class RetentionPolicy:
    """자료 보유 기간을 코드로 강제한다."""

    def __init__(self, workspace: str, days: int = 365):
        self.workspace = workspace
        self.days = days
        self.db = os.path.join(workspace, "retention.sqlite")
        with sqlite3.connect(self.db) as c:
            c.execute("""CREATE TABLE IF NOT EXISTS retained (
                path TEXT PRIMARY KEY,
                registered_at TEXT,
                expires_at TEXT,
                purpose TEXT)""")

    def register(self, path: str, purpose: str,
                 days: int = None) -> None:
        d = days or self.days
        now = datetime.now()
        with sqlite3.connect(self.db) as c:
            c.execute("INSERT OR REPLACE INTO retained VALUES (?,?,?,?)",
                      (path, now.isoformat(timespec="seconds"),
                       (now + timedelta(days=d)).isoformat(timespec="seconds"),
                       purpose))

    def expired(self) -> List[str]:
        now = datetime.now().isoformat(timespec="seconds")
        with sqlite3.connect(self.db) as c:
            rows = c.execute(
                "SELECT path FROM retained WHERE expires_at < ?", (now,)
            ).fetchall()
        return [r[0] for r in rows]

    def report(self) -> str:
        exp = self.expired()
        if not exp:
            return "보유 기간이 만료된 자료가 없습니다."
        return ("보유 기간 만료 자료 {0}건:\n".format(len(exp)) +
                "\n".join("  - " + p for p in exp) +
                "\n\n삭제 여부를 검토하십시오. "
                "자동 삭제는 수행하지 않습니다.")
```

> **WARNING**
> 자동 삭제를 구현하지 않은 것은 의도적이다.
> 분석 자료의 삭제는 되돌릴 수 없으며, 진행 중인 사안의 증거일 수 있다.
> 도구는 **알리기만 하고**, 삭제는 사람이 결정한다.

## 29.4 위성영상 라이선스

| 자료원 | 라이선스 유형 | 재배포 |
|---|---|---|
| Sentinel (Copernicus) | 개방형 | 출처 표기 시 가능 |
| Landsat (USGS) | 퍼블릭 도메인 | 가능 |
| 상용 고해상도 위성 | 개별 계약 | 대부분 제한 |
| 항공사진(국가) | 기관 정책 | 확인 필요 |
| 드론 자체 촬영 | 촬영자 소유 | 촬영 허가 별도 |

표 29-2. 영상 자료 라이선스 유형

> **WARNING**
> **파생물(derivative) 규정에 특히 주의한다.**
> 상용 영상에서 만든 변화 탐지 폴리곤도 계약에 따라 파생물로 간주될 수 있다.
> "우리가 만든 벡터니까 우리 것"이라는 가정은 위험하다.

라이선스 정보를 데이터와 함께 관리한다.

```python
# core/license.py
from __future__ import annotations
from dataclasses import dataclass, asdict
from typing import Dict, List, Optional
import json


@dataclass
class DataLicense:
    dataset: str
    provider: str
    license_name: str
    attribution_text: str
    redistribution: str          # "allowed" / "restricted" / "prohibited"
    derivative_policy: str = ""
    contract_ref: str = ""
    notes: str = ""


class LicenseRegistry:
    def __init__(self):
        self._items: Dict[str, DataLicense] = {}

    def register(self, lic: DataLicense) -> None:
        self._items[lic.dataset] = lic

    def get(self, dataset: str) -> Optional[DataLicense]:
        return self._items.get(dataset)

    def attributions(self, datasets: List[str]) -> List[str]:
        out = []
        for d in datasets:
            lic = self._items.get(d)
            if lic and lic.attribution_text:
                out.append(lic.attribution_text)
        return sorted(set(out))

    def blockers(self, datasets: List[str]) -> List[str]:
        """외부 배포를 막는 항목."""
        issues = []
        for d in datasets:
            lic = self._items.get(d)
            if lic is None:
                issues.append("{0}: 라이선스 정보 미등록".format(d))
            elif lic.redistribution == "prohibited":
                issues.append("{0}: 재배포 금지 ({1})".format(d, lic.license_name))
            elif lic.redistribution == "restricted":
                issues.append("{0}: 재배포 제한 — 계약 확인 필요 ({1})"
                              .format(d, lic.contract_ref or "계약 참조 없음"))
        return issues
```

산출물 생성 시 자동으로 점검한다.

```python
def check_before_export(registry, datasets, external: bool = True):
    blockers = registry.blockers(datasets)
    if external and blockers:
        raise PermissionError(
            "외부 배포가 제한된 자료가 포함되어 있습니다:\n" +
            "\n".join("  - " + b for b in blockers))
    return registry.attributions(datasets)
```

## 29.5 분석 윤리 체크리스트

산출물을 내보내기 전 확인한다.

- [ ] 결론에 신뢰도 등급이 붙어 있는가
- [ ] 반대 근거를 검토했고 기록했는가
- [ ] 자동 탐지 결과와 사람 검토 결과가 구분되어 있는가
- [ ] 관측 시점과 보고 시점의 차이를 명시했는가
- [ ] 개인 식별 가능 정보가 포함되어 있지 않은가
- [ ] 사용된 자료의 출처 표기가 완전한가
- [ ] 재배포 제한 자료가 포함되어 있지 않은가
- [ ] "확정"으로 읽힐 수 있는 표현을 검토했는가
- [ ] 이 분석이 틀렸을 때의 영향을 고려했는가
- [ ] 분석 목적 범위를 벗어난 정보를 수집하지 않았는가

> **ENGINEERING PRACTICE**
> 마지막 두 항목이 가장 중요하다.
> **"이 분석이 틀렸을 때 누가 피해를 입는가"** 를 생각해 보면
> 필요한 신중함의 수준이 정해진다.
> 지도 위의 폴리곤 하나가 실제 사람의 삶에 영향을 줄 수 있다는 점을 잊지 않는다.

## 29.6 도구 자체의 한계 표기

```python
DISCLAIMER_TEXT = """
본 산출물은 QGeoINT 자동 분석 결과를 포함합니다.

- 자동 탐지 결과는 후보이며 확정된 사실이 아닙니다.
- 신뢰도 등급(A~F / 1~6)을 반드시 함께 해석하십시오.
- 관측 시점 이후의 변화는 반영되어 있지 않습니다.
- 사용된 영상의 위치정확도(CE90) 범위 내에서 위치를 해석하십시오.
- 본 산출물은 의사결정의 유일한 근거로 사용되어서는 안 됩니다.
"""
```

이 문구는 모든 산출물 템플릿에 기본 포함된다. 사용자가 삭제할 수는 있으나
기본값은 항상 포함이다.

### 버전 호환 노트 (Chapter 29)

- 이 장의 코드는 QGIS API에 의존하지 않는다.
- 개인정보 관련 법령은 국가·시기에 따라 다르다.
  실제 적용 시 반드시 소속 기관의 법무 검토를 받아야 한다.

---

# Chapter 30. 종합 프로젝트 — 홍수 피해 신속평가

## 30.1 시나리오

집중호우 이후 특정 유역의 침수 범위와 시설물 피해를 24시간 내에 평가한다.

**요구사항**

| 항목 | 내용 |
|---|---|
| 산출 기한 | 상황 발생 + 24시간 |
| 대상 범위 | 유역 단위 (약 300 km²) |
| 필수 산출물 | 침수 범위, 단절 도로, 피해 건물 후보, 상황도, 브리핑 |
| 자료 | Sentinel-2 전/후, DEM, OSM 도로·건물, 기상 관측 |
| 정확도 요구 | 침수 면적 ±15%, 위치 CE90 20 m 이내 |

표 30-1. 프로젝트 요구사항

## 30.2 파이프라인 설계

```text
[1] 인제스트     전/후 Sentinel-2, DEM, OSM
       ↓
[2] 전처리       구름 마스크, 정합 진단, 방사 정규화
       ↓
[3] 수체 추출    MNDWI 임계 → 전/후 수체 마스크
       ↓
[4] 침수 산출    (후 수체) − (전 상시수역) = 침수 범위
       ↓
[5] 지형 검증    DEM 저지대 조건으로 오탐 제거
       ↓
[6] 영향 분석    도로 교차, 건물 포함 판정
       ↓
[7] 융합         기상 관측·보도 자료와 교차 확인
       ↓
[8] 산출         상황도 PDF + 브리핑 PPTX + 보고서 DOCX
```

그림 30-1. 홍수 신속평가 파이프라인

## 30.3 파이프라인 구현

```python
# projects/flood_rapid.py
from __future__ import annotations

import os
from dataclasses import dataclass, field
from datetime import datetime
from typing import Dict, List, Optional

import numpy as np

from ..analysis.indices import mndwi
from ..analysis.threshold import choose_threshold
from ..analysis.morphology import closing, opening, filter_by_area
from ..analysis.coregistration import assess_coregistration
from ..analysis.terrain import slope_degrees
from ..repositories.raster_repo import read_band, write_array
from ..core.recipe import Recipe
from ..core.provenance import Provenance, attach_provenance
from ..core.audit import AuditLog


@dataclass
class FloodConfig:
    green_band: int = 2
    swir_band: int = 5
    water_threshold_method: str = "otsu"
    min_flood_area_px: int = 50
    max_slope_deg: float = 5.0        # 급경사지는 침수 불가
    dem_percentile_cap: float = 40.0  # 하위 40% 표고만 침수 후보


@dataclass
class FloodResult:
    flood_mask: np.ndarray
    permanent_water: np.ndarray
    flood_area_km2: float
    removed_by_slope: int
    removed_by_area: int
    diagnostics: Dict[str, float] = field(default_factory=dict)
    warnings: List[str] = field(default_factory=list)


def assess_flood(before_paths: Dict[str, str],
                 after_paths: Dict[str, str],
                 dem_path: str,
                 cfg: FloodConfig,
                 feedback=None) -> FloodResult:
    """전/후 영상과 DEM으로 침수 범위를 산출한다."""

    def info(msg):
        if feedback is not None:
            feedback.pushInfo(msg)

    warnings: List[str] = []

    # --- 1. 밴드 로드 -------------------------------------------------
    info("영상 밴드 로드 중...")
    g_before, meta = read_band(before_paths["green"])
    s_before, _ = read_band(before_paths["swir"])
    g_after, meta_a = read_band(after_paths["green"])
    s_after, _ = read_band(after_paths["swir"])

    if g_before.shape != g_after.shape:
        raise ValueError("전/후 영상 격자가 일치하지 않습니다. 사전 리샘플링 필요.")

    # --- 2. 정합 진단 -------------------------------------------------
    diag = assess_coregistration(g_before, g_after) or {}
    if diag:
        shift = max(abs(diag.get("median_dy", 0)), abs(diag.get("median_dx", 0)))
        info("정합 이동량: {0:.2f} px".format(shift))
        if shift > 1.5:
            warnings.append(
                "영상 정합 오차 {0:.1f} px. 침수 경계 정확도가 저하될 수 있습니다."
                .format(shift))

    # --- 3. 수체 지수 -------------------------------------------------
    info("MNDWI 계산 중...")
    w_before = mndwi(g_before, s_before)
    w_after = mndwi(g_after, s_after)

    th_before, label_b = choose_threshold(w_before, cfg.water_threshold_method)
    th_after, label_a = choose_threshold(w_after, cfg.water_threshold_method)
    info("수체 임계값 — 전: {0:.4f} ({1}) / 후: {2:.4f} ({3})"
         .format(th_before, label_b, th_after, label_a))

    water_before = w_before > th_before
    water_after = w_after > th_after

    # --- 4. 침수 = 후 수체 − 상시수역 ---------------------------------
    permanent = water_before & water_after
    flood = water_after & ~water_before
    raw_count = int(flood.sum())
    info("1차 침수 후보: {0:,} px".format(raw_count))

    # --- 5. 지형 조건 검증 --------------------------------------------
    info("지형 조건으로 오탐 제거 중...")
    dem, dem_meta = read_band(dem_path)
    if dem.shape != flood.shape:
        warnings.append("DEM 격자가 영상과 달라 지형 검증을 건너뜁니다.")
    else:
        cellsize = abs(meta.geotransform[1])
        slope = slope_degrees(np.nan_to_num(dem), cellsize)
        elev_cap = float(np.nanpercentile(dem[np.isfinite(dem)],
                                          cfg.dem_percentile_cap))

        terrain_ok = (slope <= cfg.max_slope_deg) & (dem <= elev_cap)
        before_slope = int(flood.sum())
        flood = flood & terrain_ok
        removed_slope = before_slope - int(flood.sum())
        info("  지형 조건으로 제거: {0:,} px (표고 상한 {1:.1f} m)"
             .format(removed_slope, elev_cap))

    # --- 6. 형태학 정리 -----------------------------------------------
    flood = closing(flood, iterations=1)
    flood = opening(flood, iterations=1)
    before_area = int(flood.sum())
    flood = filter_by_area(flood, cfg.min_flood_area_px)
    removed_area = before_area - int(flood.sum())

    # --- 7. 면적 산출 -------------------------------------------------
    px_area_m2 = abs(meta.geotransform[1] * meta.geotransform[5])
    area_km2 = int(flood.sum()) * px_area_m2 / 1_000_000.0
    info("최종 침수 면적: {0:.2f} km²".format(area_km2))

    if area_km2 < 0.01:
        warnings.append("탐지된 침수 면적이 매우 작습니다. 임계값을 재검토하십시오.")

    return FloodResult(
        flood_mask=flood,
        permanent_water=permanent,
        flood_area_km2=area_km2,
        removed_by_slope=locals().get("removed_slope", 0),
        removed_by_area=removed_area,
        diagnostics={
            "threshold_before": th_before,
            "threshold_after": th_after,
            "raw_candidate_px": raw_count,
            "coreg_shift_px": max(abs(diag.get("median_dy", 0)),
                                  abs(diag.get("median_dx", 0))),
        },
        warnings=warnings,
    )
```

## 30.4 영향 분석

```python
# projects/flood_impact.py
from __future__ import annotations
from typing import Dict, List

from qgis.core import (
    QgsVectorLayer, QgsFeature, QgsFeatureRequest, QgsGeometry,
    QgsSpatialIndex, QgsProject,
)


def intersect_infrastructure(flood_polygons: QgsVectorLayer,
                             target_layer: QgsVectorLayer,
                             name_field: str = "name") -> List[Dict]:
    """침수 폴리곤과 교차하는 시설물을 찾는다."""
    index = QgsSpatialIndex(flood_polygons.getFeatures())
    flood_geoms = {f.id(): f.geometry() for f in flood_polygons.getFeatures()}

    hits = []
    for feat in target_layer.getFeatures():
        geom = feat.geometry()
        candidates = index.intersects(geom.boundingBox())
        if not candidates:
            continue

        affected_len = 0.0
        for fid in candidates:
            fg = flood_geoms.get(fid)
            if fg is None or not geom.intersects(fg):
                continue
            inter = geom.intersection(fg)
            affected_len += inter.length() if geom.type() == 1 else inter.area()

        if affected_len > 0:
            hits.append({
                "id": feat.id(),
                "name": feat[name_field] if name_field in
                        [f.name() for f in target_layer.fields()] else "",
                "affected_measure": affected_len,
                "total_measure": geom.length() if geom.type() == 1
                                 else geom.area(),
            })

    return sorted(hits, key=lambda h: -h["affected_measure"])


def summarize_impact(road_hits: List[Dict],
                     building_hits: List[Dict]) -> Dict:
    total_road_m = sum(h["affected_measure"] for h in road_hits)
    return {
        "roads_affected": len(road_hits),
        "road_length_m": total_road_m,
        "buildings_affected": len(building_hits),
        "top_roads": [h["name"] for h in road_hits[:5] if h["name"]],
    }
```

## 30.5 산출물 조립

```python
# projects/flood_report.py
from __future__ import annotations
from datetime import datetime
from typing import List

from ..production.report_model import BriefingPackage, Finding
from ..production.docx_report import build_docx
from ..production.pptx_briefing import build_pptx
from ..production.layout import BriefingLayout
from ..core.grid import to_mgrs


def build_flood_package(result, impact, aoi_name: str, analyst: str,
                        before_time: str, after_time: str,
                        map_image: str, sources: List[str]
                        ) -> BriefingPackage:

    key = ("{0} 유역에서 약 {1:.2f} km²의 침수가 확인되었습니다. "
           "도로 {2}개소({3:.0f} m), 건물 {4}동이 영향권에 포함됩니다. "
           "관측 시간대는 {5} ~ {6}입니다."
           ).format(aoi_name, result.flood_area_km2,
                    impact["roads_affected"], impact["road_length_m"],
                    impact["buildings_affected"], before_time, after_time)

    findings = [
        Finding(
            finding_id="F-001",
            summary="침수 범위 {0:.2f} km² 탐지".format(result.flood_area_km2),
            location_mgrs="",
            location_desc="{0} 유역 전역".format(aoi_name),
            source_grade="B",
            cert_grade="2",
            observed_span="{0} ~ {1}".format(before_time, after_time),
            evidence=[
                "MNDWI 임계 {0:.3f}(후) 기준 수체 확장 확인".format(
                    result.diagnostics["threshold_after"]),
                "DEM 저지대 및 경사 5° 이하 조건 부합",
                "전 시점 상시수역 제외 처리 완료",
            ],
            counter_evidence=[
                "구름 그림자가 수체로 오분류될 가능성 잔존",
                "영상 정합 오차 {0:.2f} px".format(
                    result.diagnostics["coreg_shift_px"]),
            ] + result.warnings,
        ),
        Finding(
            finding_id="F-002",
            summary="도로 {0}개소 침수 영향".format(impact["roads_affected"]),
            location_mgrs="",
            location_desc=", ".join(impact["top_roads"][:3]) or "명칭 미상 구간",
            source_grade="B",
            cert_grade="3",
            observed_span="{0} ~ {1}".format(before_time, after_time),
            evidence=[
                "침수 폴리곤과 OSM 도로망 공간 교차 판정",
                "영향 연장 합계 {0:.0f} m".format(impact["road_length_m"]),
            ],
            counter_evidence=[
                "OSM 도로망의 최신성이 확인되지 않음",
                "교량 구간은 침수 판정에서 제외되지 않음",
            ],
        ),
    ]

    return BriefingPackage(
        title="{0} 홍수 피해 신속평가".format(aoi_name),
        aoi_name=aoi_name,
        analyst=analyst,
        key_judgement=key,
        findings=findings,
        sources=sources,
        limitations=[
            "위성 영상 기반 자동 분석 결과이며 현장 확인이 필요합니다.",
            "구름에 가려진 구역은 판정에서 제외되었습니다.",
            "건물 피해는 침수 영향권 포함 여부이며 실제 피해 정도가 아닙니다.",
            "관측 시점 이후의 상황 변화는 반영되지 않았습니다.",
        ],
        map_image=map_image,
    )
```

## 30.6 전체 실행 스크립트

```python
# projects/run_flood.py
"""홍수 신속평가 전체 실행. qgis_process 또는 Python 콘솔에서 실행."""
from __future__ import annotations
import os
from datetime import datetime

from qgeoint.projects.flood_rapid import assess_flood, FloodConfig
from qgeoint.projects.flood_impact import intersect_infrastructure, summarize_impact
from qgeoint.projects.flood_report import build_flood_package
from qgeoint.production.docx_report import build_docx
from qgeoint.production.pptx_briefing import build_pptx
from qgeoint.production.package import export_package
from qgeoint.core.audit import AuditLog
from qgeoint.core.recipe import Recipe


def run(config: dict) -> dict:
    out_dir = config["output_dir"]
    os.makedirs(out_dir, exist_ok=True)

    audit = AuditLog(os.path.join(out_dir, "audit.sqlite"))
    audit.record("flood_assessment_start", target=config["aoi_name"],
                 params={"before": config["before"], "after": config["after"]})

    # 1. 침수 산출
    result = assess_flood(config["before"], config["after"],
                          config["dem"], FloodConfig(**config.get("params", {})))

    # 2. 벡터화 및 영향 분석
    from qgeoint.repositories.raster_repo import write_array
    from qgeoint.analysis.vectorize import polygonize
    from qgis.core import QgsVectorLayer

    mask_path = os.path.join(out_dir, "flood_mask.tif")
    poly_path = os.path.join(out_dir, "flood.gpkg")
    # (meta는 assess_flood 내부에서 반환하도록 확장 필요)

    roads = QgsVectorLayer(config["roads"], "roads", "ogr")
    buildings = QgsVectorLayer(config["buildings"], "buildings", "ogr")
    flood_layer = QgsVectorLayer(poly_path + "|layername=change", "flood", "ogr")

    road_hits = intersect_infrastructure(flood_layer, roads)
    bld_hits = intersect_infrastructure(flood_layer, buildings)
    impact = summarize_impact(road_hits, bld_hits)

    # 3. 산출물
    pkg = build_flood_package(
        result, impact, config["aoi_name"], config["analyst"],
        config["before_time"], config["after_time"],
        config.get("map_image", ""), config["sources"])

    docx = build_docx(pkg, os.path.join(out_dir, "report.docx"))
    pptx = build_pptx(pkg, os.path.join(out_dir, "briefing.pptx"))
    zip_path = export_package(pkg, out_dir, extra_files=[docx, pptx])

    # 4. 레시피 저장 (재현성)
    Recipe(operation="flood_rapid_assessment",
           inputs=config, params=result.diagnostics
           ).save(os.path.join(out_dir, "recipe.json"))

    audit.record("flood_assessment_complete", target=config["aoi_name"],
                 params={"area_km2": result.flood_area_km2},
                 result="ok")

    return {
        "flood_area_km2": result.flood_area_km2,
        "impact": impact,
        "warnings": result.warnings,
        "package": zip_path,
    }
```

## 30.7 이 프로젝트가 검증하는 것

| 원칙 | 구현 위치 |
|---|---|
| 재현성 | `Recipe.save()`, 모든 임계값을 진단 정보로 반환 |
| 추적성 | `AuditLog`, `Provenance`, `feedback.pushInfo()` 이력 |
| 방어가능성 | `Finding.counter_evidence`, 등급 표기, `limitations` |

표 30-2. 세 원칙의 구현 대응

## 30.8 개선 과제

이 프로젝트를 실제 운영 수준으로 끌어올리려면 다음이 추가로 필요하다.

1. **구름·구름그림자 마스크** — Sentinel-2 SCL 밴드 활용
2. **SAR 보완** — 구름이 두꺼울 때 Sentinel-1 후방산란 기반 수체 추출
3. **시계열 상시수역 정의** — 단일 전 시점이 아니라 1년치 중앙값
4. **교량·고가 제외** — OSM `bridge=yes` 태그 활용
5. **건물 피해 등급화** — 침수 심도 추정(DEM − 수면고) 반영
6. **현장 보고 연동** — 신고 지점과의 교차 확인으로 등급 상향

각 항목은 앞선 장들의 기법을 조합하면 구현할 수 있다.

### 버전 호환 노트 (Chapter 30)

- `QgsSpatialIndex(layer.getFeatures())` 생성자는 3.6~4.x 동일하다.
- `np.nanpercentile`은 numpy 1.9+ 이므로 문제없다.
- `QgsGeometry.type()` 반환값은 3.30 이후 `Qgis.GeometryType` enum이다.
  정수 비교(`== 1`)는 양쪽에서 동작하지만, `compat.GEOM_LINE` 사용을 권장한다.

---
---

# 부록

---

# 부록 A. QGIS 3.6 → 4.x API 변경 대조표

## A.1 Qt 관련

| 항목 | Qt5 (QGIS 3.x) | Qt6 (QGIS 4.x) | 대응 |
|---|---|---|---|
| 다이얼로그 실행 | `dialog.exec_()` | `dialog.exec()` | `compat.exec_dialog()` |
| 정렬 플래그 | `Qt.AlignLeft` | `Qt.AlignmentFlag.AlignLeft` | `compat.ALIGN_LEFT` |
| 키 코드 | `Qt.Key_Escape` | `Qt.Key.Key_Escape` | `compat.KEY_ESCAPE` |
| 커서 | `Qt.CrossCursor` | `Qt.CursorShape.CrossCursor` | `compat.CURSOR_CROSS` |
| 마우스 버튼 | `Qt.LeftButton` | `Qt.MouseButton.LeftButton` | `compat.MOUSE_LEFT` |
| 체크 상태 | `Qt.Checked` | `Qt.CheckState.Checked` | `compat.CHECKED` |
| 데이터 역할 | `Qt.UserRole` | `Qt.ItemDataRole.UserRole` | `compat.USER_ROLE` |
| 화면 정보 | `QDesktopWidget` | 제거됨 → `QScreen` | `compat.screen_geometry()` |
| 메시지박스 아이콘 | `QMessageBox.Warning` | `QMessageBox.Icon.Warning` | `compat._enum()` |
| 정규표현식 | `QRegExp` | 제거됨 → `QRegularExpression` | Python `re` 사용 권장 |
| 문자열 변환 | `QString` 암묵 변환 | 동일 | 영향 없음 |

표 A-1. Qt5 → Qt6 주요 변경

## A.2 QGIS Core API

| 항목 | 3.6 | 변경 시점 | 4.x |
|---|---|---|---|
| 지오메트리 타입 | `QgsWkbTypes.PointGeometry` | 3.30 | `Qgis.GeometryType.Point` |
| 메시지 레벨 | `Qgis.Info` | 3.36 | `Qgis.MessageLevel.Info` |
| 좌표변환 컨텍스트 | `QgsProject` 인자 | 3.8 | `QgsCoordinateTransformContext` |
| 필드 타입 | `QVariant.String` | 3.36 | `QMetaType.Type.QString` (병행) |
| 시간 속성 | 없음 | 3.14 | `QgsVectorLayerTemporalProperties` |
| Processing 경고 | 없음 | 3.16 | `feedback.pushWarning()` |
| 그래프 엣지 | `edge.outVertex()` | 3.12 | `edge.fromVertex()` |
| 심볼 레이어 속성 | `QgsSymbolLayer.PropertyFillColor` | 3.36 | `.Property.PropertyFillColor` |
| 레이아웃 방향 | `QgsLayoutItemPage.Landscape` | 3.36 | `.Orientation.Landscape` |
| 격자 주기 표현식 | 없음 | 3.10 | `setAnnotationExpression()` |

표 A-2. QGIS Core API 변경 이력

## A.3 버전 판별 상수

```python
from qgis.core import Qgis

Qgis.QGIS_VERSION        # "4.2.1-Belém do Pará"
Qgis.QGIS_VERSION_INT    # 40201
Qgis.QGIS_RELEASE_NAME   # "Belém do Pará"
```

계산식: `major * 10000 + minor * 100 + patch`

| 버전 | INT |
|---|---:|
| 3.6.0 | 30600 |
| 3.10.14 | 31014 |
| 3.16.16 | 31616 |
| 3.28.15 | 32815 |
| 3.34.0 | 33400 |
| 3.44.13 | 34413 |
| 4.0.0 | 40000 |
| 4.2.1 | 40201 |

표 A-3. QGIS 버전 정수 대응

## A.4 마이그레이션 점검 스크립트

```python
# scripts/check_migration.py
"""Qt6/QGIS4 호환성 위험 패턴을 찾는다."""
from __future__ import annotations
import os
import re
import sys
from typing import Dict, List

PATTERNS: Dict[str, str] = {
    r"from\s+PyQt5": "직접 PyQt5 import → qgis.PyQt 사용",
    r"from\s+PyQt6": "직접 PyQt6 import → qgis.PyQt 사용",
    r"\.exec_\(\)": "exec_() → compat.exec_dialog() 사용",
    r"QDesktopWidget": "Qt6에서 제거됨 → QScreen",
    r"QRegExp": "Qt6에서 제거됨 → QRegularExpression 또는 re",
    r"Qt\.Align(Left|Right|Center|Top|Bottom)": "스코프드 enum 필요",
    r"Qt\.Key_": "스코프드 enum 필요",
    r"Qt\.(Left|Right|Mid)Button": "스코프드 enum 필요",
    r"QgsWkbTypes\.\w+Geometry": "Qgis.GeometryType 권장",
    r"\.outVertex\(\)|\.inVertex\(\)": "3.12+ fromVertex()/toVertex()",
    r"QVariant\.": "QMetaType 전환 검토 (3.36+)",
}

SKIP_DIRS = {"ext_libs", "__pycache__", ".git", "tests"}


def scan(root: str) -> List[tuple]:
    hits = []
    compiled = {re.compile(p): msg for p, msg in PATTERNS.items()}
    for dirpath, dirs, files in os.walk(root):
        dirs[:] = [d for d in dirs if d not in SKIP_DIRS]
        for fn in files:
            if not fn.endswith(".py") or fn == "compat.py":
                continue
            path = os.path.join(dirpath, fn)
            with open(path, "r", encoding="utf-8", errors="replace") as f:
                for lineno, line in enumerate(f, start=1):
                    for pattern, msg in compiled.items():
                        if pattern.search(line):
                            hits.append((path, lineno, line.strip(), msg))
    return hits


if __name__ == "__main__":
    root = sys.argv[1] if len(sys.argv) > 1 else "qgeoint"
    results = scan(root)
    for path, lineno, code, msg in results:
        print("{0}:{1}: {2}\n    → {3}".format(path, lineno, code, msg))
    print("\n총 {0}건".format(len(results)))
    sys.exit(1 if results else 0)
```

---

# 부록 B. compat.py 전체 소스

Chapter 5, 18, 22, 25에서 추가된 내용을 통합한 최종본이다.

```python
# qgeoint/compat.py
"""QGIS 3.6 ~ 4.x / Qt5 ~ Qt6 호환 계층 (통합본).

규칙: 이 모듈 밖에서는 버전 분기를 하지 않는다.
"""
from __future__ import annotations

from typing import Any, Optional

from qgis.core import Qgis
from qgis.PyQt import QtCore, QtGui, QtWidgets
from qgis.PyQt.QtCore import QT_VERSION_STR

# ======================================================================
# 1. 버전 판별
# ======================================================================
QGIS_VERSION_INT = Qgis.QGIS_VERSION_INT
QT6 = QT_VERSION_STR.startswith("6")
QGIS4 = QGIS_VERSION_INT >= 40000


def qgis_version_int() -> int:
    return QGIS_VERSION_INT


def at_least(major: int, minor: int) -> bool:
    return QGIS_VERSION_INT >= (major * 10000 + minor * 100)


def runtime_report() -> str:
    return "QGIS {0} / Qt {1} / {2}".format(
        Qgis.QGIS_VERSION, QT_VERSION_STR, "Qt6" if QT6 else "Qt5")


# ======================================================================
# 2. enum 해석 헬퍼
# ======================================================================
def _enum(owner: Any, scope: str, name: str) -> Any:
    scoped = getattr(owner, scope, None)
    if scoped is not None and hasattr(scoped, name):
        return getattr(scoped, name)
    return getattr(owner, name)


Qt = QtCore.Qt

ALIGN_LEFT = _enum(Qt, "AlignmentFlag", "AlignLeft")
ALIGN_RIGHT = _enum(Qt, "AlignmentFlag", "AlignRight")
ALIGN_CENTER = _enum(Qt, "AlignmentFlag", "AlignCenter")
ALIGN_VCENTER = _enum(Qt, "AlignmentFlag", "AlignVCenter")
ALIGN_HCENTER = _enum(Qt, "AlignmentFlag", "AlignHCenter")

KEY_ESCAPE = _enum(Qt, "Key", "Key_Escape")
KEY_RETURN = _enum(Qt, "Key", "Key_Return")
KEY_DELETE = _enum(Qt, "Key", "Key_Delete")

CURSOR_CROSS = _enum(Qt, "CursorShape", "CrossCursor")
CURSOR_WAIT = _enum(Qt, "CursorShape", "WaitCursor")
CURSOR_ARROW = _enum(Qt, "CursorShape", "ArrowCursor")

MOUSE_LEFT = _enum(Qt, "MouseButton", "LeftButton")
MOUSE_RIGHT = _enum(Qt, "MouseButton", "RightButton")
MOUSE_MIDDLE = _enum(Qt, "MouseButton", "MiddleButton")

CHECKED = _enum(Qt, "CheckState", "Checked")
UNCHECKED = _enum(Qt, "CheckState", "Unchecked")
PARTIAL = _enum(Qt, "CheckState", "PartiallyChecked")

ITEM_SELECTABLE = _enum(Qt, "ItemFlag", "ItemIsSelectable")
ITEM_ENABLED = _enum(Qt, "ItemFlag", "ItemIsEnabled")
ITEM_CHECKABLE = _enum(Qt, "ItemFlag", "ItemIsUserCheckable")

USER_ROLE = _enum(Qt, "ItemDataRole", "UserRole")
DISPLAY_ROLE = _enum(Qt, "ItemDataRole", "DisplayRole")

MSG_WARNING = _enum(QtWidgets.QMessageBox, "Icon", "Warning")
MSG_CRITICAL = _enum(QtWidgets.QMessageBox, "Icon", "Critical")
MSG_INFORMATION = _enum(QtWidgets.QMessageBox, "Icon", "Information")


# ======================================================================
# 3. QGIS enum
# ======================================================================
try:
    GEOM_POINT = Qgis.GeometryType.Point
    GEOM_LINE = Qgis.GeometryType.Line
    GEOM_POLYGON = Qgis.GeometryType.Polygon
    GEOM_UNKNOWN = Qgis.GeometryType.Unknown
except AttributeError:
    from qgis.core import QgsWkbTypes
    GEOM_POINT = QgsWkbTypes.PointGeometry
    GEOM_LINE = QgsWkbTypes.LineGeometry
    GEOM_POLYGON = QgsWkbTypes.PolygonGeometry
    GEOM_UNKNOWN = QgsWkbTypes.UnknownGeometry

try:
    LEVEL_INFO = Qgis.MessageLevel.Info
    LEVEL_WARNING = Qgis.MessageLevel.Warning
    LEVEL_CRITICAL = Qgis.MessageLevel.Critical
    LEVEL_SUCCESS = Qgis.MessageLevel.Success
except AttributeError:
    LEVEL_INFO = Qgis.Info
    LEVEL_WARNING = Qgis.Warning
    LEVEL_CRITICAL = Qgis.Critical
    LEVEL_SUCCESS = Qgis.Success


# ======================================================================
# 4. 위젯 · 다이얼로그
# ======================================================================
def exec_dialog(dialog) -> int:
    runner = getattr(dialog, "exec", None) or getattr(dialog, "exec_")
    return runner()


def screen_geometry() -> "QtCore.QRect":
    app = QtWidgets.QApplication.instance()
    screen = app.primaryScreen() if app else None
    if screen is not None:
        return screen.availableGeometry()
    return QtCore.QRect(0, 0, 1280, 800)


def add_action_shortcut(action, sequence: str) -> None:
    action.setShortcut(QtGui.QKeySequence(sequence))


# ======================================================================
# 5. 좌표변환
# ======================================================================
def transform_context():
    from qgis.core import QgsProject
    project = QgsProject.instance()
    getter = getattr(project, "transformContext", None)
    return getter() if getter is not None else None


def make_transform(src_crs, dst_crs):
    from qgis.core import QgsCoordinateTransform, QgsProject
    ctx = transform_context()
    if ctx is not None:
        return QgsCoordinateTransform(src_crs, dst_crs, ctx)
    return QgsCoordinateTransform(src_crs, dst_crs, QgsProject.instance())


# ======================================================================
# 6. 레이어 URI
# ======================================================================
def memory_uri(geometry: str, crs_authid: str, fields: str = "") -> str:
    base = "{0}?crs={1}".format(geometry, crs_authid)
    if fields:
        base += "&" + fields
    return base + "&index=yes"


def make_field(name: str, type_name: str = "string", comment: str = ""):
    """QVariant / QMetaType 양쪽을 흡수하는 필드 팩토리."""
    from qgis.core import QgsField
    from qgis.PyQt.QtCore import QVariant
    mapping = {
        "string": QVariant.String,
        "int": QVariant.Int,
        "double": QVariant.Double,
        "bool": QVariant.Bool,
        "date": QVariant.Date,
        "datetime": QVariant.DateTime,
    }
    f = QgsField(name, mapping.get(type_name, QVariant.String))
    if comment:
        f.setComment(comment)
    return f


# ======================================================================
# 7. 네트워크 분석 그래프
# ======================================================================
def edge_from_vertex(graph, edge_id: int) -> int:
    edge = graph.edge(edge_id)
    getter = getattr(edge, "fromVertex", None) or getattr(edge, "outVertex")
    return getter()


def edge_to_vertex(graph, edge_id: int) -> int:
    edge = graph.edge(edge_id)
    getter = getattr(edge, "toVertex", None) or getattr(edge, "inVertex")
    return getter()


# ======================================================================
# 8. Processing 피드백
# ======================================================================
def push_warning(feedback, text: str) -> None:
    fn = getattr(feedback, "pushWarning", None)
    if fn is not None:
        fn(text)
    else:
        feedback.pushInfo("경고: " + text)


def report_error(feedback, text: str, fatal: bool = False) -> None:
    try:
        feedback.reportError(text, fatalError=fatal)
    except TypeError:
        feedback.reportError(text)


# ======================================================================
# 9. Processing 공급자 접두어
# ======================================================================
def provider_prefix(*candidates: str) -> Optional[str]:
    from qgis.core import QgsApplication
    reg = QgsApplication.processingRegistry()
    for pid in candidates:
        if reg.providerById(pid) is not None:
            return pid
    return None


def grass_prefix() -> str:
    pid = provider_prefix("grass7", "grass")
    if pid is None:
        raise RuntimeError("GRASS Processing 공급자를 찾을 수 없습니다.")
    return pid
```

---

# 부록 C. 좌표·격자 레퍼런스

## C.1 좌표 표기 형식

| 형식 | 예 | 용도 |
|---|---|---|
| 십진도 (DD) | 37.5665, 126.9780 | 데이터 저장 표준 |
| 도분초 (DMS) | 37°33'59.4"N 126°58'40.8"E | 문서·구술 |
| 도분 (DM) | 37°33.99'N 126°58.68'E | 항해·항공 |
| UTM | 52S 322037E 4159012N | 거리 계산 편의 |
| MGRS | 52S CH 22037 59012 | 격자 참조 |
| 평면직각 | X=552037, Y=192012 (EPSG:5186) | 국내 측량 |

표 C-1. 좌표 표기 형식

## C.2 변환 코드 모음

```python
# 십진도 ↔ 도분초
def dd_to_dms(dd: float, is_lat: bool = True) -> str:
    hemi = ("N" if dd >= 0 else "S") if is_lat else ("E" if dd >= 0 else "W")
    v = abs(dd)
    d = int(v)
    m = int((v - d) * 60)
    s = (v - d - m / 60.0) * 3600.0
    return "{0}°{1:02d}'{2:05.2f}\"{3}".format(d, m, s, hemi)


def dms_to_dd(d: float, m: float, s: float, hemi: str) -> float:
    v = d + m / 60.0 + s / 3600.0
    return -v if hemi.upper() in ("S", "W") else v
```

```python
# 경위도 → UTM 존 번호
def utm_zone(lon: float) -> int:
    return int((lon + 180.0) / 6.0) + 1


def utm_epsg(lon: float, lat: float) -> int:
    zone = utm_zone(lon)
    return (32600 if lat >= 0 else 32700) + zone


# 한국: 경도 124~132 → 51N(32651) 또는 52N(32652)
```

## C.3 거리·면적 계산

```python
from qgis.core import QgsDistanceArea, QgsUnitTypes, QgsProject


def make_measurer(crs, ellipsoid: str = "WGS84") -> QgsDistanceArea:
    da = QgsDistanceArea()
    da.setSourceCrs(crs, QgsProject.instance().transformContext())
    da.setEllipsoid(ellipsoid)
    return da


def geodesic_distance_m(da, p1, p2) -> float:
    return da.measureLine(p1, p2)


def area_km2(da, geometry) -> float:
    m2 = da.measureArea(geometry)
    return da.convertAreaMeasurement(m2, QgsUnitTypes.AreaSquareKilometers)
```

## C.4 정확도 표기 대응

| 표기 | 정의 | 환산 |
|---|---|---|
| RMSE(수평) | 제곱평균제곱근 오차 | 기준값 |
| CE90 | 90% 원형오차 | ≈ 2.146 × RMSE |
| CE95 | 95% 원형오차 | ≈ 2.448 × RMSE |
| LE90 | 90% 수직오차 | ≈ 1.6449 × RMSE(수직) |
| NSSDA 95% | 미국 표준 | ≈ 1.7308 × RMSE |

표 C-2. 위치정확도 표기 환산

```python
def rmse_to_ce90(rmse_h: float) -> float:
    return 2.146 * rmse_h


def rmse_to_le90(rmse_v: float) -> float:
    return 1.6449 * rmse_v
```

## C.5 MGRS 100 km 격자 문자

MGRS의 100 km 격자 식별 문자는 UTM 존과 세트 번호에 따라 결정된다.
직접 구현하기보다 검증된 라이브러리를 쓰는 편이 안전하다.
자체 구현이 필요하면 다음 규칙을 참고한다.

- 열 문자: 존 번호 mod 3 에 따라 A–H / J–R / S–Z 세트
- 행 문자: 존 번호 홀짝에 따라 A–V 시작점이 5행 이동
- I, O는 사용하지 않음 (1, 0과 혼동 방지)

> **WARNING**
> MGRS 자체 구현은 오차가 발견되기 어렵다.
> 반드시 알려진 기준점 목록으로 왕복 변환 테스트를 수행한다.

---

# 부록 D. 폐쇄망 배포 체크리스트

## D.1 반입 전 준비

- [ ] `qgeoint-full.zip` 빌드 완료 (ext_libs 포함)
- [ ] 대상 환경의 QGIS 버전 확인
- [ ] 대상 환경의 Python 버전 확인 → 해당 ABI 휠 포함 여부 확인
- [ ] 벤더링 패키지 라이선스 파일 동봉 (`ext_libs/LICENSES/`)
- [ ] `THIRD_PARTY.md` 최신화
- [ ] 악성코드 검사 완료
- [ ] ZIP 구조 검증 (`scripts/build_plugin.py verify`)
- [ ] SHA-256 체크섬 산출 및 별도 전달

```bash
# 체크섬 생성
sha256sum dist/qgeoint-full.zip > dist/qgeoint-full.zip.sha256

# Windows
certutil -hashfile dist\qgeoint-full.zip SHA256
```

## D.2 사전 반입 자료

플러그인 외에 별도로 반입해야 하는 것들이다.

- [ ] PROJ 격자 파일 (`.gsb`, `.tif`) — 정밀 좌표변환용
- [ ] 지오이드 모델 래스터 — 표고 변환용
- [ ] 지명 사전 DB (`gazetteer.sqlite`) — 오프라인 지오코딩
- [ ] 배경지도 MBTiles / GeoPackage
- [ ] AI 모델 파일 (`.onnx`) + 모델 카드 JSON
- [ ] 한글 폰트 (산출물 생성용)
- [ ] OSM 추출본 GeoPackage

## D.3 설치 후 자체 진단

```python
# scripts/self_check.py
"""설치 직후 실행하는 자체 진단."""
from __future__ import annotations
import sys
from typing import List, Tuple

CheckResult = Tuple[str, bool, str]


def run_all() -> List[CheckResult]:
    results: List[CheckResult] = []

    # 1. QGIS 버전
    try:
        from qgeoint.compat import runtime_report, at_least
        ok = at_least(3, 6)
        results.append(("QGIS 버전", ok, runtime_report()))
    except Exception as e:
        results.append(("QGIS 버전", False, str(e)))

    # 2. 의존성 경로
    try:
        from qgeoint.ext_libs import install, report
        install()
        results.append(("ext_libs 경로", True, report()))
    except Exception as e:
        results.append(("ext_libs 경로", False, str(e)))

    # 3. 선택 의존성
    from qgeoint.core.optional import GATES
    for mod, feature in [("mgrs", "MGRS 격자"),
                         ("onnxruntime", "AI 추론"),
                         ("docx", "DOCX 보고서"),
                         ("pptx", "PPTX 브리핑"),
                         ("scipy", "고속 라벨링")]:
        avail = GATES.available(mod)
        results.append(("의존성: {0}".format(feature), avail,
                        "사용 가능" if avail else "미설치 (해당 기능 비활성)"))

    # 4. GDAL 드라이버
    try:
        from qgeoint.ingest.drivers import driver_report
        rep = driver_report()
        ok = not rep["missing_required"]
        msg = ("필수 드라이버 모두 사용 가능" if ok
               else "누락: " + ", ".join(rep["missing_required"]))
        results.append(("GDAL 드라이버", ok, msg))
    except Exception as e:
        results.append(("GDAL 드라이버", False, str(e)))

    # 5. 좌표변환 왕복
    try:
        from qgis.core import QgsPointXY
        from qgeoint.core.crs import roundtrip_error_m
        err = roundtrip_error_m(QgsPointXY(127.0, 37.5), "EPSG:4326", "EPSG:5186")
        ok = err < 0.001
        results.append(("좌표변환 왕복", ok, "오차 {0:.6f} m".format(err)))
    except Exception as e:
        results.append(("좌표변환 왕복", False, str(e)))

    # 6. Processing 공급자
    try:
        from qgis.core import QgsApplication
        reg = QgsApplication.processingRegistry()
        ok = reg.providerById("qgeoint") is not None
        results.append(("Processing 공급자", ok,
                        "등록됨" if ok else "미등록"))
    except Exception as e:
        results.append(("Processing 공급자", False, str(e)))

    return results


def print_report() -> int:
    results = run_all()
    width = max(len(name) for name, _, _ in results)
    failed = 0
    for name, ok, detail in results:
        mark = "OK  " if ok else "FAIL"
        print("[{0}] {1:<{2}}  {3}".format(mark, name, width, detail))
        if not ok:
            failed += 1
    print("\n{0}개 항목 중 {1}개 실패".format(len(results), failed))
    return failed


if __name__ == "__main__":
    sys.exit(print_report())
```

QGIS Python 콘솔에서 실행한다.

```python
from qgeoint.scripts.self_check import print_report
print_report()
```

## D.4 운영 점검 주기

| 주기 | 항목 |
|---|---|
| 매 실행 | 자체 진단 요약을 로그에 기록 |
| 주간 | 감사 로그 체인 검증 (`AuditLog.verify_chain()`) |
| 월간 | 보유 기간 만료 자료 보고 (`RetentionPolicy.report()`) |
| 분기 | 골든 데이터 회귀 테스트 재실행 |
| 반기 | PROJ 격자·지오이드 모델 갱신 검토 |
| 연간 | QGIS 버전 상향 검토, 의존성 라이선스 재확인 |

표 D-1. 운영 점검 주기

## D.5 문제 발생 시 수집할 정보

```python
# scripts/collect_diagnostics.py
def collect() -> str:
    import json, platform, sys, os
    from qgis.core import Qgis, QgsApplication

    info = {
        "qgis_version": Qgis.QGIS_VERSION,
        "qgis_version_int": Qgis.QGIS_VERSION_INT,
        "python": sys.version,
        "platform": platform.platform(),
        "prefix_path": QgsApplication.prefixPath(),
        "profile_path": QgsApplication.qgisSettingsDirPath(),
        "sys_path": sys.path[:20],
        "env": {k: v for k, v in os.environ.items()
                if k.startswith(("QGIS", "PROJ", "GDAL", "PYTHON"))},
    }
    try:
        from qgeoint.scripts.self_check import run_all
        info["self_check"] = [
            {"name": n, "ok": o, "detail": d} for n, o, d in run_all()]
    except Exception as e:
        info["self_check_error"] = str(e)

    return json.dumps(info, ensure_ascii=False, indent=2)
```

---

# 부록 E. 용어집

## E.1 GeoINT 일반

| 영문 | 한국어 | 설명 |
|---|---|---|
| Geospatial Intelligence (GEOINT) | 지리공간정보 분석 | 공간 자료로 판단을 생산하는 활동 |
| Area of Interest (AOI) | 관심지역 | 분석 대상 공간 범위 |
| Collection | 수집 | 자료를 확보하는 단계 |
| Exploitation | 분석·활용 | 자료에서 정보를 도출하는 단계 |
| Provenance | 출처·내력 | 자료가 만들어진 경로 기록 |
| Admiralty Code | 애드미럴티 등급 | 출처 신뢰도 + 정보 확실성 표기 |
| Key Judgement | 핵심 판단 | 보고서 최상단의 결론 문장 |
| Finding | 발견사항 | 개별 분석 결론 단위 |
| Corroboration | 교차 확인 | 독립 출처로 뒷받침 |
| Confidence Level | 신뢰도 | 결론의 확실성 정도 |

표 E-1. GeoINT 일반 용어

## E.2 원격탐사·영상

| 영문 | 한국어 | 설명 |
|---|---|---|
| Ground Sample Distance (GSD) | 지상표본거리 | 픽셀 하나가 지상에서 차지하는 거리 |
| Orthorectification | 정사보정 | 지형 기복에 의한 왜곡 제거 |
| Co-registration | 정합 | 두 영상의 격자 일치 |
| Radiometric Normalization | 방사 정규화 | 밝기 특성 보정 |
| Change Detection | 변화 탐지 | 시점 간 차이 추출 |
| Spectral Index | 분광지수 | 밴드 조합 지표 (NDVI 등) |
| SAR | 합성개구레이더 | 구름 투과 관측 센서 |
| Backscatter | 후방산란 | SAR 반사 강도 |
| Cloud Mask | 구름 마스크 | 구름 영역 제외 |
| Circular Error 90 (CE90) | 90% 수평오차 | 위치정확도 표준 표기 |

표 E-2. 원격탐사 용어

## E.3 지형·측지

| 영문 | 한국어 | 설명 |
|---|---|---|
| Digital Elevation Model (DEM) | 수치표고모델 | 표고 격자 자료 |
| Digital Terrain Model (DTM) | 수치지형모델 | 지표면 표고 |
| Digital Surface Model (DSM) | 수치표면모델 | 지물 포함 표고 |
| Viewshed | 가시권 | 특정 지점에서 보이는 범위 |
| Line of Sight (LOS) | 시선 | 두 지점 간 직선 통시 |
| Fresnel Zone | 프레넬 존 | 전파 전달에 필요한 공간 |
| Geoid | 지오이드 | 평균해수면 등가 중력면 |
| Ellipsoid Height | 타원체고 | GPS가 직접 측정하는 높이 |
| Orthometric Height | 표고 | 지오이드 기준 높이 |
| Cost Surface | 비용면 | 통행 난이도 격자 |
| Least Cost Path | 최소비용경로 | 누적비용 최소 경로 |
| Isochrone | 등시선·도달권 | 동일 소요시간 범위 |

표 E-3. 지형·측지 용어

## E.4 개발

| 영문 | 한국어 | 설명 |
|---|---|---|
| Vendoring | 벤더링 | 의존성을 프로젝트에 동봉 |
| Compatibility Layer | 호환 계층 | 버전 차이를 흡수하는 모듈 |
| Headless | 헤드리스 | 화면 없이 실행 |
| Golden Data | 골든 데이터 | 회귀 테스트 기준 결과 |
| Provider | 공급자 | Processing 알고리즘 묶음 |
| Recipe | 레시피 | 재현 가능한 실행 정의 |
| Audit Log | 감사 로그 | 행위 기록 |
| Air-gapped | 폐쇄망 | 외부 네트워크 단절 환경 |
| Feature Gate | 기능 게이트 | 선택 의존성 가용 여부 관리 |

표 E-4. 개발 용어

## E.5 표기 원칙

이 책의 표기 원칙은 다음과 같다.

1. **API 클래스명은 원문 그대로** — `QgsVectorLayer`, `QgsTask`
2. **개념어는 한국어 우선, 첫 등장 시 영문 병기** — 가시권(viewshed)
3. **약어는 첫 등장 시 풀어 쓴다** — 지상표본거리(Ground Sample Distance, GSD)
4. **QGIS 공식 한국어 번역과 다를 경우 이 책의 표기를 따르고 각주로 밝힌다**

---
---

# 마치며

## 이 책에서 만든 것

QGeoINT는 30개 장을 거치며 다음을 갖추게 되었다.

```text
qgeoint/
├── compat.py            버전 분기를 한곳에 격리
├── core/                QGIS GUI 비의존 도메인 로직
│   ├── provenance.py    모든 산출물의 출처 추적
│   ├── recipe.py        재현 가능한 실행 정의
│   ├── audit.py         해시 체인 감사 로그
│   ├── privacy.py       개인정보 자동 점검
│   └── license.py       자료 라이선스 관리
├── ingest/              영상·벡터 인제스트와 카탈로그
├── analysis/            변화탐지·지형·추론
├── fusion/              다중출처 융합과 등급 결합
├── production/          지도·보고서·브리핑 자동 생성
├── processing/          Processing 알고리즘 (재현성)
├── tasks/               QgsTask 백그라운드
├── ext_libs/            폐쇄망 의존성 벤더링
└── tests/               3.6/LTR/4.x 매트릭스 검증
```

## 세 원칙의 회고

책 앞머리에서 제시한 세 원칙이 실제로 어디에 구현되었는지 다시 확인한다.

| 원칙 | 구현 |
|---|---|
| 재현성 | `Recipe.fingerprint()`, Processing 알고리즘화, 골든 데이터 테스트 |
| 추적성 | `Provenance`, `AuditLog` 해시 체인, `feedback.pushInfo()` 이력 |
| 방어가능성 | Admiralty 등급, `counter_evidence` 필수화, 모델 한계 표기, 면책 문구 |

표 F-1. 원칙과 구현의 대응

이 세 가지는 기능이 아니라 **제약**이다.
제약이 있어야 도구가 만들어 내는 결론을 신뢰할 수 있다.

## 다음 단계

이 교재를 마친 뒤 확장할 수 있는 방향이다.

1. **SAR 분석 확장** — 구름과 야간에 영향받지 않는 관측
2. **점군(LiDAR) 통합** — 구조물 변화의 3차원 검증
3. **시계열 이상 탐지** — 단일 쌍 비교가 아닌 장기 추세 이탈 탐지
4. **QGIS Server 연동** — 산출물의 웹 서비스 배포
5. **협업 워크플로** — 다수 분석관의 검토 상태 동기화
6. **모델 성능 모니터링** — 현장 확인 결과를 되먹여 AI 성능 추적

## 마지막 당부

이 책의 기법들은 재난 대응을 빠르게 하고, 허위정보를 걸러 내고,
사회기반시설을 안전하게 관리하는 데 쓰일 수 있다.
같은 기법이 다른 목적에도 쓰일 수 있다는 점을 알고 있다.

도구를 만드는 사람은 도구가 쓰이는 방식에 대해 생각할 책임이 있다.
Chapter 29를 부록이 아니라 본문에 둔 이유가 이것이다.

지도 위의 폴리곤 하나가 실제 장소이고,
그 장소에 사람이 산다는 사실을 잊지 않기를 바란다.

---

**초고 종료**

- 작성일: 2026-08-31
- 구성: 9 Part / 30 Chapter / 부록 A~E
- 다음 산출: HTML 변환, DOCX 변환
