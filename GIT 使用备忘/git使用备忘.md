# GIT 使用备忘

## misc(杂项)

- git clone hunter@localhost:/home/hunter/git_practice/INIT_GIT/.git INIT_GIT_COPY

不借助 gitblit,github 等第三方服务即可让任意用户从其他任意用户的本地复制 git 仓库(*相比 gitblit,github 少了权限管理功能*)

本质上是借助于机器的ssh功能进行目录的复制,是不是 scp 命令有点像?

- 不同目录下的 .gitignore

根目录下的 .gitignore 应用到整个项目,子目录下的的 .gitignore 只作用其所在的目录.

```shell
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```
