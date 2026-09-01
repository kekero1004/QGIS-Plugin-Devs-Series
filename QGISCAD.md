# QGIS Plugin으로 구현하는 DXF 기반 CAD 시스템

## — DXF 캐드와 동일한 작업 경험을 제공하는 QGIS CAD 플러그인 개발 실전 교재 —

**부제: PyQGIS · ezdxf · Qt로 만드는 Production-Grade CAD Plugin**

- 교재 초안 버전: **v0.1 (Draft)**
- 작성 기준일: 2026-08-29
- 대상 QGIS 버전: **QGIS 3.6+ ~ QGIS 4.x (Qt5 / Qt6 동시 호환)**
- 예상 분량: **250 ~ 300 페이지**
- 주요 기술: Python / PyQGIS / `qgis.PyQt` / ezdxf / OGR DXF Driver / QgsMapTool / QgsSnappingUtils / QgsDxfExport
- 대상 독자:
  - CAD(AutoCAD, ZWCAD, GstarCAD 등) 도면 작업을 QGIS로 이관하려는 토목·측량·도시계획 엔지니어
  - DXF 도면 자동화·검수·변환 도구를 개발하려는 GIS 개발자
  - QGIS 위에서 CAD와 동일한 제도(製圖) 경험을 구현하고자 하는 Python 개발자

---

# 서문 (Preface)

## 왜 QGIS 위에 CAD를 구현하는가

토목·측량·수자원·도시계획 실무의 성과품은 여전히 CAD 도면(DWG/DXF) 중심으로 유통된다. 반면 기초조사·입지선정·환경영향평가 등 공간분석 업무는 GIS 환경에서 수행된다. 이 두 세계 사이에서 실무자는 매일 다음과 같은 반복 작업을 수행한다.

1. GIS 분석 결과를 CAD 도면 좌표계·레이어 체계로 변환하여 납품
2. CAD로 작성된 계획선·종횡단·용지도를 GIS로 역변환하여 분석에 활용
3. 변환 과정에서 발생하는 레이어 매핑 오류, 좌표계 불일치, 폴리선 끊김, 블록·치수 소실의 수작업 보정

이 교재는 이 문제를 정면으로 다룬다. 단순히 "QGIS에서 DXF를 열고 저장하는 방법"이 아니라, **CAD 사용자가 이질감 없이 사용할 수 있는 수준의 제도 환경 — 명령행(Command Line), 객체 스냅(OSNAP), 직교/극좌표 추적, 레이어 관리자, 선종류·색상 체계(ACI), 블록, 치수, 해치 — 를 QGIS 플러그인으로 구현하는 전 과정**을 다룬다.

## 이 책의 목표 산출물

교재를 끝까지 따라 하면 다음 기능을 갖춘 통합 CAD 플러그인 **"QCAD-Bridge"**(가칭)가 완성된다.

| 영역 | 구현 기능 |
|---|---|
| 파일 | DXF 읽기/쓰기(R12~R2018), 레이어·블록·선종류 보존, DWG 우회 전략 |
| 화면 | CAD식 레이어 관리자, 객체 속성바, 명령행 인터페이스, 상태바(직교/스냅/그리드) |
| 작도 | LINE, PLINE, CIRCLE, ARC, RECTANG, POLYGON, XLINE — 절대/상대/극좌표 입력 |
| 스냅 | 끝점·중간점·중심점·교차점·수직점·접점·연장선 OSNAP |
| 편집 | MOVE, COPY, ROTATE, MIRROR, SCALE, OFFSET, TRIM, EXTEND, FILLET, ARRAY, EXPLODE |
| 주석 | TEXT/MTEXT, 선형·정렬·각도 치수(DIM), HATCH, 지시선 |
| 블록 | 블록 정의/삽입/속성(ATTRIB), 동적 배치 |
| 출력 | 축척 기반 도곽 출력, DXF 재내보내기, 레이어 매핑 테이블 |
| 품질 | 3.6~4.x 호환 계층, 자동 테스트, CI/CD, 플러그인 저장소 배포 |

## 버전 정책 — 3.6+와 4.x를 하나의 코드베이스로

이 교재의 모든 예제는 다음 원칙으로 작성한다.

1. import는 항상 `qgis.PyQt`를 사용한다. (`PyQt5`/`PyQt6` 직접 import 금지)
2. Qt enum은 **호환 액세서 함수**(3장에서 구현)를 통해 접근한다.
3. QGIS 3.6에 없는 API를 사용할 때는 기능 감지(feature detection) 후 폴백을 구현한다.
4. 각 장 말미의 **[호환성 노트]** 박스에서 3.6 / 3.44 LTR / 4.x 간 차이를 명시한다.

> **일러두기**
> 본문 코드는 QGIS 4.2 + Qt6에서 검증하고, 호환성 노트의 폴백 코드는 QGIS 3.6.3 / 3.44 LTR에서 별도 검증하는 것을 원칙으로 한다.

---

# 페이지 배분 계획 (250~300p 기준)

| 구성 | 장 | 배정 페이지 |
|---|---|---:|
| 서문·차례·일러두기 | — | 10 |
| Part I. CAD와 QGIS, 그리고 DXF | Ch.1 ~ Ch.3 | 28 |
| Part II. 플러그인 골격과 호환 계층 | Ch.4 ~ Ch.6 | 26 |
| Part III. DXF 데이터 엔진 | Ch.7 ~ Ch.10 | 38 |
| Part IV. CAD 스타일 UI 프레임 | Ch.11 ~ Ch.13 | 30 |
| Part V. 작도 도구와 스냅 시스템 | Ch.14 ~ Ch.17 | 42 |
| Part VI. 편집 도구 | Ch.18 ~ Ch.21 | 40 |
| Part VII. 주석 — 문자·치수·해치·블록 | Ch.22 ~ Ch.24 | 32 |
| Part VIII. 내보내기·출력·상호운용 | Ch.25 ~ Ch.27 | 24 |
| Part IX. 품질·테스트·배포 | Ch.28 ~ Ch.30 | 20 |
| 부록 A ~ E | — | 20 |
| **합계** | | **310 (여유 포함)** |

---

# 전체 목차

## Part I. CAD와 QGIS, 그리고 DXF
- Chapter 1. CAD 제도 환경의 해부 — 무엇을 재현해야 하는가
- Chapter 2. DXF 포맷 완전 분석
- Chapter 3. QGIS의 CAD 기능 현황과 좌표계 전략

## Part II. 플러그인 골격과 호환 계층
- Chapter 4. QCAD-Bridge 플러그인 골격 설계
- Chapter 5. Qt5/Qt6 · QGIS 3.6~4.x 호환 계층 구현
- Chapter 6. 개발 워크플로 — Plugin Reloader, 디버깅, 프로파일

## Part III. DXF 데이터 엔진
- Chapter 7. ezdxf로 DXF 읽기 — ENTITIES와 TABLES
- Chapter 8. DXF → QGIS 데이터 모델 변환 엔진
- Chapter 9. 스타일 재현 — ACI 색상, 선종류, 선굵기, 레이어 상태
- Chapter 10. 대용량 도면과 QgsTask 백그라운드 로딩

## Part IV. CAD 스타일 UI 프레임
- Chapter 11. CAD식 레이어 관리자 Dock
- Chapter 12. 명령행 인터페이스(Command Line) 구현
- Chapter 13. 객체 속성바와 상태바 — 실시간 좌표·모드 표시

## Part V. 작도 도구와 스냅 시스템
- Chapter 14. QgsMapTool 기반 작도 도구 프레임워크
- Chapter 15. 좌표 입력 시스템 — 절대·상대(@)·극좌표(<)
- Chapter 16. 객체 스냅(OSNAP) 엔진
- Chapter 17. LINE부터 ARC까지 — 작도 명령 8종 구현

## Part VI. 편집 도구
- Chapter 18. 선택 시스템과 그립(Grip) 편집
- Chapter 19. 변환 편집 — MOVE, COPY, ROTATE, MIRROR, SCALE
- Chapter 20. 형상 편집 I — OFFSET, ARRAY, EXPLODE
- Chapter 21. 형상 편집 II — TRIM, EXTEND, FILLET

## Part VII. 주석 — 문자·치수·해치·블록
- Chapter 22. TEXT/MTEXT와 한글 폰트(SHX/TTF) 전략
- Chapter 23. 치수(DIMENSION) 시스템
- Chapter 24. HATCH와 블록(BLOCK/INSERT/ATTRIB)

## Part VIII. 내보내기·출력·상호운용
- Chapter 25. QGIS → DXF 내보내기 엔진 (QgsDxfExport + ezdxf 후처리)
- Chapter 26. 레이어 매핑 테이블과 납품 표준(도로·하천·지적 레이어 코드)
- Chapter 27. 도곽·축척 출력과 DWG 상호운용 전략

## Part IX. 품질·테스트·배포
- Chapter 28. 플러그인 아키텍처 리팩터링과 테스트
- Chapter 29. 3.6 ~ 4.x 교차 검증과 pyqgis4-checker
- Chapter 30. 패키징·플러그인 저장소 배포·CI/CD

## 부록
- 부록 A. AutoCAD 명령 ↔ QCAD-Bridge 명령 대응표
- 부록 B. DXF 그룹 코드 요약표
- 부록 C. ACI 256색 ↔ RGB 변환표 (발췌)
- 부록 D. Qt5/Qt6 enum 대응표
- 부록 E. 참고문헌·공식 문서 소스맵

---

# Part I. CAD와 QGIS, 그리고 DXF

---

# Chapter 1. CAD 제도 환경의 해부 — 무엇을 재현해야 하는가

**(배정: 10p | 난이도: ★☆☆ | 핵심)**

## 1.1 학습 목표

- CAD 사용자의 작업 흐름을 기능 단위로 분해한다.
- "CAD와 동일한" 경험을 구성하는 핵심 요소 12가지를 정의한다.
- QGIS 기본 기능으로 이미 되는 것 / 플러그인으로 만들어야 하는 것을 구분한다.

## 1.2 CAD 사용자의 1분 — 작업 흐름 관찰

CAD 실무자가 선 하나를 긋는 1분을 관찰하면, 재현해야 할 요구사항이 모두 드러난다.

```text
[명령행] LINE ↵
[화면]   첫 번째 점 지정:
[입력]   1000,2000 ↵                ← 절대좌표
[화면]   다음 점 지정:
[마우스] 기존 선 끝점 근처 이동      ← 녹색 사각형(끝점 스냅) 표시
[클릭]   스냅된 점 확정
[입력]   @50<90 ↵                    ← 상대 극좌표: 거리 50, 각도 90°
[키]     F8                          ← 직교 모드 토글
[입력]   ESC                         ← 명령 취소
```

여기서 추출되는 요구사항은 다음과 같다.

| # | 요소 | CAD 기능 | 구현 장 |
|---|---|---|---|
| 1 | 명령행 | 키보드 중심 명령/좌표 입력, 프롬프트, 히스토리 | Ch.12 |
| 2 | 좌표 입력 | 절대 `x,y` / 상대 `@dx,dy` / 극좌표 `@d<a` | Ch.15 |
| 3 | 객체 스냅 | 끝점·중간점·중심·교차·수직·접점 | Ch.16 |
| 4 | 모드 토글 | 직교(F8), 스냅(F9), 그리드(F7), 극좌표추적(F10) | Ch.13, 15 |
| 5 | 러버밴드 | 미리보기 선/도형의 실시간 표시 | Ch.14 |
| 6 | 레이어 체계 | 이름·색상(ACI)·선종류·ON/OFF/FREEZE/LOCK | Ch.9, 11 |
| 7 | 객체 속성 | 개별 객체의 색상·선종류·선굵기 (ByLayer/ByBlock) | Ch.9, 13 |
| 8 | 선택/그립 | 창 선택(W)·걸침 선택(C)·그립 편집 | Ch.18 |
| 9 | 편집 명령 | MOVE/COPY/TRIM/EXTEND/OFFSET/FILLET… | Ch.19~21 |
| 10 | 주석 | TEXT/DIM/HATCH/LEADER | Ch.22~24 |
| 11 | 블록 | 재사용 심볼, 속성 블록 | Ch.24 |
| 12 | 파일 왕복 | DXF 열기→편집→저장 시 정보 무손실 | Ch.7~8, 25 |

> **ENGINEERING PRACTICE**
> "CAD와 동일하다"의 실무적 판정 기준은 UI의 외형이 아니라 **① 키 입력 순서가 같고 ② 스냅이 같은 위치에 걸리며 ③ 저장한 DXF를 CAD에서 다시 열었을 때 레이어·색·선종류가 유지되는 것** 세 가지다. 이 교재의 모든 장은 이 세 기준을 검수 항목으로 사용한다.

## 1.3 QGIS가 이미 제공하는 것

QGIS는 3.0 이후 "고급 디지타이징 패널(Advanced Digitizing Panel)"과 스냅 프레임워크를 통해 CAD 기능의 상당 부분을 내장하고 있다. 플러그인 개발 전 반드시 기본 기능의 한계를 파악해야 중복 개발을 피할 수 있다.

| 기능 | QGIS 기본 | 한계 → 플러그인 목표 |
|---|---|---|
| DXF 열기 | OGR 드라이버로 가능 | 지오메트리 타입별 강제 분리, 블록 분해, 스타일 소실 → **구조 보존 로더**(Ch.8) |
| DXF 저장 | `QgsDxfExport` | 치수·해치·블록 미지원 → **ezdxf 후처리**(Ch.25) |
| 스냅 | `QgsSnappingUtils` (꼭짓점/선분/교차) | 중간점·중심점·수직·접점 없음 → **OSNAP 엔진**(Ch.16) |
| 좌표 입력 | 고급 디지타이징 패널 (d/a 입력) | `@dx,dy`, `@d<a` 문법 없음, 명령행 없음 → Ch.12, 15 |
| 직교 모드 | 고급 디지타이징 잠금 | CAD 단축키 체계와 상이 → Ch.13 |
| 레이어 관리 | 레이어 패널 | FREEZE/LOCK/ByLayer 개념 없음 → Ch.11 |

## 1.4 목표 플러그인 QCAD-Bridge의 최종 화면 구성

**[그림 1-1] QCAD-Bridge 최종 UI 구성도 (도판 삽입 예정)**

```text
┌─────────────────────────────────────────────────────────────┐
│ QGIS 메뉴 │ QCAD 리본 툴바 (작도/편집/주석/블록)              │
├───────────┬─────────────────────────────────┬───────────────┤
│ CAD 레이어 │                                 │ 객체 속성바    │
│ 관리자     │        Map Canvas               │ (색/선종류/    │
│ (Dock)    │   (러버밴드·스냅마커·그립)        │  선굵기)      │
│           │                                 │               │
├───────────┴─────────────────────────────────┴───────────────┤
│ 명령행: LINE ▸ 다음 점 지정 또는 [명령취소(U)]:              │
├─────────────────────────────────────────────────────────────┤
│ 1234.56, 7890.12  │ 직교 │ OSNAP │ 그리드 │ 축척 1:1000     │
└─────────────────────────────────────────────────────────────┘
```

## 1.5 요약

- CAD 재현의 본질은 **입력 체계(명령행+좌표+스냅)** 와 **데이터 왕복 무손실(DXF)** 두 축이다.
- QGIS 내장 기능을 최대한 활용하되, 6개 영역(구조 보존 로더, OSNAP 확장, 명령행, CAD 레이어 관리, 편집 명령, DXF 후처리)을 플러그인으로 구현한다.

> **[호환성 노트 | QGIS 3.6 ~ 4.x]**
> 고급 디지타이징 패널과 `QgsSnappingUtils`는 QGIS 3.0부터 존재하므로 3.6에서도 동일한 전략이 유효하다. 단, 스냅 관련 일부 enum(`Qgis.SnappingType` 등)은 3.26에서 재정리되었으므로 Ch.5의 호환 계층을 경유해 사용한다.

---

# Chapter 2. DXF 포맷 완전 분석

**(배정: 10p | 난이도: ★★☆ | 핵심)**

## 2.1 학습 목표

- DXF 파일의 물리 구조(그룹 코드)와 논리 구조(섹션)를 읽을 수 있다.
- ENTITIES / TABLES / BLOCKS 섹션의 역할과 상호 참조를 이해한다.
- 버전(R12~R2018)별 차이와 실무 납품에서 R12 ASCII가 여전히 요구되는 이유를 안다.

## 2.2 DXF의 물리 구조 — 그룹 코드 쌍

DXF는 **(그룹 코드, 값)** 쌍의 연속으로 이루어진 텍스트 포맷이다. 메모장으로 열리는 것이 최대 장점이자, 이 교재가 DWG가 아닌 DXF를 기반으로 삼는 이유다.

```text
  0            ← 그룹 코드 0: 엔티티 시작
LINE           ← 값: 엔티티 타입
  8            ← 그룹 코드 8: 레이어 이름
도로중심선
 10            ← 그룹 코드 10: 시작점 X
1000.0
 20            ← 그룹 코드 20: 시작점 Y
2000.0
 11            ← 그룹 코드 11: 끝점 X
1500.0
 21            ← 그룹 코드 21: 끝점 Y
2000.0
```

주요 그룹 코드는 부록 B에 전체 표로 정리한다. 본문에서 반복 사용하는 코드는 다음과 같다.

| 그룹 코드 | 의미 |
|---:|---|
| 0 | 엔티티/테이블 레코드 타입 |
| 2 | 이름 (블록명, 선종류명 등) |
| 8 | 레이어 이름 |
| 10, 20, 30 | 기준점 X, Y, Z |
| 11, 21, 31 | 두 번째 점 X, Y, Z |
| 40 | 반지름/높이/축척 등 실수 값 |
| 50 | 각도 (도 단위, 반시계) |
| 62 | ACI 색상 번호 (0=ByBlock, 256=ByLayer) |
| 6 | 선종류 이름 |
| 370 | 선굵기 (1/100 mm) |

## 2.3 논리 구조 — 7개 섹션

```text
HEADER    ← 도면 변수 ($INSUNITS 단위, $EXTMIN/$EXTMAX 범위, $DIMSCALE …)
CLASSES   ← (거의 사용 안 함)
TABLES    ← LAYER, LTYPE(선종류), STYLE(문자), DIMSTYLE(치수), APPID …
BLOCKS    ← 블록 정의 (엔티티 묶음)
ENTITIES  ← 모델공간/도면공간의 실제 도형
OBJECTS   ← 딕셔너리, 레이아웃 등 비도형 객체
THUMBNAILIMAGE
```

**핵심은 참조 관계다.** ENTITIES의 LINE은 레이어 이름(코드 8)으로 TABLES→LAYER를 참조하고, INSERT는 블록명(코드 2)으로 BLOCKS를 참조한다. **이 참조 구조를 QGIS 데이터 모델로 옮길 때 어떻게 보존할 것인가가 8장의 주제다.**

**[그림 2-1] DXF 섹션 간 참조 관계도 (도판 삽입 예정)**

## 2.4 주요 엔티티 15종과 QGIS 지오메트리 대응

| DXF 엔티티 | 설명 | QGIS 대응 | 변환 난이도 |
|---|---|---|---|
| POINT | 점 | Point | ★ |
| LINE | 2점 선분 | LineString | ★ |
| LWPOLYLINE | 경량 폴리선(벌지 호 포함) | LineString/CircularString | ★★★ |
| POLYLINE/VERTEX | 구식 폴리선 (R12) | LineString | ★★ |
| CIRCLE | 원 | CircularString(폐합) 또는 세그먼트화 | ★★ |
| ARC | 호 | CircularString 또는 세그먼트화 | ★★ |
| ELLIPSE | 타원(호) | 세그먼트화 | ★★★ |
| SPLINE | NURBS | 세그먼트화 | ★★★ |
| TEXT | 단일행 문자 | Point + 라벨/보조필드 | ★★ |
| MTEXT | 다중행 문자 | Point + 서식 해석 | ★★★ |
| INSERT | 블록 참조 | Point + 블록 테이블 / 전개 | ★★★★ |
| HATCH | 해치 | Polygon + 채움 심볼 | ★★★★ |
| DIMENSION | 치수 (익명 블록 동반) | 전개 or 재생성 | ★★★★★ |
| SOLID | 삼각/사각 채움 | Polygon | ★ |
| 3DFACE | 3D 면 | PolygonZ | ★★ |

> **WARNING**
> LWPOLYLINE의 **벌지(bulge)** 값은 정점 사이 구간이 원호임을 의미한다(코드 42, `bulge = tan(사잇각/4)`). 벌지를 무시하고 직선으로 이으면 곡선부 도로 경계가 다각형으로 깨진다 — DXF 변환기에서 가장 빈번한 품질 사고이며 8.4절에서 정확한 호 복원 알고리즘을 구현한다.

## 2.5 DXF 버전 전략

| 버전 코드 | 명칭 | 실무 의미 |
|---|---|---|
| AC1009 | R12 | 최대 호환. 공공 납품에서 여전히 요구. LWPOLYLINE·MTEXT 없음 |
| AC1015 | 2000 | LWPOLYLINE·MTEXT 안정화. **읽기 기준 하한 권장** |
| AC1021 | 2007 | UTF-8 도입 — **한글 레이어명 처리 분기점** |
| AC1027 | 2013 | 현행 실무 표준 |
| AC1032 | 2018 | 최신. ezdxf 저장 기본값 |

> **TIP**
> 읽기는 R12~2018 전체를 지원하되(ezdxf가 담당), 쓰기는 "2013 기본 + R12 옵션" 두 가지만 제공하는 것이 유지보수 비용 대비 효과가 가장 좋다. R12 쓰기 시 MTEXT→TEXT 분해, LWPOLYLINE→POLYLINE 강등이 필요하다(25장).

## 2.6 인코딩 — 한글 도면의 함정

- R2004 이하: 코드페이지 기반. 한국 도면은 `$DWGCODEPAGE = ANSI_949`(CP949).
- R2007 이상: UTF-8.
- CAD 특유의 `\U+XXXX` 유니코드 이스케이프가 문자 값에 섞여 들어온다.

