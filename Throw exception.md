# 抛出异常
## 异常的传播

当某个方法抛出了异常时，如果当前方法没有捕获异常，异常就会被抛到上层调用方法，直到遇到某个try ... catch被捕获为止：

// exception
public class Main {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        process2();
    }

    static void process2() {
        Integer.parseInt(null); // 会抛出NumberFormatException
    }
}

通过printStackTrace()可以打印出方法的调用栈，类似：
java.lang.NumberFormatException: null
    at java.base/java.lang.Integer.parseInt(Integer.java:614)
    at java.base/java.lang.Integer.parseInt(Integer.java:770)
    at Main.process2(Main.java:16)
    at Main.process1(Main.java:12)
    at Main.main(Main.java:5)

printStackTrace()对于调试错误非常有用，
上述信息表示：NumberFormatException是在java.lang.Integer.parseInt方法中被抛出的，
从下往上看，调用层次依次是：

main()调用process1()；
process1()调用process2()；
process2()调用Integer.parseInt(String)；
Integer.parseInt(String)调用Integer.parseInt(String, int)。

查看Integer.java源码可知，抛出异常的方法代码如下：

public static int parseInt(String s, int radix) throws NumberFormatException {
    if (s == null) {
        throw new NumberFormatException("null");
    }
    ...
}
并且，每层调用均给出了源代码的行号，可直接定位。

## 抛出异常

当发生错误时，例如，用户输入了非法的字符，我们就可以抛出异常。

如何抛出异常？参考Integer.parseInt()方法，抛出异常分两步：

1.创建某个Exception的实例；
2.用throw语句抛出。

下面是一个例子：

void process2(String s) {
    if (s==null) {
        NullPointerException e = new NullPointerException();
        throw e;
    }
}

实际上，绝大部分抛出异常的代码都会合并写成一行：

void process2(String s) {
    if (s==null) {
        throw new NullPointerException();
    }
}

如果一个方法捕获了某个异常后，又在catch子句中抛出新的异常，就相当于把抛出的异常类型“转换”了：

void process1(String s) {
    try {
        process2();
    } catch (NullPointerException e) {
        throw new IllegalArgumentException();
    }
}

void process2(String s) {
    if (s==null) {
        throw new NullPointerException();
    }
}
当process2()抛出NullPointerException后，被process1()捕获，然后抛出IllegalArgumentException()。

如果在main()中捕获IllegalArgumentException，我们看看打印的异常栈：

// exception
public class Main {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException();
        }
    }

    static void process2() {
        throw new NullPointerException();
    }
}
打印出的异常栈类似：
java.lang.IllegalArgumentException
    at Main.process1(Main.java:15)
    at Main.main(Main.java:5)

这说明新的异常丢失了原始异常信息，我们已经看不到原始异常NullPointerException的信息了。

为了能追踪到完整的异常栈，在构造异常的时候，把原始的Exception实例传进去，新的Exception就可以持有原始Exception信息。对上述代码改进如下：

// exception
public class Main {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException(e);
        }
    }

    static void process2() {
        throw new NullPointerException();
    }
}

运行上述代码，打印出的异常栈类似：

java.lang.IllegalArgumentException: java.lang.NullPointerException
    at Main.process1(Main.java:15)
    at Main.main(Main.java:5)
Caused by: java.lang.NullPointerException
    at Main.process2(Main.java:20)
    at Main.process1(Main.java:13)

注意到Caused by: Xxx，说明捕获的IllegalArgumentException并不是造成问题的根源，

根源在于NullPointerException，是在Main.process2()方法抛出的。

在代码中获取原始异常可以使用Throwable.getCause()方法。

如果返回null，说明已经是“根异常”了。

有了完整的异常栈的信息，我们才能快速定位并修复代码的问题。

捕获到异常并再次抛出时，一定要留住原始异常，否则很难定位第一案发现场！

如果我们在try或者catch语句块中抛出异常，finally语句是否会执行？例如：

// exception
public class Main {
    public static void main(String[] args) {
        try {
            Integer.parseInt("abc");
        } catch (Exception e) {
            System.out.println("catched");
            throw new RuntimeException(e);
        } finally {
            System.out.println("finally");
        }
    }
}

catched
finally
Exception in thread "main" java.lang.RuntimeException: java.lang.NumberFormatException: For input string: "abc"
    at Main.main(Main.java:8)
Caused by: java.lang.NumberFormatException: For input string: "abc"
    at ...
 
第一行打印了catched，说明进入了catch语句块。
第二行打印了finally，说明执行了finally语句块。

因此，在catch中抛出异常，不会影响finally的执行。
JVM会先执行finally，然后抛出异常。

## 异常屏蔽
如果在执行finally语句时抛出异常，那么，catch语句的异常还能否继续抛出？例如：

// exception
public class Main {
    public static void main(String[] args) {
        try {
            Integer.parseInt("abc");
        } catch (Exception e) {
            System.out.println("catched");
            throw new RuntimeException(e);
        } finally {
            System.out.println("finally");
            throw new IllegalArgumentException();
        }
    }
}

执行上述代码，发现异常信息如下：
catched
finally
Exception in thread "main" java.lang.IllegalArgumentException
    at Main.main(Main.java:11)

这说明finally抛出异常后，原来在catch中准备抛出的异常就“消失”了，
因为只能抛出一个异常。
没有被抛出的异常称为“被屏蔽”的异常（Suppressed Exception）。

在极少数的情况下，我们需要获知所有的异常。
如何保存所有的异常信息？
方法是先用origin变量保存原始异常，然后调用Throwable.addSuppressed()，
把原始异常添加进来，最后在finally抛出：

// exception
public class Main {
    public static void main(String[] args) throws Exception {
        Exception origin = null;
        try {
            System.out.println(Integer.parseInt("abc"));
        } catch (Exception e) {
            origin = e;
            throw e;
        } finally {
            Exception e = new IllegalArgumentException();
            if (origin != null) {
                e.addSuppressed(origin);
            }
            throw e;
        }
    }
}

当catch和finally都抛出了异常时，虽然catch的异常被屏蔽了，但是，finally抛出的异常仍然包含了它：

Exception in thread "main" java.lang.IllegalArgumentException
    at Main.main(Main.java:11)
Suppressed: java.lang.NumberFormatException: For input string: "abc"
    at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.base/java.lang.Integer.parseInt(Integer.java:652)
    at java.base/java.lang.Integer.parseInt(Integer.java:770)
    at Main.main(Main.java:6)

通过Throwable.getSuppressed()可以获取所有的Suppressed Exception。

绝大多数情况下，在finally中不要抛出异常。因此，我们通常不需要关心Suppressed Exception。

## 提问时贴出异常

异常打印的详细的栈信息是找出问题的关键，许多初学者在提问时只贴代码，不贴异常，相当于只报案不给线索，福尔摩斯也无能为力。






















































