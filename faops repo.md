# 01-help.bats  
## bats代码①:  
```
@test "run faops" {  
    run $BATS_TEST_DIRNAME/../faops help  
    assert_success  
}  
```
## 可以运行的bash代码①:
检测 faops 程序是否成功安装并能够成功执行，成功输出faops具体信息和“ Success ”，失败输出“ Failed ”
```
faops help && echo "Success" || echo "Failed"
```
## bats代码②:  
```
@test "help: contents" {
    run $BATS_TEST_DIRNAME/../faops help
    echo "${output}" | grep "Usage"
    assert_success
}
```
## 可以运行的bash代码②:  
检查 faops help 的输出结果中是否包含 Usage ，包含输出“ faops help output contains 'Usage'”，不包含输出“ faops help output doesn't contains 'Usage'”  
```
faops help | grep "Usage"
if [ $? -eq 0 ]; then  
   echo "faops help output contains 'Usage' "
else
   echo "faops help output doesn't contains 'Usage' "
fi  
```
## bats代码③:  
```
@test "help: lines of contents" {
    run $BATS_TEST_DIRNAME/../faops help
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal "27" "${output}"
}
```
## 可以运行的bash代码③:  
计算 faops help 输出的行数，并判断行数是否等于 27  
```
line_count=$(echo "$(faops help)" | wc -l | xargs echo)
echo "line_count:$line_count"
if [ $line_count -eq 27 ] ;then
   echo "The line number of faops help is 27"
else
   echo "The line number of faops help isn't 27,the actual line number of faops help is $line_count"
fi
```
# 02-count.bats  
## bats代码①:  
```
@test "count: read from file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa | head -n 2"
    assert_equal "#seq${tab}len${tab}A${tab}C${tab}G${tab}T${tab}N" "${lines[0]}"
    assert_equal "read0${tab}359${tab}99${tab}89${tab}92${tab}79${tab}0" "${lines[1]}"
}  
```
## 可以运行的bash代码①:  
计算 ufasta.fa 文件中序列的序列名、序列长度和各碱基（A/C/G/T/N）的数量，只取前两行输出（去掉 head 可以输出全部）
```
cd $HOME/faops/test
faops count ufasta.fa | head -n 2
#或 faops count $HOME/faops/test/ufasta.fa | head -n 2
```
## bats代码②: 
```
@test "count: read from gzipped file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa.gz | head -n 2"
    assert_equal "#seq${tab}len${tab}A${tab}C${tab}G${tab}T${tab}N" "${lines[0]}"
    assert_equal "read0${tab}359${tab}99${tab}89${tab}92${tab}79${tab}0" "${lines[1]}"
}
```
## 可以运行的bash代码②:
计算 ufasta.fa.gz 压缩文件中序列的序列名、序列长度和各碱基（A/C/G/T/N）的数量，与 ① 类似
```
cd $HOME/faops/test
faops count ufasta.fa.gz | head -n 2
#或 faops count $HOME/faops/test/ufasta.fa.gz | head -n 2
```
## bats代码③:
```
@test "count: read from stdin" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops count stdin | head -n 2"
    assert_equal "#seq${tab}len${tab}A${tab}C${tab}G${tab}T${tab}N" "${lines[0]}"
    assert_equal "read0${tab}359${tab}99${tab}89${tab}92${tab}79${tab}0" "${lines[1]}"
}
```
## 可以运行的bash代码③:
先从 ufasta.fa 文件中读取内容传输给 stdin（标准输入），后从 stdin 中读取内容，计算序列的序列名、序列长度和各碱基（A/C/G/T/N）的数量，只取前两行输出。比前面更灵活，可以处理动态生成的数据
```
cd $HOME/faops/test
cat ufasta.fa | faops count stdin | head -n 2
```
## bats代码④:
```
@test "count: lines of result" {
    run $BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 52 "${output}"
}
```
## 可以运行的bash代码④:
计算 faops count ufasta.fa 输出结果的行数  
• 50条序列 + 1个表头 + total = 52行
```
cd $HOME/faops/test
line_count=$(echo "$(faops count ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑤:
```
@test "count: mixture of stdin and actual file" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops count stdin $BATS_TEST_DIRNAME/ufasta.fa"
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 102 "${output}"
}
```
## 可以运行的bash代码⑤:
利用 faops count 同时处理 stdin（标准输入）和原文件，计算两者行数的总和  
• 从stdin读取：50条序列  
• 从原文件读取：50条序列  
• 总行数：50 + 50 + 1个表头 + total = 102行  
```
cd $HOME/faops/test
line_count=$(echo "$(cat ufasta.fa | faops count stdin ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑥:
```
@test "count: sum of sizes" {
    run bash -c "
        $BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa \
            | perl -ne '/^total\t(\d+)/ and print \$1'
    "
    assert_equal 9317 "${output}"
}
```
## 可以运行的bash代码⑥:
计算 ufasta.fa 文件中所有序列长度的总和（count）  
以下为perl语言解释：  
• ^：匹配行首  
• total：匹配文字"total"  
• \t：匹配制表符  
• (\d+)：匹配一个或多个数字，并用括号捕获（保存到$1）  
• print $1：打印正则表达式中第一个括号捕获的内容（即数字部分）
```
cd $HOME/faops/test
total=$(faops count ufasta.fa | perl -ne '/^total\t(\d+)/ and print $1')
echo "total:$total"
```
## bats代码⑦:
```
@test "faCount: without cpg" {
    if ! hash faCount 2>/dev/null ; then
        skip "Can't find faCount"
    fi

    exp=$(faCount $BATS_TEST_DIRNAME/ufasta.fa | perl -anle 'print join qq{\t}, @F[0 .. 6]')
    res=$($BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑦:
对比 faops count 和 facount 的输出结果是否一致  
以下为perl语言解释：  
• -a：自动分割模式，将每行按空白分割到 @F 数组  
• -n：逐行处理输入  
• -l：自动处理换行符  
• @F[0 .. 6]：取数组的前7个元素（第1-7列）  
• join qq{\t}：用制表符连接这些元素  
```
if ! hash faCount 2>/dev/null ; then        
   echo "Can't find faCount"
else
exp=$(faCount ufasta.fa | perl -anle 'print join "\t", @F[0..6]')
res=$(faops count ufasta.fa)
   if [ "$exp" = "$res" ]; then
       echo "The outputs of faops_count and faCount are the same"
   else
       echo "The outputs of faops_count and faCount are different"
   fi
fi
```
# 03-size.bats  
## bats代码①:
```
@test "size: read from file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | head -n 2"
    assert_equal "read0${tab}359" "${lines[0]}"
    assert_equal "read1${tab}106" "${lines[1]}"
}
```
## 可以运行的bash代码①:
计算 ufasta.fa 文件中序列的长度（只输出前两条）  
```
cd $HOME/faops/test
faops size ufasta.fa | head -n 2
```
## bats代码②:
```
@test "size: read from gzipped file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa.gz | head -n 2"
    assert_equal "read0${tab}359" "${lines[0]}"
    assert_equal "read1${tab}106" "${lines[1]}"
}
```
## 可以运行的bash代码②:
计算 ufasta.fa.gz 压缩文件中序列的长度，与①类似  
```
cd $HOME/faops/test
faops size ufasta.fa.gz | head -n 2
```
## bats代码③:
```
@test "size: read from stdin" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops size stdin | head -n 2"
    assert_equal "read0${tab}359" "${lines[0]}"
    assert_equal "read1${tab}106" "${lines[1]}"
}
```
## 可以运行的bash代码③:
先从 ufasta.fa 文件中读取内容传输给 stdin（标准输入），后从 stdin 中读取内容，计算序列的长度  
```
cd $HOME/faops/test
cat ufasta.fa | faops size stdin | head -n 2
```
## bats代码④:
```
@test "size: lines of result" {
    run $BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 50 "${output}"
}
```
## 可以运行的bash代码④:
计算 faops size ufasta.fa 输出结果的行数，与 count 不同的是没有表头和total  
```
cd $HOME/faops/test
line_count=$(echo "$(faops size ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑤:
```
@test "size: mixture of stdin and actual file" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops size stdin $BATS_TEST_DIRNAME/ufasta.fa"
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 100 "${output}"
}
```
## 可以运行的bash代码⑤:
利用 faops size 同时处理 stdin（标准输入）和原文件，计算两者行数的总和  
```
cd $HOME/faops/test
line_count=$(echo "$(cat ufasta.fa | faops size stdin ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑥:
```
@test "size: sum of sizes" {
    run bash -c "
        $BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa \
            | perl -ane '\$c += \$F[1]; END { print qq{\$c\n} }'
    "
    assert_equal 9317 "${output}"
}
```
## 可以运行的bash代码⑥:
计算 ufasta.fa 文件中所有序列长度的总和（size）  
• -a：自动分割模式，将每行按空白分割到 @F 数组  
• -n：逐行处理输入  
• -e：执行后面的 Perl 代码  
• \$c += \$F[1]：累加第二列（序列长度）的值  
• END { print qq{\$c\n} }：处理完所有行后打印累加结果  
```
cd $HOME/faops/test
total=$(faops size ufasta.fa | perl -ane '$c += $F[1]; END { print qq{$c\n} }')
echo "total:$total"
```
# 04-frag.bats
## bats代码①:
```
@test "frag: from first sequence" {
    res=$($BATS_TEST_DIRNAME/../faops frag $BATS_TEST_DIRNAME/ufasta.fa 1 10 stdout | grep -v "^>")
    assert_equal "tCGTTTAACC" "${res}"
}
```
## 可以运行的bash代码①:
`frag`可以提取 ufasta.fa 文件中的序列片段，提取第 1-10 个碱基。`grep -v "^>"`可以删除以“>”开头的行，即描述行  
```
cd $HOME/faops/test
res=$(faops frag ufasta.fa 1 10 stdout | grep -v "^>")
echo "The extracted sequence is:$res"
```
## bats代码②:
```
@test "frag: from specified sequence" {
    res=$($BATS_TEST_DIRNAME/../faops some $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout \
        | $BATS_TEST_DIRNAME/../faops frag stdin 1 10 stdout \
        | grep -v "^>")
    assert_equal "AGCgCcccaa" "${res}"
}
```
## 可以运行的bash代码②:
`some`可以提取指定序列，`<(echo read12)`提供序列名为 read12 的序列，输出到 stdout  
`frag`可以从 stdin 中提取第 1-10 个碱基，输出到 stdout  
```
cd $HOME/faops/test
res=$(faops some ufasta.fa <(echo read12) stdout | faops frag stdin 1 10 stdout | grep -v "^>")
echo "The extracted sequence is:$res"
```
# 05-rc.bats
## bats代码①:
```
@test "rc: output same length" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa)
    res=$($BATS_TEST_DIRNAME/../faops rc -n $BATS_TEST_DIRNAME/ufasta.fa stdout \
        | $BATS_TEST_DIRNAME/../faops size stdin)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
