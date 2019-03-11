DTD与XSD的区别

DTD (Document Type Definition) 即文档定义类型，是一种XML约束模式语言，是XML文件的验证机制，属于XML文件组成的一部分。DTD是一种保证XML文档格式正确的有效方法。可以通过比较XML文档和DTD文档来看文档是否符合规范，元素和标签使用是否正确。 一个DTD文档包含：元素的定义规则，元素间关系的定义规则，元素可使用的属性，可使用的实体或符号规则。

​	要使用DTD验证模式的时候要在XML文件头部声明

​	<?xml version="1.0" encoding="UTF-8"?>



~~~ xml
<!DOCTYPE beans PUblIC "-//Spring//DTD BEAN2.0//EN" "http://www.Springframework.org/dtd/Spring-beans-2.0.dtd"
~~~

XML Schema语言就是XSD（XML Schemas Definition）。XML Schema描述了XML文档的结构。可以用一个指定的XML Schema来验证某个XML文档，以检查该XML文档是否符合其要求。文档设计者可以通过XML Schema指定一个XML文档所允许的结构和内容，并可根据此检查一个XML文档是否有效。XML Schema本身是一个XML文档，它符合XML语法结构。可以用通过XML解析器解析它。

​	在使用XML Schema wendagn 对XML实例文档进行检查，除了要声明空间外（xmlns:http://www.Springframework.org/schema/beans）,还必须指定该名称空间所对应的XML Schema 文档的存储位置。通过schemaLocation属性来指定名称空间所对应的XML Schema文档的存储位置，它包含两个部分，一部分是名称空间的URI，另一部分就是该名称空间所表示的XML Schema文件位置或URL地址（xsi:schemaLocation="http://www.Springframework.org/schema/beans http://www.Springframework.org/schema/beans/Spring-beans.xsd"）

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.Springframework.org/schema/beans"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="http://www.Springframework.org/schema/beans http://www.Springframework.org/schema/beans/Spring-beans.xsd">
~~~





EntityResolver

​	如果SAX应用程序需要实现自定义处理外部实体，则必须实现此接口并使用setEntityResolver方法向SAX驱动器注册一个实例。也就是说，对于解析一个XML，SAX首先读取该XML文档上的声明，根据声明去寻找相应的DTD定义，以便对文档进行一个验证。默认的寻找规则，即通过网络（实现上就是声明DTD的URI地址）来下载相应的DTD声明，并进行认证。下载的过程是漫长的过程，而且当网络中断或不可用时，这里会报错，就是因为相应的DTD声明没有找到的原因。	

​	EntityResolver的的作用是项目本身就可以提供一个如何寻找DTD声明的方法，即由程序来实现寻找DTD声明的过程，比如我们将DTD文件放到项目中某处，在实现时直接将此文档读取并返回给SAX即可。这样就避免了通过网络来寻找相应的声明。



profile 

​	解析xml文件时，可以部署不同环境的配置文件，通过profile进行区分，