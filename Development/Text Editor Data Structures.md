# Text Editor Data Structures

제가 [GeekNews](https://news.hada.io/guidelines) 라는 곳에서 매 주 월요일마다 주요뉴스(?)를 메일로 받고 있는데요.
텍스트 에디터에서 어떤 구조로 데이터를 관리하는지에 대해 나와있어서 재밌어보여 가져왔습니다.

뭔가 한 번도 고민해봤던 적 없는 이슈인데 제목을 보고나서 "어 그렇네?? 어떻게 저장하고 있는 거지?" 라는 생각이 들더라구요

## 출처

https://cdacamar.github.io/data%20structures/algorithms/benchmarking/text%20editors/c++/editor-data-structures/

## In The Beginning

가장 단순한 방법은 "아주 큰 문자열" 로 데이터를 관리하는 것일겁니다.

~~~cpp
// next(iterator, number of elements to advance);
// n 개의 데이터를 순회하는 iterator 를 반환

// begin(), end()
// 해당 buffer 의 begin/end 주소를 반환

// rep()
// 표준 함수로 정의된 건 아닌 것 같은데, repeat는 아닌 것 같고 representation 이지 않나 싶네요.

// string_view
// string 을 복사하지 않고, 다룰 수 있도록 포인터를 제공하는 클래스 (직접 수정은 안 되지만, 원본데이터가 수정되면 string_view 도 같이 수정됨)

void insert_buffer(Editor::Data* data, std::string_view buf) {
    // 1. 기존 크기를 계산. abcdef 라면 6
    auto old_size = data->buf.size();

    // 2. 삽입할 문자열의 크기만큼 재할당. abcdef + zxy = 9
    data->buf.resize(data->buf.size() + buf.size());

    auto insert_at = next(begin(data->buf), rep(data->cursor));
    auto end_range = next(begin(data->buf), old_size);

    // 3. buf 를 end_range(기존 문자열의 끝 부분)에 복사해넣음. O(N)
    // 여기까지의 결과: abcdefzxy
    std::copy(begin(buf), end(buf), end_range);

    // 4. 중간에 삽입하려고 했을 수도 있으므로 문자열을 이동시켜줘야함. O(N)
    // abcdefzxy 를 abczxydef 로 변경.
    std::rotate(insert_at, end_range, end(data->buf));
    data->need_save = true;
}

void remove_range(Editor::Data* data, CharOffset first, CharOffset last) {
    // abcdefxyz 중 def를 삭제하려고 한다.
    auto remove_start = next(begin(data->buf), rep(first));
    auto remove_last = next(begin(data->buf), rep(last));

    // abcxyzdef 로 변경하고 def 의 시작포인터를 갖는다. O(N)
    auto removed_begin = std::rotate(remove_start, remove_last, end(data->buf));

    // d부터 끝까지 지운다. O(N)
    data->buf.erase(removed_begin, end(data->buf));
    data->need_save = true;
}
~~~

해당 방식은 몇가지 장점들이 있는데, 한 마디로 표현하자면 "겁나게 단순하다" 입니다.

1. 문자열을 표현할 수 있는 가장 단순한 방식이다.
2. 삽입/삭제 알고리즘이 단순하다.
3. 단순히 잘 잘라서 보여주기만 하면 되기 때문에, 추가적인 메모리할당 없이 데이터를 보여줄 수 있다.
    - 대략, "에디터에서 데이터를 보여줄 때 단순히 나눠서 보여주기만 하면 된다" 정도로 이해했습니다.

단점은 꽤나 큰데요.

1. 데이터의 사이즈가 커지면 커질수록 연산의 속도가 느려지게 된다.
    - 글에서는 1MB를 기준으로 넘겼을 때를 말하고 있긴 합니다만, 1MB 가 어떤 기준으로 나온 건지는 잘 모르겠네요.
    - 데이터가 변경될 때마다, 해당 데이터를 저장할 메모리를 재할당하고 복사하고 기존에 할당된 메모리를 해제하는 작업을 수행해야합니다.
        - 메모리를 적절하게 크게 만들어두고 재사용하는 방법으로 최적화를 할 수는 있겠지만, 결국 재할당/해제하는 작업이 필요하다는 점은 어쩔 수 없는 것 같네요.
        - O(n) 연산을 O(n^2)으로 바꿀 수 있다고 하는데요. 정확히 어떤 메커니즘에 이해 이렇게 변경되는지..? 잘 모르겠네요? (아신다면 코멘트 부탁드립니다....)
2. undo / redo 를 구현하는 것이 자연스럽지 못하다.
    - 구현자체는 가능하지만, 메모리의 할당과 해제의 오버헤드가 너무 크다.

## Investigation

그래서 저자가 아래 요소들을 만족하는 알고리즘을 찾아보았다고 합니다.

1. 효율적인 insertion/deletion
2. 효율적인 undo/redo
3. UTF-8 인코딩이 가능할 수 있게 유연해야한다.
4. multi-cursor 를 사용할 수 있으면 좋겠다.

알고리즘|삽입/삭제|undo/redo|UTF-8|multi-cursor
:---:|:---:|:---:|:---:|:---:
Gap Buffer|✔|✘|✔|✘
Rope|✔|✘|✔|✔
Piece Table|✔|✔|✔|✔
Piece Tree|✔|✔|✔|✔
Revisiting Deletion|✔|✔|✔|✔

### Gap Buffer

> 사용처: [emacs](https://ko.wikipedia.org/wiki/이맥스)
> https://nullprogram.com/blog/2017/09/07/

알고리즘|삽입/삭제|undo/redo|UTF-8|multi-cursor
:---:|:---:|:---:|:---:|:---:
Gap Buffer|✔|✘|✔|✘

제가 이해했을 때는 `giant string 알고리즘의 개선판` 정도의 느낌인 것 같습니다.
undo/redo 를 하기 위해서는 똑같이 추가적인 메모리 할당/해제의 작업이 필요하고 구조상 multi-cursor를 구현할 수는 있지만 효율적이지는 않습니다.

구현이 아주 간단해서 코어기능은 C로 60줄도 안 된다네요.

// gap buffer gif

간단히 말하면 cursor를 기준으로 (커서 앞의 데이터를 관리하는 버퍼)/(커서 뒤의 데이터를 관리하는 버퍼)로 구분해서 관리하는 알고리즘입니다.

커서가 뒤로 움직이게 되면, 커서 뒤의 버퍼에서 커서 앞의 버퍼로 데이터를 옮기게 됩니다.
커서를 중간에 두고 문자열을 삽입한다고 가정했을 때, 커서 앞의 버퍼에 데이터를 넣어주면 됩니다. (제일 처음 알고리즘 처럼 rotate 가 필요하지 않음)
커서를 중간에 두고 문자열을 삭제할 때는, 커서 앞의 버퍼에서 하나씩 제거하기만 하면 됩니다. (똑같이 rotate 필요없음.)

// multi-cursor gif

multi-cursor 의 경우에는 동작이 되게 비효율적으로 되어있습니다.

~~~cpp
struct gapbuf {
    char *buf;
    size_t total;
    size_t front;
    size_t gap;
}
~~~

### Rope

> https://en.wikipedia.org/wiki/Rope_(data_structure)

알고리즘|삽입/삭제|undo/redo|UTF-8|multi-cursor
:---:|:---:|:---:|:---:|:---:
Rope|✔|✘|✔|✔

// wikipedia image

이진트리로 구성하고, leaf-node 에 문자열을 나눠서 저장하게 됩니다. 해당 노드에는 left sub-tree 의 모든 left-node 의 길이값을 저장합니다. (사진보시면 6 = 6, 9 = 6 + 3, 6 = 2 + 4, 2 = 2 이런 식) 

트리를 사용하다보니, 문자열을 최종적으로 보여주게 될 때는 모든 leaf-node 를 순회해야하고, 리밸런싱도 필요한 것 같습니다.

리밸런싱 작업은 코드 잠깐 보았을 때는 완전이진트리로 만드려고 하는 것 같네요. 
리밸런싱은 O(N) 까지 걸릴 수 있습니다. 모든 성능의 한계가 여기서 발생합니다. (한쪽으로 기울어진 트리일 때 이런 성능이 나오지 않을까 싶네요.)

// 이진탐색 image
특정 위치의 문자를 찾기 위해서는 (index - 노드에 저장된 weight) 가 0보다 큰지, 작은지, 같은지를 비교하여 이분탐색으로 해결합니다.
O(logN) 에 수행할 수 있습니다.

// split

문자열을 붙이고 자르는 작업이 생각보다 복잡하게 생겼는데요.
트리를 우선 잘라내고, 잘라진 트리들 중 합쳐야 되는 트리를 찾아서 합치는 과정이 필요합니다.

O(logN) 에 수행할 수 있습니다. `순회를 통해서 잘라낼 지점을 찾아내고, 트리를 2개로 분리시킨다.` 로 이해했습니다.

// insertion, deletion

insertion 은 간단히 split을 수행하고 앞의 트리와 concat, 결과를 뒤의 트리와 concat 해서 수행합니다.
deletion 은 insertion 과 반대로 split 을 두 번 수행해서 제거할 tree 를 분리해내고 나머지 두 트리를 concat 합니다.
모두 O(logN) 만에 수행할 수 있습니다.

undo, redo 는 구현할 수는 있으나, snapshot 을 따로 저장해야하므로 효율적이지는 못하다고 저자는 생각한 것 같네요.
확실히 결국에는 계속 String 에 대한 값을 복사해서 저장해두어야하므로 공감이 되는 것 같습니다.
그리고 모든 노드를 immutable 하게 관리하므로, 간단한 수정에서도 snapshot을 계속 생성하게 될 것으로 보는 것 같습니다.

### Piece Table

> 사용처: Microsoft word
> https://en.wikipedia.org/wiki/Piece_table

알고리즘|삽입/삭제|undo/redo|UTF-8|multi-cursor
:---:|:---:|:---:|:---:|:---:
Piece Table|✔|✔|✔|✔

Original Buffer / Add Buffer 로 두 벌로 관리한다는 점은 Gap Buffer 랑 비슷한데요.
추가로 Piece Table 이라는 테이블을 통해서 데이터의 조회/삽입/삭제를 수행할 수 있습니다.

Buffer|Content
:---:|:---:
Original|ipsum sit amet
Add|Lorem deletedtext dolor

Which|Start Index|Length|Result (설명을 위해 넣은 컬럼)
:---:|:---:|:---:|:---:
Add|0|6|Lorem 
Original|0|6|ipsum 
Add|17|6|dolor
Original|6|8|sit amet

위 Piece Table 을 기준으로 최종 문자열을 계산하면 "Lorem ipsum dolor sit amet" 이 됩니다.

#### Insert

Add buffer 에 데이터를 추가하고, Piece Table 을 update 합니다.
어느 위치에 넣어야하는지 계산하고 (ADD / index / length) 를 추가하게 되겠네요.

"Lorem ipsum dolor sit amet" 을
"Lorem ipsum dolor ADD sit amet" 으로 변경했다고 가정하면

Buffer|Content
:---:|:---:
Original|ipsum sit amet
Add|Lorem deletedtext dolor ADD

Which|Start Index|Length|Result (설명을 위해 넣은 컬럼)
:---:|:---:|:---:|:---:
Add|0|6|Lorem 
Original|0|6|ipsum 
Add|17|6|dolor
Add|23|3|ADD
Original|6|8|sit amet

요렇게 나올 것으로 보입니다.

#### Delete

- 해당 Entry의 가장 앞이나 끝에서 에서 삭제가 일어났다면, 해당 Entry 만을 수정한다.
    - Start Index 나 Length 를 수정해서 처리할 수 있다.
- Entry 의 중간에서 삭제가 일어났다면, 해당 entry 를 둘로 나눈뒤 해당 삭제가 포함된 Entry를 수정한다.

"Lorem ipsum dolor sit amet" 을
"Lore ipsum dolor sit amet" 으로 변경했다고 가정하면 아래처럼 제일 처음의 Length 를 수정하면 될 것입니다.

Buffer|Content
:---:|:---:
Original|ipsum sit amet
Add|Lorem deletedtext dolor ADD

Which|Start Index|Length|Result (설명을 위해 넣은 컬럼)
:---:|:---:|:---:|:---:
Add|0|**5**|**Lore**
Original|0|6|ipsum 
Add|17|6|dolor
Add|23|3|ADD
Original|6|8|sit amet

만약 "Lore ipsum dolor sit amet" 을
"Lore ipm dolor sit amet" 으로 변경했다고 가정하면 아래처럼 변경될 것입니다.

만약 하나의 Entry 자체가 삭제되었다면 그냥 해당 Entry를 지우는 최적화도 가능하지 않을까 싶네요.

Buffer|Content
:---:|:---:
Original|ipsum sit amet
Add|Lorem deletedtext dolor ADD

Which|Start Index|Length|Result (설명을 위해 넣은 컬럼)
:---:|:---:|:---:|:---:
Add|0|5|Lorem
Original|0|**2**|**ip**
**Original**|**3**|**2(띄어쓰기 포함)**|**m**
Add|17|6|dolor
Add|23|3|ADD
Original|6|8|sit amet


### Piece Tree

> 사용처: VSCode

알고리즘|삽입/삭제|undo/redo|UTF-8|multi-cursor
:---:|:---:|:---:|:---:|:---:
Piece Tree|✔|✔|✔|✔


#### My Own Piece Tree

### Revisiting Deletion

알고리즘|삽입/삭제|undo/redo|UTF-8|multi-cursor
:---:|:---:|:---:|:---:|:---:
Revisiting Deletion|✔|✔|✔|✔

### fredbuf

...