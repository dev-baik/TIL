## Git vs GitHub 차이점
- Git : 소스 코드 기록을 관리하고 추적할 수 있는 버전 제어 시스템

- GitHub : Git 저장소를 관리하는 클라우드 기반 호스팅 서비스
  - Git 저장소 호스팅 서비스 : 로컬 컴퓨터/서버 외부에서 Git 버전 제어 프로젝트를 추적하고 공유할 수 있는 온라인 데이터베이스

- - -

## Git Branch 종류 (5가지)
- [Gitflow Workflow](https://steemit.com/kr/@wnsgml972/kr-github)에서는 항상 유지되는 메인 브랜치들(master, develop)과 일정 기간 동안만 유지되는 보조 브랜치들(feature, release, hotfix)을 포함하여 총 5가지의 브랜치를 사용한다.

<img src="https://velog.velcdn.com/images/dev-baik/post/931e404b-da2c-4a51-bfed-32781bf8dadd/image.png"/>

### Master Branch
- 제품으로 출시될 수 있는 브랜치로, 배포(Release) 이력을 관리하기 위한 것으로만 사용된다.

### Develop Branch
- 다음 출시 버전을 개발하는 브랜치로, 기능 개발을 위한 브랜치들을 병합하기 위해 사용된다.
- 모든 기능이 추가되고 버그가 수정되어 배포 가능한 안정적인 상태라면 develop 브랜치를 master 브랜치에 병합한다.

### Feature Branch
- 기능을 개발하는 브랜치로, 새로운 기능 개발 및 버그 수정이 필요할 때마다 develop 브랜치로부터 분기한다.
- 해당 브랜치의 작업은 기본적으로 공유할 필요가 없기 때문에, 자신의 로컬 저장소에서 관리한다.
- 개발이 완료되면 develop 브랜치로 병합하여 다른 사람들과 공유한다.

### Release Branch
- WHAT) 이번 출시 버전을 준비하는 브랜치
- WHY) 배포를 위한 전용 브랜치를 사용함으로써 한 팀이 해당 배포를 준비하는 동안 다른 팀은 다음 배포를 위한 기능 개발을 계속할 수 있다.
1. develop 브랜치에서 배포할 수 있는 수준의 기능이 모이면 또는 정해진 배포 일정이 되면, release 브랜치를 분기한다.
    - release 브랜치를 만드는 순간부터 배포 사이클이 시작된다.
    - release 브랜치에서는 배포를 위한 최종적인 버그 수정, 문서 추가 등 릴리스와 직접적으로 관련된 작업을 수행한다.
    - 직접적으로 관련된 작업들을 제외하고는 release 브랜치에 새로운 기능을 추가로 병합(merge)하지 않는다.

2. release 브랜치에서 배포 가능한 상태가 되면(배포 준비가 완료되면), master 브랜치에 병합한다. (이때, 병합한 커밋에 Release 버전 태그를 부여!)
   - 배포 가능한 상태: 새로운 기능을 포함한 상태로 모든 기능이 정상적으로 동작 하는 상태
   - 배포를 준비하는 동안 release 브랜치가 변경되었을 수 있으므로 배포 완료 후 develop 브랜치에도 병합한다.

- 다음 번 배포(Release)를 위한 개발 작업은 develop 브랜치에서 계속 진행해 나간다.

### Hotfix Branch
- 출시 버전에서 발생한 버그를 수정 하는 브랜치로, 배포한 버전에 긴급하게 수정을 해야 할 필요가 있을 경우, master 브랜치에서 분기하는 브랜치이다.

- - -

## fetch와 pull의 차이
- fetch : 원격 저장소의 데이터를 로컬에 가져오기만 하기
- pull(fetch + merge) : 원격 저장소의 내용을 가져와 자동으로 병합 작업을 실행
- 즉, 단순히 원격 저장소의 내용을 확인만 하고 로컬 데이터와 병합은 하고 싶지 않은 경우에는 fetch 명령어를 사용한다.

## pull reqeust
- 기능 개발을 끝내고 main에 바로 병합하는 것이 아니라, 브랜치를 중앙 원격 저장소에 올리고 병합해달라고 요청하는 것
  - 중앙 원격 저장소 : 여러 명이 같은 프로젝트를 관리하는 데 사용하는 그룹 계정의 중립된 원격 저장소

## Merge
- 일반적으로 많이 사용되는 병합으로, 커밋 이력을 모두 남길 때 사용한다.

### Merge (Fast-Forward)
- 새로운 브랜치 my-branch 가 main 브랜치로부터 분기된 이후 main 브랜치에 새로운 커밋이 올라오지 않았다면, my-branch 가 main 와 비교하여 최신의 브랜치라고 할 수 있다. 이런 경우 my-branch 의 변경 이력을 그대로 main 으로 가져올 수 있는데, 이를 Fast-Forward Merge 라고 한다.
- Fast-Forward Merge가 가능한 상태에서 git merge 명령에 --no-ff 옵션을 주면 강제로 Merge Commit을 생성하게 할 수 있다.
  - --no-ff 옵션 예시 : feature 브랜치에 존재하는 커밋 이력을 모두 합쳐서 하나의 새로운 커밋 객체를 만들어 ‘develop’ 브랜치로 병합(merge)하는 것

