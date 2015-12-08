## Groovy特点 ##
- Groovy语法允许省略变量类型。
- Groovy允许定义简单脚本，同时无需定义正规的class对象。
- 除非另行制定，Groovy的所有内容都是public。
- Groovy中的一切都是标准的Java对象，都是继承自Java Object，没有int，只有Integer。


## Groovy集合 ##
对比Java中使用集合的方式——导入java.util类，初始化集合，将项加入集合，这三个步骤都增加不少代码。而Groovy可以直接在语言内使用集合，不需要导入专门的类，也不需要初始化对象，同时，Groovy也使集合或列表的操作变得更加容易。
### 可以将范围当集合 ###
范围表达式"0..4"代表数字的集合——0,1,2,3,4。

    def range = 0..4
	println range.class
	assert range intanceof List

结果得出范围就是List的实例。另外，assert 表达式， 当表达式为真时，通过（没有提示）；当表达式为假时，中断程序执行，打印出错信息。
###添加项###
Groovy有三种添加项的方式，add()方法（与Java类似）;<<（Groovy重载了这个运算符，可以向列表中添加项）；通过下标添加。

    def coll = ["Groovy", "Java", "Ruby"]
	println coll.class
	assert coll instanceof Collection
	assert coll instanceof ArrayList
	coll.add("Python")
	coll[6] = "Perl"
	coll << "Smalltalk"
	println coll
结果：

    class java.util.ArrayList
	[Groovy, Java, Ruby, Python, null, null, Perl, Smalltalk]

可知，下标和Java一样，从0开始。另外，add和<<都是在列表的末尾添加，而下标可以在任意位置添加（位置之前不存在的元素置为null,如果在已经存在元素的下标添加就是修改对应位置的元素）。

Groovy还可以在集合中添加或去掉集合（元素）。

   	def coll = ["Groovy", "Java", "Ruby"]
	coll -= "Ruby"
	println coll

	coll -= "Ruby"
	println coll

	coll += ["Ruby"]
	println coll 

	coll += ["Ruby"]
	println coll 
结果：
    [Groovy, Java]
	[Groovy, Java]
	[Groovy, Java, Ruby]
	[Groovy, Java, Ruby，Ruby]
可以知道，当集合中减去不存在的元素时，集合不会有变化，当集合重复添加同一个元素时，都会将元素重复放置尾部。 
 
当集合中本身存在多个ruby时，减去ruby,会将所有的ruby都会删除。
###检索项###
可以通过下标检索，