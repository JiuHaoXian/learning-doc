# Java的字符



#### String、StringBuilder、StringBuffer三者区别

* String：不可变字符序列
* StringBuilder（JDK1.5）：可变字符序列、效率高、线程不安全
* StringBuffer：可变字符序列、效率低、线程安全，通过**synchronized**关键字实现。
* 初始化上的区别，String可以空赋值，后者不行，报错



#### String为什么不可变？

1. 字符串常量池的需要

2. 允许String对象缓存HashCode

3. 安全性



#### Java9的改进

  Java9改进了字符串（包括String、StringBuffer、StringBuilder）的实现。在Java9以前字符串采用char[]数组来保存字符，因此字符串的每个字符占2字节；而Java9的字符串采用byte[]数组再加一个encoding-flag字段来保存字符，因此字符串的每个字符只占1字节。所以Java9的字符串更加节省空间，字符串的功能方法也没有受到影响。





