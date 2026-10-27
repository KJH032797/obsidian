
> 대상: 교과 8 Vision 기반 객체검출 팀 프로젝트  
> 팀 구성: 5명  
> 개발 환경 예시: Windows 11 + WSL2 Ubuntu + VS Code  
> 협업 방식: `main` 브랜치 + 팀원별 작업 브랜치 + Pull Request + Merge

---

# 0. 먼저 전체 흐름을 이해합니다

이번 팀 프로젝트에서는 5명이 하나의 GitHub Repository를 공유하지만,  
**같은 브랜치에서 동시에 작업하지 않습니다.**

각자 자신의 Branch를 만들어 작업하고, 작업이 끝나면 GitHub에서 Pull Request를 만든 뒤 `main`에 병합합니다.

```text
팀장
GitHub Repository 생성
        ↓
팀원 4명 Collaborator 초대
        ↓
5명 모두 같은 Repository Clone
        ↓
각자 자신의 Branch 생성
        ↓
각자 파일 제작 · 수정
        ↓
Commit
        ↓
자기 Branch를 GitHub에 Push
        ↓
Pull Request 생성
        ↓
코드 확인
        ↓
main Branch에 Merge
        ↓
모든 팀원이 최신 main을 다시 Pull
```

핵심은 다음입니다.

```text
main
→ 항상 팀의 통합된 안정 버전

개인 Branch
→ 각 팀원이 개발하는 작업 공간
```

---

# PART 1. 팀장이 처음 프로젝트를 만듭니다

# 1. 프로젝트 폴더를 만듭니다

WSL 터미널에서 작업할 위치로 이동합니다.

```bash
cd ~
```

프로젝트 폴더를 만듭니다.

```bash
mkdir subject08_vision_project
cd subject08_vision_project
```

현재 위치를 확인합니다.

```bash
pwd
```

예:

```text
/home/user1/subject08_vision_project
```

---

# 2. 기본 프로젝트 폴더를 만듭니다

처음부터 역할별로 폴더를 나누어 두면 여러 명이 작업하기 편합니다.

```bash
mkdir -p src
mkdir -p docs
mkdir -p configs
mkdir -p scripts
mkdir -p reports
mkdir -p tests
mkdir -p data
mkdir -p models
```

기본 파일도 만듭니다.

```bash
touch README.md
touch requirements.txt
touch .gitignore
```

예상 구조:

```text
subject08_vision_project/
├── src/
├── docs/
├── configs/
├── scripts/
├── reports/
├── tests/
├── data/
├── models/
├── README.md
├── requirements.txt
└── .gitignore
```

---

# 3. 가상환경을 만듭니다

먼저 Python 버전을 확인합니다.

```bash
python3 --version
```

가상환경을 생성합니다.

```bash
python3 -m venv .venv
```

활성화합니다.

```bash
source .venv/bin/activate
```

정상적으로 활성화되면 터미널 앞에 보통 다음과 같이 표시됩니다.

```text
(.venv) user1@PC:~/subject08_vision_project$
```

## 만약 venv 오류가 발생한다면

Ubuntu에서 다음 패키지가 필요할 수 있습니다.

```bash
sudo apt update
sudo apt install python3-venv
```

특정 Python 버전을 사용하는 경우 예:

```bash
sudo apt install python3.14-venv
```

설치 후 기존에 불완전하게 만들어진 `.venv`가 있다면 삭제하고 다시 생성합니다.

```bash
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
```

---

# 4. 필요한 패키지를 설치합니다

예:

```bash
pip install --upgrade pip
pip install requests
```

설치한 패키지를 기록합니다.

```bash
pip freeze > requirements.txt
```

다른 팀원은 나중에 다음 명령으로 동일한 환경을 만들 수 있습니다.

```bash
pip install -r requirements.txt
```

---

# 5. `.gitignore`를 작성합니다

가상환경, 대용량 데이터, 모델 Weight, 임시파일 등을 Git에 올리지 않도록 합니다.

`.gitignore`

```gitignore
# Python
__pycache__/
*.pyc
*.pyo

# Virtual Environment
.venv/
venv/

# VS Code
.vscode/

# Jupyter
.ipynb_checkpoints/

# OS
.DS_Store
Thumbs.db

# Dataset
data/raw/
data/train/
data/val/
data/test/

# Model Weight
*.pt
*.onnx
*.engine

# Training outputs
runs/

# Logs
*.log

# Cache
.cache/
```

