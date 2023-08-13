## Git Diff algorithms

https://luppeng.wordpress.com/2020/10/10/when-to-use-each-of-the-git-diff-algorithms/
https://velog.io/@jshme/diff-algorithm-deep-dive-1

- myers
- minimal
- patience
- histogram

### myers

> https://blog.jcoglan.com/2017/02/12/the-myers-diff-algorithm-part-1/

a = ABCABBA
b = CBABAC

- A
- B
  C
+ B
  A
  B
- B
  A
+ C

![image.jpeg](https://velog.velcdn.com/images/jshme/post/833cfebe-6b7c-424e-a693-f9e8ab5b4a94/image.png)
*https://velog.io/@jshme/diff-algorithm-deep-dive-1*

1. 모든 연산은 (삽입/삭제/유지) 3가지로 구분된다.
2. 삽입/삭제 연산은 비용이 1이고, 유지는 비용이 0이다.
3. 해당 연산을 그래프로 표현하게 되면, 삽입(세로), 삭제(가로), 유지(대각선) 이다.
4. greedy 하게 최소로 변경되는 경로를 찾는다. (local optimum)

### myers (linear)

> https://blog.jcoglan.com/2017/03/22/myers-diff-in-linear-space-theory/

### minimal

greedy 한 myers 알고리즘을 기반으로, global optimum 를 구해서 변경사항을 가능한 최소로 보여주는 알고리즘
분석 시간이 myers 보다는 오래 걸리지만, 최소한의 변경사항을 보여주기 때문에 변경사항이 큰 파일에서의 diff 를 보기에 유용하다고 하네요.

### patience

> https://blog.jcoglan.com/2017/09/19/the-patience-diff-algorithm/

1. unique line을 기준으로 시퀀스를 분할한다.
2. 분할된 시퀀스에서 각각 1을 수행한다. (재귀적으로)
3. 일치하지 않는 것으로 판단되는 시퀀스에서 myers 알고리즘을 수행하여 diff 를 계산한다.
4. unique line이 없으면 LCS 알고리즘을 활용하여 차이를 계산한다.

#### 예시 1

~~~
                                        1   this
                                        2   is
    1   this                            3   good
    2   is                              4   and
    3   incorrect                       5   correct
    4   and                             6   and

    5   so          <--------------->   7   so

    6   is                              8   is
    7   this                            9   this
~~~

#### 예시 2

~~~
    1   this        <--------------->   1   this
    2   is          <--------------->   2   is

                                        3   good
    3   incorrect                       4   and
    4   and                             5   correct
                                        6   and
--------------------------------------------------------------------
*   5   so                              7   so
--------------------------------------------------------------------
    6   is          <--------------->   8   is
    7   this        <--------------->   9   this
~~~

#### 예시 3

~~~
*   1   this                            1   this
--------------------------------------------------------------------
*   2   is                              2   is
--------------------------------------------------------------------
                                        3   good
    3   incorrect                       4   and
    4   and                             5   correct
                                        6   and
--------------------------------------------------------------------
*   5   so                              7   so
--------------------------------------------------------------------
*   6   is                              8   is
--------------------------------------------------------------------
*   7   this                            9   this
~~~

> The first thing to note is that patience diff is not a diff algorithm in and of itself. What it really is is a method of matching up lines in two versions of a document in order to break them into smaller pieces, before using an actual diff algorithm like Myers on those pieces.

### histogram

> https://stackoverflow.com/questions/32365271/whats-the-difference-between-git-diff-patience-and-git-diff-histogram

patience 에서 좀 더 개선(용도에 따라 다르긴 할 듯 합니다.)한 버전인데요.

patience 에서는 unique line 을 기준으로 분할하였으나, histogram 에서는 해당 line 의 빈도 수를 기록해놓고 시퀀스 A 에서의 빈도보다 시퀀스 B 에서의 빈도가 낮은 경우 분할의 기준이 될 수 있도록 합니다.

중복되는 시퀀스그룹이 모두 기준이 되다보니, 중복되는 문자열을 잘 처리할 수 있는 장점이 있다고 합니다.

