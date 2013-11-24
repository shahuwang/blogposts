## Go语言中文分词sego代码学习（1）

从大二下开始下定决心当程序员，到现在毕业工作两个月，我一直都不敢称自己是程序员，软件工程师就不敢自称了。很弱很弱，而且我还非常不专注，导致我无一技能足以夸耀，找工作也自然没能进入那些比较好的互联网公司。

然而很奇怪的是，像我这样的人，在大学里依旧是比较专注的人。这就是我在人人上发了N多状态，鞭笞现在大学普遍推崇的通识教育的原因。通识教育是一个非常扯的教育模式，之前我说过：“通识教育喊得越响，大学就业率就越低！”

也许很多人会扒拉扒拉说大学又不是培养技工之类的话，但你知道嘛，通识教育的目标就是培养大师，但你我之中有多少人能成为大师呢？用几千个学生去做实验，就是为了找出那其中的两三个大师，剩下的人，就和药渣一样，自食其果了。而且，有句话不是说，天才从来不是学校培养的，放他在北大还是 放清华，他都是天才。

好了，扯淡结束，开始进入主题。

前几天在微博上看到[sego](https://github.com/huichen/sego)这个用Go语言实现的中文分词包，是谷歌的一个华人工程师写的。好奇去看了一下，作者的代码写得相当清晰，注释也很清晰。一直以来觉得读源码是某个阶段的程序员进阶的最佳途径，譬如我当前这个阶段。但是读源码，高级的程序员可以去读MySQL，Lucene这样大的项目的代码，他们可以读到很多架构知识。但如我这种，最佳的就是去读个人作品的代码，要是有足够的注释，那就更好了。因为个人作品的代码，风格比较一直，而且一般不会很大。

要安装这个包的话，用命令：go get github.com/huichen/sego即可，当然，你要先安装了git在你电脑上才行。

安装好之后，写个Sample程序运行一下，看看效果如何，代码如下：

    package main
    
    import (
    	"fmt"
    	"github.com/huichen/sego"
    )
    
    func main() {
    	// 载入词典
    	var segmenter sego.Segmenter
    	segmenter.LoadDictionary("C:/Go/package/src/github.com/huichen/sego/data/dictionary.txt")
    
    	// 分词
    	
    	text := []byte("中华人民共和国中央人民政府")
    	segments := segmenter.Segment(text)
    
    	// 处理分词结果
    	// 支持普通模式和搜索模式两种分词，见代码中SegmentsToString函数的注释。
    	fmt.Println(sego.SegmentsToString(segments, true))
    }

注意，词典的路径要修改为你安装的路径。

今天我要讲解的代码是sego.Sementer这个代码里的func splitTextToWords 。这是个私有方法，为什么我要先研究它呢？因为我在看sego载入字典的操作的步骤中，遇到这一步相当不明白，然后花了一天才搞明白了。而且，我觉得它作用很大，就是将一长串字切成字元。譬如“你好，hello，中国”，会被切为“你”，“好”，“，”，“hello”，“中”，“国”。

由于是私有方法，不能导出来进行测试，所以我就复制代码，放到外部进行测试了，自己增加了些代码，所有代码如下：

    package main
    
    import (
    	"fmt"
    	"unicode/utf8"
    )
    
    type Text []byte
    
    func main() {
    	test := []byte("wm你好，hello，中国")
    	text := splitTextToWords(test)
    	for i := 0; i < len(text); i++ {
    		t := CToGoString(text[i])
    		fmt.Println(t)
    
    	}
    }
    
    // 将英文词转化为小写
    func toLower(text []byte) []byte {
    	output := make([]byte, len(text))
    	for i, t := range text {
    		if t >= 'A' && t <= 'Z' {
    			output[i] = t - 'A' + 'a'
    		} else {
    			output[i] = t
    		}
    	}
    	return output
    }
    
    //把byte数组转换为对应的字符串
    func CToGoString(c []byte) string {
    	n := -1
    	for i, b := range c {
    		fmt.Println("(i,b)", i, b)
    		if b == 0 {
    			break
    		}
    		n = i
    	}
    	return string(c[:n+1])
    }
    
    func splitTextToWords(text Text) []Text {
    	output := make([]Text, len(text))
    	current := 0
    	currentWord := 0
    	inAlphanumeric := true //默认要切分的字符串都以字母或数字开始，如果第一个是汉字，那么代码会将其改
    	//改为false，并把当前汉字放进数组里
    	alphanumericStart := 0
    	for current < len(text) {
    		//该函数获取到byte数组的第一个utf8字符，并返回该字符的byte宽带
    		//当返回1的时候，表示是ASCII字符.(文档这么写的)
    		//经测试发现，貌似是针对ANSCII符号，只要数组第一个是ANSCII，那么就返回1
    		_, size := utf8.DecodeRune(text[current:])
    
    		if size == 1 &&
    			(text[current] >= 'a' && text[current] <= 'z') ||
    			(text[current] >= 'A' && text[current] <= 'Z') ||
    			(text[current] >= '0' && text[current] <= '9') {
    			// 当前是英文字母或者数字
    			//进入这个部分，则是刚好到hello的h字母，此时inAlphanumeric为false
    			if !inAlphanumeric {
    				//假如第一个就是字母或数字，这里并不运行
    
    				alphanumericStart = current
    				inAlphanumeric = true
    			}
    		} else {
    			if inAlphanumeric {
    				inAlphanumeric = false
    				if current != 0 {
    					//此时遇到第一个汉字
    					//前面两个字母都只运行了current+=size这一句，此时的currentWord=0
    					//alphanumericStart:current即0:2，这样就把前面的两个字母作为一个词
    					//放进数组里了。
    
    					output[currentWord] = toLower(text[alphanumericStart:current])
    					currentWord++
    				}
    			}
    
    			//这里把当前汉字放进数组了，size=3，参考Go的unicode编码问题
    			output[currentWord] = text[current : current+size]
    			//fmt.Println(output[currentWord])
    			//fmt.Println("current word", CToGoString(text[current:current+size]))
    			currentWord++
    		}
    		current += size
    	}
    
    	// 处理最后一个字元是英文的情况
    	if inAlphanumeric {
    		if current != 0 {
    			output[currentWord] = toLower(text[alphanumericStart:current])
    			currentWord++
    		}
    	}
    
    	return output[:currentWord]
    }

    
总共有四个变量的作用是需要说明一下的：

+ current，表示byte数组里当前的index
+ currentWord表示output数组里当前的index
+ inAlphanumeric，这个是标识符，为true的时候，说明它遇到了一个字母或数字，然后后面的代码会有相应的操作，直到它变为false，那么就说明从它为true到false的这段空间里，就为一个英文单词或者数字。
+ alphanumericStart这个用来记录inAlphanumeric变成true的那个位置

如下，就大概能明白了吧？说实在的，让我重写一次，估计还不能完全写出来。