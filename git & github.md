### 1. 왜 쓸까
작업한 코드들 안전하게 기록 보관 가능. 실수하면 복구가능. 히스토리 확인가능. 버전관리를 위한 변경관리 툴이다. git이 파일 기록해두는 장소를 repository라고 한다. repository를 온라인에 만들면 컴퓨터가 고장나도 안심할 수 있겠다. 또한 협업도 가능하겠다. 그래서 쓴다. (온라인 repository = 원격 저장소) 
### 2. 설치 
맥북에서의 설치는 공식적으로는 Homebrew 또는 MacPorts 이걸 통해서 설치하라고 권장 중.
아래와 같이 설치한다. 혼자 사용할 땐 기본 브랜치 이름을 master에서 main으로 바꿔주면 편하다. 
```bash
brew install git
git config --global init.defaultBranch main
```
### 3. git
<mark>git init</mark> repository 생성  
<mark>git add</mark> 파일명 파일의 현재 상태를 기록 (git add . 는 모든 파일 스테이징)
<mark>git commit -m</mark> "메모" 버전생성/기록 
<mark>git status</mark> 상태확인
<mark>git log --all --oneline</mark> commit 내역 조회
<mark>git diff</mark> 최근 commit vs 현재파일 차이점 보여줌
<mark>git difftool</mark> 최근 commit vs 현재파일 차이점을 Vim 에디터로 보여줌. 뒤에 커밋id 입력가능 (h j k l 방향키)
<mark>git restore 파일명</mark> 최종 commit 상태로 파일 복구
<mark>git restore --source 커밋아이디 파일명</mark> 특정 commit 상태로 파일 복구 (파일 단위 복구)
<mark>git revert 커밋아이디</mark> 특정 commit 내용 미반영   
<mark>git revert 커밋아이디1 커밋아이디2</mark> 특정 commit 내용 미반영 . 여러개 작성 가능
<mark>git revert HEAD</mark> 최근 commit 내용 미반영  
<mark>git reset --hard 커밋아이디</mark> 특정 commit 상태로 일괄 복구 (저장소 전체 복구. 협업시엔 사용금지)
<mark>git reset --soft 커밋아이디</mark> 동일하게 리셋인데 변동사항 지우지 않고 스테이징 
<mark>git reset --mixed 커밋아이디</mark> 동일하게 리셋인데 변동사항 지우지 않고 un스테이징 
![668](image/Pasted%20image%2020260614175131.png)
1. `Staging area` 는 commit 하기 전에 commit 할 파일들을 모아놓는 곳
2. `repository` 는 commit 된 파일의 버전들을 모아놓는 곳
<small>* "스테이징 한다" 라는 건 commit 할 파일을 고르는 단계를 말한다. </small>
### 4. 브랜치
왜 쓸까? 기능을 추가할 때 파일 복사본 만들어서 먼저 코드를 작성해 보고 실행해 보기 위함. 복사본을 쉽게 만들 수 있다. 
```bash
git branch test #test 라는 브랜치 작성
git switch test #test 브랜치로 이동
git commit -m "test에 커밋"
git switch main #main 브랜치로 이동하면 test 브랜치에서 만든 파일은 사라짐
git log --oneline -all --graph #모든 브랜치에서 commit 한 내용을 보여줌
git merge test #정상작동하면 main에 합쳐야지. merge
```
main 브랜치와 test 브랜치에서 같은 부분을 수정한 경우 충돌이 발생한다. 
`CONFLICT (content): Merge conflict in app.txt`
원하는 식으로 코드를 수정하고 난 뒤 재 커밋하면 된다. 
협업 시 각자의 브랜치를 만들어서 기능 단위테스트 한뒤 커밋하는 식으로 진행한다.  
### 5. 다양한 git merge 방법
<mark>3-way-merge</mark> `main`에도 새 커밋 `C`가 있고, `newbranch`에도 새 커밋 `D`, `E`가 있으면 Git이 단순히 포인터만 옮길 수 없으므로 병합 커밋(아래 M)을 만들어 합치는 방식
```
A --- B --- C ------- M  main
      \             /
       D --- E ----
```
<mark>fast-forward merge</mark> **main에는 새 커밋이 없고, newbranch 에만 새 커밋이 있을 때**, 굳이 merge commit을 만들지 않고, `main` 위치만 `D`로 옮기는 방식 (log를 깔끔하게 관리하기 위해 사용한다.)
```
A --- B  main
      \
       C --- D  newbranch
       
A --- B --- C --- D  main
```
<mark>Rebase</mark> merge가 아니라 **커밋 위치를 다시 쌓는 작업**
```
A --- B --- C  main
      \
       D --- E  newbranch
       
A --- B --- C --- D' --- E'  main
```
<mark>Squash & Merge</mark> 여러 커밋을 **하나로 압축해서 합치는 방식**
```bash
git merge --squash newbranch
git commit -m "기능 추가"
```
```
A --- B  main
      \
       C --- D --- E  newbranch
A --- B --- F  main
```

정리하면,  
- 3-way merge: 양쪽 브랜치가 갈라져서 각각 작업했을 때 병합 커밋 생성
- fast-forward: main이 뒤처져 있을 때 포인터만 앞으로 이동
- rebase: 내 커밋을 main 최신 커밋 뒤로 다시 쌓음
- squash & merge: 여러 커밋을 하나로 합쳐서 main에 반영