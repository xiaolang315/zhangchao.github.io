# learn

## how to delete all other branch exclude master

``` shell
git branch -D `git branch | grep -v master | xargs`
```


