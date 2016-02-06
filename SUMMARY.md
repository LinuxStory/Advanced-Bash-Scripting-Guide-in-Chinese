# Summary

* [第一部分 初见shell](source/part1/part1.md)
	* [1. 为什么使用shell编程](source/part1/01_shell_programming.md)
	* [2. 和Sha-Bang(#!)一起出发](source/part1/02_starting_off_with_a_sha_bang.md)
		* [2.1 调用一个脚本](source/part1/02_1_invoking_the_script.md)
		* [2.2 小试牛刀](source/part1/02_2_preliminary_exercises.md)
* [第二部分 shell基础](source/part2/part2.md)
	* [3. 特殊字符](source/part2/03_special_characters.md)
	* [4. 变量与参数](source/part2/04_introduction_to_variables_and_parameters.md)
		* [4.1 变量替换](source/part2/04_1_variable_substitution.md) 
		* [4.2 变量赋值](source/part2/04_2_variable_assignment.md)
		* [4.3 Bash变量是弱类型的](source/part2/04_3_bash_variables_are_untyped.md)
		* [4.4 特殊的变量类型](source/part2/04_4_special_variable_types.md)
	* [5. 引用](source/part2/05_quoting.md)
		* [5.1 引用变量](source/part2/05_1_quoting_variables.md) 
		* [5.2 转义](source/part2/05_2_escaping.md)
	* [6. 退出与退出状态](source/part2/06_exit_and_exit_status.md)
	* [7. 测试](source/part2/07_tests.md)
		* 7.1 测试结构
		* 7.2 文件测试操作
		* 7.3 其他比较操作
		* 7.4 嵌套 `if/then` 条件测试
		* 7.5 牛刀小试
	* 8\. 运算符和相关话题
* [第三部分 shell进阶](source/part3/part3.md)
	* 9\. 换个角度看变量
		* 9.1 内部变量
		* 9.2 指定变量属性：`decalre` 或 `typeset`
		* 9.3 `$RANDOM`：随机产生整数
	* [10. 变量处理](source/part3/10_manipulating_variables.md)
		* [10.1 字符串处理](source/part3/10_1_manipulating_strings.md)
			* [10.1.1 使用 `awk` 处理字符串](source/part3/10_1_1_manipulating_strings_using_awk.md)
			* [10.1.2 参考资料](source/part3/10_1_2_further_reference.md)
		* [10.2 参数替换](source/part3/10_2_parameter_substitution.md)
	* [11. 循环与分支](source/part3/11_loops_and_branches.md)
		* [11.1 循环](source/part3/11_1_loops.md)
		* [11.2 嵌套循环](source/part3/11_2_nested_loops.md)
		* [11.3 循环控制](source/part3/11_3_loop_control.md)
		* [11.4 测试与分支](source/part3/11_4_testing_and_branching.md)
	* [12. 命令替换](source/part3/12_command_substitution.md)
	* [13. 算术扩展](source/part3/13_arithmetic_expansion.md)
	* [14. 休息时间](source/part3/14_recess_time.md)
* [第五部分 进阶话题](source/part5/part5.md)
	* [19. 嵌入文档](source/part5/19_here_documents.md)
	* [20. I/O 重定向](source/part5/20_io_redirection.md)
		* [20.1 使用 exec](source/part5/20_1_use_exec.md)
		* [20.2 重定向代码块](source/part5/20_2_redirecting_code_blocks.md)
		* [20.3 应用程序](source/part5/20_3_applications.md)
	* [25. 别名](source/part5/25_aliases.md)
