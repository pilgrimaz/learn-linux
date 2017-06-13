#### Apache的order allow deny

这个东西确实挺容易让我们迷糊。其实也不难。只要你掌握这样一条规则即可：
首先举个例子：

```shell
Order deny,allow
deny from all
allow from 127.0.0.1
```

我们判断的依据是这样的：

1. 看Order后面的，哪个在前，哪个在后
2. 如果deny在前，那么就需要看deny from这句，然后看allow from这一句
3. 规则是一条一条的匹配的，不管是deny在前还是allow在前，都是会生效的。比如例子中，先deny了所有，然后allow了127.0.0.1，所以127.0.0.1是通过的。

不妨多举几个例子

```shell
Order allow,deny
deny from all
allow from 127.0.0.1
```

这个就会deny所有了，127.0.0.1也会被deny.   因为顺序是先allow然后deny，虽然一开始allow了127.0.0.1，但是后面又拒绝了它。



```shell
Order allow，deny
deny from all
```

全部不能通过



```shell
Order deny,allow
deny from all
```

全部不能通过



```shell
Order deny,allow
```

全部都可以通行（默认的），记住即可



```shell
Order allow,deny
```

全部都不能通行（默认的），记住即可

