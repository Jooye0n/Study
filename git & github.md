### 1. 왜 쓸까
작업한 코드들 안전하게 기록 보관 가능. 실수하면 복구가능. 히스토리 확인가능. 버전관리를 위한 변경관리 툴이다. git이 파일 기록해두는 장소를 repository라고 한다. repository를 온라인에 만들면 컴퓨터가 고장나도 안심할 수 있겠다. 또한 협업도 가능하겠다. 그래서 쓴다. (온라인 repository = 원격 저장소) 
### 2. 설치 
맥북에서의 설치는 공식적으로는 Homebrew 또는 MacPorts 이걸 통해서 설치하라고 권장 중.
아래와 같이 설치한다. 기본 브랜치 이름을 master에서 main으로 바꿔주면 편하다. 여러 이슈로 깃이 권장중.
```bash
brew install git
git config --global init.defaultBranch main
```
### 3. git
![668](image/Pasted%20image%2020260614175131.png)
1. `Staging area` 는 commit 하기 전에 commit 할 파일들을 모아놓는 곳
2. `repository` 는 commit 된 파일의 버전들을 모아놓는 곳
<small>* "스테이징 한다" 라는 건 commit 할 파일을 고르는 단계를 말한다. </small>
```bash
git init # repository 생성
git add 파일명 # 파일의 현재 상태를 기록
git add . # 모든 파일 스테이징
git commit -m "메모" # 버전 생성 및 기록
git remote add origin https://github.com/... # 원격 저장소 경로를 origin 변수에 저장
git push -u origin main # push 할 때 -u는 원격 저장소와 브랜치 정보를 기억
git push # 이후에는 push만 사용 가능
git status # 상태 확인
git log --all --oneline # commit 내역 조회
git diff # 최근 commit과 현재 파일의 차이점 조회
git difftool # 최근 commit과 현재 파일의 차이점을 Vim 에디터로 조회 (hjkl 방향키)
git difftool 커밋아이디 # 특정 commit과 비교
git restore 파일명 # 최종 commit 상태로 파일 복구
git restore --source 커밋아이디 파일명 # 특정 commit 상태로 파일 복구 (파일 단위 복구)
git revert 커밋아이디 # 특정 commit의 변경사항을 취소하는 commit 생성
git revert 커밋아이디1 커밋아이디2 # 여러 commit의 변경사항 취소
git revert HEAD # 가장 최근 commit의 변경사항 취소
git reset --hard 커밋아이디 # 특정 commit 상태로 저장소 전체 복구 (협업 시 사용 주의)
git reset --soft 커밋아이디 # commit만 되돌리고 변경사항은 스테이징 상태 유지
git reset --mixed 커밋아이디 # commit만 되돌리고 변경사항은 Unstaged 상태 유지
git fetch # 원격 저장소의 신규 commit을 로컬로 가져오기
git pull # 원격 저장소 내용을 가져와 자동 병합 (fetch + merge)
git pull origin main # 원격 저장소의 특정 브랜치 내용을 가져와 병합
```
### 4. 브랜치
왜 쓸까? 기능을 추가할 때 파일 복사본 만들어서 먼저 코드를 작성해 보고 실행해 보기 위함. 복사본을 쉽게 만들 수 있다. 
```bash
git branch test #test 라는 브랜치 생성
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
### 6. git flow 전략
git flow 전략은 크게 5개 브랜치를 운영한다. 
- main 브랜치
- develop 브랜치 (개발용)
- feature 브랜치 (develop에 기능추가용)
- hotfix 브랜치 (main 브랜치 버그해결용)
- 가끔 release 브랜치 (develop 브랜치를 main 브랜치에 합치기 전에 최종 테스트용)
1. develop 브랜치부터 생성한다. main 브랜치에 신기능을 바로 추가하는것은 너무 위험하다. 모든 개발은  develop 브랜치에서 진행한다. ![578](image/Pasted%20image%2020260615000825.png)
2. 신기능 개발은 feature 브랜치에서 진행한다.  feature/guild 브랜치 만들어서 길드기능 만들고 , feature/friend 브랜치 만들어서 친구기능 만들어 본다.완성되면 develop 브랜치에 merge 한다.  ![585](image/Pasted%20image%2020260615001024.png)
3. 신버전 출시 준비는 release 브랜치에서 진행한다. develop 에서 만든 기능들을 바로 main에 합치기엔 불안하다. release 브랜치에 복사한 다음 테스트나 QA 를 진행한다. 버그를 발견하면 임시 브랜치를 만들어서 수정한다. 
   ![583](image/Pasted%20image%2020260615001204.png)
4. 버그가 발견되면 main 브랜치에서 hotfix 브랜치를 만들어 버그를 수정하고, 수정이 완료되면 main브랜치에 바로 merge를 진행한다. ![586](image/Pasted%20image%2020260615001509.png)
### 7. trunk-based 전략
main 브랜치 하나만 잘 관리하자는 전략이다. github flow도 유사하다.
테스트를 많이 자주 해야한다. 테스트 자동화로 이어진다. 안정화된 프로젝트들이 잘 사용한다.  ![603](image/Pasted%20image%2020260615001936.png)
### 8. git stash
코드를 잠깐 보관하기 위해 사용한다. 주석처리를 해도 되지만 주석으로 처리하면 commit 할때 반영이 되어버린다. commit 할때도 반영이 되지 않도록 관리하고 싶다면 사용해보자. 브랜치를 새로 만들어서 코드를 짜는 것과 다를 바 없긴 하다. 
```bash
git stash #코드가 임시 공간으로 이동한다. (파일들은 최근 commit 상태로 되돌아간다.)
git stash save "메모 작성도 가능하다"
git stash list #현재 stash 되어있는 코드 목록을 전부 출력
git stash pop #보관했던 코드 다시 불러오기 (여러개면 가장 최근것부터 가져온다.)
git stash drop 삭제할id #특정 stash 삭제
git stash clear #모든 stash 삭제
git stash -p #일부 코드만 git stash
```
