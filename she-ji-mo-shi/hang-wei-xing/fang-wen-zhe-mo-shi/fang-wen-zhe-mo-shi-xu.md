# 访问者模式续

在访问者模式中，访问者必须要能够访问到对象结构中的每个对象，因为访问者要为每个对象添加功能，为此特别在模式中定义出一个ObjectStructure来，然后由ObjectStructure来负责遍历访问一系列对象中的每个对象。  
1：在ObjectStructure迭代所有的元素时，又分成两种情况。  
（1）一种是元素的对象结构是通过集合来组织的，那么直接在ObjectStructure中对集合进行迭代，对每一个元素调用accept就好了。如同前面示例所采用的方式。  
（2）另一种情况是元素的对象结构是通过组合模式来组织的，通常可以构成对象树，这种情况一般就不需要在ObjectStructure中迭代了，而通常的做法是在组合对象的accept方法里面，递归遍历它的子元素，然后调用子元素的accept方法，如同前面示例中Composite的实现，在accept方法里面进行递归调用子对象的操作。  
2：不需要ObjectStructure的时候  
在实际开发中，有一种典型的情况可以不需要ObjectStructure对象，那就是只有一个被访问对象的时候。只有一个被访问对象，当然就不需要使用ObjectStructure来组合和迭代了，只要调用这个对象就好了。  
事实上还有一种情况也可以不使用ObjectStructure，比如上面访问的组合对象结构，从客户端的角度看，他访问的其实就是一个对象，因此可以把ObjectStructure去掉，然后直接从客户端调用元素的accept方法。

3：有些时候，遍历元素的方法也可以放到访问者当中去，当然也是需要递归遍历它的子元素的。出现这种情况的主要原因是：想在访问者中实现特别复杂的遍历，访问者的实现依赖于对象结构的操作结果。

**访问者模式的本质是：预留通路，回调实现**

**何时选用访问者模式**

1：如果想对一个对象结构，实施一些依赖于对象结构中的具体类的操作，可以使用访问者模式

2：如果想对一个对象结构中的各个元素，进行很多不同的而且不相关的操作，为了避免这些操作使得类变得杂乱，可以使用访问者模式，把这些操作分散到不同的访问者对象中去，每个访问者对象实现同一类功能。

3：如果对象结构很少变动，但是需要经常给对象结构中的元素对象定义新的操作，可以使用访问者模式

内容来自：《研磨设计模式-第25章 访问者模式》

