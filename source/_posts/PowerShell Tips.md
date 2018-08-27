
---
title: PowerShell Tips
date: 2018-07-07 14:08:04
categories:
   - 技术相关
tags:
   - PowerShell
   - 笔记
   - Tips

---


## 0x01 PowerShell条件判断

PowerShell Switch 条件
如果语句中有多路分支,使用IF-ELSEIF-ELSE不友好,可以使用Switch,看起来比较清爽一点。
下面的例子将If-ElseIF-Else转换成Switch语句

	# 使用 IF-ElseIF-Else
	If( $value -eq 1 )
	{
		"Beijing"
	}
	Elseif( $value -eq 2)
	{
		"Shanghai"
	}
	Else
	{
		"Chongqing"
	}
	 
	# 使用 Switch
	switch($value)
	{
		1 {"Beijing"}
		2 {"Shanghai"}
		3 {"Chongqing"}
	}
	

<!-- more -->

PowerShell IF-ELSEIF-ELSE 条件

Where-Object 进行条件判断很方便,如果在判断后执行很多代码可以使用IF-ELSEIF-ELSE语句。语句模板：

If(条件满足){如果条件满足就执行代码}Else{如果条件不满足}

条件判断必须放在圆括号中,执行的代码必须紧跟在后面的花括号中。

	$n=8
	if($n -gt 15) {"$n  大于 15 " }
	if($n -gt 5) {"$n  大于 5 " }
	8  大于 5
	if($n -lt 0 ){"-1" } elseif($n -eq 0){"0"} else {"1"}
	1

没有匹配条件

在IF-Else语句中如果没有合适的条件匹配,可以在Else中进行处理,同样在Switch语句中如果case中没有条件匹配,可以使用关键字Default进行处理。
如果碰到匹配条件时只处理一次,可以使用Break关键字

	$value=99
	# 使用 Switch 测试取值范围
	switch($value)
	{
		5 { "等于5"; break}
		{$_ -gt 0 }   { "大于0"; break}
		{$_ -lt 100}  { "小于100"; break}
		Default {"没有匹配条件"}
	}
	 
	#大于0


大小写敏感
怎样在比较字符串时能够恢复为大小写敏感模式,Switch有一个-case 选项,一旦指定了这个选项,
比较运算符就会从-eq 切换到 -ceq,即大小写敏感比较字符串：

	$domain="eddDEE"
	#大小写敏感
	switch -case ($domain)
	{
		"edDDEE" {"Ok 1"}
		"EddDEE" {"Ok 2"}
		"eddDEE" {"Ok 3"}
	}
	#Ok 3

使用通配符
字符串非常特殊,可是使用通配符,幸运的是PowerShell也支持,果然Power啊。但是在Switch语句后要指定 -wildcard 选项

	$domain="xxx.x.com"
	#使用通配符
	switch -wildcard($domain)
	{
		"*"     {"匹配'*'"}
		"*.com" {"匹配*.com" }
		"*.*.*" {"匹配*.*.*"}
	}
	匹配'*'
	匹配*.com
	匹配*.*.*

在字符串匹配中,比通配符功能更强大是正则表达式,PowerShell的Switch语句也支持,真是太棒了。当然需要给Switch关键字指定选项-regex

	$mail="www@xxx.com"
	#使用通配符
	switch -regex ($mail)
	{
		"^www"     {"www打头"}
		"com$"     {"com结尾" }
		"d{1,3}.d{1,3}.d{1,3}.d{1,3}" {"IP地址"}
	}
	#www打头
	#com结尾

同时处理多个值

Switch支持对集合所有元素进行匹配,下面的例子使用PowerShell Switch语句演示打印水仙花数：

	$value=100..999
	switch($value)
	{
	{[Math]::Pow($_%10,3)+[Math]::Pow( [Math]::Truncate($_%100/10) ,3)+[Math]::Pow( [Math]::Truncate($_/100) , 3) -eq $_} {$_}
	}
	 
	#153
	#370
	#371
	#407

## 0x02 PowerShell 流控制语句

### 1. PowerShell Do While 循环

