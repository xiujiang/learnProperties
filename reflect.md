# reflect

## 1.interface

### 1. AnnotatedArrayType	

​	`AnnotatedArrayType`表示数组类型的潜在注释使用，其组件类型本身可以表示类型的注释使用。

​	**开始时间**：1.8

​	**继承**：AnnotatedArrayType-->AnnotatedType-->AnnotatedElement

​	**方法**：AnnotatedType getAnnotatedGenericComponentType()

​		返回此数组类型的可能注释的通用组件类型。

### 2.AnnotatedElement

​	表示当前在此VM中运行的程序的注释元素。该界面允许反射读取注释。 通过这个接口方法返回的所有注释都是不可变并且可序列化。	

​	**开始时间**：1.5

​	**方法**

​		<T extends Annotation> T getAnnotation(类<T> annotationClass)

​		Annotation[] getAnnotations()

​		<T extends Annotation> T[] getDeclaredAnnotationsByType(类<T> annotationClass)

​		Annotation[] getDeclaredAnnotations()	//返回此类上的注释，忽略继承的注释

​		isAnnotationPresent()([类](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/Class.html)<? extends [Annotation](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/annotation/Annotation.html)> annotationClass)

​		 <T extends Annotation> T getDeclaredAnnotation(类<T> annotationClass)

​		 <T extends Annotation> T[] getAnnotationsByType(类<T> annotationClass)

### 3.AnnotatedType

​	`AnnotatedType`表示当前在此VM中运行的程序中可能注释的类型的使用。 使用可能是Java编程语言中的任何类型，包括数组类型，参数化类型，类型变量或通配符类型。

​	**开始时间** 1.8

​	**继承** AnnotatedType -->AnnotatedElement

​	**方法** 

​		Type getType()	//返回此注释类型标识的底层类型

### 4.AnnotatedParameterizedType

​	`AnnotatedParameterizedType`表示参数化类型的潜在注释使用，其类型参数本身可以表示类型的注释使用。

​	**开始时间** 1.8

​	**继承** AnnotatedParameterizedType-->AnnotatedType

​	**方法** 

​		AnnotatedType[] getAnnotatedActualTypeArguments()	

​		返回此参数化类型的可能注释的实际类型参数

### 5.AnnotatedTypeVariable

​	`AnnotatedTypeVariable`表示类型变量的潜在注释使用，其声明可能具有其自身表示注释类型使用的界限。

​	**开始时间** 1.8

​	**继承** AnnotatedTypeVariable-->AnnotatedType

​	**方法**

​		AnnotatedType[] getAnnotatedBounds()		

​		返回此类型变量的可能注释的边界

### 6.AnnotatedWildcardType

​	`AnnotatedWildcardType`表示通配符类型参数的潜在注释使用，其上限或下限本身可以表示类型的注释使用。

​	**开始时间** 1.8

​	**继承** AnnotatedWildcardType-->AnnotatedType

​	**方法**

​		AnnotatedType[] getAnnotatedLowerBounds()

​		AnnotatedType[] getAnnotatedUpperBounds()

### 7.GenericArrayType

​	`GenericArrayType`表示组件类型是参数化类型或类型变量的数组类型。

​	**开始时间** 1.5

​	**继承** GenericArrayType --> Type

​	**方法** 

​		Type getGenericComponentType()

​		返回表示此数组的组件类型的`Type`对象。

​		**Exception**:TypeNotPresentException，MalformedParameterizedTypeException

### 8.GenericDeclaration

​	声明类型变量的所有实体的通用接口

​	**开始时间** 1.5

​	**继承** GenericDeclaration-->AnnotationElement

​	**方法**

​		TypeVariable<?>[] getTypeParameters()

### 9.InvocationHandler

​	InvocationHandler 是由代理实例的实现类实现的接口

​	每个代理实例都有一个关联的接口，当调用实例中的某个方法时，方法调用会被分配到invoke方法

​	**开始时间** 1.3

​	**方法**

​		Object invoke(Object proxy,method method,Object[] args) throws Throwable

### 10.Member

​	Member是一个反映关于单个成员（字段或者方法）或构造函数的标识信息接口

​	**属性**

| `static final int`  | `DECLARED` | 标识类或接口的已声明成员集                         |
| ------------------- | ---------- | -------------------------------------------------- |
| ` static final int` | `PUBLIC`   | 标识类或接口的所有公共成员的集合，包括继承的成员。 |

