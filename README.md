# Make_an_Operating_System
基于《30天自制操作系统》一书自制的一个操作系统
以下是进度记录

## Day 0 着手开发之前

#### 如何开发操作系统

1. 开发步骤
   + 在win(或者其他os上)编写源代码
   + 用从语言编译器编译源代码生成机器语言文件
   + 对机器语言进行加工, 生成**软盘映像文件**
   + 将映像文件写入磁盘, 做成含操作系统的启动盘
2. 解释
   1. 映像文件简单来说就是软盘的备份数据

#### 操作系统开发中的困难

似乎要用魔改过的gcc? 

## Day 1 从计算机结构到汇编程序入门

#### 先动手操作

1. 软件下载

这个是个啥软件啊???

lzh是什么上古时代的文件格式了吧

这里直接VS大法, 我什么插件找不到非得用这玩意?

(这里插件我直接输入一个Hex然后下载的第一个, 至于具体好不好用咱也不知道)

2. helloos.img

我选择直接网上下载, 谁真的一个个敲我笑他一辈子

然后敲出来之后是可以看见要输出一个hello world的

![](./001.png)

然后这里书上说的是要买一个软盘的, 但是我不可能真的给你买一个软盘, 什么上古时代的东西啊, 我去博物馆里抢一个吗? 所以这里选择虚拟机

新建一个空白的虚拟机, 什么都不要安装, 因为我们就是在写自己的操作系统, 然后系统类型选择其他, 新建空白虚拟机之后在编辑虚拟机配置那里添加刚刚那个img文件

然后就会出现这样

![](./002.png)

#### 究竟做了什么

这里主要就是介绍了一下CPU和二进制十六进制, 稍微有点计算机基础的都可以不用看

#### 初次体验汇编程序

现在开始要用汇编来写程序了, 书上用的是作者自己魔改NASM造出来的NASK

打开!cons_nt.bat文件, 输入下面这一串神秘代码, 获得helloos.img映像文件

```
..\z_tools\nask.exe helloos.nas helloos.img
```

这个的大概意思就是用nask.exe来执行helloos.nas文件,  将生成的东西命名为helloos.img文件.至于helloos.nas文件, 就是汇编代码了. 

后面每次进行汇编编译都要来这么一出的话会很麻烦, 所以出现了批处理文件(文件后缀为.bat batch file), 这里批处理文件命名为asm.bat. 每次只要在!cons文件中输入asm就可以生成helloos.img文件, 然后再执行run指令就可以用qemu来跑(Hey, bro~)

![](./003.png)

(备注:这里bat文件里面其实就一行..\z_tools\nask.exe helloos.nas helloos.img, 就是懒得打那么多字XD)

接下来对helloos.nas文件进行一些解释

+ DB: data byte 往文件里直接写入一个字节的指令
+ RESB: reserve byte 从现在的地址开始空出10个字节出来, 意思是提前占用这10个字节, 并且填入0x00

#### 加工润色

```
; hello-os
; TAB=4

; 以下这段是标准FAT12格式软盘专用的代码

		DB		0xeb, 0x4e, 0x90
		DB		"HELLOIPL"		; 启动区的名称可以是任意的字符串
		DW		512				; 每个扇区（sector)的大小（必须为512字节）
		DB		1				; 簇（cluster)的大小（必须为1个扇区）
		DW		1				; FAT12的起始位置（一般从第一个扇区开始）
		DB		2				; FAT的个数（必须为2)
		DW		224				; 根目录的大小（一般设成224项）
		DW		2880			; 该磁盘的大小（必须是2880扇区）
		DB		0xf0			; 磁盘的种类（必须是0xf0）
		DW		9				; FAT的长度（必须是9扇区）
		DW		18				; 1个磁道（track)有几个扇区（必须是18）
		DW		2				; 磁头数（必须是2）
		DD		0				; 不使用分区，必须是0
        DD      2880            ; 重写一次磁盘大小
		DB		0,0,0x29		; 意义不明，固定
		DD		0xffffffff		; (可能是)卷标号码
		DB		"HELLO-OS   "	; 磁盘的名称（11字节）
		DB		"FAT12   "		; 磁盘格式名称
		RESB	18				; 先空出18字节

; 程序主体

		DB		0xb8, 0x00, 0x00, 0x8e, 0xd0, 0xbc, 0x00, 0x7c
		DB		0x8e, 0xd8, 0x8e, 0xc0, 0xbe, 0x74, 0x7c, 0x8a
		DB		0x04, 0x83, 0xc6, 0x01, 0x3c, 0x00, 0x74, 0x09
		DB		0xb4, 0x0e, 0xbb, 0x0f, 0x00, 0xcd, 0x10, 0xeb
		DB		0xee, 0xf4, 0x 

; 以下是启动区以外部分的输出

		DB		0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
		RESB	4600
		DB		0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
		RESB	1469432

```

