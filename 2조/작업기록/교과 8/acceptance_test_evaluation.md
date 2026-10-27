# Acceptance Test — Evaluation 파트 확인 결과

- 담당: 하나 (역할 4, Evaluation/Failure)
- 확인일: 2026-10-21
- 기준 Split: v4-B (Train 609 / Val 66 / Test 75 / Bridge 150, 촬영 묶음 단위 배정)
- 결과 기준: 수행가이드 22.7의 Evaluation 항목

---

## Evaluation 점검 항목

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

## FAIL → 수정 → Retest 기록

| 항목 | 문제 | 원인 | 수정 | 재시험 결과 |
|---|---|---|---|---|
| E-7 | Frozen Test 75장 평가에 `eval_exclude` 사진 `250411_140406550160.jpg`가 포함됨 | 최초 평가 스크립트가 제외 목록을 반영하지 않음 | 제외 목록을 적용한 `13b` 평가를 추가하고 74장 기준으로 결과를 별도 기록 | PASS — `reports/final_metrics_eval74.csv`; 74장 기준 P .605 / R .466 / mAP50 .499 / mAP50-95 .417 (문서에 기록된 Class별 평균 계산값). 기존 75장 값은 .605 / .466 / .493 / .412. 산출 방식이 달라 수치 비교 시 구분 필요 |

## 주요 결과 요약

| 모델 | 평가 데이터 | P | R | mAP50 | mAP50-95 |
|---|---|---:|---:|---:|---:|
| Baseline (yolo11n, 640) | Val 66 | 0.917 | 0.444 | 0.506 | 0.345 |
| EXP003 (yolo11n, 1280) | Val 66 | 0.948 | 0.491 | 0.550 | 0.446 |
| EXP003 | Test 75 (최초 평가) | 0.605 | 0.466 | 0.493 | 0.412 |
| EXP003 | Test 74 (`eval_exclude` 반영, 문서상 계산값) | 0.605 | 0.466 | 0.499 | 0.417 |

### Test 74 Class별 결과 (문서에 기록된 값)

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

## 알려진 한계

- 병해(Class 5)는 Val·Test에서 Recall 0으로 기록되어 거의 놓치는 것으로 보고됨.
- Test에서 Precision은 Val .948에서 Test .605로 낮아졌으며, leaf_paper·insect 오탐과 branch를 insect·leaf로 오분류한 것이 주요 원인으로 분석됨.
- 작은 객체 FN 비율이 높음 (Val 67%, Test 39%).
- Class 6 및 이물만 있는 장면(T17)은 평가 표본이 부족해 성능 결론을 내릴 수 없음.

## 결과

- Critical Error: 0 (원문 기록)
- Unresolved Issue: 74장 지표의 집계 방식 및 CSV 전체 평균 대조 필요
- Evaluation 파트 결과: 주요 점검 항목 PASS. 단, 위 지표 산출 근거 확인은 남아 있음.
