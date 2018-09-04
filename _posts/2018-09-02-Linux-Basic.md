---
layout: post
title: Linux Basic
category: Linux
tags:
  - Linux
---





### 框架
	服务器：DHCP, DNS, FTP, HTTP, cobbler

	IaaS: xen, kvm, lxc
	
    
    知识框架：
    	Linux基础知识
    		系统管理
    	shell脚本编程
    	Linux服务管理
    		openssl, web, ftp, samba, nfs, dhcp, dns
    	MySQL数据库系统
    	Linux集群：
    		LB：lvs, nginx, haproxy
    		HA: heartbeat, corosync, rhcs, keepalived
    	分布式应用：
    		MogileFS
    		MongoDB (NoSQL)
    		HDFS, 
    		MapReduce
    	缓存系统：varnish
    	虚拟化：xen, kvm, openstack
    	监控和自动化：
    		zabbix, puppet, cobbler, ansible


### Linux基础命令
#### bash特性之文件名通配(globbing):
	*: 任意长度的任意字符
		p*d, pad, pbd, pd
		*ab*c
	?: 匹配任意单字符
	[]: 匹配指定范围内的任意单字符
		[abc], [a-z], [0-9], [0-9a-z]
	[^]：匹配指定范围以外的任意单字符
		[^0-9a-z]

		字符集合：
			[:space:] : 所有空白字符
			[:punct:] : 所有标点符号
			[:lower:] ：所有小写字母
			[:upper:]
			[:digit:]
			[:alnum:]
			[:alpha:]

	练习：
		1、显示/var目录下所有以l开头，以一个小字母结尾，且中间出现一位数字的文件或目录；
			# ls /var/l*[[:digit:]]*[[:lower:]]
		2、显示/etc目录下，以任意一位数字开头，且以非数字结尾的文件或目录；
			# ls -d /etc/[[:digit:]]*[^[:digit:]]
		3、显示/etc目录下，以非字母开头，后面跟了一个字母及其它任意长度字符的文件或目录；
			# ls -d /etc/[^[:alpha:]][[:alpha:]]*

	练习：
		1、在/tmp/mytest目录中创建以testdir打头，后跟当前日期和时间的空目录，形如testdir-2014-07-03-09-15-33；
			# mkdir -pv /tmp/mytest/testdir-$(date +%F-%H-%M-%S)
			
			
####     文本处理类命令：
	wc: Word Count
		-l: 仅显示行数
		-w:
		-c:

	cut: 
		-d: 指定分隔符
		-f: 指定要显示的字段
			m: 第m列
			m,n: 第m和n列
			m-n: 第m到第n列

	sort: 
		sort [option] FILE...
			-f: 忽略字符大小写
			-t: 指定分隔符
			-k: 指定分隔之后要进行排序比较的字段
			-n: 以数值大小进行排序
			-u: 排序后去重

	uniq: 
		-d
		-u
		-c: 统计行出现的次数

	练习：
		1、显示当前系统上每个用户的shell；
			# cut -d: -f1,7 /etc/passwd

		2、显示当前系统上所有用户使用的各种shell；
			# cut -d: -f7 /etc/passwd | sort | uniq

		3、取出/etc/inittab文件的第7行；
			# head -n 7 /etc/inittab | tail -n 1

		4、取出/etc/passwd文件中第7个用户的用户名；
			# head -n 7 /etc/passwd | tail -n 1 | cut -d: -f1

		5、统计/etc目录下以大小写p开头的文件的个数；
			# ls -d /etc/[pP]* | wc -l
			
			

#### 文件管理类命令：
	复制：cp
	移动：mv
	删除：rm

	cp: 
		cp SRC DEST
			SRC是文件：
				如果DEST不存在：复制SRC为DEST
				如果DEST存在：
					如果DEST是文件：则覆盖
					如果DEST是目录：将SRC复制进DEST中，并保持原名

		cp SRC... DEST
			如果SRC不止一个，则DEST必须得是目录；

		cp SRC DEST
			SRC是目录：
				可使用-r选项：

		cp -r SRC... DEST

		练习：复制/etc目录下，所有以p开头，以非数字结尾的文件或目录至/tmp/mytest1目录；
			# mkdir /tmp/mytest1
			# cp -r /etc/p*[^[:digit:]]  /tmp/mytest1

		练习：复制/etc/目录下，所有以.d结尾的文件或目录至/tmp/mytest2目录；
			# mkdir /tmp/mytest2
			# cp -r /etc/*.d  /tmp/mytest2

		练习：复制/etc/目录下所有以l或m或n开头，以.conf结尾的文件至/tmp/mytest3目录；
			# mkdir /tmp/mytest3
			# cp -r /etc/[lmn]*.conf /tmp/mytest3
			
			
#### 日期时间命令			
	date: 日期和时间
		date [options] [+FORMAT]
			%s: 时间戳计时法，从Unix元年(1970-01-01 00:00:00)到此刻所经过的秒数
			%F, %D
			%T
			%Y
			%m
			%d
			%H
			%M
			%S

		date [MMDDhhmm[[CC]YY][.ss]]

		Linux有两个时钟：系统时钟和硬件时钟
			硬件时钟：
			系统时钟：Linux

		hwclock
			-s: 以硬件为准
			-w：以系统为准

	ntp: Network Time Protocol
		通过网络同步系统时间

		C/S: Server, Client

	ntpdate SERVER

	who: 登录至当前系统的所有用户
	whoami: 当前终端上登录的用户

	which: 显示指定命令的完整路径
		--skip-alias: 路过命令别名



#### 文本处理



        
        
        
        cut:
        主要参数:
            -b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
            -c ：以字符为单位进行分割。
            -d ：自定义分隔符，默认为制表符。
            -f  ：与-d一起使用，指定显示哪个区域。
            -n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的<br />范围之内，该字符将被写出；否则，该字符将被排除。


        sort:
        -b：忽略每行前面开始出的空格字符；
        -c：检查文件是否已经按照顺序排序；
        -d：排序时，处理英文字母、数字及空格字符外，忽略其他的字符；
        -f：排序时，将小写字母视为大写字母；
        -i：排序时，除了040至176之间的ASCII字符外，忽略其他的字符；
        -m：将几个排序号的文件进行合并；
        -M：将前面3个字母依照月份的缩写进行排序；
        -n：依照数值的大小排序；
        -o<输出文件>：将排序后的结果存入制定的文件；
        -r：以相反的顺序来排序；
        -t<分隔字符>：指定排序时所用的栏位分隔字符；
        
    -k选项的语法格式：
     FStart.CStart 选项  ,  FEnd.CEnd 选项
     FStart就是表示使用的域，而CStart则表示在FStart域中从第几个字符开始算“排序首字符”
     
    从公司英文名称的第二个字母开始进行排序：
    $ sort -t ' ' -k 1.2 facebook.txt
    baidu 100 5000
    sohu 100 4500
    google 110 5000
    guge 50 3000
    
    使用了-k 1.2，表示对第一个域的第二个字符开始到本域的最后一个字符为止的字符串进行排序。
    
    只针对公司英文名称的第二个字母进行排序，如果相同的按照员工工资进行降序排序：
    $ sort -t ' ' -k 1.2,1.2 -nrk 3,3 facebook.txt
    baidu 100 5000
    google 110 5000
    sohu 100 4500
    guge 50 3000
    
    由于只对第二个字母进行排序，所以我们使用了-k 1.2,1.2的表示方式，
    表示我们“只”对第二个字母进行排序。
    对于员工工资进行排序，我们也使用了-k3,3，这是最准确的表述，表示我 “只”对本域进行排序，
    因为如果你省略了后面的3，就变成了我们“对第3个域开始到最后一个域位置的内容进行排序” 了。



	重定向意味着：
		改变其标准位置

	输出重定向：
		COMMAND > POSITION：覆盖输出
		COMMAND >> POSITION: 追加输出

	错误重定向：
		COMMAND 2> POSITION：覆盖输出
		COMMAND 2>> POSITION: 追加输出

	合并重定向：
		COMMAND &> POSITION
		COMMAND > POSITION 2> &1

	分别重定向
		COMMAND > POSTIION 2> POSTION2

	输入重定向：
		 COMMAND < POSITION

		 <<：Here Document

	文本处理命令：tr
		tr 'SET1' 'SET2'
			-d: 删除指定字符集合中的所有字符

	多道输出：
		COMMAND | tee POSITION
		
		
		
	练习：  
		1、统计当前系统上所有已经登录的用户会话数；
		# who | wc -l

		2、列出当前系统上所有已经登录的用户的用户名；
		# who | cut -d' ' -f 1 | sort -u

		3、取出最后登录到当前系统的用户的用户名；
		# who | sort -k 3,4 | cut -d' ' -f 1 | tail -1

		4、取出当前系统上被使用的次数最多的shell；(从/etc/passwd中取) 
		# cut -d: -f7 /etc/passwd | sort | uniq -c | sort -n | tail -1

		5、将/etc/passwd中第三个字段数据最大的后10个用户的信息全改为大写字符后保存到/tmp/mypasswd.txt文件中；
		# sort -t: -k3 -n /etc/passwd | tail | tr 'a-z' 'A-Z' > /tmp/mypasswd.txt



### Linux的文件系统


    权限：
    		r
    		w
    		x

	文件：
		r: 查看文件内容
		w: 修改文件内容
		x: 把此文件启动为一个运行的程序（进程）

	目录：
		r: 可使用ls命令查看目录中的文件名列表
		w: 可以在目录中创建或删除文件
		x: 可以cd到此目录中，以及使用ls -l显示目录中文件的元数据信息
	
	用户管理：
		Linux:
			/etc/passwd: 用户的帐号信息
			/etc/shadow: 用户密码和相关的帐户设定
			/etc/group: 组的帐号信息
			/etc/gshaow: 组的密码信息

		/etc/passwd文件：
			account:password:UID:GID:GECOS:directory:shell

			登录名:密码点位符:UID:GID:注释信息:家目录:用户的默认shell

				用户可以加入不止一个组：
					基本组
					额外组，附加组

		/etc/group文件：
			组名:组密码点位符:GID:以逗号分隔属于此组（以之做为额外组）的用户列表


			基本正则表达式：grep
			扩展正则表达式: egrep, grep -E
			fgrep: fast, 不支持使用正则表达式



#### 文本匹配处理
		基本正则表达式的元字符：
			字符匹配：
				.: 匹配任意单个字符
				[]: 匹配指定范围内的任意单个字符
					[0-9], [[:digit:]]
					[a-z], [[:lower:]]
					[A-Z], [[:upper:]]
					[[:space:]]
					[[:punct:]]
					[[:alpha:]]
					[[:alnum:]]
				[^]:
			次数匹配元字符：用于实现指定其前面的字符所能够出现的次数
				*: 任意长度，它前面的字符可以出现任意次
					例如：x*y
						xxy, xyy, y, 
				\?: 0次或1次，它前面的字符是可有可无的
					例如：x\?y
						xy, y, ay
				\{m\}: m次，它前的字符要出现m次
					例如：x\{2\}y
						xy, xxy, y, xxxxy, xyy
				\{m,n\}: 至少m次，至多n次
					例如：x\{2,5\}y
						xy, y, xxy
				\{m,\}：至少m次
				\{0,n\}: 至多n次

				.*：任意长度的任意字符

					工作于贪婪模式：尽可能多的去匹配
			位置锚定：
				^: 行首锚定；
					写在模式最左侧
				$: 行尾锚定：
					写在模式最右侧
				^$: 空白行

				不包含特殊字符的连续字符组成的串叫单词：
				\<: 词首，出现于单词左侧，\b
					\<char
				\>: 词尾，出现于单词右侧, \b
					char\>
			分组：
				\(\)
					例如：\(ab\)*
					分组中的模式匹配到的内容，可由正则表达式引擎记忆在内存中，之后可被引用

				引用：
					例如\(ab\(x\)y\).*\(mn\)
						有编号：自左而后的左括号，以及与其匹配右括号
						\(a\(b\(c\)\)mn\(x\)\).*\1

				\#: 引用第n个括号所匹配到的内容，而非模式本身
					例如：
						\(ab\?c\).*\1

							abcmnaaa
							abcmnabc
							abcmnac
							acxyac

		命令选项：
            -i 匹配时PATTERN中的条件，忽略器大小写
            grep -i 'Root' /etc/passwd
            -v 反向匹配，显示没有匹配到的行
            grep -v 'linux' /etc/passwd
            -o 只显示匹配到的内容
            grep -o 'gentoo' /etc/passwd
            --color=auto 支持扩展表达式
            grep --color=auto 'root' /etc/passwd
            -A 显示匹配行的后面指定数目行
            grep -A 2 'linux' /etc/passwd
            -B 显示匹配行的前面指定数目行
            grep -B 3 'linux' /etc/passwd
            -C 显示匹配行的前后指定相同数目行
            grep -C 2 'linux' /etc/passwd
            -n 显示输出行的行号
            grep -nC 2 'linux' /etc/passwd
            -E 支持扩展正则表达式
            grep -E '[0-9]+' /etc/passwd
            -c 只显示匹配到行的数目
            grep -c '^root\>' /etc/passwd

		练习：
			1、显示/proc/meminfo文件中以大写或小写S开头的行；
			# grep -i '^s' /proc/meminfo
			# grep '^[Ss]' /proc/meminfo

			# grep -E '^(S|s)' /proc/meminfo

			2、显示/etc/passwd文件中其默认shell为非/sbin/nologin的用户；
			# grep -v "/sbin/nologin$" /etc/passwd | cut -d: -f1

			3、显示/etc/passwd文件中其默认shell为/bin/bash的用户；
				进一步：仅显示上述结果中其ID号最大的用户；
			# grep "/bin/bash$" /etc/passwd | sort -t: -k3 -n | tail -1 | cut -d: -f1				

			4、找出/etc/passwd文件中的一位数或两位数；
			# grep "\<[0-9][0-9]\?\>" /etc/passwd
			# grep "\<[0-9]\{1,2\}\>" /etc/passwd

			5、显示/boot/grub/grub.conf中以至少一个空白字符开头的行；
			# grep "^[[:space:]]\{1,\}" /boot/grub/grub.conf

			6、显示/etc/rc.d/rc.sysinit文件中，以#开头，后面跟至少一个空白字符，而后又有至少一个非空白字符的行；
			# grep "^#[[:space:]]\{1,\}[^[:space:]]\{1,\}" /etc/rc.d/rc.sysinit

			7、找出netstat -tan命令执行结果中以'LISTEN'结尾的行；
			# netstat -tan | grep "LISTEN[[:space:]]*$"

			8、添加用户bash, testbash, basher, nologin（SHELL为/sbin/nologin），而找出当前系统上其用户名和默认shell相同的用户；
			# grep "^\([[:alnum:]]\{1,\}\):.*\1$" /etc/passwd

			9、扩展题：新建一个文本文件，假设有如下内容：
				He like his lover.
				He love his lover.
				He like his liker.
				He love his liker.
			找出其中最后一个单词是由此前某单词加r构成的行。
				\(l..e\).*\1r

		扩展正则表达式：
			字符匹配：
				.
				[]
				[^]
			次数匹配：
				*：任意次
				?: 0次或1次
				+: 至少1次；
				{m}: 精确匹配m次
				{m,n}: 至少m次，至多n次
				{m,}
				{0,n}
			锚定：
				^
				$
				\<, \b
				\>, \b
				^$, ^[[:space:]]*$
			分组：
				()

				引用：\1, \2, \3

			或者：
				a|b: a或者b
					con(C|c)at
						concat或conCat？
						conC或cat

			grep -E  'PATTERN' FILE...
			egrep 'PATTERN' FILE...

			练习：使用扩展的正则表达式
			10、显示当前系统上root、fedora或user1用户的默认shell；
			# grep -E "^(root|fedora|user1):" /etc/passwd | cut -d: -f7

			11、找出/etc/rc.d/init.d/functions文件中某单词后跟一组小括号“()”行；
			# grep -o -E "\<[[:alnum:]]+\>\(\)" /etc/rc.d/init.d/functions

			12、使用echo命令输出一个路径，而后使用grep取出其基名；
				echo "/etc/sysconfig/" | grep -o -E "[[:alnum:]]+/?"

				# echo "/etc/sysconfig/" | grep -o -E "[^/]+/?$" | cut -d/ -f1

			13、找出ifconfig命令结果中的1-255之间的数字；
			# ifconfig | grep -o -E "\<([1-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>"

			14、挑战题：写一个模式，能匹配合理的ipv4地址；
			1.0.0.1-239.255.255.255
			
			