Do和While可能产生死循环,为了防止死循环的发生,你必须确切的指定循环终止的条件。指定了循环终止的条件后,一旦条件不满足就会退出循环。

继续与终止循环的条件

do-while()会先执行再去判断,能保证循环至少执行一次。

	do { $n=Read-Host } while( $n -ne 0)
	10
	100
	99
	20
	0		(退出了)

如果你知道循环的确切次数可以使用For循环,For循环属于计数型循环,一旦达到最大次数,循环就会自动终止。
下面的例子通过循环求1-100的数列和。

	$sum=0
	for($i=1;$i -le 100;$i++)
	{
		$sum+=$i
	}
	$sum
	
### 2. For循环是特殊类型的While循环

在For循环开始的圆括号中,由分号隔开的语句为循环的控制条件,分别为：初始化,循环执行满足的条件,增量。
For循环的控制语句第一个和第三个可以为空：

	$sum=0
	$i=1
	for(;$i -le 100;)
	{
		$sum+=$i
		$i++
	}
	$sum

下面的例子演示逐行读取文本文件

	for($file=[IO.File]::OpenText("C:\Users\w0nd\Desktop\mimi1.txt") ; !($file.EndOfStream);$line=$file.ReadLine() )
	{
		$line;
	}
	$file.Close()

label,break,continue,例：

	for ($i = 0; $i -le 15; $i++)
	{
		if ($i % 2)
		{   continue }
		if ($i -eq 10)
		{   break   }

		$i
	}

	:outer while ($true)
	{
		while　($true)
		{   break outer }
	}

### 3. PowerShell ForEach-Object 循环

PowerShell管道就像流水线,对于数据的处理是一个环节接着一个环节,

如果你想在某一环节对流进来的数据逐个细致化的处理,可是使用ForEach-Object,$_ 代表当前的数据。

对管道对象逐个处理

如果使用Get-WmiObject 获取系统中的服务,为了排版可能会也会使用Format-Table对结果进行表格排版。

	Get-WmiObject win32_service | ft DisplayName,processid,state -AutoSize

但是如果想对每个服务进行更定制化的处理可是使用ForEach-Object

	Get-WmiObject Win32_Service | ForEach-Object {"Name:"+ $_.DisplayName, ", Is ProcessId more than 100:" + ($_.ProcessId -gt 100)}

结合条件处理

ForEach-Object的处理可以包含任意PowerShell脚本,当然也包括条件语句

	Get-WmiObject Win32_Service | ForEach-Object {if ($_.ProcessId -gt 3000){ "{0}({1})" -f $_.DisplayName,$_.ProcessID}}
	--> 
		Client License Service (ClipSVC)(6072)
		SSDP Discovery(3224)
		Windows Modules Installer(4572)
		Windows Search(5008)

调用方法

在ForEach-Object中,$_代表当前对象,当然也允许通过$_,调用该对象支持的方法。

下面的例子杀死所有IE浏览器进程：

	Get-Process iexplore
	Get-Process iexplore | ForEach-Object {$_.kill()}


### 4. PowerShell Foreach 循环
Foreach-object 为cmdlet命令,使用在管道中,对管道结果逐个处理,foreach为遍历集合的关键字
下面举两个例子：

	$array=7..10
	foreach ($n in $array)
	{
		$n*$n
	}
				or  1..10 | % {$a=$_ ; foreach($xx in $a){$xx * $xx}}
	#49
	#64
	#81
	#100
 
	foreach($file in dir c:\windows)
	{
		if($file.Length -gt 1mb)
		{
			$File.Name
		}
	}
				or foreach($file in dir c:\windows){if($file.Length -gt 1mb) {  $File.Name}}
	#explorer.exe
	#WindowsUpdate.log

这里只为了演示foreach,其实上面的第二个例子可以用Foreach-Object更简洁。

	dir C:\Windows | where {$_.length -gt 1mb} |foreach-object {$_.Name}



## 0x03 PowerShell定义函数

### 1. 函数是自定义的PowerShell代码,有三个原则

	简短：函数名简短,并且显而易见。
	聚合：函数可以完成多个操作。
	封装和扩展：将一批PowerShell语句进行封装,实现全新的功能需求。