​	**方法**

​		Class<?> getDeclaringClass()

​		返回表示声明该成员表示的成员或构造函数的类或接口的Class对象。

​		String getName()

​		返回由此成员表示的基础成员或构造函数的简单名称。

​		int getModifiers()

​		返回此成员的java 语言修饰符	[`Modifier`](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/Modifier.html)

​		boolean isSynthetic()	

​		该成员是否是由编译器引入

### 11.ParameterizedType

​	ParameterizedType表示一个参数化类型，如：Collection<String>

​	实现此接口，必须实现equals()方法

​	**开始时间** 1.5

​	**继承** ParameterIzedType-->Type

​	**方法** 

​		Type[] getActualTypeArguments()	

​		Type getRawType()

​		Type getOwnerType() 

### 12.Type

​	Type是java中所有类型的通用父接口，包括原始类型，参数化类型，数组类型，变量类型。

​	**开始时间** 1.5

​	**方法** 

​		String getTypeName()	

​		返回描述此类型的字符串,1.8开始，默认实现为调用toString()

### 13.TypeVariable<D extends [GenericDeclaration](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/GenericDeclaration.html)>

​	TypeVariable是类型变量的常用父接口。

​	**开始时间** 1.5

​	**继承** TypeVariable-->AnnotatedElement,Type

​	**方法**

​		Type[] getBounds()

​		D getGenericDeclaration()

​		String getName()

​		AnnotatedType[] getAnnotatedBounds()		

### 14.WildcardType

​	WildcardType表示一个通配符表达,如？，？ extends Number或 ？ extends Integer

​	**开始时间** 1.5

​	**继承** WildcardType -->Type

​	**方法**

​		Type[] getUpperBounds()

​		Type[] getLowerBounds()

## 2.Classes

### 1.AccessibleObject

​	AccessibleObject是Field,Method和Constructor对象的基类，它提供了将反射对象标记为在使用它时默认Java语言访问控制检查的功能。当使用Fields，Methods或Constructors来设置或获取字段，调用方法，或创建和初始化新的类实例时，执行访问检查（对于public，默认（包）访问，受保护和私有成员） 分别。在反射对象中设置`accessible`标志允许具有足够权限的复杂应用（如Java对象序列化或其他持久性机制）以通常被禁止的方式操纵对象。

​	默认情况下，反射对象不可访问

​	**开始时间** 1.2

​	**继承** AccessibleObject-->AnnotatedElement

​	**构造方法**

~~~ java
protected AccessibleObject()   //仅由java虚拟机使用
~~~

​	**方法**

| `<T extends Annotation>T`   | `getAnnotation(类<T> annotationClass)`返回该元素的，如果这样的注释 *，*否则返回null指定类型的注释。 |
| --------------------------- | ------------------------------------------------------------ |
| `Annotation[]`              | `getAnnotations()`返回此元素上 *存在的*注释。                |
| `<T extends Annotation>T[]` | `getAnnotationsByType(类<T> annotationClass)`返回与此元素相关 联的注释 。 |
| `<T extends Annotation>T`   | `getDeclaredAnnotation(类<T> annotationClass)`如果这样的注释 *直接存在* ，则返回指定类型的元素注释，否则返回null。 |
| `Annotation[]`              | `getDeclaredAnnotations()`返回 *直接存在*于此元素上的注释。  |
| `<T extends Annotation>T[]` | `getDeclaredAnnotationsByType(类<T> annotationClass)`如果此类注释 *直接存在*或 *间接存在，*则返回该元素的注释（指定类型）。 |
| `boolean`                   | `isAccessible()`获取此对象的 `accessible`标志的值。          |
| `boolean`                   | `isAnnotationPresent(类<? extends Annotation> annotationClass)`如果此元素上 *存在*指定类型的注释，则返回true，否则返回false。 |
| `static void`               | `setAccessible(AccessibleObject[] array, boolean flag)`方便的方法来设置 `accessible`标志的一系列对象的安全检查（为了效率）。 |
| `void`                      | `setAccessible(boolean flag)`将此对象的 `accessible`标志设置为指示的布尔值。true标识不执行语言访问检查 |

### 2.Array

​	Array类提供静态方法来动态创建和访问java数组

​	Array允许在执行get/set操作时扩大数组，如果缩小数组，则会抛出`IllegalArgumentException` 

