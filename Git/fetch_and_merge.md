![](https://velog.velcdn.com/images/dodo4723/post/dc9dba2d-244f-4482-acb6-c5cd467c4703/image.png)

저는 매일 백준 문제 하나를 풀어 깃허브에 커밋하는 저와의 약속이 있습니다. 그래서 매일 깃허브를 사용하긴 합니다만 커밋 외의 기능들은 두루뭉실하게 알고 있어 머릿속을 정리하기 위해 깃과 관련된 책 한권을 읽었습니다.

![](https://velog.velcdn.com/images/dodo4723/post/0d5a66d8-e863-4852-bf19-60f662340752/image.png)

대부분의 내용은 제가 알고있던 것이 맞았습니다. 하지만, 제가 애매하게 알고있던 부분을 확실하게 정리하고 넘어갈까 합니다.

대표적으로 2가지입니다.

<br>
<br>
<br>

## 1. git fetch

**원격 저장소의 정보를 가져오는 명령어**입니다. 일단 변경된 내용만 가져옵니다.

깃허브를 통해 팀 작업을 할 때 다른 사람이 원격 저장소에 올려놓은 커밋을 무조건 내 지역 저장소에 합치지 않고, 원격 저장소에서 수정한 내용을 가져와서 흝어본 후에 필요할때만 지역 저장소에 합치고 싶을때 사용합니다.

예제를 통해 순서대로 알아보겠습니다

#### 1. git fetch

원격 저장소의 최신 커밋 정보를 가져왔지만 아직 지역 저장소에
합치지 않았습니다.

#### 2. git status
현재 깃의 상태를 확인해보면 현재 브랜치가 `origin/main`에 비해 1개의 커밋이 뒤처져 있다고 나옵니다. `git pull`을 사용하면 지역 저장소를 업데이트할 수 있다고 알려줍니다.

![](https://velog.velcdn.com/images/dodo4723/post/28cab1ee-f844-47c4-9dda-6f8a251e8fae/image.png)


#### 3. git diff
`git diff HEAD origin/main`을 사용해서 현재 최신 커밋과 원격 저장소에서 가져온 커밋의 차이를 살펴볼 수 있습니다.

![](https://velog.velcdn.com/images/dodo4723/post/ffa4fa40-5aff-4a9a-bbf2-c1c67ea37b24/image.png)

#### 4. git merge
원격 저장소의 커밋을 확인하고 지역 저장소에 합치겠다고 결정했다면 `git merge origin/main` 으로 합치면 됩니다.

<br>
<br>
<br>

## 2. 브랜치의 병합

제가 예전부터 계속 궁금했던 내용을 여기서 해결할 수 있었습니다. 2개의 브랜치에서 같은 문서를 수정한다면 어떻게 병합할까요?

일단 예상대로 같은 문서로 서로 다른 위치를 수정했을 경우는 자동으로 잘 합쳐줍니다. 하지만, 같은 부분을 수정하면 어떻게 될까요?

**깃에서는 줄 단위로 변경 여부를 확인**하기 때문에 서로 다른 브랜치에서 같은 문서의 같은 줄을 수정했을 경우, 브랜치를 병합하면 **브랜치 충돌(conflict)**이 발생합니다.

![](https://velog.velcdn.com/images/dodo4723/post/acb5fdfa-ab20-4004-97c4-b02b20682713/image.jpg)


예를들어, 위 문서와

![](https://velog.velcdn.com/images/dodo4723/post/3fbda887-d8b1-43e5-b4cd-19fd65c59cd8/image.jpg)

위 문서를 병합했을 경우,

![](https://velog.velcdn.com/images/dodo4723/post/c2957535-a63c-4bba-a53b-f0fcdf9e50a6/image.jpg)

충돌했다는 경고 메세지가 나타납니다.

![](https://velog.velcdn.com/images/dodo4723/post/e3de410d-ccdd-48dc-aa93-600718026d7c/image.jpg)

문서를 열어보면 위와 같이 <, =, > 가 추가되어있습니다.

결론은 **양쪽 브랜치의 내용을 참고하면서 내용을 직접 수정해야 합니다.**

몇년간의 궁금증이 풀려 시원합니다.