ezdxf는 이 처리를 대부분 자동화하지만, **CP949로 저장된 R12 도면을 잘못된 인코딩으로 읽으면 레이어 이름이 깨진 채 QGIS에 로드되고, 이후 저장 시 원본 참조가 끊긴다.** 7.6절에서 인코딩 감지·복구 루틴을 작성한다.

## 2.7 요약

- DXF는 그룹 코드 쌍 + 7개 섹션 + 참조 구조로 이뤄진 텍스트 포맷이다.
- 변환 품질의 3대 난제는 **벌지 호, 블록/치수, 한글 인코딩**이며 각각 8장, 24장, 7장에서 해결한다.

> **[호환성 노트 | QGIS 3.6 ~ 4.x]**
> 본 장은 포맷 이론이므로 QGIS 버전과 무관하다. 단 CircularString 렌더링 품질은 3.10 이후 개선되었으므로, 3.6 지원 시 원호를 세그먼트화하는 폴백 경로(8.4절)를 기본 켜두는 것을 권장한다.

---

# Chapter 3. QGIS의 CAD 기능 현황과 좌표계 전략

**(배정: 8p | 난이도: ★★☆)**

## 3.1 학습 목표

- QGIS 버전별(3.6 → 3.44 → 4.x) CAD 관련 기능 변화를 파악한다.
- CAD 로컬 좌표계와 국가 좌표계(EPSG:5186/5187) 간 변환 전략을 수립한다.
- 플러그인의 좌표계 정책(도면 단위 = 지도 단위)을 결정한다.

## 3.2 QGIS 버전별 CAD 관련 기능 연표

| 버전 | CAD 관련 변화 |
|---|---|
| 3.0 (2018) | 고급 디지타이징 패널, 스냅 프레임워크 개편, `QgsDxfExport` 정비 |
| 3.6 (2019) | **본 교재 하한선.** 지오메트리 스냅 안정화 |
| 3.10 | DXF export 개선, 곡선 지오메트리 편집 향상 |
| 3.26 | 스냅 enum 정리(`Qgis.SnappingTypes`), 고급 디지타이징 개선 |
| 3.44 LTR | 현행 LTR — 기업/기관 표준 |
| 4.0+ | **Qt6 전환.** enum 접근 방식 변화, 일부 시그널 시그니처 변경 |

## 3.3 좌표계 전략 — 세 가지 시나리오

CAD 도면은 좌표계 메타데이터가 없다. 플러그인은 열기 시점에 다음 셋 중 하나를 사용자에게 결정하게 해야 한다.

```text
시나리오 A. 도면이 국가 좌표계 수치 그대로 작성됨 (측량 성과 도면)
  → CRS 지정만 하면 됨 (EPSG:5186 중부원점 등)

시나리오 B. 도면이 로컬 원점 (0,0 부근) 기준
  → 2점 대응 이동/회전/축척 변환 (Helmert) 필요 → 27장 구현

시나리오 C. 좌표계 불명
  → "지오레퍼런싱 보류" 상태로 열고, 이후 변환 도구 제공
```

> **ENGINEERING PRACTICE**
> 한국 실무 도면의 대다수는 시나리오 A + 중부원점(EPSG:5186)이다. 플러그인 기본값을 EPSG:5186으로 두되, HEADER의 `$EXTMIN/$EXTMAX` 값이 원점 근처(±10km)이면 시나리오 B로 추정하여 경고를 띄우는 휴리스틱을 8.7절에서 구현한다.

## 3.4 단위 정책

- DXF `$INSUNITS` (70번 헤더): 0=무단위, 4=mm, 6=m.
- 토목 도면은 m 단위(6) 또는 무단위-m 관행이 혼재한다.
- **플러그인 정책: 프로젝트 CRS의 지도 단위(m)와 도면 단위를 1:1로 고정**하고, mm 도면(기계도면 등)은 열기 시 0.001 배율 변환을 제안한다.

## 3.5 요약 및 Part I 마무리

Part I에서 요구사항(1장), 데이터 포맷(2장), 플랫폼 현황·좌표계(3장)를 정리했다. Part II부터 실제 코드를 작성한다.

> **[호환성 노트 | QGIS 3.6 ~ 4.x]**
> `QgsCoordinateTransform` 생성자는 3.x 전 구간에서 `(srcCrs, dstCrs, QgsProject.instance())` 3-인자 형태가 안전하다. 4.x에서도 동일 시그니처가 유지된다.

---

# Part II. 플러그인 골격과 호환 계층

---

# Chapter 4. QCAD-Bridge 플러그인 골격 설계

**(배정: 10p | 난이도: ★★☆ | 핵심)**

## 4.1 학습 목표

- 30개 이상의 CAD 명령을 수용할 수 있는 확장형 플러그인 아키텍처를 설계한다.
- `classFactory → initGui → unload` 생명주기에 맞춰 골격을 구현한다.

## 4.2 디렉터리 구조

일반 예제 플러그인과 달리, CAD 플러그인은 **명령(command)이 계속 늘어나는 구조**다. 처음부터 명령 레지스트리 패턴으로 설계한다.

```text
qcad_bridge/
├── __init__.py                # classFactory
├── metadata.txt
├── plugin.py                  # 생명주기·UI 조립만 담당
├── compat.py                  # ★ Qt5/Qt6·3.6~4.x 호환 계층 (Ch.5)
│
├── core/
│   ├── document.py            # CadDocument — 도면 상태의 단일 진실 공급원
│   ├── cad_layer.py           # CAD 레이어 모델 (색/선종류/freeze/lock)
│   ├── geometry_utils.py      # 벌지·호·교차 계산
│   └── units.py
│
├── io/
│   ├── dxf_reader.py          # ezdxf → CadDocument (Ch.7~8)
│   ├── dxf_writer.py          # CadDocument → DXF (Ch.25)
│   ├── style_mapper.py        # ACI·선종류 → QGIS 심볼 (Ch.9)
│   └── encoding.py
│
├── commands/
│   ├── registry.py            # ★ 명령 레지스트리
│   ├── base_command.py        # 명령 상태기계 베이스
│   ├── draw/                  #   line.py, pline.py, circle.py, arc.py …
│   ├── modify/                #   move.py, trim.py, fillet.py …
│   └── annotate/              #   text.py, dim_linear.py, hatch.py …
│
├── tools/
│   ├── cad_map_tool.py        # 공용 QgsMapTool (Ch.14)
│   ├── snap_engine.py         # OSNAP (Ch.16)
│   └── input_parser.py        # 좌표 문법 파서 (Ch.15)
│
├── ui/
│   ├── layer_manager_dock.py  # Ch.11
│   ├── command_line_dock.py   # Ch.12
│   ├── property_bar.py        # Ch.13
│   └── *.ui
│
├── resources/                 # 아이콘, 선종류 정의(.lin), 해치 패턴(.pat)
├── i18n/
└── tests/
```

## 4.3 최소 골격 코드

`__init__.py`:

```python
def classFactory(iface):
    """QGIS 플러그인 진입점."""
    from .plugin import QcadBridgePlugin
    return QcadBridgePlugin(iface)
```

`metadata.txt` (3.6~4.x 동시 지원 선언):

```ini
[general]
name=QCAD-Bridge
qgisMinimumVersion=3.6
qgisMaximumVersion=4.99
description=DXF-based CAD drafting environment for QGIS
version=0.1.0
supportsQt6=True
author=...
tracker=https://github.com/.../issues
repository=https://github.com/...
```

> **WARNING**
> `supportsQt6=True`는 QGIS 4.x 플러그인 관리자가 설치 가능 여부를 판정하는 필수 키다. 이 키가 없으면 코드가 완벽히 호환되어도 QGIS 4에서 설치가 차단된다.

`plugin.py`:

```python
from qgis.PyQt.QtWidgets import QAction, QToolBar
from qgis.PyQt.QtGui import QIcon
from qgis.PyQt.QtCore import QCoreApplication

from .commands.registry import CommandRegistry
from .core.document import CadDocument


class QcadBridgePlugin:

    def __init__(self, iface):
        self.iface = iface
        self.document = CadDocument()          # 도면 상태
        self.registry = CommandRegistry(self)  # 명령 등록소
        self.toolbar = None
        self.docks = []
        self.actions = []

    def tr(self, text):
        return QCoreApplication.translate("QcadBridge", text)

    def initGui(self):
        self.toolbar = self.iface.addToolBar("QCAD-Bridge")
        self.toolbar.setObjectName("QcadBridgeToolbar")

        self.registry.discover()               # commands/ 하위 자동 수집
        for cmd in self.registry.toolbar_commands():
            action = QAction(QIcon(cmd.icon), cmd.label, self.iface.mainWindow())
            action.triggered.connect(
                lambda checked=False, name=cmd.name: self.registry.run(name))
            self.toolbar.addAction(action)
            self.actions.append(action)

        self._create_docks()                   # Ch.11~13에서 구현

    def unload(self):
        for dock in self.docks:
            self.iface.removeDockWidget(dock)
            dock.deleteLater()
        if self.toolbar:
            self.toolbar.deleteLater()
        self.actions.clear()
        self.registry.shutdown()               # 활성 MapTool 해제 포함
```

## 4.4 명령 레지스트리 패턴

모든 CAD 명령은 동일한 인터페이스를 갖는다. 명령행(Ch.12)·툴바·단축키가 전부 이 레지스트리 하나를 호출한다.

```python
# commands/registry.py
class CommandRegistry:
    def __init__(self, plugin):
        self.plugin = plugin
        self._commands = {}     # name → Command class
        self._aliases = {}      # "l" → "line"
        self.active = None      # 현재 실행 중인 명령

    def register(self, cmd_cls):
        self._commands[cmd_cls.name] = cmd_cls
        for a in cmd_cls.aliases:
            self._aliases[a] = cmd_cls.name

    def run(self, name, *args):
        name = self._aliases.get(name.lower(), name.lower())
        if name not in self._commands:
            raise KeyError(f"알 수 없는 명령: {name}")
        if self.active:
            self.active.cancel()               # CAD 규칙: 새 명령은 기존 명령을 종료
        self.active = self._commands[name](self.plugin)
        self.active.start(*args)
```

```python
# commands/base_command.py
class BaseCommand:
    name = ""          # "line"
    aliases = ()       # ("l",)
    label = ""         # 툴바 표시명
    icon = ""
    in_toolbar = True

    def __init__(self, plugin):
        self.plugin = plugin
        self.iface = plugin.iface
        self.doc = plugin.document

    def start(self, *args): ...
    def on_point(self, pt): ...        # MapTool/명령행에서 좌표 수신
    def on_keyword(self, kw): ...      # "U"(undo), "C"(close) 등 옵션 키워드
    def cancel(self): ...
    def finish(self): ...
```

**[그림 4-1] 명령 실행 시퀀스: 툴바/명령행 → Registry → Command → MapTool → Canvas (도판 삽입 예정)**

## 4.5 CadDocument — 단일 진실 공급원

CAD 플러그인의 상태(현재 레이어, 현재 색상, 로드된 DXF 구조, 블록 테이블, undo 스택)는 반드시 한 곳에 모은다. UI와 명령은 이 객체만 바라본다.

```python
# core/document.py
from qgis.PyQt.QtCore import QObject, pyqtSignal

class CadDocument(QObject):
    layersChanged = pyqtSignal()
    currentLayerChanged = pyqtSignal(str)

    def __init__(self):
        super().__init__()
        self.source_path = None        # 원본 DXF 경로
        self.dxf_version = None
        self.cad_layers = {}           # name → CadLayer (Ch.9)
        self.blocks = {}               # name → BlockDefinition (Ch.24)
        self.current_layer = "0"
        self.current_color = 256       # ByLayer
        self.current_ltype = "ByLayer"
        self.qgis_layers = {}          # 지오메트리 타입별 QgsVectorLayer (Ch.8)
```

## 4.6 요약

- 명령 레지스트리 + BaseCommand + CadDocument 세 축이 이후 26개 장 전체의 뼈대다.

> **[호환성 노트 | QGIS 3.6 ~ 4.x]**
> `pyqtSignal`은 `qgis.PyQt.QtCore`에서 import하면 Qt5(PyQt5)와 Qt6(PyQt6) 모두에서 동작한다. `QAction`의 모듈 위치가 Qt6에서 QtWidgets→QtGui로 이동했지만, `qgis.PyQt.QtWidgets`는 양쪽 모두에서 QAction을 재수출하므로 위 코드는 수정 없이 호환된다.

---

# Chapter 5. Qt5/Qt6 · QGIS 3.6~4.x 호환 계층 구현

**(배정: 9p | 난이도: ★★★ | 핵심)**

## 5.1 학습 목표

- 단일 코드베이스로 Qt5/Qt6을 지원하는 `compat.py` 모듈을 완성한다.
- 버전 감지·기능 감지 유틸리티를 구현한다.

## 5.2 무엇이 깨지는가 — 4대 파손 지점

| 파손 지점 | Qt5 (QGIS 3.x) | Qt6 (QGIS 4.x) |
|---|---|---|
| enum 접근 | `Qt.LeftButton` | `Qt.MouseButton.LeftButton` (스코프드) |
| QAction 위치 | QtWidgets | QtGui |
| exec 메서드 | `exec_()` | `exec()` |
| 일부 API | `QRegExp` | `QRegularExpression`만 존재 |

다행히 PyQt6도 QGIS 빌드에서는 비스코프드 enum 접근을 상당 부분 허용하지만, **버전에 따라 동작이 달라지는 코드를 남기지 않는 것**이 상용 플러그인의 원칙이다.

## 5.3 compat.py 전체 구현

```python
# compat.py — QCAD-Bridge 호환 계층
"""QGIS 3.6+ (Qt5) / QGIS 4.x (Qt6) 단일 코드베이스 지원 모듈.

사용 규칙:
1) 모든 Qt import는 qgis.PyQt 경유
2) 문제성 enum은 이 모듈의 상수 사용
3) 버전 분기는 QGIS_3X / HAS_* 플래그 사용
"""
from qgis.core import Qgis
from qgis.PyQt.QtCore import Qt, QT_VERSION_STR

# ── 버전 플래그 ──────────────────────────────────────────
QT6 = int(QT_VERSION_STR.split(".")[0]) >= 6
QGIS_VER = Qgis.versionInt() if hasattr(Qgis, "versionInt") else int(
    Qgis.QGIS_VERSION_INT)             # 3.6에는 versionInt()가 없음 → 상수 폴백
QGIS_3X = QGIS_VER < 40000


def _enum(scoped_owner, name, legacy_owner=None):
    """Qt6 스코프드 enum과 Qt5 플랫 enum을 동시에 지원하는 접근자."""
    owner = legacy_owner or scoped_owner
    if QT6:
        return getattr(scoped_owner, name)
    return getattr(owner, name)

# ── 자주 쓰는 enum 상수 ──────────────────────────────────
LEFT_BUTTON   = _enum(Qt.MouseButton, "LeftButton", Qt) if QT6 else Qt.LeftButton
RIGHT_BUTTON  = _enum(Qt.MouseButton, "RightButton", Qt) if QT6 else Qt.RightButton
KEY_ESCAPE    = _enum(Qt.Key, "Key_Escape", Qt) if QT6 else Qt.Key_Escape
KEY_RETURN    = _enum(Qt.Key, "Key_Return", Qt) if QT6 else Qt.Key_Return
KEY_F8        = _enum(Qt.Key, "Key_F8", Qt) if QT6 else Qt.Key_F8
CROSS_CURSOR  = _enum(Qt.CursorShape, "CrossCursor", Qt) if QT6 else Qt.CrossCursor
SOLID_LINE    = _enum(Qt.PenStyle, "SolidLine", Qt) if QT6 else Qt.SolidLine
DASH_LINE     = _enum(Qt.PenStyle, "DashLine", Qt) if QT6 else Qt.DashLine


def exec_dialog(dlg):
    """Qt5 exec_() / Qt6 exec() 통합."""
    fn = getattr(dlg, "exec", None) or getattr(dlg, "exec_")
    return fn()


def has_api(obj, attr):
    """기능 감지: 3.6에 없는 API를 안전하게 확인."""
    return hasattr(obj, attr)
```

> **TIP**
> enum을 나열식으로 상수화하는 이유는 두 가지다. ① IDE 자동완성이 유지된다. ② 이후 pyqgis4-checker(29장)를 돌릴 때 위반 지점이 compat.py 한 파일로 수렴하여 심사가 쉬워진다.

## 5.4 스냅 enum 호환 — 3.6 / 3.26+ / 4.x

스냅 타입 enum은 세 시대를 거쳤다. 스냅 엔진(Ch.16)이 사용할 통합 함수를 미리 만든다.

```python
# compat.py 계속
from qgis.core import QgsSnappingConfig

def vertex_segment_flags():
    """VertexFlag|SegmentFlag를 버전에 맞게 반환."""
    if hasattr(Qgis, "SnappingType"):                 # 3.26+ / 4.x
        return Qgis.SnappingTypes(
            Qgis.SnappingType.Vertex | Qgis.SnappingType.Segment)
    # 3.6 ~ 3.24
    return QgsSnappingConfig.VertexAndSegment
```

## 5.5 요약

- 호환성 문제는 "발견 즉시 compat.py로 격리"가 유일한 규칙이다.
- 이후 모든 장의 코드는 compat 상수를 사용하며, 별도 언급 없이 3.6~4.x에서 동작한다.

> **[호환성 노트]**
> 본 장 자체가 호환성 장이다. 3.6에서 `Qgis.versionInt()` 부재, 3.26 이전 스냅 enum, 4.x 스코프드 enum 세 가지가 실측 기준 가장 빈번한 파손 지점이다.

---

# Chapter 6. 개발 워크플로 — Plugin Reloader, 디버깅, 프로파일

**(배정: 7p | 난이도: ★☆☆)**

## 6.1 학습 목표

- 코드 수정 → 재로드 → 검증 사이클을 10초 이내로 단축한다.
- QGIS 3.6과 4.x를 동시에 띄워 교차 검증하는 멀티 프로파일 환경을 구성한다.

## 6.2 개발 사이클 구성

```text
VS Code (플러그인 폴더 직접 편집)
   ↓ 저장
QGIS ① 4.2 (기본 검증)  +  QGIS ② 3.44 LTR  +  QGIS ③ 3.6 (호환 하한 검증)
   ↓
Plugin Reloader (Ctrl+F5 지정)
   ↓
명령행에서 명령 실행 → Canvas 확인 → QgsMessageLog 확인
```

- 플러그인 폴더를 QGIS 프로파일의 `python/plugins/qcad_bridge`에 **심볼릭 링크**로 연결하면 세 QGIS가 동일 소스를 공유한다.

```powershell
# Windows (관리자 PowerShell)
New-Item -ItemType SymbolicLink `
  -Path "$env:APPDATA\QGIS\QGIS3\profiles\dev\python\plugins\qcad_bridge" `
  -Target "C:\dev\qcad_bridge\qcad_bridge"
```

## 6.3 로깅 표준

```python
from qgis.core import QgsMessageLog, Qgis

def log(msg, level=Qgis.Info):
    QgsMessageLog.logMessage(str(msg), "QCAD-Bridge", level)
```

> **WARNING**
> MapTool의 `canvasMoveEvent` 안에서 `print()`/log를 호출하면 마우스 이동마다 I/O가 발생해 캔버스가 버벅인다. 이동 이벤트 내부 로깅은 개발 중에도 조건부 플래그로 감싼다.

## 6.4 요약 및 Part II 마무리

빈 플러그인 → 골격 → 호환 계층 → 개발 사이클까지 완료했다. Part III에서 DXF 데이터를 실제로 읽는다.

> **[호환성 노트]**
> Plugin Reloader는 QGIS 4용 릴리스가 별도 제공된다. 3.x/4.x 프로파일에 각각 맞는 버전을 설치한다.

---

# Part III. DXF 데이터 엔진

---

# Chapter 7. ezdxf로 DXF 읽기 — ENTITIES와 TABLES

**(배정: 10p | 난이도: ★★☆ | 핵심)**

## 7.1 학습 목표

- ezdxf 설치 전략(QGIS 번들 Python 환경)과 API 기본 구조를 익힌다.
- 레이어 테이블·엔티티를 순회하며 CadDocument에 적재한다.
- CP949 R12 도면의 인코딩 문제를 해결한다.

## 7.2 왜 OGR만으로 부족한가

QGIS는 OGR DXF 드라이버로 DXF를 열 수 있지만, CAD 플러그인 목적에는 다음 한계가 있다.

| 항목 | OGR DXF | ezdxf |
|---|---|---|
| 레이어 테이블(색/선종류/상태) | 속성으로 평탄화 | 원본 구조 그대로 |
| 블록 정의 접근 | 전개 결과만 | 정의·참조 분리 접근 |
| DIMENSION 원본 파라미터 | 불가 | 가능 |
| 쓰기(왕복) | 제한적 | R12~2018 완전 |
| HEADER 변수 | 제한적 | 완전 |

**전략: 읽기·쓰기의 구조 해석은 ezdxf, QGIS 렌더링·공간 인덱스·스냅은 QgsVectorLayer.** 두 세계를 잇는 변환기가 8장이다.

## 7.3 ezdxf 설치 — QGIS 번들 Python에

```python
# QGIS Python Console에서 1회 실행 (또는 플러그인 최초 구동 시 자동 설치 루틴)
import subprocess, sys
subprocess.check_call([sys.executable, "-m", "pip", "install", "ezdxf"])
```

> **WARNING**
> Windows OSGeo4W 환경에서는 `sys.executable`이 qgis-bin.exe를 가리키는 경우가 있다. 이때는 `os.path.join(sys.prefix, "python.exe")` 경로를 사용하거나 OSGeo4W Shell의 `py3_env` 후 pip를 실행한다. 플러그인 배포 시에는 ezdxf를 플러그인 내부 `ext_libs/`에 동봉(vendoring)하고 `sys.path`에 추가하는 방식이 기관 PC(오프라인)에서 가장 안전하다 — 30장에서 자동화한다.