#### vim

	ESC键：
		编辑模式-->输入模式：
			i: 在光标所在处的前方转换为输入模式
			a: 在光标所在的后方转换为输入模式
			o: 在光标所在行的下方新建一个空行并转换为输入模式

			I: 行首 
			A：行尾
			O: 光标所在行的上方新建一个空白行

		输入模式-->编辑模式
			ESC

		编辑模式-->末行模式
			:
		末行模式-->编辑模式
			ESC

		输入-->编辑-->末行

	退出文件：
		:q! 不保存退出
		:wq 保存退出
		:x 保存退出
		:wq! 强制保存退出

		编辑模式保存退出：ZZ
	
	编辑文本：

		光标移动：
			字符间移动：
				h,j,k,l
				#{h|j|k|l}: 跳#个字符

			单词间移动：
				w: 下一个单词词首
				e: 当前单词或下一个单词词尾
				b: 当前单词或前一个单词词首
				#{w|e|b}:

			行内移动：
				^: 行首第一个非空白字符
				0：绝对行首
				$: 绝对行尾

			句子间移动：
				) 
				(

			段落间移动：
				}
				{

			行间移动：
				#G: 直接跳转至第#行；
				G：最后一行

		编辑命令：
			x: 删除光标所在处的字符
			#x:

			d: 删除命令
				结合光标跳转字符使用，删除跳转范围内的字符
					w, b, e, 
					$, 0, ^
				#d
			dd: 删除光标所在行
			D: d$

			注意：最后一次删除的内容会被保存至缓冲区

			p: paste, 粘贴
				行级别：
					p: 粘贴于当前行下方
					P:             上
				小于行级别：
					p: 粘贴于当前光标所在处的后方
					P：                      前

			y: yank, 复制
				结合光标跳转字符使用，复制跳转范围内的字符
					w, b, e, 
					$, 0, ^
				#y
			Y: yy

			c: change，修改
				结合光标跳转字符使用，修改跳转范围内的字符
					w, b, e, 
					$, 0, ^
				先删除，再转换为输入模式
				cc, C: 删除光标所在处的整行而后转换为输入
				#c

		撤消编辑：
			u: undo, 
			#u: 撤消最近的#次操作

		撤消此前的撤消操作：
			Ctrl+r

		重复前一条命令：
			.

		vimtutor

	末行模式：
		行间跳转
			#
				$: 最后一行

		内容定界：
			startpos,endpos
				#: 第#行
				.: 当前行
				$: 最后一行
				%: 全文，相当于1,$
				10,$-1

			c, d, y等命令可以直接附加在地址范围后使用

			w /path/to/somefile: 将选定范围内的内容保存至某文件中
			r /path/from/somefile: 将指定的文件中的内容读取到指定位置

			s/查找模式/要替换成的内容/gi
				查找模式：可以使用正则表达式
				要替换成的内容：不能使用模式，仅能使用引用

			s@@@gi

				g: global, 全行替换
				i: 不区分字符大小写

				引用模式匹配到的所有内容，可以使用&符号

			练习：复制/etc/rc.d/init.d/functions至/tmp目录
				替换/tmp/functions文件中的/etc/sysconfig/init为/var/log
				%s/\/etc\/sysconfig\/init/\/var\/log/gi
				%s@/etc/sysconfig/init@/var/log@gi


			练习：
				1、复制/etc/grub.conf至/tmp目录，删除/tmp/grub.conf文件中的行首的空白字符；
				%s@^[[:space:]]\{1,\}@@g

				2、复制/etc/rc.d/rc.sysinit至/tmp目录，将/tmp/rc.sysinit文件中的以至少一个空白字符开头的行的行首加#号；
				%s@^\([[:space:]]\{1,\}.*\)@#\1@

				%s@^[[:space:]]\{1,\}.*@#&@

				3、删除/tmp/rc.sysinit文件中以#开头，且后面跟了至少一个空白字符的行的行首的#号和空白字符；
				%s@^#[[:space:]]\{1,\}@@

				4、为/tmp/grub.conf文件中前三行的行首加#号；
				1,3s@^@#@

				5、将/etc/yum.repos.d/CentOS-Media.repo文件中的所有enable=0和gpgcheck=0两行最后的0改为1；
				%s@enable=0@enable=1@
				%s@\(enable\|gpgcheck\)=0@\1=1@g
				

#### bash测试：				
			整型测试：
				-gt: 例如 [ $num1 -gt $num2 ]
				-lt: 
				-ge: 
				-le:
				-eq:
				-ne:

			字符串测试：
				双目
					>: [[ "$str1" > "$str2" ]]
					<:
					>=
					<=
					==
					!=

				单目：
				  	-n String: 是否不空，不空则为真，空则为假
				  	-z String: 是否为空，空则为真，不空则假


          -a FILE
          -e FILE: 存在则为真；否则则为假；

          -f FILE: 存在并且为普通文件，则为真；否则为假；
          -d FILE: 存在并且为目录文件，则为真；否则为假；
          -L/-h FILE: 存在并且为符号链接文件，则为真；否则为假；
          -b: 块设备
          -c: 字符设备
          -S: 套接字文件
          -p: 命名管道

          -s FILE: 存在并且为非空文件则为值，否则为假；

          -r FILE
          -w FILE
          -x FILE

          file1 -nt file2: file1的mtime新于file2则为真，否则为假；
          file1 -ot file2：file1的mtime旧于file2则为真，否则为假；



        组合条件查找：
			与：-a, 同时满足
			或：-o, 满足一个即可
			非：-not, !，条件取反

			-not A -a -not B = -not (A -o B)
			-not A -o -not B = -not (A -a B)
			
			
		-type TYPE: 根据文件类型查找
			f: 普通文件
			d: 目录文件
			l: 符号链接
			b: 块设备
			c: 字符设备
			s: 套接字文件
			p: 命名管道
			
			
	   根据时间戳查找：
			以“天”为单位
				-atime [+|-]#
					+#：x >= #+1
					-#：x < #
					#: # <= x < #+1 
				-mtime
				-ctime

			以“分钟”为单位 
				-amin
				-mmin
				-cmin

		根据权限查找：
			-perm [+|-]MODE
				MODE: 与MODE精确匹配
					find ./ -perm 644
				+MODE: 任何一类用户的权限只要能包含对其指定的任何一位权限即可；以属主为例，
					find ./ -perm +222	
				-MODE：每类用户指定的检查权限都匹配：	
					为三类用户所有指定的检查权限都能够被包含
					find ./ -perm -222

    练习：
    1、查找/var/目录属主为root且属组为mail的所有文件；
    # find /var -user root -a -group mail
    
    2、查找/usr目录下不属于root、bin或hadoop的所用文件；
    find /usr -not -user root -a -not -user bin -a -not -user hadoop
    find /usr -not \(-user root -o -user bin -o -user hadoop\)
    
    3、查找/etc/目录下最近一周内其内容修改过的，且不属于root且不属于hadoop的文件；
    find /etc -mtime -7 -a -not \(-user root -o -user hadoop\)
    
    4、查找当前系统上没有属主或属组，且最近1个月内曾被访问过的文件；
    find / \(-nouser -o -nogroup\) -a -atime -30
    
    5、查找/etc/目录下大于1M且类型为普通文件的所有文件；
    find /etc -size +1M -type f
    
    6、查找/etc/目录所有用户都没有写权限的文件；
    find /etc/ -not -perm +222
    
    7、查找/etc/目录下至少有一类用户没有写权限；
    find /etc/ -not -perm -222
    
    8、查找/etc/init.d/目录下，所有用户都有执行权限且其它用户有写权限的文件；
    find /etc/init.d/ -perm -113 


#### 特殊权限：
    
    u 文件属主权限
    g 同组用户权限
    o 其它用户权限
    a 所有用户（包括以上三种）

	mode: 
		ls -l

	安全上下文
		1、进程以某用户身份运行；进程是发起此进程的用户的代理，其用户身份进程发起者；
		2、权限匹配模型：
			(1) 进程的属主，是否与被访问的文件属主相同；
			(2) 进程的属主所属于的组当中，是否有一个与被访问的文件的属组相同；
			(3) 以other的权限进行访问；

	suid: Set UID
		前提：此类文件为有可执行权限的命令
		任何用户运行此命令为一个进程时，此进程的有效身份不是发起者，而是命令文件自身的属主；

		chmod u+s FILE...
			使用ls -l查看时，此s可能显示为大写或小写两种形式之一；
				属主原有执行权限时，显示为小写；

	sgid: Set GID
		前提：
			常用方法：如果将目录的属组设置SGID权限之后，所有用户在此目录创建文件的属组不再是用户的基本组，而是目录的属组

		chmod g+s FILE...

	有那么一个目录：
		指定的用户都能够在其中创建文件，也能删除自己的文件；但不能删除别人的文件；

	sticky: 沾滞位

		chmod o+t FILE...

	suidsgidsticky
	000: 
	001: 
	010
	011
	100
	101
	110
	111

	chmod 7755 

	练习：
		1、让普通用户使用/tmp/cat能查看管理员才有权限访问的文件；
    		[root@server ~]#cp `which cat`/tmp/cat
            [root@server ~]#chmod u+s /tmp/cat
            [root@server ~]#su - centos
            [centos@server ~]#/tmp/cat /etc/shadow #可以看到/etc/shadow的内容
		2、新建目录/project/test，让普通用户hadoop和openstack对此目录都能创建文件，且创建的文件的属组为此目录的属组，而非用户自身的属组，此外还要求，每个用户不能删除其它人的文件；
    		[root@server ~]#mkdir -p /project/test
            [root@server ~]#chmod g+s /project/test
            [root@server ~]#chmod o+t /project/test



#### 计划任务
所谓的计划任务是在未来某一特定的时间执行一次或多次特定的作业（任务）。实现无需人工干预的情况下执行作业。
    
        在命令行下：
    at TIME
    at> COMMAND1
    at> COMMAND2
    ...
    at> 
    Ctrl+d：提交任务
    
    一次性计划任务常用的命令有at,batch。
    在CentOS中通过/etc/init.d/atd start 命令来启动此服务。    
    at是通过作业队列来管理的，通过at后者at -l命令查看作业队列
    删除不想执行的任务 at -d 作业ID 或者 atrm 作业ID
    可以通过/etc/at.allow和/etc/at.deny  2个文件来控制用户是否可以使用at来设置一次性任务。
    
    batch功能同at，但相比于at而言，无需指定其特定的时间。系统会选择系统资源比较空闲的时候执行指定的作业。
    
    
     周期性任务是在未来某一特定的时间内循环执行特定的作业。是通过cron机制来完成相应功能的。是由后台进程crond一直在监控。
     
    通过“/etc/init.d/crond status”命令语句来查看crond的状态，如果状态是“is stopped.”，需要启动此服务，执行“/etc/init.d/crond start;chkconfig crond on”。
    /etc/init.d/crond status    查看crond的状态
    /etc/init.d/crond stop    关闭crond服务
    /etc/init.d/crond start    打开crond服务
    
    cron任务分类
    根据cron任务的执行者不同，将cron分为两类：
    系统cron
        定义在/etc/crontab中，每行定义一个独立的任务，这是针对于系统级的。
    用户cron
        定义在/var/spool/cron目录中，每个用户都有一个与用户名同名的文件，其功能类似于/etc/crontab，每行定义一个独立的任务。这是针对于用户级别的。
    2、相关目录和文件
    /etc/cron.hourly/ 每个小时要执行的脚本的目录
    /ect/cron.daily/  每个天要执行的脚本的目录
    /etc/cron.monthly/  每个月要执行的脚本的目录
    /etc/cron.weekly/   每个周要执行的脚本的目录
    /etc/cron.deny   只有cron.allow文件中列出的用户才能使用cron服务，同时忽略cron.deny文件
    /etc/cron.allow 如果cron.allow文件不存在，cron.deny文件中列出的用户将被禁止使用cron服务
    
    环境变量：
    SHELL=/bin/bash #默认shell解释器
    PATH=/sbin:/bin:/usr/sbin:/usr/bin #默认环境PATH的路径，所以在设定周期性任务的话有时要写全脚本的路径
    MAILTO=root #如果出现错误，或者有数据输出，数据作为邮件发给这个帐号
    HOME=/ #默认的家目录
        
    # .---------------- minute (0 - 59)
    # |  .------------- hour (0 - 23)
    # |  |  .---------- day of month (1 - 31)
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * [user-name] command to be executed
    
    #主要有3部分组成，第一部分 * * * * * 所代表的时间是：minute hour day month day
    # * * * * *   每分钟 
    # 2 */2 * * * 每隔2小时  */#:在对应的时间位的有效取值上每#一次
    # 3 4 * * 1,3,5 每周1,3,5    ','代表离散取值
    # 3 9-17 * * * 每天9-17'-'某个时间位上的连续区间
    #注意：在设置时间是，比它小单位一般都是确定的
    
    #第二部分是：user-name 用户名,这一部分在编辑/etc/crontab设定周期性任务时，需要指定用户名。其他两中方式不需要
    
    #第三部分是 command to be executed 可执行的命令或者是脚本。如果有run-parts关键字，后面是脚本所在的目录
    #1 * * * * root run-parts /etc/cron.hourly/


        设定周期性任务的方式有三种：
    1）通过crontab命令
        crontab [-u USER_NAME] -e
        此时会打开一个编辑器，按照相应的格式就可以设定。
    2）通过编辑/var/soop/USER_NAME文件设定
    3）通过编辑/etc/crontab文件设定
    4、丢弃邮件通知：
        使用输出重定向：
        &>/dev/null 
        >/dev/null
    5、crontab命令其他常用用法
        -l: 列出已经定义的所有任务
        -e: 打开编辑界面定义任务
        -r: 移除所有任务 如果是要删除某一条特定的任务，可以通过设定周期任务的三种方式来删除对应任务所在的那一行。
    6、其他
        anacron: crontab的补充机制
    	检查有没有过去一个有效周期未曾执行的任务，如果有，在开机后的指定时间点执行一次；
        实现秒级别的任务：
    	* * * * * for i in {1..5};do 脚本;sleep 10;done (每隔10秒)
    7、练习
        1、每月执行一次对/目录的备份，备份至/backup目录中，保存的目录名格式为rootfs-2014-07；
            # 0 4 1 * * [ -d /backup ] || mkdir /backup; /bin/cp -a / /backup/etc-$(date +'%Y-%m')
        2、每周3,5,7备份/proc/cpuinfo文件至/backup/cpu.info/目录中，保存的文件名为cpu-2013081009；
            # 3 1 * * 3,5,7 [ -d /backup/cpu.info/ ] || mkdir -p /backup/cpu.info/; /bin/cp -a /proc/cpuinfo /backup/cpu.info/cpu-$(date +'%Y%m%d%H')
        3、每天每两小时取当前系统/proc/meminfo中的以S开头的信息保存至/stats/memory.txt中
            # 2 */2 * * * grep -i "^S" /proc/meminfo >> /stats/memory.txt 
        4、工作日的工作时间内，每小时执行一次'echo "hello"'
            # 1 9-17 * * 1-5 echo "hello"
     
####     acl:       
    某大牛在QQ群内直播讲解Linux系统的权限管理，
    讲解完之后，他在一个公有的Linux系统中创建了一个 /project 目录，
    里面存放的是课后参考资料。那么 /project 目录对于大牛而言是所有者，
    拥有读写可执行（rwx）权限，
    对于QQ群内的所有用户他们都分配的一个所属组里面，
    也都拥有读写可执行（rwx）权限，
    而对于 QQ 群外的其他人，
    那么我们不给他访问/project 目录的任何权限，
    那么 /project 目录的所有者和所属组权限都是（rwx），
    其他人权限无。
    　　问题来了，这时候直播有旁听的人参与（不属于QQ群内），
    　　听完之后，我们允许他访问/project目录查看参考资料
    　　，但是不能进行修改，也就是拥有（r-x）的权限，
    　　这时候我们该怎么办呢？我们知道一个文件只能有一个所属组
    　　，我们将他分配到QQ群所在的所属组内，
    　　那么他拥有了写的权限，这是不被允许的；
    　　如果将这个旁听的人视为目录/project 的其他人，
    　　并且将/project目录的其他人权限改为（r-x），
    　　那么不是旁听的人也能访问我们/project目录了，
    　　这显然也是不被允许的。怎么解决呢？
    　　
    　　
    　　查看分区 ACL 权限是否开启：dump2fs
    　　查看当前系统有哪些分区：df -h
    　　查看指定分区详细文件信息：dumpe2fs -h 分区路径
    　　临时开启分区 ACL 权限 mount -o remount,acl /
    　　永久开启分区 ACL 权限:
    　　                        修改配置文件 /etc/fstab
    　　                       UUID=490ed737-f8cf-46a6-ac4b-b7735b79fc63 /                       ext4    defaults,acl        1 1
    　　                       
    　　                       重新挂载文件系统或重启系统，使得修改生效  mount -o remount /
    　　                       
    　　                       
        设定 ACL 权限：setfacl 选项 文件名
![image](A798BAD810104D9985E39F95D14D33F2)

    　　给用户设定 ACL 权限：setfacl -m u:用户名:权限 指定文件名
    　　给用户组设定 ACL 权限:setfacl -m g:组名:权限 指定文件名
    　范例：所有者root用户在根目录下创建一个文件目录/project，然后创建一个QQ群所属组，所属组里面创建两个用户zhangsan和lisi。所有者和所属组权限和其他人权限是770。
    　　　　　然后创建一个旁听用户 pt，给他设定/project目录的 ACL 为 r-x。　
![image](95110D5072FC4DEEBA1F7D2B0D524018)
    目录 /project 的所有者和所属组其他人权限设定为 770。接下来我们创建旁听用户 pt，并赋予 acl 权限 rx
![image](BE5CC5D570BB4BB9A217E4491409A14C)

    　为了验证 pt 用户对于 /project 目录没有写权限
    　，我们用 su 命令切换到 pt 用户，
    　然后进入 /project 目录，在此目录下创建文件
    　，看是否能成功：
    　上面提示权限不够，说明 acl 权限赋予成功
    　如果某个目录或文件下有 + 标志，
    　说明其具有 acl 权限。
    　
    　查看 ACL 权限：getfacl 文件名
    　
    　
    　最大有效权限 mask
    　我们给用户或用户组设定 ACL 权限其实并不是真正我们设定的权限
    　，是与 mask 的权限“相与”之后的权限才是用户的真正权限
    　，一般默认mask权限都是rwx
    　，与我们所设定的权限相与就是我们设定的权限。
    　我们通过 getfacl 文件名 也能查看 mask 的权限，那么我们怎么设置呢？
    　setfacl -m m:权限 文件名
    　
    删除ACL权限：
        删除指定用户的 ACL 权限 setfacl -x u:用户名 文件名
        删除指定用户组的 ACL 权限 setfacl -x g:组名 文件名
        删除文件的所有 ACL 权限 setfacl -b 文件名
        
    递归 ACL 权限：
        　通过加上选项 -R 递归设定文件的 ACL 权限，
        　所有的子目录和子文件也会拥有相同的 ACL 权限。
        　setfacl -m u:用户名:权限 -R 文件名   
        　
        　

#### Linux文件系统

    1、什么是文件系统
        广义上来说，文件系统是对存储设备的数据和元数据的进行管理或者说组织的一种机制。文件系统类型是这种机制的不同管理方式。
    2、存储空间的组成
        存储空间一般是由数据区 和 元数据区组成的，一般元数据区存放文件的元数据，每个文件都有自己的元数据，这些元数据存放在inode（index node）中,为了方便查找文件。
        inode用于存储文件的各种属性：
                            文件的大小
                            文件名
                            属主，属组
                            时间戳
                            权限
                            以及对应在磁盘上的块号（对应block的位置信息）
    每个文件中的数据，存放在数据区，由block管理。block用于存储文件内容。
    当创建一个目录时，文件系统会为该目录分配一个inode和至少一个block。
    该inode记录该目录的属性，并指向那块block。
    该block记录该目录下相关联的文件或目录的关联性和名字。   
    当创建一个文件时，
    文件系统会为该文件分配至少一个inode和与该文件大小相对应的数量的block。
    该inode记录该文件的属性，并指向block。   如果一个目录中的文件数太多，
    以至于1个block容纳不下这么多文件时，
    Linux的文件系统会为该目录分配更多的block。
    Dentry 是将 Inode 和 文件联系在一起的”粘合剂”
    它将 Inode number 和文件名联系起来
    Dentry 也在目录缓存中扮演了一定的角色，
    它缓存最常使用的文件以便于更快速的访问。
    Dentry 还保存了目录及其子对象的关系，
    用于文件系统的遍历。    
    
    
    文件系统结构：
![wKioL1PCFlPyXmDlAADIR8YN2KA552](3C41FE58A2814B0B81BFDC679113886F)

    superblock: 是记录元数据的元数据。查看超级块信息：tune2fs -l DEVICE;dumpe2fs -h DEVICE
    GDT:块组的描述信息，记录何处开始记录数据
    Block bitmap:记录那些Block是空闲的
    Inode bitmap:记录那些Inode是空闲的
    Inode table:存放Inode数据，里面有好多的inode数据。
    Data blocks:存放数据
    
    文件系统的类型以及表示方法：
    
    1）文件系统的类型
        常见的有：ext2（不支持日志）,ext3,ext4, xfs, ffs, ufs, reiserfs,
        jfs, vfat(fat32), ntfs
    2）表示方法
    a.磁盘设备文件
    磁盘设备类型
        /dev/hd:磁盘接口是IDE类型的用这种方法表示
            IDE: 并口, 133MB/s
        /dev/sd:磁盘接口是SATA类型的用这种方法表示
            USB: 串行
            SATA: 串行，6Gbps/8
            SCSI: 并行，(Small Computer System Interface)
            SAS：串行
        在centos6系列中，不管是IDE类还是SATA类都是以/dev/sd这种方式表示
    表示磁盘类型下的不同设备
        /dev/sd[a-z]：用字母a-z表示不同的设备
     分区：数字表示不同的分区
        /dev/sda1 
        /dev/sda2
        分区编号：主分区、扩展分区编号1-4，逻辑分区是>=5的
    b.特殊设备文件
        特殊设备文件是只有Inode，没有Block数据的文件
        （此时对应的就是真正的物理设备文件）。
        在Inode中关联的是硬件的渠道程序，
        通过驱动程序和硬件相关联。
        特殊文件通过设备号来标识这些文件。
        主设备号来标识设备类型，
        次设备号来标识相同设备下的不同设备。

    我们可以手动添加这样的设备文件：
    mknod: make block or character special files(man 文档的解释)
       Usage:mknod [OPTION]... NAME TYPE [MAJOR MINOR]   
    例如：mknod /dev/deviceloop b 12 4

    三、磁盘的分区，格式化，挂载，卸载，磁盘修复
    分区
        在Linux中分区是针对柱面的。在Linux中主分区和扩展分区最多只能有四个，
        可以再扩展分区上创建逻辑分区，
        最多只能有64个逻辑分区。这是由MBR分区决定的。
        常用的分区工具有fdisk,partd等。
        这里我们介绍fdisk工具。
            Usage: fdisk [option]... [DEVICE]...
        查看一块磁盘的分区信息,例如：fdisk -l /dev/sda


      给一块磁盘分区，例如：fdisk /dev/sdb。下面是fdisk的菜单栏，对常用的做了解释。
![wKiom1PCLBWw59yrAAKYs9IH5XU923](FFC071B221D54D2886DDB69929F3C94C)


    创建分区过程，例如创建一个主分区号位1的主分区，大小为500M。
![wKiom1PCLmPg6dLzAAMCHYAVehs622](80C5756CCB884A4B8DE5A0E4C73A7F54)


    下面简单介绍一下parted分区工具： 
    有时我们在写脚本时，需要非交互式的对磁盘进行分区。
    parted是一个不错的分区工具。它是一个实时的分区工具，
    不像fdisk，需要w才能保存。
    Usage: parted [OPTION]... [DEVICE [COMMAND [PARAMETERS]...]...]    
    最常用的选项：-s | --script：( never prompts for user intervention),实现非交互方式
    常用的COMMAND：mkpart PART-TYPE [FS-TYPE] START END
        PART_TYPE的取值有primary，logical，extended
        START:1,1G 默认是M
        END:12G
    rm NUMBER delete partition NUMBER 删除一个分区
    print [devices|free|list,all|NUMBER]   显示分区信息
    
    
```
####下面是一些常用的操作
[root@server scripts]# parted -s /dev/sdb print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End    Size   Type     File system  Flags
[root@server scripts]# parted -s /dev/sdb mkpart primary 1 512
[root@server scripts]# parted -s /dev/sdb mkpart extended 512  10G
[root@server scripts]# parted -s /dev/sdb mkpart logical 512 1G
[root@server scripts]# parted -s /dev/sdb print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  512MB   511MB   primary
 2      512MB   10.0GB  9489MB  extended               lba
 5      512MB   1000MB  488MB   logical
[root@server scripts]# parted -s /dev/sdb rm 1
[root@server scripts]# parted -s /dev/sdb print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start  End     Size    Type      File system  Flags
 2      512MB  10.0GB  9489MB  extended               lba
 5      512MB  1000MB  488MB   logical

[root@server scripts]# parted -s /dev/sdb rm 2
[root@server scripts]# parted -s /dev/sdb print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start  End  Size  Type  File system  Flags

```

    格式化
    格式化就是在分好的区区上创建文件系统。常见的格式化工具有mkfs，mke2fs工具。
    mkfs工具使用起来简单，而mke2fs工具可以进行比较详细的设置。
    1）mkfs
        mkfs : make file system，创建文件系统
            Usage: mkfs -t FSTYPE [DEVICE] 等价 mkfs -t FSTYPE = mkfs.FSTYPE
            例如：mkfs -t ext4 = mkfs.ext4
        注意：有些文件系统Linux内核是支持的，但是我们创建的时候却没有这种文件系统。
        这是由于Linux的发行版没有将这些功能编译到内核中去。由于Linux内核是模块化的，
        而且支持动态编译。如果我们使用某一模块的话，
        可以手动添加进去。
        使用lsmod可以查看内核编译进去的模块。
        以CentOS6为例，发行版不支持xfs文件系统，
        此时我们可以讲安装这个模块的一个管理工具。
        # yum -y install xfsprogs 
    2）mke2fs
        mke2fs是ext系列文件系统的管理工具，
        但实际上也可以管理其他的文件类型。
        mkfs的常用参数

    -t
    -指定文件系统的类型
    例如：mke2fs -t ext4 /dev/sdb1
    
    -b
    指定Block的大小，一般是1024,2048,4096
    例如：mke2fs -t ext4 -b 1024 /dev/sdb1
    
    -L
    指定分区的卷标（别名）
    例如：mke2fs -t ext4 -L DATA /dev/sdb1
    相当于：e2label /dev/sdb1 DATA
    
    -j
    支持日志，针对那些没有日志的文件系统
    例如：mke2fs -t ext2 -j /dev/sdb1
    
    -i
    定义多上个字节创建一个Inode,次字节数应大于Bock的字节数
    例如：mke2fs -t ext4 -b 1024 -i 2048 /dev/sdb1
    
    -N
    指定可用的节点数
    
    -m
    预留百分比，默认是5
    为什么要预留呢？这是为了方便管理员的一些操作。
    
    -O
    -指定分区特性
    
    -U
    设定磁盘的UUID
    这个信息通过 blkid [-L LABEL] DEVICE查看
    
    [root@server ~]# mke2fs -t ext4  -b 2048 -m 3 -L DATA /dev/sdb1
    [root@server ~]# blkid /dev/sdb1
    此格式化完毕。
    但是我们在格式化完毕后，发现某些属性忘记添加了。
    可以使用tune2fs命令来修改。
        Usage：tune2fs [options].. [DEVICE]...
          常用参数：
        -j ：添加日志
        -L LABEL修改卷标
        -m 修改预留百分比
        -O [^]FEATURE: 启用指定特性，特性前加^，表示关闭此种特性
        -o [^]mount-options: 开启或关闭指定的挂载选项
    
    
    挂载
    挂载的目的是让已经格式化好的分区和某个目录相关联，
    提供访问磁盘分区的入口。
    长使用的挂载命令式mount。
        Usage: mount [options]... DEVICE MOUNT_POINT
    这里对挂载点的基本要求是：必须是目录
    ；一般使用空目录作为挂载点。
    常用参数：
            -t
            -t 指定挂载文件系统类型
            mount -t ext4 -w /dev/sdb1 /mnt
           
            -r
            -r 只读挂载
            
            -w
            -w 读写挂载
            
            -L
            -L 卷标挂载（使用卷标来确定设备）
            mount -t ext4 -L DATA  /mnt
           
            -U
            -U UUID挂载（使用UUID来确定设备）
            
            -a
            -a 自动挂载,读取/etc/fstab文件
            mount -a -n
           
            -n
            -no-mtab 不更新/etc/mtab文件
           
           
            -o 
            -o optoin指定额外属性，多个属性用逗号隔开
            ，如果不指定，则使用默认属性。
            rw, suid, dev, exec, auto, nouser, async, and relatime
            
            async：异步I/O，数据写操作先于内存完成，而后再根据某种策略同步至持久设备中
            sync: 同步I/O，数据写操作完成后立即同步到磁盘中
            
            atime/noatime: 文件和目录被访问时是更新最近一次的访问时间戳
            
            auto/noauto：设备是否支持mount的-a选项自动挂载
            
            diratime/nodiratime: 目录被访问时是更新最近一次的访问时间戳
            
            dev/nodev: 是否支持在此设备上使用设备
            
            exec/noexec: 是否允许执行此设备上的二进制程序文件
            
            suid/nosuid: 是否支持在此设备的文件上使用suid
            
            remount: 重新挂载，通常用于不卸载的情况下重新指定挂载选项
            
            ro: 只读
            rw: 读写
            
            user/nouser: 是否允许普通挂载此文件设备
            
            acl: 在此设备是支持使用facl，默认不支持
            
            mount -t ext4 -o sync,acl,suid,noexec /dev/sdb1 /mnt
            
    挂载情况：
![wKiom1PCQyOhLEXxAALq2uScc84679](C06E05DC95294B6F8399304432F8FC84)

     至此，挂载已完成。
        但是，这些挂载内容在下次重新开机之后，就不在了。所以，为了下次开机之后还能挂载到，我们要把这些信息保存到配置文件/etc/fstab。
        
![wKioL1PCS2TBuyJnAAHBXqLbrWw808](E7CC10341D1E492FA649EEDE61152906)

    注意：如果挂载点目录不是空文件时，它原先在里面的文件就会被隐藏起来。
    
    卸载
    当不需要使用这个分区的时候，需要将这个磁盘分区卸载。
    使用 umonut DEVICE 或者 umont DEVICE_POINT 来卸载。
    但是，在卸载的时候我们刚好在访问这个目录，
    那么会提示设备忙，拒绝退出。
    此时，有2种解决方案：1、退出此目录。2、使用fuser命令强制退出。
    
![wKioL1PCR63j-NV7AAHUZjo_gOU304](A4B3D278C1C7462F92ED06BB1A3C47C1)

    磁盘检查
    什么情况下会进行磁盘检查呢？在出现意外情况下，比如说断电，导致了正在写入磁盘的数据非正常退出。使用fsck 或者 e2fsck工具来检查。
    fsck：check and repair a Linux file system
        -t fstype
        -a: 自动修复错误
        -r: 交互式修复错误
    e2fsck: 专用于检测和修复ext系列文件系统
        -y: 对问题自动回答为yes 
        -f: 强制进行检测
        
    swap分区
    swap是交换分区，一般情况下是在内存不够的情况下，
    启用的一种机制。是将外存作为在特殊情况下当做内存使用，
    一般情况下模式内存的2倍。
    ########第一种情况：使用一个分区作为交换分区###########
    
    #第一步：添加一个交换分区
    [root@server ~]# fdisk -l /dev/sdb | grep "swap"
    /dev/sdb2              66         197     1060290   82  Linux swap / Solaris 
    #第二步：将这个分区格式化为swap
    [root@server ~]# mkswap /dev/sdb2
    Setting up swapspace version 1, size = 1060284 KiB
    no label, UUID=ccf2f05f-fcf8-46ca-a304-d759887785d1
    #第三步：挂载使用
    [root@server ~]# swapon /dev/sdb2
    [root@server ~]# swapon -s
    Filename				Type		Size	Used	Priority
    /dev/dm-1                               partition	2097144	0	-1
    /dev/sdb2                               partition	1060280	0	-2
    
    ########第二种情况：使用一个文件作为交换分区###########
    #第一步：添加一个作为交换分区的文件
    [root@server ~]# dd if=/dev/zero of=./myswap bs=1M count=1024
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 50.1967 s, 21.4 MB/s
    #第二步：将这文件区格式化为swap
    [root@server ~]# mkswap -f ~/myswap 
    Setting up swapspace version 1, size = 1048572 KiB
    no label, UUID=f818dd1c-395f-4da2-bf5e-1edb1483100e
    #第三步：挂载使用
    [root@server ~]# swapon ./myswap 
    [root@server ~]# swapon -s
    Filename				Type		Size	Used	Priority
    /dev/dm-1                               partition	2097144	0	-1
    /root/myswap                            file		1048568	0	-2
    
    ###卸载使用 swapoff [DEVICE] 就可以。当然，要想在下次开机时加载，必须将这些写入到/etc/fstab 文件中去。

#### linux下磁盘管理机制--RAID
      RAID(Redundant Array Of Independent Disks)：独立磁盘冗余阵列。
      RAID的最初出现的目的是为了解决中小型企业因经费原因使用不起SCSCI硬盘，
      而不得不使用像IDE较廉价的磁盘情况下，
      将多块IDE磁盘通过某种机制组合起来，
      使得IDE磁盘在一定程度上提高读写性能的一种机制。
      当然，现在也可以将SCSCI类的磁盘也可以做成RAID来提高磁盘的读写性能。
      
      RAID的级别
        RAID机制通过级别来RAID级别来定义磁盘的组合方式。
        常见的级别有：RAID0,RAID1,RAID4,RAID5, RAID6这几种级别，下面分别叙述。
        
        RAID0：最少使用2块磁盘，
        出现的目的是为了提高读写性能，
        磁盘利用率达到100%，但没有容错能力，
        磁盘损坏的几率也增加了一倍。
        
        RAID1：最少使用2块磁盘，常作为磁盘的镜像使用，
        所以一般情况下，自己2块磁盘。
        设备利用率一般情况下是50%，
        允许一块磁盘损坏，读性能提高，写性能有所下降。
        
        RAID4：最少使用3块磁盘，其中一块专门保存其他2块数据的备份。
        通常情况是：保存其他2块磁盘数据按位异或的结果。
        设备利用率是 n-1/n (n为磁盘数目)，可以损坏一块磁盘，
        读写性能都有相应的提升。
        
        RAID5：同RAID4一样，不同的一点是，RAID5将备份信息分散到每个磁盘上。
        
        RAID6：在RAID5的基础上，将备份信息多存了一份。
        所以，它可以允许2块磁盘损坏，
        其他的同RAID5一样。
![wKioL1PF-0rSxT6UAACJ8_2lCE0899](DBF99FDCCE8E49D1BCC8461287FE0451)
          
          
         
        我们也可以使用级别组合，
        常用的有RAID10,RAID50,RAID60等。
        以RAID10为例：假设现在有6块磁盘，
        可以先每2块作为一个RAID1,
        然后将3组RAID1作为一个RAID0，
        来提高磁盘的读写性能和容错能力。
          
        二、RAID的实现
    RAID实现方式有2种:
    硬件RAID
    由硬件控制器来实现，
    在操作系统中看到的仅是一个单独的设备。
    企业一般都使用硬件RAID来实现。
    软件RAID
    由软件模拟硬件RAID的控制器来实现，依赖于操作系统。
    
    下面通过软件RAID的实现，来理解RAID的实现方式。
        在Linux中使用mdadm这个命令来实现。
        具体在内核中是由 md（multdisk）模块实现的。
        下面介绍mdadm的常用参数：
    -C
    创建RAID,还有子选项
    -n: 用于创建RAID的磁盘数目
    -l: 创建RAID级别
    -c: chunk的大小，chunk就是RAID的数据块大小，它和磁盘中的Block的大小不同
    -a: {yes|no}是否自动创建RAID设备文件
    mdadm -C /dev/md0 -l 5 -n 3 -a yes -c 1024K /dev/sdb{5,6,7}
    
    
    -A
    重新装载,比如在操作系统崩溃之后，但磁盘还在，在里一个操作系统里可以使用此选项重新加载
    mdadm -A /dev/md1 /dev/sdb{5,6,7}
    
    
    -S
    停用RAID
    mdadm -S /dev/md0
    
    -D
    显示RAID的详细信息
    mdadm -D /dev/md0
    
    -r
    从RAID设备里移除一块磁盘（分区）
    mdadm /dev/md0 -r /dev/sdb5
    
    -a
    往RAID设备里添加一块磁盘（分区）
    mdadm /dev/md0 -a /dev/sdb5
    
    -f
    手动损坏一块磁盘
    mdadm /dev/md0 -f /dev/sdb5
    
    
    正式开始创建RAID，我们这里示例创建RAID5:
    第一步准备磁盘，我们这里用分区代替：
![wKioL1PGFmbgha27AAIWzIhozj8061](81FC3D23292144A7BDAB1FC96B5ECD49)    
    第二步：就是创建RAID5。
![wKiom1PGGj-xOI_TAAFFr5B0gBU083](6DDF0D13985C4EE9813D91A1D11133F3)  
    当看到以下信息时，说明数据同步完毕。
![untitled](6C2C1E741EF245EF9670CE6628169C65) 
    下面就可以格式化此设备，并挂载使用了。
![wKiom1PGG5Piu7XKAAEB3-9S6VE146](A45C41E19166432DA8F3582E08CB06DD)
    查看RAID的详细信息：
![wKioL1PGHm3h4DAFAALoWC2klHE406](99791F8802FA4605959601B4C421E69B)
          
    磁盘故障实验
        我们手动损坏一块磁盘，数据是否正常。
        
![wKiom1PGHrfjasJpAALBFFTQrmk324](1A94F429C5204C298A699DC521372CEF)

        此时我们依然可以正常访问数据，正常读写数据。
![wKioL1PGIFyghK0AAADKK5nKO_w817](F7A2650A84F241B0A6CF17BBDE4F756F) 

        此时我们依然可以正常访问数据，正常读写数据。
![wKiom1PGIFGSkr37AALJ_rR6aX0993](818335C862154F16B0BFB3E0440B7128)

        但是，我们经常是这样做的，在没损坏的情况下，添加一块磁盘作为热备磁盘。
![wKioL1PGIt3zF_1GAAMAxI9RXQM241](37D612D1C22047E89703F7EA10B7773D)

        当有一块磁盘损坏可以立即顶上去。
![wKiom1PGIiuTPVzvAAMG_33BIkA997](47DDB0786B07436E8F1AF862EBFA2519)
        
        
        
    磁盘重新装载
        我们这里使用 mdadm -S RAID_DEVICE 来模拟操作系统崩溃的情况，然后重新装载。
        
![wKioL1PGJOTxQwR9AAJYpBo8-_o376](973CA3E6430E4C2D8260310BB032B3CB)

          此时，我们挂载使用，数据依然存在。
![wKioL1PGJXaTGm6oAAB40vGr_5I947](649DFB29D80D4CD6AC3332D63B14FA29)



    read命令是用于从终端或者文件中读取输入的内部命令，
    read命令读取整行输入，每行末尾的换行符不被读入。
    在read命令后面，如果没有指定变量名，
    读取的数据将被自动赋值给特定的变量REPLY。
    ​ 
    从标准输入读取输入并赋值给变量。
    read [var]
    例如:
    where@ubuntu:~$ read var
    wenong
    where@ubuntu:~$ echo $var
    wenong
    
    从标准输入读取输入到第一个空格或者回车，
    将输入的第一个单词放到变量中，
    第二个单词放第二个变量中，以此类推，
    剩下的字符留给最后一个变量。
    
    read [var1] [var2] ...
    例如：
    where@ubuntu:~$ read var1 var2 var3
    1 2 3 4 5 6
    where@ubuntu:~$ echo $var1
    1
    where@ubuntu:~$ echo $var2
    2
    where@ubuntu:~$ echo $var3
    3 4 5 6
    从标准输入读取一行并赋值给特定变量REPLY。
    例如：
    readwhere@ubuntu:~$ read 
    hello
    where@ubuntu:~$ echo $REPLY
    hello
    where@ubuntu:~$ 
    
    
    把单词清单读入数组里
    read -a [arrayname]
    例如：
    where@ubuntu:~$ read -a array
    1 2 3 4 5
    where@ubuntu:~$ echo ${array[2]}
    3
    
    打印提示，等待输入
    read -p [info] [var]
    例如：
    where@ubuntu:~$ read -p "what is your name?" name
    what is your name?wenong
    where@ubuntu:~$ echo $name
    wenong
    读超时
    read -t [timeout] [var]
    例如：
    where@ubuntu:~$ read -t 3 var
    where@ubuntu:~$                 #3秒后退出read命令
    
    读取指定个数字符
    read -n [size] [var]
    例如：
    where@ubuntu:~$ read -n 2 var
    dkwhere@ubuntu:~$ echo $var      #输入2个字符后，read命令自动退出。
    dk
    where@ubuntu:~$        
    
    自定义结束输入行
    read -d [char] [var]
    例如：
    where@ubuntu:~$ read -d ':' var
    huang:where@ubuntu:~$ echo $var  #输入：后read自动退出。
    huang
    where@ubuntu:~$ 
    隐藏输入字符
    read -s [var]
    例如：
    where@ubuntu:~$ read -s var
    where@ubuntu:~$ echo $var
    wenong
    where@ubuntu:~$ 
    
#### linux下磁盘管理机制--LVM
    
    当我们用传统分区方法使用磁盘时，
    当出现分区大小不够用的时候，
    通常只能添加添加一个更大的磁盘，
    重新创建分区来扩展空间。但是，
    这样只能是将原来的磁盘下线，换上新的磁盘，
    在将原始数据写入，在实际的生产过程中是不允许的。
    此时就需要使用逻辑卷LVM这种磁盘分区管理了。
    逻辑卷是将硬盘空间重新“分割”成大小相等的块（PE）
    组成的PV放到一个容器（VG）中，
    当需要可以随时向这个容器中取出这样的块，
    来实现动态调整磁盘空间大小。
    当然新添加的块不会改变原来的文件系统，
    而且原磁盘也不用下线。
    
    
    下面说明逻辑卷的基本组成部分和各部分之间的关系：
            PE(physical extend):物理扩展 
            PV(physical volume):物理卷
            VG(volume group):卷组 
            LV(logical volume):逻辑卷 
            
    下图说明了PE,PV,VG，LV之间的关系。
    LVM的基本单位是PE，PV是将每隔磁盘（分区）格式化成大小相等的PE,
    VG的作用是经PV格式化号的PE全部集合到一起，
    相当于一个容器。LV在需要的时候，
    可以随时向这个容器（VG）中区取PE,实现动态扩容。
![wKioL1PD7MaDJ4dKAAEx2v2w1Dc674](F51AC33F7DFB405D84CF2E67A5E33555)    



    LVM的创建
        由前面的知识我们可以得到，创建一个LVM设备，首先需要创建PV,VG，然后才能创建LV。
    前提条件：
        准备好要做LVM的磁盘，我们这在里以分区代替。
![wKiom1PD8-eQ3mroAAHUUxPYA_s008](1E8E3DA78F2B4702B94A8DD30C12A1B4)

     做好这一步后，就可进行PV的创建了，使用 pvcreate device...就可以实现。创建好PV后可以使用pvs或者pvdisplay来查看创建好的PV。
 ![wKioL1PD9XHywVh6AAH3t29llnE686](50B3CE77281D47448270699AA907848C)   
    
    下一步就是创建VG了，使用命令 vgcreate -s PE_SIZE VG_NAME DEVICE...。创建好VG后可以使用vgs或者vgdisplay来查看相关信息。 
 ![wKioL1PD98CSVRUHAAJKnEfCh98241](E4FE03FB74E14700A720E03DEB5094F1)
 
    
     最后一步就是创建LV，使用lvcreate [options]... VG_NAME来创建，同样可以采用lvs,lvdisplay来查看相关的信息。 
![wKiom1PD_fbgt7RYAAM7ZReAc2I642](92F65C79EAB24596937A479A830DEDEF)


    逻辑卷的在线扩展
    扩展可以在线执行，不需要卸载逻辑卷，但是的保证VG中有足够的空间，如果不够的话需要扩展。
    在VG有足够的空间下：
    扩展LV时，首先扩展物理边界，其次在扩展逻辑边界（文件系统）。
    扩展物理边界,使用命令lvextend [options] lV_DEVICE。 
    
![wKiom1PEAwmgL1mhAAOVsAywbug708](996EA2D689D34DB6936EC74A2C99C4CD)

    下面就是扩展逻辑边界了，使用 resize2fs LV_DEVICE。
    注意：centos7 使用 xfs_growfs LV_DEVICE
![wKiom1PEA7PhyqpCAALBlnFEL8M023](F8D2F8D261B7442BBB87D8DF58163CD7)

     至此在线扩容就完成了，在扩容的过程中一直在线。
    在VG空间不够情况下：
        此时首先要扩容VG,然后才能扩展LV。但是要想扩展LV首先得添加磁盘分区，如下：
![wKiom1PEBtbAkNeyAABjr5ZymHQ456](C126C73C2EC44B60BFDF476A7EA5EB27)  
     
        
    然后就可扩展VG了。使用vgextend VG_NAME DEVICE
    
![wKiom1PEBx6A-O1LAABqe3mR8zY513](7AF747DC4D874C19A391ACBE4FDB8E15)
    
    此时，我们就可以看到VG扩展成功了，此时就可以进行LV的扩展了。

![wKiom1PEB0ig-cx7AACpujeBe2o439](B8B2AB04DB754632A258E3C0AAD045A7)

    补充：
        在移除 vg 中的 pv 时候，应该先将数据移动到 pv 中的其他次磁盘上，以防数据丢失。使用pvmove命令。
        Usage: pvmove [option] SourcePhysicalVolume[:PhysicalExtent[-PhysicalExtent]...]}
    [DestinationPhysicalVolume[:PhysicalExtent[-PhysicalExtent]...]...]
        这个命令是在 vgreduce 或者 vgremove 的时候会用到，这样可以保证数据不被丢失。
        
        
        
    LVM的快照（snapshat)
    LVM的快照是LV的在过去某一时间点上的备份。LVM快照是临时保留所更改的逻辑卷的原始数据的逻辑卷。快照提供原始卷的静态视图，从而能够以一致状态备份其数据。
    1. 快照卷大小只需足以存储在它存在期间更改的数据即可。
    2. 如果数据更改量大于快照存储容量，则快照将自动变为不可用。（原始卷原封不动，仍然需要从卷组中手动解除挂载和删除不可用的快照。）
    快照的创建和使用：
    快照创建完成后可以直接使用，不需要格式化。
    