1. 这里的:就相当于是注释的开始符号
2. DB后面可以直接跟字符串, 汇编语言会自动把字符串给转成编码再排列起来
3. DE: data word: 16个字节
4. DD: data double word:32个字节(一个word就是1个字嘛)
5. RESB 0x1fe-$  这里的美元符号代表这一行现在的字节数, 这里就是从现在这个位置到0x1fe全部写上0x00
6. FAT12格式 用Win或者MS-DOS格式化出来的软盘的格式
7. 启动区(boot sector) 软盘的第一个扇区, 计算机读取文件是以一个个扇区为单位(512字节)进行读写, 启动区强制要求最后两个字节是55AA 否则程序不能启动(没有为什么)
8. IPL(initial program loader) 启动程序加载器, 把加载操作系统本身的程序写在启动区而不是把整个操作系统塞进启动区里

## Day2 汇编语言学习与Makefile入门

#### 介绍文本编辑器

软件下载, 这里直接用notepad++就好了(其实后面用的主要是vscode, vs是真的香XD), 这个terapad是什么玩意?

#### 继续开发

主要就是一些汇编语言的讲解

```
; hello-os
; TAB=4

		ORG		0x7c00			; 指明程序的装载地址

; 以下的记叙用于标准FAT12格式的软盘

		JMP		entry
		DB		0x90
		DB		"HELLOIPL"		; ブートセクタの名前を自由に書いてよい（8バイト）
		DW		512				; 1セクタの大きさ（512にしなければいけない）
		DB		1				; クラスタの大きさ（1セクタにしなければいけない）
		DW		1				; FATがどこから始まるか（普通は1セクタ目からにする）
		DB		2				; FATの個数（2にしなければいけない）
		DW		224				; ルートディレクトリ領域の大きさ（普通は224エントリにする）
		DW		2880			; このドライブの大きさ（2880セクタにしなければいけない）
		DB		0xf0			; メディアのタイプ（0xf0にしなければいけない）
		DW		9				; FAT領域の長さ（9セクタにしなければいけない）
		DW		18				; 1トラックにいくつのセクタがあるか（18にしなければいけない）
		DW		2				; ヘッドの数（2にしなければいけない）
		DD		0				; パーティションを使ってないのでここは必ず0
		DD		2880			; このドライブ大きさをもう一度書く
		DB		0,0,0x29		; よくわからないけどこの値にしておくといいらしい
		DD		0xffffffff		; たぶんボリュームシリアル番号
		DB		"HELLO-OS   "	; ディスクの名前（11バイト）
		DB		"FAT12   "		; フォーマットの名前（8バイト）
		RESB	18				; とりあえず18バイトあけておく

; 程序核心

entry:
		MOV		AX,0			; 初始化寄存器
		MOV		SS,AX
		MOV		SP,0x7c00
		MOV		DS,AX
		MOV		ES,AX

		MOV		SI,msg
putloop:
		MOV		AL,[SI]
		ADD		SI,1			; SI+1
		CMP		AL,0
		JE		fin
		MOV		AH,0x0e			; 显示一个文字
		MOV		BX,15			; 指定字符颜色
		INT		0x10			; 调用显卡BIOS
		JMP		putloop
fin:
		HLT						; 让CPU先停止等待指令
		JMP		fin				; 无限循环

msg:
		DB		0x0a, 0x0a		; 换行2次
		DB		"hello, world"
		DB		0x0a			; 换行
		DB		0

		RESB	0x7dfe-$		; 0x7dfeまでを0x00で埋める命令

		DB		0x55, 0xaa

; 以下はブートセクタ以外の部分の記述

		DB		0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
		RESB	4600
		DB		0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
		RESB	1469432

```