## 7.4 문서 열기와 기본 순회

```python
# io/dxf_reader.py
import ezdxf
from ezdxf import recover

def open_dxf(path):
    """손상 도면까지 고려한 안전한 열기."""
    try:
        doc = ezdxf.readfile(path)          # 정상 경로
        auditor = doc.audit()
    except ezdxf.DXFStructureError:
        doc, auditor = recover.readfile(path)   # 복구 모드
    if auditor.has_errors:
        # 오류 요약을 QgsMessageLog로 (치명 오류 개수만)
        pass
    return doc

doc = open_dxf("도로계획평면도.dxf")
msp = doc.modelspace()

print(doc.dxfversion)          # 'AC1027'
print(len(msp))                # 엔티티 수
for e in msp.query("LINE LWPOLYLINE"):
    print(e.dxftype(), e.dxf.layer)
```

## 7.5 레이어 테이블 읽기 → CadLayer

```python
from ..core.cad_layer import CadLayer

def read_layer_table(doc):
    layers = {}
    for rec in doc.layers:
        layers[rec.dxf.name] = CadLayer(
            name=rec.dxf.name,
            color=rec.color,                    # ACI (음수면 OFF 상태)
            linetype=rec.dxf.linetype,
            lineweight=rec.dxf.lineweight,      # -3=Default
            is_off=rec.is_off(),
            is_frozen=rec.is_frozen(),
            is_locked=rec.is_locked(),
        )
    return layers
```

CAD의 레이어 상태 3종은 QGIS 대응을 다음과 같이 정의한다(11장에서 UI로 노출).

| CAD 상태 | 의미 | QGIS 구현 |
|---|---|---|
| OFF | 안 보임, 재생성 대상 유지 | 규칙 기반 렌더러 가시성 off |
| FREEZE | 안 보임 + 연산 제외 | 가시성 off + 스냅/선택 필터 제외 |
| LOCK | 보이지만 편집 금지 | 선택 필터에서 제외(읽기전용) |

## 7.6 인코딩 복구 루틴

```python
def detect_and_reopen(path):
    """CP949 R12 도면 깨짐 감지 후 재열기."""
    doc = open_dxf(path)
    if doc.dxfversion <= "AC1015":                  # R2000 이하
        cp = doc.header.get("$DWGCODEPAGE", "")
        if cp.upper() in ("ANSI_949", "ANSI949"):
            doc = ezdxf.readfile(path, encoding="cp949")
    return doc
```

깨진 레이어명("µµ·Î" 등)이 이미 로드된 뒤에는 복구가 어려우므로 **열기 시점 감지가 유일한 기회**다. `\U+XXXX` 이스케이프는 ezdxf가 자동 복원한다.

## 7.7 요약

- ezdxf로 구조(레이어·블록·헤더)를, 이후 장에서 QGIS로 지오메트리를 담당시키는 이원 구조를 확정했다.

> **[호환성 노트]**
> ezdxf는 Python 3.9+를 요구한다. QGIS 3.6 번들 Python은 3.7이므로 **3.6 지원 빌드에서는 ezdxf 0.17.x 계열을 동봉**해야 한다. 30장의 패키징 스크립트가 QGIS 버전에 따라 ext_libs를 분기 선택하도록 구현한다. (이 제약은 3.6 지원의 실질적 최대 비용이므로, 출간 시점에 하한을 3.16 LTR로 올리는 선택지를 편집 회의에서 재논의할 것 — 집필 메모)

---

# Chapter 8. DXF → QGIS 데이터 모델 변환 엔진

**(배정: 12p | 난이도: ★★★★ | 핵심)**

## 8.1 학습 목표

- CAD 엔티티를 QGIS 메모리 레이어로 무손실 변환하는 엔진을 구현한다.
- 벌지(bulge) → 원호 복원 알고리즘을 구현한다.
- 원본 재작성(왕복)을 위한 엔티티 핸들 보존 체계를 설계한다.

## 8.2 저장 모델 설계 — 지오메트리 타입별 5개 레이어

QGIS 벡터 레이어는 단일 지오메트리 타입만 담는다. CAD 도면 전체를 다음 5개 메모리 레이어로 수용한다.

```text
QCAD|points      (PointZ)         ← POINT, TEXT 삽입점, INSERT 기준점
QCAD|lines       (CompoundCurveZ) ← LINE, LWPOLYLINE, ARC, CIRCLE, SPLINE…
QCAD|polygons    (CurvePolygonZ)  ← HATCH 경계, SOLID, 폐합 폴리선(옵션)
QCAD|texts       (PointZ)         ← TEXT/MTEXT (회전·높이 필드 보유)
QCAD|inserts     (PointZ)         ← 블록 참조 (24장에서 렌더링)
```

공통 속성 스키마 — **왕복 무손실의 핵심**:

| 필드 | 타입 | 설명 |
|---|---|---|
| handle | string | DXF 엔티티 핸들 (원본 추적 키) |
| etype | string | LINE / LWPOLYLINE / … |
| layer | string | CAD 레이어명 |
| aci | int | 색 (256=ByLayer) |
| ltype | string | 선종류 |
| lweight | int | 선굵기 |
| extra | string(JSON) | 타입별 잔여 파라미터 (벌지 목록, 문자 서식 등) |

> **ENGINEERING PRACTICE**
> `handle` 필드가 있으면 "QGIS에서 수정된 피처만 골라 원본 DXF의 해당 엔티티를 교체"하는 **차등 저장**(25장)이 가능해진다. 전체 재작성 방식보다 원본 보존성이 압도적으로 높다.

## 8.3 변환 엔진 골격

```python
# io/dxf_reader.py 계속
from qgis.core import (QgsVectorLayer, QgsFeature, QgsGeometry,
                       QgsField, QgsFields, QgsPointXY, QgsPoint,
                       QgsCircularString, QgsCompoundCurve, QgsLineString)
from qgis.PyQt.QtCore import QVariant
import json, math

COMMON_FIELDS = [
    ("handle",  QVariant.String), ("etype", QVariant.String),
    ("layer",   QVariant.String), ("aci",   QVariant.Int),
    ("ltype",   QVariant.String), ("lweight", QVariant.Int),
    ("extra",   QVariant.String),
]

class DxfToQgisConverter:
    def __init__(self, doc, crs_authid="EPSG:5186"):
        self.doc = doc
        self.crs = crs_authid
        self.dispatch = {
            "LINE": self._conv_line,
            "LWPOLYLINE": self._conv_lwpolyline,
            "POLYLINE": self._conv_polyline,
            "CIRCLE": self._conv_circle,
            "ARC": self._conv_arc,
            "POINT": self._conv_point,
            "TEXT": self._conv_text,
            "MTEXT": self._conv_text,
            "INSERT": self._conv_insert,
            "HATCH": self._conv_hatch,
            "SPLINE": self._conv_spline,
        }

    def _attrs(self, e, extra=None):
        dxf = e.dxf
        return [e.dxf.handle, e.dxftype(), dxf.layer,
                getattr(dxf, "color", 256),
                getattr(dxf, "linetype", "ByLayer"),
                getattr(dxf, "lineweight", -1),
                json.dumps(extra or {}, ensure_ascii=False)]

    def _conv_line(self, e):
        s, t = e.dxf.start, e.dxf.end
        g = QgsGeometry(QgsLineString(
            [QgsPoint(s.x, s.y, s.z), QgsPoint(t.x, t.y, t.z)]))
        return "lines", g, self._attrs(e)
```

## 8.4 벌지 → 원호 복원 (본 장의 핵심 알고리즘)

벌지 `b = tan(θ/4)` (θ: 호의 중심각, 부호=방향)이다. 두 정점 P1, P2와 벌지로부터 **호 위의 중간 통과점**을 구하면 `QgsCircularString(P1, Pm, P2)`으로 정확한 원호를 만들 수 있다.

수식 유도 (본문에서 그림과 함께 2쪽 상세 전개 — 초안에서는 결과만):

```text
현의 길이       c = |P2 - P1|
중심각          θ = 4·atan(b)
반지름          r = c / (2·sin(θ/2))
새그(sagitta)   s = b · c / 2        ← 현 중점에서 호까지 수직 거리
호 중간점 Pm    = 현 중점 + (현의 수직 단위벡터) × s
```

```python
    @staticmethod
    def _bulge_mid(p1, p2, bulge):
        """벌지 구간의 호 중간 통과점."""
        mx, my = (p1[0]+p2[0])/2.0, (p1[1]+p2[1])/2.0
        dx, dy = p2[0]-p1[0], p2[1]-p1[1]
        c = math.hypot(dx, dy)
        if c == 0:
            return None
        s = bulge * c / 2.0
        # 진행방향 좌측 수직 단위벡터 × s (bulge 부호가 방향을 결정)
        return (mx - dy/c * s, my + dx/c * s)

    def _conv_lwpolyline(self, e):
        pts = list(e.get_points("xyb"))          # (x, y, bulge)
        curve = QgsCompoundCurve()
        bulges = []
        n = len(pts)
        segs = n if e.closed else n - 1
        for i in range(segs):
            x1, y1, b = pts[i]
            x2, y2, _ = pts[(i+1) % n]
            p1, p2 = QgsPoint(x1, y1), QgsPoint(x2, y2)
            if abs(b) > 1e-12:
                m = self._bulge_mid((x1, y1), (x2, y2), b)
                curve.addCurve(QgsCircularString.fromTwoPointsAndCenter(
                    p1, p2, QgsPoint(*m)) if False else
                    QgsCircularString(p1, QgsPoint(*m), p2))
                bulges.append(b)
            else:
                curve.addCurve(QgsLineString([p1, p2]))
                bulges.append(0.0)
        extra = {"closed": e.closed, "bulges": bulges,
                 "const_width": e.dxf.const_width}
        return "lines", QgsGeometry(curve), self._attrs(e, extra)
```

**[그림 8-1] 벌지 기하: 현·새그·중심각 관계 (도판 삽입 예정)**
**[그림 8-2] 벌지 무시 변환(좌) vs 원호 복원 변환(우)의 곡선부 비교 (도판 삽입 예정)**

## 8.5 CIRCLE / ARC

```python
    def _conv_circle(self, e):
        c, r = e.dxf.center, e.dxf.radius
        # 반원 2개의 CircularString으로 폐합 원 구성
        p0 = QgsPoint(c.x + r, c.y); p180 = QgsPoint(c.x - r, c.y)
        top = QgsPoint(c.x, c.y + r); bot = QgsPoint(c.x, c.y - r)
        curve = QgsCompoundCurve()
        curve.addCurve(QgsCircularString(p0, top, p180))
        curve.addCurve(QgsCircularString(p180, bot, p0))
        return "lines", QgsGeometry(curve), self._attrs(
            e, {"center": [c.x, c.y], "radius": r})

    def _conv_arc(self, e):
        c, r = e.dxf.center, e.dxf.radius
        a1 = math.radians(e.dxf.start_angle)
        a2 = math.radians(e.dxf.end_angle)
        if a2 <= a1:
            a2 += 2*math.pi
        am = (a1 + a2) / 2.0
        pt = lambda a: QgsPoint(c.x + r*math.cos(a), c.y + r*math.sin(a))
        g = QgsGeometry(QgsCircularString(pt(a1), pt(am), pt(a2)))
        return "lines", g, self._attrs(
            e, {"center": [c.x, c.y], "radius": r,
                "a1": e.dxf.start_angle, "a2": e.dxf.end_angle})
```

## 8.6 메모리 레이어 생성과 일괄 적재

```python
    def build_layers(self):
        sinks = {}
        defs = {"points": "PointZ", "lines": "CompoundCurveZ",
                "polygons": "CurvePolygonZ", "texts": "PointZ",
                "inserts": "PointZ"}
        for key, wkb in defs.items():
            vl = QgsVectorLayer(f"{wkb}?crs={self.crs}", f"QCAD|{key}", "memory")
            pr = vl.dataProvider()
            pr.addAttributes([QgsField(n, t) for n, t in COMMON_FIELDS])
            vl.updateFields()
            sinks[key] = (vl, pr, [])

        for e in self.doc.modelspace():
            fn = self.dispatch.get(e.dxftype())
            if not fn:
                continue                        # 미지원 타입은 로그만
            try:
                key, geom, attrs = fn(e)
            except Exception as ex:
                continue                        # 개별 엔티티 오류 격리
            f = QgsFeature()
            f.setGeometry(geom); f.setAttributes(attrs)
            sinks[key][2].append(f)

        for key, (vl, pr, feats) in sinks.items():
            pr.addFeatures(feats)               # 일괄 추가 (개별 추가 대비 수십 배 빠름)
            vl.updateExtents()
        return {k: v[0] for k, v in sinks.items()}
```

## 8.7 좌표계 휴리스틱 (3.3절 정책의 구현)

```python
def guess_local_origin(doc, threshold=10_000):
    ext_min = doc.header.get("$EXTMIN", (0, 0, 0))
    return abs(ext_min[0]) < threshold and abs(ext_min[1]) < threshold
```

## 8.8 요약

- 5-레이어 모델 + handle 보존 + 벌지 복원으로 "구조 보존 로더"의 골격이 완성됐다.
- TEXT/INSERT/HATCH의 상세 변환은 22·24장에서 완성한다.

> **[호환성 노트]**
> `QgsCircularString`·`QgsCompoundCurve`는 3.0부터 제공되어 3.6에서 사용 가능하다. 단 3.6의 곡선 렌더링은 세그먼트 근사 품질이 낮으므로, `settings`에서 "곡선 세그먼트화 각도"를 노출하고 3.x에서는 기본값을 더 조밀하게 둔다. `QVariant` 기반 `QgsField` 생성자는 3.38부터 deprecated 경고가 있으나 4.x까지 동작한다 — 정식 원고에서는 `QMetaType` 분기를 compat.py에 추가한다(집필 메모).

---

# Chapter 9. 스타일 재현 — ACI 색상, 선종류, 선굵기, 레이어 상태

**(배정: 9p | 난이도: ★★★)**

## 9.1 학습 목표

- ACI(AutoCAD Color Index) → RGB 변환과 ByLayer/ByBlock 해석을 구현한다.
- DXF 선종류(LTYPE) 대시 패턴을 QGIS 심볼로 재현한다.
- 데이터 정의 재정의(data-defined override)로 **레이어 1개 = 렌더러 1개**로 전체 스타일을 처리한다.

## 9.2 ACI 색 해석 규칙

객체의 실제 색은 3단 폴백으로 결정된다.

```text
aci == 256 (ByLayer) → 레이어 테이블의 색
aci == 0   (ByBlock) → 블록 삽입 객체의 색 (없으면 흰색)
1 ≤ aci ≤ 255        → ACI 팔레트 색
```

```python
# io/style_mapper.py
from ezdxf.colors import aci2rgb   # 표준 ACI 팔레트

def resolve_rgb(aci, layer, cad_layers, insert_aci=None):
    if aci == 256:
        aci = cad_layers[layer].color if layer in cad_layers else 7
    elif aci == 0:
        aci = insert_aci or 7
    if aci == 7:
        return (0, 0, 0)     # 배경 흰색 기준: CAD 관행상 7번은 흑/백 반전색
    return aci2rgb(abs(aci))
```

> **TIP**
> CAD 화면은 검정 배경이 관행이라 7번(white)이 흰색으로 보이지만, QGIS 흰 배경에서는 검정으로 그려야 "종이 도면과 같은" 모습이 된다. 플러그인 설정에 "배경: 백지/흑지" 토글을 두고 7번 색을 반전한다.

## 9.3 데이터 정의 재정의 렌더러

피처마다 심볼을 만들면 10만 엔티티 도면에서 파산한다. **단일 심볼 + 색상/폭 표현식 재정의**가 정답이다.

```python
from qgis.core import (QgsSymbol, QgsProperty, QgsSingleSymbolRenderer,
                       QgsSymbolLayer)

ACI_CASE_EXPR = """
CASE
  WHEN "aci" = 256 THEN map_get(@qcad_layer_colors, "layer")
  WHEN "aci" = 7   THEN '0,0,0'
  ELSE map_get(@qcad_aci_palette, to_string("aci"))
END
"""

def apply_cad_renderer(vlayer):
    sym = QgsSymbol.defaultSymbol(vlayer.geometryType())
    sl = sym.symbolLayer(0)
    sl.setDataDefinedProperty(
        QgsSymbolLayer.PropertyStrokeColor,
        QgsProperty.fromExpression(ACI_CASE_EXPR))
    sl.setDataDefinedProperty(
        QgsSymbolLayer.PropertyStrokeWidth,
        QgsProperty.fromExpression(
            'CASE WHEN "lweight" > 0 THEN "lweight"/100.0 ELSE 0.18 END'))
    vlayer.setRenderer(QgsSingleSymbolRenderer(sym))
```

`@qcad_layer_colors`, `@qcad_aci_palette`는 프로젝트 사용자 변수(expression variable)로 주입한다 — 레이어 색을 바꾸면 변수만 갱신하면 되므로 11장의 레이어 관리자와 자연스럽게 연동된다.

## 9.4 선종류(LTYPE) 재현

DXF LTYPE는 대시-공백-점 패턴을 도면 단위 길이로 정의한다(예: CENTER = `1.25, -0.25, 0.25, -0.25`).

```python
from qgis.core import QgsSimpleLineSymbolLayer, QgsUnitTypes

def linetype_to_dash(ltype_record):
    """DXF LTYPE 패턴 → QGIS customDashVector (지도 단위)."""
    dashes = []
    for elem in ltype_record.pattern_tags.get_style_elements():
        dashes.append(abs(elem.length))          # QGIS는 [dash, gap, ...] 절대값 교대
    return dashes or None
```

렌더러에는 선종류 이름별 대시 벡터를 표현식 함수(커스텀 QGIS expression function)로 등록해 해석시킨다 — 정식 원고에서 3쪽 분량으로 상세화(집필 메모).

> **WARNING**
> QGIS 대시 패턴 단위를 mm(기본)로 두면 축척과 무관하게 화면 크기가 고정되어 CAD와 다르게 보인다. `setCustomDashPatternUnit(QgsUnitTypes.RenderMapUnits)`로 **지도 단위**를 지정해야 CAD의 LTSCALE 개념과 일치한다.

## 9.5 레이어 상태(OFF/FREEZE/LOCK) 구현

- 규칙 기반 렌더러 대신 **레이어별 부분집합 필터가 아닌**, `layer NOT IN (@qcad_hidden_layers)` 형식의 피처 필터 표현식을 5개 QGIS 레이어에 일괄 적용한다.
- LOCK은 렌더링에 영향이 없고, 선택 도구(18장)와 스냅 엔진(16장)이 `@qcad_locked_layers`를 조회해 대상에서 제외한다.

## 9.6 요약

- "심볼 폭발" 없이 표현식 재정의만으로 CAD 스타일 3요소(색/선종류/굵기)를 재현했다.

> **[호환성 노트]**
> `QgsSymbolLayer.PropertyStrokeColor` 등 심볼 프로퍼티 enum은 3.30에서 `Qgis.SymbolLayerProperty`로 이관 예고되었다. compat.py에 `SYMPROP_STROKE_COLOR` 상수를 추가해 흡수한다. 표현식 함수 등록(`qgsfunction` 데코레이터)은 3.6~4.x 동일하다.

---

# Chapter 10. 대용량 도면과 QgsTask 백그라운드 로딩

**(배정: 7p | 난이도: ★★★)**

## 10.1 학습 목표

- 10만~100만 엔티티 도면을 UI 프리즈 없이 로드한다.
- QgsTask에서 지켜야 할 스레드 규칙을 체득한다.

## 10.2 스레드 규칙 — 절대 원칙 3개

1. 백그라운드 스레드에서 **QgsVectorLayer를 만들지도, 만지지도 않는다.**
2. 백그라운드에서는 순수 데이터(QgsFeature 리스트, dict)만 생성한다.
3. 레이어 생성·추가·렌더러 적용은 `finished()` 콜백(메인 스레드)에서 수행한다.

## 10.3 구현

```python
from qgis.core import QgsTask, QgsApplication

class DxfLoadTask(QgsTask):
    def __init__(self, path, crs, on_done):
        super().__init__(f"DXF 로드: {path}", QgsTask.CanCancel)
        self.path, self.crs, self.on_done = path, crs, on_done
        self.payload = None
        self.error = None

    def run(self):
        try:
            doc = detect_and_reopen(self.path)             # Ch.7
            conv = DxfToQgisConverter(doc, self.crs)       # Ch.8
            total = len(doc.modelspace())
            feats = {k: [] for k in ("points","lines","polygons","texts","inserts")}
            for i, e in enumerate(doc.modelspace()):
                if self.isCanceled():
                    return False
                fn = conv.dispatch.get(e.dxftype())
                if fn:
                    key, geom, attrs = fn(e)
                    f = QgsFeature(); f.setGeometry(geom); f.setAttributes(attrs)
                    feats[key].append(f)
                if i % 2000 == 0:
                    self.setProgress(i * 100.0 / max(total, 1))
            self.payload = (doc, feats, read_layer_table(doc))
            return True
        except Exception as ex:
            self.error = str(ex)
            return False

    def finished(self, ok):
        # ★ 메인 스레드 — 여기서만 레이어 생성/추가
        self.on_done(ok, self.payload, self.error)

# 실행
task = DxfLoadTask(path, "EPSG:5186", plugin.on_dxf_loaded)
QgsApplication.taskManager().addTask(task)
```

> **WARNING**
> `QgsTask`를 지역 변수로만 잡으면 GC가 태스크를 회수해 조용히 중단된다. 반드시 플러그인 멤버(`self._tasks.append(task)`)로 참조를 유지한다.

## 10.4 성능 수치 목표와 측정 (집필 시 실측표 삽입)

