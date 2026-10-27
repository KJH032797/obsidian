# Acceptance Test — Experiment 파트 확인 결과

- 담당: 박하솔 (⑤ Experiment)
- 확인일: 2026-10-21
- 기준: GitHub main
- Dataset Version: kimchi_v4 (Split v4-B)
- Final Model: `models/final/final_best.pt` (= EXP003 · yolo11n · imgsz 1280)
- 결과 기준: 수행가이드 22.7의 Model 및 실험 비교 관련 항목

---

## Model 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| M-1 | Baseline · Final Weight 구분 | 모델 파일을 용도별로 구분 보관 | PASS | `models/baseline/baseline_best.pt` · `models/final/final_best.pt` |
| M-2 | Improved Weight 보관 | 실험별 Weight 확인 가능 | PASS | `models/improved/exp001 ~ 004_best.pt` |
| M-3 | Final Weight 원본 확인 | `final_best.pt`가 선택된 EXP003과 동일 | PASS | `final_best.pt` = `exp003_best.pt` (기록된 md5 `6f61447c` 일치) |
| M-4 | Final 선택 근거 | 선택·제외 이유가 문서화 | PASS | `reports/final_model_selection.md` ①~④ |
| M-5 | Final Weight 로드 | 가중치 로드 및 평가 실행 가능 | PASS | 10/20 `scripts/13b_evaluate_final_test_eval74.py` 실행 기록 (YOLO11n 101 layers · 74장 평가 완료) |
| M-6 | Baseline Weight 로드 | 가중치 로드 및 비교 실행 가능 | PASS | `scripts/11c_compare_failures.py` 실행 기록 → `reports/failure_compare.csv` baseline 행 |

## 실험 비교 점검 항목

| No | 점검 항목 | 기대 결과 | 결과 | 근거 |
|---|---|---|---|---|
| X-1 | 한 변수 실험 | 실험별 변경 변수가 기록됨 | PASS* | `reports/exp00N_run_info.json`의 `changed_variable`; EXP004는 변수 2개라 별도 표시 |
| X-2 | 동일 데이터·Validation | 각 실험의 Train·Val 데이터가 동일 | PASS | run_info 5개 모두 train md5 `fa803448` · val md5 `e1c68baa` |
| X-3 | Test·Bridge 미사용 | 실험 학습·선택에 Test·Bridge 미사용 | PASS | run_info 5개 모두 `test_bridge_used: false` |
| X-4 | 채택 기준 사전 기록 | 결과 확인 전에 채택 기준 기록 | PASS | `reports/improvement_experiment.md` Part 1 (10/16 · mAP50-95 +0.02) |
| X-5 | Baseline vs Improved 비교 | 같은 Validation에서 P·R·mAP·FP/FN 비교 | PASS | `reports/experiment_log.csv` · `baseline_vs_improved.md` · `failure_compare.csv` · `fn_by_size_compare.csv` |
| X-6 | Final 선택 시점 | Frozen Test 확인 전에 Final 선택 | PASS | Final 선택 10/19 → Test 10/20, `reports/final_model_selection.md` · `scripts/13_evaluate_final_test.py` 재실행 차단 기록 |
| X-7 | Frozen Test 결과 구분 | 최초 75장과 제외 반영 74장 결과를 구분 기록 | PASS | `reports/final_metrics.csv` (75장, mAP50-95 .4115) · `reports/final_metrics_eval74.csv` (74장, .4171) |

## FAIL → 수정 → Retest 기록

| 항목 | 문제 | 원인 | 수정 | 재시험 결과 |
|---|---|---|---|---|
| X-7 / E-7 | Frozen Test 75장 결과에 `eval_exclude` 사진 `250411_140406550160.jpg`가 포함됨 | 최초 평가 스크립트가 제외 목록을 반영하지 않음 | 같은 모델·설정으로 제외 목록을 반영한 74장 결과를 별도 추가 기록 (`scripts/13b_evaluate_final_test_eval74.py`, PR #72) | PASS — 75장 mAP50-95 .4115 / 74장 .4171로 구분 기록. 모델·설정은 변경하지 않았고 결론은 동일하다고 기록됨. 세부 지표의 산출 형식은 Evaluation 문서의 주의사항 참조 |

## 결과

- Critical Error: 0 (원문 기록)
- Unresolved Issue: 없음 (원문 기록)
- Experiment 파트 결과: PASS

> * 수행가이드의 “한 변수 개선” 취지와 달리 EXP004는 변수 2개를 바꾼 실험이라고 이 문서가 별도 표시한다. EXP004는 단일 변수 실험으로 간주하지 않는다.
