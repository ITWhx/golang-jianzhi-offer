### 	深度优先遍历（DFS）题型

**前言**

* 递归是一种算法结构（在函数中调用函数本身来解决问题），回溯是一种算法思想
* 回溯就是通过不同的尝试来生成问题的解，有点类似于穷举，但是和穷举不同的是回溯会**“剪枝”**，就是对已经知道错误的结果没必要再搜索下去了。

> 比如一个有序数列1,2,3,4,5，我要找和为5的所有集合，从前往后搜索我选了1，然后2，然后选3 的时候发现和已经大于预期，那么4,5肯定也不行，这就是一种对搜索过程的优化

* 回溯法是求问题的解，使用的是DFS（深度优先搜索）。在DFS的过程中发现不是问题的解，
  那么就开始回溯到上一层或者上一个节点 (也就是**“剪枝”**)。
* DFS是遍历整个搜索空间，而不管是否是问题的解
* 回溯搜索是深度优先搜索（DFS）的一种





### 矩阵中的路径

[LeetCode 面试题12. 矩阵中的路径]( https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/ )

**题目描述**

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

**示例**

> 输入：board = [["A","B","C","E"],
>
> ​                           ["S","F","C","S"],
>
> ​                            ["A","D","E","E"]], word = "ABCCED"
> 输出：true



**解题思路**

* 需要从矩阵中每个位置为起点递归搜索一遍

* 搜索过程中，当前位置满足条件，标记走过。接下来，尝试上，下，左，右移动一格

* 当前搜索路径不满足条件，则回溯到上一个位置（**“剪枝”**），搜索其余方向的路径。

  > 不满足条件：
  >
  > 1.路径越界超出矩阵范围
  >
  > 2.当前位置已经访问过
  >
  > 3.当前位置字符和目标字符不匹配



**代码实现**

```
func exist(board [][]byte, word string) bool {
	row := len(board)
	if row == 0 {
		return false
	}
	col := len(board[0])
	if col == 0 {
		return false
	}
	n := len(word)
	if n == 0 {
		return false
	}
	// r,c 当前搜索路径下标
	//idx 当前需要匹配的字符下标
	var dfs func(r, c, idx int) bool
	dfs = func(r, c, idx int) bool {
		// 目标字符串匹配完成
		if idx == n {
			return true
		}
		//当前搜索路径是否满足限制条件，不满足则“剪枝”，回溯到上一步。
		if r < 0 || r >= row || c < 0 || c >= col || board[r][c] != word[idx] {
			return false
		}
		//记录当前字符，以便之后恢复现场
		tmp := board[r][c]
		//当前路径字符字符设为 '0',暂时标记走过路径
		board[r][c] = '0'
		//继续上，下，左，右搜索
		if dfs(r+1, c, idx+1) ||
			dfs(r-1, c, idx+1) ||
			dfs(r, c+1, idx+1) ||
			dfs(r, c-1, idx+1) {
			return true
		}
		// 搜索回溯，撤销走过标记，恢复现场
		board[r][c] = tmp
		return false
	}

	//从矩阵每个位置出发搜索一遍
	for i := 0; i < row; i++ {
		for j := 0; j < col; j++ {
			if dfs(i, j, 0) {
				return true
			}
		}
	}
	return false
}

```

### 机器人运动范围

[LeetCode 面试题13]( https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/ )

**题目描述**

> 地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？
>

**示例**

>
> 输入：m = 2, n = 3, k = 1
> 输出：3


**核心思想**

* 计算机器人所有能走到的格子之和！！

**解题思路**

* 从左上角开始移动，当进入一个格子，通过格子的下标判断是否满足进入条件
* 满足则使用一个二维数组标记走过的格子，并且更新走过格子数。接着向周围其他格子移动

**代码实现**

```go
func movingCount(m int, n int, k int) int {
	var count int
	used := make([][]bool, m)
	//go创建二维数组有点麻烦
	for i := 0; i < m; i++ {
		used[i] = make([]bool,n)
	}

	var dfs func(r, c int)
	dfs = func(r, c int) {
		//判断当前格子是否满足进入条件
		if r >= m || r < 0 || c >= n || c < 0 || used[r][c] || cal(r, c) > k {
			return
		}

		//标记走过的格子
		used[r][c] = true
		count++

		//递归向四周移动
		dfs(r+1, c)
		dfs(r-1, c)
		dfs(r, c+1)
		dfs(r, c-1)
	}

	dfs(0, 0)

	return count
}

func cal(i, j int) int {
	var res int
	if i > 0 {
		res += i % 10
		res += i / 10
	}

	if j > 0 {
		res += j % 10
		res += j / 10
	}
	return res
}

```

**复杂度分析**

* **时间复杂度**：O(MN)
* **空间复杂度**：O(MN)



### 面试题38. 字符串的排列

[leetcode](  https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/  )

**题目描述**

> 输入一个字符串，打印出该字符串中字符的所有排列。

**示例**

> 输入：s = "abc"
> 输出：["abc","acb","bac","bca","cab","cba"]

**解题思路**

* 使用一个数组记录使用过的字符下标
* 每次递归使用一个map记录当前添加字符的下标使用过的字符。

**实现代码**

```go
func permutation(s string) []string {
	n := len(s)
	var res [] string
	//存储排列
	solution := make([]byte, n)
	//记录已经使用过的字符下标
	used := make([]bool, n)

	var helper func(idx int)
	//idx：当前加入排列元素的下标
	helper = func(idx int) {
		//表示一个排列完成，加入结果集
		if idx == n {
			tmp := make([]byte, n)
			copy(tmp, solution)
			res = append(res, string(tmp))
			return
		}
        //记录当前排列下标使用过的字符
		flag := make(map[byte]bool)
		for i := 0; i < n; i++ {
			if used[i] || flag[s[i]] {
				continue
			}
			flag[s[i]], used[i] = true, true
			solution[idx] = s[i]
			helper(idx + 1)
			used[i] = false
		}
	}

	helper(0)
	return res
}

```

**复杂度分析**

* **空间复杂度 O(N^2)**
*  **空间复杂度 O(N^2)**

