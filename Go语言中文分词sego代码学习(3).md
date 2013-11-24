## Go语言中文分词sego代码学习(3)

上一篇[《Go语言中文分词sego代码学习（2）---前缀树》](http://www.shahuwang.com/2013/08/23/go%E8%AF%AD%E8%A8%80%E4%B8%AD%E6%96%87%E5%88%86%E8%AF%8Dsego%E4%BB%A3%E7%A0%81%E5%AD%A6%E4%B9%A0%EF%BC%882%EF%BC%89-%E5%89%8D%E7%BC%80%E6%A0%91.html) 主要说了前缀树如何构造词典，如何进行查找。为了说明问题，我简化了很多的东西。sego里面的代码则复杂了些，但是知道了原理之后，就能很容易看明白了。

sego关于词典构造和查找的部分，都在dictionary.go这个源代码文件里面。实际上，有前面那篇前缀树的讲解，这个代码已经没有多少内容需要讲的了。它含有其他的一些数据属性，这些都是用来后面分词，或者用于搜索引擎wukong的。我这里大概讲一下它和我之前那篇前缀树在构造字典树的差异吧。

首先，它的前缀树构造和我的略显不同，它的是这样的：  

![前缀树](http://p.blog.csdn.net/images/p_blog_csdn_net/jia_xiaoxin/EntryImages/20080911/%E6%9C%AA%E5%91%BD%E5%90%8D.JPG)

也就是说，它把每个词语都放到最后的那个节点上了。比如in，inn。暂时不太明白这样放置的好处，因为明显过于占用空间了。我之前的那个做法是放标志位，true的话，表示从根节点到这的路线形成一个单词。

它进行单词查找的代码如下：  

        // 在词典中查找和字元组words可以前缀匹配的所有分词
    // 返回值为找到的分词数
    func (dict *Dictionary) lookupTokens(words []Text, tokens []*Token) int {
    	// 特殊情况
    	if len(words) == 0 {
    		return 0
    	}
    
    	current := &dict.root
    	numTokens := 0
    	for _, word := range words {
    		// 如果已经抵达叶子节点则不再继续寻找
    		if len(current.children) == 0 {
    			break
    		}
    
    		// 否则在该节点子节点中进行下个字元的匹配
    		index, found := binarySearch(current.children, word)
    		if !found {
    			break
    		}
    
    		// 匹配成功，则跳入匹配的子节点中
    		current = current.children[index]
    		if current.token != nil {
    			tokens[numTokens] = current.token
    			numTokens++
    		}
    	}
    	return numTokens
    }
 

 这里，tokens是一个指针，所以会修改外面的值。因此实际上，这个函数是返回两个值的，把所有找到符号words这个前缀的单词都返回，以及找到的数量。  