函数的结构由三部分组成：函数名,参数,函数体

### 2. 使用函数作为别名

假如PowerShell不支持"cd.." 命令,你可以通过定义函数实现这个功能：

	Function cd.. { cd ..}		或者Function xx { cd ..} 之后运行xx就可以了


假如PowerShell不支持Ping命令,也可以如法炮制：

	ping.exe  : -n count       Number of echo requests to send.
	Function Ping2 { PING.EXE  -n 1 $args }
	ping2 xx.com  

	del function:ping2


### 3. PowerShell处理函数的参数

PowerShell函数可以接受参数,并对参数进行处理。函数的参数有3个特性：

	任意参数：内部变量$args 接受函数调用时接受的参数,$args是一个数组类型。
	命名参数：函数的每一个参数可以分配一个名称,在调用时通过名称指定对应的参数。
	预定义参数：函数在定义参数时可以指定默认值,如果调用时没有专门指定参数的值,就会保持默认值。

$args 万能参数

给一个函数定义参数最简单的是使用$args这个内置的参数。它可以识别任意个参数。尤其适用哪些参数可有可无的函数。

	function sayHello
	{
		if($args.Count -eq 0)
		{
			"No argument!"
		}
		else
		{
			$args | foreach {"Hello,$($_)"}
		}
	}
	
多个参数调用时：

	sayHello LiLi Lucy Tom
	Hello,LiLi
	Hello,Lucy
	Hello,Tom

因为$args是一个数组,可以用它求和

	function Add
	{
	$sum=0
	$args | foreach {$sum=$sum+$_}
	$sum
	}
	Add 10 7 3 100
	#120


	function StringContact($str1,$str2)
	{
		return $str1+$str2
	}
	 
	StringContact cc dd
	StringContact -str1 word -str2 press
	ccdd
	wordpress

	function DayOfWeek([datetime]$date)
	{
		return  $date.DayOfWeek
	}
	DayofWeek '1927-8-1'
	#Monday
	DayofWeek 2008-8-1
	#Friday

Switch 参数
PowerShell函数最简单的参数类型为布尔类型,除了使用Bool类型,也可以使用Switch关键字。
下面的函数逆转字符串,但是可以通过$try 参数进行控制,如果没有指定$try的值,默认值为$false

	function  tryReverse( [switch]$try , [string]$source )
	{
	    [string]$target=""
	    if($try)
	    {
		for( [int]$i = $source.length -1; $i -ge 0 ;$i--)
		{
		    $target += $source[$i]
		}
		return $target
	    }
	    return $source
	}
	tryReverse -source www.xxx
	tryReverse -try $true -source www.xxx
	->
	www.xxx
	xxx.www


PowerShell指定函数的返回值 
一个或多个返回值

	function Square([double]$num)
	{
		return $num*$num
	}
	#在控制台输出结果
	Square 9.87
	#97.4169
	 
	#将结果赋值给变量
	$value=Square 9.87
	$value
	#97.4169
	 
	#返回值为Double类型
	$value.GetType().FullName
	#System.Double


Return语句
PowerShell会将函数中所有的输出作为返回值,但是也可以通过return语句指定具体的我返回值。

Return 语句会将指定的值返回,同时也会中断函数的执行,return后面的语句会被忽略。

	function test($num)
	{
		1
		9
		return 10
		4
		6
	}
	test
	# 1 和 9 作为输出会返回
	# return语句中的10 也会返回
	# return 语句后的4和6会被忽略
 

从函数的返回值中消除输出

函数默认会将函数中的所有输出作为函数的返回值返回,这样很方便。但有时可能会将不必要的输出误以为返回值。写脚本程序时,

可能需要自定义一些函数,这个函数可能只需要一个返回值,但是为了提高函数的可读性,可能会在函数增加一些注释输出行。

	Function Test()
	{
		"Try to calculate."
		"3.1415926"
		"Done."
	}
	 
	#保存在变量中输出,
	$value=Test
	$value
	# Try to calculate.
	# 3.1415926
	# Done.
	 
	#如果要过滤注释,只输出,不作为返回值,
	#可以使用Write-Host命令
	Function Test()
	{
		Write-Host "Try to calculate."
		"3.1415926"
		Write-Host "Done."
	}
	# 在变量值中保存返回值,在控制台输出注释行
	$value=Test
	# Try to calculate.
	# Done.
	 
	# 测试返回值
	$value
	# 3.1415926


