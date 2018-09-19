# InputStream的重复读

InputStream其实是一个数据通道，只负责数据的流通，并不负责数据的处理和存储等其他工作。但是在有的场合中，我们需要重复利用InputStream的数据，比如：

* 对于文件流，我们可能需要先读取InputStream中的前一些字节来判断文件的编码方式或者内容格式等，然后再具体地使用
* 对于一个请求，我们可能需要对请求的数据流进行重复读取。

根据输入流的来源的不同，有以下方式来实现重复读取。

**主动获取流**

即我们可以对流的来源进行主动控制，或者说我们可以主动地去获取流，这种情况，可以“曲线救国”，通过再次获取输入的方式达到重复读输入流的目的。比如，我们要读取一个文件，且可以直接从该文件获取流，则可以这样来“重复读”：

```java
InputStream inputStream = new FileInputStream(path);  
//利用inputStream  
inputStream = new FileInputStream(path);  
//再次利用inputStream  
```

**被动使用流**

这种情况是指，我们无法从源来获取输入流，只能获取到一个输入流，如下所示，doSomething\(InputStream is\)是别人提供的一个接口方法：

```java
public void doSomething(InputStream is){
    //use InputStream here
}
```

这种情况其实是真正需要重复读的情况。有两种方案：一是使用缓存，而是通过mark和reset方法。

1、使用缓存

```java
public class InputStreamCacher {  
            
    /** 
     * 将InputStream中的字节保存到ByteArrayOutputStream中。 
     */  
    private ByteArrayOutputStream byteArrayOutputStream = null;  
      
    public InputStreamCacher(InputStream inputStream) {  
        if (ObjectUtils.isNull(inputStream))  
            return;  
          
        byteArrayOutputStream = new ByteArrayOutputStream();  
        byte[] buffer = new byte[1024];    
        int len;    
        try {  
            while ((len = inputStream.read(buffer)) > -1 ) {    
                byteArrayOutputStream.write(buffer, 0, len);    
            }  
            byteArrayOutputStream.flush();  
        } catch (IOException e) {  
            logger.error(e.getMessage(), e);  
        }    
    }  
      
    public InputStream getInputStream() {  
        if (ObjectUtils.isNull(byteArrayOutputStream))  
            return null;  
          
        return new ByteArrayInputStream(byteArrayOutputStream.toByteArray());  
    }  
} 

//使用
public void doSomething(InputStream is){
    InputStreamCacher  cacher = new InputStreamCacher(is);  
    InputStream stream = cacher.getInputStream();  
    //读取stream
    stream = cacher.getInputStream(); 
}

```

对于实际的请求，可以使用这样的方式来实现重复读取：

```java
public class RepeatedlyReadRequestWrapper extends HttpServletRequestWrapper {

    private Logger logger = LoggerFactory.getLogger(RepeatedlyReadRequestWrapper.class);

    private static final int BUFFER_START_POSITION = 0;

    private static final int CHAR_BUFFER_LENGTH = 1024;

    /**
     * 将input stream 缓存到body
     */
    private final String body;

    /**
     * @param request {@link HttpServletRequest} object.
     */
    public RepeatedlyReadRequestWrapper(HttpServletRequest request) {
        super(request);

        StringBuilder stringBuilder = new StringBuilder();

        InputStream inputStream = null;
        try {
            inputStream = request.getInputStream();
        } catch (IOException e) {
            logger.error("Error reading the request body…", e);
        }
        if (inputStream != null) {
            try (BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream))) {
                char[] charBuffer = new char[CHAR_BUFFER_LENGTH];
                int bytesRead;
                while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
                    stringBuilder.append(charBuffer, BUFFER_START_POSITION, bytesRead);
                }
            } catch (IOException e) {
                logger.error("Fail to read input stream", e);
            }
        } else {
            stringBuilder.append("");
        }
        body = stringBuilder.toString();
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(body.getBytes());
        return new DelegatingServletInputStream(byteArrayInputStream);
    }
}
```

这种缓存的方式，其实是将输入流对应的源数据直接以字节的形式缓存到内存中，如果数据量比较大，则占用内存较多。

2、使用mark和reset方法

```java
//InputStream是否支持mark，默认不支持
public boolean markSupported() {  
   return false;  
} 

//mark接口，该接口在InputStream中默认实现不做任何事情。 
public synchronized void mark(int readlimit) {}  

//reset接口，该接口在InputStream中实现，调用就会抛异常。 
public synchronized void reset() throws IOException {  
   throw new IOException("mark/reset not supported");  
}
```

* 调用mark方法，会记下当前调用mark方法的时刻，InputStream被读到的位置
* 调用reset方法就会回到该位置
* readlimit参数
* * 对于BufferedInputStream，readlimit表示：InputStream调用mark方法的时刻起，在读取readlimit个字节之前，标记的该位置是有效的。如果读取的字节数大于readlimit，可能标记的位置会失效。 
  * 对于ByteArrayInputStream，readlimit参数没有用，调用mark方法的时候写多少都无所谓。 

对于常见的InputStream实现类，FileInputStream不支持mark，BufferInputStream及其父类FilterInputStream支持。

**参考**

[重复读取InputStream的方法](http://zhangbo-peipei-163-com.iteye.com/blog/2022442)

[通过mark和reset方法重复利用InputStream](https://blog.csdn.net/guoyf123321/article/details/50281007)

[输入流InputStream的reset\(\)和mark\(\)方法注意事项](https://blog.csdn.net/u011494050/article/details/41891817)

  




