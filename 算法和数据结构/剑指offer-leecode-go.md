## 数组

### 1.数组中重复的数字

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

**思路:利用map特性坐访问缓存**

```go
func findRepeatNumber(nums []int) int {
		var num int 
    for i,v := range nums[1:] {
        if nums[i] == v {
            num = v
            break
        }else {
            //未找到
            num = -1
        }
    }
    return num
}
```

### 2.二维数组的查找

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例:

现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。

给定 target = 20，返回 false。

**思路:双指针从先往左在往下**

```go
func findNumberIn2DArray(matrix [][]int, target int) bool {
    n := len(matrix)
    if n == 0 {
        return false
    }
    m := len(matrix[0])
    if  m == 0 {
        return false
    }
    n1, m1 := 0, m - 1 
    for n1 <= n - 1 && m1 >= 0 {
        if matrix[n1][m1] == target {
            return true
        } else if matrix[n1][m1] > target {
            m1 -= 1
        } else {
            n1 += 1
        }
    }
    return false
}
```

### 3.替换空格

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

**示例 1：**

```
输入：s = "We are happy."
输出："We%20are%20happy."
```

**思路:先对数组进行扩容 len*3，通过ASCII编码进行直接替换**

```go
func replaceSpace(s string) string {
	b := []rune{}
	for _, v := range s {
		if v== 32 {
			b = append(b, 37, 50, 48)
		}else{
			b = append(b, v)
		}
	}
	return string(b)
}
```

### 4.旋转数组中的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

示例 1：

```
输入：[3,4,5,1,2]
输出：1
```

示例 2：

```
输入：[2,2,2,0,1]
输出：0
```

**思路:1.暴力遍历 2.二分求最小值+双指针**

```go
func minArray(numbers []int) int {
    maxNum := numbers[0]

    for i := 1; i < len(numbers); i++ {
        if numbers[i] < maxNum {
            return numbers[i]
        }
    }
    return maxNum
}
```

```go
func minArray(numbers []int) int {
        // 求中位点
    j := len(numbers) - 1
    if j == 0 {
        return numbers[j]
    }
    m := int(j / 2)
    // 判断中位点和尾点的关系 
    // 中位点在左数组中 最小值在[m+1, j]
    if numbers[m] > numbers[j] {
        return minArray(numbers[m+1:])
    // 最小点在左数组中
    } else if numbers[m] < numbers[j] {
        return minArray(numbers[:m+1])
    // 相等时无法判断 缩小边界
    } else {
        return minArray(numbers[:j])
    }
}
```

### 5.合并两个有序数组

```go
func merge(nums1 []int, m int, nums2 []int, n int)  { 
    for m > 0 && n > 0 {
        if nums1[m - 1] > nums2[n - 1] {
            nums1[m+n-1] = nums1[m-1]
            m--
        } else {
            nums1[m+n-1] = nums2[n-1]
            n--
        }
    }
    if m == 0 {
        for i := 0; i < n; i++ {
            nums1[i] = nums2[i]
        }
    }
}
```

## 链表

### 1.从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