### Merge (Recursive)
- my-branch 가 main 브랜치에서 분기되고, main 브랜치에 새로운 커밋이 생겼다면, my-branch 를 최신이라고 간주할 수 없다. 따라서 my-branch 와 main 을 공통 부모로 한 새로운 Merge Commit 을 생성하게된다. 이런 방법을 Recursive Merge라고 한다.

## Squash & Merge
### WHAT)
- Squash : 여러개의 커밋을 하나의 커밋으로 합치는 것
- Squash Merge : 병합할 브랜치의 모든 커밋을 하나의 커밋으로 Squash한 새로운 커밋을 Base 브랜치에 추가하는 방식으로 병합하는 것
- Squash를 하게 되면 모든 커밋 이력이 하나의 커밋으로 합쳐지며 사라진다는 점을 주의해야한다.

### WHEN) feature → develop 머지
- feature 브랜치에서 기능을 개발하기 위한 지저분한 커밋 내역을 하나의 커밋으로 묶어 develop 에 병합하면서, develop에는 기능 단위로 커밋이 추가되도록 정리할 수 있다.

### HOW) Squash & Merge가 아닌 일반 Merge를 택했을 때
- 지저분한 커밋 내역이 남아있게되어 불편하다. ~~(내 커밋은 소중하다)~~
#### Revert 후에 gitHub에 Push된 commit 삭제하기
1. git log를 통해 삭제할 commit 찾기
2. git reset을 통해 commit 삭제하기
    - 최근의 commit을 삭제하고 싶을 때 = git reset HEAD^
3. github에 commit 삭제 알리기 = git push -f origin "branch name"

## reset vs revert
### reset
- 커밋 내역들을 삭제하고, 특정 시점의 커밋으로 되돌아가는 명령어
- soft
  - 이전 커밋으로 HEAD를 이동시키지만, 작업 트리와 인덱스는 이전 커밋 상태를 유지한 채로 현재 커밋 상태로 남는다. 
  - 이후에는 변경 사항을 다시 커밋할 수 있다.
- mixed
  - 이전 커밋으로 HEAD를 이동시키면서, 작업 트리는 이전 커밋 상태로 되돌리지만, 인덱스는 초기화된다.
  - 이후에는 변경 사항을 다시 스테이징하고 커밋할 수 있다.
- hard
  - 이전 커밋으로 HEAD를 이동시키면서, 작업 트리와 인덱스를 모두 이전 커밋 상태로 되돌린다.
  - 변경 사항을 완전히 삭제하여 복구할 수 없기 때문에 주의해야한다.
- 세 가지 모드로 변경 사항을 관리할 수 있다.

### revert
- 특정 커밋의 변경 사항을 취소하는 명령어
- 새로운 커밋이 생성되고, 그 커밋에서는 지정한 커밋의 변경 사항이 반대로 적용된다.

## Rebase & Merge

<img src="https://velog.velcdn.com/images/dev-baik/post/880c66bc-94fd-4443-8631-fc76ceeda547/image.png"/>

### WHAT)
- my-branch 가 main 브랜치의 A 커밋에서 분기되었을 때, my-branch의 Base는 A 커밋이다.
- Rebase : Base를 다시 설정한다는 의미로, my-branch가 분기된 Main 브랜치의 최신 커밋으로 다시 설정한다.
- Rebase를 하면 커밋들의 Base가 변경되므로 Commit Hash 또한 변경 될 수 있다. 이로 인해 Force Push를 해야할 경우도 있으니 주의해야한다.

### WHEN) develop → main 머지
- main 브랜치는 지금까지 작업한 모든 기능을 배포할 때 병합한다.
- develop 브랜치를 squash & merge 하게 되면 커밋 이력이 모두 사라져, 특정 기능에서 문제가 생겼을 때 롤백할 수 없게된다.
- main 브랜치 또한 Merge Commit 을 남길 필요 없다. 따라서 Rebase & Merge 가 적합하다.

## merge vs rebase
### merge
- 두 브랜치의 커밋을 하나의 브랜치로 합치는 작업으로, 두 브랜치의 히스토리가 모두 보존되며, "머지 커밋"이라는 새로운 커밋이 생성된다.
- 충돌이 발생하면, 한번에 해결할 수 있다.

### rebase
- 한 브랜치의 변경 사항을 다른 브랜치의 끝에 차례대로 적용하는 과정이다
- 충돌이 발생하면, 충돌이 일어난 부분부터 하나씩 해결해야 한다.
- 기존의 커밋들을 새로운 기반 위에 다시 배치하는 것이므로, 커밋의 해시 값이 변경된다.

- - -

#### 참고한 사이트
- https://gmlwjd9405.github.io/2018/05/11/types-of-git-branch.html
- https://gmlwjd9405.github.io/2018/05/12/how-to-collaborate-on-GitHub-3.html
- https://hudi.blog/git-merge-squash-rebase/
- https://yebeen-study-note.tistory.com/15
- https://han-joon-hyeok.github.io/posts/git-reset-revert/