(是的, 我能找到的资料里面显示的是日文, 呃啊, 边看边百度好痛苦)

1. ORG指令

```
ORG 0x7c00	; 指明程序的装载地址
```

有了这个指令之后, $代表的就是将要读入的内存地址, 而不是之前的在输出文件的第多少个字节

另外, 这里0x7c00是不能随便更改的, 0x7c00~0x7dff是拿来作为启动区内容的装载地址, 至于为什么不用0

x00, 因为这里需要给BIOS拿来实现不同功能, 另外, 0xf0000用来放BIOS程序本身, 也不能随便乱改(至于为什么是这些奇怪的地址, 书上的原话让我们去问问IBM的大叔, 樂)

2. JMP指令

```
JMP entry

;中间省略

entry:
	MOV AX, 0
	;后面省略
```

JMP指令就相当于C的goto语句, entry就是跳转目的地的标签

3. **MOV指令**

代码见上, `MOV AX,0`就相当于`AX = 0`, 这里的AX之类的都是寄存器

另外, MOV指令有一个规则, 就是源数据和目的数据必须位数相同

> 常用的几个寄存器(现在是计组时间!)
>
> AX	accumulator			累加寄存器
>
> CX	counter					计数寄存器
>
> DX	data						 数据寄存器
>
> BX	base					 	基址寄存器
>
> SP	stack pointer			栈指针寄存器
>
> BP	base pointer		 	基址指针寄存器
>
> SI	source index		 	源变址寄存器
>
> DI	destination index	目的变址寄存器
>
> 这些寄存器全都是16位寄存器, 然后就是各自的用途名字其实已经解释的很清楚了, 更加详细的这里就不解释了, 那就得去翻计组的书了
>
> 另外还有一些8位的寄存器, 基本上都是高位或者低位的寄存器
>
> 当然也有32位的寄存器, 比如EAX,EXC之类的, 这里的E值得就是extend

4. [内存地址]

```
MOVE BYTE [678], 123
```

用内存的678地址来保存数字 `123`, 这里BYTE是保留字, 678用方括号圈起来就表示这里是内存地址而不是简单的数字

虽然可以用寄存器来指定内存地址, 但是AX, CX, DX, SP不能用来指定内存地址, 所以想要把DX的值赋给AL, 就需要这么写

```
MOV BX, DX
MOV AL, BYTE[BX]
```

5. ADD指令

```
ADD SI,1
;SI = SI + 1
```

6. CMP指令

```
CMP AL,0
JE fin

;上面的两条指令就相当于
;if(AL == 0){
;	goto fin
;}
;JE的意思是Jump if Equal
```

7. INT指令

软件中断指令, interrupt

接下来需要讲讲BIOS(basic input output system)

这玩意本质上就是拿来给开发操作系统的苦逼进行输入输出的函数集合, INT就是用来调用这些函数的指令

```
INT 0x10
; 中断, 调用16号函数
```

所以这里的目的是要显示文字, 我们需要找于显卡相关的函数,  然后再去相关的显卡官网去查, 就可以找到相关函数的参数, 以这里的0x10为例

```
AH = 0x0e
AL = character code
BH = 0;
BL = color code
```

只要按照这里的内容按照要求往寄存器里塞值就可以了(注: 这里是BL 但是代码中是BX, 就是低位和全部字节的区别, 反正都一样, 都只填写了8位低位, 然后高位补0,所以在这里用BX和用BL差不多)

8. HLT(halt)指令

让CPU进入待机状态, 但是只要外部发生变化CPU就会退出待机状态, 这样可以节能省电, 保护电脑硬件, 否则CPU一直满负荷

所以用了上面这么多内容来解释, 汇编程序可以简化为

```
;初始化
entry:
	AX= 0;
	SS = AX;
	SP = 0x7c00;
	DS = AX
	ES = AX
	SI = msg;
	
;循环显示数据直到结尾0处	
putloop:
	AL = BYTE [SI]
	SI = SI + 1;
	if(AL == 0){
		goto fin;
	}
	AH = 0x0e;
	BX = 15;
	INT 0x10
	goto putloop;
	
;让CPU进入待机状态	
fin:
	HLT;
	goto fin;
```

