## xml文件读取

### 什么是xml文件？
xml文件的表示是一种以".xml"为扩展名的文件。
其存储结构为树形结构：
![enter description here](./images/1557912085241.png)
- 节点名称区分大小写
- xml文件开头要加上版本信息和编码方式< ?xml version="1.0" encoding="UTF-8"? >

### 应用Dom方式解析Xml文件
首先我们要明确，解析xml文件的目的是获取xml文件中的节点名、节点值、属性名及属性值

#### xml中节点的概念
先真正了解xml中节点的定义
==根据Dom，xml文档中的每个成分都是一个节点==
Dom是这样规定的：
- 整个文档是一个文档节点
- 每个xml标签是一个元素节点
- 包含在xml元素中的文本是文本节点
- 每一个xml属性是一个属性节点
- 注释属于注释节点



#### Dom
Dom就是一个对象化的xml数据接口，一个与语言无关，与平台无关的标准接口规范。
文档代表的是数据，而Dom代表了如何去处理这些数据。

#### Dom树
一个XML文档及其所对应的DOM树如下：

``` xml
<?xml version="1.0" encoding="utf-8"?>

<bookstore>
    <book category="children">
          <title lang="en">Harry Potter</title> 
          <author>J K. Rowling</author> 
          <year>2005</year> 
          <price>29.99</price> 
    </book>

    <book category="cooking">
          <title lang="en">Everyday Italian</title> 
          <author>Giada De Laurentiis</author> 
          <year>2005</year> 
          <price>30.00</price> 
    </book>

    <book category="web">
          <title lang="en">Learning XML</title> 
          <author>Erik T. Ray</author> 
          <year>2003</year> 
          <price>39.95</price> 
    </book>

    <book category="web">
          <title lang="en">XQuery Kick Start</title> 
          <author>James McGovern</author> 
          <author>Per Bothner</author> 
          <author>Kurt Cagle</author> 
          <author>James Linn</author> 
         <author>Vaidyanathan Nagarajan</author> 
          <year>2003</year> 
          <price>49.99</price> 
    </book>

</bookstore>
```
![enter description here](./images/1557913320345.png)

==要严格区分根节点和根元素节点的区别==
-  根节点代表整个文档,是我们解析xml文档的入口，通过它获取到Document对象
-  根元素节点代表xml文档的根元素

#### Dom的四个基本接口
==在DOM接口规范中，有四个基本的接口：Document, Node, NodeList, NamedNodeMap==

- Document：Document接口是对文档进行操作的入口，是从Node接口继承过来的
- Node：在Dom树中,Node接口代表树中的一个节点
- NodeList：NodeList接口是一个节点的集合，它包含了某个节点中的所有子节点
- NamedNodeMap：NamedNodeMap接口也是一个节点的集合，通过该接口，可以建立节点名和节点之间的一一映射关系，从而利用节点名可以直接访问特定的节点，这个接口主要用在属性节点的表示上




#### 具体操作
xml文件：

``` xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<bookstore>
  <book category="cooking">
    <title lang="en">Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
    <price>30.00</price>
  </book>
  <book category="children">
    <title lang="en">Harry Potter</title>
    <author>J K. Rowling</author>
    <year>2005</year>
    <price>29.99</price>
  </book>
  <book category="web">
    <title lang="en">XQuery Kick Start</title>
    <author>James McGovern</author>
    <year>2003</year>
    <price>49.99</price>
  </book>
  <book category="web" cover="paperback">
    <title lang="en">Learning XML</title>
    <author>Erik T. Ray</author>
    <year>2003</year>
    <price>39.95</price>
  </book>
</bookstore>
```
Java操作：

``` java
/**
 * Dom操作获取xml中内容
 */
public class GetNodeFromXml   {


    public static void parseXmlByDomByName (File file){
        DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
        try {
            DocumentBuilder builder = documentBuilderFactory.newDocumentBuilder();
            Document document = builder.parse(file);
            document.getDocumentElement().normalize();
            //获取根元素
            System.out.println("Root element: "+document.getDocumentElement().getNodeName());
            NodeList nodeList = document.getElementsByTagName("book");
            for(int i = 0;i < nodeList.getLength();i++){
                Node node = nodeList.item(i);
                Element element = (Element)node;
                System.out.println("-----------------------");
                if(node.getNodeType() == Element.ELEMENT_NODE){
                    System.out.println("book category: " + element.getAttribute("category"));
                    System.out.println("title name: " + element.getElementsByTagName("title").item(0).getTextContent());
                    System.out.println("author name: " + element.getElementsByTagName("author").item(0).getTextContent());
                    System.out.println("year: " + element.getElementsByTagName("year").item(0).getTextContent());
                    System.out.println("title name: " + element.getElementsByTagName("price").item(0).getTextContent());
                    System.out.println("-----------------------");
                }
            }
        } catch (Exception e) {
        }
    }


    public static void parseXmlByDom(File file){
        try{
            DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
            DocumentBuilder builder = documentBuilderFactory.newDocumentBuilder();
            Document document = builder.parse(file);
            document.getDocumentElement().normalize();
            //获取根元素
            System.out.println("Root Element: "+document.getDocumentElement().getNodeName());
            if(document.hasChildNodes()){
                printNode(document.getChildNodes());
            }
        }catch(Exception e){

        }
    }

    public static void printNode(NodeList nodeList){
        System.out.println("---------------------------");
        for(int i = 0;i < nodeList.getLength();i++){
            Node node = (Node)nodeList.item(i);
            if(node.getNodeType() == Node.ELEMENT_NODE){
                System.out.println("Node Name: "+node.getNodeName());
                System.out.println("Node Value:"+node.getTextContent());
                //NameNodeMap更多应用于属性
                if(node.hasAttributes()){
                    NamedNodeMap nodeMap = node.getAttributes();
                    for(int j = 0;i < nodeMap.getLength();i++){
                        Node nodeNew = nodeMap.item(j);
                        System.out.println("Attribute Name"+nodeNew.getNodeName());
                        System.out.println("Attribute Value"+nodeNew.getTextContent());
                    }
                }
            }
            if(node.hasChildNodes()){
                printNode(node.getChildNodes());
            }
        }
    }


    public static void main(String[] args) {
        File xmlFile = new File("src/main/java/resources/book.xml");
        parseXmlByDom(xmlFile);
    }
}
```


## xml文件的约束
Xml常用约束技术为DTD和Schema

- [Xml文件约束之Schema](https://www.jianshu.com/p/badce73c00bc)
- [Xml文件约束之DTD](https://blog.csdn.net/woshihaiyong168/article/details/52460101)

## 关于xmlns
[关于xml文件 xsi:schemaLocation](https://www.jianshu.com/p/7f4cbcd9f09f)

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xsi:schemaLocation="http://www.springframework.org/schema/beans  
  http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
  http://www.springframework.org/schema/context  
  http://www.springframework.org/schema/context/spring-context-3.1.xsd  
  http://www.springframework.org/schema/mvc  
  http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

  <context:component-scan base-package="com.pikaq"/>
  <bean id="xxx" class="xxx.xxx.xxx.Xxx">
  <property name="xxx" value="xxxx"/>
</bean>
</beans>
```

对于xmlns可以这样理解：
xml文件的约束太多，不同约束中的标签可能重复，比如都可能出现< bean >< bean/ >这个标签。因此我们通过xmlns定义命名空间来区分不同标签，比如< mvc:bean >和< bean >
而xsi:schemaLocation是确定xsi这个命名空间的模板来源。