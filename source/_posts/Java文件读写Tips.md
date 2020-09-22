---
title: Java文件读写Tips
date: 2020-09-21 21:11:35
categories: BackEnd
tags:
    - IO
top:
---
# 1. 字符编码问题

FileReader是以当前机器的默认字符集来读取文件的，如果希望指定字符集的话，需要直接使用InputStreamReader和FileInputStream 

因此对于字符编码问题，我们应当使用FileInputStream来读取文件流，然后使用InputStreamReader读取字符流，并且制定字符集


# 2. Files类的静态方法

+ `Files.readAllLines` 
    + 可以很方便的一行代码完成文件内容的读取
    + 但是读取超出内存大小的大文件的时候会出现OOM
        + 是因为readAllLines读取文件所有内容之后会放到一个List<String>当中返回，如果内存无法容纳这个List，就会OOM 

+ `Files.lines`
    + 返回的是Stream<String>
    + 使得我们在需要的时候可以不断读取，使用文件中的内容，而不是一次性的把所有内容都读取到内存当中，因此避免了OOM

+ 对于返回是Stream的方法要注意关闭句柄，可以通过使用try with resources 的方式来确保流的close方法可以调用释放资源


```
LongAdder longAdder = new LongAdder();
IntStream.rangeClosed(1, 1000000).forEach(i -> {
    try (Stream<String> lines = Files.lines(Paths.get("demo.txt"))) {
        lines.forEach(line -> longAdder.increment());
    } catch (IOException e) {
        e.printStackTrace();
    }
});
log.info("total : {}", longAdder.longValue());

```

# 3. 读写文件的缓冲区设置

## 3.1 为什么执行缓慢？

```
private static void perByteOperation() throws IOException {
    try (FileInputStream fileInputStream = new FileInputStream("src.txt");
         FileOutputStream fileOutputStream = new FileOutputStream("dest.txt")) {
        int i;
        while ((i = fileInputStream.read()) != -1) {
            fileOutputStream.write(i);
        }
    }
}
```

+ 上述代码运行起来功能上完全没有问题
    + 但是会非常缓慢
    + 原因是每读取一个字节，每写入一个字节就进行一次IO操作，代价太大了
    + 解决方案是可以使用缓冲区作为过渡，一次性从原文件当中读取一定数量的数据到缓冲区当中，一次性写入一定数量的数据到目标文件


```

private static void bufferOperationWith100Buffer() throws IOException {
    try (FileInputStream fileInputStream = new FileInputStream("src.txt");
         FileOutputStream fileOutputStream = new FileOutputStream("dest.txt")) {
        byte[] buffer = new byte[100];
        int len = 0;
        while ((len = fileInputStream.read(buffer)) != -1) {
            fileOutputStream.write(buffer, 0, len);
        }
    }
}

```