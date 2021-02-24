# GIT 使用备忘

## misc(杂项)

- git clone hunter@localhost:/home/hunter/git_practice/INIT_GIT/.git INIT_GIT_COPY

不借助 gitblit,github 等第三方服务即可让任意用户从其他任意用户的本地复制 git 仓库(*相比 gitblit,github 少了权限管理功能*)

本质上是借助于机器的ssh功能进行目录的复制,是不是 scp 命令有点像?