使用调试信息报告
可能输出这些函数中临时提示信息,给函数的返回值造成干扰。要解决这个问题,除了上述的Write-Host,也可以使用Write-Debug命令。

	Function Test()
	{
		Write-Debug "Try to calculate."
		"3.1415926"
		Write-Debug "Done."
	}
	# Debug调试信息只会在调试模式下被输出
	$value=Test
	# 3.1415926
	 
	#如果你想通过显示调试信息调试函数,可以开启调试模式
	$DebugPreference="Continue"
	$value=Test
	# DEBUG: Try to calculate.
	# DEBUG: Done.
	 
	# 测试返回值
	$value
	# 3.1415926
 
	#如果关闭调试模式,这些调试信息自然不会输出
	$DebugPreference="SilentlyContinue"
	$value=Test

抑制错误信息

函数中的错误信息,也有可能作为返回值的一部分,因为默认这些错误信息会直接输出。

	Function ErrorTest()
	{
		#该进程不存在
		Stop-Process -Name "xxx"
	}
	ErrorTest
	 
	Stop-Process : 找不到名为"xxx"的进程。
 
很明显,类似这样的错误提示信息,对调试程序很重要,但如果你觉得它不重要,特意要隐藏,可以使用$ErrorActionPreference进行设置。
 
	Function ErrorTest()
	{
		#从这里开始隐藏所有的错误信息
		$ErrorActionPreference="SilentlyContinue"
		Stop-Process -Name "www.xxx"
		#该进程不存在
	}
	 
	#错误信息不会输出
	ErrorTest


	PS E:xxx> Function output
	>> {
	>>    $input
	>> }
	>>
	PS E:xxx> output
	PS E:xxx> 1,2,3 | output
	1
	2
	3
	PS E:xxx> dir | output
	 
		目录: E:xxx
	 
	Mode                LastWriteTime     Length Name
	----                -------------     ------ ----
	d----         2012/2/28     23:34            a
	d----         2012/2/28     23:35            b


## 0x04 PowerShell 编写和运行脚本 

### 1. 通过重定向创建脚本

如果您的脚本不是很长,您甚至可以直接在控制台中要执行的语句重定向给一个脚本文件。

	PS E:> '"Hello,PowerShell Script"' > MyScript.ps1
	PS E:> .\MyScript.ps1
	Hello,PowerShell Script

脚本文件通过@''@闭合起来 (PowerShell ISE):

	@'
	get-date
	$env:commonProgramfiles
	#note : script end

	"files count"
	(ls).count

	#note : script really end     

	'@ > 11.ps1
	
	输出 ->
		Monday, August 14, 2017 1:36:26 AM
		C:\Program Files\Common Files
		files count
		106
	
Here-String以 @'开头,以'@结束.任何文本都可以存放在里面,哪怕是一些特殊字符,空号,白空格。

但是如果您不小心将单引号写成了双引号,PowerShell将会把里面的变量进行解析。


### 2. 执行策略限制

PowerShell一般初始化情况下都会禁止脚本执行。脚本能否执行取决于PowerShell的执行策略。

PS E:> .\MyScript.ps1

无法加载文件 E:MyScript.ps1,因为在此系统中禁止执行脚本....

	PS E:> Get-ExecutionPolicy
	Restricted
	PS E:> Set-ExecutionPolicy RemoteSigned
	执行策略更改
	脚本执行策略类型为：Microsoft.PowerShell.ExecutionPolicy
	查看所有支持的执行策略：

	PS E:>  [System.Enum]::GetNames([Microsoft.PowerShell.ExecutionPolicy])
	Unrestricted
	RemoteSigned
	AllSigned
	Restricted
	Default
	Bypass
	Undefined
	Unrestricted:权限最高,可以不受限制执行任何脚本。
	Default:为PowerShell默认的策略：Restricted,不允许任何脚本执行。
	AllSigned：所有脚本都必须经过签名才能在运行。
	RemoteSigned：本地脚本无限制,但是对来自网络的脚本必须经过签名。

	if：
	Import-Module : File Invoke-Phant0m.ps1 is not digitally signed. You cannot run this
	PS C:\> Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass		===> ok

