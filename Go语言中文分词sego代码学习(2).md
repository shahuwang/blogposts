## Go语言中文分词sego代码学习（2）---前缀树

按照作者写的文档，sego的词典是以前缀树（英文名为Trie）的方式组织的。说实在的，由于一直没有静下心来学习数据结构，这是我第一次听说前缀树。看了介绍之后，着实惊讶。这个数据结构非常简单，但是又异常高效，进行查找的时候，效率几近Hash表，而存储的时候，更是非常节省空间。

记得当时参加百度的笔试，有一个就是关于百度搜索框的自动提示是如何实现的。当时只知道用哈希表，现在想来，用前缀树倒是非常理想。哈希表在空间占用超过其初始值2/3的时候就会自动把空间增大一倍，非常耗费空间。

现在进入主题，到底前缀树是什么？我觉得大部分的文章说明都过于复杂了，[《Trie树|字典树的简介及实现（转）》](http://www.cppblog.com/abilitytao/archive/2009/04/21/80598.aspx)的解释已经很足够了：
>其基本性质可以归纳为：  
1. 根节点不包含字符，除根节点外每一个节点都只包含一个字符。  
2. 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串。  
3. 每个节点的所有子节点包含的字符都不相同。

如这幅图所示 
   
![前缀树](http://www.cppblog.com/images/cppblog_com/abilitytao/Trie.jpg)

根节点不包含字符。从根到某一个节点，所有的边的字母连接起来，就是一个单词。图上的红点表示从根到这个节点所形成的单词，存在于构造的字典中。

所以，前缀树的字典构造过程就很清晰了。譬如我要加入**hello**这个单词，添加过程如下：
 
+ 从根节点出发，找出第一层子节点。看这一层里面有没有含有h这个字母。如果有，则顺着h的子节点进入下一层。如果没有，则构造一个节点，把字母h插入到当前层中。然后顺着把e插入到h的子节点中，如此类推。

要搜索字典树中是否含有单词hello，则先找根节点的子节点是否有h，如果没有，说明没hello这个单词。如果有，顺着h的子节点，看看是否含有e，如此迭代下去。

另外，前缀树其实就是一个有限状态机。

说了这么多，前缀树大家其实都懂了，但是要写出代码来，还真的是很不容易呀。我照着别人写的比较复杂的写，写了一个比较简单的，都花了四五个小时。悲剧的是，指针用得太多，我也不甚明白了。接下来要好好把指针这一块弄明白才行。

我的代码是用Go语言写的，我觉得Go语言写数据结构，比C，C++，Java都好。它有指针，指针不复杂。它也能面向对象，又有结构体，写出来的数据结构代码相当清晰。

我的这份代码，主要实现了三个函数，Add是把一个单词添加到前缀树里，Search是判断一个单词是否存在于前缀树字典中，BinarySearch则是用于二分查找的。

下面是代码：

    package main
    
    import (
    	"bytes"
    	"fmt"
    )
    
    type node struct {
    	Word []byte  //保存一个单词
    	Children []*node //保存该单词的后继节点
    
    	//表明从根到这个节点所经过的所有字母组合起来是否构成了一个单词
    	//譬如我添加了hello这个单词，查找hel，exist则为false，说明不存在这个单词
    	Exist bool
    }
    
    type TrieTree struct {
    	Root node
    }
    
    func main() {
    	trie := new(TrieTree)
    	trie.Add("hello")
    	fmt.Println(trie.Search("hell"))
    	fmt.Println(trie.Search("hello2"))
    
    }
    
    func (tree *TrieTree) Search(word string) bool {
    	current := &tree.Root
    	chars := []byte(word)
    	for i := 0; i < len(chars); i++ {
    		s := chars[i : i+1]
    		nodes := current.Children
    		index, exist := tree.BinarySearch(nodes, s)
    		if !exist {
    			//fmt.Println(i, s)
    			return false
    		}
    		current = nodes[index]
    	}
    
    	return current.Exist
    }
    func (tree *TrieTree) Add(word string) {
    	//把一个单词增加到当前的前缀树中
    	current := &tree.Root
    	chars := []byte(word)
    	//把单词的每个字母都添加到前缀树中去，（有则忽略掉）
    	for i := 0; i < len(chars); i++ {
    		s := chars[i : i+1]
    		//fmt.Println(s)
    		nodes := &current.Children
    		index, exist := tree.BinarySearch(*nodes, s)
    		//fmt.Println(index, exist)
    		if !exist {
    			//这个字母在前缀树中没有则添加进去
    			//fmt.Println("index:", index)
    			*nodes = append(*nodes, nil)
    
    			copy((*nodes)[index+1:], (*nodes)[index:])
    			(*nodes)[index] = &node{Word: s, Exist: false}
    			//fmt.Println("insert:", (*nodes)[0].Word)
    
    		}
    
    		current.Children = *nodes
    		current = (*nodes)[index]
    
    	}
    	current.Exist = true //单词添加完毕，末位节点设为true表示这个从跟到这个节点的路线所构成的单词存在
    
    }
    
    //二分查找。譬如字母a这个节点是第一个节点，则其子节点，包含了所有以a开头的单词
    //该二分查找，就是找前缀树里，以a开头的单词的第二个字母是否含有s所代表的那个字母
    func (tree *TrieTree) BinarySearch(nodes []*node, s []byte) (int, bool) {
    	start := 0
    	end := len(nodes) - 1
    
    	if end == -1 {
    		return 0, false
    	}
    	compareFirst := bytes.Compare(s, nodes[0].Word)
    
    	if compareFirst < 0 {
    		//说明不存在
    		fmt.Println("compare first:", s, nodes[0].Word)
    		return 0, false
    	} else if compareFirst == 0 {
    		return 0, true
    	}
    	compareLast := bytes.Compare(s, nodes[end].Word)
    	if compareLast > 0 {
    		//说明不存在
    		return end + 1, false
    	} else if compareLast == 0 {
    		return end, true
    	}
    
    	current := end / 2
    
    	//为什么是end-start>1 而不是>0呢
    	if end-start > 1 {
    		compareCurrent := bytes.Compare(s, nodes[current].Word)
    		if compareCurrent > 0 {
    			start = current
    			current = (end + start) / 2
    		} else {
    			end = current
    			current = (end + start) / 2
    		}
    	}
    
    	return end, true
    
    }
    