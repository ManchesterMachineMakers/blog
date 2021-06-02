---
layout: post
title: "Java Reflection Tutorial, Part 1"
author: Aleks Rutins
categories: [ java, tutorial ]
---

# Java Reflection Tutorial, Part 1

_First of all, this is the first time I've ever written a blog post. My apologies if the organization makes no sense._

Reflection in programming is the act of using code to write or read itself. For example, in Java, getting the name of a class or method at runtime is reflection.

In Java, most of the reflection classes and methods are in the `java.lang.reflect` package, and tools for dealing with annotations are in `java.lang.annotation`.

To get started, create a directory to hold files for this tutorial. Create a new basic Java file, and call it `Basic.java`:

```java
import java.lang.reflect.*;
import java.lang.annotation.*;

public class Basic {
    public static void main(String[] args) {
        System.out.println("Hello Java Reflection");
    }
}
```

If you don't know what that does, a comprehensive tutorial is [here](https://www.w3schools.com/java/).

Run it with `java Basic.java`. You should see the output:

```
$ java Basic.java
Hello Java Reflection
```

Great! Your Java installation works. If it doesn't, go [ask the Oracle](https://www.oracle.com/java/). Or [ask OpenJDK](http://openjdk.java.net/), if you're on Linux.

## Getting the Name of a Class

Now, let's do some reflection! Add some lines to `main`:

```java
import java.lang.reflect.*;
import java.lang.annotation.*;

public class Basic {
    public static void main(String[] args) {
        System.out.println("Hello Java Reflection");
        Class<?> cls = Basic.class;
        System.out.println("The class's name is " + cls.getName());
    }
}
```

Run it again, and you should see this:

```
$ java Basic.java
Hello Java Reflection
The class's name is Basic
```

Whoa! How'd it know that? Let's see.

First, take a look at this line:

```java
Class<?> cls = Basic.class;
```

First of all, the `?` in `Class<?>` tells the Java compiler to auto-detect the type argument. In this case, it could also have been written as `Class<Basic>`, but that would be more typing. I could also use `var`, which tells it to completely auto-detect the type:

```java
var cls = Basic.class;
```

However, `var` is not supported in JDK 7, which is what FTC uses. If you have JDK 9 or higher, you can use `var`, though.

Next, `Basic.class` tells it to get the `Class<T>` (`T` is a type argument) instance for `Basic`. `Class<T>` is core for reflection. So essential, in fact, that it is included in `java.lang`.

On to the next line:

```java
System.out.println("The class's name is " + cls.getName());
```

Everybody should be familiar with `System.out.println`. If not, I urge you once again to check out [this tutorial](https://www.w3schools.com/java). However, `cls.getName()` should be new. If not, I urge you to skip to the next heading, wait for part 2 (annotations), or seek out a more advanced tutorial.

`getName` is a method on `Class<T>` that gets the qualified name. For example, if `Basic` were in the `com.not.a.domain` package, `cls.getName()` would return `"com.not.a.domain.Basic"`. Since `Basic` is not in a package, it just returns `"Basic"`. If you need an unqualified name, without the package, use [`Class.getSimpleName`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getSimpleName--).

## Getting the Name of a Method

To access methods, we use the [`Method`](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html) type, which is in `java.lang.reflect`. Methods can be accessed using [`Class.getMethod`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethod-java.lang.String-java.lang.Class...-) or [`Class.getMethods`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethods--), which return a specific method or a list of all methods in the class, respectively.

Add a method to `Basic`:

```java
public void doSomething(String whatever) {
    System.out.println("Doing something very interesting...");
    System.out.println("Whatever is " + whatever);
}
```

Now, just to make sure it exists, create an instance in `main` and try it:

```java
Basic basic = new Basic();
basic.doSomething("whatever");
```

Run it, and the whole output should be:

```
$ java Basic.java
Hello Java Reflection
The class's name is Basic
Doing something very interesting...
Whatever is whatever
```

It works! Now, let's get a `Method` instance for that method. Remember, `cls` is `Basic.class`. The `try` block is necessary because `getMethod` can throw a `NoSuchMethodException` if the method was not found.

```java
try {
    Method doSomething = cls.getMethod("doSomething", String.class);
} catch(NoSuchMethodException e) {
    System.out.println("A whatsit happened: " + e.toString());
}
```

Now, let's look at the interesting line:

```java
Method doSomething = cls.getMethod("doSomething", String.class);
```

If you look at the description for `Class.getMethod`, you can see that the first argument is the name of the method, and any remaining arguments are parameter types, as `Class<T>` instances. In this case, since the method is called `doSomething` and it takes one `String` argument, the first argument is `"doSomething"` and the second argument is `String.class`.

Now that we have the `Method` instance, it's easy enough to get the name (inside the `try` block):

```java
System.out.println("The method's name is " + doSomething.getName());
```

Now, the output should be:

```
$ java Basic.java
Hello Java Reflection
The class's name is Basic
Doing something very interesting...
Whatever is whatever
The method's name is doSomething
```

It worked!

If something went wrong, here's the full code of `Basic.java` (organized into sections):

```java
import java.lang.reflect.*;
import java.lang.annotation.*;

public class Basic {
    public static void main(String[] args) {
        // SETUP //
        System.out.println("Hello Java Reflection");

        // GETTING THE NAME OF A CLASS //
        Class<?> cls = Basic.class;
        System.out.println("The class's name is " + cls.getName());
        
        // GETTING THE NAME OF A METHOD //
        Basic basic = new Basic();
        basic.doSomething("whatever");
        try {
            Method doSomething = cls.getMethod("doSomething", String.class);
            System.out.println("The method's name is " + doSomething.getName());
        } catch(NoSuchMethodException e) {
            System.out.println("A whatsit happened: " + e.toString());
        }
    }
    public void doSomething(String whatever) {
        System.out.println("Doing something very interesting...");
        System.out.println("Whatever is " + whatever);
    }
}
```

See you in Part 2, which will go over annotations!

## Other Notes & Exercises

1. Try putting `Basic` in a package, and see what `getName` returns. Then, try to get the name _without_ the package name.

2. If you aren't sure how to set up your IDE, I use [Visual Studio Code](https://code.visualstudio.com) with the [Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack).

3. The `java` command-line tool, as used above, can also compile files on-the-fly. If you experience problems, you can use the longer version:

   ```
   $ javac Basic.java
   $ java Basic
   ...
   ```

4. Use the [documentation for Method](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html) to figure out how to get the name of the class that declared the method.