### 3. 像命令一样执行脚本

怎样像执行一个命令一样执行一个脚本,不用输入脚本的相对路径或者绝对路径,甚至*.ps1扩展名。

那就将脚本的执行语句保存为别名吧：

	PS E:> Set-Alias Invok-MyScript .11.ps1
	PS E:> Invok-MyScript

	Monday, August 14, 2017 1:36:26 AM
	C:\Program Files\Common Files
	files count
	106

	@'
	For($i=0;$i -lt $args.Count; $i++)
	{
		Write-Host "parameter $i : $($args[$i])"
	}
	'@ >111.ps1

	PS C:\Users\w0nd\Desktop> .\111.ps1 dd 3232 4 2
	parameter 0 : dd
	parameter 1 : 3232
	parameter 2 : 4
	parameter 3 : 2


### 4. PowerShell 增强脚本的可读性 
在脚本中使用函数

要在脚本中使用函数,最简单的方法自然是将函数直接写在脚本中：

	param([int]$n=$(throw "请输入一个正整数"))
	Factorial $n
	Function Factorial([int]$n)
	{
		$total=1
		for($i=1;$i -le $n;$i++)
		{
			$total*=$i
		}
		return $total
	}

	PS C:\Users\w0nd> .\12.ps1 10
	3628800

将脚本分为工作脚本和类库

真正的脚本开发需要处理的问题可能包含许多函数。如果在一个脚本的开头定义许多函数,脚本会显得很凌乱。

把函数和工作脚本分开,可以隔离函数,使它不容易被修改。

将Factorial函数保存在PSLib.ps1

	Function Factorial([int]$n)
	{
		$total=1
		for($i=1;$i -le $n;$i++)
		{
			$total*=$i
		}
		return $total
	}
	将脚本修改为：
	param([int]$n=$(throw "请输入一个正整数"))
	. .PSLib.ps1
	Factorial $n
	PS C:\Users\w0nd> .\12.ps1 10
	3628800


### 5. PowerShell 创建管道脚本
低速顺序模式

如果你在脚本中使用管道,脚本收集上一个语句的执行结果,默认保存在$input自动变量中。

但是直到上一条语句完全执行彻底,管道脚本才会执行。

	@'
	foreach ($element in $input)
	{
		if($element.Extension -eq ".exe")
		{
			Write-Host -fore "red" $element.Name
		}
		else
		{
			Write-Host -fore "Green" $element.Name
		}
	}   
	'@ > 12.ps1

	PS C:\Users\YvY> ls $env:APPDATA | .\12.ps1
		Adobe
		Everything
		Microsoft
		Notepad++

高速流模式

在PowerShell脚本的处理中,绝大多数情况下遇到的都是集合,一旦上一条命令产生一个中间结果,
下一条命令就对这个中间结果及时处理,及时释放资源。这样可以节省内存,也减少了用户的等待时间。在处理大量数据时,尤其值得推荐。
高速流模式的管道定义包括三部分：begin,process,end。上面的描述中提到了中间结果,中间结果保存在$_自动化变量中。

	@'
	begin
	{
	    Write-Host "管道脚本环境初始化"
	}
	process
	{
	    $ele=$_
	    if($_.Extension -ne "")
	    {
	        switch($_.Extension.tolower())
	        {
	            ".ps1" { Write-Host -fore "red" "脚本文件："+ $ele.name}
	            ".txt" { Write-Host -fore "red" "文本文件："+ $ele.Name}
	            ".gz"  { Write-Host -fore "red" "压缩文件："+  $ele.Name}
				".exe"  { Write-Host -fore "green" "exe文件："+  $ele.Name}
				".log"  { Write-Host -fore "red" "日志文件：" + $ele.Name}			# +号后面必须有空格
	        }
	    }
	}
	end
	{
	    Write-Host "管道脚本环境恢复"
	}   
	'@ > test.ps1