| 도면 규모 | 목표 로드 시간 | 비고 |
|---|---|---|
| 1만 엔티티 | < 1 s | 동기 로드 허용 |
| 10만 | < 8 s | QgsTask 필수 |
| 100만 | < 90 s | 진행률 + 취소 + 부분 로드 옵션 |

## 10.5 요약 및 Part III 마무리

DXF → CadDocument → QGIS 5-레이어 + CAD 렌더러 + 백그라운드 로딩까지, "열기" 파이프라인이 완성됐다. Part IV에서 CAD다운 화면을 만든다.

> **[호환성 노트]**
> `QgsTask.setProgress`/`isCanceled`는 3.6~4.x 동일. 3.6에서는 `taskCompleted` 시그널 연결보다 `finished()` 오버라이드가 안정적이다.

---

# Part IV. CAD 스타일 UI 프레임

---

# Chapter 11. CAD식 레이어 관리자 Dock

**(배정: 10p | 난이도: ★★☆)**

## 11.1 학습 목표

- AutoCAD 레이어 특성 관리자와 동일한 컬럼 구성(상태/이름/색/선종류/굵기)의 Dock을 만든다.
- 색상 스와치 클릭 → ACI 팔레트 다이얼로그, 더블클릭 → 현재 레이어 지정을 구현한다.

## 11.2 UI 구성

**[그림 11-1] CAD 레이어 관리자 Dock 목표 화면 (도판 삽입 예정)**

| 컬럼 | 위젯 | 동작 |
|---|---|---|
| 현재 | ✔ 표시 | 더블클릭으로 현재 레이어 변경 |
| ON | 전구 아이콘 토글 | OFF ↔ ON |
| FREEZE | 눈꽃 아이콘 토글 | 스냅·선택 제외 |
| LOCK | 자물쇠 토글 | 편집 금지 |
| 이름 | 편집 가능 텍스트 | 개명 시 피처 layer 필드 일괄 갱신 |
| 색 | 색 스와치 | 클릭 → ACI 팔레트 |
| 선종류 | 콤보 | LTYPE 목록 |
| 굵기 | 콤보 | 0.00~2.11 mm |

## 11.3 핵심 구현 — 모델/뷰

```python
# ui/layer_manager_dock.py
from qgis.PyQt.QtWidgets import (QDockWidget, QTreeWidget, QTreeWidgetItem,
                                 QWidget, QVBoxLayout, QToolBar)
from qgis.PyQt.QtGui import QColor, QBrush, QIcon
from ..compat import exec_dialog

class LayerManagerDock(QDockWidget):
    COL_CUR, COL_ON, COL_FRZ, COL_LCK, COL_NAME, COL_CLR, COL_LT, COL_LW = range(8)

    def __init__(self, plugin):
        super().__init__("CAD 레이어")
        self.plugin = plugin
        self.doc = plugin.document
        self.tree = QTreeWidget()
        self.tree.setHeaderLabels(["", "켜기", "동결", "잠금", "이름",
                                   "색", "선종류", "굵기"])
        self.tree.setRootIsDecorated(False)
        self.setWidget(self.tree)
        self.tree.itemDoubleClicked.connect(self._on_double)
        self.tree.itemClicked.connect(self._on_click)
        self.doc.layersChanged.connect(self.refresh)
        self.refresh()

    def refresh(self):
        self.tree.clear()
        for name, cl in sorted(self.doc.cad_layers.items()):
            it = QTreeWidgetItem(["", "", "", "", name, "", cl.linetype,
                                  f"{max(cl.lineweight,0)/100:.2f}"])
            it.setText(self.COL_CUR,
                       "✔" if name == self.doc.current_layer else "")
            it.setBackground(self.COL_CLR, QBrush(QColor(*cl.rgb())))
            it.setData(0, 0x0100, name)          # Qt.UserRole
            # 상태 아이콘 (전구/눈꽃/자물쇠) — 리소스에서 로드
            self.tree.addTopLevelItem(it)

    def _on_click(self, item, col):
        name = item.data(0, 0x0100)
        cl = self.doc.cad_layers[name]
        if col == self.COL_ON:
            cl.is_off = not cl.is_off
            self._push_visibility()
        elif col == self.COL_CLR:
            self._pick_aci_color(cl)

    def _on_double(self, item, col):
        if col in (self.COL_CUR, self.COL_NAME):
            self.doc.current_layer = item.data(0, 0x0100)
            self.doc.currentLayerChanged.emit(self.doc.current_layer)
            self.refresh()

    def _push_visibility(self):
        """숨김 레이어 목록을 표현식 변수와 피처 필터에 반영 (Ch.9)."""
        hidden = [n for n, c in self.doc.cad_layers.items()
                  if c.is_off or c.is_frozen]
        self.plugin.style_service.set_hidden_layers(hidden)
```

## 11.4 ACI 팔레트 다이얼로그

255색 그리드(15×17) 버튼 + ByLayer/ByBlock 버튼으로 구성한다. 선택 결과는 ACI 번호로 반환하고, 색 변경 시 표현식 변수 `@qcad_layer_colors`를 갱신 → 캔버스 자동 리프레시된다. (전체 코드는 예제 저장소 `ui/aci_palette_dialog.py` — 지면에는 발췌 수록)

## 11.5 요약

- CAD 레이어 관리 UX가 QGIS 안에서 그대로 재현되었고, 데이터는 CadDocument 한 곳으로 수렴한다.

> **[호환성 노트]**
> `Qt.UserRole` 상수(0x0100)는 Qt5/Qt6 값이 같아 리터럴 사용이 가능하나, 정식 원고에서는 compat.py의 `USER_ROLE` 상수로 교체한다. QDockWidget API는 전 구간 동일하다.

---

# Chapter 12. 명령행 인터페이스(Command Line) 구현

**(배정: 10p | 난이도: ★★★ | 핵심)**

## 12.1 학습 목표

- 프롬프트·히스토리·자동완성·키워드 옵션을 갖춘 CAD 명령행을 구현한다.
- 명령행과 MapTool이 하나의 명령 상태기계를 공유하게 한다.

## 12.2 CAD 명령행의 동작 규격

```text
명령:                        ← 대기 프롬프트
명령: L ↵                    ← 별칭 입력
LINE 첫 번째 점 지정:         ← 명령이 프롬프트 갱신
LINE 다음 점 지정 또는 [닫기(C)/명령취소(U)]:
                              ← 키워드는 대괄호, 괄호 안 문자가 단축입력
- ↵ 또는 Space: 직전 명령 반복
- ESC: 취소
- ↑/↓: 히스토리
```

## 12.3 구현

```python
# ui/command_line_dock.py
from qgis.PyQt.QtWidgets import (QDockWidget, QWidget, QVBoxLayout,
                                 QPlainTextEdit, QLineEdit)
from qgis.PyQt.QtCore import Qt, pyqtSignal
from ..compat import KEY_ESCAPE, KEY_RETURN
from ..tools.input_parser import parse_input   # Ch.15

class CommandLineDock(QDockWidget):
    def __init__(self, plugin):
        super().__init__("명령행")
        self.plugin = plugin
        self.history, self.hidx = [], -1
        self.last_command = None

        w = QWidget(); lay = QVBoxLayout(w); lay.setContentsMargins(2,2,2,2)
        self.log = QPlainTextEdit(readOnly=True, maximumBlockCount=500)
        self.edit = QLineEdit()
        self.prompt = "명령:"
        self.edit.setPlaceholderText(self.prompt)
        lay.addWidget(self.log); lay.addWidget(self.edit)
        self.setWidget(w)

        self.edit.returnPressed.connect(self._on_enter)
        self.edit.installEventFilter(self)

    # ── 명령/좌표/키워드 분배 ────────────────────────────
    def _on_enter(self):
        text = self.edit.text().strip()
        self.edit.clear()
        self._append(f"{self.prompt} {text}")

        reg = self.plugin.registry
        if not text:                              # 빈 엔터 = 직전 명령 반복
            if reg.active:
                reg.active.on_enter()             # 진행 중이면 '입력 종료' 의미
            elif self.last_command:
                reg.run(self.last_command)
            return

        if reg.active:
            token = parse_input(text)             # Point / 숫자 / 키워드 판별
            if token.kind == "point":
                reg.active.on_point(token.point)
            elif token.kind == "number":
                reg.active.on_number(token.value)
            else:
                reg.active.on_keyword(token.text.upper())
        else:
            self.history.append(text); self.hidx = len(self.history)
            self.last_command = text
            try:
                reg.run(text)
            except KeyError:
                self._append(f"알 수 없는 명령입니다: {text}")

    def set_prompt(self, text):
        """활성 명령이 프롬프트를 갱신할 때 호출."""
        self.prompt = text
        self.edit.setPlaceholderText(text)

    def _append(self, line):
        self.log.appendPlainText(line)

    def eventFilter(self, obj, ev):
        if obj is self.edit and ev.type() == ev.Type.KeyPress if hasattr(ev, "Type") else ev.type() == 6:
            key = ev.key()
            if key == KEY_ESCAPE:
                if self.plugin.registry.active:
                    self.plugin.registry.active.cancel()
                    self._append("*취소*")
                return True
            # ↑/↓ 히스토리 — 지면 관계상 저장소 코드 참조
        return super().eventFilter(obj, ev)
```

> **TIP**
> 명령행 dock에 포커스가 없어도 캔버스에서 타이핑을 시작하면 자동으로 명령행에 문자가 입력되도록, 캔버스에도 eventFilter를 설치해 문자 키를 포워딩하면 CAD와 완전히 같은 손맛이 된다. (13.4절에서 구현)

## 12.4 명령 ↔ 프롬프트 프로토콜

BaseCommand에 프롬프트 헬퍼를 추가해 모든 명령이 같은 규격을 쓰게 한다.

```python
# base_command.py 확장
    def prompt(self, text, keywords=()):
        kw = "/".join(f"{k[0]}({k[1]})" for k in keywords)   # ("닫기","C")
        full = f"{self.label} {text}" + (f" 또는 [{kw}]:" if kw else ":")
        self.plugin.command_line.set_prompt(full)
```

## 12.5 요약

- 명령행이 Registry·Command와 연결되어, 이제 키보드만으로 명령을 구동할 수 있다.

> **[호환성 노트]**
> `QEvent.Type.KeyPress`(Qt6 스코프드) vs `QEvent.KeyPress`(Qt5) 분기가 위 코드에 노출되어 있다 — 정식 원고에서는 compat.py에 `EV_KEYPRESS` 상수로 정리한다(집필 메모). `QPlainTextEdit.maximumBlockCount`는 전 구간 동일.

---

# Chapter 13. 객체 속성바와 상태바 — 실시간 좌표·모드 표시

**(배정: 10p | 난이도: ★★☆)**

## 13.1 학습 목표

- 현재 레이어/색/선종류를 지정하는 속성바(콤보 3종)를 툴바에 배치한다.
- 상태바에 실시간 좌표와 직교/스냅/그리드 토글을 표시하고 F7/F8/F9 단축키를 연결한다.

## 13.2 속성바

```python
# ui/property_bar.py
from qgis.PyQt.QtWidgets import QComboBox, QLabel
class PropertyBar:
    def __init__(self, plugin, toolbar):
        self.doc = plugin.document
        self.layer_cb = QComboBox(); self.color_cb = QComboBox()
        self.ltype_cb = QComboBox()
        toolbar.addWidget(QLabel(" 레이어 ")); toolbar.addWidget(self.layer_cb)
        toolbar.addWidget(QLabel(" 색상 "));   toolbar.addWidget(self.color_cb)
        toolbar.addWidget(QLabel(" 선종류 ")); toolbar.addWidget(self.ltype_cb)

        self.color_cb.addItem("ByLayer", 256); self.color_cb.addItem("ByBlock", 0)
        for aci in (1,2,3,4,5,6,7,8,9):
            self.color_cb.addItem(f"색 {aci}", aci)

        self.layer_cb.currentTextChanged.connect(self._set_layer)
        self.doc.layersChanged.connect(self.reload)

    def _set_layer(self, name):
        if name:
            self.doc.current_layer = name
```

새로 작도되는 객체(17장)는 저장 직전에 `doc.current_layer / current_color / current_ltype`를 속성으로 기록한다 — CAD의 "현재 특성" 개념과 동일하다.

## 13.3 상태바 — 모드 토글과 좌표 표시

```python
from qgis.PyQt.QtWidgets import QToolButton, QLabel

class StatusBarPanel:
    def __init__(self, plugin):
        self.plugin = plugin
        sb = plugin.iface.mainWindow().statusBar()
        self.coord = QLabel("0.0000, 0.0000")
        self.btn_ortho = self._btn(sb, "직교", "F8")
        self.btn_osnap = self._btn(sb, "OSNAP", "F3")
        self.btn_grid  = self._btn(sb, "그리드", "F7")
        sb.addPermanentWidget(self.coord)

        canvas = plugin.iface.mapCanvas()
        canvas.xyCoordinates.connect(self._on_move)

    def _btn(self, sb, text, shortcut):
        b = QToolButton(); b.setText(text); b.setCheckable(True)
        b.setToolTip(f"{text} 켜기/끄기 ({shortcut})")
        sb.addPermanentWidget(b)
        return b

    def _on_move(self, pt):
        self.coord.setText(f"{pt.x():.4f}, {pt.y():.4f}")
```

`btn_ortho.isChecked()`는 15장의 좌표 필터가, `btn_osnap`은 16장의 스냅 엔진이 조회한다.

## 13.4 캔버스 타자 포워딩 (CAD 손맛의 완성)

```python
class CanvasKeyForwarder(QObject):
    """캔버스에서 문자를 치면 명령행으로 전달."""
    def __init__(self, plugin):
        super().__init__()
        self.plugin = plugin
        plugin.iface.mapCanvas().installEventFilter(self)

    def eventFilter(self, obj, ev):
        if ev.type() == 6:                      # KeyPress (compat 상수화 예정)
            ch = ev.text()
            if ch and (ch.isalnum() or ch in "@.,-<*"):
                cl = self.plugin.command_line
                cl.edit.setFocus()
                cl.edit.insert(ch)
                return True
        return False
```

## 13.5 요약 및 Part IV 마무리

레이어 관리자·명령행·속성바·상태바로 CAD 화면 프레임이 완성됐다. Part V에서 실제로 그린다.

> **[호환성 노트]**
> `QgsMapCanvas.xyCoordinates` 시그널은 3.x~4.x 전 구간 동일 시그니처(`QgsPointXY`)다. statusBar 접근은 표준 QMainWindow API라 버전 무관하다.

---

# Part V. 작도 도구와 스냅 시스템

---

# Chapter 14. QgsMapTool 기반 작도 도구 프레임워크

**(배정: 10p | 난이도: ★★★ | 핵심)**

## 14.1 학습 목표

- 모든 작도/편집 명령이 공유하는 단일 `CadMapTool`을 구현한다.
- 러버밴드·스냅마커·임시 안내선 렌더링 체계를 만든다.

## 14.2 설계 — MapTool은 하나, 명령은 여럿

명령마다 MapTool을 만들면 도구 전환 비용과 상태 누수가 커진다. **MapTool은 입출력 장치, 명령은 상태기계**로 역할을 나눈다.

```text
CadMapTool (1개)
  ├─ 마우스 이동 → 스냅 계산 → active_command.on_mouse(pt)
  ├─ 클릭       → active_command.on_point(pt)
  ├─ 우클릭     → active_command.on_enter()   (CAD 관행: 우클릭=엔터)
  └─ ESC        → active_command.cancel()
```

## 14.3 CadMapTool 구현

```python
# tools/cad_map_tool.py
from qgis.gui import QgsMapTool, QgsRubberBand, QgsVertexMarker
from qgis.core import QgsWkbTypes, QgsPointXY
from qgis.PyQt.QtGui import QColor
from ..compat import LEFT_BUTTON, RIGHT_BUTTON, KEY_ESCAPE, CROSS_CURSOR

class CadMapTool(QgsMapTool):
    def __init__(self, plugin):
        canvas = plugin.iface.mapCanvas()
        super().__init__(canvas)
        self.plugin = plugin
        self.snap = plugin.snap_engine            # Ch.16
        self.setCursor(CROSS_CURSOR)

        # 스냅 마커 (스냅 종류별 모양은 16장에서 확장)
        self.marker = QgsVertexMarker(canvas)
        self.marker.setColor(QColor(0, 200, 0))
        self.marker.setIconType(QgsVertexMarker.ICON_BOX)
        self.marker.setPenWidth(2)
        self.marker.hide()

    def _resolve_point(self, event):
        raw = self.toMapCoordinates(event.pos())
        snapped = self.snap.snap(event.pos())     # OSNAP 우선
        if snapped is not None:
            self.marker.setCenter(snapped); self.marker.show()
            pt = snapped
        else:
            self.marker.hide()
            pt = raw
        # 직교/극좌표 필터 (Ch.15)
        return self.plugin.input_filter.apply(pt)

    def canvasMoveEvent(self, event):
        pt = self._resolve_point(event)
        cmd = self.plugin.registry.active
        if cmd:
            cmd.on_mouse(pt)

    def canvasPressEvent(self, event):
        cmd = self.plugin.registry.active
        if not cmd:
            return
        if event.button() == LEFT_BUTTON:
            cmd.on_point(self._resolve_point(event))
        elif event.button() == RIGHT_BUTTON:
            cmd.on_enter()

    def keyPressEvent(self, event):
        if event.key() == KEY_ESCAPE:
            cmd = self.plugin.registry.active
            if cmd:
                cmd.cancel()
        else:
            super().keyPressEvent(event)

    def deactivate(self):
        self.marker.hide()
        super().deactivate()
```

## 14.4 러버밴드 서비스

명령이 직접 QgsRubberBand를 만들지 않도록 공용 서비스로 캡슐화한다.

```python
class PreviewService:
    def __init__(self, canvas):
        self.canvas = canvas
        self.bands = []

    def line_band(self, dashed=True):
        rb = QgsRubberBand(self.canvas, QgsWkbTypes.LineGeometry)
        rb.setColor(QColor(255, 255, 255) if False else QColor(80, 80, 80))
        rb.setWidth(1)
        if dashed:
            rb.setLineStyle(2)        # Qt.DashLine 값 — compat 상수 사용 권장
        self.bands.append(rb)
        return rb

    def clear(self):
        for rb in self.bands:
            rb.reset()
        self.bands.clear()
```

## 14.5 요약

- 단일 MapTool + 명령 상태기계 + 프리뷰 서비스로, 이후 20여 개 명령이 이 세 객체만으로 구현된다.

> **[호환성 노트]**
> `QgsRubberBand` 생성자는 3.30부터 `Qgis.GeometryType`을 받는 오버로드가 추가되었으나 `QgsWkbTypes.LineGeometry`도 전 구간 유효하다. `QgsVertexMarker.ICON_BOX` 등 아이콘 상수는 3.6~4.x 동일하다.

---

# Chapter 15. 좌표 입력 시스템 — 절대·상대(@)·극좌표(<)

**(배정: 10p | 난이도: ★★★ | 핵심)**

## 15.1 학습 목표

- CAD 좌표 문법 전체를 파싱하는 `parse_input`을 구현한다.
- 직교 모드(F8)와 극좌표 추적을 좌표 필터로 구현한다.

## 15.2 입력 문법 규격

| 입력 | 의미 |
|---|---|
| `1000,2000` | 절대좌표 |
| `@50,30` | 마지막 점 기준 상대좌표 |
| `@100<45` | 마지막 점 기준 거리 100, 각도 45° |
| `100<45` | 원점 기준 극좌표 (드묾) |
| `250` (숫자 단독) | 문맥값: 거리/반지름/각도 (명령이 해석) |
| `C`, `U` 등 | 키워드 |

## 15.3 파서 구현

```python
# tools/input_parser.py
import re, math
from dataclasses import dataclass
from qgis.core import QgsPointXY

NUM = r"[-+]?\d*\.?\d+(?:[eE][-+]?\d+)?"
RE_XY    = re.compile(rf"^(@?)({NUM})\s*,\s*({NUM})$")
RE_POLAR = re.compile(rf"^(@?)({NUM})\s*<\s*({NUM})$")
RE_NUM   = re.compile(rf"^{NUM}$")

@dataclass
class Token:
    kind: str                 # "point" | "number" | "keyword"
    point: QgsPointXY = None
    value: float = None
    text: str = ""

class InputContext:
    """마지막 점을 기억하는 파서 문맥 — CadDocument가 보유."""
    last_point = None

def parse_input(text, ctx=InputContext):
    text = text.strip()
    m = RE_XY.match(text)
    if m:
        rel, x, y = m.group(1) == "@", float(m.group(2)), float(m.group(3))
        if rel and ctx.last_point is not None:
            return Token("point", QgsPointXY(ctx.last_point.x() + x,
                                             ctx.last_point.y() + y))
        return Token("point", QgsPointXY(x, y))
    m = RE_POLAR.match(text)
    if m:
        rel, d, a = m.group(1) == "@", float(m.group(2)), math.radians(float(m.group(3)))
        base = ctx.last_point if (rel and ctx.last_point) else QgsPointXY(0, 0)
        return Token("point", QgsPointXY(base.x() + d*math.cos(a),
                                         base.y() + d*math.sin(a)))
    if RE_NUM.match(text):
        return Token("number", value=float(text))
    return Token("keyword", text=text)
```

> **TIP — 측량 방위각 모드**
> CAD 각도는 동쪽 0°·반시계이지만, 측량 실무는 북쪽 0°·시계방향 방위각을 쓴다. 설정에 "각도 기준: CAD/방위각" 옵션을 두고 파서에서 `a = math.pi/2 - a`로 변환하면 측량 성과 입력이 편해진다 — 국내 실무용 차별화 포인트.

## 15.4 직교 모드 필터

```python
class InputFilter:
    def __init__(self, plugin):
        self.plugin = plugin

    def apply(self, pt):
        doc = self.plugin.document
        base = InputContext.last_point
        if base is None:
            return pt
        if self.plugin.statusbar.btn_ortho.isChecked():
            dx, dy = abs(pt.x()-base.x()), abs(pt.y()-base.y())
            return (QgsPointXY(pt.x(), base.y()) if dx >= dy
                    else QgsPointXY(base.x(), pt.y()))
        return pt
```