| `static Object`  | `get(Object array, int index)`返回指定数组对象中的索引组件的值。 |
| ---------------- | ------------------------------------------------------------ |
| `static boolean` | `getBoolean(Object array, int index)`返回指定数组对象中的索引组件的值，如 `boolean` 。 |
| `static byte`    | `getByte(Object array, int index)`返回指定数组对象中的索引组件的值，如 `byte` 。 |
| `static char`    | `getChar(Object array, int index)`返回指定数组对象中索引组件的值，如 `char` 。 |
| `static double`  | `getDouble(Object array, int index)`返回指定数组对象中的索引组件的值，如 `double` 。 |
| `static float`   | `getFloat(Object array, int index)`返回指定数组对象中的索引组件的值，如 `float` 。 |
| `static int`     | `getInt(Object array, int index)`返回指定数组对象中的索引组件的值，如 `int` 。 |
| `static int`     | `getLength(Object array)`返回指定数组对象的长度，如 `int` 。 |
| `static long`    | `getLong(Object array, int index)`返回指定数组对象中索引组件的值，如 `long` 。 |
| `static short`   | `getShort(Object array, int index)`返回指定数组对象中的索引组件的值，如 `short` 。 |
| `static Object`  | `newInstance(类<?> componentType, int... dimensions)`创建具有指定组件类型和尺寸的新数组。 |
| `static Object`  | `newInstance(类<?> componentType, int length)`创建具有指定组件类型和长度的新数组。 |
| `static void`    | `set(Object array, int index, Object value)`将指定数组对象的索引组件的值设置为指定的新值。 |
| `static void`    | `setBoolean(Object array, int index, boolean z)`将指定数组对象的索引组件的值设置为指定的 `boolean`值。 |
| `static void`    | `setByte(Object array, int index, byte b)`将指定数组对象的索引组件的值设置为指定的 `byte`值。 |
| `static void`    | `setChar(Object array, int index, char c)`将指定数组对象的索引组件的值设置为指定的 `char`值。 |
| `static void`    | `setDouble(Object array, int index, double d)`将指定数组对象的索引组件的值设置为指定的 `double`值。 |
| `static void`    | `setFloat(Object array, int index, float f)`将指定数组对象的索引组件的值设置为指定的 `float`值。 |
| `static void`    | `setInt(Object array, int index, int i)`将指定数组对象的索引组件的值设置为指定的 `int`值。 |
| `static void`    | `setLong(Object array, int index, long l)`将指定数组对象的索引组件的值设置为指定的 `long`值。 |
| `static void`    | `setShort(Object array, int index, short s)`将指定数组对象的索引组件的值设置为指定的 `short`值。 |

### 3.Executable

Method和Constructor 共同功能的父类

**开始时间** 1.8

**继承** Executable-->AnnotatedElement,GenericDeclaration,Member

**方法**

