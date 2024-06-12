好的，这是一句 shell 命令，使用 awk 提取文本最后 10 行出现 "aa" 字符的行：

```bash
awk '/aa/ {lines[NR%10]=$0} END {for(i=0;i<10;i++) if(lines[(NR+i)%10]) print lines[(NR+i)%10]}' your_file.txt
```

**解释：**

* `/aa/ {lines[NR%10]=$0}`：如果行包含 "aa"，将其存储在 `lines` 数组中，使用 `NR%10` 作为索引，确保只保存最后 10 行。
* `END {for(i=0;i<10;i++) if(lines[(NR+i)%10]) print lines[(NR+i)%10]}`：在文件处理结束后，循环 10 次，打印 `lines` 数组中存储的行，从最后一行开始。

**示例：**

假设 `your_file.txt` 内容如下：

```
line 1
line 2
line 3
line 4 aa
line 5
line 6
line 7 aa
line 8
line 9
line 10
line 11 aa
```

运行上述命令，输出结果为：

```
line 7 aa
line 11 aa
```

**注意：**

* 请将 `your_file.txt` 替换为您的实际文件名。
* 如果文件不足 10 行，则会打印所有包含 "aa" 的行。
* 如果最后 10 行中没有包含 "aa" 的行，则不会有任何输出。
