## 구조
<img width="474" alt="스크린샷 2023-07-30 12 53 49" src="https://github.com/wool0826/Notes/assets/19607962/ee1a339f-caa9-474a-8636-b610fcbd9ce1">

### 보면서 생각해 본 코드?

- 커밋 히스토리는 Tree 구조로 저장하되, key 가 데이터를 SHA-1 로 해쉬한 값
- 모든 정보(커밋 정보, 브랜치 정보, 데이터) 는 objects 내에 저장되고 나머지 폴더에서는 해당 데이터를 해쉬값으로 참조하는 구조

~~~kotlin
// key: value 의 hash 값
// 실제 objects 폴더에 저장됨.
val objects: Map<String, Node> = mutableMapOf()

data class Node(
    val hashValue: String, // 이 커밋(또는 데이터)의 해쉬값 5d082144ca4519eee63ef3004fd7f853fa835afc
    val parent: Node?, // 이 커밋의 부모커밋의 해쉬값 0266b24248272e57c9e2da7ab440c1b2b4af3ddc
    val tree: Changes, // 이 커밋의 변경사항이 포함된 objects 의 해쉬값 0dfb53f4815371731d83771edf4c2a63c1dfda3d
    val author: String, // chanwool-jo
    val committer: String, // chanwool-jo
    val commitMessage: String, // 1: commit message
)

data class Changes(
    val changes: List<Change>,
    /*
     * 100644 blob 34211b4b7da82a79a0600b371908e0277661fa4a	a.txt
     * 100644 blob fc156103617770d204b8f92713f5a7554001dca0	b.txt
     */
)

data class Change(
    val permission: String, // 100644
    val type: String, // blob, tree
    val hash: String, // 34211b4b7da82a79a0600b371908e0277661fa4a
    val fileName: String, // a.txt
)
~~~

### objects

커밋된 정보(tree) 또는 데이터(blob)를 objects 폴더 하위에 저장합니다.
저장할 때, 해쉬값을 기준으로 변경이 되었는지를 확인할 수 있습니다.

<img width="724" alt="스크린샷 2023-07-30 14 05 16" src="https://github.com/wool0826/Notes/assets/19607962/1fa9644c-d15f-4feb-8854-5ab5d83eff48">

아래 스크린샷을 보시면, 똑같은 a.txt 인데도 저장된 값이 다른 것을 확인할 수 있습니다.
데이터를 SHA-1 로 해싱하여 데이터가 조금만 변경되어도 파일이 변경되었음을 감지할 수 있습니다.

<img width="300" alt="스크린샷 2023-07-30 13 52 26" src="https://github.com/wool0826/Notes/assets/19607962/2be623d5-8aec-4eec-b235-6249f143f275">

*0266b24248272e57c9e2da7ab440c1b2b4af3ddc의 부모커밋의 트리에는 수정 전 정보가 담겨있다.*

<img width="300" alt="스크린샷 2023-07-30 13 52 32" src="https://github.com/wool0826/Notes/assets/19607962/f47c13e1-bf77-4fe1-9634-ce38030d67c6">

*0266b24248272e57c9e2da7ab440c1b2b4af3ddc의 커밋의 트리에는 수정 후 정보가 담겨있다.*

### index

인덱스에는 현재 staging area 에 반영되어있는 변경사항을 확인할 수 있습니다.

<img width="539" alt="스크린샷 2023-07-30 14 32 30" src="https://github.com/wool0826/Notes/assets/19607962/7837ed9e-f253-439b-948f-4ddcb20aca46">

### refs

refs 폴더에는 브랜치가 어떤 커밋을 head 로 가지고 있는지를 확인할 수 있고, tag가 어떤 커밋을 참조하고 있는지 확인할 수 있습니다.

<img width="343" alt="스크린샷 2023-07-30 13 55 05" src="https://github.com/wool0826/Notes/assets/19607962/293f4b14-fd9a-4ade-ad88-4fe0c4aa8c10">

### HEAD

현재 working directory의 HEAD 값을 확인할 수 있습니다.

~~~shell
> $ cat HEAD                                                                                   ⬡ system [±master ✓]
ref: refs/heads/master

> $ cat HEAD                                                                                ⬡ system [±feature/1 ✓]
ref: refs/heads/feature/1
~~~

### config



### description
### hooks
### info

## 어떻게 변경사항을 추적하는지?

## 정리

제 나름 이해한대로 정리해보면.

1. 커밋정보, 데이터, 브랜치 정보, HEAD, FETCH_HEAD 는 모두 해시값을 저장하는 용도이다.
    - 서로서로 해당 해시값을 통해 참조할 수 있다.
    - ex) HEAD 에 `ref: refs/heads/feature/1` 가 저장되어있으면 ref/heads/feature/1 에 있는 해시값을 참조하겠다는 의미.
2. 동일한 내용의 파일은 가급적 재사용하기 위해 `Map<String, Object> objects` 를 파일시스템에 저장하였다. key는 Object 를 해시처리한 값이다.
3. 커밋 히스토리는 Tree 구조로 구성한다.
    - data class TreeNode(TreeNode? parent, TreeNode currentChanges, `List<Object>?` changes) { }
    - parent 에서는 상위 커밋이 존재한다면, 해당 커밋을 참조한다.
    - currentChanges 에서는 현재 커밋에서 수정된 사항을 저장한 노드를 참조한다.
    - changes 에서는 현재 커밋에서 수정된 사항을 저장한다.
4. index 는 staging 개념을 위해 추가한다. (git add 시에 잠시 저장해두고 git commit 시에 해당 내용을 objects / refs 에 저장한다.)