![wKiom1PEDerxKXc3AAIWf_Vb9xg617](BA69449B8D8248C8B6163A457CD9EB5F)


    移除逻辑卷(LV)
    当我们不在需要使用逻辑卷后逻辑卷的快照的时候，我们就需要对已做好的逻辑卷进行移除，当然在移除前我们要做好备份。
        移除一个快照，当我们不需要使用快照的时候，就需要移除快照。使用lvremove SNAP_DEVICE命令移除。
        
![wKiom1PFQdOgRIJWAAG7wmQw30M484](DA95EA0A484040D0AD520D3EB272AAC1)

    移除逻辑卷的过程，首先删除逻辑卷，其次移除卷组，物理卷，最后还原分区。一切还原如初。
        移除逻辑卷：
        
![wKiom1PFRJjSMKaKAAG3qaLyjvM419](41F8FA5E872745ECB9D606CA9B116423)

    移除卷组：
![wKioL1PFRSfA_yCNAAFxTlf1YCI430](9BBF584BFD1D4BC88D64D8566F5F11D9) 

    移除物理卷：
![wKiom1PEBtbAkNeyAABjr5ZymHQ456](2CC42C772F8E486CAF74CCB96AC74193)

    最后，快速还原删除磁盘分区：
    
![wKiom1PFRxmQf5qoAANE__As8xQ874](F8B1B763050C401C9EFB071BE0C1D868)

    此文侧重于，LVM的操作，许多命令只是说明了最常用的方法。其他好多参数，还请读者查询 man 手册。 
    
    
