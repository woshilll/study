# class文件介绍

## class文件里存了什么数据
>xxx-数字 数字表示的字节数
> <br>命令：javap -v [class]

魔数-4|副版本号-2|主版本号-2|
常量池计数器-2|常量池数据区-n|
访问标志-2|类索引-2|父类索引-2|
接口计数器-2|接口信息数据区-n|
字段计数器-2|字段信息数据区-n|
方法计数器-2|方法信息数据区-n|
属性计数器-2|属性信息数据区-n|

## class常量池
> cp_info: 常量池项
>
> constant_pool_count: 常量池计数器
>
> 常量池计数器是从 **1** 开始计数, 而不是从0开始的, 如果constant_pool_count=22
> 则后面的常量池项(cp_info)的个数是21 （第0项常量空出来单独考虑，为满足某些指向常量池的索引值的数据在待定的情况下表达不引用任何一个常量池项的意思）

### cp_info结构

     cp_info {
 
       u1    tag;
 
       u1    info[];
 
     }
> tag - 1个字节<br>info[] - 1个字节组成的数组

#### tag
> 1-6属于字面量结构体, 剩余是引用型结构体
> <br>字面量结构体：一个cp_info表示，info[]里存储的是字面量值
> <br>引用型结构体：两个cp_info表示，一个cp_info表示引用类型，一个cp_info表示utf8_info

| Tag值 | 表示的字面量 | 更细化的结构 |
| :---- | :---- | :---- |
| 1 | 字符串常量 | CONSTANT_Utf8_info |
| 3 | 4字节int | CONSTANT_Integer_info |
| 4 | 4字节float | CONSTANT_Float_info |
| 5 | 8字节long | CONSTANT_Long_info |
| 6 | 8字节double | CONSTANT_Double_info |
| 7 | 类或接口全限定名 | CONSTANT_Class_info |
| 8 | String类型 | CONSTANT_String_info |
| 9 | 类中的字段 | CONSTANT_Fieldref_info |
| 10 | 类中的方法 | CONSTANT_Methodref_info |
| 11 | 类所实现的接口的方法 | CONSTANT_InterfaceMethodref_info |
| 12 | 字段或方法的名称和类型 | CONSTANT_NameAndType_info |
| 15 | 方法句柄 | CONSTANT_MethodHandle_info |
| 16 | 方法类型 | CONSTANT_MethodType_info |
| 18 | invokedynamic指令所使用<br>到的引导方法、引导方法<br>使用到动态调用名称、参数<br>和请求返回类型等 | CONSTANT_InvokeDynamic_info |

#### 示例

1. tag 为1-6
    ```
    cp_info {
      u1 tag=1;
      un info=[xxx];
    }
    ```
2. tag 为7-18
    ```
    cp_info1 {
     u1 tag=7
     u1 info=[cp_info2索引]
    }
    cp_info2 {
     u1 tag=1;
     un info=[xxx];
    }
    ```

### long和double在常量池中是如何存储的
```
long_cp_info {
  u1 tag=5;
  u4 high_bytes;
  u4 low_bytes;
}

double_cp_info {
  u1 tag=6;
  u4 high_bytes;
  u4 low_bytes;
}
```
> 由此可知long和double虽然是8个字节，但是在计算机中是分成两个4字节进行存储，分为高4位和低4位（为了在32位和64位系统通用）

引出：long和double在操作的时候是否存在线程安全问题

### double、float、long不声明final就能在常量池中存在，int不声明final也会存在吗？
[解答](#classInt)


### String类型的字符串常量在常量池中是怎样存储的？
```
string_cp_info {
  u1 tag=8;
  u2 constant_pool_index;
}
utf8_cp_info {
  u1 tag=1;
  u2 length;
  u1 bytes[length]
}
```

#### 那么，字符串常量是否有长度限制呢？

### 哪些字面量会进入常量池中
1. final类型的8中基本类型的值
2. <span id="classInt">非final类型（包括static的）的8中基本类型的值，只有double、float、long的值会进入常量池</span>
3. 常量池中包含的字符串类型字面量（双引号引起来的字符串值）


## class中的符号引用和直接引用

### 符号引用
> 以一组符号来描述所引用的目标，符号可以使任何形式的字面量。如"/java/Math"<br>
> 与内存无关

### 直接引用
> 1. 指向目标的指针
> 2. 相对偏移量
> 3. 一个能间接定位到目标的句柄

### 引用替换的时机

> 类加载过程(加载 -> 连接(验证、准备、解析) -> 初始化中的解析阶段

## class中的特殊字符串
1. 类的全限定名
2. 字段和方法的描述符
3. 特俗方法的方法名

### 类的全限定名

> 在java文件中是java.lang.Object<br>
> 在class文件中是java/lang/Object

### 描述符
> 1. 类型描述符
> 2. 字段描述符
> 3. 方法描述符

#### 字段类型描述符

| 数据类型 | 描述符 |
| :---- | :---- |
| byte | B |
| char | C |
| double | D |
| float | F |
| int | I |
| long | J |
| short | S |
| boolean | Z |
| 特殊类型void | V |
| 对象类型 | L + 类的全限定名 + ; 如：Ljava/lang/String;|
| 数组类型 | 若干个[ + 数组中元素类型对应的字符串 如 String[][] -> [[Ljava/lang/String;|

#### 方法描述符

| 方法描述符 | 方发声明 |
| :---- | :---- |
| ()I | int getSize() |
| ()Ljava/lang/String; | String toString() |
| ([Ljava/lang/String;)V | void main(String[] args) |
| ()V | void wait() |
| (JI)V | void wait(long timeout, int nanos) |
| (ZILjava/lang/String;TT)Z | boolean test(boolean a, int b, String c int d, int e) |
| ([BII)I | int test(byte[] b, int a, int c) |
| ()[[Ljava/lang/Object; | Object[][] test() |

#### 特殊方法的方法名
> 类的构造方法和类型初始化方法<br>
> 构造方法 <init><br>
> 静态初始化方法 <clinit>