#### 先制作启动区

考虑到以后的开发, 我们只用nask来做启动区, 剩下的部分用的是磁盘映像管理工具 

在helloos4的文件夹中, helloos.nas文件变为了ipl.nas, 只保留了启动区该有的内容, 把一些额外的输出给删掉了

另外, asm.bat将输出文件改名为ipl.bin, 还输出了列表文件ipl.lst(文本文件, 用于确认每个指令是如何翻译为机器语言的)

```
..\z_tools\nask.exe ipl.nas ipl.bin ipl.lst
```

makeimg.bat 是以ipl.bin为基础, 制作helloos.img文件(edimg是书的作者自己捏的)

```
..\z_tools\edimg.exe   imgin:../z_tools/fdimg0at.tek   wbinimg src:ipl.bin len:512 from:0 to:0   imgout:helloos.img
```

这样以后再从编译到测试就是asm->makeimg->run

#### Makefile入门

```
#井号后面的是注释

#文件生成规则
ipl.bin : ipl.nas Makefile
	../z_tools/nask.exe ipl.nas ipl.bin ipl.lst
	
# 翻译一下上面这一段鸟语
# 如果要生成ipl.bin文件就需要先检查一下ipl.nas文件是否存在, 然后再进行makefile
# 按照下面的规则进行生成(其实就是asm.bat文件里的内容)

helloos.img : ipl.bin Makefile
	../z_tppls/edim.exe imgin:../z_tools/fdimg0at.tek   wbinimg src:ipl.bin len:512 from:0 to:0   imgout:helloos.img

# 这里也是一样的, 用makefile的规则重新描述了一遍makeimg.bat中的内容, 甚至下面一行的内容和makeimg.bat中的内容一模一样
```

然后运行的时候, 输入`make -r ipl.bin`, make.exe就会自动进入makefile文件中去read寻找制作ipl.bin文件的方法,  如果缺少某个文件, makefile还会自动去找缺失文件的生成方法, 另外, makefile还会自动将文件更新为最新版本, 所以遇事不决直接makefile就好力(喜)

另外, 在makefile文件中加入

```
img :
	../z_tools/make.exe -r helloos.img
	
asm : 
	../z_tools/make.exe -r ipl.bin
	
run :
	../z_tools/make.exe img
	copy helloos.img ..\z_tools\qemu\fdimage0.bin
	../z_tools/make.exe -C ../z_tools/qemu
	
install :
	../z_tools/make.exe img
	../z_tools/imgtol.com w a: helloos.img
```

以后只输入make img就可以直接去生成(或者是更新)helloos.img文件



最终的Makefile文件长这个鬼样子

```

# 僨僼僅儖僩摦嶌

default :
	../z_tools/make.exe img

# 僼傽僀儖惗惉婯懃

ipl.bin : ipl.nas Makefile
	../z_tools/nask.exe ipl.nas ipl.bin ipl.lst

helloos.img : ipl.bin Makefile
	../z_tools/edimg.exe   imgin:../z_tools/fdimg0at.tek \
		wbinimg src:ipl.bin len:512 from:0 to:0   imgout:helloos.img

# 僐儅儞僪

asm :
	../z_tools/make.exe -r ipl.bin

img :
	../z_tools/make.exe -r helloos.img

run :
	../z_tools/make.exe img
	copy helloos.img ..\z_tools\qemu\fdimage0.bin
	../z_tools/make.exe -C ../z_tools/qemu

install :
	../z_tools/make.exe img
	../z_tools/imgtol.com w a: helloos.img

clean :
	-del ipl.bin
	-del ipl.lst

src_only :
	../z_tools/make.exe clean
	-del helloos.img

```

(这里是什么编码我也不知道, vs自动推荐的编码是ISO 8859-2 啥玩意啊, 输出的也是乱码, 还不如这样, 至少输出的乱码是中文, 看起来和蔼可亲一点XD)

这样以后要编译并测试就可以直接`make run`一步到位, 好欸

## Day 3 进入32位模式并导入C语言

#### 制作真正的IPL

IPL: Initial Program Loader 启动程序装载器 

