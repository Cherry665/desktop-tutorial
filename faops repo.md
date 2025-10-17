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
计算 ufasta.fa 文件中所有序列长度的总和  
以下为perl语言解释：  
• ^：匹配行首  
• total：匹配文字"total"  
• \t：匹配制表符  
• (\d+)：匹配一个或多个数字，并用括号捕获（保存到$1）  
• print $1：打印正则表达式中第一个括号捕获的内容（即数字部分）
```
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
