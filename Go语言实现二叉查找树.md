## Go语言实现二叉查找树

什么是二叉查找树？根据维基百科的词条：[二叉查找树](http://zh.wikipedia.org/zh-cn/%E4%BA%8C%E5%85%83%E6%90%9C%E5%B0%8B%E6%A8%B9)，二叉查找树有如下的特征：

+ 根节点的左子树上所有的节点的值都小于根节点的值
+ 根节点的右子树上所有的节点的值都大于根节点的值
+ 每一棵子树本身也是一个二叉查找树，也就是说，左子树上的节点的值都小于该子树根节点的值，右子树上的节点的值都大于该子树根节点的值。

维基百科上的这幅图很好的说明了这种数据结构

![二叉查找树](http://upload.wikimedia.org/wikipedia/commons/thumb/d/da/Binary_search_tree.svg/300px-Binary_search_tree.svg.png)

其实上面的描述还存在一个问题，当新插入的值和某个节点的值相等该怎么办? 直接抛弃掉就可以了。

用Go语言来实现数据结构真的是太爽了，不仅清晰，而且还非常完美。没有了C语言各种落后时代的语法指针问题，也没有Java纯面向对象的繁琐，有指针，有结构体。教科书上大部分的数据结构都是基于C或C++实现的，我这两门语言都不好，要转到Java来实现的话，还有点儿难转。而Go语言则完全没有这个问题，可以模仿C语言的代码过程，又没有C语言的各种麻烦。

还有一个问题，我最讨厌大部分的教程，谈到树这种结构的时候，只谈遍历查找的部分，从来不告诉我们如何构建一个符合要求的树，导致代码无法运行来验证结果。

整个代码最难的地方在于删除操作。由于删除会造成树的结构变化，如果没有按照某种规则进行删除的话，可能就会造成二叉查找树不再是二叉查找树了。

删除有三种情况是要考虑的：

1. 删除的是叶节点，也就是左右子树都为空
2. 删除的节点，可能有左子树或者有右子树（不同时有）
3. 删除的节点，有左子树和右子树

面对这三种情况，就需要有三种考量。首先，情况 1 处理起来很简单，直接删除掉就可以。而情况 2 也比较简单，让该要被删除的节点的父节点，指向该节点的左（右）子树就可以了。情况 3 就异常复杂了。我看了网上很多的资料，最好的当属维基百科的了，但是在这点上讲的还是不好。有兴趣的话，可以看这几篇文章：[二叉查找树](http://zh.wikipedia.org/zh-cn/%E4%BA%8C%E5%85%83%E6%90%9C%E5%B0%8B%E6%A8%B9) ,   [6天通吃树结构—— 第一天 二叉查找树](http://www.cnblogs.com/huangxincheng/archive/2012/07/21/2602375.html) ,    [《算法导论》读书笔记之第12章 二叉查找树](http://www.cnblogs.com/Anker/archive/2013/01/28/2880581.html)  

根据这几篇文章，我最后弄明白针对第三种情况该如何处理了。即找到要删除节点的直接后继，用该后继节点代替要删除节点（就是把值给换成后继节点的值），然后删除掉直接后继节点。

但是，这里就有很模糊的问题了。什么是直接后继节点？这一点去看看线索二叉树，可能就能明白了，中序遍历的线索二叉树，所获得的直接后继节点就是我们要的。按照中序遍历，对要删除节点的右子树进行遍历，如果该子树没有左子树，那么直接后继就是该子树的根节点。如果有左子树，那么直接后继就是最左左子树。譬如下图中，如果我要删除的是 3 这个节点，那么就中序遍历 6 4 7 这个右子树，由于 6 这个根节点有左子树，所以找到其最左左子树，即 4 作为 3 的直接后继。而如果我要删除的是 8 呢？ 由于 10 没有左子树，那么 10 就是 8 的直接后继。把 4 这个节点删除掉，把 3 这个节点的值 替换为 4 。


 ![二叉查找树](http://upload.wikimedia.org/wikipedia/commons/thumb/d/da/Binary_search_tree.svg/300px-Binary_search_tree.svg.png)

删除前，中序遍历的结果是：8, 3, 1, 6, 4, 7, 10, 14, 13  
删除后，中序遍历的结果是：8, 4, 1, 6, 7, 10, 14, 13  
由于我画图不好，就不画出移位后的图了。  
移位的过程可参照这幅图

![二叉查找树](http://pic002.cnblogs.com/images/2012/214741/2012072114312025.png)

下面是我实现的代码，总共四个方法，Add构造二叉查找树，Delete 删除某个节点，Search判断是否存在这个值，PreOrder则是对树进行前序遍历，把结果输出，好看看我们构造的树以及删除某个节点之后，是否仍符号二叉查找树的规定。

        // BSTree.go
    package main
    
    import "fmt"
    
    type Node struct {
    	Value int
    	Left  *Node
    	Right *Node
    }
    
    type BSTree struct {
    	Root *Node
    }
    
    func (tree *BSTree) Add(data int) {
    	root := tree.Root
    	if root == nil {
    
    		//这里注意不能用root = &Node{Value:data},这样只是让root指向其他地方，并没有给Root赋值
    		tree.Root = &Node{Value: data}
    
    	} else {
    		current := root
    		parent := root
    		for current != nil {
    
    			value := current.Value
    			if value == data {
    				return
    			}
    			parent = current
    			if value > data {
    
    				current = current.Left
    			} else {
    				current = current.Right
    			}
    		}
    		node := Node{Value: data}
    		if parent.Value > data {
    			parent.Left = &node
    		} else {
    			parent.Right = &node
    		}
    
    	}
    
    }
    
    func (tree *BSTree) Search(data int) bool {
    	if tree.Root == nil {
    		return false
    	}
    	current := tree.Root
    	for current != nil {
    		if current.Value == data {
    			return true
    		} else {
    			if current.Value > data {
    				current = current.Left
    			} else {
    				current = current.Right
    			}
    		}
    	}
    	return false
    }
    
    func (tree *BSTree) Delete(data int) bool {
    	if tree.Root == nil {
    		return false
    	}
    	current := tree.Root
    	parent := tree.Root
    	for current != nil {
    
    		if current.Value == data {
    			//找到了，进行删除处理
    			//第一种情况：该节点是叶节点，直接删除
    			if current.Left == nil && current.Right == nil {
    				parent.Left = nil
    				parent.Right = nil
    				if parent == tree.Root {
    					tree.Root = nil
    				}
    			} else if current.Left != nil && current.Right == nil {
    				//第二种情况，该节点有左子树或者右子树（不同时有）
    				//直接删除节点，该节点的父节点
    				if parent.Value > data {
    					parent.Left = current.Left
    				} else {
    					parent.Right = current.Left
    				}
    				current = nil
    			} else if current.Right != nil && current.Left == nil {
    				if parent.Value > data {
    					parent.Left = current.Right
    				} else {
    					parent.Right = current.Right
    				}
    				current = nil
    			} else {
    				//第三种情况，该节点有左右子树。将该节点与其右子树看成一棵线索二叉树
    				//找到该节点的直接后继，用该后继节点占领该节点，就完成了删除操作
    				//对右子树（不包括删除的那个节点）进行中序遍历找到直接后继节点
    
    				cltree := current.Right
    				clparent := cltree
    				for cltree.Left != nil {
    					clparent = cltree
    					cltree = cltree.Left
    				}
    
    				//用cltree 占领 current的位置
    				if parent.Value > data {
    					parent.Left = cltree
    
    				} else {
    					parent.Right = cltree
    				}
    
    				//很显然，cltree是没有左子树的
    				cltree.Left = current.Left
    				if current.Right != cltree {
    					//说明直接后继不是current的右子节点
    					//此时要把cltree与其父节点的关系切断，否则会造成死循环
    					clparent.Left = nil
    					cltree.Right = current.Right
    				}
    				current = nil
    				return true
    			}
    		} else {
    			parent = current
    			if current.Value > data {
    				current = current.Left
    			} else {
    				current = current.Right
    			}
    		}
    	}
    	return false
    }
    
    func PreOrder(tree *BSTree) []int {
    	values := make([]int, 0, 50)
    	//进行前序遍历，把结果输出，看看二叉查找树构造是否正确
    	current := tree.Root
    
    	if current == nil {
    		return values
    	}
    	leftTree := new(BSTree)
    	leftTree.Root = current.Left
    	rightTree := new(BSTree)
    	rightTree.Root = current.Right
    	values = append(values, current.Value)
    	left := PreOrder(leftTree)
    	right := PreOrder(rightTree)
    	values = append(values, left...)
    	values = append(values, right...)
    	return values
    }
    
    func main() {
    	tree := new(BSTree)
    	tree.Add(8)
    	tree.Add(3)
    	tree.Add(10)
    	tree.Add(1)
    	tree.Add(6)
    	tree.Add(4)
    	tree.Add(7)
    	tree.Add(14)
    	tree.Add(13)
    	values := PreOrder(tree)
    	fmt.Println(values)
    	fmt.Println(tree.Search(1))
    	tree.Delete(3)
    	values = PreOrder(tree)
    	fmt.Println(values)
    
    }
