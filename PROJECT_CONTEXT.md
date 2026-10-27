# 프로젝트 컨텍스트

## 문서 적용 원칙

이 문서는 연결된 Obsidian 수업 자료와 `C:\Users\user1\Desktop\group2-project` 복사본의 문서·코드·설정·산출물을 함께 대조해 이후 작업에 필요한 기준만 정리한다.

내용이 충돌할 때는 다음을 구분한다.

- **수업 요구사항:** 교과 수행가이드와 팀 운영 규칙에 적힌 목표·제약
- **팀의 결정/현재 상태:** 작업 기록과 사용자가 알려준 진행 상황
- **실제 구현:** 현재 프로젝트 복사본의 코드·Manifest·보고서 내용

어느 하나를 근거 없이 다른 하나의 정답으로 간주하지 않는다. 구현 변경 전에 충돌을 보고하고 필요한 결정을 확인한다. 수업 수행가이드는 메인 기준이며, 참고 문서와 산출물 예시는 이해·작성 보조자료다.

## 프로젝트 전체 목적

조각김치 이물 검출 과제를 두 단계로 수행한다.

1. **교과 7:** 기존 YOLO 라벨을 검수·보정하고 신뢰할 수 있는 FINAL Dataset과 QA 기록을 만든다.
2. **교과 8:** FINAL Dataset을 감사·분할하고 YOLO11n 모델을 학습·평가·개선한 뒤, Frozen Test와 Detection·Auto Label까지 검증한다.

데이터는 교육 목적의 100% 가상 생성 데이터로 설명되어 있으나 NDA 및 기관 보안 규칙의 적용을 받는다. 바탕화면 프로젝트 복사본에는 이미지(JPG)와 라벨 TXT가 첨부되지 않았고 디렉터리 구조만 있다. 따라서 이 복사본만으로 이미지 육안 확인, 원본-라벨 대조, 데이터 Audit 재실행, 해시 검증을 했다고 볼 수 없다. CSV/Markdown 산출물에 기록된 내용을 파일 자체의 재검증 결과와 혼동하지 않는다. 공개 저장소·개인 클라우드·메신저로 원본이나 민감한 파생물을 옮기지 않는다. Git 허용 범위는 아래 “문서·구현·진행 상태의 차이”에 적힌 상충 사항을 먼저 확인한다.

## 교과 7 — 라벨 검수와 FINAL

### 수업 목표와 데이터 흐름

교과 7은 모델 학습이나 신규 Train/Validation/Test 분할이 목적이 아니다. 제공된 이미지 900장과 YOLO TXT 900개의 기존 라벨을 화면에서 확인하고 필요한 부분만 수정·추가·삭제해 교과 8의 정답 데이터로 넘긴다. 원본 Dataset과 `train`/`validation` 출처는 유지한다.

`RAW(읽기 전용, 해시 확인) → WORK(수정본·작업 Manifest) → 자동 Validation → Human QA/교차검수 → 통과 조건 확인 → FINAL(images, labels, Manifest)` 흐름이다. 자동 Validation은 형식·좌표 등 기계적으로 찾을 수 있는 문제를 보고하며 자동으로 라벨을 고치지 않는다. 모호한 판단은 추측하지 않고 `REVIEW`와 근거를 기록한다.

### QA와 Human QA의 의미

- Human QA는 자동 검사 결과나 수정·신규·REVIEW 라벨처럼 확인 대상이 된 사례를 사람이 실제 이미지와 대조해 판정하고, 결과·근거를 남기는 절차다. 보고서 목차의 담당자별 처리량은 Human QA 자체를 뜻하지 않는다.
- 수업 실행 체크리스트는 수정·신규·REVIEW·Class 4·빈 TXT를 교차 확인하고 PASS는 표본 확인하도록 제시한다. 자동 검사가 표시한 빈 TXT를 곧바로 오류로 단정하지 않고 이미지와 장면 유형을 확인한다.
- 현재 QA Summary가 기록한 Human QA는 `250513_021424892102.jpg`, `250410_160155606350.jpg` 두 건이다. 고무장갑만 있고 김치와 검출 대상 객체가 없어 Class 4 BBox를 제거하고 `scene_type=none`으로 기록했다. 이 기록은 나머지 빈 라벨 24건을 사람이 확인했다는 뜻이 아니다.

### Class·장면·출력 조건