| `AnnotatedType[]`            | `getAnnotatedExceptionTypes()`返回一个 `AnnotatedType`对象的数组， `AnnotatedType`使用类型来指定由此可执行文件表示的方法/构造函数声明的异常。 |
| ---------------------------- | ------------------------------------------------------------ |
| `AnnotatedType[]`            | `getAnnotatedParameterTypes()`返回一个 `AnnotatedType`对象的数组， `AnnotatedType`使用类型来指定此可执行文件所表示的方法/构造函数的形式参数类型。 |
| `AnnotatedType`              | `getAnnotatedReceiverType()`返回一个 `AnnotatedType`对象，该对象表示使用类型来指定此可执行文件对象所表示的方法/构造函数的接收器类型。 |
| `abstract AnnotatedType`     | `getAnnotatedReturnType()`返回一个 `AnnotatedType`对象，表示使用一个类型来指定此可执行文件所表示的方法/构造函数的返回类型。 |
| `<T extends Annotation>T`    | `getAnnotation(类<T> annotationClass)`返回该元素的，如果这样的注释 *，*否则返回null指定类型的注释。 |
| `<T extends Annotation>T[]`  | `getAnnotationsByType(类<T> annotationClass)`返回与此元素相关 *联的注释* 。 |
| `Annotation[]`               | `getDeclaredAnnotations()`返回 *直接存在*于此元素上的注释。  |
| `abstract 类<?>`             | `getDeclaringClass()`返回 `类`表示声明该对象表示的可执行的类或接口对象。 |
| `abstract 类<?>[]`           | `getExceptionTypes()`返回一个 `类`对象的数组， `类`表示由该对象表示的底层可执行文件抛出的异常类型。 |
| `Type[]`                     | `getGenericExceptionTypes()`返回一个 `Type`对象的数组， `Type`表示声明为该可执行对象抛出的异常。 |
| `Type[]`                     | `getGenericParameterTypes()`返回一个 `Type`对象的数组， `Type`以声明顺序表示由该对象表示的可执行文件的形式参数类型。 |
| `abstract int`               | `getModifiers()`返回由该对象表示的可执行文件的Java语言[modifiers](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/Modifier.html) 。 |
| `abstract String`            | `getName()`返回由此对象表示的可执行文件的名称。              |
| `abstract Annotation[][]`    | `getParameterAnnotations()`返回一个 `Annotation` s的数组数组，表示由该对象表示的Executable的形式参数的声明顺序的 `Executable` 。 |
| `int`                        | `getParameterCount()`返回由此对象表示的可执行文件的形式参数（无论是显式声明还是隐式声明）的数量。 |
| `Parameter[]`                | `getParameters()`返回一个 `Parameter`对象的数组，表示由该对象表示的底层可执行文件的所有参数。 |
| `abstract 类<?>[]`           | `getParameterTypes()`返回一个 `类`对象的数组， `类`以声明顺序表示由该对象表示的可执行文件的形式参数类型。 |
| `abstract TypeVariable<?>[]` | `getTypeParameters()`返回一个 `TypeVariable`对象的数组，它们以声明顺序表示由此 `GenericDeclaration`对象表示的通用声明声明的类型变量。 |
| `boolean`                    | `isSynthetic()`返回`true`如果这个可执行文件是一个合成的构建体; 返回`false`其他。 |
| `boolean`                    | `isVarArgs()`返回`true`如果这个可执行文件被宣布为带有可变数量的参数; 返回`false`否则。 |
| `abstract String`            | `toGenericString()`返回描述此 `Executable`的字符串，包括任何类型的参数。 |

### 4.Constructor

​	Constructor提供了一个类的单个构造函数的信息和访问

​	Constructor允许在将实际参数与newInstance()与底层构造函数的形式参数进行匹配时进行扩展转换，但如果发生缩小转换，则抛出IllegalArgumentException.

​	**继承** Constructor-->Executor

### 5.Method

​	Method提供有关类和接口上单一方法的信息和访问权限。反映的方法可以是类方法，也可以是实例方法

​	Method允许在匹配实际参数以使用底层方法的形式参数进行`IllegalArgumentException`时发生扩展转换

​	**继承** Method-->Executor

### 6.Field

​	`Field`提供有关类或接口的单个字段的信息和动态访问。 反射的字段可以是类（静态）字段或实例字段。

​	`Field`允许在获取或设置访问操作期间扩展转换，但如果发生缩小转换，则抛出IllegalArgumentException.

​	**继承** Field-->AnnotatedElement,Member

### 7.Modifier

​	Modifier类提供了`static`方法和常量来解码类和成员访问修饰符。 修饰符集合被表示为具有表示不同修饰符的不同位位置的整数。 

**属性**

| `static int` | `ABSTRACT``int`值代表 `abstract`修饰符。         |
| ------------ | :----------------------------------------------- |
| `static int` | `FINAL``int`值代表 `final`修饰符。               |
| `static int` | `INTERFACE``int`值代表 `interface`修饰符。       |
| `static int` | `NATIVE``int`值代表 `native`修饰符。             |
| `static int` | `PRIVATE``int`值代表 `private`修饰符。           |
| `static int` | `PROTECTED``int`值代表 `protected`修饰符。       |
| `static int` | `PUBLIC``int`值代表 `public`修饰符。             |
| `static int` | `STATIC``int`值代表 `static`修饰符。             |
| `static int` | `STRICT``int`值代表 `strictfp`修饰符。           |
| `static int` | `SYNCHRONIZED``int`值代表 `synchronized`修饰符。 |
| `static int` | `TRANSIENT``int`值代表 `transient`修饰符。       |
| `static int` | `VOLATILE``int`值代表 `volatile`修饰符。         |

**方法**