### 6. PowerShell自动执行脚本之profile 

在PowerShell控制台的许多更改只会在当前会话有效。一旦关闭当前控制台,你自定义地所有别名、函数、和其它改变将会消失,

除非将更改保存在windows环境变量中。这也就是为什么我们需要profile来保存一些基本的初始化工作。

四中不同的profile脚本

PowerShell支持四种可以用来初始化任务的profile脚本。

应用之前要弄清楚你的初始化是当前用户个人使用,还是所有用户。

如果是个人使用,可以使用"当前用户profile",但是如果你的初始化任务是针对所有用户,可是使用"所有用户profile"。

	Profile			描述										位置
	所有用户			所有用户共有的profile						$pshomeprofile.ps1
	所有用户(私有)	PowerShell.exe 中验证。						$pshomeMicrosoft.PowerShell_profile.ps1
	当前用户			当前用户的profile							$((Split-Path $profile -Parent) + "profile.ps1")
	当前用户(私有)	当前用户的profile;只在PowerShell.exe中验证	$profile

创建自己的profile

Profile脚本并不是强制性的,换言之,profile可有可无。下面会很方便的创建自己的profile。 
在控制台执行：
 
	notepad $((Split-Path $profile -Parent) + "profile.ps1") 		===> C:\Users\w0nd\Documents\WindowsPowerShellprofile.ps1

如果不存在profile默认会创建,在打开的记事本中输入： Set-Alias edit notepad.exe

也就是给notepad添加edit别名,保存关闭,之后重启控制台,输入： 

	edit $((Split-Path $profile -Parent) + "profile.ps1")
	
控制台会调用记事本打开之前的profile,可见edit别名已经生效。


脚本的数字签名
http://www.pstips.net/PowerShell-scripts-signature.html   	[没看懂]

PowerShell错误处理
what-if
http://www.pstips.net/PowerShell-what-if.html				[没看懂]

## 0x05 PowerShell命令发现和脚本块

### 1. PowerShell 发现命令 

	PS C:> Get-command LS

	CommandType Name Definition
	----------- ---- ----------
	Alias       ls   Get-ChildItem
	
如果你想查看更加详细的信息可以使用：

	PS C:>Get-Command ls | fl *
	
事实上,Get-Command 返回的是一个对象CommandInfo,ApplicationInfo,FunctionInfo,或者CmdletInfo;

	PS C:> $info = Get-Command ping
	PS C:> $info.GetType().fullname
	System.Management.Automation.ApplicationInfo

通过函数可以重写ipconfig ,一旦删除该函数,原始的ipconfig才会重新登上历史的舞台：

	PS C:> function ipconfig () {}
	PS C:> ipconfig
	PS C:> del Function:ipconfig

### 2. PowerShell 语句块

脚本块是一种特殊的命令模式。一个脚本块可以包含许多的 PowerShell命令和语句。它通常使用大括号定义。

最小最短的脚本块,可能就是一对大括号,中间什么也没有。可以使用之前的调用操作符"&"执行脚本块：

	PS C:\Users\w0nd> & {"当前时间:" + (get-date) }
	当前时间:07/06/2018 23:01:37

将命令行作为整体执行

可能你已经意识到,在PowerShell中,调用操作符不但可以执行一条单独的命令,还可以执行"命令行".

最方便的方式就是讲你的命令行放在一个语句块中,作为整体。

在之前的文章中说过,调用操作符只能执行一条命令,但是借助语句块的这把利器,

可以让调用操作符执行,多条PowerShell命令,例如：

	PS C:\Users\w0nd> & {$files=ls;Write-Host "文件数：" $files.Count }
	文件数： 21

执行表达式

另外还有一条PowerShell命令集,Invoke-Expression=IEX,这条命令的逻辑就是将一条字符串传递给调用操作符。例如：

PS C:\Users\w0nd> iex 'Get-Process | Where-Object { $_.Name -like "e*"}'

	Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                      
	-------  ------    -----      -----     ------     --  -- -----------                      
		126       8     1844       1904       1.06    732   0 Everything                       
		253      16    18532      17872       3.91   3936   1 Everything                       
		808      45    44380      45352       5.27   4968   1 EXCEL                            
	   2007     105    50536      99288     271.77   3100   1 explorer    
	   
