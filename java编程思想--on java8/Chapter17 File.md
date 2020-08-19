PS：Java7开始引入了java.nio.file，再配合Java8引入的Stream与文件结合使得文件操作编程变得更加优雅。
[toc]

# 文件或目录路径
## 1. Path对象
Java.nio.file.Paths类中包含一个重载方法static get()，该方法接收一系列String或者URI作为参数，返回一个Path对象；
如果参数是:"C:","test","a.txt",那么返回的是绝对路径；
如果参数是:"A.java",则以代码当前路径作为基本路径,在这个基本路径下去找A.jaa
```
Paths.get(String…param)

Paths.get(URI url)
```
可以获得一个Path对象，用来进行文件或者目录的操作，并且可以借助Files类的一些方法进行文件属性的分析。

```
@Test
    public void test() throws IOException {
        System.out.println(System.getProperty("os.name"));
        Path path = Paths.get("F:", "Speed.log");
        System.out.println("toString：" + path);
        System.out.println("Exists: " + Files.exists(path));
        System.out.println("RegularFile: " + Files.isRegularFile(path));
        System.out.println("Readable: " + Files.isReadable(path));
        System.out.println("Hidden: " + Files.isHidden(path));
        System.out.println("Directory: " + Files.isDirectory(path));
        System.out.println("Absolute: " + path.isAbsolute());
        System.out.println("Parent Path: " + path.getParent());
        System.out.println("FileName: " + path.getFileName());
        System.out.println("Root: " + path.getRoot());
```
输出：
```
Windows 10
toString：F:\Speed.log
Exists: true
RegularFile: true
Readable: true
Hidden: false
Directory: false
Absolute: true
Parent Path: F:\
FileName: Speed.log
Root: F:\
```

## 2.选取路径部分片段
Path对象非常容易生成路径中的某一部分。
1. Path对象的getNameCount()能够获取**除了根路径外**的路径数量；        
2. Path对象的getName(index i)可以获取到分割后的每一级路径的名称；
3. Path的endWith(String str)匹配的是整个路径部分，是不包含文件路径的后缀名的;
4. Path的startWith(String str)需要匹配Path.getRoot()才会返回true。
```
@Test
    public void test1() {
        Path path = Paths.get("F:", "迅雷下载", "寄生虫", "寄生虫", "Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4");
        //String path = "F:\迅雷下载\寄生虫\寄生虫\Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4"
        System.out.println(path.getNameCount());
        for (Path path1 : path) {
            System.out.println("Current: " + path1);
        }
        //true
        System.out.println(path.startsWith(path.getRoot()));
        //false
        System.out.println(path.startsWith("\\迅雷下载"));
        //false
        System.out.println(path.endsWith(".map"));
        //true
        System.out.println(path.endsWith("F:\\迅雷下载\\寄生虫\\寄生虫\\Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4"));

    }
```
```
4
Current: 迅雷下载
Current: 寄生虫
Current: 寄生虫
Current: Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4
true
false
true
true
```
## 3.路径分析
Files工具类包含一系列完整的方法用于获得Path相关的信息。
```
@Test
    public void test2() throws IOException {
        //返回以相对地址为基础的路径，不判断文件是否存在
        Path path = Paths.get("pom.xml").toAbsolutePath();
        System.out.println(path);
        System.out.println("文件是否存在: " + Files.exists(path));
        System.out.println("是否是目录: " + Files.isDirectory(path));
        System.out.println("是否是可执行文件: " + Files.isExecutable(path));
        System.out.println("是否可读: " + Files.isReadable(path));
        System.out.println("判断是否是一个文件: " + Files.isRegularFile(path));
        System.out.println("是否可写: " + Files.isWritable(path));
        System.out.println("文件是否不存在: " + Files.notExists(path));
        System.out.println("文件是否隐藏: " + Files.isHidden(path));
        System.out.println("文件大小: " + Files.size(path));
        System.out.println("文件存储在SSD还是HDD: " + Files.getFileStore(path));
        System.out.println("文件修改时间：" + Files.getLastModifiedTime(path));
        System.out.println("文件拥有者： "  + Files.getOwner(path));
        System.out.println("文件类型: " + Files.probeContentType(path));
    }
```
```
输出：
D:\ideaworkspace\webchang2\pom.xml
文件是否存在: true
是否是目录: false
是否是可执行文件: true
是否可读: true
判断是否是一个文件: true
是否可写: true
文件是否不存在: false
文件是否隐藏: false
文件大小: 3254
文件存储在SSD还是HDD: DATA (D:)
文件修改时间：2020-07-27T05:28:27.150795Z
文件拥有者： MRDRAGON\mrdra (User)
文件类型: text/xml
```
## 4.Paths的增减删改
1. 使用relativize()移除Path的根路径;
2. 使用resolve()添加Path的尾路径(不一定是存在的路径)。
```
@Test
    public void test3() {
        //当前路径是D:\ideaworkspace\webchang2
        //所以get(..)就是获取其上一级路径
        Path path = Paths.get("").toAbsolutePath().normalize();
        //输出:D:\ideaspace
        System.out.println(path);
        //输出:D:\
        System.out.println(Paths.get("..", "..").toAbsolutePath().normalize());
        //===============
        Path base = Paths.get("F:", "迅雷下载");
        Path fullPath = Paths.get("F:", "迅雷下载", "寄生虫", "寄生虫", "Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4");
        //relative是从fullPath中将base部分截取掉
        System.out.println(base.relativize(fullPath));
        //如果参数不是从根路径开始,那么base会完全拼接参数形成一个全新的路径
        System.out.println(base.resolve(Paths.get("迅雷下载", "123")));
        //PS:如果base和fullPath从根路径下是重合的了,那么只会添加不重合的部分
        System.out.println(base.resolve(fullPath));
    }
```
```
输出：
D:\ideaworkspace\webchang2
D:\
寄生虫\寄生虫\Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4
F:\迅雷下载\迅雷下载\123
F:\迅雷下载\迅雷下载\寄生虫\寄生虫\Parasite.寄生虫.2019.中文字幕.BDrip.1080P-BR.mp4
```

