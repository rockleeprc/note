## 定义shell脚本和执行
```shell
# 声明脚本使用那种shell解析
#!
# 查看系统支持shell
ls /bin/bash | grep sh
# 优先使用文件中定义的shell解析，没有找到使用系统默认的
./my.sh
# 指定使用bash解析，没有找到使用系统默认的
bash my.sh
. my.sh
```

## 变量声明
```shell
# 定义变量名称，不需要声明类型
num
# 定义只读变量
readonly num
# 清楚变量
unset num
# 使用变量
$num
# 双引号解析变量
"$num"
# 单引号不解析变量
'$num'
# 读取键盘输入，赋值给num
read -p "input value:"num
```

## 变量扩展
```shell
# 变量num不存在时，使用100，不能单独使用
${num:-100}
# 变量num不存在时，将100赋值给num，不能单独使用
${num:=100}
```

## 预定义变量
```shell
# 参数名称
$1 $2 $3
# 参数数量
$#
# 参数内容
$*
# 上一个命令返回状态 0：执行成功 非零0：执行失败
$?
# 执行当前shell的进程名称
$0
# 执行当前shell的进程id
$$
```

 ## ()、{}作用域
 ```shell
value=10
echo "before:$value"
(
    # 修改变量值只在()内有效
    value=100
    echo "()inner $value"
)

echo "()after:$value"
{
    # 修改变量值在{}外依然有效
    value=101
    echo "{}inner $value"
}
echo "{}after:$value"
 ```

## 字符串操作
```shell
# 字符串str的长度
${#str}
# 从索引为3的位置提取到字符串结尾
${str:3}
# 从索引为3的位置提取长度为6个字节
${str:3:6}
# 用new替换字符串中的第一个出现的old
${str/old/new}
# 用new替换字符串中所有的出现的old
${str//old/new}
```

## 文件测试
```shell
# test condition
test -e $file
# [ condition ] 左右两边都要有空格
[ -e $file]

-e # 是否存在
-d # 是目录
-f # 是文件
-r # 可读
-w # 可写
-x # 可执行
-L # 软链接
-c # 是否字符设备
-b # 是否块设备
-s # 文件非空
```

## 字符串测试
```shell
test operator $str
[ operator $str]
-z # 空串
-n # 非空串
= # 两个字符串相等
!= # 两个字符串不相等
```

## 数值测试
```shell
-eq # 数值相等 
-ne # 数值不等
-gt # 大于
-ge # 大于等于
-lt # 小于
-le # 小于等于
```

## 多重判断
```shell
-a # and &&
-o # or  ||
! # 取反
```

## if
```shell
# 格式一
if [条件1]; then
  执行第一段程序
else
  执行第二段程序
fi
# 格式二
if [条件1]; then
  执行第一段程序
elif [条件2]；then
执行第二段程序
else
  执行第三段程序
fi
```

## for
```shell
# 格式一
for fileName in `ls`
do
    执行程序
done

# 格式二
for i in 1 2 3 4
do
    执行程序
done 

# 格式三
# 强制把i变量当做int类型参数运算
declare -i i
for(( i=0; i<5; i++ ))
do
    执行程序
done

# for + if
for fileName in `ls`
do
        if [ -d $fileName ];then
                echo "$fileName is direct"
        elif [ -f $fileName ];then
                echo "$fileName is file"
        fi
done

```