- Class ID 0~6은 고정이며 재번호화하지 않는다. Class 4(고무장갑)는 미사용 Class다. 김치는 검출 Class가 아니다.
- 수업의 교과 7 장면 유형 예시는 `kimchi_with_target`, `normal_kimchi`, `object_only`, `other_review`다. `scene_type`은 Class가 아니라 이미지 전체를 설명하는 Manifest 메타데이터다.
- FINAL 생성 조건은 미처리 상태(REVIEW/PENDING/WORKING 등)와 미교차검수 EDITED가 없고 CRITICAL Validation이 0인 것 등이다. RAW 보존 해시가 있으면 원본 불변도 확인한다. FINAL은 직접 수정하지 않는다.
- 교과 7 공식 산출물은 프로그램·코드·FINAL 증빙·Baseline·Class/BBox 기준서·Dataset Manifest·QA Summary·Test Report·README·교과 8 Handoff를 포함한 11개다.

현재 프로젝트 QA 기록에는 900장, TXT 900개, 빈 TXT 26개, 전체 BBox 4,399→4,405개가 적혀 있다. 이는 현재 보고서 내용이며, 별도 원본 이미지 전수 재검증을 했다는 뜻은 아니다.

## 교과 8 — 데이터 감사, 분할, 객체 검출

### 목표와 진행 순서

교과 7 기준과 정답 라벨을 바꾸지 않고 이어받아 `FINAL 확인 → Dataset Audit → Split 정책·Freeze → YOLO Dataset 구성 → Smoke Test → Baseline → Validation 평가 → Failure Analysis → 한 변수 개선 → Baseline/Improved 비교 → Final Model 선택 → Frozen Test → Detection/Auto Label + Human Review → Acceptance Test·문서화` 순으로 진행한다.

수업 문서는 600/75/75/150을 분할 예시로 제시하며, 실제 장수는 Dataset Audit과 촬영 그룹·Class·장면 분포를 확인한 뒤 정책에 근거를 적어 확정하도록 한다. 현재 저장소 README와 팀 기록은 600 Train / 75 Val / 75 Test / 150 Bridge Holdout을 팀 결과로 제시한다. 이 값은 해당 팀 결정의 기록이지 모든 프로젝트에 적용되는 보편 규칙은 아니다.

### Dataset Audit과 Split

- Audit은 이미지·TXT 수와 Pair, Manifest의 `source_dataset`/`original_split`/`scene_type`, YOLO 형식·Class·좌표, 빈 라벨, Class/BBox/장면 분포, 고아 파일 등을 조사한다. TXT 통계와 스크립트가 후보를 찾고, 사람은 발견된 이상·장면·촬영 그룹을 확인한다. 900장을 사람이 새로 라벨링하거나 일일이 수동 분할하는 단계가 아니다.
- Split은 단순 무작위나 파일명 순이 아니라 출처·기존 Split·장면·Class와 중복/유사 이미지·촬영 그룹을 근거로 구성한다. 파일명의 시각이 촬영 그룹을 뜻한다고 확인 없이 단정하지 않는다. 가능한 경우 한 촬영 그룹을 여러 Split에 나누지 않아 Data Leakage를 막는다.
- 각 이미지에는 `train`, `val`, `test`, `bridge_holdout` 중 하나만 부여하고 전체 Master Manifest 및 역할별 CSV에 추적 정보를 보존한다. Split Freeze 후 Baseline/Improved 비교 중 임의 이동하지 않는다.
- Train은 학습, Validation은 개발·비교, Frozen Test는 Final Model 선택 후 최종 평가, Bridge Holdout은 교과 14 연결용이다. Test 결과를 보고 재튜닝하지 않으며 Bridge는 교과 8 학습·튜닝·모델 선택·Test에 사용하지 않는다. `dataset.yaml`에 Bridge를 넣지 않는다.
- Smoke Test는 작은 데이터로 파이프라인이 실행되는지 확인할 뿐 Baseline이 아니다. Baseline은 Smoke Test 이후 확정된 전체 Train 조건의 첫 정식 모델이다.

### 모델 평가·개선

