# catch exception

在Java中，凡是可能抛出异常的语句，都可以用try ... catch捕获。
把可能发生异常的语句放在try { ... }中，然后使用catch捕获对应的Exception及其子类。

## 多catch语句

可以使用多个catch语句，每个catch分别捕获对应的Exception及其子类。

JVM在捕获到异常后，会从上到下匹配catch语句，
匹配到某个catch后，执行catch代码块，然后 不再 继续匹配。

简单地说就是：多个catch语句只有一个能被执行。例如：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException e) {
        System.out.println(e);
    } catch (NumberFormatException e) {
        System.out.println(e);
    }
}

存在多个catch的时候，catch的顺序非常重要：子类必须写在前面。例如：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException e) {
        System.out.println("IO error");
    } catch (UnsupportedEncodingException e) { // 永远捕获不到
        System.out.println("Bad encoding");
    }
}
对于上面的代码，UnsupportedEncodingException异常是永远捕获不到的，

因为它是IOException的子类。

当抛出UnsupportedEncodingException异常时，会被catch (IOException e) { ... }捕获并执行。

因此，正确的写法是把子类放到前面：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (UnsupportedEncodingException e) {
        System.out.println("Bad encoding");
    } catch (IOException e) {
        System.out.println("IO error");
    }
}

## finally语句

无论是否有异常发生，如果我们都希望执行一些语句，例如清理工作，怎么写？

可以把执行语句写若干遍：正常执行的放到try中，每个catch再写一遍。例如：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
        System.out.println("END");
    } catch (UnsupportedEncodingException e) {
        System.out.println("Bad encoding");
        System.out.println("END");
    } catch (IOException e) {
        System.out.println("IO error");
        System.out.println("END");
    }
}

上述代码无论是否发生异常，都会执行System.out.println("END");这条语句。

那么如何消除这些重复的代码？Java的try ... catch机制还提供了finally语句，finally语句块保证有无错误都会执行。
上述代码可以改写如下：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (UnsupportedEncodingException e) {
        System.out.println("Bad encoding");
    } catch (IOException e) {
        System.out.println("IO error");
    } finally {
        System.out.println("END");
    }
}

注意finally有几个特点：

finally语句不是必须的，可写可不写；
finally总是最后执行。
如果没有发生异常，就正常执行try { ... }语句块，然后执行finally。
如果发生了异常，就中断执行try { ... }语句块，然后跳转执行匹配的catch语句块，最后执行finally。

可见，finally是用来保证一些代码必须执行的。

某些情况下，可以没有catch，只使用try ... finally结构。例如：

void process(String file) throws IOException {
    try {
        ...
    } finally {
        System.out.println("END");
    }
}
因为方法声明了可能抛出的异常，所以可以不写catch。

## 捕获多种异常

如果某些异常的处理逻辑相同，但是异常本身不存在继承关系，那么就得编写多条catch子句：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException e) {
        System.out.println("Bad input");
    } catch (NumberFormatException e) {
        System.out.println("Bad input");
    } catch (Exception e) {
        System.out.println("Unknown error");
    }
}

因为处理IOException和NumberFormatException的代码是相同的，所以我们可以把它两用|合并到一起：

public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException | NumberFormatException e) {
        // IOException或NumberFormatException
        System.out.println("Bad input");
    } catch (Exception e) {
        System.out.println("Unknown error");
    }
}





























































































































































