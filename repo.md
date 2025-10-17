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
