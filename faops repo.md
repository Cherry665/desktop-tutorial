# 01-help.bats  
## bats代码①:  
```
@test "run faops" {  
    run $BATS_TEST_DIRNAME/../faops help  
    assert_success  
}  
```
## 可以运行的bash代码①:
检测faops程序是否成功安装并能够成功执行，成功输出faops具体信息和“Success”，失败输出“Failed”
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
检查faops help的输出结果中是否包含Usage，包含输出“faops help output contains 'Usage'”，不包含输出“faops help output doesn't contains 'Usage'”  
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
计算faops help输出的行数，并判断行数是否等于27  
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