#### Linux常用命令之压缩和解压缩命令 

    压缩解压缩格式 .gz 
        将文件压缩为 .gz 格式,只能压缩文件：gzip
        将 .gz 文件解压：gunzip
        
    压缩解压缩格式 .tar.gz 
    将文件或目录压缩为 .tar.gz 格式：tar -zcf
                -c 打包
                -v 显示详细信息
                -f  指定文件名
                -z 打包同时压缩
        与前面的gzip命令不同，通过tar压缩后是保留原文件或原目录的。
    
    将 .tar.gz 文件解压：tar -zxf
                -x 解包
                -v 显示详细信息
                -f 指定解压文件
                -z 解压缩

    压缩解压缩格式 .zip
        将文件或目录压缩为 .zip 格式：zip
        　-r  压缩目录
        
        将 .zip 文件解压：unzip 
        
    压缩解压缩格式 .bz2
        -k　　产生压缩文件后保留原文件
    
    将 .bz2 文件解压：bunzip2
        -k　　解压缩文件后保留原文件


#### Linux软件包的管理--RPM包管理器

接下来一CentOS6系统为例，讲解如何使用 RPM 包管理器，yum包管理器以及源码的方式来管理我们的软件包。

    RPM包管理器的使用
        rpm -ivh 包全名
        -i(install)：安装
        -v(verbose)：显示详细信息
        -h(hash)：显示进度
        --nodeps：不检测依赖性
        --nodeps：不检测依赖性

    RPM包升级
    rpm -Uvh 包全名（可替代安装） 
    -U（upgrade）：升级
    
    RPM包卸载
    rpm -e 包名（只能跟包名，不能跟包全名，可在任何目录执行） 
    -e（erase）：卸载
    --nodeps：不检查依赖性
    
    RPM包查询
    rpm -q 包名：查询包是否安装
    rpm -qa：查询所有已安装的RPM包
    rpm -qa | grep httpd：查询匹配
    rpm -qi 包名 
    i（information）：查询软件信息
    p（package）：查询未安装包信息（包全名）
    rpm -ql 包名：查询包中文件安装位置 
    -l
    -p
![image](94A417DE12F642D68ADC0E53B0BE5AEA)

    rpm -qf 系统文件名：查询系统文件属于哪个RPM包 
    -f（file） ：查询系统文件属于哪个软件包


#### Linux软件包的管理--YUM

    增强版的RPM管理器-YUM
    命令：
     ####查看软件包
      yum list all              ##列出yum源仓库里面的所有可用的安装包 
      yum list installed        ##列出所有已经安装的安装包  
      yum list available        ##列出没有安装的安装包
     ####安装软件
      yum install softwarename  ##安装指定的软件
      -y：自动输入yes
      yum reinstall softarename ##重新安装指定的软件
      yum localinstall 第三方software  ##安装第三方文件并且会解决软件的依赖关系
      yum remove  softwarename  ##卸装指定的软件
     ####查找软件的信息
      yum info software         ##查看软的信息
      yum search keywords       ##根据关键字查找到相关安装包软件的信息
      yum whatprovides filename ##查找包含指定文件的相关安装包
     ####对于软件组
       yum groups list          ##列出软件组
       yum groups install       ##安装一个软件组
       yum group remove         ##卸载一个软件组
       yum groups info          ##查看一个软件组的信息
        