> 기업 제공 데이터나 보안 대상 데이터는 GitHub에 올리지 않습니다.  
> 코드와 공개 가능한 문서만 Repository에 올리고, 데이터는 기관 정책에 맞는 내부 저장공간을 사용합니다.

---

# PART 2. Git 저장소를 만듭니다

# 6. Git 저장소를 초기화합니다

```bash
git init
```

현재 상태를 확인합니다.

```bash
git status
```

기본 브랜치를 `main`으로 맞춥니다.

```bash
git branch -M main
```

---

# 7. 첫 번째 Commit을 만듭니다

모든 파일을 Git 관리 대상으로 추가합니다.

```bash
git add .
```

Commit 합니다.

```bash
git commit -m "chore: initialize subject08 project"
```

확인합니다.

```bash
git log --oneline
```

---

# PART 3. GitHub Repository를 만듭니다

# 8. 팀장이 GitHub에서 Repository를 생성합니다

GitHub에서 다음 순서로 진행합니다.

```text
GitHub 로그인
→ New repository
→ Repository name 입력
→ Private 또는 기관 정책에 맞는 공개범위 선택
→ Create repository
```

예:

```text
Repository name
subject08-vision-project
```

이미 로컬에서 `README.md`를 만들었으므로 GitHub Repository 생성 화면에서는  
가능하면 README 자동 생성을 하지 않고 빈 Repository로 만드는 것이 충돌을 줄일 수 있습니다.

---

# 9. 로컬 Repository와 GitHub를 연결합니다

GitHub에서 Repository 주소를 복사합니다.

예:

```text
https://github.com/teamname/subject08-vision-project.git
```

로컬 터미널에서 연결합니다.

```bash
git remote add origin https://github.com/teamname/subject08-vision-project.git
```

연결을 확인합니다.

```bash
git remote -v
```

처음 Push 합니다.

```bash
git push -u origin main
```

이제 GitHub에 프로젝트 초기 구조가 올라갑니다.

---

# PART 4. 팀원 4명을 초대합니다

# 10. Collaborator를 초대합니다

팀장이 GitHub Repository에서 다음 순서로 이동합니다.

```text
Repository
→ Settings
→ Collaborators
→ Add people
```

팀원 4명의 GitHub 계정 또는 이메일을 입력하여 초대합니다.

예:

```text
팀장: member1
팀원: member2
팀원: member3
팀원: member4
팀원: member5
```

팀원은 GitHub 알림이나 이메일에서 초대를 수락합니다.

---

# PART 5. 모든 팀원이 프로젝트를 Clone 합니다

# 11. 팀원은 Repository를 Clone 합니다

각 팀원은 자신의 PC에서 작업 폴더로 이동합니다.

```bash
cd ~
```

Repository를 Clone 합니다.

```bash
git clone https://github.com/teamname/subject08-vision-project.git
```

폴더로 이동합니다.

```bash
cd subject08-vision-project
```

확인합니다.

```bash
git status
```

---

# 12. 각 팀원도 자신의 가상환경을 만듭니다

`.venv`는 GitHub에서 내려받지 않습니다.

각 팀원 PC에서 별도로 생성합니다.

```bash
python3 -m venv .venv
source .venv/bin/activate
```

패키지를 설치합니다.

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

즉:

```text
GitHub에서 공유
→ 소스코드
→ requirements.txt

GitHub에서 공유하지 않음
→ .venv
```

---

# PART 6. 5명이 각자 Branch를 만듭니다

# 13. Branch 역할을 정합니다

예를 들어 다음처럼 역할을 나눌 수 있습니다.

| 팀원   | Branch                  | 담당 예시                |
| ---- | ----------------------- | -------------------- |
| 팀장   | `feature/project-base`  | 프로젝트 구조·공통 설정        |
| 팀원 2 | `feature/dataset-audit` | Dataset Audit        |
| 팀원 3 | `feature/train`         | YOLO 학습              |
| 팀원 4 | `feature/evaluation`    | 평가·Failure Analysis  |
| 팀원 5 | `feature/inference`     | Detection·Auto Label |

Branch 이름은 역할이 보이도록 만드는 것이 좋습니다.

---

# 14. Branch를 만들기 전에 main을 최신 상태로 만듭니다

항상 먼저:

```bash
git switch main
git pull origin main
```

그 다음 자신의 Branch를 만듭니다.

예: 새 브랜치를 만들고, 그 브랜치로 바로 이동하라는 뜻

