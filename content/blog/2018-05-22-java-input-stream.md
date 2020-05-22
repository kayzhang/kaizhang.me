---
title: What exactly is System.in?
subtitle: Java input stream
date: '2018-05-22'
slug: java-input-stream
categories:
  - Java
tags:
  - Java
---

本文分析了 `Java` 标准输入 System.in，文中在涉及到 System.out 与 System.err 时将相关信息及代码也进行了描述，但并未深入探究，其与 System.in 存在一些差别，留待后续研究。

# java.io.FileDescriptor
> [public final class FileDescriptor extends Object](https://docs.oracle.com/javase/10/docs/api/java/io/FileDescriptor.html)
>
> Instances of the file descriptor class serve as an opaque handle to the underlying machine-specific structure representing an open file, an open socket, or another source or sink of bytes. The main practical use for a file descriptor is to create a FileInputStream or FileOutputStream to contain it.
>
> Applications should not create their own file descriptors.

FileDescriptor 是文件描述符，可以理解为系统不同输入输出装置的句柄，其有三个非常重要的 fields，代表了三个不同的输入输出句柄，即标准输入流句柄、标准输出流句柄和标准错误流句柄。

| Modifier and Type     |      Field    | Description |
| :-: | :-: | :-: |
| static FileDescriptor |       in      | A handle to the standard input stream. |
| static FileDescriptor |       out     | A handle to the standard output stream. |
| static FileDescriptor |       err     | A handle to the standard err stream. |

# java.io.FileInputStream
> [public class FileInputStream extends InputStream](https://docs.oracle.com/javase/10/docs/api/java/io/FileInputStream.html)
>
> A FileInputStream obtains input bytes from a file in a file system. What files are available depends on the host environment.
>
> FileInputStream is meant for reading streams of raw bytes such as image data. For reading streams of characters, consider using FileReader.

FileInputStream 是文件输入流，继承 InputStream，用来从“文件”中读取 raw bytes。其中一个构造器为 `FileInputStream​(FileDescriptor fdObj)`：

> public FileInputStream​(FileDescriptor fdObj)
> 
> Creates a FileInputStream by using the file descriptor fdObj, which represents an existing connection to an actual file in the file system.

FileInputStream 重写了抽象类 InputStream 中的抽象方法 read()：

> public int read() throws IOException
> 
> Reads a byte of data from this input stream. This method blocks if no input is yet available.

# 通过 FileDescriptor.in 构造读取标准输入的 FileInputStream

通过上述构造器，可将 FileDescriptor.in 即标准输入句柄作为参数传递给 FileInputStream(FileDescriptor fdObj) 构造器，创建 FileInputStream 实例 fdIn：`FileInputStream fdIn = new FileInputStream(FileDescriptor.in);`，由此便可调用 FileInputStream 中的方法读取标准输入。

```java
import java.io.IOException;
import java.io.FileDescriptor;
import java.io.FileInputStream;

public class myFileInputStream {
    public static void main (String[] args) throws IOException {
        FileInputStream fdIn = new FileInputStream(FileDescriptor.in);
        int i = fdIn.read();
        System.out.println(i);
    }
}
```

由于 fdIn.read() 每次从输入读取一个字节，余下的字节相当于直接输入到命令行，因此存在以下情形：

1. 直接按回车，实际输入为 "\n"，read() 读取 '\n'，输出其 ASCII 码：10，结束；
2. 输入 1 然后按回车，实际输入为 "1\n"，read() 读取 '1'，输出：49，然后将余下的 "\n" 返回给命令行，相当于继续在命令行按一个回车，结束；
3. 输入 123 然后按回车，实际输入为 "123\n"，read() 读取 '1'，输出：49，然后将余下的 "23\n" 返回给命令行，相当于继续在命令行输入 23 然后按一个回车，结束。

对应的输出为：

```zsh
➜  algs java myFileInputStream

10
➜  algs java myFileInputStream
1
49
➜  algs
➜  algs java myFileInputStream
123
49
➜  algs 23
zsh: command not found: 23
➜  algs
```

# java.io.BufferedInputStream
> [public class BufferedInputStream extends FilterInputStream](https://docs.oracle.com/javase/10/docs/api/java/io/BufferedInputStream.html)
>
> A BufferedInputStream adds functionality to another input stream-namely, the ability to buffer the input and to support the mark and reset methods. When the BufferedInputStream is created, an internal buffer array is created. As bytes from the stream are read or skipped, the internal buffer is refilled as necessary from the contained input stream, many bytes at a time. The mark operation remembers a point in the input stream and the reset operation causes all the bytes read since the most recent mark operation to be reread before new bytes are taken from the contained input stream.

BufferedInputStream 继承 java.io.FilterInputStream，而 FilterInputStream 继承 InputStream。

BufferedInputStream 相对一般的 InputStream 增加了一个缓存区，其默认大小是 8192 字节，执行 read() 先将字节读进缓冲区，然后从缓冲区一个读取一个字节，当缓冲区数据读完时再把缓冲区填满。

其中一个构造器为：

> public BufferedInputStream(InputStream in)
> 
> Creates a BufferedInputStream and saves its argument, the input stream in, for later use. An internal buffer array is created and stored in buf.
>
> Parameters:
> 
> in - the underlying input stream.

因此，可以将 FileInputStream 的实例 fdIn 作为参数构造 BufferedInputStream 实例。

```java
import java.io.IOException;
import java.io.FileDescriptor;
import java.io.FileInputStream;
import java.io.BufferedInputStream;

public class myBufferedInputStream {
    public static void main (String[] args) throws IOException {
        FileInputStream fdIn = new FileInputStream(FileDescriptor.in);
        BufferedInputStream bdIn = new BufferedInputStream(fdIn);
        int i = bdIn.read();
        System.out.println(i);
    }
}
```

由于此时，read() 一次性将键盘输入全部读取到了缓冲区（包括回车 '\n'），因此运行与 fdIn.read() 不同，不会再把未读的字节返回到终端：

```zsh
➜  algs java myBufferedInputStream

10
➜  algs java myBufferedInputStream
1
49
➜  algs java myBufferedInputStream
123
49
➜  algs
```

# BufferedInputStream 与 FileInputStream 对比
参考：[Why is using BufferedInputStream to read a file byte by byte faster than using FileInputStream?](https://stackoverflow.com/questions/18600331/why-is-using-bufferedinputstream-to-read-a-file-byte-by-byte-faster-than-using-f)

在 FileInputStream 中，read() 方法每次读取一个字节，源码为：

```java
/**
 * Reads a byte of data from this input stream. This method blocks
 * if no input is yet available.
 *
 * @return     the next byte of data, or <code>-1</code> if the end of the
 *             file is reached.
 * @exception  IOException  if an I/O error occurs.
 */
public native int read() throws IOException;
```

这是一个 native 方法，每读一个字节都要访问一次硬盘，性能比较差，运行较慢。而 BufferedInputStream 存在缓存区，由于读取内存的速度远远快于读取硬盘的速度，性能大幅度提升。

**注意：**

1. **FileInputStream 中存在 read(byte[] b) 方法，其相当于 BufferedInputStream 的 read() 方法，具体差异有待研究；**
2. **对于读取文件而言，更快的方式是 FileChannel，有待研究。**

# System.in 源码验证
System 类中 in 变量的定义为：`public final static InputStream in = null;`，InputStream 本身是一个抽象类，其 read() 方法本身也是一个抽象方法。但是可以直接调用 System.in.read() 读取标准输入。

```java
import java.io.IOException;

public class mySystemIn {
    public static void main (String[] args) throws IOException {
        int i = System.in.read();
        System.out.println(i);
    }
}
```

运行结果为：

```bash
➜  algs java ioTest      
1
49
➜  algs
```

那么 System.in 指向的实例对象到底是谁呢？

可以采用以下代码找到 System.in、System.out、System.err 指向的实例：

```java
public class SystemDotInClassFinder {
    public static void main(String[] args) {
        System.out.println(System.in.getClass().getName());
        System.out.println(System.out.getClass().getName());
        System.out.println(System.err.getClass().getName());
    }
}
```

得到的结果为：

```bash
java.io.BufferedInputStream
java.io.printStream
java.io.printStream
```

由此可知，System.in 实际上指向的是 BufferedInputStream 类的实例。

**Java 如何实现的 System.in 的初始化？**

阅读源码，可以发现 java.lang.System.java 类中与 in、out、err 初始化相关的代码如下：

```java
// The "standard" input stream.
public final static InputStream in = null;

// The "standard" output stream.
public final static PrintStream out = null;

// The "standard" error output stream.
public final static PrintStream err = null;

private static native void setIn0(InputStream in);
private static native void setOut0(PrintStream out);
private static native void setErr0(PrintStream err);

public final class System {

    /* register the natives via the static initializer.
     *
     * VM will invoke the initializeSystemClass method to complete
     * the initialization for this class separated from clinit.
     * Note that to use properties set by the VM, see the constraints
     * described in the initializeSystemClass method.
     */
    private static native void registerNatives();
    static {
        registerNatives();
    }
}

// Initialize the system class.  Called after thread initialization.
private static void initializeSystemClass() {
    FileInputStream fdIn = new FileInputStream(FileDescriptor.in);
    FileOutputStream fdOut = new FileOutputStream(FileDescriptor.out);
    FileOutputStream fdErr = new FileOutputStream(FileDescriptor.err);
    setIn0(new BufferedInputStream(fdIn));
    setOut0(new PrintStream(new BufferedOutputStream(fdOut, 128), true));
    setErr0(new PrintStream(new BufferedOutputStream(fdErr, 128), true));
}
```

注意到，native 方法 registerNatives() 的注释中提到：*VM will invoke the initializeSystemClass method to complete the initialization for this class separated from clinit*，而在 initializeSystemClass() 方法中调用了 native 方法 setIn0、setOut0、setErr0 对 in、out、err 进行了初始化。

因此 System.in 的初始化过程具体为：

1. 首先将 FileDescriptor.in 即标准输入句柄作为参数传递给 FileInputStream(FileDescriptor fdObj) 构造器，创建 FileInputStream 实例 fdIn：`FileInputStream fdIn = new FileInputStream(FileDescriptor.in);`
2. 然后将 fdIn 传递给 BufferedInputStream(InputStream in) 构造器，创建 BufferedInputStream 实例；
3. 然后再把创建好的 BufferedInputStream 实例赋给 `System.in：setIn0(new BufferedInputStream(fdIn));` (具体机制不清楚，因为 setIn0 为 native 方法)。

**更改 System.in、System.out、System.err**

System 类中给出了更改这三个变量的方法，在更改前要首先进行安全检查，查看是否有更改的权限。

```java
// Reassigns the "standard" input stream.
public static void setIn(InputStream in) {
    checkIO();
    setIn0(in);
}

// Reassigns the "standard" output stream.
public static void setOut(PrintStream out) {
    checkIO();
    setOut0(out);
}

// Reassigns the "standard" error output stream.
public static void setErr(PrintStream err) {
    checkIO();
    setErr0(err);
}

private static void checkIO() {
    SecurityManager sm = getSecurityManager();
    if (sm != null) {
        sm.checkPermission(new RuntimePermission("setIO"));
    }
}
```