- 완료 기준은 특정 mAP 숫자가 아니라 지표와 실패 사례를 해석하고, 개선 근거·결과·한계를 설명할 수 있는가다.
- Precision·Recall·mAP50·mAP50-95와 Class별 결과를 확인하고, 실제 Validation 이미지의 FP/FN, Class 및 BBox 오류를 검토한다. 성능이 낮다고 곧바로 재학습하지 않고 GT와 Prediction을 먼저 비교한다.
- Failure Analysis의 근거를 바탕으로 한 번에 주요 변수 하나를 바꾸고, Baseline과 Improved를 같은 Dataset Version·Validation에서 비교한다. 실패한 실험도 기록한다.
- GT 라벨을 고치면 Dataset Version을 갱신하고, Baseline/Improved 비교도 동일한 새 버전 기준으로 맞춘다. 현재 산출물의 `kimchi_v1`이 두 교과 7 Human QA 수정 이후 버전인지는 확인되지 않았다.
- Auto Label은 정답 라벨이 아니라 후보 TXT다. 사람이 교과 7 도구 등에서 검수·수정·승인한 뒤에만 정답으로 확정한다.

수업의 필수 흐름에는 Dataset Version, Baseline, Failure Analysis, 개선 및 비교, Final Model, Frozen Test, Detection/Auto Label-Human Review, README와 Acceptance Test가 포함된다. 대규모 Hyperparameter Search, Segmentation, 복잡한 Ensemble, TensorRT/Jetson 배포, 실시간 제조 판정, Dashboard는 교과 8 필수범위가 아니다.

## 이후 작업에서 지킬 조건

- 작업 대상은 사용자가 연결 경로라고 알려준 Desktop 복사본 `C:\Users\user1\Desktop\group2-project`다. 이 복사본에는 현재 `.git` 메타데이터가 없어 브랜치·원격·미커밋 상태를 여기서 확인할 수 없다. 실제 Git 작업은 올바른 저장소 경로를 먼저 확인한다.
- 답변은 한국어로 간결하게 한다. 요청한 파일·기능만 변경하고, 코드 변경 전 관련 수업 기준·기존 코드·설정·데이터 흐름을 확인한다.
- 사용자 작성 변경을 덮어쓰거나 삭제하지 않는다. 요청 없이 commit/push/merge 등 Git 이력을 변경하지 않는다. 수업 팀 협업 문서는 작업 Branch→PR→PM 검토/병합을 안내하지만, 실제 Git 변경은 사용자의 별도 요청과 현재 저장소 규칙을 따른다.
- Class ID, 라벨 형식, Manifest 컬럼·출처·상태, 이미지/TXT basename Pair, Split 역할과 인코딩을 보존한다. 라벨·Manifest·Split 변경은 영향 범위와 파생 보고서를 먼저 확인하고 관련 산출물의 정합성을 점검한다.
- 교과 8 수행가이드 목차 9의 공통 코딩 규칙을 적용한다: 모든 스크립트는 프로젝트 루트에서 실행하고, `Path(__file__).resolve().parents[1]` 기준으로 프로젝트 루트를 계산해 필요한 경우 `sys.path`에 추가한다. 함수 실행은 `main()`에 모으고 `if __name__ == "__main__":` 가드를 둔다(특히 Windows의 PyTorch/Ultralytics worker 안전성). 절대 경로를 여기저기 하드코딩하지 말고 `Path`로 경로를 구성한다. 실행 전에 필요한 입력이 존재하는지 확인하고, 결과를 콘솔에만 출력하지 말고 CSV/MD 등 파일로 저장한다. 코드 변경은 작은 단위로 계획하고 실행 결과를 확인·기록하며, 오류는 재현·증거를 바탕으로 원인을 좁혀 한 가지씩 수정한다.
- 수업의 반복 개발 기준은 `PLAN → CODE → RUN → CHECK → RECORD`다. 단, 프로젝트의 `AGENTS.md`에 적힌 테스트 실행 조건을 우선한다. 실행하지 않은 검증을 통과했다고 말하지 않는다.
- 데이터 재생성, 대량 변경/삭제, 기능 추가, 불필요한 UI·설정 수정은 요청 없이 진행하지 않는다. 변경 이유·영향 파일·부작용을 설명한다.

## 문서·구현·진행 상태의 차이 및 확인 필요

