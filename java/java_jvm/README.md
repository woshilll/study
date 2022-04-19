# Java_JVM

## JIT 运行方式
>JVM有两种运行模式-server -client

区别
- server VM:初始堆空间大些, 默认使用并行垃圾回收器, 启动慢运行快
- client VM:初始堆空间小写, 默认使用串行垃圾回收器, 启动快运行满
- 64位系统:只有server模式

## JVM 架构


#### javap命令

> 直接javap -help搜索呗


### 运行时数据区

### 执行引擎

## JVM 参数

### 1.标准参数
> 一般稳定，在未来JVM版本中不会改变

```shell
java -help
java -version
```
### 2.-x参数


### 3.-xx参数