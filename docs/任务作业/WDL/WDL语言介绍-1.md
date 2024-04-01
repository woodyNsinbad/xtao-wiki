# WDL语言介绍-1


WDL (Workflow Description Language) 是一种专门用于描述工作流程的编程语言。它被广泛应用于生物信息学和数据处理领域，特别是在基因组学、转录组学和蛋白质组学等领域。

WDL 的设计目标是简洁、灵活和可扩展。它采用了结构化的编程风格，以描述工作流程的不同阶段和步骤，可以方便地定义输入、输出、任务和工作流之间的依赖关系。

WDL 支持强大的类型系统，包括原始数据类型（如文件、字符串、整数、布尔值等）和复杂数据类型（如数组、Map、Object 等）。它还提供了丰富的语法和操作符，以实现条件判断、scatter 操作、错误处理和子任务的调用。

WDL 的优点在于它的**可读性**和**可维护性**。其结构清晰，易于理解和修改，使团队成员能够协作开发和维护复杂的工作流程。此外，它还与常用的工作流程管理系统兼容，便于工作流程的部署和执行。

---

## 1. WDL脚本基本结构

开发至今，WDL语法已经经历了数个版本，目前比较重要的版本有 WDL [draft–2](https://github.com/openwdl/wdl/blob/main/versions/draft-2/SPEC.md)、[version 1.0](https://github.com/openwdl/wdl/blob/main/versions/1.0/SPEC.md) 、[version 1.1](https://github.com/openwdl/wdl/blob/main/versions/1.1/SPEC.md) 。本次以 WDL version 1.1 语法为基础。WDL 主要分为两级结构：*workflow*与*task*

![wdl基本结构.png](/pic/wdl/wdl基本结构.png)

### 1.1 task 结构

+ *输入*： 输入指程序运行过程中，需要指定的文件、数值、字符串等信息。
+ *命令行*（command）：等同于Linux shell 环境，可以支持多行。

> 命令行在不同的WDL语法版本中支持的表现形式略有不同，可用`{...}`限定，也可用`<<< ... >>>`限定。两者的区别在于对`python`代码块使用时，应优先使用`<<< ... >>>`的形式。同时，在command中使用 `<<< ... >>>` 和变量使用 `~{}`表示能更有效的区分变量的作用范围。
{.is-info}

+ *输出*（output）： 规定输出结果，并为后续操作提供可以引用的信息，需要规定输出的类型。支持WDL自定义函数的引用
+ *运行参数*（runtime）：记录运行中是使用的资源信息，包括docker、CPU、memory、GPU等信息。
+ *meta信息*（非必须）：记录辅助信息，如作者、作者联系方式等

### 1.2 workflow 结构

workflow 是提交作业的基本单位。其结构是对task的组织（也就是再workflow中call task）。虽然是按顺序结构书写，但是bioflow 对WDL解析时，会按照并行方式解析。

WDL的基本结构与Linux shell不同，其书写前后关系并不代表执行的前后关系。执行顺序是由task之间的依赖关系来确定的。
+ **if-控制语句**

	与一般语言结构区别不大。但是需要注意，WDL解析器语法规定比较严格，因此其判断部分应为Bloon型数据。
![if-语句.png](/pic/wdl/if-语句.png)

> WDL语言的结构分配中，没有else关键字。但在表达式中，存在`if-then-else`表达式。
{.is-warning}

+ **scatter-并发语句**

	scatter结构与Linux shell中的for\while不同，可以同时执行多条相同形式的命令，从而减少计算时间，增加硬件设备的利用率。
![scatter-语句.png](/pic/wdl/scatter-语句.png)

> scatter模块返回的结果可以用 Array 类型的获取并在后续task中引用。
{.is-success}

```
workflow A{
	Array[Int] numbers 
  
	scatter(number in numbers){
  	call add_1 {input: x= number}
  }
  Array[Int] x_add_1 = add_1.y
  
}

task add_1{
	Int x
	command{ ... }
  output {
  	Int y = x + 1
  }
}

```

由于biocli会根据文件依赖关系自动解析执行顺序。因此用户无需担心书写顺序。下面为一个解析实例

![poros界面展示的流程图实例.png](/pic/wdl/poros界面展示的流程图实例.png)


--- 

## 2. WDL语言数据类型

WDL语言中的“变量”，其实并不是指一般编程语言中的变量，更准确的叫法应该叫做**占位符**：即一旦赋初值之后，不能在后面的运行过程中变化。其次，在与Perl等脚本语言不同，WDL语言中对变量类型有着较为严格的规定，主要体现在字符串（String）和文件（File）通常情形下不能混用、if判断条件式的值应为布尔型等情况。

+ 基础数据类型 例如：File、String、Int、Float 
+ 复合数据类型 例如：Array、Struct、Pair、Map

> 布尔型数据变量 Boolean
其取值范围为True 和 False 两种。
注意，在给其付初值的时候，不需要加上引号
{.is-warning}

```json
{
	"myworkflow.bloanvalue" : True 			//正确写法
	"myworkflow.bloanvalue" : "True" 		//错误写法
}
```


### 2.1 占位符使用原则

WDL变量的使用遵循“先声明，再使用”的原则。声明变量时，需要规定类型,例如：

```
File fastq 
String prefix
Int insert_size = 170
```

其作用范围包含workflow和task，占位符的作用域独立。简单来说，task和workflow的之间占位符名是隔离的。


> 再次强调，占位符仅仅起占位作用，并不能适合做自增、改变值等操作。
{.is-info}


### 2.2 占位符使用实例

+ 基础用法：可直接通过变量表示符，直接引用

```
task A {
	File fastq 
	command{
		cat ~{fastq}
    cat ${fastq}
  }
  ...
 }
```

+ 使用 `sep`进行链接 : 一般应用于Array类型变量
```
task B {
	Array[File] bams 
	command{
		GATK combinebam ~{sep="-I" bams}
  }
  ...
 }
```

+ 使用 `true-false` : 进行判断
```
task C {
	Boolean normalize_flag 
	command{
  	plot_heatmap.R ~{true="-m " false="" normalize_flag}
  }
  ...
 }
```
+ 设置默认值 `default`

```
task D {
	String? Insert_size   
	command{
  	call_sv --insertsize ~{default="170 " Insert_size}
  }
  ...
 }
```


### 2.3 占位符的约束

+ 一般占位符： 一旦定义，在输入必须存在 
```
String prefix 
```
+ 可选占位符： 在输入时，可以不存在

```
String? something 
```
+ 非空数组： 输入定义时，必须不为空（只对Array类型起作用）

```
Array[File]+ my_inputs 
```

### 2.4 复杂数据类型

基础数据类型表达能力十分有限，且对代码可读性造成一定的障碍。因此WDL语法中支持用户定义复杂变量类型。

例如：
- `Array` 类型为一系列**元素类型相同**的数组。
- `Pair` 类型为单一对应关系，例如：
```
Pair raw_reads = { "r1.fastq.gz" ,  "r2.fastq.gz" }
File read1 = raw_reads.left  ## 引用第一个变量时使用`.left`
File read1 = raw_reads.right ## 引用第二个变量时使用`.right`
```
- `Map` 类型为键值对类型的占位符，例如：
```
Map[String, Int] string_to_int = { "a": 1, "b": 2 }
```


另外，用户可通过`struct`自己构建变量类型，例如：

```
struct Sample_Reads{
        File read1
        File read2
        String prefix
}

struct Ref_Files{
        File fa
        File gtf
        File? star_index
        File? bowtie2_index
        File? hisat2_index
        File bed
}
```
在定义变量时，用例如下：

```
workflow base_rna_seq {
	Array[Sample_Reads] raw_samples
 	Ref_Files ref
						...
 }

```

> 目前除了struct之外，还可以通过Object类型定义用户自定义的占位符，但目前WDL开发组已经不再推荐使用Object类型，因此推荐使用struct来定义占位符类型。
{.is-warning}


---

## 3 内置函数

WDL提供了丰富的内置函数，可以用于输入输出传递、变量转化等操作，但是需要注意的是，与其他编程语言不同，WDL函数主要的目的在于粘合task，**一般数据处理操作不建议使用**。

> 目前对WDL支持的内置函数，可参考对应说明页面。目前 `biocli` 和`wdlrun` 对WDL 1.1版本的内置函数全部支持，用户可放心使用。但`cromwell`目前存在对WDL语法支持有限，使用时需谨慎。
{.is-info}

内置函数主要分为大三类：

+ **输入输出** ：如stdout(),stderr(),read_tsv()…
+ **信息获取类** ：defined(), glob() ,basename(),select_first()
+ **变量操作**：prefix(), sub()

---

### 3.1 输入输出函数

可以细分为三类：

- 标准输入输出控制：stdout() \ stderr()
- 文件入读：read_tsv() \ read_lines() … 
- 文件输出：write_tsv() \ write_lines() … 

具体功能使用和实例，可参考WDL标准语法的用例。

### 3.2 信息获取类函数

WDL信息获取函数与`perl`或`python` 类似。例如：

+ glob() 函数 # 获取目录下某一类型文件。
+ defined() 函数 # 判断某变量是否被定义
+ **select_first() 函数** # 用于选择首个有定义的变量，常用于if模块后的变量选择。
+ base_name() 函数 # 获取文件或字符串的子串

### 3.3 变量操作类函数

变量操作函数，可以在一定程度上对变量进行基础运算和修改,例如：

+ prefix() ： # 为字符串数组变量增加前缀字符。
+ sub() ： # 提供正则表达式功能

--- 


