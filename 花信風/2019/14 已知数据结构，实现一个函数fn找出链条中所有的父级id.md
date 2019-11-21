[pixiv: 014]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/14.jpg'

# 第 92 题：已知数据格式，实现一个函数 fn 找出链条中所有的父级 id
```javascript
const value = '112'
const fn = (value) => {
...
}
fn(value) // 输出 [1， 11， 112]
```
![数据格式](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/141.jpg)

## 目前发现的最优题解，摘录学习之
### dfs
```javascript
const fn = (data, value) => {
  let res = []
  const dfs = (arr, temp = []) => {
    for (const node of arr) {
      if (node.children) {
        dfs(node.children, temp.concat(node.id))
      } else {
        if (node.id === value) {
          res = temp
        }
        return
      }
    }
  }
  dfs(data)
  return res
}
```
作者是github: <b>ZodiacSyndicate</b>

dfs，英文全称Depth First Search，顾名思义，即是深度优先搜索算法。
深度优先搜索属于图算法的一种，是一个针对图和树的遍历算法。深度优先搜索是图论中的经典算法，利用深度优先搜索算法可以产生目标图的相应拓扑排序表，利用拓扑排序表可以方便的解决很多相关的图论问题，如最大路径问题等等。一般用栈数据结构来辅助实现DFS算法。其过程简要来说是对每一个可能的分支路径深入到不能再深入为止，而且每个节点只能访问一次。

相对应的有，bfs，英文全称breadth first search，广度优先搜索（也称宽度优先搜索，缩写BF）是连通图的一种遍历算法这一算法也是很多重要的图的算法的原型。Dijkstra单源最短路径算法和Prim最小生成树算法都采用了和宽度优先搜索类似的思想。其别名又叫BFS，属于一种盲目搜寻法，目的是系统地展开并检查图中的所有节点，以找寻结果。换句话说，它并不考虑结果的可能位置，彻底地搜索整张图，直到找到结果为止。基本过程，BFS是从根节点开始，沿着树(图)的宽度遍历树(图)的节点。如果所有节点均被访问，则算法中止。一般用队列数据结构来辅助实现BFS算法。


# 题目来源
- [木易杨每天一道大厂前端面试题 issue143](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/143)
- [基本算法——深度优先搜索（DFS）和广度优先搜索（BFS）](https://www.jianshu.com/p/bff70b786bb6)