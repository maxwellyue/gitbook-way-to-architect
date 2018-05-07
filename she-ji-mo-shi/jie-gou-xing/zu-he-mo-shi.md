# 组合模式

---

组合模式：将对象组合成**树形结构**以表示"部分-整体"的层次结构，使得客户端对单个对象和组合对象的使用具有一致性。

组合模式的关键是抽象出一个类，这个类既可以表示叶子节点，也可以表示组合对象。

理解组合模式的两个关键词：抽象类、树形结构

**组合模式的中的角色**

* Component（抽象构件）：接口或抽象类，为Leaf和Composite声明共有行为的方法以及部分方法实现：如增加子构件、删除子构件、获取子构件等。

* Leaf（叶子构件）：叶子节点对象，没有子节点。

* Composite（容器构件）：容器节点对象，包含子节点（子节点可以是Leaf，也可以是Composite），提供一个集合用于存储子节点。

**组合模式结构代码**

Component

```java
public abstract class Component {  
    public abstract void add(Component c); //增加成员  
    public abstract void remove(Component c); //删除成员  
    public abstract Component getChild(int i); //获取成员  
    public abstract void operation();  //Leaf和Composite共有的业务方法  
}
```

Leaf：由于Leaf没有add、remove、getChild等操作，所以在实现中可以抛出UnSupportOperationException或直接为空。

```java
public class Leaf extends Component {  

    public void add(Component c) {   
        //异常处理或错误提示   
    }     

    public void remove(Component c) {   
        //异常处理或错误提示   
    }  

    public Component getChild(int i) {   
        //异常处理或错误提示  
        return null;   
    }

    public void operation() {  
        //叶子构件具体业务方法的实现  
    }   
}
```

Composite

```java
public class Composite extends Component {  

    private ArrayList<Component> list = new ArrayList<Component>();  

    public void add(Component c) {  
        list.add(c);  
    }  

    public void remove(Component c) {  
        list.remove(c);  
    }  

    public Component getChild(int i) {  
        return (Component)list.get(i);  
    }  

    public void operation() {  
        //容器构件具体业务方法的实现  
        //递归调用成员构件的业务方法  
        for(Object obj:list) {  
            ((Component)obj).operation();  
        }  
    }     
}
```

现实中，电脑上的目录，公司中的部门，软件中的菜单，都是这种树形结构的形式，都可以考虑使用组合模式来简化客户端的操作。

举个例子：开发一个杀毒\(AntiVirus\)软件，该软件既可以对某个文件夹\(Folder\)杀毒，也可以对某个指定的文件\(File\)进行杀毒。该杀毒软件还可以根据各类文件的特点，为不同类型的文件提供不同的杀毒方式，例如图像文件\(ImageFile\)和文本文件\(TextFile\)的杀毒方式就有所差异。用户在使用的时候，无论是文件夹还是文件，无论是何种类型的文件，都可以通过同一种操作完成杀毒这个工作。

Component：AbstractFile

```java
public abstract class AbstractFile {  
    public abstract void add(AbstractFile file);  
    public abstract void remove(AbstractFile file);  
    public abstract AbstractFile getChild(int i);  
    public abstract void killVirus();  
}
```

Composite：Folder

```java
public class Folder extends AbstractFile {  

    //定义集合fileList，用于存储AbstractFile类型的成员  
    private ArrayList<AbstractFile> fileList=new ArrayList<AbstractFile>();  

    private String name;  

    public Folder(String name) {  
        this.name = name;  
    }  

    public void add(AbstractFile file) {  
       fileList.add(file);    
    }  

    public void remove(AbstractFile file) {  
        fileList.remove(file);  
    }  

    public AbstractFile getChild(int i) {  
        return (AbstractFile)fileList.get(i);  
    }  

    public void killVirus() {  
        //模拟杀毒
        System.out.println("****对文件夹'" + name + "'进行杀毒"); 
        //递归调用成员构件的killVirus()方法  
        for(Object obj : fileList) {  
            ((AbstractFile)obj).killVirus();  
        }  
    }  
}
```

Leaf：有2个，分别是`ImageFile ，TextFile`

```java
//图像文件类
class ImageFile extends AbstractFile {  
    private String name;  

    public ImageFile(String name) {  
        this.name = name;  
    }  

    public void add(AbstractFile file) {  
       System.out.println("对不起，不支持该方法！");  
    }  

    public void remove(AbstractFile file) {  
        System.out.println("对不起，不支持该方法！");  
    }  

    public AbstractFile getChild(int i) {  
        System.out.println("对不起，不支持该方法！");  
        return null;  
    }  

    public void killVirus() {  
        //模拟杀毒  
        System.out.println("----对图像文件'" + name + "'进行杀毒");  
    }  
}  

//文本文件类
class TextFile extends AbstractFile {  
    private String name;  

    public TextFile(String name) {  
        this.name = name;  
    }  

    public void add(AbstractFile file) {  
       System.out.println("对不起，不支持该方法！");  
    }  

    public void remove(AbstractFile file) {  
        System.out.println("对不起，不支持该方法！");  
    }  

    public AbstractFile getChild(int i) {  
        System.out.println("对不起，不支持该方法！");  
        return null;  
    }  

    public void killVirus() {  
        //模拟杀毒  
        System.out.println("----对文本文件'" + name + "'进行杀毒");  
    }  
} 
```

客户端使用

```java
public class Client {  
    public static void main(String args[]) {  
        
        AbstractFile file1,file2,file3,file4,file5,folder1,folder2,folder3,folder4;  

        folder = new Folder("my");  
        imageFolder = new Folder("img");  
        textFolder = new Folder("text");  
        
        folder.add(imageFolder);  
        folder.add(textFolder); 
               
        iamgeFolder.add(new ImageFile("dog.jpg"));  
        imageFolder.add(new ImageFile("cat.png"));  
        textFolder.add(new TextFile("way-to-architect.txt"));           

        //用户需求1：对目录my进行杀毒
        folder.killVirus();
        
        //用户需求2：只对图片杀毒
        imageFolder.killVirus();
        
        //用户需求3：只对way-to-architect.txt文件杀毒
        new TextFile("way-to-architect.txt").killVirus();
    }  
}
```

可以看出，无论用户操作的是树形结构的叶子节点还是容器节点，所执行的操作都是一样的（都是执行`killVirus()`方法）。即用户无须关心节点的层次结构，可以对所选节点进行统一处理。

注意到，上面的示例代码中，叶子节点