```
; 读光盘

		MOV		AX,0x0820
		MOV		ES,AX
		MOV		CH,0			; 柱面0
		MOV		DH,0			; 磁头0
		MOV		CL,2			; 扇区2

		MOV		AH,0x02			; AH=0x02 : 读盘
		MOV		AL,1			; 1个扇区
		MOV		BX,0
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 调用磁盘BIOS
		JC		error
```

+ 这里的柱面, 磁头, 扇区就涉及到硬盘了(机械硬盘才有物理意义上的柱面磁头扇区, 固态硬盘没有, 但是为了统一, 固态硬盘也这么分区)

+ JC: jump if carry, 如果进位标志(carry flag)是1的话, 就跳转

+ `INT 0x13` 调用磁盘BIOS的0x13号函数

  >**调用参数**
  >
  >AL=扇区数
  >CH,CL=磁盘号,扇区号
  >DH=磁头号
  >ES:BX=数据缓冲区地址
  >DL:驱动器号
  >00H~7FH：软盘；
  >80H~0FFH：硬盘
  >
  >
  >
  >**返回参数**
  >
  >读成功:CF=0,AH=0
  >AL=读取的扇区数
  >读失败:CF=1,AH=出错代码
  >(参见功能号02H中的说明)
  >
  >(这里CF就是进位标志carry flag,加上后面的JC(jump if carry), 也就是说如果出错了的话CF = 1
  >
  >
  >
  >参考连接:[BIOS中断表(整理更新中2013-10-23)_bios2013_瞧红尘的博客-CSDN博客](https://blog.csdn.net/xa04xa04/article/details/12871425)
  >
  >(啧, 找不到官网, 烦)
  
  在有多个驱动器的时候, 用磁盘驱动器号来指定从哪个驱动器的软盘上读取数据, 现在的电脑, 基本上只有一个软盘驱动器(???现在的电脑有软盘吗???)
  
  柱面磁头之类的概念一般出现在机械硬盘, 但是固态硬盘也有, 这里就不做更多解释了, 我直接就是一个跳转到计组的大动作(说不定后面我会把这个放到我那个网站上, 然后再在这里挂一个链接)
  
  一张软盘一般有80个柱面, 2个磁头, 18个扇区, 并且一个扇区是512字节, 所以一般一张软盘的容量是1440KB
  
  含有IPL的启动区, 位于C0-H0-S1, 这次要读取的扇区是C0-H0-S2

**关于缓冲区地址的选择**

+ 内存地址, 表明我们要把从软盘读出来的数据装载到内存的什么位置
+ 因为BX只能表示0~0xffff的值, 只有64K, 太小了, 所以增加了一个叫做EBX的寄存器, 这样就可以处理4G内存了(后面再说,, 设计BIOS的时候32位寄存器都还没有), 当时设计了一个器辅助作用的段寄存器, 在指定主存地址的时候, 可以用这个段寄存器
+ 在使用段寄存器的时候, 以ES:BX这种方式来表示地址, 写成`MOV AL,[ES:BX]`,它代表ES* 16 + BX的内存地址, --> 可以理解为先用ES寄存器指定一个大致的地址,  然后再用BX来指定其中一个具体地址
+ 虽然这样最大也就0xffff * 16 + 0xffff, 也就是只有1M. 但是在当时还是够了
+ 这次指定的ES = 0x0820, BX = 0. 所以数据被装载到内存的0x8200到0x83ff的地方 (0x8000~0x81ff是留给启动区的, 要将启动区的内容读取到那里)

一般不管要指定内存的什么地址, 都必须要同时指定段寄存器, 一般如果省略的话会把`DS:`作为默认的段寄存器

以前写MOV CX, [1234], 实际上是MOV CX,[DS:1234], 但是这么写台麻烦了, 所以直接给省略了

因为这样的规则, 所以DS必须预先指定为0,. 否则地址的值就要加上这个数的16倍, 会报错

#### 试错

软盘有时候会发射不能读取数据的错误, 所以需要多试几次, 但是也有可能是真的坏了, 所以对之前的代码进行改进, 多读几次, 要是读了5次还是不行那说不定就是真的出了问题, 所以将之前的代码修改如下

```
; 读磁盘

		MOV		AX,0x0820
		MOV		ES,AX
		MOV		CH,0			; 柱面0
		MOV		DH,0			; 磁头0
		MOV		CL,2			; 扇区2

		MOV		SI,0			; 记录失败次数的寄存器
retry:
		MOV		AH,0x02			; AH=0x02 : 读入磁盘
		MOV		AL,1			; 1个扇区
		MOV		BX,0
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 调用磁盘BIOS
		JNC		fin				; 没出错的话就直接跳转到fin
		ADD		SI,1			; SI+1
		CMP		SI,5			; 比较SI与5
		JAE		error			; SI >= 5 跳转到error
		MOV		AH,0x00
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 重置驱动器
		JMP		retry
```

+ JNC: jump if not carry, 如果进位标志是0的话就跳转
+ JAE: jump if above or equal 大于等于的时候跳转
+ 被判定为出错之后, 在重新读盘之前, 我们将AH = 0x00, DL = 0x00, INT = 0x13 进行系统复位(自己去查找BIOS中断表), 恢复软盘状态, 然后再来读取

#### 读到18扇区

```
; 读磁盘

		MOV		AX,0x0820
		MOV		ES,AX
		MOV		CH,0			; 柱面0
		MOV		DH,0			; 磁头0
		MOV		CL,2			; 扇区2

		MOV		SI,0			; 记录失败次数的寄存器
retry:
		MOV		AH,0x02			; AH=0x02 : 读入磁盘
		MOV		AL,1			; 1个扇区
		MOV		BX,0
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 调用磁盘BIOS
		JNC		fin				; 没出错的话就直接跳转到fin
		ADD		SI,1			; SI+1
		CMP		SI,5			; 比较SI与5
		JAE		error			; SI >= 5 跳转到error
		MOV		AH,0x00
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 重置驱动器
		JMP		retry
next:
		MOV		AX,ES			; 把内存地址后移0x200
		MOV		ES,AX			; 因为没有ADD ES, 0x020指令, 所以这里稍微绕个弯
		ADD		CL,1			; CL + 1
		CMP		CL,18			; CL与18进行比较
		JBE		readloop		; CL <= 18 跳转到readloop

```

+ JBE: jump if below or equal
+ next后面这里是要读取下一个扇区, CL好办, 直接+1就好了, ES是指定的地址, 至于为什么是0x020, 因为16进制下512/16就是这个结果

在这里用了一个循环, 这样就可以把磁盘上c0-H0-S2~C0-H0-S18的512 * 17= 8704字节的内容装入内存的0x8200~0xa3处

#### 读入10个柱面

```
; 读磁盘

		MOV		AX,0x0820
		MOV		ES,AX
		MOV		CH,0			; 柱面0
		MOV		DH,0			; 磁头0
		MOV		CL,2			; 扇区2

		MOV		SI,0			; 记录失败次数的寄存器
retry:
		MOV		AH,0x02			; AH=0x02 : 读入磁盘
		MOV		AL,1			; 1个扇区
		MOV		BX,0
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 调用磁盘BIOS
		JNC		next			; 没出错的话就直接跳转到next
		ADD		SI,1			; SI+1
		CMP		SI,5			; 比较SI与5
		JAE		error			; SI >= 5 跳转到error
		MOV		AH,0x00
		MOV		DL,0x00			; A驱动器
		INT		0x13			; 重置驱动器
		JMP		retry
next:
		MOV		AX,ES			; 把内存地址后移0x200
		MOV		ES,AX			; 因为没有ADD ES, 0x020指令, 所以这里稍微绕个弯
		ADD		CL,1			; CL + 1
		CMP		CL,18			; CL与18进行比较
		JBE		readloop		; CL <= 18 跳转到readloop

		MOV		CL,1
		ADD		DH,1
		CMP		DH,2
		JB		readloop		; DH < 2 跳转到readloop
		MOV		DH,0
		ADD		CH,1
		CMP		CH,CYLS
		JB		readloop		; CH < CYLS 跳转到readloop
```

读完C0-H0-S18, 下一个扇区就是C0-H1-S0, 按照顺序, 再往后就会自然而然的读到C1-H0-S1,这个要一直读到C9-H1-S18

+ JB: jump if below
+ EQU: equal 相当于C语言的`#define`指令, 用来声明常数
+ `CYLS  EQU  10`的意思就是CYLS = 10(现在先暂时定义成10个柱面)(CYLS: cylinders)
+ 磁头最多只有两个, 所以DH >= 2 的时候要先把DH置0才能读取下一个柱面
+ 同理, 最多只有S18, 所以CL需要和18比大小, 否则CL直接变成1了

#### 着手开发操作系统

这里需要将编译生成的sys文件保存到磁盘映像文件`haribote.img`里, 然后在64位上跑会有点问题, 可以参考下面这个链接

[在Windows 7 64位环境中制造的自制操作系统 第一天（第一部分） 安装无法启动问题 - T.Shion的日记 (hatenablog.jp)](https://t-shion.hatenablog.jp/entry/2015/11/23/031225)

会输出一些奇怪的东西, 别管

`make img`之后去看.img文件, 可以看到0x2600保存着文件名,0x4200可以看见`F4 EB FD`(这个就是之前的haribote.sys的内容)

![004](./004.png)

![005](./005.png)

所以, 一般向一个空软盘保存文件的时候, 

+ 文件名会写在0x2600以后的地方
+ 文件的内容会写在0x4200以后的地方

#### 从启动区执行操作系统

现在要执行位于磁盘映像文件上位于0x4200位置上的代码, 但是现在的程序是从启动区开始的, 需要把磁盘上的内容装载到0x8000号地址, 所以磁盘上的内容需要往后移动0x8000, 所以磁盘0x4200位置上的内容应该位于内存0x8000+0x4200 = 0xc200号地址上

对haribote.nas进行修改

```
; haribote-os
; TAB=4

		ORG		0xc200			
fin:
		HLT
		JMP		fin

```

现在程序的装载地址就是0xc200, 然后直接halt循环了

#### 确认操作系统的执行情况

上面的代码跑出来是没有结果的, 所以

```
; haribote-os
; TAB=4

		ORG		0xc200			; 这个程序会被读取到哪里?
		MOV		AL,0x13			; VGA图形，320x200x8bit色彩
		MOV		AH,0x00
		INT		0x10
fin:
		HLT
		JMP		fin

```

这回用了BIOS INT=0x10, AH = 0x00的函数, 这个是拿来设置显卡模式的, 现在用的是VGS图形模式, 320 * 200 * 8 位彩色模式, 调色板模式(好吧我也不知道这么多模式都有什么用处......)

![](./006.png)

make run之后的结果是这样的

![](./007.png)

因为变成了图形模式, 所以画面一篇漆黑并且没有鼠标

#### 32位模式前期准备

CPU有16位和32位两种模式, 两种模式下寄存器和代码不互通

32位模式下的内存容量远远大于1MB, 另外, CPU的自我保护功能(识别出可以的机器语言进行屏蔽, 以免破坏操作系统)在16位下不能使用, 所以还是32位好

但是32位下不能调用BIOS功能(BIOS是16位写的), 所以现在开始要用BIOS就一开始用完, 

之前的代码我们还需要获取键盘状态, 所以这次还得修改haribote.nas

```
; haribote-os
; TAB=4

; 有关BOOT_INFO
CYLS	EQU		0x0ff0			; 引导扇区设置
LEDS	EQU		0x0ff1
VMODE	EQU		0x0ff2			; 关于颜色数量的信息。多少比特的颜色?
SCRNX	EQU		0x0ff4			; 分辨率的X
SCRNY	EQU		0x0ff6			; 分辨率的Y
VRAM	EQU		0x0ff8			; 图形缓冲器的起始地址
		ORG		0xc200			; 这个程序会被读取到哪里?
		MOV		AL,0x13			; VGA图形，320x200x8bit色彩
		MOV		AH,0x00
		INT		0x10
		MOV		BYTE [VMODE],8	; 记录画面模式
		MOV		WORD [SCRNX],320
		MOV		WORD [SCRNY],200
		MOV		DWORD [VRAM],0x000a0000
; 让BIOS告诉你键盘的LED状态
		MOV		AH,0x02
		INT		0x16 			; keyboard BIOS
		MOV		[LEDS],AL

fin:
		HLT
		JMP		fin

```

设置画面模式之后, 把画面模式的信息保存在了内存里, 方便后面修改备用

VRAM里保存的是0xa0000(VMA指的是显卡内存, video RAM), 它的各个地址对应的是画面上的像素

不同画面模式的像素数目不一样, 所以可以使用的内存也不一样, 所以前面才需要把VRAM的地址保存在BOOT_INFO中备用

#### 开始导入C语言

好耶, 终于进入C语言的部分了

<img src="./008.jpg" style="zoom: 15%;" />

**从C语言到汇编语言**

+ 使用ccl.exe将bootpack.c变成bootpack.gas

  这里用的编译器是用gcc魔改出来的, gcc是以gas汇编语言为基础, , 输出的是gas的源程序, 不能直接转变成nask

+ 使用gas2nask.exe从bootpack.gas变成bootpack.nas

+ 使用nask.exe从bootpack.nas变成bootpack.obj(目标文件)

+ 使用obi2bim.exe从bootpack.obj变成bootpack.bim

  目标文件还不能直接拿来用, 还需要link一下库函数, (虽然这里只有一个halt但是该有的流程不能少)

  这个也是那本书的作者自己魔改的, 意思是binary image, 可以把目标文件之间相互连接, 构成完整的机器语言

+ 使用bim2hrb.exe从bootpack,bim变成bootpack.hrb

#### 实现HLT(harib00j)

```
; naskfunc
; TAB=4

[FORMAT "WCOFF"]				; 制作目标文件的模式	
[BITS 32]						; 制作32位模式用的机器语言


; オブジェクトファイルのための情報

[FILE "naskfunc.nas"]			; 源文件名信息

		GLOBAL	_io_hlt			; 程序中包含的函数名


; 以下是实际的函数

[SECTION .text]		; 目标文件写了这些之后再写程序

_io_hlt:	; void io_hlt(void);
		HLT
		RET

```

这里用汇编语言写了io_hlt, 用汇编写的函数, 之后还需要于bootpack.obj进行连接, 所以也需要变成obj文件, 所以将输出设定为WCOFF模式, 另外, 还要设定成32位机器语言模式

在nask目标文件的模式下(.nas文件), 必须设定文件名信息, 函数名前面还必须的加上`_`, 需要连接的函数, 都必须要用`GLOBAL`指令声明

以下是bootpack.c

```
/* 告诉C编译器你有一个函数是用其他文件做的。 */

void io_hlt(void);

/*  是函数声明却不用{ }, 而用 ; 
	这表示的意思是: 函数是在别的文件中,你自己找找XD */

void HariMain(void)
{

fin:
	io_hlt(); /* 执行naskfunc.nas里面的_io_hlt */
	goto fin;

}

```

现在make run以下, 虽然还是黑屏, 但是我们跨出了从二进制到汇编到C语言的巨大进步, 好欸

## Day 4 C语言与画面显示的练习

#### 用C语言实现内存写入

#### 条纹图案(harib01b)

#### 挑战指针(harib01c)

#### 指针的应用(1)(harib01b)

#### 指针的应用(2)(harib01c)

#### 色号设定

#### 绘制矩形

#### 今天的成果

## Day 5 结构体, 文字显示与 GDT/IDT初始化

## Day 6 分隔编译与中断处理

## Day 7 FIFO与鼠标控制

   

## Finally

最后放几首一直陪着我的歌吧

每天熬夜都是这几首歌在陪着我XD

1. [敢问云间宿]:https://www.bilibili.com/video/BV1H14y1M7tj

2. [殉道之人]:https://monster-siren.hypergryph.com/music/125083

3. [踏上旅途(目前还没有官方的版本, 我发发我自己常听的版本)]: https://www.bilibili.com/video/BV1uV4y1d7bT/?share_source=copy_web&vd_source=89823c744096e29cafad55e0e1765912

4. [野火(目前还没有官方的版本, 我发发我自己常听的版本)]: https://www.bilibili.com/video/BV1uM411G7hM/?share_source=copy_web&vd_source=89823c744096e29cafad55e0e1765912

6. [长梦应觉]: https://www.bilibili.com/video/BV1wv4y1F7kK/?share_source=copy_web&vd_source=89823c744096e29cafad55e0e1765912

:musical_note:  **昨日已去谁复烦忧　我更何求**:musical_note: 

![丹恒好帅啊](./000.png)
