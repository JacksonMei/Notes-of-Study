## Shell语法

#### 运行

+ `./aa.sh`
  + 打开一个**subshell**去读取、执行aa.sh，而aa.sh**需要有执行权限**
  + 依赖首行明确解释器类型，否则就不在subshell中执行
  + 通知系统在当前目录找aa.sh,而直接执行aa.sh`，linux 系统将在 PATH里的目录里找aa.sh
+ `sh/bash aa.sh`  
  + 打开一个**subshell**运行解释器去读取、执行aa.sh，文件名作为参数，**无需执行权限**
  + 无需在首行写类似`#!/bin/bash*`来明确解释器
+ `source aa.sh`
  + **当前shell内**读取、执行aa.sh，会保留和改变当前shell中的变量，**无需执行权限**
  + soure可以简化为点号"."，推荐点号

#### 变量

+ \$aa与\${aa}没有区别，但是{}可以用来明确变量边界，同时支持拼接

+ `readonly aa` 设置变量aa为只读

+ `unset aa` 删除变量aa, 不能删只读变量

+ shell变量

  + ```
    $$ // shell本身PID
    $* $@  参数列表（不包括脚本文件名）
    $# 参数列表中参数个数
    $0 脚本文件名
    $n 第n个参数
    ```

#### 字符串

+ 单引号里的任何字符都会原样输出，因而变量是无效的
+ 字符串长度 `${#string}`
+ 截取子串 `${string:1:4}` 截取string位置1开始的4个字符

#### 数组

```shell
array_name=(value0 value1 value2 value3) #不限定长度，下标从0开始，空格分割
array_name[n]=valuen 
${array_name[@]} # 使用 @ 符号可以获取数组中的所有元素
length=${#array_name[@]} # 取得数组元素的个数
lengthn=${#array_name[n]} # 取得数组单个元素的长度
```



#### 多行注释

+ ```shell
  : <<'COMMENT'  
  something
  COMMENT
  或者
  : <<! 
  something
  !
  ```

#### Shell编码

#### cd安全

+ cd可能由于错误路径、无权限等问题而失败，一旦失败后续操作将在错误目录上继续执行，引发误操作。
+ 通过添加错误判断并exit方式避免错误操作。

```shell
cd foo || exit // exit返回cd错误码
cd foo || { echo "Failure"; exit 1; }
```

#### 路径

进入当前脚本路径

```shell
$(cd "$(dirname $0)";pwd)
```

进入当前函数执行路径，如脚本A调用脚本B，在B中执行则得到B的路径

```shell
$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )
```



## 静波代码检视

#### 离线检测

```shell
apt-get install shellcheck
shellcheck   file.sh
```

#### 在线检测

+ 在线检测https://www.shellcheck.net/，贴入代码，自动检测

+ 注意加上 `#!/bin/bash`，否则报错
