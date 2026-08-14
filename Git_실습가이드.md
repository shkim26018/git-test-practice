# Git 실습 가이드 — ISP 레지스터 튜닝 산출물 버전관리

> 대상: Git을 한 번도 써본 적 없는 사용자
> 환경: 일반 Git (CLI) + VS Code, GitHub/GitLab 등 어떤 원격 서비스에도 적용 가능
> 함께 보기: `Git_기초_교육자료.pptx` (개념 설명용)

---

## 0. 사전 준비

1. Git 설치 확인
   ```
   git --version
   ```
   버전이 출력되지 않으면 [git-scm.com](https://git-scm.com/download/win)에서 설치.

2. VS Code 설치 확인 (이미 있다면 생략). VS Code는 Git이 내장되어 있어 별도 플러그인 없이 바로 사용 가능.

3. (팀 협업 시) GitHub 또는 GitLab 계정 준비 — 원격 저장소를 어디에 둘지에 따라 선택.

---

## 1. 최초 설정 (컴퓨터당 1회만)

Git에게 "누가 커밋했는지" 알려주는 설정입니다. 모든 커밋 이력에 이름/이메일이 기록됩니다.

```bash
git config --global user.name "홍길동"
git config --global user.email "shkim26@nextchip.com"
```

확인:
```bash
git config --global --list
```

---

## 2. 로컬 저장소 만들기

### 방법 A — 새 프로젝트를 처음 시작하는 경우
```bash
cd D:\isp-tuning-project
git init
```
`.git` 이라는 숨김 폴더가 생기며, 이 폴더가 있는 위치부터가 하나의 저장소입니다.

### 방법 B — 이미 존재하는 원격 저장소를 받아오는 경우
```bash
git clone https://github.com/회사계정/isp-tuning-project.git
```

---

## 3. 기본 명령어 실습

작업 순서는 항상 **수정 → status 확인 → add → commit** 흐름을 따릅니다.

### 3.1 현재 상태 확인
```bash
git status
```
수정되었지만 아직 커밋되지 않은 파일 목록을 보여줍니다.

### 3.2 변경사항 스테이징 (커밋 대상 선택)
```bash
git add register_config.json      # 특정 파일만
git add .                         # 변경된 전체 파일
```

### 3.3 커밋 (이력으로 저장)
```bash
git commit -m "[Gamma] 저조도 계조 개선 - Lux 5 조건 확인"
```
커밋 메시지는 **무엇을 왜 바꿨는지** 알 수 있게 작성 (`update`, `수정` 같은 메시지는 지양).

### 3.4 이력 확인
```bash
git log                    # 상세 이력
git log --oneline --graph  # 한 줄 요약 + 브랜치 그래프
```

### 3.5 변경 내용 비교
```bash
git diff                   # 아직 add하지 않은 변경사항
git diff --staged          # add했지만 아직 커밋 안 한 변경사항
git diff HEAD~1 HEAD       # 최근 두 커밋 사이의 차이
```
레지스터 값이 텍스트(csv/json 등)라면 `git diff`로 **어떤 파라미터가 어떻게 바뀌었는지** 한눈에 확인 가능.

---

## 4. .gitignore 설정 — 불필요한 파일 제외

프로젝트 루트에 `.gitignore` 파일을 만들고 아래처럼 작성합니다.

```
# 원본 캡처/대용량 바이너리
*.raw
*.bin
*.mp4
*.avi
/capture_images/

# 로그, 빌드 산출물
*.log
/build/
/output/

# 임시/캐시 파일
__pycache__/
*.pyc
Thumbs.db
```

레지스터 설정값(csv/json/ini)처럼 **텍스트로 관리 가능한 산출물은 반드시 git으로 추적**하고, 이미지·동영상 같은 원본 캡처 파일은 제외해 저장소를 가볍게 유지합니다.

---

## 5. 브랜치 활용 — 튜닝 실험을 안전하게 분리

메인 이력(`main`)을 건드리지 않고 실험하고 싶을 때 브랜치를 사용합니다.

```bash
# 브랜치 생성 + 바로 전환 (가장 많이 씀)
git checkout -b tuning/sensorA-lowlight

# ... 레지스터 값 수정, add, commit 반복 ...

# 실험이 끝나고 검증되면 main으로 돌아가서 병합
git checkout main
git merge tuning/sensorA-lowlight
```

| 명령어 | 설명 |
|---|---|
| `git branch` | 현재 브랜치 목록 확인 |
| `git branch <이름>` | 새 브랜치 생성 (전환 안 함) |
| `git checkout <브랜치>` | 다른 브랜치로 전환 |
| `git checkout -b <브랜치>` | 생성 + 전환 동시에 |
| `git merge <브랜치>` | 현재 브랜치에 다른 브랜치 내용 병합 |

---

## 6. 원격 저장소 연동 (팀 공유)

```bash
# 최초 1회: 원격 저장소 주소 등록
git remote add origin https://github.com/회사계정/isp-tuning-project.git

# 로컬 커밋을 원격에 업로드
git push -u origin main     # 최초 1회는 -u 옵션 사용
git push                    # 이후에는 이것만

# 원격의 최신 변경사항을 로컬로 가져오기
git pull
```

---

## 7. 실전 시나리오 — ISP 레지스터 튜닝 예제

1. 프로젝트 폴더에서 저장소 초기화
   ```bash
   cd D:\isp-tuning-project
   git init
   ```
2. 초기 레지스터 설정 파일(`register_config.json`) 작성 후 최초 커밋
   ```bash
   git add register_config.json
   git commit -m "Initial register configuration baseline"
   ```
3. 저조도 튜닝을 위한 실험 브랜치 생성
   ```bash
   git checkout -b tuning/sensorA-lowlight
   ```
4. 게인/노이즈 관련 레지스터 값 수정 후 변경 내용 확인
   ```bash
   git diff
   ```
5. 의미 있는 변경 단위로 커밋
   ```bash
   git add register_config.json
   git commit -m "[Noise] 3DNR strength 상향, SNR +2dB 개선 - Lux 3 조건"
   ```
6. 여러 차례 튜닝 반복 (수정 → diff 확인 → commit) 후, 결과가 좋으면 main에 병합
   ```bash
   git checkout main
   git merge tuning/sensorA-lowlight
   ```
7. 팀 원격 저장소에 반영
   ```bash
   git push
   ```

---

## 8. VS Code에서 Git 사용하기 (명령어 없이)

1. `Ctrl+Shift+G` → 소스 제어(Source Control) 탭 열기
2. 변경된 파일 목록이 자동으로 표시됨
3. 파일 옆 `+` 클릭 → Stage (= `git add`)
4. 상단 메시지 입력창에 커밋 메시지 작성 → `✓ Commit`
5. 하단 상태바의 **Sync Changes / ↑↓ 아이콘** 클릭 → push/pull
6. 좌하단 브랜치 이름 클릭 → 브랜치 생성/전환 가능
7. 파일을 클릭하면 변경 전/후가 좌우로 비교되어 표시됨 (diff 뷰)

---

## 9. 자주 발생하는 상황과 해결법

| 상황 | 해결 명령어 |
|---|---|
| 방금 쓴 커밋 메시지를 고치고 싶음 | `git commit --amend -m "새 메시지"` |
| 마지막 커밋을 취소하되 수정 내용은 남기고 싶음 | `git reset --soft HEAD~1` |
| 특정 파일을 마지막 커밋 상태로 되돌리고 싶음 | `git restore <파일명>` |
| 실수로 큰 바이너리 파일을 커밋함 | `.gitignore`에 추가 후 `git rm --cached <파일명>` |
| 브랜치 병합 중 충돌(conflict) 발생 | 파일 내 `<<<<<<<` / `=======` / `>>>>>>>` 구간을 직접 수정 → `git add` → `git commit` |
| 원격 저장소 최신 내용과 내 로컬이 다름 | `git pull` 먼저 실행 후 다시 `push` |

---

## 10. 명령어 치트시트

| 명령어 | 설명 |
|---|---|
| `git init` | 현재 폴더를 새 저장소로 초기화 |
| `git clone <url>` | 원격 저장소 복제 |
| `git status` | 현재 상태 확인 |
| `git add <파일>` / `git add .` | 스테이징 |
| `git commit -m "메시지"` | 커밋 |
| `git log --oneline --graph` | 이력 확인 |
| `git diff` | 변경 내용 비교 |
| `git branch` / `git checkout -b <이름>` | 브랜치 목록 / 생성+전환 |
| `git merge <브랜치>` | 병합 |
| `git remote add origin <url>` | 원격 저장소 등록 |
| `git push` / `git pull` | 업로드 / 다운로드 |
| `git restore <파일>` | 파일을 마지막 커밋 상태로 되돌림 |

---

## 11. 실습 체크리스트

- [ ] `git --version`으로 설치 확인
- [ ] `git config --global user.name/email` 설정 완료
- [ ] 테스트 폴더에서 `git init` 실행
- [ ] 파일 하나 만들고 add → commit 성공
- [ ] `git log`로 커밋 이력 확인
- [ ] `.gitignore` 작성 및 적용 확인
- [ ] 브랜치 생성 → 수정 → 커밋 → main에 merge 성공
- [ ] (해당 시) 원격 저장소에 push/pull 성공
- [ ] VS Code 소스 제어 탭에서 동일 작업 수행해보기
