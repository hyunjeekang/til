# Git Merge Conflict

`push`를 하려는데 내 브랜치가 업데이트 되어 있지 않아서 (`master`보다 뒤쳐져 있어서(commits behind)) 충돌이 났을 때 해결하는 방법

---

### 1. master 최신화

`master`를 원격 저장소와 맞춰주기

```bash
git checkout master
git pull origin master
```

<br>

### 2. 내 브랜치에 합치기

`master` 내용을 작업 브랜치로 가져오기

```bash
git checkout {branch_name}
git merge master
# 여기서 CONFLICT 메시지 발생
```

<br>

### 3. 충돌 해결 (w/ VS Code)

1. 충돌 발생 파일 열기
2. 코드 내 `<<<<<<<`, `>>>>>>>`, `=======` 표시 찾기
3. 옵션 중 하나 선택하기
   - **Accept Current Change** : 내가 수정한 코드 살리기
   - **Accept Incoming Change** : `master`에서 가져온 코드 살리기
   - **Accept Both Changes** : 둘 다 남기기 (직접 수정)

<br>

### 4. 해결 후 푸시

```bash
git add .
git commit -m "fix: resolve merge conflicts"
git push origin {branch_name}
```

Merge 버튼이 활성화됐다면 충돌 해결 성공!
