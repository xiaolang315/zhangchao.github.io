# learn

## how to delete all other branch exclude master

``` shell
git branch -D `git branch | grep -v master | xargs`
```

## 选择链接
在链接.a时，如果没有使用里面的内容，不会把符号搬移到最终的可执行程序里。如果全部文件只链接一个可执行程序，那么不管用不用.o中的内容，都会包含在内。因此想要做到链接目标最小化，尽可能的组件化是非常有必要执行的。

