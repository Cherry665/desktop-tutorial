# 01-help.bats  
  bats代码:  
  @test "run faops" {  
    run $BATS_TEST_DIRNAME/../faops help  
    assert_success  
}  