극좌표 추적(마우스가 15° 배수 방향에 근접하면 자석처럼 끌림 + 각도 안내선 표시)은 같은 필터의 확장으로 구현한다 — 정식 원고에서 3쪽, 안내선 러버밴드 포함(집필 메모).

## 15.5 요약

- 키보드 좌표·마우스 좌표가 동일한 필터 체인을 거쳐 명령에 도달한다: `raw → OSNAP → ortho/polar → command`.

> **[호환성 노트]**
> 순수 Python + QgsPointXY라 버전 이슈 없음. `dataclass`는 Python 3.7+ — QGIS 3.6 번들(3.7)에서도 동작한다.

---

# Chapter 16. 객체 스냅(OSNAP) 엔진

**(배정: 12p | 난이도: ★★★★ | 핵심)**

## 16.1 학습 목표

- QGIS 스냅(꼭짓점/선분/교차)을 기반으로 CAD OSNAP 7종(끝점·중간점·중심·교차·수직·최근접·접점)을 구현한다.
- 스냅 종류별 마커(사각/삼각/원/X)를 표시한다.

## 16.2 아키텍처

```text
1차: QgsSnappingUtils           → 끝점(Vertex)·최근접(Segment)·교차(Intersection)
2차: 후보 피처 지오메트리 해석    → 중간점·중심점·수직점·접점 (자체 계산)
우선순위: 끝점 > 교차 > 중심 > 중간점 > 수직 > 접점 > 최근접
```

## 16.3 구현

```python
# tools/snap_engine.py
import math
from qgis.core import (QgsSnappingConfig, QgsPointXY, QgsGeometry,
                       QgsTolerance, QgsPointLocator)
from ..compat import vertex_segment_flags

class SnapKind:
    END, MID, CEN, INT, PERP, TAN, NEAR = range(7)

class SnapResult:
    def __init__(self, point, kind):
        self.point, self.kind = point, kind

class OsnapEngine:
    PRIORITY = [SnapKind.END, SnapKind.INT, SnapKind.CEN,
                SnapKind.MID, SnapKind.PERP, SnapKind.TAN, SnapKind.NEAR]

    def __init__(self, plugin):
        self.plugin = plugin
        self.canvas = plugin.iface.mapCanvas()
        self.enabled = {k: True for k in self.PRIORITY}
        self.ref_point = None            # 수직/접점 계산용 기준점 (명령이 설정)

    def _utils(self):
        u = self.canvas.snappingUtils()
        cfg = QgsSnappingConfig(u.config())
        cfg.setEnabled(True)
        cfg.setTypeFlag(vertex_segment_flags())      # compat (Ch.5)
        cfg.setIntersectionSnapping(True)
        u.setConfig(cfg)
        return u

    def snap(self, screen_pos):
        if not self.plugin.statusbar.btn_osnap.isChecked():
            return None
        u = self._utils()
        m = u.snapToMap(screen_pos)                  # QgsPointLocator.Match
        candidates = []

        if m.isValid():
            if m.hasVertex():
                candidates.append(SnapResult(m.point(), SnapKind.END))
            if m.hasEdge():
                candidates.extend(self._edge_snaps(m))

        for kind in self.PRIORITY:
            if not self.enabled[kind]:
                continue
            for c in candidates:
                if c.kind == kind:
                    return c.point
        return None

    def _edge_snaps(self, match):
        """선분 매치에서 중간점·중심·수직·접점 파생."""
        out = [SnapResult(match.point(), SnapKind.NEAR)]
        layer, fid = match.layer(), match.featureId()
        if layer is None:
            return out
        feat = layer.getFeature(fid)
        geom = feat.geometry()

        # 중간점: 매치된 선분의 두 정점
        p1, p2 = match.edgePoints()
        out.append(SnapResult(QgsPointXY((p1.x()+p2.x())/2,
                                         (p1.y()+p2.y())/2), SnapKind.MID))

        # 중심점: extra JSON에 center가 있으면 (ARC/CIRCLE — Ch.8)
        import json
        extra = json.loads(feat["extra"] or "{}")
        if "center" in extra:
            out.append(SnapResult(QgsPointXY(*extra["center"]), SnapKind.CEN))

        # 수직점: 기준점에서 선분에 내린 발
        if self.ref_point is not None:
            perp = self._foot_of_perpendicular(self.ref_point, p1, p2)
            if perp:
                out.append(SnapResult(perp, SnapKind.PERP))

        # 접점: 기준점에서 원/호에 그은 접선의 접점
        if self.ref_point is not None and "center" in extra:
            out.extend(self._tangent_points(self.ref_point,
                                            QgsPointXY(*extra["center"]),
                                            extra["radius"]))
        return out

    @staticmethod
    def _foot_of_perpendicular(p, a, b):
        ax, ay, bx, by = a.x(), a.y(), b.x(), b.y()
        dx, dy = bx-ax, by-ay
        L2 = dx*dx + dy*dy
        if L2 == 0:
            return None
        t = ((p.x()-ax)*dx + (p.y()-ay)*dy) / L2
        if not (0.0 <= t <= 1.0):
            return None
        return QgsPointXY(ax + t*dx, ay + t*dy)

    @staticmethod
    def _tangent_points(p, c, r):
        d = math.hypot(p.x()-c.x(), p.y()-c.y())
        if d <= r:
            return []
        a = math.atan2(p.y()-c.y(), p.x()-c.x())
        b = math.acos(r / d)
        pts = []
        for s in (+1, -1):
            ang = a + s*b
            pts.append(SnapResult(QgsPointXY(c.x()+r*math.cos(ang),
                                             c.y()+r*math.sin(ang)),
                                  SnapKind.TAN))
        return pts
```

**[그림 16-1] OSNAP 마커 규격: □끝점 △중간점 ○중심 ✕교차 ⊥수직 (도판 삽입 예정)**

## 16.4 스냅 마커 도형화

`QgsVertexMarker`의 아이콘 종류를 스냅 종류에 매핑하고, 14장의 CadMapTool `_resolve_point`가 `SnapResult.kind`를 받아 마커 모양을 바꾸도록 확장한다. (저장소 코드 참조 — 지면 1쪽)

## 16.5 FREEZE/LOCK 레이어 제외

`_edge_snaps`에서 `match.layer()`가 QCAD 레이어인 경우, 피처의 `layer` 필드가 `@qcad_frozen_layers`에 포함되면 후보에서 제외한다 — 9.5절 정책의 실행 지점이다.

## 16.6 요약

- QGIS 스냅 1차 + 기하 해석 2차의 이중 구조로 CAD OSNAP 7종이 완성됐다.
- `extra` JSON(중심/반지름)을 8장에서 저장해 둔 것이 여기서 회수되는 설계 배당금이다.

> **[호환성 노트]**
> `QgsPointLocator.Match.edgePoints()`는 3.x~4.x 동일. `setTypeFlag`는 3.26+에서 `setTypeFlag(Qgis.SnappingTypes)`, 이전에는 `setType(int)`이므로 compat의 `vertex_segment_flags()`가 반환형까지 흡수하도록 정식 원고에서 보강한다(집필 메모).

---

# Chapter 17. LINE부터 ARC까지 — 작도 명령 8종 구현

**(배정: 10p | 난이도: ★★★ | 핵심)**

## 17.1 학습 목표

- LINE / PLINE / CIRCLE / ARC / RECTANG / POLYGON / POINT / XLINE 8개 명령을 완성한다.
- 명령 상태기계 패턴을 반복 훈련한다.

## 17.2 LINE — 표준 패턴 확립

```python
# commands/draw/line.py
from qgis.core import QgsPointXY, QgsGeometry
from ..base_command import BaseCommand
from ...tools.input_parser import InputContext

class LineCommand(BaseCommand):
    name, aliases, label = "line", ("l",), "LINE"
    icon = ":/qcad/icons/line.svg"

    def start(self, *args):
        self.points = []
        self.rb = self.plugin.preview.line_band()
        self.plugin.map_tool.activate_for(self)
        self.prompt("첫 번째 점 지정", [])

    def on_point(self, pt):
        self.points.append(pt)
        InputContext.last_point = pt
        if len(self.points) >= 2:
            self._commit_segment(self.points[-2], self.points[-1])
        kws = [("닫기", "C"), ("명령취소", "U")] if len(self.points) >= 2 \
              else [("명령취소", "U")]
        self.prompt("다음 점 지정", kws)

    def on_mouse(self, pt):
        if self.points:
            self.rb.reset()
            self.rb.addPoint(self.points[-1]); self.rb.addPoint(pt)

    def on_keyword(self, kw):
        if kw in ("U", "명령취소"):
            if self.points:
                self.points.pop()
                self.doc.undo_last()               # Ch.18 undo 스택
        elif kw in ("C", "닫기") and len(self.points) >= 3:
            self._commit_segment(self.points[-1], self.points[0])
            self.finish()

    def on_enter(self):
        self.finish()

    def _commit_segment(self, p1, p2):
        geom = QgsGeometry.fromPolylineXY([p1, p2])
        self.doc.add_entity("LINE", geom)          # ↓ 17.3

    def cancel(self):
        self.plugin.preview.clear()
        self.plugin.registry.active = None

    def finish(self):
        self.cancel()
        self.plugin.command_line.set_prompt("명령:")
```

## 17.3 CadDocument.add_entity — 현재 특성 반영과 신규 핸들

```python
# core/document.py 확장
    def add_entity(self, etype, geom, extra=None):
        vl = self.qgis_layers["lines" if etype != "POINT" else "points"]
        f = QgsFeature(vl.fields())
        f.setGeometry(geom)
        f["handle"] = self.new_handle()        # "NEW:00000001" — 저장 시 재부여(Ch.25)
        f["etype"], f["layer"] = etype, self.current_layer
        f["aci"], f["ltype"] = self.current_color, self.current_ltype
        f["extra"] = json.dumps(extra or {}, ensure_ascii=False)
        vl.dataProvider().addFeature(f)
        vl.triggerRepaint()
        self.undo_stack.push_add(vl.id(), f.id())
```

## 17.4 CIRCLE — 3가지 정의 방식

CAD의 CIRCLE은 [중심-반지름 / 2점 / 3점 / TTR] 옵션을 갖는다. 초안에서는 중심-반지름과 2점만 구현하고, 3점 원(외접원 공식)과 TTR은 연습문제로 남긴다.

```python
class CircleCommand(BaseCommand):
    name, aliases, label = "circle", ("c",), "CIRCLE"

    def start(self, *args):
        self.center = None
        self.plugin.map_tool.activate_for(self)
        self.prompt("원의 중심점 지정", [("2점", "2P"), ("3점", "3P")])

    def on_point(self, pt):
        if self.center is None:
            self.center = pt
            InputContext.last_point = pt
            self.prompt("반지름 지정")
        else:
            r = self.center.distance(pt)
            self._commit(r)

    def on_number(self, value):          # 명령행 "250 ↵" → 반지름
        if self.center is not None:
            self._commit(value)

    def on_mouse(self, pt):
        if self.center is not None:
            # 원 미리보기: 64각형 근사 러버밴드 (저장소 코드)
            self.plugin.preview.circle_preview(self.center, self.center.distance(pt))

    def _commit(self, r):
        geom = circle_geometry(self.center, r)      # 8.5절 로직 재사용
        self.doc.add_entity("CIRCLE", geom,
                            {"center": [self.center.x(), self.center.y()],
                             "radius": r})
        self.finish()
```

## 17.5 ARC / RECTANG / POLYGON / XLINE — 규격 요약

| 명령 | 상태 시퀀스 | 핵심 기하 |
|---|---|---|
| ARC | 시작점→통과점→끝점 (3P) | `QgsCircularString(p1,p2,p3)` 그대로 |
| RECTANG | 코너1→코너2 | 폐합 LWPOLYLINE, `closed=True` |
| POLYGON | 변 수(number)→중심→내접/외접 | 정다각형 정점 생성 |
| XLINE | 통과점→방향점 | 캔버스 범위로 클리핑된 무한선(재계산 훅) |

(각 명령 전체 코드는 예제 저장소 `commands/draw/` — 지면에는 ARC 전체와 나머지 발췌 수록. 정식 원고에서 각 1.5쪽 배정 — 집필 메모)

## 17.6 검수 기준 (1.2절의 판정 기준 적용)

- [ ] `L ↵ 1000,2000 ↵ @50<90 ↵ ↵` 입력 시 CAD와 동일한 선이 생성되는가
- [ ] 우클릭이 엔터로 동작하는가
- [ ] 직교 모드에서 미리보기가 수평/수직으로 고정되는가
- [ ] 생성 객체의 레이어·색·선종류가 속성바의 현재 특성과 일치하는가

## 17.7 요약 및 Part V 마무리

명령행·좌표 파서·OSNAP·MapTool이 결합되어 실제 제도가 가능해졌다. Part VI에서 편집 명령을 만든다.

> **[호환성 노트]**
> `QgsGeometry.fromPolylineXY`는 3.x~4.x 동일. 러버밴드의 원 미리보기에서 `QgsRubberBand.addGeometry` 사용 시 3.6에서는 CRS 인자를 명시(`addGeometry(geom, None)`)해야 경고가 없다.

---

# Part VI. 편집 도구

---

# Chapter 18. 선택 시스템과 그립(Grip) 편집

**(배정: 10p | 난이도: ★★★)**

## 18.1 학습 목표

- CAD식 선택(개별 픽 / 창(W) / 걸침(C))을 구현한다.
- 선택 객체의 그립(정점 파란 사각형)을 표시하고 드래그로 이동한다.
- 명령-선행(verb-noun)과 선택-선행(noun-verb) 두 흐름을 모두 지원한다.
- undo/redo 스택을 구축한다.

## 18.2 창 선택 vs 걸침 선택

CAD 규칙: 좌→우 드래그는 **완전 포함(Window, 실선 파랑)**, 우→좌 드래그는 **교차 포함(Crossing, 점선 초록)**.

```python
# tools/selection.py
from qgis.core import QgsRectangle, QgsFeatureRequest

class SelectionService:
    def __init__(self, plugin):
        self.plugin = plugin
        self.selected = {}          # layer_id → set(fid)

    def box_select(self, p1, p2):
        rect = QgsRectangle(p1, p2)
        crossing = p2.x() < p1.x()              # 우→좌 = 걸침
        for key, vl in self.plugin.document.qgis_layers.items():
            req = QgsFeatureRequest().setFilterRect(rect)
            fids = set()
            for f in vl.getFeatures(req):
                if self._is_locked(f):
                    continue
                g = f.geometry()
                if crossing or rect.contains(g.boundingBox()):
                    if crossing and not g.intersects(rect):
                        continue
                    fids.add(f.id())
            vl.selectByIds(list(fids | set(vl.selectedFeatureIds())))
```

`_is_locked`는 9.5절 LOCK 정책의 실행 지점이다.

## 18.3 그립 표시와 드래그

- 선택 변경 시 각 피처의 정점을 `QgsVertexMarker`(파란 사각) 풀(pool)로 표시한다.
- 그립 클릭 → 해당 정점이 "핫 그립"(빨강)이 되고 이동 모드 진입 → 새 점 지정 시 `QgsVectorLayer.moveVertex` 계열이 아닌 **지오메트리 재작성**으로 처리(곡선 보존).

```python
def move_grip(vl, fid, vertex_index, new_pt):
    f = next(vl.getFeatures(QgsFeatureRequest(fid)))
    g = QgsGeometry(f.geometry())
    g.moveVertex(new_pt.x(), new_pt.y(), vertex_index)
    vl.dataProvider().changeGeometryValues({fid: g})
    vl.triggerRepaint()
```

> **WARNING**
> `moveVertex`는 CircularString의 통과점 정점도 이동시키지만, ARC의 `extra`(center/radius)와 불일치가 생긴다. 그립 이동 후 `extra`를 재계산하는 후처리(`sync_extra_from_geometry`)를 반드시 호출한다 — OSNAP 중심점 스냅이 어긋나는 잠복 버그의 예방책.

## 18.4 Undo/Redo 스택

QGIS 편집 세션 undo와 별개로, 플러그인은 **문서 수준의 명령 단위 undo**를 갖는다.

```python
# core/undo.py
class UndoStack:
    def __init__(self, doc):
        self.doc, self.undo_list, self.redo_list = doc, [], []

    def push(self, op):            # op: ("add"|"del"|"mod", layer_id, payload)
        self.undo_list.append(op)
        self.redo_list.clear()

    def undo(self):
        if not self.undo_list:
            return
        op = self.undo_list.pop()
        inv = self._apply_inverse(op)
        self.redo_list.append(inv)
```

`U` 명령·Ctrl+Z를 레지스트리에 등록해 명령행에서 `U ↵`가 동작하게 한다.

## 18.5 요약

- 선택/그립/undo는 이후 모든 편집 명령의 공통 기반이다.

> **[호환성 노트]**
> `selectByIds`·`changeGeometryValues`는 3.x~4.x 동일. Qt6에서 `QgsFeatureRequest` 복사 시 암묵 변환 경고가 늘었으므로 항상 명시 생성자를 쓴다.

---

# Chapter 19. 변환 편집 — MOVE, COPY, ROTATE, MIRROR, SCALE

**(배정: 10p | 난이도: ★★★)**

## 19.1 공통 구조 — TransformCommand 베이스

다섯 명령은 "선택 → 기준점 → 파라미터(제2점/각도/배율) → 아핀 변환 적용"으로 동일하다. 베이스 클래스 하나로 통일한다.

```python
# commands/modify/transform_base.py
from qgis.core import QgsGeometry
import math

class TransformCommand(BaseCommand):
    keeps_original = False          # COPY만 True

    def start(self, *args):
        self.base = None
        sel = self.plugin.selection.snapshot()
        if sel:                                   # noun-verb: 이미 선택됨
            self.targets = sel
            self._ask_base()
        else:                                     # verb-noun: 선택부터
            self.targets = None
            self.prompt("객체 선택")
            self.plugin.map_tool.activate_selection(self._on_selected)

    def _on_selected(self, sel):
        self.targets = sel
        self._ask_base()

    def _ask_base(self):
        self.prompt("기준점 지정")
        self.plugin.map_tool.activate_for(self)
        self.plugin.snap_engine.ref_point = None

    def on_point(self, pt):
        if self.base is None:
            self.base = pt
            self.plugin.snap_engine.ref_point = pt   # 수직/접점 스냅 기준
            self.prompt(self.second_prompt)
        else:
            self._apply(pt)
            self.finish()

    def _apply(self, second_pt):
        for (vl, fid) in self.targets:
            f = next(vl.getFeatures(QgsFeatureRequest(fid)))
            g = QgsGeometry(f.geometry())
            self.transform_geometry(g, self.base, second_pt)
            if self.keeps_original:
                self.doc.clone_feature(vl, f, g)
            else:
                vl.dataProvider().changeGeometryValues({fid: g})
            vl.triggerRepaint()
        self.doc.undo_stack.push(("transform", ...))
```

## 19.2 다섯 명령의 변환 정의

```python
class MoveCommand(TransformCommand):
    name, aliases, label = "move", ("m",), "MOVE"
    second_prompt = "두 번째 점 지정"
    def transform_geometry(self, g, base, pt):
        g.translate(pt.x()-base.x(), pt.y()-base.y())

class CopyCommand(MoveCommand):
    name, aliases, label = "copy", ("co", "cp"), "COPY"
    keeps_original = True
    # CAD 규칙: COPY는 연속 복사 — on_point에서 finish 대신 반복 (저장소 코드)

class RotateCommand(TransformCommand):
    name, aliases, label = "rotate", ("ro",), "ROTATE"
    second_prompt = "회전 각도 지정"
    def transform_geometry(self, g, base, pt):
        ang = math.degrees(math.atan2(pt.y()-base.y(), pt.x()-base.x()))
        g.rotate(-ang, base)            # QGIS rotate는 시계방향 양수

    def on_number(self, value):         # "45 ↵" 각도 직접 입력
        if self.base is not None:
            for (vl, fid) in self.targets:
                ...
                g.rotate(-value, self.base)

class ScaleCommand(TransformCommand):
    name, aliases, label = "scale", ("sc",), "SCALE"
    second_prompt = "축척 비율 지정"
    def transform_geometry(self, g, base, pt):
        k = base.distance(pt) / max(self._ref_len, 1e-12)
        g.transform(QgsMatrix4x4(...))  # 정식 원고: scale 헬퍼 함수로 전개

class MirrorCommand(TransformCommand):
    name, aliases, label = "mirror", ("mi",), "MIRROR"
    second_prompt = "대칭선의 두 번째 점 지정"
    def transform_geometry(self, g, base, pt):
        # 대칭 변환: 평행이동→회전→X축반사→역회전→역이동 합성
        ang = math.atan2(pt.y()-base.y(), pt.x()-base.x())
        g.translate(-base.x(), -base.y())
        g.rotate(math.degrees(ang), QgsPointXY(0, 0))
        g.transform(QgsCoordinateTransform())     # 정식 원고: y→-y 아핀 헬퍼
        g.rotate(-math.degrees(ang), QgsPointXY(0, 0))
        g.translate(base.x(), base.y())
```

> **집필 메모**
> `QgsGeometry`에는 순수 scale/reflect 메서드가 없어 정식 원고에서는 `geometry_utils.affine(g, a,b,c,d,tx,ty)` 공용 헬퍼(정점 순회 방식, 곡선 통과점 보존)를 2쪽에 걸쳐 구현하고 다섯 명령이 이를 사용하는 형태로 통일한다. MIRROR 후 TEXT의 반전 방지(MIRRTEXT=0 동작)도 여기서 다룬다.

## 19.3 실시간 미리보기

`on_mouse`에서 대상 지오메트리 사본에 동일 변환을 적용해 러버밴드로 표시한다. 500개 이상 선택 시에는 바운딩박스만 미리보기하는 성능 폴백을 넣는다.

