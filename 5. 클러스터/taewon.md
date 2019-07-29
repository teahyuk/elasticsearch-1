# 9.엘라스틱서치와 루신 이야기
## 9.3.4 Flush, Commit, Merge
- 루씬 내부의 메모리에 전달된 데이터가 쌓임
- 일정 주기에 한번씩 세그먼트를 생성하고 디스크에 동기화 하는 작업이 Flush
- 실제로 데이터를 디스크에 쓰는 작업이 Commit
- 다수의 세그먼트를 하나로 합치는 작업이 Merge

[![Video Label](http://blog.mikemccandless.com/2011/02/visualizing-lucenes-segment-merges.html)](https://youtu.be/YW0bOvLp72E)


    