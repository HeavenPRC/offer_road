

### 1.数组中重复的数字

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

**思路:利用map特性坐访问缓存**

```go
func findRepeatNumber(nums []int) int {
    numsMap := make(map[int]int)

    for _, v := range nums {
        if _, ok := numsMap[v]; ok {
            return v
        }
        numsMap[v] = 1
    }
    return 0
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

### 4.从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

**思路:**

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reversePrint(head *ListNode) []int {

}
```



来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof
著作权归领扣网络所有。