```bash
git switch -c feature/dataset-audit
```

현재 Branch를 확인합니다.

```bash
git branch
```

예:

```text
* feature/dataset-audit
  main
```

---

# PART 7. 각자 파일을 제작합니다

# 15. 팀원별 작업 예시

## 팀원 2 — Dataset Audit

```bash
touch src/dataset_audit.py
```

예:

```text
src/dataset_audit.py
```

이 팀원은 주로 이 파일을 작업합니다.

---

## 팀원 3 — 학습

```bash
touch src/train_detector.py
```

예:

```text
src/train_detector.py
```

---

## 팀원 4 — 평가

```bash
touch src/evaluate_detector.py
touch src/failure_analyzer.py
```

---

## 팀원 5 — 추론

```bash
touch src/inference.py
touch src/auto_label.py
```

---

# 16. 작업 중에는 자주 상태를 확인합니다

```bash
git status
```

예:

```text
modified: src/dataset_audit.py
```

변경 내용을 확인합니다.

```bash
git diff
```

---

# 17. 자신의 작업을 Commit 합니다

예:

```bash
git add src/dataset_audit.py
```

Commit:

```bash
git commit -m "feat: add dataset audit function"
```

여러 파일이라면:

```bash
git add .
git commit -m "feat: implement dataset audit"
```

Commit 메시지는 가능하면 무엇을 했는지 알 수 있게 작성합니다.

예:

```text
feat: add dataset audit
feat: add YOLO training script
fix: handle empty label file
docs: update dataset guide
test: add validation test
```

---

# PART 8. 자신의 Branch를 GitHub에 올립니다

# 18. 자신의 Branch를 Push 합니다

예:

```bash
git push -u origin feature/dataset-audit
```

이후 같은 Branch에서는 간단히:

```bash
git push
```

하면 됩니다.

중요:

```text
팀원 작업
→ 자신의 Branch에 Push

처음부터 main에 직접 Push
→ 권장하지 않음
```

---

# PART 9. Pull Request를 만듭니다

# 19. GitHub에서 Pull Request를 생성합니다

Push가 끝나면 GitHub Repository에 들어갑니다.

보통 다음 버튼이 나타납니다.

```text
Compare & pull request
```

또는:

```text
Pull requests
→ New pull request
```

다음과 같이 설정합니다.

```text
base:
main

compare:
feature/dataset-audit
```

즉:

```text
feature/dataset-audit
        ↓
main에 합쳐 주세요
```

라는 요청입니다.

---

# 20. Pull Request 내용을 작성합니다

예:

```md
## 작업 내용
- dataset_audit.py 작성
- JPG/TXT Pair 검사 추가
- YOLO Class 범위 검사 추가
- BBox 좌표 범위 검사 추가

## 확인 방법
```bash
python src/dataset_audit.py
```

## 결과
- Pair Error 확인 가능
- Empty Label 확인 가능
- Class별 BBox 수 출력 가능
```

Pull Request 제목 예:

```text
[Dataset] Dataset Audit 기능 구현
```

---

# PART 10. 다른 팀원이 코드를 확인합니다

# 21. 바로 Merge하지 않고 먼저 확인합니다

팀장이나 다른 팀원이 Pull Request에서 다음을 확인합니다.

```text
코드가 실행되는가?
        ↓
담당 기능이 제대로 구현되었는가?
        ↓
기존 기능을 망가뜨리지 않는가?
        ↓
파일 경로가 팀 기준과 맞는가?
        ↓
불필요한 Dataset이나 .venv가 올라오지 않았는가?
```

문제가 있으면 GitHub의 Review 기능으로 수정 요청을 남깁니다.

---

# PART 11. 수정 요청을 받은 경우

# 22. 자신의 Branch에서 수정합니다

예:

```bash
git switch feature/dataset-audit
```

파일을 수정한 후:

```bash
git add .
git commit -m "fix: correct class range validation"
git push
```

기존 Pull Request에 자동으로 새 Commit이 추가됩니다.

새 Pull Request를 다시 만들 필요는 없습니다.

---

# PART 12. main에 병합합니다

# 23. Pull Request가 승인되면 Merge 합니다

GitHub에서:

```text
Pull Request
→ Review 완료
→ Merge pull request
→ Confirm merge
```

이제:

```text
feature/dataset-audit
        ↓
main
```

으로 코드가 합쳐집니다.

---

# PART 13. 병합 후 모든 팀원이 최신 main을 받습니다

