# Acceptance Test — 교과 8

- 작성일: 2026-10-21
- 작성 기준: 교과8 수행가이드 22.7
- 순서: Dataset → Model → Evaluation → Inference → Auto Label
- 확인: 이성형
- 확인일: 2026-10-21
- 기준: GitHub main

---

## 1. Dataset

> 담당자 확인 자료를 받은 뒤 작성합니다.


## 2. Model

- 담당: 이상직(② Train), 박하솔 (④ Experiment)
- Dataset Version: kimchi_v4 (Split v4-B)
- Final Model: `models/final/final_best.pt` (= EXP003 · yolo11n · imgsz 1280)

### Model 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| M-1 | Baseline · Final Weight 구분 | 모델 파일을 용도별로 구분 보관 | PASS | `models/baseline/baseline_best.pt` · `models/final/final_best.pt` |
| M-2 | Improved Weight 보관 | 실험별 Weight 확인 가능 | PASS | `models/improved/exp001 ~ 004_best.pt` |
| M-3 | Final Weight 원본 확인 | `final_best.pt`가 선택된 EXP003과 동일 | PASS | `final_best.pt` = `exp003_best.pt` (기록된 md5 `6f61447c` 일치) |
| M-4 | Final 선택 근거 | 선택·제외 이유가 문서화 | PASS | `reports/final_model_selection.md` ①~④ |
| M-5 | Final Weight 로드 | 가중치 로드 및 평가 실행 가능 | PASS | 10/20 `scripts/13b_evaluate_final_test_eval74.py` 실행 기록 (YOLO11n 101 layers · 74장 평가 완료) |
| M-6 | Baseline Weight 로드 | 가중치 로드 및 비교 실행 가능 | PASS | `scripts/11c_compare_failures.py` 실행 기록 → `reports/failure_compare.csv` baseline 행 |

### 실험 비교 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| X-1 | 한 변수 실험 | 실험별 변경 변수가 기록됨 | PASS* | `reports/exp00N_run_info.json`의 `changed_variable`; EXP004는 변수 2개라 별도 표시 |
| X-2 | 동일 데이터·Validation | 각 실험의 Train·Val 데이터가 동일 | PASS | run_info 5개 모두 train md5 `fa803448` · val md5 `e1c68baa` |
| X-3 | Test·Bridge 미사용 | 실험 학습·선택에 Test·Bridge 미사용 | PASS | run_info 5개 모두 `test_bridge_used: false` |
| X-4 | 채택 기준 사전 기록 | 결과 확인 전에 채택 기준 기록 | PASS | `reports/improvement_experiment.md` Part 1 (10/16 · mAP50-95 +0.02) |
| X-5 | Baseline vs Improved 비교 | 같은 Validation에서 P·R·mAP·FP/FN 비교 | PASS | `reports/experiment_log.csv` · `baseline_vs_improved.md` · `failure_compare.csv` · `fn_by_size_compare.csv` |
| X-6 | Final 선택 시점 | Frozen Test 확인 전에 Final 선택 | PASS | Final 선택 10/19 → Test 10/20, `reports/final_model_selection.md` · `scripts/13_evaluate_final_test.py` 재실행 차단 기록 |
| X-7 | Frozen Test 결과 구분 | 최초 75장과 제외 반영 74장 결과를 구분 기록 | PASS | `reports/final_metrics.csv` (75장, mAP50-95 .4115) · `reports/final_metrics_eval74.csv` (74장, .4171) |

### FAIL → 수정 → Retest 기록