这里有一点需要注意,在传递给invoke-expression的字符串使用了单引号,单引号可以防止变量被替换。

如果上面的命令使用了双引号,会先去解释$_.name,但是当前作用域中,$_.Name 为null,所以结果不是期望的。
	
管道中的foreach-object语句块

管道中的foreach-object本身后面也会带语句块,针对数组中的每一个元素分别传递给语句块处理。 例如：

	http://www.pstips.net/PowerShell-using-scriptblocks.html
	

向命令,函数和文件脚本传递文件

因为Dir的结果中返回的是独立的文件或目录对象,Dir可以将这些对象直接交付给其它命令或者你自己定义的函数与脚本。

这也使得Dir成为了一个非常重要的的选择命令

将多个Dir 命令执行的结果结合起来。

	$list1 = Dir $env:windir\system32\*.dll
	$list2 = Dir $env:programfiles -recurse -filter *.dll
	$totallist = $list1 + $list2
	$totallist | ForEach-Object {
	$info =
	[system.diagnostics.fileversioninfo]::GetVersionInfo($_.FullName);
	"{0,-30} {1,15} {2,-20}" -f $_.Name, `
	$info.ProductVersion, $info.FileDescription
	}

	# 只列出目录::
	Dir | Where-Object { $_ -is [System.IO.DirectoryInfo] }
	Dir | Where-Object { $_.PSIsContainer }
	Dir | Where-Object { $_.Mode.Substring(0,1) -eq "d" }
	# 只列出文件:
	Dir | Where-Object { $_ -is [System.IO.FileInfo] }
	Dir | Where-Object { $_.PSIsContainer -eq $false}
	Dir | Where-Object { $_.Mode.Substring(0,1) -ne "d" }
	
	Dir $home -filter *.ps1 -recurse	-filter支持简单的模式匹配
	Dir $home -include *.ps1 -recurse 	-include支持正则表达式

Where-Object也可以根据其属性来过滤。

通过管道过滤2007年5月12日后更改过的文件:

	Dir | Where-Object { $_.CreationTime -gt [datetime]::Parse("May 12, 2007") }

也可以使用相对时间获取2周以内更改过的文件:

	Dir | Where-Object { $_.CreationTime -gt (Get-Date).AddDays(-14) }
	Dir -Recurse | Where-Object { $_.CreationTime -gt (Get-Date).AddDays(-14) }




相对路径转换成绝对路径

当Resolve-Path命令只有在文件确实存在时,才会有效。

	PS> Resolve-Path $env:windir\*.log



构造路径

	$path = [Environment]::GetFolderPath("Desktop") + "\file.txt"
	
使用命令 Join-Path方法,或者.NET中的Path静态类。

	$path = Join-Path ([Environment]::GetFolderPath("Desktop")) "test.txt"

	$path = [System.IO.Path]::Combine([Environment]::GetFolderPath("Desktop"), "test.txt")

Path类还包含了许多用来合并或者获取目录特定信息的额外方法。你只需要在下面表格中列出的方法中前加[System.IO.Path]::,比如:

	[System.IO.Path]::ChangeExtension("test.txt", "ps1")
	test.ps1


使用Get-Process返回所有的当前进程 ,但是你可能并不对所有的进程感兴趣

	Get-Process | select -First 1 | fl *			
	
然后通过每个Process对象的属性进行过滤。首先得知道每个对象支持那些属性

根据进程名过滤(显示)所有记事本进程

	Get-Process | Where-Object {$_.Name -eq "notepad"}

根据company过滤所有产品发布者以"Microsoft"打头的进程：

	Get-Process | Where-Object {$_.company -like '*Microsoft*' }| select Name,Description,Company

Where-Object的使用概率比较高,所以有一个很形象的别名 ?

	Get-Service | ? {$_.Name -like "B*"}
	
## 0x06 Reference

	https://www.pstips.net/
	<< PowerShell 实战指南(第二版) >>
	
	
	
	
	
	
	