**思路:递归栈**

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reversePrint(head *ListNode) []int {
		if head == nil {
        return []int{}
    }
    if head.Next != nil{
        list := reversePrint(head.Next)
        list = append(list, head.Val)
        return list
    }
    return []int{head.Val}
}
```

### 2.删除链表节点

```
func deleteNode(head *ListNode, val int) *ListNode {
    // 假头 优化head的写法判断
    dummy := &ListNode{
        Val:  0, 
        Next: head,
    }
    first := dummy
    second := dummy.Next
    for second != nil {
        if second.Val == val {
            first.Next = second.Next
            second.Next = nil
            break
        }
        first = first.Next
        second = second.Next
    }
    return dummy.Next
}
```

### 3.寻找公共链表节点

```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    // 解法1:暴力N2双循环
    // 解法2:借助Map
    // 相遇
    i := 1
    aPoint, bPoint := headA, headB
    // 防止死循环
    for i < 4 {
       if aPoint == bPoint {
           return aPoint
       }
       if aPoint.Next == nil {
           i++
           aPoint = headB
       } else {
           aPoint = aPoint.Next
       }
       if bPoint.Next == nil {
           i++
           bPoint = headA
       } else {
           bPoint = bPoint.Next
       }
    }
    return nil
}
```

### 4.环路检测

给定一个链表，如果它是有环链表，实现一个算法返回环路的开头节点。
有环链表的定义：在链表中某个节点的next元素指向在它前面出现过的节点，则表明该链表存在环路。
链接：https://leetcode-cn.com/problems/linked-list-cycle-lcci

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */

func detectCycle(head *ListNode) *ListNode {
    fast,slow := head,head
	if fast == nil {
		return nil
	}
    // 先找出第一个相遇点，快指针是满指针的2倍速度
	for fast != nil {
		if fast.Next == nil{
			return nil
		}
		
		fast = fast.Next.Next
		slow = slow.Next
		
		if fast == slow{
			//说明走到一块了
			break
		}
	}
	
	if fast == nil{
		return nil
	}
  fast = head	
	for slow != fast {
		fast=fast.Next
        slow=slow.Next
	}
	
	return slow
}
```

### 5.合并两个有序链表(类似合并数组)

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	tmp := &ListNode{}
	res := tmp
	for l1 != nil && l2 != nil {
		if l1.Val < l2.Val {
			tmp.Next = l1
			l1 = l1.Next
		} else {
			tmp.Next = l2
			l2 = l2.Next
		}
		tmp = tmp.Next
	}
	if l1 == nil {
		tmp.Next = l2
	} else {
		tmp.Next = l1
	}
	return res.Next
}
```

## 二叉树

### 1.重建二叉树

例如，给出

前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
返回如下的二叉树：

    		3
       / \
      9  20
        /  \
       15   7
**思路: 根据两种遍历的规律，可以确认每一颗子树的根节点**

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func buildTree(preorder []int, inorder []int) *TreeNode {
    // 前序遍历 先自己->左->右
    // 中序遍历 先左->自己->右
    for k := range inorder {
        if inorder[k] == preorder[0] { // preorder 是根节点
            return &TreeNode{
                Val: preorder[0],
                Left: buildTree(preorder[1:k + 1], inorder[0:k]),
                Right: buildTree(preorder[k+1:], inorder[k+1:]),
            }
        }
    }
    return nil
}
```

### 2.二叉树层级遍历

```go
func levelOrderBottom(root *TreeNode) [][]int {
    levelOrder := [][]int{}
    if root == nil {
        return levelOrder
    }
    queue := []*TreeNode{}
    queue = append(queue, root)
    for len(queue) > 0 {
        level := []int{}
        size := len(queue)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        levelOrder = append(levelOrder, level)
    }
    for i := 0; i < len(levelOrder) / 2; i++ {
        levelOrder[i], levelOrder[len(levelOrder) - 1 - i] = levelOrder[len(levelOrder) - 1 - i], levelOrder[i]
    }
    return levelOrder
}
```



## 队列

### 1.用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

示例 1：

输入：

```
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]

```

示例 2：

输入：

```
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]
```

**思路: 1.实现栈2.A栈存储 B栈辅助 3.数据插入时存储栈必须为空 数据先压入辅助栈 再全部弹出至存储栈**

```go
/**
 * Your CQueue object will be instantiated and called as such:
 * obj := Constructor();
 * obj.AppendTail(value);
 * param_2 := obj.DeleteHead();
 */
type CQueue struct {
    DataStack Stack
    AuxStack Stack
}

type Stack struct {
    Data []int
}

// 出栈
func (s *Stack) OutStack() int {
    var data int
    if len(s.Data) != 0 {
        data = s.Data[0]
        s.Data = s.Data[1:]
    } else {
        data = -1
    }
    return data
}

// 压栈
func (s *Stack) InStack(val int) {
    s.Data = append(s.Data, val)
}

func Constructor() CQueue {
    return CQueue{
    }
}

// 数据插入时存储栈必须为空 数据先压入辅助栈 再全部弹出至存储栈
func (this *CQueue) AppendTail(value int)  {
    if len(this.DataStack.Data) == 0 {
        this.DataStack.InStack(value)
    } else {
        for len(this.DataStack.Data) != 0 {
            this.AuxStack.InStack(this.DataStack.OutStack())  
        }
        this.AuxStack.InStack(value)

        for len(this.AuxStack.Data) != 0 {
            this.DataStack.InStack(this.AuxStack.OutStack())
        }
    }
}

func (this *CQueue) DeleteHead() int {
    data := this.DataStack.OutStack()
    return data
}
```