## 19.4 요약

- 아핀 변환 5종이 베이스 클래스 하나로 정리되었다 — 명령 추가 비용이 급감하는 구간이다.

> **[호환성 노트]**
> `QgsGeometry.rotate/translate`는 3.x~4.x 동일 시그니처. 회전 방향(시계 양수)이 CAD(반시계 양수)와 반대인 점만 부호로 흡수한다.

---

# Chapter 20. 형상 편집 I — OFFSET, ARRAY, EXPLODE

**(배정: 10p | 난이도: ★★★★)**

## 20.1 OFFSET — 도로 설계의 주력 명령

시퀀스: 거리 입력 → 객체 선택 → 방향 측 클릭 → (반복).

```python
# commands/modify/offset.py
class OffsetCommand(BaseCommand):
    name, aliases, label = "offset", ("o",), "OFFSET"

    def start(self):
        self.dist = None
        self.target = None
        self.prompt("간격띄우기 거리 지정")
        self.plugin.map_tool.activate_for(self)

    def on_number(self, value):
        self.dist = abs(value)
        self.prompt("간격띄우기할 객체 선택")

    def on_point(self, pt):
        if self.dist is None:
            return
        if self.target is None:
            self.target = self.plugin.selection.pick_one(pt)   # 단일 픽
            if self.target:
                self.prompt("간격띄우기할 쪽의 점 지정")
        else:
            vl, fid = self.target
            f = next(vl.getFeatures(QgsFeatureRequest(fid)))
            g = f.geometry()
            side = self._which_side(g, pt)      # 좌 +1 / 우 -1
            off = g.offsetCurve(self.dist * side, 8,
                                Qgis.JoinStyle.Round if not QGIS_3X
                                else QgsGeometry.JoinStyleRound, 2.0)
            if off and not off.isEmpty():
                self.doc.add_entity(f["etype"], off,
                                    json.loads(f["extra"] or "{}"))
            self.target = None
            self.prompt("간격띄우기할 객체 선택")   # CAD처럼 반복

    @staticmethod
    def _which_side(geom, pt):
        # 최근접 선분의 진행방향 대비 외적 부호로 좌/우 판정
        _, seg_pt, after, _ = geom.closestSegmentWithContext(pt)[:4]
        ...
        return 1 if cross > 0 else -1
```

> **WARNING**
> `offsetCurve`는 원호(CircularString)를 세그먼트화한 결과를 반환한다. CIRCLE/ARC의 offset은 기하학적으로 "반지름 ± 거리의 동심원/호"이므로, `etype`이 CIRCLE/ARC이면 offsetCurve를 쓰지 말고 `extra`의 center/radius로 **동심 원호를 직접 생성**한다 — CAD와 결과가 일치하는 유일한 방법이다. (본문에서 두 방식 비교 그림 포함 — [그림 20-1])

## 20.2 ARRAY — 직사각형/원형 배열

- 직사각형: 행·열·간격 입력 → 이중 루프 translate 복제.
- 원형(폴라): 중심·개수·채움각 → 회전 복제. INSERT(블록 참조)의 배열은 삽입점만 복제하면 되므로 가장 저렴하다.
- UI는 명령행 파라미터 방식(초안)과 다이얼로그 방식(연습문제) 모두 제시.

## 20.3 EXPLODE — 복합 객체 분해

| 대상 | 분해 결과 |
|---|---|
| LWPOLYLINE | LINE + ARC들 (`extra.bulges` 참조) |
| INSERT | 블록 정의를 삽입 변환(이동·회전·축척) 적용해 전개 (24장 로직 역이용) |
| MTEXT | TEXT 행들 |
| DIMENSION | 치수 구성요소(선·화살표·문자) |

```python
def explode_lwpolyline(doc, vl, fid):
    f = next(vl.getFeatures(QgsFeatureRequest(fid)))
    extra = json.loads(f["extra"] or "{}")
    verts = extract_vertices(f.geometry())
    for i, b in enumerate(extra.get("bulges", [])):
        p1, p2 = verts[i], verts[(i+1) % len(verts)]
        if abs(b) > 1e-12:
            doc.add_entity("ARC", arc_from_bulge(p1, p2, b),
                           arc_params_from_bulge(p1, p2, b))
        else:
            doc.add_entity("LINE", QgsGeometry.fromPolylineXY([p1, p2]))
    vl.dataProvider().deleteFeatures([fid])
```

## 20.4 요약

- OFFSET의 원호 특수처리와 EXPLODE의 벌지 역변환 모두 8장의 `extra` 설계가 지탱한다.

> **[호환성 노트]**
> `offsetCurve`의 JoinStyle 인자가 3.x(`QgsGeometry.JoinStyleRound`)와 4.x(`Qgis.JoinStyle.Round`)에서 다르다 — 위 코드처럼 compat 분기하며, 정식 원고에서는 compat.py 상수 `JOIN_ROUND`로 격리한다.

---

# Chapter 21. 형상 편집 II — TRIM, EXTEND, FILLET

**(배정: 10p | 난이도: ★★★★★ | 본서 최고 난도)**

## 21.1 TRIM — 절단 모서리 기반 잘라내기

알고리즘:

```text
1. 절단 모서리(경계) 객체 집합 선택 (빈 엔터 = 전체 객체가 경계)
2. 자를 객체 픽 → 픽 위치가 속한 "구간" 판정
3. 대상 지오메트리를 모든 경계와의 교차점에서 분할(split)
4. 픽 지점이 포함된 분할 조각을 삭제, 나머지 조각을 재등록
```

```python
# commands/modify/trim.py — 핵심부
def trim_at(self, target, pick_pt, boundaries):
    vl, fid = target
    f = next(vl.getFeatures(QgsFeatureRequest(fid)))
    g = f.geometry()

    cut_points = []
    for (bvl, bfid) in boundaries:
        bg = next(bvl.getFeatures(QgsFeatureRequest(bfid))).geometry()
        inter = g.intersection(bg)
        cut_points.extend(as_points(inter))          # Point/MultiPoint 전개

    if not cut_points:
        self.message("절단 모서리와 교차하지 않습니다.")
        return

    pieces = split_curve_at_points(g, cut_points)    # ↓ 21.3
    victim = min(pieces, key=lambda p: p.distance(QgsGeometry.fromPointXY(pick_pt)))
    pieces.remove(victim)

    vl.dataProvider().deleteFeatures([fid])
    for p in pieces:
        self.doc.add_entity_like(f, p)               # 속성 복제 재등록
```

EXTEND는 TRIM의 쌍대(雙對)다: 대상 곡선의 픽된 끝을 접선/원호 방향으로 연장한 가상 곡선과 경계의 교차점을 구해 끝점을 이동한다. Shift 키로 TRIM↔EXTEND를 전환하는 CAD 동작도 재현한다.

## 21.2 곡선 분할 — split_curve_at_points

`QgsGeometry.splitGeometry`는 직선 분할선 기반이라 점 기반 분할에 부적합하다. **선형 참조(linear referencing)** 로 구현한다.

```python
def split_curve_at_points(geom, points, tol=1e-9):
    """곡선을 곡선상 점들에서 분할. 원호 구간 보존."""
    dists = sorted({geom.lineLocatePoint(QgsGeometry.fromPointXY(p))
                    for p in points})
    dists = [d for d in dists if tol < d < geom.length() - tol]
    pieces, prev = [], 0.0
    for d in dists + [geom.length()]:
        piece = curve_substring(geom, prev, d)       # ↓
        if piece and piece.length() > tol:
            pieces.append(piece)
        prev = d
    return pieces
```

`curve_substring`은 3.x 일부 버전에 없는 `QgsGeometry.curveSubstring`의 폴백까지 포함해 정식 원고에서 2쪽으로 전개한다(집필 메모: 3.6 폴백은 densify 후 부분추출 + ARC는 각도 파라미터로 직접 절단).

**[그림 21-1] TRIM 판정: 교차점 3개로 4조각 분할, 픽 조각 삭제 (도판 삽입 예정)**

## 21.3 FILLET — 모깎기

두 선분 L1, L2와 반지름 r이 주어질 때:

```text
1. 두 직선의 교차점 X와 사잇각 2α 계산
2. 접점까지의 거리 t = r / tan(α)
3. 각 선분 위에서 X로부터 t만큼 떨어진 접점 T1, T2
4. 중심 O = X에서 각 이등분선 방향으로 r/sin(α)
5. T1→T2 원호 생성, L1·L2는 T1·T2에서 트림
r = 0 이면: 원호 없이 두 선을 X까지 연장/트림 (코너 만들기)
```

```python
def fillet_lines(p11, p12, p21, p22, r):
    X = line_intersection(p11, p12, p21, p22)
    if X is None:
        return None                                  # 평행
    d1 = unit_vector_toward(X, pick_side_1)          # 픽 위치 쪽 방향
    d2 = unit_vector_toward(X, pick_side_2)
    cos2a = d1[0]*d2[0] + d1[1]*d2[1]
    alpha = math.acos(max(-1, min(1, cos2a))) / 2.0
    if r == 0 or alpha < 1e-9:
        return {"corner": X}
    t = r / math.tan(alpha)
    T1 = (X.x() + d1[0]*t, X.y() + d1[1]*t)
    T2 = (X.x() + d2[0]*t, X.y() + d2[1]*t)
    bis = normalize((d1[0]+d2[0], d1[1]+d2[1]))
    O = (X.x() + bis[0]*r/math.sin(alpha),
         X.y() + bis[1]*r/math.sin(alpha))
    Tm = arc_midpoint(O, T1, T2)                     # 짧은 호 쪽
    return {"arc": (T1, Tm, T2), "trim_to": (T1, T2), "center": O, "r": r}
```

원호가 낀 FILLET(선-호, 호-호)은 교차점 대신 오프셋 곡선 교차로 중심을 구한다 — 심화 절로 배치(정식 원고 3쪽).

## 21.4 검수 시나리오 — 도로 편경사 접속부 예제

실무 도면(교차로 가각부 R=15m 모깎기)을 예제 데이터로 제공하고, CAD 결과와 좌표 소수점 4자리까지 일치함을 검증하는 절차를 수록한다.

## 21.5 요약 및 Part VI 마무리

TRIM/EXTEND/FILLET까지 완성되면 "CAD와 동일한 편집"의 최종 관문을 통과한 것이다.

> **[호환성 노트]**
> `lineLocatePoint`·`interpolate`는 3.x~4.x 동일. `curveSubstring`은 3.20+이므로 3.6 폴백 필수(21.2). `Qgis.JoinStyle` 계열 enum은 20장의 compat 상수를 공유한다.

---

# Part VII. 주석 — 문자·치수·해치·블록

---

# Chapter 22. TEXT/MTEXT와 한글 폰트(SHX/TTF) 전략

**(배정: 10p | 난이도: ★★★)**

## 22.1 학습 목표

- TEXT/MTEXT를 회전·정렬·높이 보존형 라벨로 렌더링한다.
- CAD SHX 폰트(굴림체 없는 벡터 폰트) 문제의 실무 대응을 정리한다.

## 22.2 문자 렌더링 모델

8장에서 texts 레이어에 적재한 문자를 **데이터 정의 라벨**로 그린다.

| DXF 필드 | QGIS 라벨 속성 |
|---|---|
| height (코드 40) | 글자 크기 (지도 단위!) |
| rotation (코드 50) | 회전 |
| halign/valign (72/73) | 정렬(Quadrant/Offset) |
| style → font | 폰트 패밀리 매핑 테이블 |

```python
from qgis.core import (QgsPalLayerSettings, QgsVectorLayerSimpleLabeling,
                       QgsTextFormat, QgsUnitTypes, QgsProperty)

def apply_text_labeling(text_layer):
    s = QgsPalLayerSettings()
    s.fieldName = "text_value"
    fmt = QgsTextFormat()
    fmt.setSizeUnit(QgsUnitTypes.RenderMapUnits)     # ★ 축척 연동 핵심
    s.setFormat(fmt)
    s.dataDefinedProperties().setProperty(
        QgsPalLayerSettings.Size, QgsProperty.fromField("height"))
    s.dataDefinedProperties().setProperty(
        QgsPalLayerSettings.LabelRotation,
        QgsProperty.fromExpression('-"rotation"'))   # CAD 반시계 → QGIS 시계
    s.placement = QgsPalLayerSettings.OverPoint
    text_layer.setLabeling(QgsVectorLayerSimpleLabeling(s))
    text_layer.setLabelsEnabled(True)
```

> **WARNING — 라벨 충돌 회피 끄기**
> QGIS 라벨 엔진은 기본적으로 겹치는 라벨을 숨긴다. 도면 문자는 한 글자도 사라지면 안 되므로 `s.displayAll = True`(3.x) / `Qgis.LabelOverlapHandling.AllowOverlapIfRequired`(4.x)를 설정한다 — 도면 검수에서 가장 먼저 발견되는 사고 유형이다.

## 22.3 MTEXT 서식 코드 해석

MTEXT 본문에는 `\P`(줄바꿈), `\f굴림;`(폰트), `{\H2.5x;…}`(높이 배율) 등 인라인 코드가 섞여 있다. ezdxf의 `MTextParser`로 평문+스타일 런으로 분해하고, 초안 단계에서는 **평문 추출 + 첫 줄 스타일 적용**을 기본 정책으로 한다(완전 서식 재현은 심화 절).

## 22.4 SHX 폰트 문제

- CAD 도면의 한글은 대부분 whgtxt.shx 등 **SHX 벡터 폰트**로 작성되며, 이는 TTF가 아니어서 OS에 설치할 수 없다.
- 대응 전략 3단계: ① 스타일 테이블의 SHX 이름 → 유사 TTF 매핑 테이블(맑은 고딕 등) 기본 적용 ② 매핑 테이블 사용자 편집 UI 제공 ③ 완벽 재현이 필요하면 문자를 윤곽 폴리선으로 전개(TXTEXP 상당) — 각 전략의 품질/비용 비교표 수록.

## 22.5 TEXT 작도 명령

`DTEXT` 명령: 삽입점 → 높이 → 회전 → 문자 입력(명령행) → texts 레이어에 추가. 17장 패턴의 반복이므로 지면 2쪽.

> **[호환성 노트]**
> `displayAll`(3.x)→`LabelOverlapHandling`(4.x) 분기, `QgsPalLayerSettings.Size` 프로퍼티 키의 `Qgis.LabelProperty` 이관(4.x)을 compat로 흡수한다.

---

# Chapter 23. 치수(DIMENSION) 시스템

**(배정: 11p | 난이도: ★★★★★)**

## 23.1 학습 목표

- 치수 스타일(DIMSTYLE) 모델을 정의하고 선형/정렬/각도/반지름 치수를 생성한다.
- 기존 DXF의 DIMENSION 엔티티(익명 블록)를 읽어 표시한다.

## 23.2 치수의 이중 표현 문제

DXF의 DIMENSION은 ① 파라미터(측정점·문자위치·스타일)와 ② 렌더링 결과(익명 블록 `*D123`) 를 **모두** 저장한다. 전략:

```text
읽기: 익명 블록을 전개해 그대로 표시 (원본 충실도 100%)
생성: 파라미터로부터 구성요소(치수선·보조선·화살표·문자)를 자체 렌더링
저장: 파라미터 + 자체 렌더링 블록을 함께 기록 (25장)
```

## 23.3 선형 치수 렌더링 기하

**[그림 23-1] 선형 치수 구성요소: 치수보조선(offset/extension), 치수선, 화살표, 문자 (도판 삽입 예정)**

```python
# commands/annotate/dim_linear.py — 기하 계산부
def build_linear_dim(p1, p2, dim_line_y_offset, style):
    """수평 치수의 구성요소 지오메트리 생성."""
    y = dim_line_y_offset
    comps = []
    # 치수보조선 (원점 이격 ext_offset, 초과 ext_beyond)
    for px in (p1, p2):
        comps.append(("ext_line", [
            (px.x(), px.y() + sign(y)*style.ext_offset),
            (px.x(), y + sign(y)*style.ext_beyond)]))
    # 치수선
    comps.append(("dim_line", [(p1.x(), y), (p2.x(), y)]))
    # 화살표 (닫힌 채움 삼각형, 크기 style.arrow_size)
    comps.append(("arrow", arrow_triangle((p1.x(), y), direction=+1, style=style)))
    comps.append(("arrow", arrow_triangle((p2.x(), y), direction=-1, style=style)))
    # 문자: 실측값 × style.scale_factor, 소수자리 style.decimals
    value = abs(p2.x() - p1.x()) * style.measurement_scale
    comps.append(("text", ((p1.x()+p2.x())/2, y + style.text_gap),
                  f"{value:.{style.decimals}f}"))
    return comps
```

구성요소는 전용 `QCAD|dims` 레이어(lines+polygon+text 통합 규칙 렌더러)에 기록하고, 각 피처의 `extra`에 부모 치수 핸들을 저장해 **치수 단위 선택/삭제**가 되게 한다.

## 23.4 연관 치수(associativity)의 범위 결정

CAD의 완전 연관 치수(측정 대상이 바뀌면 자동 갱신)는 구현 비용이 매우 크다. 본 교재는 **"반연관"** — 그립으로 측정점을 옮기면 재계산, 대상 객체 수정에는 비연동 — 을 목표 사양으로 명시한다. (사양 결정 근거와 CAD 제품별 비교 1쪽)

## 23.5 각도/반지름/정렬 치수

- 정렬(ALIGNED): 선형 치수를 두 점 방향으로 회전한 것 — 19장의 회전 헬퍼 재사용.
- 반지름(RADIUS): `extra.center/radius` 회수(또 한 번의 설계 배당금), 지시선+`R{값}` 문자.
- 각도(ANGULAR): 두 선의 교차점 중심 호 + 도분초 표기 옵션(측량 관행 °′″) — 국내 실무 차별화 포인트.

## 23.6 DIMSTYLE 관리 UI

치수 스타일(화살표 크기·문자 높이·소수자리·측정 배율)을 저장/편집하는 다이얼로그. `measurement_scale`은 축척 도면(1:1000 도곽에 실측 기입) 관행을 지원하는 필수 항목이다.

> **[호환성 노트]**
> 전부 자체 기하 계산이라 버전 이슈가 거의 없다. 유일한 주의점은 규칙 렌더러의 채움 화살표(Polygon) simple fill이 3.6에서 안티앨리어싱 품질이 낮다는 점 — 렌더링 옵션 안내로 처리.

---

# Chapter 24. HATCH와 블록(BLOCK/INSERT/ATTRIB)

**(배정: 11p | 난이도: ★★★★)**

## 24.1 HATCH 읽기 — 경계 재구성

HATCH는 경계 루프(폴리선/엣지 목록)와 패턴(이름·각도·간격)으로 구성된다.

```python
def conv_hatch(self, e):
    from ezdxf.entities import Hatch
    rings = []
    for path in e.paths:
        if path.path_type_flags & 2:            # polyline path
            pts = [(v[0], v[1]) for v in path.vertices]
            rings.append(close_ring(pts))
        else:                                   # edge path: line/arc/ellipse/spline
            rings.append(edges_to_ring(path.edges))   # 원호 엣지는 8.5 로직 재사용
    geom = polygon_from_rings(rings)            # 외곽/구멍 방향 판정 포함
    extra = {"pattern": e.dxf.pattern_name,
             "angle": e.dxf.pattern_angle,
             "scale": e.dxf.pattern_scale,
             "solid": bool(e.dxf.solid_fill)}
    return "polygons", geom, self._attrs(e, extra)
```

렌더링: SOLID는 단순 채움, 패턴(ANSI31 등)은 **QgsLinePatternFillSymbolLayer**(각도·간격을 데이터 정의로)로 근사한다. 패턴명→선패턴 파라미터 변환표를 resources/patterns.json으로 제공한다.

## 24.2 HATCH 작도 명령

경계 자동 탐지(내부 점 클릭 → 주변 폐합영역 탐색)는 난도가 높아, 초안 사양은 **객체 선택 방식**(폐합 폴리선/원 선택 → 패턴 지정)으로 한정하고, 내부점 방식은 `QgsGeometry.polygonize` 기반 심화 절로 배치한다.

## 24.3 블록 정의와 INSERT 렌더링

전략 비교 후 채택:

| 전략 | 장점 | 단점 |
|---|---|---|
| A. 로드 시 전개(explode) | 구현 단순 | 왕복 시 블록 소실, 데이터 폭증 |
| B. 삽입점만 표시 + 마커 | 가볍다 | 도면이 안 보임 |
| **C. 정의 1회 전개 캐시 + 삽입 변환 복제** | 원본 보존 + 표시 정확 | 구현 복잡 |

```python
# core/blocks.py — 전략 C
class BlockDefinition:
    def __init__(self, name, entities):        # 전개된 (etype, geom, attrs) 목록
        self.name, self.entities = name, entities

def render_insert(doc, insert_feat):
    """INSERT 피처 1건 → 변환된 표시용 피처들 생성."""
    extra = json.loads(insert_feat["extra"])
    bdef = doc.blocks[extra["block"]]
    for etype, geom, attrs in bdef.entities:
        g = QgsGeometry(geom)
        g.transform_affine(scale=extra["sx"], rotate=extra["rot"],
                           translate=extra["ins_pt"])       # 19장 아핀 헬퍼
        aci = attrs["aci"]
        if aci == 0:                                        # ByBlock 해석 (9.2)
            aci = insert_feat["aci"]
        doc.add_display_feature(etype, g, aci, parent=insert_feat["handle"])
```

표시용 피처는 `QCAD|_block_display` 임시 레이어에 넣고 원본 inserts 피처와 parent 핸들로 연결한다 — 블록 MOVE 시 표시 피처를 일괄 재생성한다.

## 24.4 속성 블록(ATTRIB)