可以验证利用`faops rc -n`得到的反向互补序列与原始序列长度相等
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa)
res=$(faops rc -n ufasta.fa stdout | faops size stdin)
#echo "The length of the original sequence:$exp"
#echo "The length of the reverse complementary sequence:$res"
if [ "$exp" = "$res" ]; then
   echo "The length of the reverse complementary sequence is equal to the original sequence"
else
   echo "The length of the reverse complementary sequence is unequal to the original sequence"
fi
```
## bats代码②:
```
@test "rc: double rc" {
    exp=$($BATS_TEST_DIRNAME/../faops filter $BATS_TEST_DIRNAME/ufasta.fa stdout)
    res=$($BATS_TEST_DIRNAME/../faops rc -n $BATS_TEST_DIRNAME/ufasta.fa stdout \
        | $BATS_TEST_DIRNAME/../faops rc -n stdin stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
验证双重反向互补操作会恢复原始序列（fa文件）  
`faops filter`用来清理数据格式？  
```
cd $HOME/faops/test
exp=$(faops filter ufasta.fa stdout)    
res=$(faops rc -n ufasta.fa stdout | faops rc -n stdin stdout)
if [ "$exp" = "$res" ]; then   
   echo "The length of the double reverse complementary sequence is equal to the original sequence"
else   
   echo "The length of the double reverse complementary sequence is unequal to the original sequence"
fi
```
## bats代码③:
```
@test "rc: double rc (gz)" {
    exp=$($BATS_TEST_DIRNAME/../faops filter $BATS_TEST_DIRNAME/ufasta.fa stdout)
    res=$($BATS_TEST_DIRNAME/../faops rc -n $BATS_TEST_DIRNAME/ufasta.fa.gz stdout \
        | $BATS_TEST_DIRNAME/../faops rc -n stdin stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
验证双重反向互补操作会恢复原始序列（fa.gz文件）  
```
cd $HOME/faops/test
exp=$(faops filter ufasta.fa stdout)    
res=$(faops rc -n ufasta.fa.gz stdout | faops rc -n stdin stdout)
if [ "$exp" = "$res" ]; then      
   echo "The length of the double reverse complementary sequence is equal to the original sequence"
else      
   echo "The length of the double reverse complementary sequence is unequal to the original sequence"
fi
```
## bats代码④:
```
@test "rc: perl regex" {
    paste <($BATS_TEST_DIRNAME/../faops rc -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -v '^>') \
        <($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -v '^>') \
        | perl -ane '
            $F[0] = uc($F[0]);
            $F[1] =~ tr/ACGTacgt/TGCATGCA/;
            $F[1] = reverse($F[1]);
            exit(1) unless $F[0] eq $F[1]
        '
    assert_success
}
```
## 可以运行的bash代码④:
利用perl验证`rc -l`反向互补的正确性  
• `rc -l 0`可以生成反向互补序列，且无长度限制  
• `paste`可以将两个命令的输出按列合并，每列包含反向互补序列 + 制表符 + 原始序列  
• `$F[0] = uc($F[0])`： 将反向互补序列转为大写  
• `$F[1] =~ tr/ACGTacgt/TGCATGCA/`： 对原始序列进行碱基互补  
• `$F[1] = reverse($F[1])`: 反转序列  
• `$F[0] ne $F[1]`: ne 为perl中的不等运算符  
• `$.`表示当前行号  
• 退出状态码1表示失败，0表示成功，`eof`判断是否到达文件末尾  
```
cd $HOME/faops/test
paste \
  <(faops rc -l 0 ufasta.fa stdout | grep -v '^>') \
  <(faops filter -l 0 ufasta.fa stdout | grep -v '^>') \
| perl -ane '
    $F[0] = uc($F[0]);
    $F[1] =~ tr/ACGTacgt/TGCATGCA/;
    $F[1] = reverse($F[1]);
    if ($F[0] ne $F[1]) {
        print "Mismatch at line $.: $F[0] vs $F[1]\n";
        exit(1);
    }
    exit(0) if eof;
'
if [ $? -eq 0 ]; then
    echo "Perl reverse complement algorithm verification successful"
else
    echo "Perl reverse complement algorithm verification failed"
fi
```
## bats代码⑤:
```
@test "rc: with list.file" {
    exp=">RC_read47"
    res=$($BATS_TEST_DIRNAME/../faops rc -l 0 -f <(echo read47) $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>RC_')

    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑤:
利用列表文件对指定序列进行反向互补操作  
• `-f <(echo read47)`只处理序列“read47”
```
cd $HOME/faops/test
exp=">RC_read47"    
res=$(faops rc -l 0 -f <(echo read47) ufasta.fa stdout | grep '^>RC_')
if [ "$exp" = "$res" ];then
   echo "The reverse complementary sequence was successfully generated using the list file"
else
   echo "Failed,exp:$exp;res:$res"
fi
```
# 06-one.bats
## bats代码①:
```
@test "one: inline names" {
    exp=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read12')
    res=$($BATS_TEST_DIRNAME/../faops one -l 0 $BATS_TEST_DIRNAME/ufasta.fa read12 stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
利用`faops one`提取单个指定序列  
• `grep -A 1`可以显示匹配行及其后面一行内容
```
cd $HOME/faops/test
exp=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read12')    
res=$(faops one -l 0 ufasta.fa read12 stdout)
if [ "$exp" = "$res" ];then   
   echo "faops one correctly extracts individual sequences"
else   
   echo "Failed,exp:$exp;res:$res"
fi
```
## bats代码②:
```
@test "faSomeRecords: inline names" {
    if ! hash faSomeRecords 2>/dev/null ; then
        skip "Can't find faSomeRecords"
    fi

    exp=$(faSomeRecords $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops one $BATS_TEST_DIRNAME/ufasta.fa read12 stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
借助faSomeRecords验证`faops one`是否正确提取单个指定序列，与①类似
```
cd $HOME/faops/test
if ! hash faSomeRecords 2>/dev/null ; then           
   echo "Can't find faSomeRecords"
else
exp=$(faSomeRecords ufasta.fa <(echo read12) stdout | grep '^>')
res=$(faops one ufasta.fa read12 stdout | grep '^>')   
   if [ "$exp" = "$res" ]; then       
      echo "The outputs of faops_one and faSomeRecords are the same"   
   else       
      echo "The outputs of faops_one and faSomeRecords are different"   
   fi
fi
```
# 07-some.bats
## bats代码①:
```
@test "some: inline names" {
    exp=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read12')
    res=$($BATS_TEST_DIRNAME/../faops some -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
`faops some`可以用来提取多条指定序列  
将`<(echo read12)`替换为`<(echo -e "read12\nread13")`可以提取多条序列  
```
cd $HOME/faops/test
exp=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read12')    
res=$(faops some -l 0 ufasta.fa <(echo read12) stdout)
if [ "$exp" = "$res" ]; then             
   echo "Faops some correctly extracts sequences"      
else             
   echo "Failed,exp：$exp;res:$res"      
fi
```
## bats代码②:
```
@test "faSomeRecords: inline names" {
    if ! hash faSomeRecords 2>/dev/null ; then
        skip "Can't find faSomeRecords"
    fi

    exp=$(faSomeRecords $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops some $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
借助faSomeRecords验证`faops some`是否正确提取指定序列，与①类似  
```
cd $HOME/faops/test
if ! hash faSomeRecords 2>/dev/null ; then              
   echo "Can't find faSomeRecords"
else
exp=$(faSomeRecords ufasta.fa <(echo read12) stdout | grep '^>')
res=$(faops some ufasta.fa <(echo read12) stdout | grep '^>')    
   if [ "$exp" = "$res" ]; then             
      echo "The outputs of faops_some and faSomeRecords are the same"      
   else             
      echo "The outputs of faops_some and faSomeRecords are different"      
   fi
fi
```
## bats代码③:
```
@test "faSomeRecords: exclude" {
    if ! hash faSomeRecords 2>/dev/null ; then
        skip "Can't find faSomeRecords"
    fi

    exp=$(faSomeRecords -exclude $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops some -i $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
利用`faops some -i`可以排除指定序列，保留剩余序列
```
cd $HOME/faops/test
if ! hash faSomeRecords 2>/dev/null ; then              
   echo "Can't find faSomeRecords"
else
exp=$(faSomeRecords -exclude ufasta.fa <(echo read12) stdout | grep '^>')
res=$(faops some -i ufasta.fa <(echo read12) stdout | grep '^>')    
   if [ "$exp" = "$res" ]; then             
      echo "The outputs of faops_some and faSomeRecords are the same"      
   else             
      echo "The outputs of faops_some and faSomeRecords are different"      
   fi
fi
```
# 08-filter.bats
## bats代码①:
```
@test "filter: as formatter, sequence in one line" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | wc -l | xargs echo)
    assert_equal "$(( exp * 2 ))" "$res"
}
```
## 可以运行的bash代码①:
`faops filter -l 0`可以格式化序列，且每条序列在一行上  
每个格式化后的序列占两行：1行头信息 + 1行序列数据  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | wc -l | xargs echo)    
res=$(faops filter -l 0 ufasta.fa stdout | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
## bats代码②:
```
@test "filter: as formatter, blocked fasta files" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops filter -b $BATS_TEST_DIRNAME/ufasta.fa stdout | wc -l | xargs echo)
    assert_equal "$(( exp * 3 ))" "$res"
}
```
## 可以运行的bash代码②:
`faops filter -b`可以生成分块格式化的fa文件，后每条序列占3行，序列后面有一行空行  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | wc -l | xargs echo)    
res=$(faops filter -b ufasta.fa stdout | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
## bats代码③:
```
@test "filter: as formatter, identical headers" {
    exp=$(grep '^>' $BATS_TEST_DIRNAME/ufasta.fa)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
使用`faops filter`格式化序列后描述行信息不变  
```
cd $HOME/faops/test
exp=$(grep '^>' ufasta.fa)    
res=$(faops filter -l 0 ufasta.fa stdout | grep '^>')
if [ "$exp" = "$res" ]; then                   
   echo "The description line information remains unchanged after formatting the sequence"         
else                   
   echo "The description line information changes after formatting the sequence"         
fi
```
## bats代码④:
```
@test "filter: as formatter, identical sequences" {
    exp=$(grep -v '^>' $BATS_TEST_DIRNAME/ufasta.fa | perl -ne 'chomp; print')
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -v '^>' | perl -ne 'chomp; print')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码④:
使用`faops filter`格式化序列后序列信息不变  
• `chomp`：移除每行的换行符  
• `print`：连续打印（不移除序列字符间的换行）  
• 可以去除描述行，将所有序列数据连接成一个长字符串  
```
cd $HOME/faops/test
exp=$(grep -v '^>' ufasta.fa | perl -ne 'chomp; print')    
res=$(faops filter -l 0 ufasta.fa stdout | grep -v '^>' | perl -ne 'chomp; print')
if [ "$exp" = "$res" ]; then                      
   echo "The sequence information remains unchanged after formatting the sequence"         
else                      
   echo "The sequence information changes after formatting the sequence"         
fi
```
## bats代码⑤:
```
@test "filter: as formatter, identical sequences (gz)" {
    exp=$(grep -v '^>' $BATS_TEST_DIRNAME/ufasta.fa | perl -ne 'chomp; print')
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑤:
使用`faops filter`格式化 fa.gz 压缩文件的序列后序列信息不变  
```
cd $HOME/faops/test
exp=$(grep -v '^>' ufasta.fa | perl -ne 'chomp; print')    
res=$(faops filter -l 0 ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
if [ "$exp" = "$res" ]; then                      
   echo "The sequence information remains unchanged after formatting the sequence"         
else                      
   echo "The sequence information changes after formatting the sequence"         
fi
```
## bats代码⑥:
```
@test "filter: as formatter, identical sequences (gz) with -N" {
    exp=$(grep -v '^>' $BATS_TEST_DIRNAME/ufasta.fa | perl -ne 'chomp; print')
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -N $BATS_TEST_DIRNAME/ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑥:
`faops filter -l 0 -N`中的`-N`参数可以将所有IUPAC模糊碱基代码（如M、R、S、W、Y、K等）转换为标准的N，保持A、G、C、T不变  
```
cd $HOME/faops/test
exp=$(grep -v '^>' ufasta.fa | perl -ne 'chomp; print')    
res=$(faops filter -l 0 -N ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
if [ "$exp" = "$res" ]; then                      
   echo "The sequence information remains unchanged after formatting the sequence"         
else                      
   echo "The sequence information changes after formatting the sequence"         
fi
```
## bats代码⑦:
```
@test "filter: convert IUPAC to N" {
    exp=$(printf ">read\n%s\n" ANNG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -N <(printf ">read\n%s\n" AMRG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑦:
利用`faops filter -l 0 -N`的`-N`参数将所有IUPAC模糊碱基代码（M、R）转换为标准的N  
M=A/C;R=A/G  
`<()`可以创建一个临时文件，并传递给 faops 执行，执行完自动删除  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ANNG)
res=$(faops filter -l 0 -N <(printf ">read\n%s\n" AMRG) stdout)
if [ "$exp" = "$res" ]; then                      
   echo "-N converts successfully"         
else                      
   echo "Failed"         
fi
```
## bats代码⑧:
```
@test "filter: remove dashes" {
    exp=$(printf ">read\n%s\n" ARG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -d <(printf ">read\n%s\n" A-RG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑧:
`faops filter -l 0 -d`的`-d`参数可以用来删除序列中的破折号（-）  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ARG)
res=$(faops filter -l 0 -d <(printf ">read\n%s\n" A-RG) stdout)
if [ "$exp" = "$res" ]; then                         
   echo "'-' has been removed successfully"         
else                         
   echo "Failed"         
fi
```
## bats代码⑨:
```
@test "filter: Upper cases" {
    exp=$(printf ">read\n%s\n" ATCG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -U <(printf ">read\n%s\n" AtcG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑨:
`faops filter -l 0 -U`的`-U`参数可以将序列中的小写字母转换为大写字母  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ATCG)    
res=$(faops filter -l 0 -U <(printf ">read\n%s\n" AtcG) stdout)
if [ "$exp" = "$res" ]; then                         
   echo "-U converts successfully"         
else                         
   echo "Failed"         
fi
```
## bats代码⑩:
```
@test "filter: simplify seq names" {
    exp=$(printf ">read\n%s\n" ANNG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -s <(printf ">read.1\n%s\n" ANNG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑩:
`faops filter -l 0 -s`的`-s`参数可以简化 FASTA 文件的描述行，移除序列名称中的后缀和额外描述  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ANNG)    
res=$(faops filter -l 0 -s <(printf ">read.1\n%s\n" ANNG) stdout)
echo "exp:$exp; res:$res"
```
## bats代码11:
```
@test "filter: fastq to fasta" {
    run $BATS_TEST_DIRNAME/../faops filter $BATS_TEST_DIRNAME/test.seq stdout
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 6 "${output}"
}
```
## 可以运行的bash代码11:
利用`faops filter`可以将 fastq 格式文件转换为 fasta 格式文件  
```
cd $HOME/faops/test
res=$(faops filter test.seq stdout | wc -l | xargs echo)
echo "res:$res"
```
## bats代码12:
```
@test "faFilter: minSize" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -minSize=10 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -a 10 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码12:
利用`faops filter -a 10`可以过滤出长度≥10的序列  
• `-a`：最小序列长度阈值  
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                 
   echo "Can't find faFilter"
else
exp=$(faFilter -minSize=10 ufasta.fa stdout | grep '^>')
res=$(faops filter -a 10 ufasta.fa stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                   
      echo "faops_filter filters successfully"         
   else                   
      echo "Failed"         
   fi
fi
```
## bats代码13:
```
@test "faFilter: maxSize" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -maxSize=50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -a 1 -z 50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码13:
利用`faops filter -z 50`可以过滤出长度≤50的序列  
• `-z`：最大序列长度阈值
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                    
   echo "Can't find faFilter"
else
exp=$(faFilter -maxSize=50 ufasta.fa stdout | grep '^>')
res=$(faops filter -a 1 -z 50 ufasta.fa stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                         
      echo "faops_filter filters successfully"            
   else                         
      echo "Failed"            
   fi
fi
```
## bats代码14:
```
@test "faFilter: minSize maxSize" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -minSize=10 -maxSize=50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -a 10 -z 50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码14:
利用`faops filter -a 10 -z 50`可以过滤出长度在10和50之间的序列  
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                    
   echo "Can't find faFilter"
else
exp=$(faFilter -minSize=10 -maxSize=50 ufasta.fa stdout | grep '^>')
res=$(faops filter -a 10 -z 50 ufasta.fa stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                         
      echo "faops_filter filters successfully"            
   else                         
      echo "Failed"            
   fi
fi
```
## bats代码15:
```
@test "faFilter: uniq" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -uniq <(cat $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -u -a 1 <(cat $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa) stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码15:
利用`faops filter -u`可以去除序列中的重复序列  
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                    
   echo "Can't find faFilter"
else
exp=$(faFilter -uniq <(cat ufasta.fa ufasta.fa) stdout | grep '^>')
res=$(faops filter -u -a 1 <(cat ufasta.fa ufasta.fa) stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                         
      echo "faops_filter filters successfully"            
   else                         
      echo "Failed"            
   fi
fi
```