## 动态规划

### 1.斐波那契数列

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```


斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**思路:利用动态规划的观念动态推导。递归容易栈溢出**

```go
func fib(n int) int {
    if n == 0 {
        return 0
    }
    if n == 1 {
        return 1
    }

    cache := make([]int, n + 1)
    cache[0] = 0
    cache[1] = 1

    for i := 2; i <= n; i ++ {
        cache[i] = (cache[i - 1] + cache[i - 2]) % 1000000007
    } 
    return cache[n]
}
```

### 2.青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 `n` 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1

**示例 1：**

```
输入：n = 2
输出：2
```

**示例 2：**

```
输入：n = 7
输出：21
```

**提示：**

- `0 <= n <= 100`

**思路:本质上还是斐波那契数列 1.递归 方程f(n) = f(n-1) + f(n-2) 2.动态规划 同斐波那契数列**

```go
func numWays(n int) int {
    if n == 1 || n == 0{
        return 1
    } 
    if n == 2 {
        return 2
    }
    return (numWays(n - 1) + numWays(n - 2)) % 1000000007
}
```

```go
func numWays(n int) int {
    if n == 0 || n == 1 {
        return 1       
    }
    dp := make([]int, n+1)
    dp[0],dp[1] = 1, 1
    for i := 2; i <= n; i++ {
        dp[i] = (dp[i - 1] + dp[i - 2]) % 1000000007
    }  
    return dp[n]
}
```

### 3.最长公共子序列 TODO

## 图

### 1.矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

```
[
  ["a","b","c","e"],
  ["s","f","c","s"],
  ["a","d","e","e"]
]
```

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

示例 1：

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

示例 2：

```
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
```

**思路:DFS**

```go
// 这中leecode通过不了不知道是不是全局变量的原因
var m, n int
var found = false

func exist(board [][]byte, word string) bool {
	m = len(board)
	n = len(board[0])

	visitedNode := make(map[int]map[int]int) // 一维 二维
	for k, _ := range board {
		visitedNode[k] = make(map[int]int)
	}
	// 起点不固定的DFS
	for k, v := range board {
		for k1, v1 := range v {
			if v1 == []byte(word)[0] {
				var path []byte
				path = append(path, v1)
				visitedNode[k][k1] = 1
				DfsSearch(board, path, visitedNode, k, k1, []byte(word))
				if found {
					return true
				}
			}
		}
	}

	return false
}
// DfsSearch 深度优先搜索
func DfsSearch(board [][]byte, path []byte, visitedNode map[int]map[int]int, i, j int, word []byte)  {
	//fmt.Println(string(path), string(word))
	if string(path) == string(word) {
		found = true
		return
	}

	// 上
	if i > 0 {
		tempi := i - 1
		if !CheckVisited(visitedNode, tempi, j) {
			if board[tempi][j] == word[len(path)] {
				path = append(path, board[tempi][j])
				visitedNode[tempi][j] = 1
				DfsSearch(board, path, visitedNode, tempi, j, word)
				path = path[:len(path) - 1]
				delete(visitedNode[tempi], j)
			}
		}
	}
	// 下
	if i < m - 1 {
		tempi := i + 1
		if !CheckVisited(visitedNode, tempi, j) {
			if board[tempi][j] == word[len(path)] {
				path = append(path, board[tempi][j])
				visitedNode[tempi][j] = 1
				DfsSearch(board, path, visitedNode, tempi, j, word)
				path = path[:len(path) - 1]
				delete(visitedNode[tempi], j)
			}
		}
	}
	// 左
	if j > 0 {
		tempj := j - 1
		if !CheckVisited(visitedNode, i, tempj) {
			if board[i][tempj] == word[len(path)] {
				path = append(path, board[i][tempj])
				visitedNode[i][tempj] = 1
				DfsSearch(board, path, visitedNode, i, tempj, word)
				path = path[:len(path) - 1]
				delete(visitedNode[i], tempj)
			}
		}
	}
	// 右
	if j < n - 1 {
		tempj := j + 1
		if !CheckVisited(visitedNode, i, tempj) {
			if board[i][tempj] == word[len(path)] {
				path = append(path, board[i][tempj])
				visitedNode[i][tempj] = 1
				DfsSearch(board, path, visitedNode, i, tempj, word)
				path = path[:len(path) - 1]
				delete(visitedNode[i], tempj)
			}
		}
	}
}

