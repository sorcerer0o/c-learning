// 数组和指针
数组名是数组首元素的地址
arr[2] 等价于*(arr + 2);

// 数组指针和指针数组
直接看后缀，后缀是什么就是什么；
* 和 [] 谁的优先级高，就确认它是什么;
数组指针，是一个指针，它是一个指向数组的指针；
指针数组是一个数组，它是一个元素都是指针变量的数组；
数组 *:数组指针；
int[5] * :这种指针的偏倚量不一样
matrix[2][3]:  等价与 *(*(matrix + 2) + 3);
//const pointer & pointer to constant
cost 右边的内容，const右边修饰谁，谁不能操作；
constant *p; *p不能动
* sontant p; p不能动
constant * constant p: *p和p都不能动;
