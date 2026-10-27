# Acceptance Test — Inference 파트 확인 결과

- 담당: 준형 (Inference / Document)
- 확인일: 2026-10-21 (사용자 보고 기록 기준)
- 기준: 교과8 수행가이드 22.7의 Inference 항목
- Final Model: `models/final/final_best.pt`
- 추론 크기: `imgsz=1280`
- 입력: `data/test_detection/`
- 결과: `demo/prediction_images/` 및 `demo/prediction_images/inference_results.csv`

---

## Inference 점검 항목

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

## FAIL → 수정 → Retest 기록

| 항목 | 문제 | 원인 | 수정 | 재시험 결과 |
|---|---|---|---|---|
| I-1 | 최초 명령 `python 14_predict_images.py` 실행 실패 | 스크립트가 프로젝트 루트가 아닌 `scripts/` 아래에 있음 | 프로젝트 루트 기준 명령을 `python scripts/14_predict_images.py`로 수정 | PASS (사용자 확인) — 수정한 명령으로 Detection을 실행하고 결과를 확인했다고 보고함 |
| I-6 | CSV에 개인 PC 절대경로가 들어간다는 리뷰 | 입력·결과·모델 경로 기록이 프로젝트 기준 경로로 정리되지 않음 | CSV 경로 기록을 프로젝트 기준 상대경로로 바꾸고, 프로젝트 밖 경로는 파일명으로 기록하도록 수정 | PASS (사용자 확인) — CSV 경로 기록 확인 보고. NDA 복사본에는 이미지가 없어 이 문서 작성 중 직접 재실행·대조하지 않음 |

## 결과

- Critical Error: 사용자 보고 기준 0
- Unresolved Issue: 시연 입력 이미지의 출처·파일 목록은 이 기록만으로 확인되지 않음
- Inference 파트 결과: 실행과 표시·저장 확인은 사용자 보고 기준 PASS. 실행은 수행자가 직접 재현한 것이 아님.

## 실행 인터페이스 및 범위

- 프로젝트 루트에서 `python scripts/14_predict_images.py`로 실행한다.
- 자동 성능평가와 이미지 Detection 시연은 분리한다.
- 입력 이미지와 Final Weight를 사용해 검출 결과 이미지 및 CSV를 저장한다.
- Frozen Test를 시연에 복사해 사용하는 경우에도 평가 결과를 다시 산출하거나 학습·튜닝에 사용하지 않는다. `bridge_holdout`은 사용하지 않는다.
