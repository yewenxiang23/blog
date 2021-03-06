---
title: Linux基本命令
date: 2018-09-26 17:46:01
tags:
  - bash
---

### cd跳转命令 `Change Directory`
翻译过来就是改变文件夹 ,cd后面要跟一个文件夹的名字
```
cd dir
```
- 输入文件夹名字时支持tab补齐
- 两个点表示跳到上一级文件夹中 eg: `cd ..`

cd跳转又分为相对路径和绝对路径

- 相对路径

    打开当前文件夹里面的文件夹,或者 `cd .. ` 跳到上一个文件夹

- 绝对路径

    使用绝对路径最大的好处是跟当前位置无关 以老祖宗文件夹 `/` 打头,例如：不管我们在哪 `cd /` 都可以跳到根目录中, `cd /Users/wenxiangye` 可以直接跳到用户主目录 等价于 `cd ~`.

### 打印出的当前位置

```
pwd
```

### 列出当前文件夹中的所有文件和文件夹

```
ls
需要列出隐藏的文件夹时用 `ls -a`
```


### 创建一个文件

```
touch file
atom file
```

>通常使用atom编辑器创建一个文件
>

### 创建一个文件夹
```
mkdir dir
```

### 删除文件夹 `remove`

```
rm file
删除file文件
```


```
rm -r dir
删除文件夹
```

>注意 .git 文件夹有写保护 需要使用 rm -rf dir 来强制删除
>

```
rm -r *
可以删除当前位置的所有文件个文件夹
```


### 拷贝操作 `copy`
```
cp file1 file2
拷贝文件file1 变成 file2, file1和file2就文件名字不同
```


```
cp -r dir1 dir2
拷贝文件夹dir1 变成 dir2, dir1和dir2文件夹中的内容是相同的
```


```
mv aaa/* bbb
把文件夹aaa中的所有文件全部拷贝到bbb文件夹中
```


### 重命名和移动操作 `move`
```
mv file dir
mv dir1 dir2
移动file文件到dir文件夹中
```


>可以通过`file dir1`来查看文件的类型
>

```
mv file1 file2
mv dir dir1
```
>这里是把文件file1改名成了file2,当前目录不能存在 file2和dir1文件，如果存在则是移动和替换操作
>

```
mv file ..
移动文件到上一级目录
```


### 查看文件
```
cat file
会列出文件里面的内容
```


---

### 普通用户和超级用户

我们平时在用户主目录文件夹里面，用普通用户的身份就能去增删改的操作了，但是在其他目录下面需要超级用户的身份才能去执行操作，实现的方式:
```
sudo rm file
上面的命令可以用超级用户权限执行一个命令
```


```
sudo su
```
可以直接化身超级用户,可以输入 `whoami` 打印出当前用户的身份，退出超级用户成普通用户，敲 `Ctrl-D`
