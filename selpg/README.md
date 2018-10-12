Usage

```
Usage: selpg [-s startPage] [-e endPage] [-l linesPerPage | -f] [-d printDest] filename

Options:
  -d, --dest string		(Optional) Enter printing destination
  -e, --endPage int		(Mandatory) Input Your endPage
  -f, --pageBreak		(Optional) Choosing pageBreaks mode
  -l, --pageLen int     (Optional) Choosing pageLen mode, enter pageLen
  -s, --startPage int	(Mandatory) Input Your startPage
```
  代码结构

总共有三个函数：

    main函数：解析命令参数的入口函数

    processArgs函数：处理参数，进行错误处理

    processInput函数：经过processArgs函数后，这里根据命令进行（文件）操作

除此之外，还有保存参数的struct：selpgArgs，以及五个解析参数用的flag。

pflag 参数绑定操作：

```
var ip *int = flag.Int("flagname", 1234, "help message for flagname")	//绑定"--flagname"的值到*ip上，格式为int
//或
var flagvar int		//声明
func init() {
    flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")	//绑定
}
```

在Selpg中，我们需要两种格式的参数，拿输入文本的起始页做例子，我们就需要同时接受--startPage与-s这两种输入，pflag也允许我们这样做：
```
var inputS = flag.IntP("startPage", "s", -1, "...")
//startPage配合"--"使用，s配合"-"使用
```

flag.Parse()自动进行参数的捕获、解析。如果是没有带-或--的参数，则可通过flag.Args()获取，这类参数的数量为flag.Narg()。

读取输入部分：
```
//从命令行s输入
inputReader = bufio.NewReader(os.Stdin) 
//或打开一个文件后输入
file, err = os.Open(selpg.inFilename)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
inputReader = bufio.NewReader(file)	
//读取输入，直到某一字符（包含此字符）
line, err = inputReader.ReadBytes('\n')	//换页为行模式
line, err = inputReader.ReadBytes('\f')	//换页为换页符模式
```


定向输出部分：
```
outputWriter = bufio.NewWriter(os.Stdout)
outputWriter.Write(line)
outputWriter.Flush()	
```

os/exec:“-dXXX”的实现
```
//初始化外部命令
cmd = exec.Command("lp", "-d", selpg.printDest)	
//建立一个管道以输入打印内容
stdin, err = cmd.StdinPipe()
//写入管道
_, err := io.WriteString(stdin, string(line))
//关闭管道、获得输出
stdin.Close()
stderr, _ := cmd.CombinedOutput()
```
