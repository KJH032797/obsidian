## 1. 900장 분할 예시

교과 7에서 검수·확정한 FINAL Dataset 900장을 교과 8의 모델 개발 목적에 따라 네 가지 역할로 분류

아래 수량은 **예시**이며, 실제 분할은 Dataset Audit 결과를 확인한 뒤 확정

![[dataset_split.png]]

|구분|예시 수량|역할|
|---|---|---|
|Train|약 600장|모델이 실제로 학습하는 데이터. Baseline / Improved 학습에 반복 사용|
|Validation|약 75장|개발 중 모델 성능을 비교하고 개선 실험에 사용|
|Frozen Test|약 75장|Final Model을 선택한 뒤 교과 8의 최종 성능을 확인|
|Bridge Holdout|약 150장|교과 8의 학습·튜닝·모델 선택·Frozen Test에 사용하지 않고 교과 14 연결용으로 보존|
|**합계**|**900장**|900장 모두 네 역할 중 하나에 배정|

## 2. 각 데이터를 분류하는 기준

900장을 단순히 무작위로 나누지 않는다. 먼저 기존 출처와 Split 정보, 촬영 유사성, Class 및 장면 분포를 확인한 뒤 분할 정책을 정한다.

### 2-1. 확인할 항목

- `source_dataset`: Dataset 1인지 Dataset 2인지
    
- `original_split`: 원래 `train`인지 `validation`인지
    
- `capture_group`: 같은 촬영 묶음에 속하는 이미지인지
    
- `Class` 분포: 특정 클래스가 한 역할에만 몰리는지
    
- `scene_type` 분포: 특정 장면 유형이 한 역할에만 몰리는지
    
- 중복·유사 이미지: 동일하거나 거의 같은 이미지가 여러 역할에 나뉘는지
    

### 2-2. 기존 데이터의 우선 검토 방향

- 기존 `Dataset 2 / validation`은 **Validation 후보로 우선 검토**한다.
    
- 기존 `Dataset 1 / train`과 `Dataset 2 / train`은 **Train / Frozen Test / Bridge Holdout 후보 Pool**로 검토한다.
    
- 위 방향은 고정 공식이 아니다. 실제 수량, 촬영 그룹, Class 분포, `scene_type` 분포를 확인한 뒤 최종 확정하고, 선택 근거를 `docs/split_policy.md`에 기록한다.
    

### 2-3. 데이터 누수(Data Leakage) 방지

거의 같은 장면을 연속 촬영한 이미지가 Train과 Frozen Test 등에 나뉘어 들어가면 모델 성능이 실제보다 높게 측정될 수 있다.

따라서 가능하면 **같은** `**capture_group**`**은 하나의 역할에만 배정**한다.

예를 들어 같은 촬영 묶음의 이미지 10장이 있다면, 일부는 Train에 넣고 나머지는 Frozen Test에 넣는 식으로 분리하지 않는다. 파일명에 시간 정보가 있다는 이유만으로 촬영 묶음이라고 단정하지 말고 실제 의미를 확인한다.

### 2-4. 분할 확정 절차

1. 교과 7 FINAL Dataset과 Dataset Manifest를 확인한다.
    
2. `source_dataset`, `original_split`, `scene_type`, `capture_group` 정보를 점검한다.
    
3. 중복·유사 촬영 이미지와 Class / `scene_type` 분포를 확인한다.
    
4. Train / Validation / Frozen Test / Bridge Holdout의 배정 기준을 `docs/split_policy.md`에 기록한다.
    
5. 900장 전체에 각 역할을 정확히 하나씩 부여해 `manifests/split_manifest.csv`를 작성한다.
    
6. 역할별 수량과 분포, 중복·누수 여부를 확인한 뒤 Split을 Freeze한다.
    

### 2-5. 반드시 지킬 원칙

- 교과 7 FINAL Dataset은 그대로 보존하고, 교과 8용 데이터 구성을 별도로 만든다.
    
- 900장 모두가 `train`, `val`, `test`, `bridge_holdout` 중 정확히 하나에 속해야 한다.
    
- 동일 이미지나 유사 촬영 그룹이 여러 역할에 섞이지 않도록 한다.
    
- Baseline과 Improved 비교 중에는 Split을 임의로 변경하지 않는다.
    
- Frozen Test는 Final Model 선택 후 최종 평가에 사용한다. 결과를 보고 모델을 다시 튜닝하지 않는다.
    
- Bridge Holdout은 교과 8의 학습·튜닝·모델 선택·Frozen Test에 사용하지 않는다.
    
- 실제 분할 비율보다 데이터 누수 방지, 각 데이터 역할 준수, 분할 근거 기록이 중요하다.
    

### 2-6. 관련 산출물

- `docs/split_policy.md`: 분할 기준과 선택 근거
    
- `manifests/split_manifest.csv`: 900장 전체의 분할 결과
    
- `manifests/train.csv`
    
- `manifests/val.csv`
    
- `manifests/test.csv`
    
- `manifests/bridge_holdout.csv`
    
- `reports/dataset_summary.csv`: 역할별 수량과 Class / `scene_type` 분포 요약


[[교과8_Vision_객체검출_프로젝트_수행가이드#예시 2 — `dataset_audit.py`를 만든다면|이 파트 담당의 작성 코드 1]]
[[교과8_Vision_객체검출_프로젝트_수행가이드#예시 3 — `split_builder.py`를 만든다면|이 파트 담당의 작성 코드 2]]

이외 같은 파일의 목차 6-12 확인