#### Linux网络配置

    Linux主机在局域网或者互联网上通信的时候，需要设置IP地址，网关，DNS，路由等相关属性。在Linux中这些设置有2种方式：
    命令行设置
    此设置会立即生效，但系统重新登录后不会生效。
    配置文件设置
    此设置不会立即生效，但系统再次登录后会生效，所谓永久生效。
    
        一、设置主机名
    临时设置：
        hostname [USER_NAME] 
    # hostname 直接使用就是查看当前系统的主机名
    [root@server ~]# hostname 
    server.example.com
    # hostname USER_NAME 如果后面有用户名则是修改当前系统的主机名
    [root@server ~]# hostname alex.example.com
    [root@server ~]# hostname 
    alex.example.com
    [root@server ~]#
    
    永久设置：
        配置文件的路径是：/etc/sysconfig/network，如下：
    # 这个文件是Linux网络的“总开关”，像网关，DNS等属性都可以在这里设置
    [root@server ~]# cat /etc/sysconfig/network
    # 这个选项是是否启动网络服务，如果是 no 即使你的其他配置都正确，也不能上网
    NETWORKING=yes
    # HOSTNMAE 关键字就是设置主机名的，设置格式如下:
    HOSTNAME=server.example.com
    
    
    二、设置主机的IP地址
    临时设置：
        ifconfig [options] [IFNMAE [IP netmask MASK]]
    # ifconfig 查看当前处于激活状态的网卡配置信息
    # ifconfig -a 查看系统中所有的网卡配置信息
    [root@server ~]# ifconfig 
    eth0      Link encap:Ethernet  HWaddr 00:0C:29:9E:A3:0E  
              inet addr:172.16.9.10  Bcast:172.16.255.255  Mask:255.255.0.0
              inet6 addr: fe80::20c:29ff:fe9e:a30e/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:273411 errors:0 dropped:0 overruns:0 frame:0
              TX packets:2495 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:17152580 (16.3 MiB)  TX bytes:297433 (290.4 KiB)
    
    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:16436  Metric:1
              RX packets:6 errors:0 dropped:0 overruns:0 frame:0
              TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:330 (330.0 b)  TX bytes:330 (330.0 b)
              
    # ifconfig DEVICE  查看系统中特定网卡配置信息
    [root@server ~]# ifconfig eth0
    eth0      Link encap:Ethernet  HWaddr 00:0C:29:9E:A3:0E  
              inet addr:172.16.9.10  Bcast:172.16.255.255  Mask:255.255.0.0
              inet6 addr: fe80::20c:29ff:fe9e:a30e/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:273444 errors:0 dropped:0 overruns:0 frame:0
              TX packets:2510 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:17155146 (16.3 MiB)  TX bytes:299847 (292.8 KiB)
            
    [root@server ~]# ifconfig eth1
    eth1      Link encap:Ethernet  HWaddr 00:0C:29:9E:A3:18  
              inet addr:172.16.9.18  Bcast:172.16.255.255  Mask:255.255.0.0
              inet6 addr: fe80::20c:29ff:fe9e:a318/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:704 errors:0 dropped:0 overruns:0 frame:0
              TX packets:22 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:60911 (59.4 KiB)  TX bytes:1468 (1.4 KiB)
    
    # 临时修改网卡的IP地址，修改完成时候会立即生效
    # ifconfig eth1 192.168.0.55/24 up中的up是启用网卡，如果不启用可以使用down
    # 启动和关闭一个网卡,例如：启动eth1网卡
    #  igconfig eth1 up
    #  或者 ifup  eth1 （ifdown eth1 关闭）
    [root@server ~]# ifconfig eth1 192.168.0.55/24 up
    [root@server ~]# ifconfig eth1
    eth1      Link encap:Ethernet  HWaddr 00:0C:29:9E:A3:18  
              inet addr:192.168.0.55  Bcast:192.168.0.255  Mask:255.255.255.0
              inet6 addr: fe80::20c:29ff:fe9e:a318/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:844 errors:0 dropped:0 overruns:0 frame:0
              TX packets:22 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:72758 (71.0 KiB)  TX bytes:1468 (1.4 KiB)
    
    [root@server ~]# 
    
    # ifconfig设置网卡别名
    [root@server ~]# ifconfig eth1:0 192.168.0.58/24 up
    [root@server ~]# ifconfig eth1:0
    eth1:0    Link encap:Ethernet  HWaddr 00:0C:29:9E:A3:18  
              inet addr:192.168.0.58  Bcast:192.168.0.255  Mask:255.255.255.0
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
    
    [root@server ~]#
    
     
    永久设置：
        配置文件在 /etc/sysconfig/network-scripts/ 目录下：每个网卡对应一个配置文件（包括网卡别名的） ifcfg-IFNAME。例如：eth0网卡的配置文件是 ifcfg-eth0。
        这个配置文件中的一些关键字和意义如下：
    DEVICE=IFNAME 此配置文件所关联到的设备，设备名称要与本文件名ifcfg-后面保持一致
    BOOTPROTO={bootp|dhcp|static|none} IP地址配置协议，常用的是dhcp和none(static)
    HWADDR=11:22:33:44:55:66  当前设备的MAC地址,用六个字节标识
    NM_CONTROLLED={yes|no} 是否接受NetworkManager服务脚本来配置此设备，CentOS6建议关闭，因为其不支持桥接模式。CentOS7中已经支持。 
    ONBOOT={yes|no} 是否在开机过程中，自动激活此接口
    TYPE={Ethernet|Bridge} 网络接口类型
    UUID= 设备标识号
    IPADDR=IP地址    设定IP地址，只有在BOOTPROTO={none|static}设置才有效
    NETMASK=   掩码地址，此设置也可用 PREFIX=n (n为掩码位数)
    GATEWAY=    网关地址，要与IP地址属于同一网段
    DNS1=    DNS服务器地址地址
    DNS2=
    IPV6INIT={yes|no} 是否支持IPV6
    USERCTL={yes|no} 是否允许普通用控制此接口
    PEERDNS＝{yes|no} 不接受DHCP服务器指派的DNS服务器地址
        要想永久设置有效的话，将这些信息写到对应网卡的配置文件中，然后重新启动网络服务即可,/etc/init.d/network restart。
        
    三、设置路由信息
    临时设置：
        使用route [options] [add|del] [-net|-host IP gw NEXT_HOP] [dev DEVICE]命令设置。
    # 查看路由表
    [root@server ~]# route 
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    link-local      *               255.255.0.0     U     1002   0        0 eth0
    link-local      *               255.255.0.0     U     1003   0        0 eth1
    172.16.0.0      *               255.255.0.0     U     0      0        0 eth0
    172.16.0.0      *               255.255.0.0     U     0      0        0 eth1
    default         server.magelinu 0.0.0.0         UG    0      0        0 eth0
    # route -n 以数字形式显示路由表
    [root@server ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
    172.16.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
    172.16.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth1
    0.0.0.0         172.16.0.1      0.0.0.0         UG    0      0        0 eth0
    
    # 增加一条主机路由
    [root@server ~]# route add -host 172.16.9.18 gw 172.16.0.1 dev eth0
    # 增加一条网络路由
    [root@server ~]# route add -net 10.0.0.0/8 gw 172.16.0.1 dev eth0
    [root@server ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    172.16.9.18     172.16.0.1      255.255.255.255 UGH   0      0        0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
    172.16.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
    172.16.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth1
    10.0.0.0        172.16.0.1      255.0.0.0       UG    0      0        0 eth0
    0.0.0.0         172.16.0.1      0.0.0.0         UG    0      0        0 eth0
    
    # 设置默认路由
    # route add default gw NEXT_HOP
    
    # 删除主机路由和网络路由
    [root@server ~]# route del -net 10.0.0.0/8  dev eth0
    [root@server ~]# route del -host 172.16.9.18  dev eth0
    [root@server ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
    172.16.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
    172.16.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth1
    0.0.0.0         172.16.0.1      0.0.0.0         UG    0      0        0 eth0
    
    
    永久生效：
        配置文件在 /etc/sysconfig/network-scripts/ 目录下：每个网卡的路由信息对应一个配置文件（包括网卡别名的） route-IFNAME。例如：eth0网卡的配置路由文件是 route-eth0。
        设置格式：
        配置文件的格式1：每行一个路由条目
            DESTINATION via NETX_HOP
    192.168.0.0/24 via 172.16.0.1
    
        配置文件格式2: 每三行一个路由条目
            ADDRESS＃=DESTINATION
            NETMASK＃=MASK
            GATEWAY＃=GW
    # 要是设置主机路由的话，掩码位数是32位
    ADDRESS0=192.168.0.0
    NETMASK0=255.255.0.0
    GATEWAY=172.16.0.1
    
    
    四、ip命令
        ip的命令功能比较强大。
    ip命令常用选项：
    ip link : 管理接口
            show [IFNAME] 查看接口信息
            set IFNAME {up|down} 管理接口
            
    ip addr: 管理协议地址
    ip addr {show|flush} [dev DEVICE] 查看网卡的IP地址
    
    ip addr {add|del} ADDRESS dev DEVICE  [label IFALIAS] [broadcast BCAST_ADDRESS]，为一个网卡添加多个IP地址
    [root@server ~]#  ip addr add 192.168.0.23/24 dev eth0 label eth0:0
    
    ip route: 管理路由：
    ip route list
    route -n
    
    ip route flush 
    ip route add DESTINATION [via NEXT_HOP] [src SOURCE_ADDRESS] [dev DEVICE]
    [root@server ~]# ip route add 10.0.0.0/8 via 172.16.0.1 dev eth0 src 172.16.9.10
    ip route list 
    
    ip route del DESTINATION
    [root@server ~]# ip route del 10.0.0.0/8
    
    五、网络管理工具

    测试网络的连通属性：
    ping 命令，实现的ICMP协议
    常用参数：
        -W timeout: 一个报文等待响应报文的超时时长
        -c #：报文个数
        -w time：一共持续的时间
    
    追踪网络路由：
    就是显示出我们到达一个网络所经过的路由信息。常用的命令有traceroute 和 mtr。
    mtr命令式实时监测显示的：
    
    查看网络状态：
    常用命令是：netstat和ss。
    netstat常用选项：
        -t: tcp协议相关
        -u: udp协议相关
        -n: 显示数字格式的地址
        -l: listen，显示处于监听状态的连接
        -p: 显示会话中的进程程序名及进程号
        -a: 所有状态的连接
        -r: routing，显示路由表
        
        
    ss命令也可以显示这些状态：
        常用选型：
            -t: tcp相关
            -u: udp相关
            -p: 显示进程号
            -l: listen，显示处于监听状态的连接
            -n: 显示数字格式的地址
            -m: 套接字相关的内存使用信息
            -a: all
            -e: 扩展信息
            -o state {established,fin_wait_1, fin_wait_2, listening}
            '( dport =   or sport =  )'
            只显示指定状态的连接，还可以指定过滤条件
            [root@server ~]# ss -atn -o state established '(sport = :22)' dst 192.168.1/24
            
        显示网络接口设备的属性信息：
        ethtool:查看网路接口设备本身的属性
    
    
#### Linux的进程管理

      在操作系统系统中，进程是一个非常重要的概念。
    一、Linux中进程的相关知识
    1、什么是进程呢？
        通俗的来说进程是运行起来的程序。唯一标示进程的是进程描述符（PID）,在linux内核中是通过task_struck和task_list来定义和管理进程的。
    2、进程的分类
        1）根据在linux不同模式下运行分为：
            核心态：这类进程运行在内核模式下，执行一些内核指令（Ring 0）。
            用户态：这类进程工作在用户模式下，执行用户指令（Ring 3）。
        如果用户态的进程要执行一些核心态的指令，此时就会产生系统调用，系统调用会请求内核指令完成相关的请求，就执行的结果返回给用户态进程。
        2）按照进程的状态可分为：
            运行态：running 正在运行的进程
            可中断睡眠态：进程处于睡眠状态，但是可以被中断
            不可中断的睡眠态：进程处于睡眠状态，但是不可以被中断
            停止态：stoped 不会被内核调度
            僵死态：zombie产生的原因是进程结束后，它的父进程没有wait它，所导致的。
        3）按照操作的密集程度
            CPU密集型：进程在运行时，占用CPU时间较多的进程。
            I/O密集型：进程在运行时，占用I/O时间较多的进程。
        通常情况下，I/O密集型的优先级要高于CPU密集型。
        4）按照进程的处理方式
            批处理进程：
            交互式进程：
            实时进程：
    3、进程的优先级
        进程的有优先级，是用0-139数字来表示的，数字优先级从小到大依次是：0-99,139-100。
        优先级分为2类：
            实时优先级：0-99，是由内核维护的
            静态优先级：100-139，可以使用nice来调整，nice值的取值范围是[-20,19),分别对应100到139。nice默认值是0。
            动态优先级：由内核动态维护，动态调整。

    二、进程的管理工具
        1、pstree命令 查看进程数。
        2、ps 命令 查看进程的相关状态。支持SysV和BSD两种风格的选项。
    常用选型：
        a  与终端相关的进程
        x  与终端无关的进程
        u  显示运行进程的用户
    常用组合选项：ps aux
    
        # 下面分别来说明上图的各个字段的含义
    # USER   进程以什么用户身份运行
    # PID    进程描述符 具有唯一性
    # %CPU   进程运行时所占的cpu百分比
    # %MEM   进程运行时内存所占的百分比
    # VSZ    Virtual memory SiZe 虚拟内存使用大小
    # RSS    常驻内存集，所有不能被置换出去的内存集
    # STAT   表示内存状态
    #    常用的状态有：
    #    S：可中段睡眠状态
    #    R：运行态
    #    D：不可中断睡眠态
    #    T：停止态
    #    Z：僵尸态
    #    s：session leader 所谓进程的领导者
    #    +：表示是前台进程
    #    l：多线程进程
    #    N：低优先级进程
    #    <：高优先级进程
    # TTY 用来表示终端 显示为“？”的说明是与终端无关的进程
    # START 进程开始时间
    # TIME 进程运行时间
    # COMMAND 执行进程的命令 如果命令被 "[]"包围，说明是内核线程
    
    
    -e 显示所有进程
    -f 显示完成格式信息
    常组合在一起使用：ps -ef
    PPID:父进程的PID
    C：CPU利用率
    
    但是有些这种情况下，我们的命令有时候显示不完整
    
    此时想要显示完成就要 ps -efww
    
    -F：显示额外信息
    -H：显示进程的层次结构
    常用组合方式：ps -eFH
    PSR：运行到哪颗CPU上
    
    可能以后我们用到最多的选项：
    -o 我们可以自定义显示字段
    # 常用的有：
    # pid    command    psr    pri    ni    %cpu     %mem    rsz    vsz等
    
    3、pgrep,pidof
    pgrep 常用选型：
        -U 查看指定用户的进程号
        -G 查看指定用户组的进程号
        -l 显示进程名和进程号
        
    [root@server ~]# pgrep -U root -G root -l
    查看用户为root 用户组也为root的进程PID和名称
    
    pidof：只显示已启动进程的PID
     
     4、top命令
    实时监控系统资源
    
    # 执行top命令后，进入交互式模式
    # top中的一些交互式命令：
    # l：控制是否显示第一行，负载均衡信息
    # t：控制是否显示进程信息由和cpu信息
    # m：控制是否显示内存，交换信息
    # I 或者 1（数字）：是否分别显示cpu每个信息
    # M: 按%mem排序显示，从大到小
    # k: kill 杀掉进程
    # s：修改默认刷新时间 默认是3秒
    
    # 下面解释抬头信息：
    top - 21:35:17 up 10:03,  4 users,  load average: 0.00, 0.00, 0.00
    #   系统时间   启动时间 登录用户数    负载均衡：1min 5min 15min
    # 何为系统负载？在这里指的是等待在进程队列里的平均进程数
    # 此出显示的信息 等价于 uptime 命令
    
    Tasks: 165 total,   1 running, 164 sleeping,   0 stopped,   0 zombie
    # 进程总数            运行数     睡眠态数        停止态数    僵尸进程数
    Cpu(s):  0.0%us,0.0%sy, 0.0%ni, 100.0%id, 0.0%wa, 0.0%hi, 0.0%si,  0.0%st
    # 0.0%us：user space:用于运行用户空间的程序所占的cpu百分比 
    # 0.0%sy：system space:用于运行内核空间的程序所占的cpu百分比   
    # 0.0%ni：nice值调用时间所占cpu百分百比
    # 100.0%id：系统cpu空闲所占百分比
    # 0.0%wa：用于等待I/O所占的cpu百分比
    # 0.0%hi：硬中断所占cpu百分比
    # 0.0%si：软中断所占cpu百分比
    # 0.0%st：系统被“偷走”的cpu所占的百分比，一般指的是用于虚拟机运行所占的cpu
    
    Mem:   1012548k total,   396328k used,   616220k free,    99444k buffers
    #       总内存大小     使用的内存大小    剩余内存大小    缓存的大小
    Swap:  2097144k total,        0k used,  2097144k free,   144156k cached
    #    交换分区总大小        使用的        剩余的            缓冲大小
    # 此处显示的信息等价于 free 命令
    
    
    常用选项：
        -d #: 指定刷新时间间隔
        -b: 以批次的方式显示top的刷新
        -n #: 显示的批次
    例如：top -d 4 -b 2 -n 3
    
    4、htop
    htop命令是top命令的升级版，无论是在功能上还是在界面显示上，都比top命令更胜一筹。
        u: 交互式选择显示指定用户的进程
        l: 显示光标所在进程所打开的文件列表
        s: 显示光标所在进程执行的系统调用
        a: 绑定进程到指定的CPU
        #：快速定位光标至PID为#的进程上
    
    
    
    5、vmstat
    wmstat 查看虚拟使用情况
    # 常用用法：
    # vmstat 显示信息会默认1秒刷新一次，一直显示
    # vmstart -n 2 显示信息会2秒刷新一次，一直显示
    # vmstat  -n 1 4 显示信息会1秒刷新一次，刷新4次
    
    [root@server ~]# vmstat -n 1 1
    procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     0  0      0 614392 100468 144776    0    0     2     1    6    5  0  0 100  0  0
     
    # 我们解释一下每个字段的含义：
    # procs字段 关于进程的
    # r    指运行队列的进程数，如果过长，可能是cpu性能较低
    # b    阻塞队列的长度，通常是用于等待I/O的完成。如果太大，说明I/O性能较低
    
    # memory字段 关于内存使用的
    # swap 使用的交换内存大小
    # free 空余内存大小  它的值=总大小-buff-cache-used
    # buff 缓冲大小，目的是为了加速I/O的写操作(一般是磁盘)
    # cache 缓存大小，摸底是为了加速I/O的读操作(一般是磁盘)
    
    # swap字段 说明交换内存
    # si swapin 指的是数据进入交换内存的速率 单位：KB/s
    # so swapout 指的是数据出交换内存的速率 单位：KB/s
    # 如果这2个值比较大的时候，会出现抖动现象。建议增加内存
    
    # io字段  I/O的说明
    # bi：Block in 从块设备读入内存的速率 KB/s
    # bo: block out 保存到块设备的速率 KB/s
    # 这就是我们通常说的磁盘的读写性能，可以通过RAID提高。
    
    # system字段 关于系统的
    # in: interruppt 中断发生的速率
    # cs: 上下文切换的速率（进程调度）
    
    # cpu字段 说明cpu的使用情况
    # us：user space:用于运行用户空间的程序所占的cpu百分比
    # sy：system space:用于运行内核空间的程序所占的cpu百分比   
    # id：系统cpu空闲所占百分比
    # wa：用于等待I/O所占的cpu百分比，这是由于cpu和i/o速度相差太多所造成的
    # st：系统被“偷走”的cpu所占的百分比，一般指的是用于虚拟机运行所占的cpu的时间百分比
    
    
    6、nice,renice
        调整进程的优先级。
        nice 在进程启动的时候设置优先级。
    # 常用参数：
    # -n NICE 例如：nice -n 3 httpd
    # 一般情况下，nice值是负值的设定一般有管理员来设定。普通用户只能设置nice为正值。
    # 如果不指定 -n 参数，默认的nice值是10
    
        renice 重新设置已启动进程的优先级。
    # 常用选项是：
    # -n NICE 重新设定nice的值 
    # -p PID  设定进程的PID
    
    7、kill,killall
        对于有Linux C编程经验的人来说，我们知道IPC通信方式之一就是通过信号量（signal）,那么对于kill和killall命令来说，它们与信号量有着很大的关系，或者说kill,killall命令通过信号量让我们可以手动的向进程传递信号来控制进程。
        常见的信号量如下：
    [root@server ~]# kill -l
     1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
     6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
    11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
    16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
    21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
    26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
    31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
    38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
    43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
    48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
    53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
    58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
    63) SIGRTMAX-1	64) SIGRTMAX	
    
    # 我们常用到的信号是：
    # 1 SIGHUP  在不关闭进程的情况下，重读配置文件。
    ngnix在这方面做得相当的好，甚至可以实现在线升级。
    # 2 SIGINT  中断信号 相当于 ctrl + C 
    # 9 SIGKILL  暴力杀死
    # 15 SIFTERM 优雅的关闭  默认是这种情况
    
    # kill用法如下：
    # kill [-SIGNAL] PID
    # 对于SIGNAL有三种表示：例如：1) -9 -15 -1 -2   2）-SIGKILL -SIGHUP -SIGTERM   3) -HUP -KILL -TERM -INT 等。
    # 
    # killall是杀掉一类进程
    # 例如：killall httpd 等价于 kill `pidof httpd`
    
    8、jobs,bg,fg
        什么是作业呢？作业就是许多进程一起协同完成一项具体的工作。
        作业有前台作业和后台作业2种。
        使用 & 或者 ctrl + Z可以把一个进程打入后台。
    # ping 192.168.0.1 &
    # 这样打入后台的运行的作业，退出终端的时候，作业就会终止。
    # 使用 nohup 命令可避免这个问题
    # nohup ping 192.168.0.1 &
    
    # 可以使用jobs命令查看后台的作业
    # 每个作业都有一个作业号来标识作业
    
    # 作业控制命令
    # bg [[%]JOB_NUM]:   让送往后台的作业继续在后台运行 
    # fg [[%]JOB_NUM]:   将后台作业调回前台
    # kill %JOB_NUM:     终止指定的作业
    
     9、dstat命令，查看各种状态信息
    用法：dstat [-afv] [options..] [delay [count]]
    常用参数
    具体意义
    -c
    显示cpu统计数据,如有多个CPU汇总统计
    
    -d
    显示disk统计数据，如有多块磁盘则汇总统计
    
    -D DEVICE
    显示特定磁盘的信息
    
    -g
    显示page信息
    
    -i
    显示中断的统计数据
    
    -m
    显示内存的统计信息
    
    -l
    显示系统的负载信息
    
    -n
    显示网络接口的相关属性
    
    -s
    显示系统属性
    
    -N INTER_FACE_NAME
    显示特定接口的属性
    
    -s
    显示交换内存的属性
    
    -p
    显示进程队列
    
    --ipc
    显示ipc消息队列、信号量和共享内存的使用状况
    
    -a
    等价于 -cdngy 显示CPU，磁盘，网卡，page,系统属性
    
    -f
    以完整格式显示所有信息，
    
    --tcp,--udp
    显示tcp，udp状态信息
     [root@server ~]#  dstat --tcp 1 1
     1:时延       1：次数
    
    
    10、查看内存映射
    pmap PID 查看对应进程的内存映射，常用的用法是：
    pman `pidof PROCESS_NAME`。
    当然这些信息也可以查看 /proc/PID/pmap 文件查看。
    
![wKioL1PPlxuwUs3qAASA-p7eoQo539](34C7F8F14E8441D0B81B7F551BBFD6F1)
    
    
    11、glances命令
    一款强大的系统监控工具：能实时监控像cpu,meomory,
    load,swap,Network，mount,disk等信息。
    
    
    总结：本文主要进程管理的基本知识和管理工具。
    管理工具有：pstree,pgrep,pidofps,top,htop,vmstat,
    nice,renice,kill,killall,
    jobs,bg,fg,dstat,pmap,glances。
    
    

#### centOS7服务管理与启动流程

在按下电源到输入账号和密码之前，操作系统都做了些什么？下面就来讲述在这段时间发生的动作。

        CentOS7引导顺序
        UEFi或BIOS初始化，运行POST开机自检
        选择启动设备
        引导装载程序, centos7是grub2
        加载装载程序的配置文件：/etc/grub.d/ /etc/default/grub /boot/grub2/grub.cfg
        加载initramfs驱动模块
        加载内核选项
        内核初始化，centos7使用systemd代替init
        执行initrd.target所有单元，包括挂载/etc/fstab
        从initramfs根文件系统切换到磁盘根目录
        systemd执行默认target配置，配置文件/etc/systemd/system/default.target
        systemd执行sysinit.target初始化系统及basic.target准备操作系统
        systemd启动multi-user.target下的本机与服务器服务
        systemd执行multi-user.target下的/etc/rc.d/rc.local
        Systemd执行multi-user.target下的getty.target及登录服务
        systemd执行graphical需要的服务

![image](A21E3C65A12C4E2988241D98C5F93260)


    Systemd：系统启动和服务器守护进程管理器，负责在系统启动或运行时，激活系统资源，服务器进程和其它进程 
    
    Systemd新特性：
        系统引导时实现服务并行启动
        按需启动守护进程
        自动化的服务依赖关系管理
        同时采用socket式与D-Bus总线式激活服务
        系统状态快照

    unit对象
    核心概念：unit 
    unit表示不同类型的systemd对象，通过配置文件进行标识和配置；文件中主要包含了系统服务、监听socket、保存的系统快照以及其它与init相关的信息 
    配置文件：
    /usr/lib/systemd/system:每个服务最主要的启动脚本设置，类似于之前的/etc/init.d/
    /run/systemd/system：系统执行过程中所产生的服务脚本，比上面目录优先运行
    /etc/systemd/system：管理员建立的执行脚本，类似于/etc/rc.d/rcN.d/Sxx类的功能，比上面目录优先运行
     
    unit类型
    Systemctl –t help 查看unit类型
    Service unit: 文件扩展名为.service, 用于定义系统服务
    Target unit: 文件扩展名为.target，用于模拟实现运行级别
    Device unit: .device, 用于定义内核识别的设备
    Mount unit: .mount, 定义文件系统挂载点
    Socket unit: .socket, 用于标识进程间通信用的socket文件，也可在系统启动时，延迟启动服务，实现按需启动
    Snapshot unit: .snapshot, 管理系统快照
    Swap unit: .swap, 用于标识swap设备
    Automount unit: .automount，文件系统的自动挂载点
    Path unit: .path，用于定义文件系统中的一个文件或目录使用,常用于当文件系统变化时，延迟激活服务，如：spool 目录
    特性
    关键特性：
    基于socket的激活机制：socket与服务程序分离
    基于d-bus的激活机制：
    基于device的激活机制：
    基于path的激活机制：
    系统快照：保存各unit的当前状态信息于持久存储设备中向后兼容sysv init脚本
    不兼容：
    systemctl命令固定不变，不可扩展
    非由systemd启动的服务，systemctl无法与之通信和控制
     
    service unit文件格式
    在/etc/systemd/system下的unit文件是系统管理员和用户使用 
    在/usr/lib/systemd/system下的供发行版打包者使用
    在unit文件中，以“#” 开头的行后面的内容会被认为是注释，相关布尔值
    ，1、yes、on、true 都是开启，0、no、off、false 都是关闭，时间单位默认是秒，
    所以要用毫秒（ms）分钟（m）等须显式说明
    service unit file文件通常由三部分组成
    [Unit]：定义与Unit类型无关的通用选项；用于提供unit的描述信息、unit行为及依赖关系等
    [Service]：与特定类型相关的专用选项；
    此处为Service类型
    [Install]：定义由“systemctlenable”以及"systemctldisable“命令在实现服务启用或禁用时用到的一些选项
    unit段的常用选项
    Description：描述信息
    After：定义unit的启动次序，
    表示当前unit应该晚于哪些unit启动，其功能与Before相反
    Requires：依赖到的其它units，强依赖，
    被依赖的units无法激活时，当前unit也无法激活
    Wants：依赖到的其它units，弱依赖
    Conflicts：定义units间的冲突关系
    Service段的常用选项
    Type：定义影响ExecStart及相关参数的功能的unit进程启动类型 
    simple：默认值，这个daemon主要由ExecStart接的指令串来启动，启动后常驻于内存中
    forking：由ExecStart启动的程序透过spawns延伸出其他子程序来作为此daemon的主要服务。
    原生父程序在启动结束后就会终止
    oneshot：与simple类似，不过这个程序在工作完毕后就结束了，不会常驻在内存中
    dbus：与simple类似，但这个daemon必须要在取得一个D-Bus的名称后，
    才会继续运作.因此通常也要同时设定BusNname= 才行
    notify：在启动完成后会发送一个通知消息。还需要配合NotifyAccess 来让Systemd 接收消息
    idle：与simple类似，要执行这个daemon必须要所有的工作都顺利执行完毕后才会执行。
    这类的daemon通常是开机到最后才执行即可的服务
    EnvironmentFile：环境配置文件
    ExecStart：指明启动unit要运行命令或脚本的绝对路径
    ExecStartPre：ExecStart前运行
    ExecStartPost：ExecStart后运行
    ExecStop：指明停止unit要运行的命令或脚本
    Restart：当设定Restart=1 时，则当次daemon服务意外终止后，会再次自动启动此服务
    Install段的常用选项
    Alias：别名，可使用systemctlcommand Alias.service
    RequiredBy：被哪些units所依赖，强依赖
    WantedBy：被哪些units所依赖，弱依赖
    Also：安装本服务的时候还要安装别的相关服务
    注意： 
    对于新创建的unit文件，或者修改了的unit文件，要通知systemd重载此配置文件,而后可以选择重启 
    systemctldaemon-reload
    
    
    
    管理服务
    管理系统服务
    CentOS 7: service unit 
    注意：能兼容早期的服务脚本 
    命令： 
    systemctl COMMAND name.service
    -
    
    centOS6
    CentOS7
   
   
    启动
    service name start
    systemctl start name.service
  
   
    停止
    service name stop
    systemctl stop name.service
   
    
    重启
    service name restart
    systemctl restart name.service
    
    
    状态
    service name status
    systemctl status name.service
   
   
    条件式重启(已启动才重启，否则不做操作)
    service name condrestart
    systemctl try-restart name.service
    
    
    重载或重启服务(先加载，再启动)
    -
    systemctl reload-or-restart name.service
    
    
    重载或条件式重启服务
    -
    systemctl reload-or-try-restart name.service
    
    
    禁止自动和手动启动
    -
    systemctl mask name.service
    
    
    取消禁止
    -
    systemctl unmask name.servic
    
    服务查看
    -
    centOS6
    CentOS7
    
    
    查看某服务当前激活与否的状态
    -
    systemctl is-active name.service
    
    
    查看所有已经激活的服务
    -
    systemctl list-units --type
    
    
    查看所有服务
    -
    systemctl list-units --type service --all
    
    chkconfig命令的对应关系
    -
    centOS6
    CentOS7
    
    
    设定某服务开机自启
    chkconfig name on
    systemctl enable name.service
    
    
    设定某服务开机禁止启动
    chkconfig name off
    systemctl disable name.service
    
    
    查看所有服务的开机自启状态
    chkconfig --list
    systemctl list-unit-files --type service
    
    
    用来列出该服务在哪些运行级别下启用和禁用
    chkconfig sshd –list
    ls /etc/systemd/system/*.wants/sshd.service
    
    
    查看服务是否开机自启
    -
    systemctl is-enabled name.service
     
    其他命令
    -
    centOS6
    CentOS7
    
    
    查看服务的依赖关系
    -
    systemctl list-dependencies name.service
    
    
    杀掉进程
    -
    systemctl kill unitname
    
    
    切换至紧急救援模式
    -
    systemctl rescue
    
    
    切换至emergency模式
    -
    systemctl emergency
    
    
    关机
    -
    systemctlhalt、systemctlpoweroff
    
    
    重启
    -
    systemctl reboot
    
    
    挂起
    -
    systemctl suspend
    
    
    休眠
    -
    systemctl hibernate
    
    
    休眠并挂起
    -
    systemctlhybrid-sleep
    
    
    服务状态
  
    systemctl list-unit-files --type service --all #显示状态
    loaded:Unit配置文件已处理
    active(running):一次或多次持续处理的运行
    active(exited):成功完成一次性的配置
    active(waiting):运行中，等待一个事件
    inactive:不运行
    enabled:开机启动
    disabled:开机不启动
    static:开机不启动，但可被另一个启用的服务激活
     
    systemctl示例
    显示所有单元状态
    systemctl 或systemctl list-units
    只显示服务单元的状态
    systemctl --type=service
    显示sshd服务单元
    systemctl –l status sshd.service
    验证sshd服务当前是否活动
    systemctl is-active sshd
    启动，停止和重启sshd服务
    systemctl start sshd.service
    systemctl stop sshd.service
    systemctl restart sshd.service
    重新加载配置
    systemctl reload sshd.service
    列出活动状态的所有服务单元
    systemctl list-units --type=service
    列出所有服务单元
    systemctl list-units --type=service --all
    查看服务单元的启用和禁用状态
    systemctl list-unit-files --type=service
    列出失败的服务
    systemctl --failed --type=service
    列出依赖的单元
    systemctl list-dependencies sshd
    验证sshd服务是否开机启动
    systemctl is-enabled sshd
    禁用network，使之不能自动启动,但手动可以
    systemctl disable network
    启用network
    systemctl enable network
    禁用network，使之不能手动或自动启动
    systemctl mask network
    启用network
    systemctl unmask network
    
    
    运行级别
    在centOS7上运行级别的含义已经和之前不同了，
    运行级别就是通过开启关闭不同的服务产生的效果，
    在从netOS7上，已然由.target来代替运行级别
    ，我们可以称target为目标态，
    我们可以通过target定制更符合我们工作运行环境。 
    
    我们可以通过命令：ls /usr/lib/systemd/system/*.target
    查看我们的机器上有多少个target
    
    使用systemctl list-unit-files --type target --all可以查看所有目标态的状态，
    
    或者systemctl list-dependencies xxx.target命令查看目标态的依赖性。 
    
    在centOS7上所谓的目标态，其实就是由各种指定的服务和基础target组合而成的。
    
    运行级别与target的对照
        0 ==> runlevel0.target, poweroff.target 
        1 ==> runlevel1.target, rescue.target 
        2 ==> runlevel2.target, multi-user.target 
        3 ==> runlevel3.target, multi-user.target 
        4 ==> runlevel4.target, multi-user.target 
        5 ==> runlevel5.target, graphical.target 
        6 ==> runlevel6.target, reboot.target
        
     
    运行级别的切换
    在centOS6上，我们切换级别使用init，
    在centOS7上虽然也能使用，但是已经不再是原来的程序了
    ，现在我们使用systemctlisolate name.target来切换target。
    比如，我们想切换到字符界面，
    我们就可以使用systemctlisolate multi-user.target来进行切换。
    要想切换运行级别，
    在/lib/systemd/system/*.target文件中AllowIsolate=yes才可以。
    （修改文件需执行systemctldaemon-reload才能生效） 
    
    在centOS7上如何查看运行的目标态呢，使用命令systemctl get-default 
    使用命令systemctl set-default name.target来修改我们的目标态。
    
    
    
    设置简单的内核参数
    设置内核参数，只影响当次启动
    启动时，在linux16行后添加systemd.unit=desired.target 
    systemd.unit=emergency.target
    systemd.unit=rescue.target
    rescue.target 比emergency 支持更多的功能，例如日志等
    systemctl default 进入默认target
     
    简单的启动排错
    在centOS7中，文件系统损坏，先尝试自动修复，
    失败则进入emergency shell，提示用户修复
    在/etc/fstab不存在对应的设备和UUID等一段时间，
    如不可用，进入emergency shell
    在/etc/fstab不存在对应挂载点,
    systemd尝试创建挂载点，
    否则提示进入emergency shell.
    在/etc/fstab不正确的挂载选项,提示进入emergency shell
    
    centOS7 grub2
    在centOS6上，我们的grub文件是/boot/grub/grub.conf 
    在centOS7上，文件改成/boot/grub2/grub.cfg了，但是功能还是大致一样的都是用于加载内核的，
    不过在centOS7上设置默认启动项发生了一些变化，
    假如我们现在有两个内核，
    我们需要改变默认启动应该如何做到呢？
    首先，vim /etc/default/grub打开
    
    打开文件后，我们修改GRUB_DEFAULT的值，
    和centOS一样，0代表第一个内核，1代表第二个，
    以此类推。我们在修改完成后，并没有立即生效，
    使用grub2-mkconfig -o /boot/grub2/grub.cfg命令来生成grub2.cfg文件，
    我们在下次启动的时候就会默认选择新的默认内核。


#### Linux内核分析
Linux体系结构简介
Linux内核源码简介
Linux内核配置、编译、安装

    Linux可以分为两部分，分别为用户空间和内核空间具体如下图：
    
![image](E7CB4661533B466BBA687ECAB11BFD76)
    
    a)        用户空间包括：用户的应用程序、C库
    b)        内核空间包括：系统调用接口、内核（狭义内核）、平台架构相关的代码

    
    为什么要分为内核空间和用户空间
    我们在分析u-boot的时候就说到过，
    我们的cpu在不同的工作模式下可以访问的寄存器是不一样的，所以为了保护我们的操作系统
    ，避免用户程序将内核搞崩，
    所以进行了内核空间和用户空间的划分。
    
    a)        Arm处理器工作模式划分：usr、FIQ、IRQ、svc、abt、und、sys（具体介绍在http://www.cnblogs.com/wrjvszq/p/4199682.html）
    b)        X86处理器工作模式划分：Ring0—Ring3，Ring0下可以执行特权指令，
    可以访问IO设备，Ring3则有很多的限制
    
    注：我们可以通过系统调用和硬件中断来完成用户空间到内核空间的转移
    





Linux内核结构（广义内核）
         Linux内核由七个部分构成，具体如下图：
         
![image](21AB9C04121045E8BA7E1AB343F436F5)   

    
    a)        系统调用接口（SCI）：open、read、write等系统调用
    b)        进程管理（PM）：创建进程、删除进程、调度进程等
    c)        内存管理（MM）：内存分配、管理等
    d)        虚拟文件系统（VFS）：为多种文件系统提供统一的操作接口
    e)        网络协议栈：提供各种网络协议
    f)         CPU架构相关代码（Arch）：为的是提高至移植性
    g)        设备驱动程序（DD）：各种设备驱动，占到内核的70%左右代码
    
    
    源码目录简介
        其源码主要有以下目录（介绍重要目录）：
        a)        Arch目录：存放处理器相关的代码。下设子目录，分别对应具体的CPU，每个子目录有boot，mm，以及kernel三个子目录，分别对应系统引导以及存储管理，和系统调用
        b)        Include目录：内核所需要的大部分头文件目录。与平台无关的在include/linux子目录下，
        与平台相关的则放在include相应的子目录中。
        c)        fs目录：存放各种文件系统的实现代码。
        d)        init目录：init子目录包含核心的初始化代码（不是系统的引导代码）。
        其包含两个文件main.c和version.c，可以用来研究核心如何工作。
        e)        ipc目录：包含核心进程间的通信代码。
        f)         kernel目录：包含内核管理的核心代码。与硬件相关代码放在arch/*/kernel目录下。
        g)        mm目录：包含了所有的内存管理代码。
        与硬件相关的内存管理代码位于arch/*/mm目录下。
        h)        scripts目录：包含用于配置核心的脚本文件。
        i)          lib目录：包含了核心的库代码，
        与硬件相关的库代码被放在arch/*/lib/目录下
        
        
    Linux内核配置、编译、安装
    
        1.       X86配置
        Linux内核的编译有两种方法，具体如下：
        a)        交互式：在内核顶层的目录下运行make config，
        按照提示一步一步的按照自己的需求对内核进行配置。
        b)        菜单式：在内核顶层的目录下运行make menuconfig，
        菜单式的按照自己的需求对内核进行配置。
        2.       X86编译
        Linux内核的编译要经过以下步骤，具体如下：
        1.        内核编译：linux内核的编译有以下两种方法。
        n  make zImage：编译出的内核小于512k（老版本内核）
        n  make bzImage：通用编译命令
        
        注：在以上两个命令中加V=1可查看编译过程中的详细信息
        2.        内核模块编译：执行make modules编译内核模块。
        3.        内核模块安装：执行make modules_install将编译好的内核模块复制到当前系统的/lib/modules下的**目录下。
        4.        内核模块打包：执行mkinitrd initrd-$version $version对内核模块进行打包，
        其中initrd-$version表示要打包为的文件的名字，
        $version表示要打包的目录即我们上一步生成的目录。
        3.       X86安装
        Linux内核的安装要经过以下步骤，具体如下：
        a)        拷贝内核：复制1编译出来的内核映像到启动目录cp arch/$cpu/boot/bzImage（1编译出来的bzimage）/boot/vmlinuz-$version
        b)        拷贝内核模块文件：执行cp initrd-$version（4生成的文件） /boot/ 将4生成的文件拷贝到boot下
        c)        修改启动配置文件：修改/etc/grub.conf文件
        


