---
title: awk、sed、grep
updated: 2025-04-15T09:24:03
created: 2025-04-14T22:09:52
---

grep擅长查找功能，sed擅长取行和替换，awk擅长取列

# 1.1 grep
- grep: 过滤
<span class="mark">grep \[OPTIONS\] PATTERN \[FILE...\]  
--color=auto 对匹配到的文本着色显示  
-v 显示不被pattern匹配到的行  
-i 忽略字符大小写  
-n 显示匹配的行号  
-c 统计匹配的行数  
-o 仅显示匹配到的字符串  
-q 静默模式，不输出任何信息  
-A \# after, 后#行  
-B \# before, 前#行  
-C \# context, 前后各#行  
-e 实现多个选项间的逻辑or关系  
grep –e ‘cat ’ -e ‘dog’ file  
-w 匹配整个单词  
-E 使用ERE,相当于egrep  
-F 相当于fgrep，不支持正则表达式</span>

==eg: 包含root: grep -n root==

# 1.2 sed
- sed: 取行进行 打印、替换
<span class="mark">sed \[option\]... 'script' inputfile  
选项  
-n 不输出模式空间内容到屏幕，即不自动打印  
-e 多点编辑  
-f /PATH/SCRIPT_FILE: 从指定文件中读取编辑脚本  
-r 支持使用扩展正则表达式  
-i 直接编辑文件  
-i.bak 备份文件并原处编辑  
script 地址定界  
不给地址：对全文进行处理  
单地址：  
\#: 指定的行，\$：最后一行  
/pattern/：被此处模式所能够匹配到的每一行  
地址范围：  
\#,#  
\#,+#  
/pat1/,/pat2/  
\`#,/pat1/  
~：步进  
1~2 奇数行  
2~2 偶数行  
编辑命令：  
d 删除模式空间匹配的行，并立即启用下一轮循环  
p 打印当前模式空间内容，追加到默认输出之后  
a \[\\text1 在指定行后面追加文本,支持使用\n实现多行追加  
i \[\\text 在行前面插入文本  
c \[\\text 替换行为单行或多行文本  
w /path/somefile 保存模式匹配的行至指定文件  
r /path/somefile 读取指定文件的文本至模式空间中匹配到的行后  
= 为模式空间中的行打印行号  
! 模式空间中匹配行取反处理  
s///：查找替换,支持使用其它分隔符，s@@@，s###  
替换标记：  
g 行内全局替换  
p 显示替换成功的行  
w /PATH/TO/SOMEFILE 将替换成功的行保存至文件中</span>

eg: 打印第2行: sed -n 2p passwd

# 1.3 awk
- awk: 取列进行 打印
<span class="mark">\[root@debugresetreset54142x1 ~\]# awk --help  
Usage: awk \[POSIX or GNU style options\] -f  
progfile \[--\] file ...  
Usage: awk \[POSIX or GNU style options\]  
\[--\] 'program' file ...  
POSIX options: GNU long options: (standard)  
-f progfile --file=progfile  
-F fs --field-separator=fs  
-v var=val --assign=var=val  
Short options: GNU long options: (extensions)  
-b --characters-as-bytes  
-c --traditional  
-C --copyright  
-d\[file\] --dump-variables\[=file\]  
-e 'program-text' --source='program-text'  
-E file --exec=file  
-g --gen-pot  
-h --help  
-L \[fatal\] --lint\[=fatal\]  
-n --non-decimal-data  
-N --use-lc-numeric  
-O --optimize  
-p\[file\] --profile\[=file\]  
-P --posix  
-r --re-interval  
-S --sandbox  
-t --lint-old  
-V --version</span>