| static int | `classModifiers()`返回一个 `int`值将可以应用于类的源语言修饰符组合起来。 |
| :--------- | :----------------------------------------------------------: |
| static int | `constructorModifiers()`将一个 `int`值返回给可以应用于构造函数的源语言修饰符。 |
| static int | `fieldModifiers()`返回一个 `int`值可以将可以应用于字段的源语言修饰符OR组合在一起。 |
| static int | `interfaceModifiers()`返回一个 `int`值OR-in可以应用于接口的源语言修饰符。 |

### 8.Parameter

​	Parameter是有关方法参数的信息，包括其名称和修饰符，它还提供了获取参数属性的代替方法。

### 9.Proxy

​	`Proxy`提供了创建动态代理类和实例的静态方法，它也是由这些方法创建的所有动态代理类的超类。

~~~java
为某个接口创建代理Foo ：
  InvocationHandler handler = new MyInvocationHandler(...);
     Class<?> proxyClass = Proxy.getProxyClass(Foo.class.getClassLoader(), Foo.class);
     Foo f = (Foo) proxyClass.getConstructor(InvocationHandler.class).
                     newInstance(handler); 
或更简单地：
  Foo f = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(),
                                          new Class<?>[] { Foo.class },
                                          handler); 
~~~

代理类具有以下属性：

- 代理类是*公共的，最终的，而不是抽象的，*如果所有代理接口都是公共的。

- 如果任何代理接口*是非公开的，*代理类*是非公开的，最终的，而不是抽象的* 。

- 代理类的不合格名称未指定。 然而，以字符串`"$Proxy"`开头的类名空间应该保留给代理类。

- 一个代理类扩展了`java.lang.reflect.Proxy` 。

- 代理类完全按照相同的顺序实现其创建时指定的接口。

- 如果一个代理类实现一个非公共接口，那么它将被定义在与该接口相同的包中。 否则，代理类的包也是未指定的。 请注意，程序包密封不会阻止在运行时在特定程序包中成功定义代理类，并且类也不会由同一类加载器定义，并且与特定签名者具有相同的包。

- 由于代理类实现了在其创建时指定的所有接口， `getInterfaces`在其`类`对象上调用`getInterfaces`将返回一个包含相同列表接口的数组（按其创建时指定的顺序），在其`类`对象上调用`getMethods`将返回一个数组的`方法`对象，其中包括这些接口中的所有方法，并调用`getMethod`将在代理接口中找到可以预期的方法。

- [`Proxy.isProxyClass`](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/Proxy.html#isProxyClass-java.lang.Class-)方法将返回true，如果它通过代理类 - 由`Proxy.getProxyClass`返回的类或由`Proxy.newProxyInstance`返回的对象的类 - 否则为false。

- 所述`java.security.ProtectionDomain`代理类的是相同由引导类装载程序装载系统类，如`java.lang.Object` ，因为是由受信任的系统代码生成代理类的代码。 此保护域通常将被授予`java.security.AllPermission` 。

- 每个代理类有一个公共构造一个参数，该接口的实现[`InvocationHandler`](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/InvocationHandler.html) ，设置调用处理程序的代理实例。 而不必使用反射API来访问公共构造函数，也可以通过调用[`Proxy.newProxyInstance`](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/Proxy.html#newProxyInstance-java.lang.ClassLoader-java.lang.Class:A-java.lang.reflect.InvocationHandler-)方法来创建代理实例，该方法将调用[`Proxy.getProxyClass`](https://blog.fondme.cn/apidoc/jdk-1.8-google/java/lang/reflect/Proxy.html#getProxyClass-java.lang.ClassLoader-java.lang.Class...-)的操作与调用处理程序一起调用构造函数。

  **方法**

| Modifier and Type          | Method and Description                                       |
| -------------------------- | ------------------------------------------------------------ |
| `static InvocationHandler` | `getInvocationHandler(Object proxy)`返回指定代理实例的调用处理程序。 |
| `static 类<?>`             | `getProxyClass(ClassLoader loader, 类<?>... interfaces)`给出类加载器和接口数组的代理类的 `java.lang.Class`对象。 |
| `static boolean`           | `isProxyClass(类<?> cl)`如果且仅当使用 `getProxyClass`方法或 `newProxyInstance`方法将指定的类动态生成为代理类时，则返回true。 |
| `static Object`            | `newProxyInstance(ClassLoader loader, 类<?>[] interfaces, InvocationHandler h)`返回指定接口的代理类的实例，该接口将方法调用分派给指定的调用处理程序。 |

### 10.ReflectPermission

​	反射权限类

​	