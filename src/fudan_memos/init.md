# initを検討

## shellスクリプトを動かす

```
$ cat > rc
echo "abc"
ls /home
$ cat rc
echo "abc"
ls /home
$ /bin/dash rc
abc
vagrant
```

## initでやりたいこと

- devファイルの作成
- ext2のmount

## v7x86の`init.c`

```c
```