# 24. main으로 이동합니다

```bash
git switch main
```

최신 상태를 가져옵니다.

```bash
git pull origin main
```

이제 다른 팀원이 만든 기능도 자신의 PC에 들어옵니다.

예:

```text
src/
├── dataset_audit.py
├── train_detector.py
├── evaluate_detector.py
├── failure_analyzer.py
├── inference.py
└── auto_label.py
```

---

# PART 14. 작업 Branch를 정리합니다

# 25. Merge가 끝난 Branch는 삭제할 수 있습니다

로컬 Branch 삭제:

```bash
git branch -d feature/dataset-audit
```

GitHub 원격 Branch 삭제:

```bash
git push origin --delete feature/dataset-audit
```

반드시 삭제해야 하는 것은 아니지만, 작업이 끝난 Branch를 계속 쌓아두는 것보다 정리하는 편이 좋습니다.

---

# PART 15. 새로운 작업을 시작할 때

기존 Branch에서 계속 새 기능을 만들지 말고, 최신 `main`에서 새로운 Branch를 만듭니다.

```bash
git switch main
git pull origin main
git switch -c feature/failure-analysis
```

작업:

```bash
# 파일 수정
```

Commit:

```bash
git add .
git commit -m "feat: add failure analysis"
```

Push:

```bash
git push -u origin feature/failure-analysis
```

그리고 다시 Pull Request를 생성합니다.

---

# PART 16. 다른 팀원의 변경사항을 작업 중인 Branch에 반영합니다

예를 들어 팀원 2가 `feature/dataset-audit`에서 작업 중인데  
다른 기능들이 이미 `main`에 Merge되었다고 가정합니다.

먼저 원격 변경사항을 가져옵니다.

```bash
git fetch origin
```

현재 Branch를 확인합니다.

```bash
git branch
```

자신의 Branch에서 최신 main을 병합합니다.

```bash
git merge origin/main
```

충돌이 없다면 그대로 완료됩니다.

---

# PART 17. Merge Conflict가 발생한다면

# 26. 왜 Conflict가 발생하나요?

두 사람이 같은 파일의 같은 부분을 수정하면 Git이 어느 내용을 선택해야 할지 판단하지 못할 수 있습니다.

예:

```text
팀원 A
src/config.py 10번째 줄 수정

팀원 B
src/config.py 10번째 줄 수정

        ↓

Merge Conflict
```

충돌 파일을 열면 다음과 비슷하게 표시됩니다.

```text
<<<<<<< HEAD
batch = 8
=======
batch = 16
>>>>>>> origin/main
```

팀에서 어떤 값을 사용할지 결정한 뒤 직접 수정합니다.

예:

```python
batch = 16
```

그 다음:

```bash
git add src/config.py
git commit -m "merge: resolve config conflict"
```

---

# PART 18. 같은 파일을 여러 명이 동시에 수정하지 않는 방법

가장 좋은 방법은 역할을 명확히 나누는 것입니다.

예:

```text
팀원 1
→ configs/
→ README
→ 공통 구조

팀원 2
→ dataset_audit.py
→ split_builder.py

팀원 3
→ train_detector.py
→ experiment_runner.py

팀원 4
→ evaluate_detector.py
→ failure_analyzer.py

팀원 5
→ inference.py
→ auto_label.py
```

이렇게 하면 Merge Conflict가 크게 줄어듭니다.

---

# PART 19. 교과 8에 맞는 추천 Repository 구조

```text
subject08-vision-project/
│
├── src/
│   ├── dataset_audit.py
│   ├── split_builder.py
│   ├── train_detector.py
│   ├── evaluate_detector.py
│   ├── failure_analyzer.py
│   ├── experiment_runner.py
│   ├── inference.py
│   └── auto_label.py
│
├── configs/
│   └── dataset.yaml
│
├── docs/
│   ├── project_baseline.md
│   ├── split_policy.md
│   ├── qa_summary.md
│   └── final_report.md
│
├── reports/
│   ├── dataset_summary.csv
│   ├── audit_issue.csv
│   ├── split_manifest.csv
│   └── failure_analysis.csv
│
├── tests/
│
├── scripts/
│
├── data/
│   └── README.md
│
├── models/
│   └── README.md
│
├── .gitignore
├── requirements.txt
└── README.md
```

> 실제 Dataset과 `.pt` Weight를 GitHub에 올릴지는 반드시 기관의 보안 및 저장 정책을 먼저 확인합니다.

