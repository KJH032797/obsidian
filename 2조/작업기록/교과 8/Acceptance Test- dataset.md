
---

## 1. Dataset

- 담당: 장세훈 (① Dataset / Audit)
- Dataset Version: `kimchi_v4` 
- Split 정책: `group_fixed_v4` (v4-B)

### Dataset 점검 항목

| No  | 점검 항목              | 기대 결과                                                | 결과              | 근거                                                                                                                                   |
| --- | ------------------ | ---------------------------------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| D-1 | 교과 7 FINAL 출처      | 인계 데이터의 출처·검수 이력 확인 가능                               | PASS            | 교과 7·8 `data/final/manifest_final.csv`의 SHA256 일치. 900행 모두 `REVIEWED`, 담당자·검수자·원본 경로 기록. 라벨 TXT 900개 확인                              |
| D-2 | Dataset Version    | 설정·Manifest·실험 기록의 버전 일치                             | PASS            | `src/project_config.py`, `manifests/split_manifest.csv` 900행, Baseline·EXP001~004 run_info 모두 `kimchi_v4`                            |
| D-3 | 원본 이력 추적           | `source_dataset / original_split / scene_type` 추적 가능 | PASS            | FINAL·Split Manifest에 해당 필드 존재. Split Manifest 900행 기준 공란 0건, 이미지명·원본 경로로 조회 가능                                                      |
| D-4 | Split Manifest     | Train·Val·Frozen Test·Bridge 목록 존재, 중복 없음            | PASS            | `manifests/train.csv` 609 · `val.csv` 66 · `test.csv` 75 · `bridge_holdout.csv` 150행. 합계 900행, 통합 목록 일치. 이미지명 중복·그룹 교차 배정 0건         |
| D-5 | Split 정책           | 분할 기준·용도·평가 한계 문서화                                   | PASS            | `docs/split_policy.md` · `reports/split_report.md`. 촬영 간격 300초 초과 기준 18개 그룹 구성, 그룹 단위 고정 배정. 현재 Manifest와 일치 `group_fixed_v4` (v4-B) |
| D-6 | Frozen Test 사용 시점  | Final 선택 후 최종 평가에 사용                                 | PASS (기록 기준)    | `reports/final_model_selection.md`: Final 선택 10/19 → Test 평가 10/20. run_info 5개 모두 `test_bridge_used: false`, Train·Val md5 일치       |
| D-7 | Bridge Holdout 미사용 | 교과 8 학습·튜닝·모델 선택·Frozen Test에서 제외                    | PASS (설정·기록 기준) | `configs/dataset.yaml`에 Bridge 경로 없음. 다른 Split과 이미지명 중복 0건. run_info 5개 모두 `test_bridge_used: false`, Final 평가는 `split=test`         |

### D-01. 교과 7 FINAL 출처

- 교과 7 원본 Manifest: `../project7/data/final/manifest_final.csv`
- 교과 8 인계본 Manifest: `data/final/manifest_final.csv`
- 두 파일 SHA256: `b28725fc7f13aca8803e2f8ba0f378a032db02d2418e3d10a030d9f13ff5669d`
- 인계본: 900행, `status=REVIEWED` 900행, 라벨 TXT 900개.
- `src/project_config.py`의 `FINAL_DIR`은 `data/final`이며, `scripts/02_audit_subject7_final.py`도 해당 경로를 입력으로 사용한다.


### D-02~03. 버전 및 데이터 이력

| 구분 | 실제 집계 |
|---|---|
| Dataset Version | `kimchi_v4` 900행 |
| Policy Version | `group_fixed_v4` 900행 |
| 원본 데이터 | `이물검출_학습데이터1` 500장 / `이물검출_학습데이터2` 400장 |
| 원본 Split | `train` 720장 / `validation` 180장 |
| 장면 | `kimchi_with_target` 725장 / `object_only` 149장 / `normal_kimchi` 24장 / `none` 2장 |
| 세 추적 필드 공란 | 0행 |

`original_split`은 교과 7 원본 구분이며, 교과 8의 `split`과 다르다. `scene_type=none`은 공란이 아닌 실제 기록값이다. 해당 장면은 Val·Test 표본이 없어 성능 평가 대상으로 주장하지 않는다.

### D-04~05. 정본 Split과 정책

| 역할 | 정본 Manifest | 장수 | 배정 그룹 | 용도 |
|---|---|---:|---|---|
| Train | `manifests/train.csv` | 609 | T00·T02·T04·T05·T06·T10·T12·T13·T15·T17 | 학습 |
| Validation | `manifests/val.csv` | 66 | T01·T03·T07 | 개발·튜닝·모델 비교 및 선택 |
| Frozen Test | `manifests/test.csv` | 75 | T08·T14 | Final 선택 후 평가 |
| Bridge Holdout | `manifests/bridge_holdout.csv` | 150 | T09·T11·T16 | 교과 14 연결용 별도 보존 |
| 합계 | `manifests/split_manifest.csv` | 900 | 18개 그룹 | 이미지명 중복 및 그룹 교차 배정 없음 |

- Class 6은 Val·Test 각각 객체 1개여서 정책상 성능 판단 불가다. Class 4는 미사용이다.
- `object_only`와 `none`은 Val·Test 0장, `normal_kimchi`는 Val 15장·Test 0장이다. 없는 장면의 Test 성능을 주장하지 않는다.

### 결과

- Critical Error: Manifest 수량·중복·그룹 배정 검사에서 0건. 
- Unresolved Issue: FINAL·Split·Bridge 실물 확인, README 인계 경로 갱신. 
- Dataset 파트 결과: Manifest·설정 점검 PASS. Frozen Test·Bridge 미사용은 기록 기준 PASS. 데이터 실물 인계·실행 재현성은 추가 확인 필요.

> 상세 집계·해시는 `reports/acceptance_dataset_evidence.json` 참조.