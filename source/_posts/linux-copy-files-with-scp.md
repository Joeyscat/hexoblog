---
title: Linux下使用scp命令来通过ssh传输文件
date: 2019-06-30 20:10:28
categories:
- 笔记
tags:
- Linux
---
### Linux下使用scp命令来通过ssh传输文件



####  从服务器上下载文件到本地

###### 例如将服务器上的logo文件下载到本地

```bash
$ scp username@servername:/home/logo.png /home/joey/download
```



#### 上传本地文件到服务器

##### 将本地readme文件上传到服务器/home目录

```bash
$ scp /home/joey/readme.txt username@servername:/home/
```



#### 从服务器上下载整个目录

##### 例如从服务器下载log目录（包含所有子文件）

```bash
$ scp -r username@servername:/home/log/ /home/joey/download
```



#### 上传目录到服务器

##### 例如将本地img目录上传到服务器/home目录

```bash
$ scp -r /home/joey/img/ username@servername:/home/
```



#### ——username:服务器用户名（需要相应密码）

#### ——servername：服务器地址（IP）