---

# PART 20. 5명 협업 전체 예시

```text
[팀장]

Repository 생성
        ↓
초기 폴더 구조 생성
        ↓
main Push
        ↓
팀원 초대


[5명 공통]

Repository Clone
        ↓
.venv 생성
        ↓
requirements 설치
        ↓
최신 main 확인
        ↓
각자 Branch 생성


[팀원별]

feature/project-base
feature/dataset-audit
feature/train
feature/evaluation
feature/inference

        ↓

각자 개발
        ↓
git add
        ↓
git commit
        ↓
git push
        ↓
Pull Request


[팀]

코드 리뷰
        ↓
문제 수정
        ↓
Merge
        ↓
main 통합


[모든 팀원]

git switch main
git pull origin main
        ↓
통합 결과 실행
        ↓
다음 작업 Branch 생성
```

---

# PART 21. 실제 하루 작업에서 반복하는 명령

팀원들이 가장 자주 사용하는 흐름입니다.

## 작업 시작

```bash
cd ~/subject08-vision-project
source .venv/bin/activate

git switch main
git pull origin main

git switch -c feature/작업이름
```

## 작업 후

```bash
git status
git add .
git commit -m "feat: 작업 내용"
git push -u origin feature/작업이름
```

## GitHub

```text
Pull Request 생성
→ 팀원 Review
→ Merge
```

## Merge 후

```bash
git switch main
git pull origin main
```

이 과정을 반복합니다.

---

# PART 22. 팀 작업에서 반드시 지켜야 할 규칙

1. `main`에서 직접 개발하지 않습니다.
2. 작업 시작 전 항상 최신 `main`을 Pull 합니다.
3. 기능 하나마다 별도 Branch를 만드는 것을 권장합니다.
4. Commit은 너무 큰 단위로 한 번에 하지 않습니다.
5. Commit 메시지만 보고 무엇을 수정했는지 알 수 있게 작성합니다.
6. Push 후 반드시 Pull Request를 만듭니다.
7. 다른 팀원의 코드를 확인한 뒤 Merge합니다.
8. `.venv`, Dataset, 대용량 Weight는 기본적으로 Git에 올리지 않습니다.
9. 같은 파일을 여러 사람이 동시에 수정하지 않도록 역할을 나눕니다.
10. Merge 후 모든 팀원이 최신 `main`을 다시 Pull 합니다.

---

# PART 23. 자가 체크리스트

## 팀장

- [ ] 프로젝트 폴더를 생성했다.
- [ ] Git 저장소를 초기화했다.
- [ ] `.gitignore`를 작성했다.
- [ ] `requirements.txt`를 작성했다.
- [ ] GitHub Repository를 생성했다.
- [ ] 초기 `main`을 Push했다.
- [ ] 팀원 4명을 Collaborator로 초대했다.

## 팀원 공통

- [ ] Repository를 Clone했다.
- [ ] `.venv`를 생성했다.
- [ ] `requirements.txt`로 패키지를 설치했다.
- [ ] 자신의 Branch를 생성했다.
- [ ] 담당 파일을 작성했다.
- [ ] Commit했다.
- [ ] 자신의 Branch를 Push했다.
- [ ] Pull Request를 만들었다.

## Merge 전

- [ ] 코드가 정상 실행된다.
- [ ] 다른 기능을 망가뜨리지 않는다.
- [ ] Dataset이나 `.venv`가 실수로 포함되지 않았다.
- [ ] Pull Request 설명을 작성했다.
- [ ] 팀원 Review를 받았다.

## Merge 후

- [ ] `main`에 정상 Merge되었다.
- [ ] 모든 팀원이 `git pull origin main`을 실행했다.
- [ ] 통합된 코드가 정상 실행된다.
- [ ] 다음 작업은 새로운 Branch에서 시작한다.

---

# 최종 핵심 정리

```text
한 Repository
        ↓
5명이 Clone
        ↓
각자 다른 Branch
        ↓
각자 개발
        ↓
Commit
        ↓
자기 Branch Push
        ↓
Pull Request
        ↓
Review
        ↓
Merge
        ↓
main 통합
        ↓
모든 팀원이 최신 main Pull
```

> **Branch는 각자의 작업 공간이고, `main`은 팀 전체의 최종 통합본입니다.**  
> 이 원칙만 지켜도 5명이 하나의 프로젝트를 훨씬 안전하게 협업할 수 있습니다.
