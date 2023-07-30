## 구조
<img width="474" alt="스크린샷 2023-07-30 12 53 49" src="https://github.com/wool0826/Notes/assets/19607962/ee1a339f-caa9-474a-8636-b610fcbd9ce1">

### objects

특정 커밋(tree) 또는 데이터(blob)을 저장합니다.
저장할 때, 해쉬값을 기준으로 변경이 되었는지를 확인할 수 있습니다.

<img width="467" alt="스크린샷 2023-07-30 13 52 26" src="https://github.com/wool0826/Notes/assets/19607962/2be623d5-8aec-4eec-b235-6249f143f275">
<img width="459" alt="스크린샷 2023-07-30 13 52 32" src="https://github.com/wool0826/Notes/assets/19607962/f47c13e1-bf77-4fe1-9634-ce38030d67c6">

*왼쪽: 0266b24248272e57c9e2da7ab440c1b2b4af3ddc의 부모커밋의 트리에는 수정 전 정보가 담겨있다.*
*오른쪽: 0266b24248272e57c9e2da7ab440c1b2b4af3ddc의 커밋의 트리에는 수정 후 정보가 담겨있다.*

### index
<img width="420" alt="스크린샷 2023-07-30 13 49 22" src="https://github.com/wool0826/Notes/assets/19607962/7225bda7-8587-4365-94d5-f36252fc19dc">


### refs

### HEAD

~~~shell
> $ cat HEAD                                                                                   ⬡ system [±master ✓]
ref: refs/heads/master

> $ cat HEAD                                                                                ⬡ system [±feature/1 ✓]
ref: refs/heads/feature/1
~~~

현재 가리키고 있는 HEAD 의 값을 저장해두는 파일

### config
### description
### hooks
### info

## 어떻게 변경사항을 추적하는지?
