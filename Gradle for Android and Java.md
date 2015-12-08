
##Groovy Type##
Groovy是动态语言，可以随意改变类型，直接用def定义变量类型。

    def foo = 6.5

	println "foo has value: $foo"  //foo has value: 6.5
	
	println "Let's do some math. 5 + 6 = ${5 + 6}" // Let's do some math. 5 + 6 = 11
	
	foo = "a string"
	println "foo is now of type: ${foo.class} and has value $foo " // foo is now of type: class java.lang.String and has value a string
##Groovy Functions##

参数不需要类型，这样重载就很方便，另外，函数默认返回值，是函数体内最后一行的值。

    def doubleIt(n) {
		n + n  // 不需要return语句,默认最后一行的值作为返回值
	}
	
	def foos = 5
	println "doubleIt($foos) = ${doubleIt(foos)}"  // doubleIt(5) = 10
	
	foos = "foobar"
	println "doubleIt($foos) = ${doubleIt(foos)}"  // douleIt("foobar") = "foobarfoobar"
groov中有参数的函数，调用时，可以不用括号。

    def noArgs() {
		println "Called the no args function"
	}
	
	def oneArgs(x) {
		println "Called the 1 arg function with $x"
		x
	} 
	
	def twoArgs(x, y) {
		println "Called the 2 arg function with &x and $y"
		x + y
	}

	oneArgs 300   // Called the 1 arg function with 300
	twoArgs 200, 300 // Called the 2 arg function with 200 and 300
	noArgs // 出错
	noArgs() // Called the no args function
##Groovy Closures and Objects##

    def foo = "One million dollars"
	def myClosure = {
		println "Hello from a closure"
		println "The value of foo is $foo" // 注意这里的foo变量
	}

	myClosure()

	def bar = myClosure
	def baz = bar
	baz()

还有以下的闭包简写

	def doubleIt = { x -> x + x }
	
	def applyTwice(func, arg) {
		func(func(arg))
	}
	
	def foo = 5
	def fooDoubleTwice = applyTwice(doubleIt, foo)
	println "Applying doubleIt twice to $foo equals $fooDoubleTwice"

结果是20

    def myList = ["Gradle", "Groovy", "Android"]

	def printItem = {item -> println "List item: $item"}
	myList.each(printItem)

结果：
    List item: Gradle
    List item: Groovy
    List item: Android

闭包可以delegate

    class GroovyGreeter {
	String greeting = "Default greeting"
	def printGreeting() {
		println "Greeting: $greeting"
	}
	}
	
	def myGroovyGreeter = new GroovyGreeter()
	
	def greetingClosure = {
		greeting = "Setting the greeting from closure"
		printGreeting()
	}
	
	greetingClosure.delegate = myGroovyGreeter
	greetingClosure()


一整个build文件代理了一个project

properties或者project的methods


task之间的依赖，可以使用dependsOn，如
task hello << {
	println 'Hello world'
}

task intro(dependsOn:hello) << {
	println "I'm Gradle"
}

task A(dependsOn:['hello', 'B', 'C']) << {}



task taskX << {
	println "taskX"
}

task taskY << {
	println "taskY
}

taskX.dependsOn taskY

taskX.dependsOn {
	tasks.findAll {task -> task.name.startsWith('lib')}
}

task lib1 << {
	println "lib1"
}


gradle projects查看当前项目中包含的所有子项目


##Task Configuration##
一个build.gradle代理（delegate）了一个Project的对象。在Gradle DSL中所有的关键字都是Project对象的属性或是方法。

Project对象有一个方法叫task，用来声明tasks

    project.task("myTask1")  //创建一个名叫myTask1的Task，并将其放入project中，
	task("myTask2")  // 因为build.gradle代理了整个project，所以可以省略掉project，直接调用task方法，和上面的效果是一样的
	task "myTask3" // 因为调用groovy的方法可以不加括号，所以这种方式也可以
	task myTask4 // 因为gradle使用Groovy的高级特性，叫做abstract syntax tree transformation，使得在编译时改变了声明的语法，因此，也可以省略掉双引号。
	myTask4.description = "This is what is shown in the task list"  // task的属性
	myTask4.doLast {  // 最后执行action的内容
		println "Do this last"	
	}
	myTask4 << {
		println "Do this last"
	}
	myTask4.leftShift {
		println "Do this last"
	}
	myTask4.doFirst {
		println "Do this first"
	}
	task myTask5 << {  //创建一个task，并立即给其加入一个action
		println "Here is how to delcare a task and give it an action immediately"	
	}

	task myTask6 {  创建一个task，传给他一个configuration闭包，这个configuration闭包代理了这个task对象
		description "Here is a task with a configuration block" 
		group "Some group"  // 对于任意的属性，groovy都默认的创建与属性名同名的setter方法，实际上是 group("Some group")，由于goovy方法，省略了括号，需要注意一点，当给属性赋一个集合时，需要用等号。
		doLast {
			println "Here is the action"
		}
	}

	task myTask7(descripton: "Another description") {  //这种在括号中设置属性
		
	}


##Task Dependencies and Ordering##
A depend On B  ---- 想要执行A，需要先执行B

穿鞋之前先穿袜子

    task putOnSocks {
		doLast {
			println "Putting on Socks"
		}
	}

	task putOnShoes {
		dependsOn "putOnSocks"  //依赖多个时， dependsOn ['A', 'B', 'C']
		doLast {
			println "Putting on Shoes"
		}
	}

A finalized By B ----A完成后执行B

早餐之后需刷牙

    task eatBreakfast {
		finalizedBy "brushYourTeeth"
		doLast {
			println "eat breakfast"
		}
	}

	task brushYourTeeth {
		doLast {
			println "Brush Teeth"
		}
	}


shouldRunAfter-----A和B之间不存在依赖关系，但是有时我们希望A在B之前执行，比如A是耗时较小的任务，B是耗时较长的任务，当定义了A shouldRunAfter B，那么，在命令行中执行 gradle A, B, 就会得到A在B之后执行.

    task D {
		dependsOn tasks.matching {task -> task.name.startsWith("putOn")}
		doLast {
			println "All geared up!"
		}
	}

    task deleteBuild(type: Delete) {
		delete 'build'	
	}


每个task都有input和output，gradle为每个task的input和output加上snapshot快照，当input和output都没有变时，此task就是up-to-date，就不需要执行。


task hello(type:Copy) {}这里type是要确定这个task实例所属的类型，如果不指定，就是默认的类型。可以自己创建type，新建一个类，继承自DefaultTask


gradle --stacktrace (-s)
gradle --full-stacktrace (-S)


defaultConfig设置manifest中的attributes

applicationVariants.all {
	if (buildType.name == 'debug') {
		javaCompile.options.compilerArgs = ['-verbose']
	}
}