#### kickstart 配置文件说明

     kickstart是一个利用Anconda工具实现服务器自动化安装的方法；
     通过生成的kickstart配置文件ks.cfg，
     服务器安装可以实现从裸机到全功能服务的的非交互式（无人值守式）安装配置；
    ks.cfg是一个简单的文本文件，
    文件包含Anconda在安装系统及安装后配置服务时所需要获取的一些必要配置信息
    （如键盘设置，语言设置，分区设置等）；
    Anconda直接从该文件中读取必要的配置，
    只要该文件信息配置正确无误且满足所有系统需求，
    就不再需要同用户进行交互获取信息，
    从而实现安装的自动化；
    但是配置中如果忽略任何必需的项目，
    安装程序会提示用户输入相关的项目的选择，
    就象用户在典型的安装过程中所遇到的一样。
    一旦用户进行了选择，安装会以非交互的方式（unattended）继续。

    
    Kickstart文件可以存放于单一的服务器上,
    在安装过程中被独立的机器所读取.
    这个安装方法可以支持使用单一kickstart文件在多台机器上安装红帽企业Linux,
    这对于网络和系统管理员来说是个理想的选择.
    Kickstart给用户提供了一种自动化安装红帽企业Linux的方法.

    kickstart 安装可以使用本地光盘,本地硬盘驱动器,或通过 NFS,FTP,HTTP 来执行.
    要使用 kickstart,必须:
            1.创建一个kickstart文件.
            2.创建有kickstart文件的引导介质或者使这个文件在网络上可用.
            3.筹备安装树.
    
    创建kickstart配置文件的方式：
        文本编辑器编辑生成：vim
        用图形化界面配置：system-config-kickstat（需要安装system-config-kickstart.noarch包）
    
    系统引导后，会显示boot：命令提示符；如上，界面上会有各种模式操作提示；
    注：用户交互的文本安装方式中不能进行LVM的自定义配置，只能查看、接受默认设置；
    
    在boot：命令行里有用的几个项：
    lowres
        ：强制GUI安装时分辨率调低为640*480
    noipv6
        ：安装过程不支持ipv6网络
    noprobe
        ：不去自动检测硬件，而是提示用户；
    dd=
        ：通过网络加载设备驱动
    ks=
        ：指定kickstart文件的放置位置；
    另外还有ip、netmask、gateway、dns、vnc等选项；
    
    用ks选项被指定时kickstart文件位置时，Anaconda进入Kickstart安装模式；
    安装时获取kickstart文件的方式： 
    （1） boot：linux ks
    ks命令单独使用时，系统会尝试通过dhcp服务器配置网卡，并且从DHCP会话中获取kickstart配置文件的位置；
    在dhcp服务器dhcp配置文件中有kickstart文件位置说明，
    next-server关键字指向共享文件的NFS主机
    ，用filename关键字指向主机上的文件路径；
    如果没有filename关键字，
    则尝试在next-server关键字指向主机的/kickstart文件夹中找kickstart文件；

    下面是dhcp.conf文件中kickstart配置字段示例：
    # The following lines are examples of kickstart directives. 
            filename "/data/ks/ks.cfg"; 
            next-server 192.168.1.10; 
    # 注：上面部分需要写在subnet子段中；
    
    （2） boot：linux ks=url
    基于网络的文件服务器（网络服务器），获取配置文件，支持HTTP、FTP、NFS方式获取文件；例：
    ks=ftp://192.168.0.254/pub/kistart/ks.cfg
    ks=http:// 192.168.0.254/pub/kistart/ks.cfg
    ks=nfs:ip_addr:/path/to/ks.cfg
    
    （3） boot：linux ks=hd:device/path/to/your/kickstart_file
    基于本地的安装方式，需要依次指定设备名，路径，文件名等；例如：    
    文件在光盘中：ks=cdrom:/ks.cfg
    文件在软盘中：ks=floppy:/filedirectory/ks.cfg
    文件在硬盘中：ks=hd:/sdb1/myfile/ks.cfg
    文件也可被打包进initrd根文件系统中：ks=file:/ks.cfg
    
    kickstart文件结构介绍：
    1. 命令部分：配置系统的属性及安装中的各种必要设置信息
    2. %packages部分：
    设定需要安装的软件包及包组，Anaconda会自动解决依赖关系
    3. 脚本部分：用于定制系统，分为%pre部分在安装前运行，%post在安装后运行
    
        %pre 部分脚本作为一个bash shell脚本执行，在Kickstart文件解析后执行；
        %post 解析器默认为bash，可以自定义，缺省为chroot状态，也可指定非chroot状态；
        
    Kickstart文件中的主要项目及参数介绍：
        每个项目都由关键字来识别；关键字可跟一个或多个参数；
        如果某选项后面跟随了一个等号（=），它后面就必须指定一个值。 
        
        auth或authconfig(必需)
        bootloader(必需)
        keyboard(必需)
        lang(必需)
        rootpw(必需)
        
        
        
    
    
    Anaconda简介：
        Anaconda是Red Hat、CentOS、Fedora等Linux的安装管理程序。
        它可以提供文本、图形等安装管理方式，并支持Kickstart等脚本提供自动安装的功能。
        此外，其还支持许多启动参数，
        熟悉这些参数可为安装带来很多方便。该程序是把位于光盘或其他源上的数据包，
        根据设置安装到主机上的一个程序；
        为实现该定制安装，其提供一个定制界面，
        可以实现交互式界面供用户选择配置（如选择语言，键盘，时区等信息）；
        
        Anaconda支持的管理模式：
        Kickstart提供的自动化安装
        对一个RedHat实施upgrade
        Rescuse模式对不能启动的系统进行故障排除；
        要进入安装步骤，需要先有一个引导程序引导启动一个特殊的Linux安装环境系统；引导有多种方式：
        基于网络方式的小型引导镜像，需要提供小型的引导镜像；
        U盘引导，通过可引导存储介质中的小型引导镜像启动安装过程；
        基于PXE的网络安装方式，要提供PXE的完整安装环境；
        其他bootloder引导（如GRUB）
        可用的安装方式：
        本地CDROM
        磁盘驱动器
        NFS映像
        FTP
        HTTP


#### SELinux简介
Linux的一个扩张强制访问控制(MAC)安全模块，是 Linux® 上最杰出的新安全子系统,进程只能访问那些在他的任务中所需要文件。

MAC情况下的安全策略完全控制着对所有资源的访问。这是MAC和DAC本质的区别。
SELinux提供了比传统的UNIX权限更好的访问控制。

    基本概念
    2.1 主体
          通常指用户，或代表用户意图运行进程或设备。
          主体是访问操作的主动发起者，
          它是系统中信息流的启动者，
          可以使信息流在实体之间流动。
    2.2 客体
          通常是指信息的载体或从其他主体或客体接收信息的实体。
          主体有时也会成为访问或受控的对象，
          如一个主体可以向另一个主体授权，
          一个进程可能控制几个子进程等情况，
          这时受控的主体或子进程也是一种客体。
    2.3 访问控制分类
          客体不受它们所依存的系统的限制，可以包括记录、数据块、存储页、存储段、文件、目、目录树、
          库表、邮箱、消息、程序等，还可以包括比特位、
          字节、字、字段、变量、处理器、通信信道、
          时钟、网络结点等。
    2.3.1 自主访问控制DAC
          管理的方式不同就形成不同的访问控制方式。
          一种方式是由客体的属主对自己的客体进行管理，
          由属主自己决定是否将自己客体的访问权或部分访问权授予其他主体，
          这种控制方式是自主的，
          我们把它称为自主访问控制（Discretionary Access Control——DAC）。在自主访问控制下，
          一个用户可以自主选择哪些用户可以共享他的文件。Linux系统中有两种自主访问控制策略，
          一种是9位权限码（User-Group-Other），
          另一种是访问控制列表ACL（Access Control List）。
    2.3.2  强制访问控制MAC
          强制访问控制（Mandatory Access Control——MAC），用于将系统中的信息分密级和类进行管理，
          以保证每个用户只能访问到那些被标明可以由他访问的信息的一种访问约束机制。
          通俗的来说，在强制访问控制下，
          用户（或其他主体）与文件（或其他客体）都被标记了固定的安全属性（如安全级、访问权限等），
          在每次访问发生时，系统检测安全属性以便确定一个用户是否有权访问该文件。
          其中多级安全（MultiLevel Secure, MLS）就是一种强制访问控制策略。


    3. 控制切换
        Fedora core 5 里的/etc/sysconfig/selinux标准设置如下：
    
        # This file controls the state of SELinux on the system.
        
        # SELINUX= can take one of these three values:
        
        # enforcing - SELinux security policy is enforced.
        
        # permissive - SELinux prints warnings instead of enforcing.
        
        # disabled - SELinux is fully disabled.
        
        SELINUX=enforcing
        
        
        
        # SELINUX=disabled
        
        # SELINUXTYPE= type of policy in use. Possible values are:
        
        # targeted - Only targeted network daemons are protected.
        
        # strict - Full SELinux protection.
        
        SELINUXTYPE=targeted

    3.1 SELINUX
    SELINUX 有【disabled】、【permissive】、【enforcing】3种选择。
    • disabled：不启用SELINUX功能
    • permissive：SELINUX有效，但是即使你违反了策略，
    它让你继续操作，但是把你的违反的内容记录下来。
    在我们开发策略的时候非常的有用。
    相当于Debug模式。
    • enforcing：当你违反了策略，你就无法继续操作下去。
    
    3.2 SELINUXTYPE
    目前主要有2大类，【targeted】和【strict】。
    • targeted：它是红帽子开发的targeted，它只是对于主要的网络服务进行保护，比如apache, sendmail, bind, postgresql等，不属于那些domain的就都让他们在unconfined_t里，可导入性高，
    可用性好但是不能对整个系统进行保护。
    • strict：是由NAS开发的，能对整个系统进行保护，但是设定复杂，
    但是只要掌握一些基本的知识，还是可以玩得动的。
    我们除了在/etc/sysconfig/selinux设它有效无效外，在启动的时候，
    也可以通过传递参数selinux给内核来控制它。（Fedora 5默认是有效）
    
    4. 基本操作
    4.1.1  ls命令
    在命令后加个 -Z 或者加 -context 
    4.1.2 chcon
    更改文件的标签
    4.1.3 restorecon
    当这个文件在策略里有定义时，可以恢复原来的 文件标签。
    
    4.1.4 setfiles
    跟chcon一样可以更改一部分文件的标签，不需要对整个文件系统重新设定标签。 
    
    
    4.1.5 fixfiles
     一般是对整个文件系统的， 后面一般跟 relabel，对整个系统 relabel后，一般我们都重新启动。如果，在根目录下有.autorelabel空文件的话，每次重新启动时都调用 fixfiles relabel 
    
    4.1.6 star
    就是tar在SELinux下的互换命令，能把文件的标签也一起备份起来。
    4.1.7 cp
    可以跟 -Z,--context=CONTEXT 在拷贝的时候指定目的地文件的security context
    
    4.1.8 find
    可以跟 –context 查特定的type的文件。
    find /home/fu/ --context fu:fu_r:amule_t -exec ls -Z {} \:
    
    4.2 进程domain的确认
      程序现在在那个domain里运行，我们可以在ps 命令后加 －Z进行查看：
    4.3 ROLE的确认和变更
    命令id能用来确认自己的 security context。
    
    4.4 模式切换
    4.4.1 getenforce
    得到当前的SELINUX值。
    4.4.2 setenforce
    更改当前的SELINUX值 ，后面可以跟 enforcing,permissive 或者 1,0。 
    4.4.3 sestatus
    显示当前的 SELinux的信息。
    
    4.5 其他重要命令
    4.5.1 audit2allow
    很重要的一个以python写的命令，主要用来处理日志，把日志中的违反策略的动作的记录，转换成 access vector，对开发安全策略非常有用。
    在refpolicy里，它的功能比以前有了很大的扩展。 
    4.5.2 checkmodule
    
    编译模块。
    checkmodule -m -o local.mod local.te
    4.5.3 semodule_package
    创建新的模块。
    semodule_package -o local.pp -m local.mod
    4.5.4 semodule
    可以显示、加载、删除模块，加载的例子：        
    semodule -i local.pp
    
    
    4.5.5 semanage
    这是一个功能强大的策略管理工具，有了它即使没有策略的源代码，也是可以管理安全策略的。
    因为本文主要是介绍用源代码来修改策略的，详细用法大家可以参考它的man页。


#### sed
   sed（Stream Editor）是一个行编辑工具。
  基本处理流程:
    每次读入文本文件的一行到内存中的模式空间中，
    在模式空间中处理后将处理的结果输出，
    默认会打印到屏幕上。因此，默认情况下，不会改变原文件的内容。
    
    
    
    正则表达式：
    char            字符本身就匹配字符本身，如/abc/就是定位包含abc的行。
    
    *            匹配前面表达式出现了0或若干次，如/a*/可以帮你找到a,aa,aaa,... ...等等。
    
    \+            类似于*，但匹配前面表达式的1次或多次，这属于扩展正则表达式。
    
    \?            类似于*，但匹配前面表达式的0次或1次，这属于扩展正则表达式。
    
    \{i\}         类似于*，但匹配前面表达式的i次(i为整数)，如：a\{3\}可以帮你找到aaa。
    
    \{i,j\}       匹配前面表达式的i到j次，如a\{1,2\}可以帮你找到a或aa或aaa。
    
    \{i,\}        匹配前面表达式至少i次。
    
    \( \)         将\( \)内的模式存储在保留空间。最多可以存储9个独立子模式，可
    通过转义\1至\9重复保留空间的内容至此点。
    
    \n            转义\1至\9重复保留空间的内容至此点。
    例：test.txt的内容为ssttss    
    grep '\(ss\)tt\1' test.txt            \1表示将ss重复在tt后面
    该grep命令等同于grep  ssttss  test.txt    在test.txt文件中找ssttss
    
    .            (点)匹配任意字符。
    
    ^            匹配行的开始，如^test    将匹配所有以test开始的行。
    
    $            匹配行的结尾，如test$    将匹配所有以test结尾的行。
    
    []           匹配括号中的任意单个字符，如a[nt]    将匹配an或at。
    
    [^]          匹配不包含在[]中的字符，如[^a-z]    将匹配除a-z以外的字符。
    
    \n           匹配换行符。
    
    \char        转义特殊字符，如\*，就是匹配字面意义上的星号。

    sed 命令的使用
    基本语法：sed [options]... '地址定位 编辑命令' FILE...
    地址定位的方法：
    1、行定位：
        start_line[,end_line]
    2、模式匹配
       /pattern1/,/pattern2/ 第一次被pattern1匹配到的行开始，到第一次被pattern2匹配到的行结束之间的所有行
        /pattern/ 被pattern匹配到的行
    3、没有地址定界，代表的是全文。
    常用参数：
    -n: 静默模式，不显示模式空间中的内容
    -r: 支持使用扩展正则表达式
    -i: 修改原文件；
    -e: sed -e "" -e "" -e "", sed "{COM1;COM2;COM3}"
    -f: -e的功能差不多，只是将多个COM写到文件中区。
    编辑命令：命令可在之前加!取反
    
    p：打印    
    例如：打印 /etc/fstab 文件的 3 到 5 行
    sed '3,5p' /etc/fstab
    一般 p 命令与 -n 参数一起使用，才能达到想要的效果。
    
    d: 删除
    sed '/^#/./^UUID/d' /etc/fstab
    删除第一次以#开头的行，到第一次出现UUID开头的行
    
    i \text: 行上方，text即为插入的内容
    a \text: 行下方，text即为插入的内容
    
    r /path/from/some_file: 把符合条件的行读到指定文件中
    
    w /path/to/some_file: 把符合条件的行保存至指定的文件中
    
    =: 显示符合条件行的行号
    
    s///: s@@@ 查找替换
    g,i：g是全文替换，i忽略大小写
    
    
   &ldquo;表示中文输入法下的双引号
   &quot; 表示英文输入法下的双引号
   &lt;表示小于号 
   &gt;表示大于号
   &nbsp;表示空格
   
    sed的\( \)的功能可以记住正则表达式的一部分，
    其中，\1为第一个记住的模式即第一个小括号中的匹配内容，\2第二记住的模式，
    即第二个小括号中的匹配内容，sed最多可以记住9个。
    
    echo I am oldboy teacher.
    本例我们要保留oldboy把其他内容删除。
    
    sed 's#^.*am \([a-z].*\) tea.*$#\1#g' test.txt 
    
    命令说明：#命令说明中的□代替空格

    a）^.*am□     -->这句的意思是以任意字符开头到am□为止，匹配文件中的&ldquo;I am□&rdquo;字符串,
    
    b)\([a-z].*\)□-->这句的外壳就是括号\(\），里面的[a-z]表示匹配26个字母的任何一个，[a-z].* 合起来就是匹配任意多个字符，本题来说就是匹配oldboy字符串，
    由于oldboy字符串是需要保留的,因此用括号括起来匹配，
    后面通过\1来取oldboy字符串。
    
    c)□tea.*$     -->表示以空格tea起始任意字符结尾，
    实际就是匹配oldboy字符串后，紧接着的字符串&ldquo;□teacher.&rdquo;.
    
    d)后面被替换的内容中的\1就是取前面的括号里的内容了，
    也就是我们要的oldboy字符串。
    
    取字符串的技巧 
        [root@oldboy ~]# stat /ett|sed -n '4p'
        
        Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
        
        1）处理需要的目标（获取的字符串如本文开篇的644）前的字符串一般用以..开头(^.*)来匹配开头，匹配的目标前的结尾写上实际的字符，如：&ldquo;^.*(0&rdquo;表达式匹配&ldquo;Access: (0&rdquo;，说到这先给个例子：
        [root@oldboy ~]# stat /ett|sed -n '4p'|sed 's#^.*(0##g'
        
        644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
        
        大家看到了吧，644前面的部分已经被删除了。
        2）而处理目标后的内容一般在紧接着目标后匹配的开头写上实际的字符，而结尾是用以...结尾（.*$）来匹配，如&ldquo;/-r.*$&rdquo;表达式匹配&ldquo;/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)&rdquo;在来个例子：
        [root@oldboy ~]#  stat /ett|sed -n '4p'|sed 's#/-r.*$##g'
        
        Access: (0644
        
        哈哈，这次644后面的又被删除了。
        
        有一点要注意，就是实际字符的选取最好要唯一，
        因为正则表达式是贪婪的，
        它总是尽可能的匹配更远的符合匹配的内容。
        另外不要落了字符串中的空格。
        
        合起来就是答案了。
        
        [root@oldboy ~]# stat /ett|sed -n '4p'|sed 's#^.*(0##g'|sed 's#/-r.*$##g'
        
        644
        
        先删除目标前面的，在删除目标后的，
        这是应用了两次sed，每次只匹配了一半，第一次匹配目标前，
        第二次匹配目标后，那么能不能直接匹配整行字符串呢？
        
        所有的台阶都上来后，接下来就引出本文的主题。      
        
    [root@oldboy ~]# stat /ett|sed -n 's#^.*(0\([0-7].*\)\/-.*$#\1#gp'            
    
    644
    
    命令说明：
    
    a)&ldquo;^.*(0&rdquo;匹配目标前的内容，
    这个不用再解释了吧，这里匹配的是&ldquo;Access: (0
    b)&ldquo;\/-.*$&rdquo; 匹配目标后的内容，这个也不用再解释了吧，
    这里匹配的是&ldquo;/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)&rdquo;
    
    c)这中间的644就是我们要打印的结果，
    所以要用括号括起来呦，于是\(\)就不用再解释了，
    [0-7].*就是匹配644，匹配到斜线/前为止，虽然不是精确匹配但是这里够用了。因此，&ldquo;\([0-7].*\)&rdquo;匹配的就是&ldquo;644&rdquo;了，
    前面的台阶已经讲过了，匹配模式中\(\)的内容，
    可以用\1来取出（如果是第二个扩号就是\2,类推），
    
    
    请执行命令取出linux中eth0的IP地址
        [root@oldboy ~]# ifconfig eth0|grep 'inet addr'|awk -F ":" '{print $2}'|awk '{print $1}'
        10.0.0.162
        
        [root@oldboy ~]# ifconfig eth0|grep 'inet addr'|awk -F '[ :]' '{print $13}'
        192.168.1.186
        
        [root@oldboy ~]# ifconfig eth0 |awk -F '[ :]+' 'NR==2 {print $4}' 
        10.0.0.185


    提示：本题NR是行号，分隔符+号匹配，[]里一个或多个任意一个分隔符，
    这里就是匹配一个或多个冒号或空格。
    
    1）awk -F 后面跟分隔符&lsquo;[空格:]+&rsquo;，
    其中[空格:]多分隔符写法，意思是以空格或冒号做分隔，
    后面的"+"号是正则表达式，
    意思是匹配前面空格或冒号，两者之一的1个或1个以上。
    
    2）NR==2和sed -n "2p",相当，意思都是选择第几行,例：
    
    [root@oldboy ~]# ifconfig eth0|awk NR==2
    
              inet addr:10.0.0.185  Bcast:10.0.0.255  Mask:255.255.255.0
              
    3）指定awk -F '[ :]+'分隔符后，不同字符串被分隔的列依次为：
    
              
    
    inet
    addr
    10.0.0.185Bcast:10.0.0.255  Mask:255.255.255.0
    
    第一列
    第二列
    第三列
    第四列
    后面忽略不计。
    
    
    4）整个答案awk部分意思是，通过NR==2取出第二行，
    然后，通过-F '[-:]+多分隔符正则匹配，
    然后通过｛print $4｝打印出第四列 
    