측점번호·관경 라벨 등 실무 심볼의 핵심. ATTDEF(정의)/ATTRIB(값)를 읽어 표시 문자로 렌더링하고, 삽입 시 값 입력 다이얼로그를 띄운다. 속성값 일괄 추출(→ CSV) 유틸리티를 실습으로 포함 — **"도면에서 수량 뽑기"** 라는 실무 최다 요청 기능이다.

## 24.5 요약 및 Part VII 마무리

주석 3종(문자·치수·해치)과 블록까지, 도면을 "보이는 그대로" 재현하는 층이 완성됐다.

> **[호환성 노트]**
> `QgsLinePatternFillSymbolLayer`는 3.x~4.x 존재. 회전 데이터 정의 키 이름이 3.30 전후로 정리되었으므로 compat에 상수화한다.

---

# Part VIII. 내보내기·출력·상호운용

---

# Chapter 25. QGIS → DXF 내보내기 엔진 (QgsDxfExport + ezdxf 후처리)

**(배정: 10p | 난이도: ★★★★ | 핵심)**

## 25.1 학습 목표

- 왕복(round-trip) 무손실 저장의 3단 전략(차등 저장 / 전체 재작성 / QgsDxfExport)을 구현한다.
- CAD에서 다시 열었을 때 레이어·색·선종류·블록·치수가 유지됨을 검증한다.

## 25.2 세 가지 저장 경로와 선택 기준

| 경로 | 방식 | 사용 시점 |
|---|---|---|
| **① 차등 저장 (기본)** | 원본 DXF를 ezdxf로 다시 열고, handle이 있는 수정 피처만 해당 엔티티 교체·신규 추가·삭제 반영 | 원본 DXF에서 출발한 문서 |
| ② 전체 재작성 | CadDocument 전체를 새 DXF 문서로 직렬화 | 신규 작성 도면, R12 강등 저장 |
| ③ QgsDxfExport | 일반 QGIS 레이어(SHP 등)를 DXF로 | GIS 데이터 → CAD 납품 (26장) |

## 25.3 차등 저장 구현

```python
# io/dxf_writer.py
import ezdxf, json

class DiffWriter:
    def __init__(self, doc):
        self.doc = doc

    def save(self, out_path):
        src = ezdxf.readfile(self.doc.source_path)
        msp = src.modelspace()
        by_handle = {e.dxf.handle: e for e in msp}

        for key, vl in self.doc.qgis_layers.items():
            for f in vl.getFeatures():
                h = f["handle"]
                if h.startswith("NEW:"):
                    self._append_entity(src, msp, f)          # 신규
                elif h in self.doc.dirty_handles:
                    old = by_handle.get(h)
                    if old is not None:
                        msp.delete_entity(old)
                    self._append_entity(src, msp, f, handle=h) # 교체
        for h in self.doc.deleted_handles:
            if h in by_handle:
                msp.delete_entity(by_handle[h])

        self._sync_layer_table(src)        # 레이어 색/상태 변경 반영 (Ch.11)
        src.saveas(out_path)
```

`dirty_handles`는 18~21장의 편집 명령이 `changeGeometryValues` 직후 등록한다 — undo 스택과 같은 지점에 훅을 두면 누락이 없다.

## 25.4 지오메트리 → 엔티티 역변환

8장의 정변환을 거울로 뒤집는다. 핵심은 **CircularString → 벌지 역산**이다.

```python
def circularstring_to_bulge(p1, pm, p2):
    """호의 3점에서 벌지 계산: b = tan(θ/4), 부호는 진행방향."""
    c, r = circle_from_3pts(p1, pm, p2)          # 외접원
    a1 = math.atan2(p1[1]-c[1], p1[0]-c[0])
    a2 = math.atan2(p2[1]-c[1], p2[0]-c[0])
    sweep = normalize_sweep(a1, a2, via=pm, center=c)   # 통과점 쪽 각
    return math.tan(sweep / 4.0)

def write_lwpolyline(msp, feat):
    verts, bulges = decompose_compound_curve(feat.geometry())
    pts = [(v[0], v[1], 0, 0, b) for v, b in zip(verts, bulges)]
    e = msp.add_lwpolyline(pts, format="xyseb",
                           dxfattribs=base_attribs(feat))
    extra = json.loads(feat["extra"] or "{}")
    e.closed = extra.get("closed", False)
    return e

def base_attribs(feat):
    return {"layer": feat["layer"], "color": feat["aci"],
            "linetype": feat["ltype"], "lineweight": feat["lweight"]}
```

## 25.5 R12 강등 저장

```text
MTEXT   → 줄 단위 TEXT 분해 (22.3의 파서 재사용)
LWPOLYLINE → POLYLINE/VERTEX 시퀀스
SPLINE  → 폴리선 세그먼트화
HATCH   → 경계 폴리선 + (옵션) 패턴 선 전개
한글    → CP949 인코딩 저장 ($DWGCODEPAGE=ANSI_949)
```

ezdxf `saveas` 전에 `doc.dxfversion = "AC1009"` 지정과 함께 위 변환 파이프를 통과시키는 `R12Downgrader` 클래스로 구현한다(정식 원고 3쪽).

## 25.6 왕복 검증 자동화 — 본 장의 백미

```python
# tests/test_roundtrip.py
def test_roundtrip_preserves_structure(sample_dxf, tmp_path):
    doc1 = load(sample_dxf)                    # 열기
    save(doc1, tmp_path / "out.dxf")           # 무편집 저장
    d1, d2 = ezdxf.readfile(sample_dxf), ezdxf.readfile(tmp_path / "out.dxf")

    assert layer_table_sig(d1) == layer_table_sig(d2)     # 레이어·색·선종류
    assert entity_census(d1) == entity_census(d2)         # 타입별 개수
    assert extents(d1) == pytest.approx(extents(d2), abs=1e-6)
```

"무편집 왕복 후 diff 제로"를 CI 게이트(30장)로 삼는다. 이것이 이 플러그인의 품질을 한 문장으로 증명하는 지표다.

> **[호환성 노트]**
> 이 장은 ezdxf 단독 영역이라 QGIS 버전 이슈가 없다. 단 3.6 동봉 ezdxf 0.17.x에는 `format="xyseb"` 인자 표기가 다르므로(문자 순서 동일, 키워드명 `format`) 버전 가드를 둔다.

---

# Chapter 26. 레이어 매핑 테이블과 납품 표준

**(배정: 8p | 난이도: ★★☆ | 국내 실무 차별화 장)**

## 26.1 학습 목표

- SHP/GPKG 등 일반 GIS 레이어를 **발주처 CAD 레이어 표준**에 맞춰 DXF로 내보낸다.
- 매핑 테이블(CSV) 기반의 재사용 가능한 변환 프로파일을 만든다.

## 26.2 매핑 테이블 설계

```csv
# mapping_road.csv — GIS 필드 조건 → CAD 레이어/색/선종류
src_layer, filter_expr,            cad_layer,     aci, ltype,      lweight
도로중심선, "등급"='주간선',        C-ROAD-CTR-1,  1,   CENTER,     35
도로중심선, "등급"='보조간선',      C-ROAD-CTR-2,  2,   CENTER,     25
용지경계,  ,                       C-LAND-BNDY,   3,   CONTINUOUS, 18
등고선,    "주곡선"=1,             C-TOPO-MAJR,   8,   CONTINUOUS, 13
등고선,    "주곡선"=0,             C-TOPO-MINR,   9,   CONTINUOUS, 9
```

## 26.3 매핑 실행기

```python
def export_with_mapping(project_layers, mapping_rows, out_path):
    doc = ezdxf.new("AC1027", setup=True)      # 표준 선종류 포함
    msp = doc.modelspace()
    for row in mapping_rows:
        vl = project_layers[row.src_layer]
        ensure_cad_layer(doc, row)             # LAYER 테이블 레코드 생성
        req = QgsFeatureRequest()
        if row.filter_expr:
            req.setFilterExpression(row.filter_expr)
        for f in vl.getFeatures(req):
            write_geometry(msp, f.geometry(),
                           dxfattribs={"layer": row.cad_layer, "color": 256})
    doc.saveas(out_path)
```

`QgsDxfExport`를 쓰지 않고 ezdxf 직접 기록을 택한 이유(선종류 테이블 완전 제어, 폐합 폴리선 보장, 한글 레이어명 인코딩 제어)를 본문 비교표로 제시한다.

## 26.4 매핑 프로파일 UI와 검증 리포트

- 프로파일(도로/하천/단지/지적) 저장·불러오기 콤보 + 테이블 편집 위젯.
- 내보내기 후 검증 리포트: 매핑 누락 피처 수, 빈 레이어, 지오메트리 붕괴(0길이) 목록을 HTML로 출력 — 납품 검수 대응.

> **집필 메모**
> 발주 기관별 레이어 표준(국토부 건설CALS 도면작성기준 등)의 실제 코드표는 저작권·개정 문제가 있으므로, 교재에는 "구조와 예시"만 싣고 실제 표준표는 독자가 CSV로 작성하도록 안내한다. 부록에 CALS 기준 문서의 공식 출처 URL만 수록.

> **[호환성 노트]**
> `setup=True`는 ezdxf가 표준 선종류·문자 스타일을 자동 생성하는 옵션으로 전 버전 동일하다.

---

# Chapter 27. 도곽·축척 출력과 DWG 상호운용 전략

**(배정: 8p | 난이도: ★★★)**

## 27.1 도곽(제목 블록) 출력

- 도곽을 속성 블록(24장)으로 정의: 도면번호·과업명·축척·작성일 ATTRIB.
- 출력 명령 `PLOTFRAME`: 축척 선택(1:500~1:6000) → 캔버스에 도곽 미리보기 러버밴드 → 배치 확정 → QGIS 인쇄 레이아웃(QgsLayoutManager) 자동 생성 → PDF 출력.

```python
from qgis.core import QgsLayout, QgsLayoutItemMap, QgsLayoutSize, QgsProject

def make_plot_layout(extent, scale, paper="A1"):
    layout = QgsLayout(QgsProject.instance())
    layout.initializeDefaults()
    item = QgsLayoutItemMap(layout)
    item.setExtent(extent)
    item.setScale(scale)
    layout.addLayoutItem(item)
    return layout
```

## 27.2 로컬 좌표 도면의 지오레퍼런싱 (3.3절 시나리오 B의 해결)

2점 대응 Helmert(등각) 변환 도구:

```text
1. 도면상 기준점 2점 픽 (OSNAP 활용)
2. 실좌표 2점 입력 (또는 지도에서 픽)
3. 축척 s, 회전 θ, 이동 (tx,ty) 산출 → 5개 레이어 전체에 아핀 적용
검증: 잔차(residual) 표시, 3점 이상 입력 시 최소제곱
```

19장의 아핀 헬퍼와 15장의 좌표 입력이 그대로 재사용된다 — 교재 전체 설계가 수렴하는 지점으로 서술한다.

## 27.3 DWG 상호운용 — 현실적 선택지

| 방법 | 라이선스/비용 | 권장도 |
|---|---|---|
| ODA File Converter (DWG↔DXF) 외부 호출 | 무료 배포(고지 필요) | ★★★★ 기본 채택 |
| GDAL DWG 빌드(Teigha) | 상용 SDK 필요 | ★ |
| LibreDXF/기타 | 성숙도 낮음 | ★★ |

플러그인 설정에 ODA 변환기 실행파일 경로를 등록하면 "DWG 열기"가 임시 DXF 변환을 경유해 투명하게 동작하도록 `QProcess` 래퍼를 구현한다. 재배포 금지·사용자 직접 설치 안내 등 라이선스 준수 절차를 명확히 기술한다.

## 27.4 요약 및 Part VIII 마무리

열기-편집-저장-납품-출력의 전체 수명주기가 닫혔다. 남은 것은 품질과 배포다.

> **[호환성 노트]**
> `QgsLayout` API는 3.x~4.x 동일. `QProcess` 시그널 `finished(int)` → `finished(int, ExitStatus)` 시그니처가 Qt6에서 엄격해졌으므로 람다 수신부에 두 인자를 받는다.

---

# Part IX. 품질·테스트·배포

---

# Chapter 28. 플러그인 아키텍처 리팩터링과 테스트

**(배정: 7p | 난이도: ★★★)**

## 28.1 최종 아키텍처 점검

**[그림 28-1] 최종 의존 그래프: ui → commands → core/tools → io, compat는 전역 (도판 삽입 예정)**

- 순환 의존 금지 규칙: `io`는 `qgis.core`와 ezdxf만, `core`는 QGIS만 안다.
- 명령 26종의 코드 중복률 측정(초안 목표 < 15%)과 베이스 클래스 추출 회고.

## 28.2 테스트 3층 구조

```text
1층. 순수 함수 테스트 (QGIS 불필요)  — 벌지 수학, 파서, FILLET 기하, 매핑
2층. QGIS 헤드리스 테스트            — 변환 엔진, 렌더러, 왕복(Ch.25)
3층. 수동 검수 시나리오              — 17.6/21.4 체크리스트, 실도면 5종
```

```python
# tests/conftest.py — 헤드리스 QGIS 부팅
import pytest
from qgis.core import QgsApplication

@pytest.fixture(scope="session")
def qgis_app():
    app = QgsApplication([], False)
    app.initQgis()
    yield app
    app.exitQgis()
```

```python
# tests/test_geometry_utils.py — 1층 예시
import math
from qcad_bridge.core.geometry_utils import bulge_mid, fillet_lines

def test_bulge_semicircle():
    # 벌지 1.0 = 반원: 중간점은 현 중점에서 반지름만큼 이격
    m = bulge_mid((0, 0), (10, 0), 1.0)
    assert m == pytest.approx((5.0, 5.0))

def test_fillet_right_angle_r5():
    r = fillet_lines((0,0), (10,0), (10,0), (10,10), 5)
    assert r["center"] == pytest.approx((5.0, 5.0), abs=1e-9)
```

## 28.3 테스트 데이터셋

`tests/data/`에 등급별 도면 5종을 동봉한다: ① 최소(엔티티 10개) ② 곡선부 도로(벌지 밀집) ③ 한글 R12(CP949) ④ 블록·치수 밀집 ⑤ 10만 엔티티 성능용(스크립트 생성).

> **[호환성 노트]**
> 헤드리스 부팅은 3.6~4.x 동일하나 Linux CI에서 `QT_QPA_PLATFORM=offscreen` 환경변수가 필요하다.

---

# Chapter 29. 3.6 ~ 4.x 교차 검증과 pyqgis4-checker

**(배정: 6p | 난이도: ★★☆)**

## 29.1 정적 검사

```bash
# Docker로 pyqgis4-checker 실행 (QGIS 4 호환성 위반 검출)
docker run --rm -v $(pwd):/src qgis/pyqgis4-checker:latest \
  pyqgis4-check /src/qcad_bridge
```

- 검출 항목: PyQt5 직접 import, 비호환 enum, 제거된 API.
- 목표: **위반 0건, 예외는 compat.py 1파일에만 허용** (5장 정책의 검증).

## 29.2 3계열 매트릭스 CI

| 잡 | 이미지 | 실행 |
|---|---|---|
| qgis-3.6 | qgis/qgis:release-3_6 | 1·2층 테스트 (ezdxf 0.17 경로) |
| qgis-ltr | qgis/qgis:ltr | 전체 테스트 + 왕복 diff 게이트 |
| qgis-4 | qgis/qgis:latest | 전체 테스트 + pyqgis4-checker |

## 29.3 버전별 알려진 차이 대장(臺帳)

교재 전체의 [호환성 노트]를 표 하나로 집계한 "차이 대장"을 이 장에 수록한다 — 독자가 유지보수 시 첫 페이지로 펼치는 표. (초안: 항목 14건 집계 예정)

> **[호환성 노트]**
> 이 장 자체가 집계 장이다.

---

# Chapter 30. 패키징·플러그인 저장소 배포·CI/CD

**(배정: 7p | 난이도: ★★☆)**

## 30.1 ext_libs 동봉 패키징

```python
# scripts/build_package.py — ezdxf vendoring 포함 ZIP 생성
TARGETS = {
    "qgis3": {"ezdxf": "0.17.2", "python": "3.7"},
    "qgis4": {"ezdxf": "latest", "python": "3.12"},
}
# pip install --target ext_libs/ ; zip 구조 검증 ; metadata 버전 주입
```

`__init__.py` 최상단에서 `ext_libs`를 `sys.path`에 삽입하는 부트스트랩과, 이미 설치된 ezdxf와의 버전 충돌 회피 규칙을 기술한다.

## 30.2 qgis-plugin-ci 릴리스 파이프라인

```text
git tag v0.1.0
  ↓ GitHub Actions
ruff + pytest (3계열 매트릭스, Ch.29)
  ↓
qgis-plugin-ci package / release
  ↓
GitHub Release + QGIS Plugin Repository 업로드
```

## 30.3 저장소 심사 대비 체크리스트

- [ ] metadata.txt: qgisMinimumVersion=3.6, supportsQt6=True, changelog
- [ ] 외부 의존(ezdxf) 고지 및 라이선스(MIT) 동봉
- [ ] ODA 변환기 미동봉·경로 설정 방식 설명 (27.3)
- [ ] 실험 기능 플래그 분리, 개인정보/네트워크 접근 없음 명시
- [ ] 왕복 diff 게이트 통과 로그 첨부

## 30.4 교재 총정리 — QCAD-Bridge 기능 지도

1.2절의 요구사항 12항목 표를 다시 가져와 구현 장과 검수 결과를 매핑한 최종 표로 책을 닫는다. "CAD와 동일한가"라는 처음의 질문에, 세 판정 기준(키 입력·스냅·왕복 무손실)의 충족으로 답한다.

---

# 부록

## 부록 A. AutoCAD 명령 ↔ QCAD-Bridge 대응표 (발췌)

| AutoCAD | 별칭 | QCAD-Bridge | 구현 장 |
|---|---|---|---|
| LINE | L | line | 17 |
| PLINE | PL | pline | 17 |
| CIRCLE | C | circle | 17 |
| ARC | A | arc | 17 |
| MOVE | M | move | 19 |
| COPY | CO/CP | copy | 19 |
| ROTATE | RO | rotate | 19 |
| MIRROR | MI | mirror | 19 |
| SCALE | SC | scale | 19 |
| OFFSET | O | offset | 20 |
| ARRAY | AR | array | 20 |
| EXPLODE | X | explode | 20 |
| TRIM | TR | trim | 21 |
| EXTEND | EX | extend | 21 |
| FILLET | F | fillet | 21 |
| DTEXT | DT | dtext | 22 |
| DIMLINEAR | DLI | dimlinear | 23 |
| HATCH | H | hatch | 24 |
| INSERT | I | insert | 24 |
| U / UNDO | U | u | 18 |

(정식 원고: 전체 42개 명령 수록)

## 부록 B. DXF 그룹 코드 요약표

(2.2절 표 확장 — 정식 원고에서 코드 0~1071 중 실무 빈출 60개 수록)

## 부록 C. ACI ↔ RGB 변환표 (발췌)

| ACI | RGB | 관행 용도 |
|---:|---|---|
| 1 | 255,0,0 | 중심선·주요 계획선 |
| 2 | 255,255,0 | 보조선 |
| 3 | 0,255,0 | 경계 |
| 4 | 0,255,255 | 수계 |
| 5 | 0,0,255 | 구조물 |
| 6 | 255,0,255 | 참고선 |
| 7 | 255,255,255(흑백 반전) | 일반 |
| 8 | 128,128,128 | 지형 보조 |

(전체 255색은 `resources/aci_palette.json` 및 정식 원고 부록 4쪽)

## 부록 D. Qt5/Qt6 enum 대응표

(5장 compat.py 상수의 근거 표 — Key/MouseButton/PenStyle/CursorShape/ItemDataRole 5계열, 정식 원고 3쪽)

## 부록 E. 참고문헌·공식 문서 소스맵

- PyQGIS Developer Cookbook (4.2/3.44ko) — 사전조사 문서 §2·§3의 URL 체계 준용
- ezdxf documentation: https://ezdxf.readthedocs.io/
- DXF Reference (Autodesk 공개 문서)
- QGIS API: https://qgis.org/pyqgis/ , https://api.qgis.org/
- ODA File Converter: https://www.opendesign.com/guestfiles/oda_file_converter
- 인용 관리: 사전조사 문서 §33 YAML 스키마 준용 (`references/sources.yaml`)

---

# 초안 후기 — 집필 진행 관리표

| 상태 | 의미 | 해당 |
|---|---|---|
| ✅ 본문 초안 완료 | 지면 그대로 확장 가능 | Ch.1~21, 25 |
| 🔶 골격+핵심 코드 | 정식 원고에서 1.5~3쪽/절 확장 필요 (집필 메모 표기) | Ch.22~24, 26~30 |
| 📷 도판 | [그림 N-M] 플레이스홀더 14점 — QGIS 실캡처 후 교체 | 전체 |
| 🧪 코드 검증 | QGIS 4.2 / 3.44 / 3.6 3계열 실행 검증 전 (v0.2 과제) | 전체 |

**v0.2 예정 작업:** ① 3계열 실행 검증 및 [호환성 노트] 실측 갱신 ② 집필 메모 25건 해소 ③ 예제 저장소(`examples/qcad_bridge`) 코드 완성 ④ 도판 캡처 ⑤ 연습문제(장당 3~5문) 추가.

**End of Draft v0.1**

# Part VIII. 내보내기·출력·상호운용

---

# Chapter 25. QGIS → DXF 내보내기 엔진 (QgsDxfExport + ezdxf 후처리)

**(배정: 10p | 난이도: ★★★★ | 핵심)**

## 25.1 학습 목표

- 왕복(round-trip) 무손실 저장: 원본 DXF에 변경분만 반영하는 **차등 저장**을 구현한다.
- 일반 QGIS 레이어(SHP/GPKG 분석 결과)를 CAD 레이어 체계로 내보내는 **신규 작성** 경로를 구현한다.

## 25.2 두 가지 저장 경로

