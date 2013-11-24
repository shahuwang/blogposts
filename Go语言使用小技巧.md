## Go语言一些小技巧  
正如上篇文章所说的，想用Go语言做一个半静态博客的努力失败了，做到最后，还是觉得自己更需要的是wordpress这样的博客，同时具备静态博客写作的方便性。  

不过，既然花了这么多的努力，写了将近三百行代码，那么也就不要浪费掉了，还是先记录到网上来先。写博客对于我来说挺重要的，因为很多东西，我的脑子里只能记住索引，不能记住全部。记在博客上，我知道哪里可以找到，以后找来一看，马上就能明白了。  

#####判断文件夹是否存在

很多时候写代码，真的不是什么惊天动地的事情，大部分时候都是在处理这样的繁琐事情上。Go语言对文件夹和文件的操作比Java简单了许多，判断文件夹是否存在的代码如下 ：  

    func isDirExists(path string) bool {
    	fi, err := os.Stat(path)
    
    	if err != nil {
    		return os.IsExist(err)
    	} else {
    		return fi.IsDir()
    	}
    
    	panic("not reached")
    }   

#####复制文件到指定的地方

代码示例如下：

        
    /** 复制文件到指定地方 **/
    func CopyFile(src, dest string) (w int64, err error) {
    	srcFile, err := os.Open(src)
    	if err != nil {
    		fmt.Println(err)
    	}
    	defer srcFile.Close()
    
    	desFile, err := os.Create(dest)
    	if err != nil {
    		fmt.Println(err)
    	}
    	defer desFile.Close()
    
    	return io.Copy(desFile, srcFile)
    }

#####复制文件夹到指定的地方
这短代码还是找了很久才找着了，自己能力不够，只能看别人的代码了：  
    
    /**复制某个文件夹的文件到另一个文件夹 **/
    func CopyDir(srcDir, destDir string) {
    
    	if isDirExists(srcDir) {
    		//存在附件文件夹
    		tmpSrc := strings.TrimSpace(srcDir)
    		files, _ := ioutil.ReadDir(tmpSrc)
    		for _, f := range files {
    			CopyFile(srcDir+f.Name(), destDir+f.Name())
    		}
    	}
    }

#####把字符串数组拼接为字符串  
要高效得把数组里所有的字符串拼接为一个字符串，用到了bytes包的功能，代码如下：

    result := bytes.Buffer{}
	for i := end; i < len(lines); i++ {
		result.WriteString(lines[i])
	}
	
	return result.String()