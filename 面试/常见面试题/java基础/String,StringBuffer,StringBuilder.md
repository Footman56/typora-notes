String 是不可变的。StringBuffer，StringBuilder 都是可变的

String 是线程安全的，每次操作都会创建一个String 对象，  拼接时有会有对象创建

StringBuffer 也是线程安全的， 通过synchronized 字段来实现同步

StringBuilder不安全的 单线程下性能好