1. **Class 4 규정:** 교과 7 수행가이드와 실행 체크리스트, 저장소의 `project7/docs/class_guide.md`는 기존 Class 4를 자동 삭제하지 않고 REVIEW/교차검수 대상으로 두라고 설명한다. 반면 QA Summary와 두 Manifest 행은 특정 두 건의 Class 4 BBox를 Human QA 후 삭제했다고 기록한다. 이 개별 예외가 일반 기준으로 확대되었는지는 확인되지 않았다.
2. **`scene_type=none`:** 교과 7 수업 기준의 대표 장면 유형은 네 가지이며 `none`은 목록에 없다. 교과 8 수행가이드·README도 장면 목록을 `kimchi_with_target`, `normal_kimchi`, `object_only`, `other_review`로 적는다. 현재 QA와 Manifest에는 김치·대상 객체가 모두 없는 두 사례가 `none`으로 들어가 있고, 교과 8 감사 코드는 해당 파일을 알려진 빈 라벨 예외로 취급한다. `none`을 공식 허용값으로 일반화할지, 특정 예외로 유지할지는 기준 확인이 필요하다.
3. **Split 구성:** `project8/README.md`와 현재 Split Report/Manifest는 T02·T04·T17 일부를 중심으로 Val을 설명하지만 `project8/docs/split_policy.md` 최종 결과 표는 T03_V·T04_V·T12_V·T13_V·T17_V를 기재한다. 동결 Manifest와 팀 결정/PR 기록을 확인하기 전 어느 목록도 임의로 정답 처리하지 않는다.
4. **Dataset Version:** 수업 기준상 GT 수정 후 새 버전 및 재비교가 필요하다. 저장소 Manifest는 `kimchi_v1`로 표시되나, 현재 QA Summary의 Human QA 수정이 이 버전에 반영됐는지는 복사본만으로 확인할 수 없다.
5. **진행 상태:** 사용자는 현재 교과 8 `dataset_audit` 진행 중이라고 했다. 복사본에는 Split Manifest/보고서와 README의 Split Freeze 기록도 있지만, 이 파일들의 존재만으로 Audit 승인·최종 Freeze 완료를 확정하지 않는다. README의 Baseline 이후 산출물 다수는 TBD다.
6. **데이터 저장소/보안:** 교과 7 자료는 JPG/TXT/WORK/FINAL을 Git·메신저·개인 클라우드에 올리지 말라고 한다. 교과 8 메인 가이드는 기업 원본·파생 Dataset, Weight, 민감 Demo 결과의 공개 GitHub 업로드를 금지하고 기관 보안정책 우선을 명시한다. 반면 `project8/README.md`는 2026-10-13 강사 확인으로 라벨 TXT·CSV 허용이라고 기록한다. 이는 공개 여부·저장소 종류·파일 범위가 다른 규칙일 수 있어 포괄 허가로 간주하지 말고 적용 범위를 확인한다.
7. **Handoff:** `project7/docs/subject08_handoff.md`는 현재 생성 안내만 있는 상태다. `scripts/08_handoff.py`는 FINAL을 읽어 내용을 생성하므로, 실제 실행 후 검토·보완 여부를 별도로 확인해야 한다.
8. **데이터 파일 부재:** Desktop 복사본의 이미지·라벨 디렉터리는 비어 있고 구조만 첨부되어 있다. 현재 QA Summary/Manifest 등 문서 산출물은 읽을 수 있지만 원본 JPG/TXT 자체를 열어 사례를 재판정하거나 Audit/해시 검증을 다시 수행할 수는 없다.

## 참고한 수업 자료

연결된 Obsidian 자료에서 수행가이드와 관련 기준/보조 문서를 검토했다. 상세 내용을 복사하지 않고 위 원칙과 미해결 차이만 반영했다.

- 교과 7 수행가이드·산출물 기준 최종 수정본, 산출물 예시, 팀 실행 체크리스트 PDF, 코드 분석 공부자료 PDF
- 교과 8 Vision 객체검출 프로젝트 수행가이드, FINAL Dataset 분할 기준, 파트 분배, GitHub Branch 협업 가이드
- 교과 8 데이터 사용 참고, 산출물 예시, 산출물 설명, YOLO11n 학습부터 최종 결과까지, 성능평가 참고, 컨베이어 밸트 전체 흐름 설명
- 2조 팀 규칙의 검수 오류 대응·보고체계·Git/팀원 시작 가이드와 교과 7·8 작업 기록

## 프로젝트 복사본에서 확인한 주요 파일

- 교과 7: `project7/README.md`, `project7/scripts/02_validate.py`, `05_qa_summary.py`, `07_build_final.py`, `08_handoff.py`, `project7/configs/classes.yaml`, `project7/docs/class_guide.md`, `project7/reports/qa_summary.md`, `project7/manifests/manifest_final.csv`
- 교과 8: `project8/README.md`, `project8/src/dataset_audit.py`, `project8/src/split_builder.py`, `project8/scripts/02_audit_subject7_final.py`, `03_build_split.py`, `project8/configs/dataset.yaml`, `project8/docs/split_policy.md`, `project8/reports/split_report.md`, `project8/manifests/split_manifest.csv`