

| \[`-aFILE`\] | 如果`FILE`存在则为真。 |
| :--- | :--- |
| \[`-bFILE`\] | 如果`FILE`存在且是一个块特殊文件则为真。 |
| \[`-cFILE`\] | 如果`FILE`存在且是一个字特殊文件则为真。 |
| \[`-dFILE`\] | 如果`FILE`存在且是一个目录则为真。 |
| \[`-eFILE`\] | 如果`FILE`存在则为真。 |
| \[`-fFILE`\] | 如果`FILE`存在且是一个普通文件则为真。 |
| \[`-gFILE`\] | 如果`FILE`存在且已经设置了SGID则为真。 |
| \[`-hFILE`\] | 如果`FILE`存在且是一个符号连接则为真。 |
| \[`-kFILE`\] | 如果`FILE`存在且已经设置了粘制位则为真。 |
| \[`-pFILE`\] | 如果`FILE`存在且是一个名字管道\(F如果O\)则为真。 |
| \[`-rFILE`\] | 如果`FILE`存在且是可读的则为真。 |
| \[`-sFILE`\] | 如果`FILE`存在且大小不为0则为真。 |
| \[`-tFD`\] | 如果文件描述符`FD`打开且指向一个终端则为真。 |
| \[`-uFILE`\] | 如果`FILE`存在且设置了SUID \(set user ID\)则为真。 |
| \[`-wFILE`\] | 如果`FILE`如果 FILE 存在且是可写的则为真。 |
| \[`-xFILE`\] | 如果`FILE`存在且是可执行的则为真。 |
| \[`-OFILE`\] | 如果`FILE`存在且属有效用户ID则为真。 |
| \[`-GFILE`\] | 如果`FILE`存在且属有效用户组则为真。 |
| \[`-LFILE`\] | 如果`FILE`存在且是一个符号连接则为真。 |
| \[`-NFILE`\] | 如果`FILE`存在 and has been mod如果ied since it was last read则为真。 |
| \[`-SFILE`\] | 如果`FILE`存在且是一个套接字则为真。 |
| \[`FILE1-ntFILE2`\] | 如果`FILE1`has been changed more recently than`FILE2`, or 如果`FILE1`exists and`FILE2`does not则为真。 |
| \[`FILE1-otFILE2`\] | 如果`FILE1`比`FILE2`要老, 或者`FILE2`存在且`FILE1`不存在则为真。 |
| \[`FILE1-efFILE2`\] | 如果`FILE1`和`FILE2`指向相同的设备和节点号则为真。 |
| \[`-o`OPTIONNAME \] | 如果 shell选项 “OPTIONNAME” 开启则为真。 |
| `[ -z`STRING \] | “STRING” 的长度为零则为真。 |
| `[ -n`STRING \] or \[ STRING \] | “STRING” 的长度为非零 non-zero则为真。 |
| \[ STRING1 == STRING2 \] | 如果2个字符串相同。 “=” may be used instead of “==” for strict POSIX compliance则为真。 |
| \[ STRING1 != STRING2 \] | 如果字符串不相等则为真。 |
| \[ STRING1 &lt; STRING2 \] | 如果 “STRING1” sorts before “STRING2” lexicographically in the current locale则为真。 |
| \[ STRING1 &gt; STRING2 \] | 如果 “STRING1” sorts after “STRING2” lexicographically in the current locale则为真。 |
| \[ ARG1 OP ARG2 \] | “OP” is one of`-eq`,`-ne`,`-lt`,`-le`,`-gt`or`-ge`. These arithmetic binary operators return true if “ARG1” is equal to, not equal to, less than, less than or equal to, greater than, or greater than or equal to “ARG2”, respectively. “ARG1” and “ARG2” are integers. |













  


