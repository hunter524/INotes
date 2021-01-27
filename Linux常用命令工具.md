# Linux

## Linux 常用命令工具

- tmux

- unar

- dstat
  
    流量等PC 资源使用统计工具(python写的).使用 --nocolor 选项时 dstat 才会显示单位而不是显示颜色.

- ufw
  
   防火墙配置工具

- sdkman(sdk)
  
  java,gradle,kotlin,groovy,maven 等等开发 sdk 的管理工具.

- nvm

  <https://github.com/nvm-sh/nvm> 用于管理 node 的版本.

- convmv/iconv/dos2unix

convmv 文件名称编码转换工具
iconv 文件内容编码
dos2unix 文件换行符,转换工具 win \r\n -> unix 

## Linux 配置

### fcitx

- fcitx-diagnose

  查看 fcitx 配置可能存在的问题.

- XMODIFIERS
  
  Gnome On Wayland 用户无法使用 fcitx由于 wayland 无法读取 ~/.xprofile 中的环境变量，所以请在/etc/environment中加入：(Wayland 与 Xwindow 均是一个与显示服务通信的协议)

  export GTK_IM_MODULE=fcitx
  export QT_IM_MODULE=fcitx
  export XMODIFIERS=@im=fcitx
  
## Linux 效率命令

- pushd/popd

  Shell 内置命令.pushd 用于切换到指定目录,并且将切换的目录和当前目录入栈.popd 则弹出栈顶的目录,切换到栈的下一个目录.

## Linux 常用配置

### /etc/profile 配置

```shell
export PATH=$PATH:/home/hunter/AndroidSoft/AndroidSdk/tools
export PATH=$PATH:$HOME/.local/bin
export PATH=$PATH:/home/hunter/AndroidSoft/AndroidSdk/platform-tools
export PATH=$PATH:/home/hunter/AndroidSoft/AndroidSdk/tools/bin
export PATH=$PATH:/home/hunter/kotlin-compiler-1.3.61/kotlinc/bin
export ANDROID_HOME=/home/hunter/AndroidSoft/AndroidSdk/

export GRADLE_HOME=/home/hunter/.sdkman/candidates/gradle/current
export PATH=$PATH:$GRADLE_HOME/bin
export JAVA_HOME=/home/hunter/.sdkman/candidates/java/current
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export FLUTTER_HOME=/home/hunter/flutter/flutter_linux_v1.0.0-stable/flutter
export PATH=$FLUTTER_HOME/bin:$PATH
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

export DART_HOME=/usr/lib/dart
export PAHT=$DART_HOME/bin:$PATH
#flutter lincense problem
#export SDKMANAGER_OPTS="--add-modules java.se.ee"


export ANT_HOME=/home/hunter/AndroidSoft/apache-ant-1.9.14
export PATH=$ANT_HOME/bin:$PATH

export MVN_HOME=/home/hunter/AndroidSoft/apache-maven-3.6.3
export PATH=$MVN_HOME/bin:$PATH
#export HTTPS_PROXY=http://hexin:hx300033@127.0.0.1:1082/
#export ftp_proxy=http://hexin:hx300033@127.0.0.1:1082/
#export FTP_PROXY=http://hexin:hx300033@127.0.0.1:1082/
#export https_proxy=http://hexin:hx30003@127.0.0.1:1082/
#export HTTP_PROXY=http://hexin:hx300033@127.0.0.1:1082/
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

#export http_proxy=http://hexin:hx300033@127.0.0.1:1082/

#export PATH=/home/hunter/smartsvn/bin:$PATH 
alias sctl="sudo systemctl"
```