```text
경로 1. 차등 저장 (QCAD 도면 → 원본 DXF 갱신)     ← 본 장의 주인공
  원본 doc(ezdxf) 유지
  + handle 필드로 수정/삭제 엔티티 대조
  + NEW: 핸들 피처는 신규 엔티티로 추가
  → doc.saveas()  … 블록·치수·레이아웃 등 미변경 요소 100% 보존

경로 2. 신규 작성 (임의 QGIS 레이어 → 새 DXF)
  ezdxf.new() + 레이어 매핑 테이블(Ch.26) 적용
  (QgsDxfExport는 폴백/비교 검증용으로만 사용)
```

## 25.3 차등 저장 구현

```python
# io/dxf_writer.py
import ezdxf, json

class DifferentialWriter:
    def __init__(self, document):
        self.doc = document                 # CadDocument
        self.dxf = document.source_doc      # 로드 때 보관한 ezdxf Drawing

    def save(self, path, version=None):
        msp = self.dxf.modelspace()
        handles = {e.dxf.handle: e for e in msp}

        for key, vl in self.doc.qgis_layers.items():
            for f in vl.getFeatures():
                h = f["handle"]
                if h.startswith("NEW:"):
                    self._append_entity(msp, f)          # 신규
                elif h in handles and self.doc.is_modified(h):
                    self._rewrite_entity(handles[h], f)  # 수정: 파라미터 갱신
        for h in self.doc.deleted_handles:
            if h in handles:
                msp.delete_entity(handles[h])

        if version:
            # 하위 버전 저장: R12 강등 규칙 적용 (25.5)
            self._downgrade(version)
        self.dxf.saveas(path)

    def _append_entity(self, msp, f):
        etype = f["etype"]; extra = json.loads(f["extra"] or "{}")
        attribs = {"layer": f["layer"], "color": f["aci"],
                   "linetype": f["ltype"]}
        if etype == "LINE":
            p = f.geometry().asPolyline()
            msp.add_line(p[0], p[-1], dxfattribs=attribs)
        elif etype == "LWPOLYLINE":
            msp.add_lwpolyline(self._to_xyb(f), format="xyb",
                               close=extra.get("closed", False),
                               dxfattribs=attribs)
        elif etype == "CIRCLE":
            msp.add_circle(extra["center"], extra["radius"], dxfattribs=attribs)
        elif etype == "ARC":
            msp.add_arc(extra["center"], extra["radius"],
                        extra["a1"], extra["a2"], dxfattribs=attribs)
        # TEXT/INSERT/HATCH/DIM … (22~24장의 파라미터 역직렬화)

    def _to_xyb(self, f):
        """QGIS 곡선 → (x, y, bulge) 목록 — 8.4의 역변환."""
        verts, bulges = decompose_compound_curve(f.geometry())
        return [(v.x(), v.y(), b) for v, b in zip(verts, bulges)]
```

> **ENGINEERING PRACTICE — 역변환 검증 루프**
> `bulge → 원호 → bulge` 왕복 시 부동소수 오차로 벌지가 미세하게 변한다. 저장 직후 파일을 다시 읽어 원본과 좌표 RMS를 비교하는 `roundtrip_check()`를 저장 다이얼로그의 "검증" 버튼으로 제공한다. 납품 도면 신뢰성의 결정적 기능이다.

## 25.4 신규 작성 경로 — 분석 결과의 도면화

```python
def export_qgis_layer(vlayer, dxf_doc, mapping):
    """임의 QGIS 벡터 레이어 → CAD 레이어 체계로 기록."""
    msp = dxf_doc.modelspace()
    cad_layer = mapping.layer_for(vlayer)            # Ch.26
    if cad_layer.name not in dxf_doc.layers:
        dxf_doc.layers.add(cad_layer.name, color=cad_layer.aci,
                           linetype=cad_layer.ltype)
    for f in vlayer.getFeatures():
        g = f.geometry()
        for part in flatten(g):
            if part.type() == 0:      # Point
                msp.add_point(part.asPoint(), dxfattribs={"layer": cad_layer.name})
            else:
                msp.add_lwpolyline(
                    [(p.x(), p.y()) for p in part.asPolyline()],
                    dxfattribs={"layer": cad_layer.name})
```

폴리곤은 외곽/내곽 링을 각각 폐합 LWPOLYLINE으로, 필요시 HATCH 동반 기록 옵션을 제공한다.

## 25.5 R12 강등 규칙

| 2013 요소 | R12 변환 |
|---|---|
| LWPOLYLINE | POLYLINE/VERTEX/SEQEND |
| MTEXT | 줄 단위 TEXT 분해 |
| HATCH | 경계 폴리선 + SOLID 근사 (또는 생략 경고) |
| 유니코드 레이어명 | CP949 or `\U+` 이스케이프 |
| SPLINE | 폴리선 세그먼트화 |

ezdxf는 `saveas`에서 자동 강등하지 않으므로, 위 규칙을 명시적 전처리 함수로 구현한다(정식 원고 3쪽).

## 25.6 요약

- handle 기반 차등 저장으로 "CAD에서 다시 열어도 그대로"라는 1장 판정 기준 ③을 달성했다.

> **[호환성 노트]**
> 본 장은 ezdxf 중심이라 QGIS 버전 무관. 단 3.6(ezdxf 0.17.x 동봉) 환경에서 `add_lwpolyline(format=)` 인자 형식이 동일함을 tests로 고정한다(29장).

---

# Chapter 26. 레이어 매핑 테이블과 납품 표준

**(배정: 7p | 난이도: ★★☆)**

## 26.1 학습 목표

- QGIS 레이어/필드값 → CAD 레이어·색·선종류 매핑 규칙 테이블을 설계한다.
- 기관 납품 레이어 코드 체계를 프리셋으로 제공한다.

## 26.2 매핑 규칙 모델

```yaml
# resources/mappings/road_design.yaml (예시 프리셋)
name: 도로설계 납품 표준(예시)
rules:
  - match: {layer: "centerline*"}
    cad: {layer: "도로-중심선", aci: 1, ltype: "CENTER", lweight: 30}
  - match: {layer: "row", attr: {type: "계획"}}
    cad: {layer: "도로-계획경계", aci: 3, ltype: "CONTINUOUS"}
  - match: {geometry: "polygon"}
    cad: {layer: "부지-경계", aci: 8, hatch: "ANSI31"}
default:
  cad: {layer: "0", aci: 7}
```

매핑 편집 다이얼로그(테이블 위젯 + YAML 가져오기/내보내기)와, 규칙 적용 미리보기(대상 피처 수 카운트)를 구현한다. 실무에서는 발주처마다 코드가 다르므로 **프리셋 공유 가능성**이 도입 결정 요인이 된다.

> **집필 메모**
> 국내 기관별 실제 레이어 코드(국토부 건설CALS 도면작성기준 등)는 출판 시점 최신 고시를 확인해 부록 표로 수록하되, 본문 예시는 가상 코드로 유지한다(개정 대응).

## 26.3 요약

- 매핑 테이블은 25장의 신규 작성 경로와 결합해 "분석 → 납품 도면" 원클릭 파이프라인을 완성한다.

> **[호환성 노트]**
> 순수 Python(YAML은 표준 번들 외 라이브러리이므로 ruamel 대신 json 병행 지원) — 버전 이슈 없음.

---

# Chapter 27. 도곽·축척 출력과 DWG 상호운용 전략

**(배정: 7p | 난이도: ★★☆)**

## 27.1 도곽 배치와 출력

- 도곽(A1/A3 표준 프레임 블록) 삽입 → 축척(1:500/1:1000/1:1200) 지정 → `QgsLayoutManager`로 인쇄 레이아웃 자동 생성(지도 항목 extent = 도곽 범위, 축척 고정) → PDF 일괄 출력.
- 축척 종속 표현: 문자 높이·대시 패턴이 지도 단위이므로(22장·9.4절) 어떤 축척에서도 CAD와 동일 비율로 인쇄된다 — 이 설계의 최종 회수 지점.

## 27.2 로컬 좌표 도면의 지오레퍼런싱 (시나리오 B)

2점 대응 Helmert 변환(이동+회전+축척) 도구:

```python
def helmert_from_two_points(src1, src2, dst1, dst2):
    """도면 2점 ↔ 실좌표 2점 → (scale, rot, tx, ty)."""
    dsx, dsy = src2.x()-src1.x(), src2.y()-src1.y()
    ddx, ddy = dst2.x()-dst1.x(), dst2.y()-dst1.y()
    s = math.hypot(ddx, ddy) / math.hypot(dsx, dsy)
    r = math.atan2(ddy, ddx) - math.atan2(dsy, dsx)
    tx = dst1.x() - s*(src1.x()*math.cos(r) - src1.y()*math.sin(r))
    ty = dst1.y() - s*(src1.x()*math.sin(r) + src1.y()*math.cos(r))
    return s, r, tx, ty
```

## 27.3 DWG 상호운용

DWG는 사양 비공개 포맷이다. 교재의 공식 입장과 3가지 실무 우회로를 명시한다.

| 경로 | 방법 | 비고 |
|---|---|---|
| A | ODA File Converter(무료)로 DWG↔DXF 일괄 변환 | 플러그인에서 외부 실행 연동(QProcess) |
| B | GDAL DWG 빌드(Teigha 연계) | 배포 라이선스 제약 커서 교재 권장 안 함 |
| C | 발주처에 DXF 납품 협의 | 계약 실무 팁 |

경로 A의 자동화(폴더 감시 일괄 변환 다이얼로그)를 실습으로 구현한다.

## 27.4 요약 및 Part VIII 마무리

열기→편집→저장→출력→상호운용의 전체 수명주기가 닫혔다. 남은 것은 품질과 배포다.

> **[호환성 노트]**
> `QgsLayoutManager`/`QgsPrintLayout`은 3.0+ 공통. `QgsLayoutExporter.exportToPdf` 시그니처도 전 구간 동일하다.

---

# Part IX. 품질·테스트·배포

---

# Chapter 28. 플러그인 아키텍처 리팩터링과 테스트

**(배정: 7p | 난이도: ★★★)**

## 28.1 최종 아키텍처 점검

**[그림 28-1] 최종 의존성 다이어그램: ui → commands → core/tools → io (역방향 의존 금지) (도판 삽입 예정)**

- io(ezdxf)와 core(순수 기하)는 QGIS GUI에 의존하지 않는다 → **QGIS 없이 pytest 가능**.
- commands는 iface를 직접 만지지 않고 plugin 서비스(preview/snap/selection)만 사용한다.

## 28.2 3계층 테스트 전략

```text
L1. 순수 단위 테스트 (CI에서 QGIS 불필요)
    - 좌표 파서, 벌지 수식, FILLET 기하, 매핑 규칙
L2. QGIS 헤드리스 테스트 (qgis.core 로드, GUI 없음)
    - DXF→레이어 변환, 차등 저장 왕복, 표현식 렌더러
L3. 수동 검수 시나리오 (17.6/21.4 검수표)
```

```python
# tests/test_bulge.py (L1)
import math
from qcad_bridge.core.geometry_utils import bulge_mid, bulge_from_arc

def test_bulge_quarter_circle():
    # 90° 호: bulge = tan(90/4) = tan(22.5°)
    b = math.tan(math.radians(22.5))
    m = bulge_mid((0, 0), (10, 0), b)
    assert abs(m[0] - 5.0) < 1e-9
    assert abs(m[1] - (b * 10 / 2)) < 1e-9

def test_roundtrip():
    b0 = 0.4142
    arc = arc_from_bulge((0,0), (10,0), b0)
    assert abs(bulge_from_arc(arc) - b0) < 1e-9
```

```python
# tests/test_roundtrip_dxf.py (L2) — 왕복 무손실의 자동화
def test_open_edit_save_reopen(tmp_path, sample_dxf):
    doc1 = detect_and_reopen(sample_dxf)
    n1 = len(doc1.modelspace())
    # ... 로드 → 엔티티 1개 이동 → 차등 저장
    out = tmp_path / "out.dxf"
    writer.save(str(out))
    doc2 = ezdxf.readfile(str(out))
    assert len(doc2.modelspace()) == n1          # 개수 보존
    assert doc2.layers.get("도로중심선").color == doc1.layers.get("도로중심선").color
```

## 28.3 요약

- "핵심 수식은 L1, 왕복은 L2, 손맛은 L3"로 역할을 나누면 유지비가 최소화된다.

> **[호환성 노트]**
> L2 테스트는 CI 매트릭스에서 qgis 3.6 / 3.44 / 4.x Docker 이미지 3종으로 실행한다(29~30장).

---

# Chapter 29. 3.6 ~ 4.x 교차 검증과 pyqgis4-checker

**(배정: 6p | 난이도: ★★☆)**

## 29.1 정적 검사

- `pyqgis4-checker`로 Qt5 잔재(직접 PyQt5 import, 플랫 enum) 스캔 → 위반은 compat.py로 이송.
- Ruff 규칙에 `PyQt5`/`PyQt6` 직접 import 금지 커스텀 룰(banned-api) 추가.

```toml
# pyproject.toml 발췌
[tool.ruff.lint.flake8-tidy-imports.banned-api]
"PyQt5".msg = "qgis.PyQt를 사용하세요 (compat 규칙 5.2)"
"PyQt6".msg = "qgis.PyQt를 사용하세요 (compat 규칙 5.2)"
```

## 29.2 런타임 매트릭스 검증

| 항목 | 3.6 | 3.44 LTR | 4.2 |
|---|---|---|---|
| 로드/언로드 반복 10회 메모리 | ☐ | ☐ | ☐ |
| DXF 10만 엔티티 로드 | ☐ | ☐ | ☐ |
| 검수표 17.6 / 21.4 | ☐ | ☐ | ☐ |
| 왕복 저장 RMS < 1e-6 | ☐ | ☐ | ☐ |

## 29.3 요약

- 호환성은 "compat로 격리 + 매트릭스로 증명" 두 문장으로 요약된다.

> **[호환성 노트]**
> 3.6 전용 이슈 재확인 목록: ezdxf 0.17 동봉(7장), curveSubstring 폴백(21장), versionInt 부재(5장), 곡선 렌더 품질(8장).

---

# Chapter 30. 패키징·플러그인 저장소 배포·CI/CD

**(배정: 7p | 난이도: ★★☆)**

## 30.1 ext_libs 동봉 패키징

```python
# scripts/build_zip.py 개념
TARGETS = {
  "qgis3":  {"ezdxf": "0.17.2", "python": "3.7"},
  "qgis4":  {"ezdxf": "latest", "python": "3.12"},
}
# pip download --only-binary --python-version ... → ext_libs/ 동봉 → zip 2종 생성
```

`__init__.py` 첫 줄에서 `sys.path.insert(0, ext_libs)` 처리. 플러그인 저장소 규정상 바이너리 동봉 시 출처·라이선스 명시(metadata의 `# external deps`)가 필요하다.

## 30.2 GitHub Actions 파이프라인

```text
push tag v* → ruff + pytest(L1) → docker qgis 3.6/3.44/4.x pytest(L2)
           → qgis-plugin-ci package (zip 2종)
           → GitHub Release + QGIS Plugin Repository 업로드
```

## 30.3 저장소 심사 대비 체크리스트

- [ ] metadata.txt 필수 키 + supportsQt6
- [ ] 실험적(experimental) 플래그로 초기 배포
- [ ] 외부 바이너리(ezdxf) 고지
- [ ] 홈페이지/트래커/저장소 URL 유효
- [ ] 아이콘 라이선스
- [ ] 개인정보/네트워크 접근 없음 명시

## 30.4 교재 대단원 요약

1장에서 정의한 판정 기준 세 가지 — 입력 동일성(12·15·16장), 스냅 동일성(16장), 왕복 무손실(8·25장) — 이 모두 자동 테스트(28장)로 고정된 상태로 배포된다. 이것이 "DXF 기반, CAD와 동일한 CAD 플러그인"의 완성 정의다.

> **[호환성 노트]**
> qgis-plugin-ci는 zip 2종(3.x/4.x) 병행 배포를 지원한다. 저장소의 QGIS 4 마이그레이션 가이드 문서를 출판 직전 재확인할 것.

---

# 부록

## 부록 A. AutoCAD 명령 ↔ QCAD-Bridge 대응표 (발췌)

| AutoCAD | 별칭 | QCAD-Bridge | 구현 장 | 상태 |
|---|---|---|---|---|
| LINE | L | line | 17 | ● |
| PLINE | PL | pline | 17 | ● |
| CIRCLE | C | circle | 17 | ● |
| ARC | A | arc | 17 | ● |
| RECTANG | REC | rectang | 17 | ● |
| POLYGON | POL | polygon | 17 | ● |
| XLINE | XL | xline | 17 | ● |
| MOVE | M | move | 19 | ● |
| COPY | CO/CP | copy | 19 | ● |
| ROTATE | RO | rotate | 19 | ● |
| MIRROR | MI | mirror | 19 | ● |
| SCALE | SC | scale | 19 | ● |
| OFFSET | O | offset | 20 | ● |
| ARRAY | AR | array | 20 | ● |
| EXPLODE | X | explode | 20 | ● |
| TRIM | TR | trim | 21 | ● |
| EXTEND | EX | extend | 21 | ● |
| FILLET | F | fillet | 21 | ● |
| DTEXT | DT | dtext | 22 | ● |
| DIMLINEAR | DLI | dimlinear | 23 | ● |
| DIMALIGNED | DAL | dimaligned | 23 | ● |
| DIMANGULAR | DAN | dimangular | 23 | ○ 심화 |
| HATCH | H | hatch | 24 | ● |
| BLOCK | B | block | 24 | ● |
| INSERT | I | insert | 24 | ● |
| CHAMFER | CHA | chamfer | — | △ 연습문제 |
| STRETCH | S | stretch | — | △ 연습문제 |
| PEDIT | PE | pedit | — | △ 연습문제 |

## 부록 B. DXF 그룹 코드 요약표

(정식 원고에서 4쪽: 0~9 문자열, 10~39 좌표, 40~59 실수, 60~79 정수, 90~99 32bit, 100 서브클래스, 210 법선, 330~369 핸들참조, 370 선굵기, 420 트루컬러 … 엔티티별 필수 코드 매트릭스 포함)

## 부록 C. ACI ↔ RGB 변환표 (발췌)

| ACI | RGB | 관행 용도 |
|---:|---|---|
| 1 | 255,0,0 | 중심선/기준 |
| 2 | 255,255,0 | 문자 |
| 3 | 0,255,0 | 계획선 |
| 4 | 0,255,255 | 보조선 |
| 5 | 0,0,255 | 현황 |
| 6 | 255,0,255 | 용지경계 |
| 7 | 255,255,255(흑백반전) | 일반 |
| 8 | 128,128,128 | 흐린 보조 |
| 250~255 | 회색 계열 | 배경/래스터 |

(전체 256색 표는 예제 저장소 `resources/aci_palette.json` + 정식 원고 부록 2쪽)

## 부록 D. Qt5/Qt6 enum 대응표 (발췌)

| 용도 | Qt5 | Qt6 | compat 상수 |
|---|---|---|---|
| 마우스 좌클릭 | `Qt.LeftButton` | `Qt.MouseButton.LeftButton` | LEFT_BUTTON |
| ESC | `Qt.Key_Escape` | `Qt.Key.Key_Escape` | KEY_ESCAPE |
| 십자 커서 | `Qt.CrossCursor` | `Qt.CursorShape.CrossCursor` | CROSS_CURSOR |
| 점선 | `Qt.DashLine` | `Qt.PenStyle.DashLine` | DASH_LINE |
| KeyPress 이벤트 | `QEvent.KeyPress` | `QEvent.Type.KeyPress` | EV_KEYPRESS |
| UserRole | `Qt.UserRole` | `Qt.ItemDataRole.UserRole` | USER_ROLE |

## 부록 E. 참고문헌·공식 문서 소스맵

- PyQGIS Developer Cookbook (4.2 EN / 3.44 KO) — 플러그인 구조·MapTool·QgsTask 1차 자료
- PyQGIS API: https://qgis.org/pyqgis/4.2/ · C++ API: https://api.qgis.org/api/4.2/
- ezdxf 공식 문서: https://ezdxf.readthedocs.io/
- DXF Reference (Autodesk 공개 사양)
- QGIS Plugin Repository 배포/승인/QGIS4 마이그레이션 문서: https://plugins.qgis.org/docs/
- qgis-plugin-ci, pyqgis4-checker, Plugin Builder 4, Plugin Reloader (사전조사 문서 소스맵 준용)
- 라이선스: QGIS(GPLv2+), QGIS Docs(GFDL 1.3+), ezdxf(MIT) — 코드 인용 시 각 라이선스 조건 준수, 출판 전 법무 검토

---

# 집필 진행 관리표 (초안 v0.1 기준)

| 장 | 초안 상태 | 정식 원고 시 보강 항목 |
|---|---|---|
| 1~3 | 본문 완성도 80% | 그림 3점, CAD 제품 스크린샷 사용권 확인 |
| 4~6 | 80% | registry.discover() 자동 수집 코드 전문 |
| 7~10 | 75% | QMetaType 분기, 실측 성능표, 100만 엔티티 벤치마크 |
| 11~13 | 70% | ACI 팔레트 다이얼로그 전문, 히스토리 키 처리 |
| 14~17 | 75% | ARC/RECTANG/POLYGON/XLINE 각 1.5쪽 전개, 극좌표 추적 3쪽 |
| 18~21 | 65% | affine 헬퍼 2쪽, curve_substring 3.6 폴백 2쪽, 호-호 FILLET 3쪽 |
| 22~24 | 65% | MTEXT 서식 심화, 내부점 해치 심화, 동적블록 제외 사유 박스 |
| 25~27 | 70% | R12 강등 전처리 3쪽, ODA 연동 QProcess 전문 |
| 28~30 | 75% | Docker CI 매트릭스 yml 전문 |
| 부록 | 60% | 그룹코드 4쪽, ACI 256 전체표 |
| 그림 | 도판 28점 목록 확정 | 캡처 QGIS 버전 4.2.x로 통일 |

**End of Draft v0.1**