#### shell-脚本-字符串切片

    
    ${#var} : 返回字符串变量var的长度
    [root@centos6 ~]# alpha=`echo {a..z} |tr -d " "`  \\创建一个变量将26个字母赋值进去，并且不要空格
    [root@centos6 ~]# echo $alpha
    abcdefghijklmnopqrstuvwxyz
    [root@centos6 ~]# echo ${#alpha}  \\查看变量的字符有多少个
    26
    
    ${var:offset}：返回字符串变量var中从第offset个字符后（不包括第offset个字符）的字符开始，
    到最后的部分，offset的取值在0到${#var}-1之间（bash4.2后，允许为负值）
    [root@centos6 ~]# echo ${alpha:3} //跳过前3个显示后面全部
    defghijklmnopqrstuvwxyz
    
    ${var:offset:number}:返回字符串变量var中从第offset个字符后（不包括第offset个字符）的字符开始，
    长度为number的部分
    [root@centos6 ~]# echo ${alpha:3:4}   \\跳过3个取4个
    defg
    
    ${var: -length}:取字符串的最右侧几个字符 
    注意：冒号后必须有一个空白字符
    [root@centos6 ~]# echo ${alpha: -3}
    xyz
    
    
    ${var:offset:-length}: 从最左侧跳过offset字符，一直向右取到距离最右侧length个字符之前的内容
    [root@centos6 ~]# echo ${alpha:3:-4}   //centos6不支持这种写法
    -bash: -4: substring expression < 0
    [root@centos7 ~]# echo ${alpha:3:-4}  //7上是可以的，取前3个到倒数第4个之间的
    defghijklmnopqrstuv
    
    ${var: -length:-offset}:先从最右侧向左取到length个字符开始，
    在向右取到距离最右侧offset个字符之间的内容 
    注意：-length前的空格
    [root@centos7 ~]# echo ${alpha: -3:-2}  \\取倒数3个但是最后又2个不要
    x
    [root@centos7 ~]# echo ${alpha: -5:-2}  \\前面的数字一定要比后面的数字大
    vwx
    
     基于模式取子符串 
    ${var#*word}:其中word可以是指定任意字符
    功能：自左向右，查找var变量所存储的字符串中，
    第一次出现的word,删除字符串开头至第一次出现word字符之间的所有字符 
    bash 
    [root@centos7 ~]# echo ${alpha#*v} 
    wxyz 
    [root@centos7 ~]# echo ${alpha#*w} 
    xyz    

    ${var##*word}:同上，贪婪模式，不同的是，删除的是字符串开头至最后一次由work指定的字符之间的所有内容 
    bash 
    [root@centos7 ~]# line=`getent passwd root` 
    [root@centos7 ~]# echo $line 
    [root@centos7 ~]# echo ${line##*root} 
    :/bin/bash 
    
    file="var/log/messages“
    ${file#*/}: log/messages
    ${file##*/}: messages

    ${var%word*}:其中word可以是指定的任意字符； 
    功能：自右而左，查找var变量所存储的字符串中，
    第一次出现的word，
    删除字符串最后一个字符向左至第一次出现word字符之间的所有字符 
    bash 
    [root@centos7 ~]# echo ${line%root*} 
    root:x:0:0:root:/ 
    
    ${var%%word*}:同上，只不过删除字符串最右侧的字符向左直至最后一次出现word字符之间的所有字符
    [root@centos7 ~]# echo ${line%%root*}
    
    url=http://www.magedu.com:80
    ${url##*:} 80
    ${url%%:*} http
    
    查找替换
    
    ${var/pattern/substr}:查找var所表示的字符串中，第一次被pattern所匹配到的字符串，以substr替换之
    [root@centos7 ~]# echo ${line/root/admin}
    admin:x:0:0:root:/root:/bin/bash
    
    ${var//pattern/substr}:查找var所表示的字符串中，所有能被pattern所匹配到的字符串，以substr替换之
    [root@centos7 ~]# echo ${line//root/admin}
    admin:x:0:0:admin:/admin:/bin/bash
    
    ${var/#pattern/substr}:查找var所表示的字符串中，行首被pattern所匹配到的字符串，以substr替换之
    [root@centos7 ~]# echo ${line/#root/admin}
    admin:x:0:0:root:/root:/bin/bash
    
    ${var/%pattern/substr}:查找var所表示的字符串中，行尾被pattern所匹配到的字符串，以substr替换之
    [root@centos7 ~]# echo ${line/%bash/admin}
    root:x:0:0:root:/root:/bin/admin
    
    查找并删除
    ${var/pattern}：删除var所表示的字符串中第一次被pattern所匹配到的字符串
    [root@centos7 ~]# echo ${line/root}
    :x:0:0:root:/root:/bin/bash
    [root@centos7 ~]# echo ${line/root/}   //查找替换也可以就是查找用空代替
    :x:0:0:root:/root:/bin/bash
    
    ${var//pattern}:删除var所表示的字符串中所有被pattern所匹配到的字符串
    [root@centos7 ~]# echo ${line//root}
    :x:0:0::/:/bin/bash
    
    ${var/#pattern}:删除var所表示的字符串中所有以pattern为行首所匹配到的字符串
    [root@centos7 ~]# echo ${line/#root}
    :x:0:0:root:/root:/bin/bash
    
    ${var/%pattern}:删除var所表示的字符串中有所以pattern为行尾所匹配到的字符串
    [root@centos7 ~]# echo ${line/#root}
    :x:0:0:root:/root:/bin/bash
    
    字符大小写转换
    ${var^^}:把var中的所有小写字母转换为大写
    [root@centos7 ~]# echo ${line^^}
    ROOT:X:0:0:ROOT:/ROOT:/BIN/BASH
    [root@centos7 ~]# echo ${line}   //但是变量里是没有变的只是显示变了
    root:x:0:0:root:/root:/bin/bash
    
    ${var,,}:把var中的所有大写字母转换为小写
    
    变量赋值
![image](F716FF33081749899658A622ECC5374C)



    [root@centos7 ~]# var=${line-"haha"}  //当line有值时var的值就是line的值
    [root@centos7 ~]# echo $var
    root:x:0:0:root:/root:/bin/bash
    [root@centos7 ~]# line=""
    [root@centos7 ~]# var=${line-"haha"}  //当line的值为空时var的值就是line的值
    [root@centos7 ~]# echo $var         
    
    [root@centos7 ~]# unset line
    [root@centos7 ~]# var=${line-"haha"}   //当line没有定义时var的值就是haha
    [root@centos7 ~]# echo $var         
    haha
        
    var是一个变量，str也是一个变量，expr是一个表达式，
    可以是字符串，这里表达的意思是当str在没有定义或定义为空或定义了，对var变量的影响


#### GNU/Linux awk命令用法详解

    AWK命令格式如下：
    awk [options] 'program' input-file1 input-file2 ...
    或者
    awk [options] -f program-file input-file1 input-file2 ...
    
    第一种格式中，awk从file中获取输入流，然后执行单引号内的程序。
    第二种格式则是从文件program-file中获取将要执行的程序。
    上述AWK命令的program部分的结构可分为三块：BEGIN、BODY和END。
    
    BEGIN：在AWK命令的一开始执行的动作，它只执行一次，
    可以把变量初始化放在这里。注意，BEGIN部分是可选的，
    并且一个AWK命令中可以有多个BEGIN块。
    另外，如果有-v选项的赋值操作，则-v的操作在BEGIN之前。
    BEGIN块的写法为：BEGIN{…}
    BODY：程序主体，对输入流的每一行执行动作。
    如果存在BEGIN或END，则这部分是可选的。
    一个AWK命令中可以有多个BODY块。
    BODY块的写法为：{…}
    END：在AWK命令的最后执行的动作，它只执行一次。
    注意，END部分是可选的，并且一个AWK命令中可以有多个END块。另外，使用END { }块时，awk的file参数不能省略。
    END块的写法为：END{…}
    

    [root@ubuntu]awk_test:$ awk 'BEGIN{print "Output start!"}; {print}; END{print "Output done!"}' awk_test.txt 
    
    可以看到这条命令在命令的开始打印一行输出“Output start!”，
    然后对文件中的每一行内容执行print操作，最后又打印出一行输出“Output done!”。
    
    从句法结构上来讲，program由一条条规则组成，
    每条规则由模式和动作组成，即模式匹配后执行相应的动作。动作放在花括号内以与模式区分。
    所以，program一般的格式是这样的：
    pattern { action }
    pattern { action }
    …
    那么，下面这条命令（打印长度大于80字符的行）：
    awk 'length($0) > 80' data
    可以看到这条awk命令只有pattern而没有action部分。
    如果没有action部分，则执行默认动作：打印整个record。
    也可以把program写到文件里（不用加单引号），
    通过AWK的第二种格式来执行。
    [root@ubuntu]awk_test:$ cat progfile 
    BEGIN{print "Output start!"};
    {print};
    END{print "Output done!"};
    [root@ubuntu]awk_test:$ awk –f progfile awk_test.txt
    
    为了方便后期维护，建议将AWK的程序文件以.awk作为后缀名。
    另外，可以利用Shell的#!机制，将progfile内容改为：
    
    #!/usr/bin/awk –f
    BEGIN{print "Output start!"};
    {print};
    END{print "Output done!"};
    这样的话，执行./progfile awk_test.txt即可。
    
    注：使用#!机制的话，执行的shell命令./progfile实际上是执行："#!后面的命令" +"./progfile脚本" + "./progfile脚本的参数"。
    另外，这种写法，awk后面最多只能跟一个参数；
    并且ARGV[0]的值在不同系统上可能表现不同
    ，比如可能被解释为awk或/usr/bin/awk或./progfile。
    比较特别的，如果要在program内使用或打印单引号，
    可以用其ASCII码'\47'表示。或者把程序写在文件中，
    这样就不用担心单引号和program外围的单引号混淆的问题。注意，在单引号中，反斜杠后接一个字符，
    会被解释成这个字符的字面意思，
    即和不加反斜杠是一样的含义。    
    
    你也可以通过下面两种方法来打印单引号：
    awk 'BEGIN { print "Here is a single quote <'"'"'>" }'
    或
    awk 'BEGIN { print "Here is a single quote<'\''>" }' 
    
    AWK选项
     -f program-file 执行文件中的程序，
     
         环境变量AWKPATH用来指定-f的搜索路径，
         如果不指定AWKPATH，则默认搜索“.:/usr/local/share/awk”，可以通过修改AWKPATH或ENVIRON["AWKPATH"]来修改搜索路径，
         每个路径之间用冒号隔开（.或::都可以表示当前路径）。
         如果-f选项后面跟的是包含“/”的文件名，
         就不会去额外搜索路径了。

    -v var=value 变量赋值，在BEGIN之前进行。
    
         例如，定义变量name，并赋值为“jason”：
        [root@ubuntu]awk_test:$ awk -v name=jason 'BEGIN{printf("name=%s\n", name)}'
        name=jason   

    注意变量的引用不需要加“$”，实际上AWK的语法很多跟ANSI C的语法类似。“$”在AWK中是用来引用field的
    
    -F fs 使用fs作为分隔符（默认是空格）
    例如，打印/etc/passwd文件中的用户名一列：
    [root@ubuntu]awk_test:$ cat /etc/passwd | awk -F ':' '{print $1}'
    
    --profile[=prof_file] 生成awk命令的profile文件
    这个选项以优雅的格式将awk命令保存到文件，如果不指定文件名，则默认为awkprof.out。
    
    [root@ubuntu]awk_test:$ awk --non-decimal-data --profile '{sum+=($1)}; END{print sum}' number.txt


    AWK的变量、Records和Fields
    
    
    AWK会将“var=value”形式的参数认为是变量赋值，
    例如下面这个命令，awk将“var=2”和“var=1”认为是给var赋值，
    而不是一个文件名，这个过程在awk顺序处理参数列表时进行的。
    awk 'var == 1 {print 1} var == 2 { print 2}'  var=2 awk_test2.txt var=1 awk_test1.txt


    Records
    一个record就是awk认为的一行输入，对一个输入流，默认以换行符分隔。不过可以通过内置变量RS来修改。例如，把RS赋值为Jan07：
    [root@ubuntu]awk_test:$ awk -v RS=Jan07 '{print}' awk_test.txt 


    USER       PID %MEM    VSZ   RSS STAT START   TIME COMMAND
    
    root         1  0.1   3652  1916 Ss   
       0:03 /sbin/init
    root         2  0.0      0     0 S    
       0:00 [kthreadd]
    root         3  0.0      0     0 S    
       0:02 [ksoftirqd/0]
    root        26  0.0      0     0 S    
       0:40 [kswapd0]
    user       495  0.1   3588  1092 Ss   
       0:00 /sbin/udevd --daemon
    user       860  0.0   3584   908 S    
       0:01 /sbin/udevd --daemon
    user      1137  0.0   4520   776 S    
       0:00 smbd -F
    user      1550  0.1   4521  1816 Ss   
       0:15 nmbd -D   


    看效果就知道是什么意思了。注意“Jan07”本身没有打印出来。
    如果RS被赋值为单个字符，则这个字符就是records的分隔符；如果被赋值为多个字符，那RS实际上是一个正则表达式。
    如果RS被赋值为空，则以空行作为record的分隔符。
    
    fields
    AWK会把一个record再分隔成一个个的field依次进行处理，
    以变量FS的值作为分隔符。
    如果FS被赋值为单个字符，则这个字符就是fields的分隔符；如果被赋值为多个字符，那就是一个正则表达式；
    如果FS是空，则record中的每个字符都是分隔符。
    注意，如果FS是空格，则多个空格、tab和换行，
    都会被认为是分隔符。
    一个record中的各个field可以通过各自的位置来引用，
    依次为$1，$2，$3，...。而$0表示整个record。
    如果给FIELDWIDTHS变量赋值为一个数值列表（以空格隔开），如下面的例子，record就会按照字符串长度来分隔fields而不是按照FS的值。这时FS变量会被忽略。
    

    [root@ubuntu]awk_test:$ awk 'BEGIN{FIELDWIDTHS="2 4 4 4"}{print $1"||"$2"||"$3"||"$4}' awk_test.txt 
    US||ER  ||    || PID
    ro||ot  ||    ||   1
    ro||ot  ||    ||   2
    ro||ot  ||    ||   3
    ro||ot  ||    ||  26
    us||er  ||    || 495
    us||er  ||    || 860
    us||er  ||    ||1137
    us||er  ||    ||1550  
    
    
    如果给FS重新赋值，就会切回到按照FS分隔fields。
    NF变量用于获取当前record共分了多少了fields。
    你可以给NF、$0以及某个field赋值，
    那么相应record会更新并重新划分fields。例如：
    

    [root@ubuntu]awk_test:$ awk '{NF-=1; if (FNR!=1) $1="json"; print}' awk_test.txt 
    USER PID %MEM VSZ RSS STAT START TIME
    json 1 0.1 3652 1916 Ss Jan07 0:03
    json 2 0.0 0 0 S Jan07 0:00
    json 3 0.0 0 0 S Jan07 0:02
    json 26 0.0 0 0 S Jan07 0:40
    json 495 0.1 3588 1092 Ss Jan07 0:00 /sbin/udevd
    json 860 0.0 3584 908 S Jan07 0:01 /sbin/udevd
    json 1137 0.0 4520 776 S Jan07 0:00 smbd
    json 1550 0.1 4521 1816 Ss Jan07 0:15 nmbd
    
    这个例子中，把NF减1，则原来的每个record的最后一个field就没有打印出来。
    FNR表示当前已经输入了几个record，
    例子中将文件除了第一行之外的第一个field的值都改成了“jason”。
    
    
    内置变量
    AWK有一些内置的全局变量可以直接使用，会使编程更方便。

    ARGC
    Awk命令行的参数个数（不包括选项和选项的值以及program部分），awk自身是第0个参数。
    
    ARGV
    Awk的参数列表，共ARGC个元素。
    
    ARGIND
    当前正在处理的文件处于ARGV数组中的index。
    
    BINMODE
    是否使用binmode打开文件。1("r")，2("w")，3("rw")分别表示输入文件、输出文件、所有文件需要使用binmode打开。
   
    CONVFMT
    数字的格式化类型，默认是"%.6g"。
    
    ENVIRON
    保存当前所有环境变量的数组。
    这个数组是以环境变量名作为下标的，
    例如ENVIRON["HOME"]的值是"/root"。
   
    ERRNO
    当getline或close失败，ERRNO会保存字符串形式的错误描述。
   
    FIELDWIDTHS
    一个以空格分隔的数值列表，设置这个值后，FS失效，record改以字符长度来划分fields。
   
    FILENAME
    Awk当前处理的文件的名字。注意，在BEGIN{ }中由于还没开始处理文件，FILENAME是空（除非被getline设置）。
    
    FNR
    表示正在处理的文件目前已经输入了几个record。
   
    FS
    Field分隔符，默认是空格。
    
    IGNORECASE
    是否忽略大小写，默认是0。用于控制正则表达式和字符串操作，例如会影响records和fields的分隔符。注意，数组下标不受影响。
    
    LINT
    如果为true，则对可能不兼容的awk命令提示warning，如果为fatal，则提示为错误。默认是0：不提示。
   
    NF
    当前record中的fields的数目。
   
    NR
    已经处理的record的数目（包括当前record）。
   
    OFMT
    输出中数字的格式，默认是“%.6g”。
   
    OFS
    输出中fields的分隔符，默认是空格。例如下面的命令，
    就会以” : ”来打印fields（注意$1 $2 $3之间要加逗号）：
    awk -v OFS=" : " '{print $1,$2,$3}' awk_test.txt
   
    ORS
    输出中records的分隔符，默认是新行。
    
    PROCINFO
    一个包含正在执行的awk命令自身信息的数组，以元素名作为下标，例如PROCINFO[“egid”]，PROCINFO[“pid”]等，
    下面的命令可以列举所有的元素：
    awk '{for (var in PROCINFO) print var;}' awk_test.txt
   
    RS
    records分隔符，默认是新行。
   
    RT
    record的结束符，通常被赋值为RS。
    
    RSTART
    match()匹配成功的子串的第一个字符在原始字符串中的index（从1开始）。0表示匹配失败。
   
    RLENGTH
    match()匹配成功的子串的长度。-1表示匹配失败（可以匹配空串，此时长度为0）。
   
    SUBSEP
    数组下标分隔符，默认为\034即0x1C的ASCII符号。在访问多维数组时有用。
   
    TEXTDOMAIN
    文本域，本地化的时候用到。 
    
    对于FS，强调一个‘FS =" "’和‘FS = "[ \t\n]+"’的不同，这两种FS都是将多个空格、tab或newline作为分隔符。
    但是前者会先将record中的前导空格和尾部空格剥掉，
    再决定如何分割fields。例如下面两条命令的结果就不同：
    
  
    [root@ubuntu]awk_test:$ echo ' a b c d ' | awk '{ print $2 }'
    b
    
    [root@ubuntu]awk_test:$ echo ' a b c d ' | awk 'BEGIN { FS = "[ \t\n]+" }{ print $2 }'
    A  
    
    也可以据此特征来删除前导空格：
    [root@ubuntu]awk_test:$ echo '   a b c d' | awk '{ print; $2 = $2; print }'
       a b c d
    a b c d
    
    这条命令通过$2=$2，让awk重新划分fields，
    由于FS模式是空格，这时就会先把前导空格和尾部空格剥掉
    

    数组
    AWK中的数组是关联数组（键值对的形式），
    数组下标放在方括号“[ ]”内，数组下标可以是数值或字符串。
    可以用(expr, expr ...)来模拟多维数组的下标。例如：
    
    
    i = "A"; j = "B"; k = "C"
    x[i, j, k] = "hello, world\n"
    上面定义了数组x下标为"A\034B\034C"的值为"hello,world\n"。 
    操作符“in”可以用来测试下标是否存在，例如：
    if (val in array)
    print array[val]
    
    也可以用“in”来遍历数组：
    下面两个例子，前者定义了一个一维数组，
    数组下标是[“a,b”]的元素。后者用来定义多维数组，
    例子中定义了三维数组中下标为[”a”][“,”][”b”]的元素。
    [root@ubuntu]awk_test:$ awk 'BEGIN{array["a"",""b"]=1;for(i in array) print i}'
    a,b
    [root@ubuntu]awk_test:$ awk 'BEGIN{array["a",",","b"]=1;for(i in array) print i}'
    a\034,\034b
    
    记住，数组的下标总是字符串，如果以数值为下标，
    也会转换成字符串，也就是说，a[17]的下标是"17"，
    并且和a[021]、a[0x11]是相同的下标。
    使用delete可以删除一个数组元素，例如delete array[1]，也可以delete array来删除整个数组。
    
    
    模式（patterns）
    AWK的模式可以是下面的其中一种：
      BEGIN
      END
      /regular expression/
      relational expression
      pattern && pattern
      pattern || pattern
      pattern ? pattern : pattern
      (pattern)
      ! pattern
      pattern1, pattern2
      
    BEGIN和END是两个比较特殊的模式，他们不对输入做模式匹配，
    因为BEGIN和END分别是在处理输入之前和之后。
    多个BEGIN块会被merge成一个BEGIN块，
    这个BEGIN里面的语句会在开始读取输入之前执行；
    多个END块会被merge成一个END块，
    这个END里面的语句会在处理完所有输入之后执行（或者执行exit）。注意， BEGIN和END必须是独立的，
    不能糅合在其他pattern表达式中。并且BEGIN和END不能没有action部分，
    也就是说BEGIN和END后面不能没有花括号{ }。
    
    对于/regular expression/模式，会对所有匹配到该正则表达式的record执行其关联的语句。
    例如，打印包含“root”的行：
    awk '/root/ {print $0}' awk_test.txt    
    
    一个relational expression可以使用后面将要讲到的任何运算符，这种模式通常用来测试一个field是否匹配某个正则表达式。
    关系表达式是指使用关系运算符（>，>=，<，<=，==，!=）连接一个或两个表达式组成的式子。
    “&&”、“||”和“!”即我们熟知的与、或、非逻辑运算符，
    可以用来将多个patterns连接在一起，
    这时他们是short-circuite valuation（短路求值）的，例如对于&&操作，
    只要第一个值是false，那整个表达式就是false，
    就没必要计算后续的值了。圆括号( )可以改变运算符的运算顺序。
    例如，打印第一个field不是“root”的行且第三个field等于“0.0”的行：
    awk '$1 != "root" && $3 == "0.0" {print}' awk_test.txt 
    “?:”运算符和C语言中的三目的条件运算符一样，
    pattern1为true则选用pattern2，否则选用pattern3。
    pattern1, pattern2形式的表达式被称为范围样式，
    它将匹配pattern1的record，
    匹配pattern2的record以及这两个record之间的所有record都匹配出来。

    动作（actions）
    AWK中的动作语句（action statements）都放在花括号{ }中，由大部分语言都有的赋值、条件和循环语句组成。
    AWK中的运算符、控制语句以及输入输出语句都是仿照C语言来定义的。

    I/O语句
    输入输出语句列举如下：
    
    close(file [, how])
    关闭文件、管道或协同进程。参数“how”只有在关闭协同进程的双向管道其中一端时才会用到，
    是字符串类型，可取"to"或"from"。
   
    getline
    把$0赋值为下一个输入的record。同时会更新NF、NR和FNR。
   
    getline <file
    把$0赋值为文件file的下一个record。同时会更新NF。
    
    getline var
    把变量var赋值为下一个输入的record。同时会更新NR和FNR。
    
    getline var <file
    把变量var赋值为文件file的下一个record。
    
    command | getline [var]
    执行command并将输出作为getline或getline var的输入。
   
    command |& getline [var]
    执行协同进程command并将输出作为getline或getline var的输入。（协同进程是gawk的扩展，command也可以是一个socket）
   
    next
    停止处理当前的record，并读取下一个record，
    然后重新从awk程序中第一个pattern开始处理新的record。
    如果当前已经达到输入数据中最后一个record，
    则开始执行END{ }程序块。
    
    nextfile
    停止处理当前的文件，并从下一个文件中读取record，
    然后重新从awk程序中第一个pattern开始处理新的record。
    FILENAME和ARGIND参数会被更新，FNR被重置为1
    。如果当前已经达到输入数据中最后一个record，则开始执行END{ }程序块。
    
    print
    打印当前的record（根据ORS的值判断record是否输出完毕）。
   
    print expr-list
    打印表达式列表。如果有多个表达式，每个表达式之间用OFS分隔。（根据ORS的值判断record是否输出完毕）
    
    print expr-list >file
    打印表达式列表到文件。如果有多个表达式，
    每个表达式之间用OFS分隔。
    （根据ORS的值判断record是否输出完毕）
    
    printf fmt, expr-list
    格式化的打印
   
    printf fmt, expr-list >file
    格式化的打印到文件。例如：
    awk 'BEGIN{printf "%#x", 10 >"getnum.txt"}'
   
    system(cmd-line)
    执行命令cmd-line，并将exit status作为返回值。例如：
    system("uname –a")
    
    fflush([file])
    清空已打开的输出文件或管道文件的所有缓存。
    如果不带file参数，则清空标准输出，如果file参数为null string，则所有打开的输出文件和管道的缓存都会被清空。
    
    另外，print和printf也允许以下形式的输出重定向：
    print ... >> file
    将输出追加到文件file
    
    print ... | command
    向管道写内容
    
    print ... |& command
    向协同进程或socket发送数据
    getline命令执行成功返回1，到达文件结尾返回0，
    出错返回-1。如果出错，则字符串ERRNO包含了出错信息。
    注意，如果在打开一个双向（two-way）socket的时候失败，将会返回一个非致命的错误到调用者。
    如果在一个循环中使用管道、协同进程或socket（向getline输出或从print/printf输入），
    你必须使用close()来创建新的command或socket的实例。
    AWK在管道、协同进程或socket返回EOF的时候不会自动关闭它们。
    
    printf语句
    AWK里的printf语句和sprintf()函数支持如下的标准格式转换符：
    
    %c
    一个ASCII字符。如果传入的参数是一个数值，
    则将其转换为字符并打印；否则，认为传入的参数是字符串，只打印该字符串的第一个字符。
    
    %d, %i
    十进制数字（整数部分）
    %e, %E
    
    将一个浮点数以“[-]d.dddddde[+-]dd”的形式打印。%E使用大写的E。
    %f, %F
    
    将一个浮点数以“[-]ddd.dddddd”的形式打印。
    使用%F（需要系统库支持），则以大写字母显示"not a number"和"infinity"这类的值。
    %g, %G
    根据实际情况转换为%e或%f，选择其中最简短的格式。%G对应%E。
    
    %o
    无符号的八进制整数。
    
    %u
    无符号的十进制整数。
    
    %s
    字符串。
    
    %x, %X
    无符号的十六进制整数。
    
    %%
    字符'%'的原意（不会取实参）。
    在这些格式符中，'%'和控制字符之间可以放置如下的额外参数（和C语言中printf的格式符用法相同）：
    
    count$
    位置说明符，指定对第count个参数进行格式化转换
    。例如：printf("%2$d\n", 23, 24);打印的值是24。
    
    width
    指定格式化后的最短打印长度，如果不够长则填充空格，默认右对齐。
    
    -
    左对齐，通常与width搭配使用。
    
    0
    上述不足width的话，使用前导0填充而不是空格。只对数值类型有效。
    
    +
    对数值添加正负号。只对数值类型有效。
    
    #
    以另一种形式打印格式化的内容：
    例如%#o会在数值前面填0；%#x、%#X会在数值前面填0x或0X；
    对%#e、%#E、%#f和%#F，结果总会有小数点；%#g、%#G总会显示小数部分。
    
    space
    对正数，前面填一个空格；对负数，前面填负号。
    
    .prec
    对%e、%E、%f和%F，指明小数部分的最大位数；对%g、%G，
    指明有效数字的最大长度；对%d、%o、%i、%u、%x和%X，
    和面前的0width一样，指明最短打印长度，
    不足则前面补0；对%s，指明最长打印的字符数。
    
 
    另外，printf和sprintf()支持动态的width和.prec：
    可以从参数列表中的值作为width或.prec的值。
    通过在count$前面加一个*号来达到这种效果。例如：
    printf("%1$*2$s\n", "Bye bye!", 12);
    
    这个例子中，1$仍然是位置说明符的作用，
    而*2$的意思是将第二个参数替换在这个位置，这个语句就变成了：
    printf("%1$12s\n", "Bye bye!", 12);
    
    
#### OpenSsl工具的介绍
CA 具有注册和颁发证书的功能？ 但是，证书到底具有哪些信息或者遵循哪些标准呢？
    字段说明：
    版本号：说明证书采用x509的版本
    证书序列号：原来标明证书的个数，每发一个证书序列号就会加1
    算法参数：签名算法
    发行者名称：CA 的名称
    有效日期：证书的有效期
    主体名称：证书拥有者的名称，可能是用户名或主机名。
    公钥：申请者提供的公钥
    发行者ID：CA 的编号
    主题ID: 拥有者的编号，CA 的生成
    附加信息：其他附加请求
    CA的签名：CA 签名信息
    
    

    SSL(Secure Socket Layer/Transport Layer Security)，是一种安全协议。
    
    SSL是应用层与传输层之间的协议，有 sslv1,sslv2,sslv3版本。TLS是仿照SSL的功能，
    tlsv1等价于sslv3。应用程序如果使用SSL的功能话，
    需要调用 SSL 的公共库。

    以 web 服务访问为例，ssl会话建立的过程大致如下：
![wKioL1PdzomikN1HAADeYoePR20837](764DBD3BD51F47208211995C8663CFDD)

    OpenSSL是一个开源项目，实现了数据的加密解密，证书认证等功能。主要有 3 部分组成：
    openssl:提供 openssl 命令行工具
    libssl: 实现 ssl 协议
    libcrypto: 公共加密库，里面实现了好多的加密算法

    
![wKiom1Pd37HzGlSYAAQV5KYHFV4202](00DA6B0DB03E42C79920ED7151CD0748)
    
    



openssl命令的使用：     
```
# 示例
    # 查看版本号
    [root@centos6-5 ~]# openssl version
    OpenSSL 1.0.1e-fips 11 Feb 2013
    
    # 利用 MD5 算法生成用户密码
    [root@server ~]# openssl passwd -1 -salt `openssl rand -hex 4` 
    Password: 
    $1$8da8005e$3rBnudq4DtUPDtB5oX/AQ0
    
    # 提取特征码，实现单向加密
    [root@centos6-5 ~]# openssl dgst -md5 /etc/passwd
    MD5(/etc/passwd)= a957b39dff384602a7048b758a046695
    [root@centos6-5 ~]# openssl dgst -out ./md5-passwd -md5 /etc/passwd
    [root@centos6-5 ~]# cat ./md5-passwd 
    MD5(/etc/passwd)= a957b39dff384602a7048b758a046695
    
    # 实现对称加密
    # -a 是基于 base64 编码，-e 加密 
    [root@centos6-5 ~]# openssl enc -des3 -e -a -salt -in ./md5-passwd -out ./md5-passwd.enc
    enter des-ede3-cbc encryption password:
    Verifying - enter des-ede3-cbc encryption password:
    [root@centos6-5 ~]# cat ./md5-passwd.enc 
    U2FsdGVkX1/1B23N1i5DH5fmBVX1vHMG6ko2yBtt0XchzRExE1z8RaCR5NU98w5q
    vYITDKLWWzL4ilW1aSlnq3fAVLS6cI9b
    # 解密查看
    # -d 解密 diff命令查看俩个文件的差异
    [root@centos6-5 ~]# openssl enc -des3 -d -a -salt -in ./md5-passwd.enc -out ./md5-passwd2
    enter des-ede3-cbc decryption password:
    [root@centos6-5 ~]# diff ./md5-passwd ./md5-passwd2
    
    # 生成秘钥
    [root@centos6-5 ~]# (umask 077;openssl genrsa -out /root/mykey 1024)
    Generating RSA private key, 1024 bit long modulus
    ..........++++++
    ...................++++++
    e is 65537 (0x10001)
    [root@centos6-5 ~]# cat /root/mykey 
    -----BEGIN RSA PRIVATE KEY-----
    MIICXAIBAAKBgQC6DEaXhrKwnB0JM7VvLFV1TxQmXFjEKIxoDUugBeCZ1mWt1FEL
    pi7hiN2u3BGBwrbwKAZj8I55Z6pGSXEcKmMMTf1vbUopO9tUknV7nfIGjQeBzwqB
    hPK9+gj4xK/LvLr/FM8+WcxXW04FFNNW8WGz7uwUgKbW6XtsjMXbVLuVpwIDAQAB
    AoGAVp+0loSe2mA1nL04suSffZkuNpY0tlBy31ehaIaUBsyuVvtOKPBdT6FcJjhM
    5m/0oWjhYNL2Y0yDGWrEgWqy5pJlbCT7Pow9w0gvoMT+tVzIwXJ4Cz7eL12X+8vk
    x6IfNAS0YWHlSF/ZzX3WviYy/77fD8qRaqZj4oy0HwjB4LECQQDiQvuyjJQDrNzy
    2TQB20tSIBtZXz4AKlh23At6Emv+EeSiOT12/Juxn2gIDgCuPMiE6WUTr/0TKeDn
    djAzu1uDAkEA0oA5SL93UsHTeSDPe3l1A3KdfC7/38f1VCHS/+Ab2T+g0OpI2X5R
    dTZLo+pvkvyZhBhooyrsXO71vE9RfLVQDQJBAKORrQgNHMvzYd+mKjTVZgQ+9caM
    VfQkqMN0nE9pleyc3t5v5wFn6N5l0P1RsihEBOohGFM9PQVnlxF9nacoYSUCQAdR
    7CwKdHTNRrRUnsJ1c8s95hoWbFF026QkVPkO6wj//HCnZQcjLGP+El1N3rlmzVPZ
    oXHjITsOGD+HJpdGmtUCQFcXAjw8bQekrSzL7R+f4RgpCfuy749B+Hg9QZc0R79t
    /d3dwHBm1fFFe6QEVI1ArL1I8uhlFCwDaZdfQhRz8Y4=
    -----END RSA PRIVATE KEY-----
    
    # 提取公钥
    # -pubout 指定是公钥输出
    [root@centos6-5 ~]# openssl rsa -in /root/mykey -out /root/mypub.key -pubout
    writing RSA key
    [root@centos6-5 ~]# cat /root/mypub.key 
    -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC6DEaXhrKwnB0JM7VvLFV1TxQm
    XFjEKIxoDUugBeCZ1mWt1FELpi7hiN2u3BGBwrbwKAZj8I55Z6pGSXEcKmMMTf1v
    bUopO9tUknV7nfIGjQeBzwqBhPK9+gj4xK/LvLr/FM8+WcxXW04FFNNW8WGz7uwU
    gKbW6XtsjMXbVLuVpwIDAQAB
    -----END PUBLIC KEY-----
```

    
    OpenSSL实现 CA 的过程
        实现自建 CA 的大致流程：
![wKiom1Pd7l6h2yVhAAFjm633G1c910](219598805BF0466396E54942CE219EB8)
    
    自建 CA 的详细操作流程
        第一步：自建 CA 服务器
            1、生成秘钥
            [root@centos6-5 ~]# (umask 077;openssl genrsa -out /etc/pki/CA/private/cakey.pem 4096)
            2、自签证书
            [root@centos6-5 ~]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 360
            
            # req 生成证书签署请求
            # -news 新请求
            # -key 指定私钥文件
            # -out 签署文件的存放地方
            # -x509 生成字签署文件
            # -days 有效天数
            
            3、创建必要文件，初始化工作环境
            [root@centos6-5 ~]# touch /etc/pki/CA/index.txt
            [root@centos6-5 ~]# echo "00" > /etc/pki/CA/serial
            
        第二步：节点申请证书
             1、生成密钥对儿
            [root@centos6-5 ~]# mkdir /etc/httpd/ssl;(umask 077;openssl genrsa -out /etc/httpd/ssl/httpd.key 4096)
           2、生成证书签署请求
            [root@centos6-5 ~]# openssl req -new -key /etc/httpd/ssl/httpd.key -out /etc/httpd/ssl/httpd.csr
            
        第三步：CA签署证书
            1、验正证书中的信息，签署证书
            [root@centos6-5 ~]# openssl ca -in /etc/httpd/ssl/httpd.csr -out /etc/httpd/ssl/httpd.crt -days 300
            2、发送给请求者
            # 使用 scp 发送给请求者  
            
        至此签名完成，如下：
        [root@centos6-5 ~]# cat /etc/pki/CA/index.txt 
        V	150528153748Z		00	unknown	/C=CN/ST=Henan/O=MageEdu/OU=Ops/CN=www.magedu.com/emailAddress=www@magedu.com
        [root@centos6-5 ~]# cat /etc/pki/CA/serial 
        01
        
    证书的吊销
        有时候我们的的节点秘钥丢了，此时就需要向 CA 申请吊销。
        在节点：此时首先要在节点获取证书的serial
        [root@server ssl]# openssl x509 -in /etc/httpd/ssl/httpd.crt -noout -serial -subject
        # noout 不输出额外信息
        # serial 输出 serial 信息，在 serial 文件中
        # 输出 subject 信息，在 index.txt 文件中
        在CA：
        1、验证信息
            根据节点提交的serial和subject信息来验正与index.txt文件中的信息是否一致
        [root@centos6-5 ~]# cat /etc/pki/CA/index.txt
        2、吊销证书
            1）吊销证书
        # 要吊销的证书一般在 /etc/pki/CA/newcerts 目录下，名称是 序列号.pem
        
        [root@centos6-5 ~]# openssl ca -revoke /etc/pki/CA/newcerts/01.pem 
        
        2）生成吊销列表（第一次吊销是需要）
        [root@centos6-5 ~]# echo 00 > /etc/pki/CA/crlnumber
        
        3）更新证书吊销列表
        [root@centos6-5 ~]# openssl ca -gencrl -out /etc/pki/CA/crl/01.crl
        
        查看 crl 里的内容：
        [root@centos6-5 ~]# openssl crl -in /etc/pki/CA/crl/01.crl -noout -text


        
        


