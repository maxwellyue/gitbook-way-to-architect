假如现在要开发一个跨平台图像浏览系统，要求该系统能够显示BMP、JPG、GIF、PNG等多种格式的文件，并且能够在Windows、Linux、Unix等多个操作系统上运行。

系统首先将各种格式的文件解析为像素矩阵\(Matrix\)，然后将像素矩阵显示在屏幕上，在不同的操作系统中可以调用不同的绘制函数来绘制像素矩阵。系统需具有较好的扩展性以支持新的文件格式和操作系统。

我们很可能会想到如下的设计方案：

我们现在对该设计方案进行分析，发现存在如下两个主要问题：

\(1\)多层继承结构，导致系统中类的个数急剧增加，具体层的类的个数 = 所支持的图像文件格式数×所支持的操作系统数。

\(2\)系统扩展麻烦，由于每一个具体类既包含图像文件格式信息，又包含操作系统信息，因此无论是增加新的图像文件格式还是增加新的操作系统，都需要增加大量的具体类，例如增加一种新的图像文件格式TIF，则需要增加3个具体类来实现该格式图像在3种不同操作系统的显示；如果增加一个新的操作系统Mac OS，为了在该操作系统下能够显示各种类型的图像，需要增加4个具体类。这将导致系统变得非常庞大，增加运行和维护开销。

我们通过分析可得知，该系统存在两个独立变化的维度：图像文件格式和操作系统。

现在，我们使用桥接模式来解决上述问题。

像素矩阵类：辅助类，各种格式的文件最终都被转化为像素矩阵，不同的操作系统提供不同的方式显示像素矩阵

```java
public class Matrix {  
    ...
}
```

抽象图像类：抽象类

```java
public abstract class Image {  

    protected ImageImp imp;  

    public void setImageImp(ImageImp imp) {  
        this.imp = imp;  
    }

    public abstract void parseFile(String fileName);  
}
```

抽象操作系统实现类：实现类接口

```java
public interface ImageImp {  
    public void doPaint(Matrix m);
}
```

具体的操作系统实现类：

```java
//Windows操作系统实现类：具体实现类  
public class WindowsImp implements ImageImp {  
    public void doPaint(Matrix m) {  
        System.out.print("在Windows操作系统中显示图像：");  
    }  
}  

//Linux操作系统实现类：具体实现类  
public class LinuxImp implements ImageImp {  
    public void doPaint(Matrix m) {  
        System.out.print("在Linux操作系统中显示图像：");  
    }  
}  

//Unix操作系统实现类：具体实现类  
pulbic class UnixImp implements ImageImp {  
    public void doPaint(Matrix m) {  
        System.out.print("在Unix操作系统中显示图像：");  
    }  
}
```

图像处理类

```java
//JPG格式图像：扩充抽象类  
class JPGImage extends Image {  
    public void parseFile(String fileName) {  
        //模拟解析JPG文件并获得一个像素矩阵对象m;  
        Matrix m = new Matrix();   
        imp.doPaint(m);  
        System.out.println(fileName + "，格式为JPG。");  
    }  
}  

//PNG格式图像：扩充抽象类  
public class PNGImage extends Image {  
    public void parseFile(String fileName) {  
        //模拟解析PNG文件并获得一个像素矩阵对象m;  
        Matrix m = new Matrix();   
        imp.doPaint(m);  
        System.out.println(fileName + "，格式为PNG。");  
    }  
}  

//BMP格式图像：扩充抽象类  
public class BMPImage extends Image {  
    public void parseFile(String fileName) {  
        //模拟解析BMP文件并获得一个像素矩阵对象m;  
        Matrix m = new Matrix();   
        imp.doPaint(m);  
        System.out.println(fileName + "，格式为BMP。");  
    }  
}  

//GIF格式图像：扩充抽象类  
pulbic class GIFImage extends Image {  
    public void parseFile(String fileName) {  
        //模拟解析GIF文件并获得一个像素矩阵对象m;  
        Matrix m = new Matrix();   
        imp.doPaint(m);  
        System.out.println(fileName + "，格式为GIF。");  
    }  
}
```

客户端

```java
publci class Client {  
    public static void main(String args[]) {  
        Image image = new GIFImage();  
        image.setImageImp(new WindowsImp());  
        image.parseFile("dog.gif");  
    }  
}
```