| 항목        | 문제                                                                   | 원인                         | 수정                                                                                           | 재시험 결과                                                                                                            |
| --------- | -------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| X-7 / E-7 | Frozen Test 75장 결과에 `eval_exclude` 사진 `250411_140406550160.jpg`가 포함됨 | 최초 평가 스크립트가 제외 목록을 반영하지 않음 | 같은 모델·설정으로 제외 목록을 반영한 74장 결과를 별도 추가 기록 (`scripts/13b_evaluate_final_test_eval74.py`, PR #72) | PASS — 75장 mAP50-95 .4115 / 74장 .4171로 구분 기록. 모델·설정은 변경하지 않았고 결론은 동일하다고 기록됨. 세부 지표의 산출 형식은 Evaluation 기록의 주의사항 참조 |

### 결과

- Critical Error: 0 (원문 기록)
- Unresolved Issue: 없음 (원문 기록)
- Model 파트 결과: PASS

> * 수행가이드의 “한 변수 개선” 취지와 달리 EXP004는 변수 2개를 바꾼 실험이라고 기록되어 있다. EXP004는 단일 변수 실험으로 간주하지 않는다.

## 3. Evaluation

- 담당: 박하나 (③ Evaluation/Failure)

### Evaluation 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| E-1 | Baseline Validation 재현 | 팀 기준 수치와 재실행 수치 일치 | PASS | `07_evaluate_baseline.py`, `models/baseline/baseline_best.pt` → P .917 / R .444 / mAP50 .506 / mAP50-95 .345. 2026-10-21 재실행. 팀 PDF의 .916 / .444 / .506 / .3445와 일치 |
| E-2 | Class별 지표 산출 | Class 0~6의 P·R·mAP 기록 | PASS | `reports/baseline_v4b_val66_metrics.md` (Class 5 R 0, Class 6 1개) |
| E-3 | 실패 후보 수집·분류 | FN·FP·CLASS·BBOX 유형과 근거 산출 | PASS | `09_collect_failures.py` → FN 82 · FP 24 · CLASS 11 · BBOX 6 (합 123), `runs/failure_candidates/`, `reports/failure_summary.csv` |
| E-4 | FN 크기별 분석 | 누락 객체 크기 분포 기록 | PASS | `10_fn_by_size.py` → small FN 60/90 (67%), 전체 82 FN 중 73%가 small |
| E-5 | Final 모델 선택 근거 | Frozen Test 전에 Validation 기준 선택 | PASS | EXP003 (yolo11n, 1280), Val 66 mAP50-95 .446 (Baseline .345 대비 +29%) |
| E-6 | Frozen Test 평가 | Final 확정 후 고정 Test 평가 결과 기록 | PASS | `13_evaluate_final_test.py` → P .605 / R .466 / mAP50 .493 / mAP50-95 .412 |
| E-7 | 평가 제외 사진 처리 | `eval_exclude` 사진이 최종 평가 집계에서 제외됨 | FAIL → 수정 후 PASS | 재시험은 아래 실패·수정·재검증 기록 참고 |
| E-8 | Frozen Test 실패 분석 | Test 결과를 튜닝에 쓰지 않고 오류 분석 | PASS | `16_frozen_test_analysis.py` → `reports/frozen_test/`: FN 93 · FP 87 · CLASS 72 · BBOX 4, FN 61% small, CLASS 72 중 branch→insect 31 · leaf 25 |
| E-9 | 평가 한계 기록 | 표본 부족 클래스는 과도한 결론을 내리지 않음 | PASS | Class 6 (Val·Test 각 1개), 이물만 장면(T17)은 평가 불가로 한계에 기록 |

### FAIL → 수정 → Retest 기록

| 항목 | 문제 | 원인 | 수정 | 재시험 결과 |
|---|---|---|---|---|
| E-7 | Frozen Test 75장 평가에 `eval_exclude` 사진 `250411_140406550160.jpg`가 포함됨 | 최초 평가 스크립트가 제외 목록을 반영하지 않음 | 제외 목록을 적용한 `13b` 평가를 추가하고 74장 기준으로 결과를 별도 기록 | PASS — `reports/final_metrics_eval74.csv`; 74장 기준 P .605 / R .466 / mAP50 .499 / mAP50-95 .417 (문서에 기록된 Class별 평균 계산값). 기존 75장 값은 .605 / .466 / .493 / .412. 산출 방식이 달라 수치 비교 시 구분 필요 |

### 주요 결과 요약

| 모델 | 평가 데이터 | P | R | mAP50 | mAP50-95 |
|---|---|---:|---:|---:|---:|
| Baseline (yolo11n, 640) | Val 66 | 0.917 | 0.444 | 0.506 | 0.345 |
| EXP003 (yolo11n, 1280) | Val 66 | 0.948 | 0.491 | 0.550 | 0.446 |
| EXP003 | Test 75 (최초 평가) | 0.605 | 0.466 | 0.493 | 0.412 |
| EXP003 | Test 74 (`eval_exclude` 반영, 문서상 계산값) | 0.605 | 0.466 | 0.499 | 0.417 |

#### Test 74 Class별 결과 (문서에 기록된 값)

| Class | P | R | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| 0 leaf_paper | 0.320 | 0.857 | 0.667 | 0.582 |
| 1 plastic_stone_metal | 0.977 | 0.478 | 0.789 | 0.643 |
| 2 branch | 0.990 | 0.558 | 0.954 | 0.807 |
| 3 insect | 0.342 | 0.900 | 0.522 | 0.441 |
| 4 rubber_glove_unused | N/A | N/A | N/A | N/A |
| 5 disease_browning | 1.000 | 0.000 | 0.064 | 0.030 |
| 6 green_onion_pepper | 0.000 | 0.000 | 0.000 | 0.000 (평가 불가) |

> 74장 전체 값은 Class 0~3·5·6 평균을 직접 계산한 값으로 기록되어 있다. `final_metrics_eval74.csv`의 전체 평균을 산출한 값인지 파일을 대조해 확정하는 작업은 이 기록에서 확인되지 않는다. Class 6 제외 시 문서상 P .726 / R .559 / mAP50 .599 / mAP50-95 .500.

### 알려진 한계

- 병해(Class 5)는 Val·Test에서 Recall 0으로 기록되어 거의 놓치는 것으로 보고됨.
- Test에서 Precision은 Val .948에서 Test .605로 낮아졌으며, leaf_paper·insect 오탐과 branch를 insect·leaf로 오분류한 것이 주요 원인으로 분석됨.
- 작은 객체 FN 비율이 높음 (Val 67%, Test 39%).
- Class 6 및 이물만 있는 장면(T17)은 평가 표본이 부족해 성능 결론을 내릴 수 없음.

### 결과

- Critical Error: 0 (원문 기록)
- Unresolved Issue: 74장 지표의 집계 방식 및 CSV 전체 평균 대조 필요
- Evaluation 파트 결과: 주요 점검 항목 PASS. 단, 위 지표 산출 근거 확인은 남아 있음.

## 4. Inference

- 담당: 김준형 (⑤Inference / Document)
- Final Model: `models/final/final_best.pt`
- 추론 크기: `imgsz=1280`
- 입력: `data/test_detection/`
- 결과: `demo/prediction_images/` 및 `demo/prediction_images/inference_results.csv`

### Inference 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| I-1 | Detection 실행 | 지정한 시연 이미지에 Final Model 추론 수행 | PASS (사용자 확인) | `python scripts/14_predict_images.py` 실행 결과 보고 |
| I-2 | Bounding Box 표시 | 결과 이미지에 검출 상자 표시 | PASS (사용자 확인) | 실행 후 GUI/결과 확인 보고 |
| I-3 | Class 표시 | 검출 클래스가 결과에 표시 | PASS (사용자 확인) | 실행 후 결과 확인 보고 |
| I-4 | Confidence 표시 | 신뢰도가 결과에 표시 | PASS (사용자 확인) | 실행 후 결과 확인 보고 |
| I-5 | 결과 이미지 저장 | 이미지 결과가 지정 산출 경로에 저장 | PASS (사용자 확인) | `demo/prediction_images/` 결과 확인 보고 |
| I-6 | 결과 CSV 경로 | 경로가 프로젝트 기준 상대경로로 기록 | PASS (사용자 확인) | `inference_results.csv` 생성 및 경로 기록 확인 보고 |
| I-7 | 시연 입력 제한 | `data/test_detection/`의 허용된 시연 이미지 사용 | 확인 필요 | 이 문서에 실제 입력 파일 목록이나 출처 기록은 없음 |
| I-8 | 추론 설정 | Final 설정에 맞춘 입력 크기 사용 | PASS (코드·사용자 보고) | `imgsz=1280`; 실행 결과 확인은 사용자 보고에 근거 |

### FAIL → 수정 → Retest 기록

| 항목 | 문제 | 원인 | 수정 | 재시험 결과 |
|---|---|---|---|---|
| I-1 | 최초 명령 `python 14_predict_images.py` 실행 실패 | 스크립트가 프로젝트 루트가 아닌 `scripts/` 아래에 있음 | 프로젝트 루트 기준 명령을 `python scripts/14_predict_images.py`로 수정 | PASS (사용자 확인) — 수정한 명령으로 Detection을 실행하고 결과를 확인했다고 보고함 |
| I-6 | CSV에 개인 PC 절대경로가 들어간다는 리뷰 | 입력·결과·모델 경로 기록이 프로젝트 기준 경로로 정리되지 않음 | CSV 경로 기록을 프로젝트 기준 상대경로로 바꾸고, 프로젝트 밖 경로는 파일명으로 기록하도록 수정 | PASS (사용자 확인) — CSV 경로 기록 확인 보고. NDA 복사본에는 이미지가 없어 이 문서 작성 중 직접 재실행·대조하지 않음 |

### 결과

- Critical Error: 사용자 보고 기준 0
- Unresolved Issue: 시연 입력 이미지의 출처·파일 목록은 이 기록만으로 확인되지 않음
- Inference 파트 결과: 실행과 표시·저장 확인은 사용자 보고 기준 PASS. 이 통합 문서를 작성하면서 직접 재현한 것은 아님.

### 실행 인터페이스 및 범위

- 프로젝트 루트에서 `python scripts/14_predict_images.py`로 실행한다.
- 자동 성능평가와 이미지 Detection 시연은 분리한다.
- 입력 이미지와 Final Weight를 사용해 검출 결과 이미지 및 CSV를 저장한다.
- Frozen Test를 시연에 복사해 사용하는 경우에도 평가 결과를 다시 산출하거나 학습·튜닝에 사용하지 않는다. `bridge_holdout`은 사용하지 않는다.

## 5. Auto Label

### 실행 및 데이터 흐름

1. 프로젝트 8 루트에서 `python scripts/15_auto_label.py`를 실행한다. 특정 이미지 1장만 처리할 때는 `python scripts/15_auto_label.py --input data/test_detection/파일명.jpg`를 사용한다. 입력을 생략하면 `data/test_detection/` 바로 아래 이미지들을 처리한다.
2. `scripts/15_auto_label.py`가 Final Weight `models/final/final_best.pt`로 추론한다. 확인된 실행 기록의 설정은 Confidence `0.25`, `imgsz=1280`이다.
3. 이미지별 YOLO 후보 라벨은 `demo/auto_label/candidate_labels/*.txt`에 저장한다. 실행 요약 CSV는 같은 폴더의 `auto_label_results.csv`에 저장된다.
4. 실행 스크립트는 후보별 상태·BBox 수를 `reports/auto_label_review.csv`에 등록한다.
5. 교과 7 라벨링 GUI에서 Auto Label 후보를 불러와 검수한다. 검수한 TXT는 `demo/auto_label/reviewed_labels/*.txt`에 저장하고, 동일한 `reports/auto_label_review.csv` 행에 최종 검수 상태·BBox 수·변경 작업·담당자·시각을 갱신한다.

### Auto Label 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| A-1 | 후보 라벨 생성 | 시연 이미지에서 Final Model로 후보 YOLO TXT 생성 | PASS | `scripts/15_auto_label.py`; 실행 CSV에 입력 이미지 `250424_160002175452.jpg`, 상태 `OK` 기록 |
| A-2 | 후보 TXT 형식·BBox 수 | `class_id x_center y_center width height` 형식, 정규화 좌표 0~1, 실행 수량 일치 | PASS | `candidate_labels/250424_160002175452.txt` 3행. Class ID 2·3, 좌표값 0~1. `auto_label_results.csv`의 `detection_count=3` |
| A-3 | 실행 결과 기록 | 입력·후보 경로·상태·BBox 수·모델·conf·imgsz 기록 | PASS | `demo/auto_label/candidate_labels/auto_label_results.csv`: 1건 OK, 3 detections, Final Weight, conf 0.25, imgsz 1280 |
| A-4 | GUI에서 후보 열기 | 교과 7 도구가 후보와 대응 이미지를 열어 검수 가능 | PASS (사용자 시행 확인) | 검수 기록의 `reviewed_label` 경로 및 `reviewed_by=kjh` |
| A-5 | 사람의 라벨 수정 | 필요 라벨을 사람이 추가·수정·삭제할 수 있음 | PASS | 검수 CSV: 3개에서 4개로 변경, `added=1`, `class_edits=0`, `bbox_edits=0`, `deleted=0` |
| A-6 | 검수본 저장 | 승인 라벨을 별도 reviewed 경로에 저장 | PASS | `demo/auto_label/reviewed_labels/250424_160002175452.txt`; 파일 4행, 후보 경로는 별도 보존 |
| A-7 | Human Review 이력 갱신 | 후보 생성 때 만든 CSV 행에 검수 결과를 기록 | PASS | `reports/auto_label_review.csv`: `APPROVED_EDITED`, 후보 3 / 최종 4, 추가 1, 검수 1회, 담당 `kjh`, `2026-10-21T10:03:37+09:00` |

### 실제 실행 결과 평가

- 확인 가능한 기록은 이미지 1장 처리 및 검수 1회다.
- Auto Label 결과: `status=OK`, 후보 BBox 3개, 실행 시간 1.513116초, Confidence 기준 0.25, `imgsz=1280`.
- Human Review 결과: 후보 3개를 최종 4개로 승인 수정했다. 구체적으로 Class 5 라벨 1개를 추가했고, 클래스 수정·기존 BBox 수정·삭제는 0건이다. 검수 전 후보 TXT와 검수 후 TXT가 각각 후보/검수 경로에 존재하며 각각 3행과 4행이고 모든 좌표값은 0~1 범위다.
- 상태는 `APPROVED_EDITED`; 승인된 검수 행은 1개다.
- 이는 후보 생성과 검수·저장 파이프라인이 정상 동작했다는 확인이다. GT 정답과 비교한 정밀도·재현율 등 모델 정확도 평가를 뜻하지 않는다. 복사본에는 원본 이미지가 없어 이미지와 라벨의 시각적 정합성은 여기서 별도 판정하지 않았다.

### 결과

- Critical Error: 실행·저장·검수 기록에서 확인된 오류 0건
- Unresolved Issue: 이미지가 NDA 복사본에 없어 시각적 라벨 정확도는 이 문서 작성 중 재검수하지 않음
- Auto Label 파트 결과: 사용자가 시행을 확인했고, 추가된 실행·Human Review CSV와 후보/검수 TXT의 수량·상태 기록은 서로 일치함.
