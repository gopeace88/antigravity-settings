---
date: 2026-01-17
source: Claude AI 대화
tags: [antigravity, vscode, sync, git, windows, linux, extensions]
---

# Antigravity 이기종간 환경 동기화 가이드

## 개요

Google Antigravity는 VS Code 기반 에디터이지만, **로그인해도 설정이 자동 동기화되지 않는다**. 여러 시스템(Windows/Linux/WSL)에서 동일한 환경을 유지하려면 **Git을 통한 수동 동기화**가 필요하다.

### 동기화 대상

| 항목 | 동기화 방법 |
|------|------------|
| 설정 파일 (settings.json) | Git repo |
| Extensions 목록 | Git repo (목록만) |
| API Keys | 환경변수 (동기화 ❌) |
| 캐시/런타임 | 동기화 불필요 |

---

## 1. 설정 파일 위치

### Windows
```
C:\Users\{username}\.antigravity\
├── User/
│   └── settings.json    # 주요 설정
├── extensions/          # 설치된 extensions
├── argv.json
└── extensions.txt       # extension 목록 (직접 생성)
```

### Linux
```
~/.antigravity/
├── User/
│   └── settings.json
├── extensions/
├── argv.json
└── extensions.txt
```

> 참고: `~/.config/Antigravity/`는 캐시/런타임 폴더로 동기화 불필요

---

## 2. Windows에서 초기 설정 (최초 1회)

### 2.1 Extensions 목록 추출

```powershell
cd C:\Users\{username}\.antigravity
ls extensions | Select-Object -ExpandProperty Name > extensions.txt
```

### 2.2 .gitignore 생성

```powershell
echo "extensions/" > .gitignore
echo "extensions.backup*/" >> .gitignore
```

### 2.3 Git 초기화 및 Push

```powershell
git init
git config user.name "your-name"
git config user.email "your-email@example.com"
git add .
git commit -m "init antigravity config from windows"
git branch -M main
git remote add origin https://github.com/{username}/antigravity-settings.git
git push -u origin main
```

---

## 3. Linux에서 설정 가져오기

### 3.1 기존 폴더 백업 및 Clone

```bash
# 기존 폴더 백업
mv ~/.antigravity ~/.antigravity.bak

# Clone
git clone https://github.com/{username}/antigravity-settings.git ~/.antigravity

# 기존 extensions 복원 (Linux에서 이미 설치된 것들)
cp -r ~/.antigravity.bak/extensions ~/.antigravity/
```

### 3.2 Extensions 일괄 설치

```bash
cd ~/.antigravity

# extension ID에서 버전 번호 제거 후 설치
cat extensions.txt | sed 's/-[0-9.]*$//' | while read ext; do
  code --install-extension "$ext" 2>/dev/null || echo "설치 실패: $ext"
done
```

---

## 4. 설정 변경 후 동기화

### Windows에서 변경 후 Push

```powershell
cd C:\Users\{username}\.antigravity
git add .
git commit -m "update settings"
git push
```

### Linux에서 Pull

```bash
cd ~/.antigravity
git pull
```

### Linux에서 변경 후 Push

```bash
cd ~/.antigravity
git add .
git commit -m "update settings from linux"
git push
```

### Windows에서 Pull

```powershell
cd C:\Users\{username}\.antigravity
git pull
```

---

## 5. 동기화하면 안 되는 항목

다음 항목들은 **OS별로 다르므로 동기화하면 안 된다**:

| 항목 | 이유 | 해결 방법 |
|------|------|----------|
| API Keys | 보안 | 환경변수로 관리 |
| 바이너리 경로 (`/usr/bin`, `C:\`) | OS 의존 | 상대경로 또는 환경변수 |
| GPU/런타임 옵션 | 하드웨어 의존 | 머신별 설정 |
| Shell 경로 | OS 의존 | 환경변수 |

### API Key 설정 예시

**Windows (PowerShell Profile)**
```powershell
# $PROFILE 편집
$env:ANTHROPIC_API_KEY = "sk-xxx"
$env:OPENAI_API_KEY = "sk-xxx"
```

**Linux (~/.bashrc)**
```bash
export ANTHROPIC_API_KEY="sk-xxx"
export OPENAI_API_KEY="sk-xxx"
```

**settings.json에서 환경변수 참조**
```json
{
  "anthropic_api_key": "${ANTHROPIC_API_KEY}",
  "openai_api_key": "${OPENAI_API_KEY}"
}
```

---

## 6. 문제 해결

### Windows 경로가 Linux에서 깨지는 경우

```bash
cd ~/.antigravity
grep -ri "C:\\\\" . 2>/dev/null
grep -ri "Users" . 2>/dev/null
```

Windows 경로 발견 시 수동 수정 필요.

### Extension 설치 실패

일부 extension은 OS별로 다른 버전이 필요할 수 있다. 실패한 extension은 Antigravity 내에서 수동 설치.

---

## 7. 체크리스트

- [ ] GitHub에 private repo 생성 (`antigravity-settings`)
- [ ] Windows에서 extensions.txt 생성
- [ ] Windows에서 .gitignore 생성 (extensions 폴더 제외)
- [ ] Windows에서 git init 및 push
- [ ] Linux에서 clone
- [ ] Linux에서 extensions 일괄 설치
- [ ] API Key는 환경변수로 분리
- [ ] OS 의존 경로 제거/수정

---

## 참고

- Extensions 폴더 자체는 용량이 크므로 Git에서 제외
- Extensions 목록(extensions.txt)만 동기화하고 각 머신에서 설치
- VS Code Settings Sync와는 별개 (Antigravity 자체 설정은 동기화 안 됨)
