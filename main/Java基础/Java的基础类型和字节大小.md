# Java的基础类型和字节大小

基本数据类型 | 字节大小
---|---
int | 4字节（32位，最大值2^31-1）
byte | 1 字节（8位）
short | 2 字节（16位）
Long | 8 字节(64位)
double | 8 字节（64位）
float | 4字节（32位）
`boolean` | 1位,至少1字节
char | 2 字节（16位）
关于boolean占几个字节，众说纷纭，虽然boolean表现出非0即1的“位”特性，但是存储空间的基本计量单位是字节，不是位。</br>
所以boolean应当至少占一个字节。</br>
JVM规范中，boolean变量作为int处理，也就是4字节；<br/>
boolean数组当做byte数组处理。