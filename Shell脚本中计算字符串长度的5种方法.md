有时在Linux操作系统中需要计算某个字符串的长度，通过查询资料整理了下目前Shell中获取字符串的长度的多种方法，在这里分享给大家，方法如下：
 
方法1: 使用wc -L命令
 wc -L可以获取到当前行的长度，因此对于单独行的字符串可以用这个简单的方法获取，另外wc -l则是获取当前字符串内容的行数。
 

复制代码 代码如下:

echo "abc" |wc -L

 
 
 ------
方法2: expr length string
 使用expr length可以获取string的长度
 
 
----- 
 
方法3: awk获取域的个数，但是如果大于10个字符的长度时是否存在问题需要后面确认

复制代码 代码如下:

echo "abc" |awk -F "" '{print NF}'


-----
方法4: 通过awk+length的方式获取字符串长度
 
复制代码 代码如下:

echo “Alex”|awk '{print length($0)}'

----- 
方法5: 通过echo ${#string}的方式（注意：这里的string是该字符串的变量名）
 
复制代码 代码如下:

name=Alex
 echo ${#name}