# 目录
Files.walkFileTree()可以用来遍历每个子目录和文件,SimpleFileVisitor提供了Visitor设计模式提供的四种方法的默认实现：
**Files.walk(Path path)可以获取指定path下的所有目录结构(文件和目录)**
```
1.  **preVisitDirectory()**：在访问目录中条目之前在目录上运行。 
2.  **visitFile()**：运行目录中的每一个文件。  
3.  **visitFileFailed()**：调用无法访问的文件。   
4.  **postVisitDirectory()**：在访问目录中条目之后在目录上运行，包括所有的子目录。
```
我们可以实现SimpleFileVisotor重写其中必要的方法来实现自己需求。
```
@Test
    public void test4() throws IOException {
        Path path = Paths.get("D:\\jdk1.8");
        Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                if (file.getFileName().toString().contains(".jar")) {
                    System.out.println(file);
                }
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult preVisitDirectory(Path path, BasicFileAttributes attributes) {
                System.out.println(path);
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFileFailed(Path path, IOException exc) {
                System.out.println(path);
                System.out.println(exc.getMessage());
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult postVisitDirectory(Path path, IOException exc) {
                System.out.println(path);
                return FileVisitResult.CONTINUE;
            }
        });
    }
```
我们可以利用SimpleFileVisitor来递归删除文件和目录：
```
// onjava/RmDir.java
package onjava;

import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.io.IOException;

public class RmDir {
    public static void rmdir(Path dir) throws IOException {
        Files.walkFileTree(dir, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                Files.delete(file);
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
                Files.delete(dir);
                return FileVisitResult.CONTINUE;
            }
        });
    }
}
```

# 文件系统
1. 可以使用静态的FileSystems工具类获取"默认"的文件系统；
2. 也可以在Path对象上调用getFileSystem()来获取创建该Path的文件系统；
3. 可以获得给定URI的文件系统；
4. 也可以构建对于支持它的新的文件系统；
5. 一个FileSystem对象也能成为WatchService和PathMatcher对象(详细见下面两个章节)。
```
@Test
    public void test5() {
        System.out.println(System.getProperty("os.name"));
        FileSystem fileSystem = FileSystems.getDefault();
        //获取逻辑磁盘信息
        for (FileStore fileStore : fileSystem.getFileStores()) {
            System.out.println("File Store :" + fileStore);
        }
        //获取根目录
        for (Path path : fileSystem.getRootDirectories()) {
            System.out.println("Root Directory :" + path);
        }
        //获取文件路径的分隔符
        System.out.println(fileSystem.getSeparator());
    }
```
```
输出：
Windows 10
File Store :OS (C:)
File Store :DATA (D:)
File Store :新加卷 (E:)
File Store :新加卷 (F:)
File Store :新加卷 (G:)
Root Directory :C:\
Root Directory :D:\
Root Directory :E:\
Root Directory :F:\
Root Directory :G:\
\
[owner, dos, acl, basic, user]
```

