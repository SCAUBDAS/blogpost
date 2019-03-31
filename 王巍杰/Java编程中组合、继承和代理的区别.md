最近在看《Java编程思想》这一本书，看到第７章复用类，学习的时候查了查网上的资料，感觉都说得不是很清楚，就来说说我的看法。
复用类即实现代码复用的类，Java中大概分为三种：组合、继承、代理（实际Java没有提供对代理的直接支持）。

 1. 组合：要使用A类的方法，为了不改变其原有结构，在一个新的类B中创建A类的一个对象a，以创建的这个对象a来调用A类的方法。
 2. 继承：要使用A类的方法，不改变其原有结构，创建一个类B，继承自A，这个类B拥有A类的所有方法和属性，并能自身做扩展,拥有A所没有的属性和方法。
 3. 代理：要使用A类的方法，不改变其原有结构，在一个新的类B中创建A的对象a，并且在B中创建方法fb，方法内部是a调用A类的方法，但是使用时是B的对象调用其自身方法fb。
 
 下面来看例子()：
 1.组合

```java
//组合
//：multiplexing/Getresult.java
class Calculater {
	Calculater() {
		System.out.println("use composition");
	}
	int Sum() {
		int res = 0; 
		for (int i = 0; i < 1000;i++) {
			res += i;
		}
		return res;
	}
}

public class Getresult {
	private Calculater cal;
	public Getresult(){
		cal = new Calculater();
	}
	public static void main(String[] args) {
		Getresult result = new Getresult();
		System.out.println(result.cal.Sum());
	}
}/*output
use composition
499500
*/
```

2.继承

```java
//：multiplexing/GetresultInheritance.java
public class GetresultInheritance extends Calculater{
	GetresultInheritance(){
		System.out.println("use inheritance");
	}
	public static void main(String[] args) {
		GetresultInheritance result = new GetresultInheritance();
		System.out.println(result.Sum());
	}
}
/*output
use inheritance
499500
*/
```

3.代理

```java
//:multiplexing/GetResultProxy.java
public class GetResultProxy {
	private Calculater cal = new Calculater();
	public long GetSum(){
		System.out.println("use proxy");
		return cal.Sum();
	}
	public static void main(String[] args) {
		GetResultProxy result = new GetResultProxy();
		System.out.println(result.GetSum());
	}
}
/*output
use proxy
499500
*/
```

那么三种复用方式的访问权限又是怎样的呢？组合是调用类

 1.组合
 

```java
public class Getresult {
	private Calculater cal;
	public Getresult(){
		cal = new Calculater();
	}
	public static void main(String[] args) {
		Getresult result = new Getresult();
		System.out.println("use compositon");
		result.cal.B();
		result.cal.C();
	}
}

class Calculater {
	Calculater() {
	}
	private void A () {
		System.out.println("private method");
	}
	protected void B() {
		System.out.println("protected method");
	}
	public void C() {
		System.out.println("public method");
	}
}/*output
use compositon
protected method
public method
*/
```
2.继承

```java
public class GetresultInheritance extends Calculater{
	GetresultInheritance(){
		System.out.println("use inheritance");
	}
	public static void main(String[] args) {
		GetresultInheritance result = new GetresultInheritance();
		result.B();
		result.C();
	}
}/*output
use inheritance
protected method
public method
*/
```
3.代理

```java
public class GetResultProxy {
	private Calculater cal = new Calculater();
	public void B() {
		cal.B();
	}
	public void C() {
		cal.C();
	}
	
	public static void main(String[] args) {
		GetResultProxy result = new GetResultProxy();
		System.out.println("use proxy");
		result.B();
		result.C();
	}
}/*output
use proxy
protected method
public method
*/
```
可以看到，实际上三种方式都只能访问protected和public方法，调用private方法则会报错。虽然组合使用的是该类的成员对象调用方法，但是Java的private成员方法只支持包含该方法的类内部访问。

