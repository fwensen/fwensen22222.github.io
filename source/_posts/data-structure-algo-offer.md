---
title: 剑指offer总结
date: 2017-08-07 00:24:33
tags: Algorithm
categories: Data structure and algorithm
---

剑指offer总结

#### 1. 在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

> 这个是典型的二分查找，直接使用传统的二分查找即可

```Java
public boolean Find(int target, int [][] array) {
		
		if (array == null || array[0].length == 0) return false;
		final int row = array.length;
		final int col = array[0].length;
		for (int i = 0; i < row; i++) {
			
			int lo = 0, hi = col-1;
			while (lo <= hi) {
				int mid = (lo + hi)/2;
				if (array[i][mid] == target) {
					return true;
				} else if (array[i][mid] < target) {
					lo = mid + 1;
				} else {
					hi = mid - 1;
				}
			}
		}
		return false;
	}
```

#### 2. 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

> 这也是一个典型二分查找问题。

```Java
public int minNumberInRotateArray(int [] array) {
		
		if (array == null || array.length == 0) return 0;
		int left = 0, right = array.length - 1;
		int mid = left;
		while (array[left] >= array[right]) {
			//找到结束时
			if  (right - left == 1) {
				mid = right;
				break;
			}
			
			mid = (left + right)/2;
			//预防序列1 0 1 1 1这种情况
			if (array[left] == array[right] && array[mid] == array[left]) {
				int ret = array[left];
				for (int i = left+1; i <= right; i++) ret = Math.max(ret, array[i]);
				return ret;
			}
			if (array[mid] >= array[left]) {
				left = mid;
			} else if (array[mid] <= array[right]) {
				right = mid;
			}
		}
		return array[mid];
}
```

#### 3. 输入一个链表，从尾到头打印链表每个节点的值。

> 这个问题的最好解决方式是使用递归，有点类似于二叉树后续遍历方式。

```Java
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {

		ArrayList<Integer> result = new ArrayList<Integer>();
		if (listNode != null) {
			printListFromTailToHeadCore(listNode, result);
		};
		return result;
    }
	
	private void printListFromTailToHeadCore(ListNode listNode, ArrayList<Integer> list) {
		if (listNode != null) {
			printListFromTailToHeadCore(listNode.next, list);
			list.add(listNode.val);
	}
}
```

#### 4. 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

> 这个问题属于二叉树操作，但这里也可以把它归类于链表，同样，解决这个问题较好的方式是使用递归。同时我们需要利用到二叉树的特性：

- 二叉树的先序遍历是先遍历父节点，然后遍历左子节点，最后遍历右子节点；  
- 中序遍历是先左子节点，然后父节点，最后右子节点。

> 所以利用这个特性，我们知道先序遍历的第一个点是整个二叉树的根节点，然后中序遍历以这个节点为中心，分成了左子树和右子树，举例来说，前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，那么以1为根节点，左子树为{4,7,2}，右子树为{5,3,8,6}，然后接下来左子树以2为根节点，分成了左子树{4,7}，右子树{}，同样，以3为根节点，将树分成了左子树{5}，右子树{8,6}，整个过程是是个递归过程，所以使用递归较为合适。

```Java
public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
		
		if (pre.length == 0 || in.length == 0) return null;
		TreeNode root = new TreeNode(pre[0]);
		for (int i = 0; i < in.length; i++) {
			//找到切分点时
			if (pre[0] == in[i]) {
				root.left = reConstructBinaryTree(Arrays.copyOfRange(pre, 1, i+1), 
						Arrays.copyOfRange(in, 0, i));
				root.right = reConstructBinaryTree(Arrays.copyOfRange(pre, i+1, pre.length), 
						Arrays.copyOfRange(in, i+1, in.length));
			}
		}
		return root;
}
```

	- 这里解释下Arrays.copyOfRange(int[] original, int from, int to)函数，这个函数复制original数组下标[from, to)部分，即from包含在范围内，而to不包含；  
	- pre[0] == in[i]语句找到中序遍历的根节点，然后将数组分成了左子树和右子树两部分，假设左子树节点数目为m，右子树节点数目为n，则这时前序遍历序列除第一个树外，接下来的n个节点在左子树中，再接下来的n个节点在右子树中，所以才有了递归的过程。  
	- 当然这个问题也可以不用递归思想，不过复杂度和代码量会大大增加。

##### 补充知识点  

- 非空二叉树的第n层上至多有2^(n-1)个元素；    
- 深度为h的二叉树至多有2^h-1个结点；  
- 在满二叉树中若其深度为h，则其所包含的结点数必为2^h-1。
- 对于完全二叉树，设一个结点为i则其父节点为i/2，则2i为左子节点，2i+1为右子节点；  
- 在完全二叉树中，具有n个节点的完全二叉树的深度为[log2]+1，其中[log2]+1是向下取整，log底取为2；  
- n0=n2+1  n0表示度数为0的节点 n2表示度数为2的节点；    
- 前序遍历：根节点->左子树->右子树；    
- 中序遍历：左子树->根节点->右子树；  
- 后序遍历：左子树->右子树->根节点。

#### 5. 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

> 栈有FILO性质，队列有FIFO性质；直接上代码

```Java
Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push(node);
    }

    public int pop() {
        
        if (stack2.isEmpty()) {
        
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }
        int ret = stack2.pop();
        return ret;
    }
```

#### 6. 我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

> 这个问题是典型的斐波拉契数列题，一般使用动态规划的方式解决，想象2X8的矩形，设为f(8)，最左边有两种放置方式： 

- 最左边竖放时，右边还剩下2X7，这时可记为f(7)；  
- 最左边横放时，这时可放置两块，右边剩下2X6，记为f(6);

所以f(n) = f(n-1) + f(n-2)，一个典型的斐波拉契数列，然后使用动规思想即可。

```
public int RectCover(int target) {
		
        if (target <= 2) return target;
        int n1 = 1, n2 = 2;
        int m = target - 2;
        while (m-- > 0) {
            n2 = n1 + n2;
            n1 = n2 - n1;
        }
        return n2;
    }
```

#### 7. 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分。

> 这个问题类似于问题：以0为界，将数组中所有负数放置到整数后面，都是指针操作，即使用两个指针，一个指针初始化为数组第一个数字下标，它只向后移动，第二个指针初始化为数组最后一个数下标，它只向前移动，在两个指针相遇前，如果第一个指针指向的数是奇数，第二个指针指向的数为偶数，则交换这两个数，指针继续移动直到相遇。

```Java
public void reOrderArray(int [] array) {
		 if (array == null || array.length == 0) return;
		 int pos1 = 0, pos2 = array.length - 1;
		 while (pos1 < pos2) {
			 while (pos1 < pos2 && (array[pos1]&1) != 0) pos1++;
			 while (pos1 < pos2 && (array[pos2]&1) == 0) pos2--;
			 //交换
			 if (pos1 < pos2) {
				 int tmp = array[pos1];
				 array[pos1] = array[pos2];
				 array[pos2] = tmp;
			 }
		 }
	 }
```

#### 8. 输入一个链表，输出该链表中倒数第k个结点。

### to be comtinue