# 路径监听--watchService
1. 可以使用FileSystems.getDefault().newWatchService()来获得一个监视器,然后需要把该监视器注册到给定的路径上:
```
path.register(watcher, ENTRY_DELETE)
```
2. register()方法的第二个参数可选3种:ENTRY_CREATE,ENTRY_DELETE,ENTRY_MODIFY;
3. 需要注意的是监视器只会监视当前给定的目录,而不会递归去监视其下子目录中的内容，如果需要监视其下子目录中的内容,同样需要给子目录注册一个watchService();
4. 每一个watchService是独立开启一个线程去运转的；
5. 监视器会根据注册到的路径和给定的参数(ENTRY_CREATE,ENTRY_DELETE,ENTRY_MODIFY)进行目录下内容的监控,当我们在其他地方执行了文件的删除，修改，创建等，调用监视器的take()方法,通过拿到WatchKey,调用WatchKey.pollEvents()方法,获的WatchEvent，通过WatchEvent可以获取到刚执行的context()文件,count()个数，kind参数类型。
```
static void watchDir(Path dir) {
        try {
            WatchService watcher =
            FileSystems.getDefault().newWatchService();
            dir.register(watcher, ENTRY_DELETE);
            Executors.newSingleThreadExecutor().submit(() -> {
                try {
                    WatchKey key = watcher.take();
                    for(WatchEvent evt : key.pollEvents()) {
                        System.out.println(
                        "evt.context(): " + evt.context() +
                        "\nevt.count(): " + evt.count() +
                        "\nevt.kind(): " + evt.kind());
                        System.exit(0);
                    }
                } catch(InterruptedException e) {
                    return;
                }
            });
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) throws Exception {
        Directories.refreshTestDir();
        Directories.populateTestDir();
        Files.walk(Paths.get("test"))
            .filter(Files::isDirectory)
            .forEach(TreeWatcher::watchDir);  //给每一个子目录都注册一个watchService
    }
```

# 文件查找--PathMathcer
1. 上面我们为了匹配文件，通常是在Path上调用toString(),但是FileSystem对象上如果调用getPathMatcher()获得一个PathMatcher;
2. PatchMatcher可以支持glob和regex两种模式
```
public static void main(String[] args) throws Exception {
        Path test = Paths.get("test");
        Directories.refreshTestDir();
        Directories.populateTestDir();
        // Creating a *directory*, not a file:
        Files.createDirectory(test.resolve("dir.tmp"));

        PathMatcher matcher = FileSystems.getDefault()
          .getPathMatcher("glob:**/*.{tmp,txt}");
        Files.walk(test)
          .filter(matcher::matches)
          .forEach(System.out::println);
        System.out.println("***************");

        PathMatcher matcher2 = FileSystems.getDefault()
          .getPathMatcher("glob:*.tmp");
        Files.walk(test)
          .map(Path::getFileName)
          .filter(matcher2::matches)
          .forEach(System.out::println);
        System.out.println("***************");

        Files.walk(test) // Only look for files
          .filter(Files::isRegularFile)
          .map(Path::getFileName)
          .filter(matcher2::matches)
          .forEach(System.out::println);
    }
```
```
输出：
test\bag\foo\bar\baz\5208762845883213974.tmp
test\bag\foo\bar\baz\File.txt
test\bar\baz\bag\foo\7918367201207778677.tmp
test\bar\baz\bag\foo\File.txt
test\baz\bag\foo\bar\8016595521026696632.tmp
test\baz\bag\foo\bar\File.txt
test\dir.tmp
test\foo\bar\baz\bag\5832319279813617280.tmp
test\foo\bar\baz\bag\File.txt
***************
5208762845883213974.tmp
7918367201207778677.tmp
8016595521026696632.tmp
dir.tmp
5832319279813617280.tmp
***************
5208762845883213974.tmp
7918367201207778677.tmp
8016595521026696632.tmp
5832319279813617280.tmp
```
在 matcher 中，glob 表达式开头的 **/ 表示“当前目录及所有子目录”，这在当你不仅仅要匹配当前目录下特定结尾的 Path 时非常有用。单 * 表示“任何东西”，然后是一个点，然后大括号表示一系列的可能性---我们正在寻找以 .tmp 或 .txt 结尾的东西。您可以在 getPathMatcher() 文档中找到更多详细信息。

# 文件读写
1. 如果一个文件很小,可以使用Files.readAllLines(),返回一个List<String>,通常使用如下语句：**Files.readAllLines(Paths.get("C:", "a.txt")).stream()
2. Files.write()可以写入一个byte数组和Iterable对象，如下：
```
byte[] bytes = new byte[1024];
Files.write(Paths.get("bytes.dat"), bytes);
====
List<String> lines = Files.readAllLines(
          Paths.get("../streams/Cheese.dat"));
        Files.write(Paths.get("Cheese.txt"), lines);
```
3. 如果不想readAllLines(),可以调用Files.lines()获取一个stream,从而可以skip或其他操作。