func CheckVisited(visitedNode map[int]map[int]int, i, j int) bool {
	if _, ok := visitedNode[i][j]; ok {
		return true
	}
	return false
}
```

```go
//时间O(MN)，M，N 为rows，cols
//空间O(K) k=len(word)
func exist(board [][]byte, word string) bool {
	rows, cols := len(board), len(board[0])
	for i := 0; i < rows; i++ {
		for j := 0; j < cols; j++ {
			if dfs(board, i, j, 0, word) {
				return true
			}
		}
	}
	return false
}

func dfs(board [][]byte, i, j, k int, word string) bool {
	//已经完全匹配到该word
	if k == len(word) {
		return true
	}

	//i,j边界
	if i < 0 || j < 0 || i >= len(board) || j >= len(board[0]) {
		return false
	}

	if board[i][j] == word[k] {
		temp := board[i][j]
		//临时占位修改board[i][j]，避免递归中使用到重复格子，因为题目要求每个格子只能使用一次
		//如果board[i][j]的上下左右都找不到符合条件的值，进行还原
		board[i][j] = '*'
		//对board[i][j]上下左右的元素进行匹配
		if dfs(board, i-1, j, k+1, word) ||
			dfs(board, i+1, j, k+1, word) ||
			dfs(board, i, j-1, k+1, word) ||
			dfs(board, i, j+1, k+1, word) {
			return true
		} else {
			//board[i][j]上下左右的元素都没有匹配，还原
			board[i][j] = temp
		}
	}
	return false
}
```

### 2.机器人的运动范围

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？ 

示例 1：

```
输入：m = 2, n = 3, k = 1
输出：3
```

示例 2：

```
输入：m = 3, n = 1, k = 0
输出：1
```

**思路:通过DFS和BFS找出所有合法点**

```go
func movingCount(m int, n int, k int) int {
	// 生成一个二维数组（golang的二维数组是真麻烦）
	arr := make([][]int, m)
	for i := 0; i < m; i++ {
		arr[i] = make([]int, n)
	}

	// 遍历全图，找出能够走的点 置为1
	for i:=0;i<len(arr);i++{
		for j:=0;j<len(arr[0]);j++ {
			if getSum(i)+getSum(j)<=k{
				arr[i][j] = 1
			}
		}
	}
	// 使用bfs进行广度优先遍历，找出可走路径，统计路径上点的个数
	queue := list.New()
	queue.PushBack(&Point{0,0})
    
	count :=1
	for queue.Len()!=0{
		item := queue.Front()
		queue.Remove(queue.Front())

		x := item.Value.(*Point).x
		y := item.Value.(*Point).y
		arr[x][y] = 2

		// 下
		if x+1<len(arr) && arr[x+1][y] == 1{
			queue.PushBack(&Point{x+1,y})
			arr[x+1][y]=2
			count++
		}
		// 右
		if y+1<len(arr[0]) && arr[x][y+1] == 1{
			queue.PushBack(&Point{x,y+1})
			arr[x][y+1] =2
			count++
		}
	}

	return count
}

//坐标点
type Point struct {
	x int
	y int
}

// 获得数位之和
func getSum(num int) int {
	sum :=0
	for(num!=0){
		sum+=num%10
		num= num/10
	}
	return sum
}
```

