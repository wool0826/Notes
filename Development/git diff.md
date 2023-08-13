## Git Diff algorithms

https://luppeng.wordpress.com/2020/10/10/when-to-use-each-of-the-git-diff-algorithms/
https://velog.io/@jshme/diff-algorithm-deep-dive-1

- myers
- minimal
- patience
- histogram

### myers

> https://blog.jcoglan.com/2017/02/12/the-myers-diff-algorithm-part-1/

![image.jpeg](https://velog.velcdn.com/images/jshme/post/833cfebe-6b7c-424e-a693-f9e8ab5b4a94/image.png)
*https://velog.io/@jshme/diff-algorithm-deep-dive-1*

1. 모든 연산은 (삽입/삭제/유지) 3가지로 구분된다.
2. 삽입/삭제 연산은 비용이 1이고, 유지는 비용이 0이다.
3. 해당 연산을 그래프로 표현하게 되면, 삽입(세로), 삭제(가로), 유지(대각선) 이다.
4. greedy 하게 최소로 변경되는 경로를 찾는다. (local optimum)

### minimal

greedy 한 myers 알고리즘을 기반으로, global optimum 를 구해서 변경사항을 가능한 최소로 보여주는 알고리즘
분석 시간이 myers 보다는 오래 걸리지만, 최소한의 변경사항을 보여주기 때문에 큰 파일에서의 diff 를 보기에 유용하다고 하네요.

### patience

> https://blog.jcoglan.com/2017/09/19/the-patience-diff-algorithm/

Longest Common Subsequences 알고리즘을 활용하여, 최대로 일치하는 부분을 찾아내고 이를 기준으로 문장을 나눈 뒤 일치하지 않는 부분에는 myers 알고리즘을 수행하고 일치하는 부분은 그대로 두는 방식으로 동작합니다.

> The first thing to note is that patience diff is not a diff algorithm in and of itself. What it really is is a method of matching up lines in two versions of a document in order to break them into smaller pieces, before using an actual diff algorithm like Myers on those pieces.

> https://blog.jcoglan.com/2017/09/19/the-patience-diff-algorithm/

### histogram
