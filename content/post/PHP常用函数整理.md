---
title: "PHP常用函数整理"
date: 2024-02-18T20:46:01+08:00
draft: false
categories: ["PHP"]
tags: ["PHP"]
keywords: ["PHP"]
---


# PHP常用函数整理

## 字符串 
* strlen() 获取字符串长度。（一个汉字长度为3个字符，一个英文字母长度为1个字符） 
* 
* strpos() 在字符串内查找一个字符或一段指定的文本，返回第一次出现的位置或者false
* stripos() 同上，但不区分大小写
* strrpos() 同上上，返回最后一次出现的位置或false
* strripos() 同上，但不区分大小写
* 
* explode() 字符串打散为数组
* implode() 数组拼接成字符串

* strtoupper() 把字符串转换为大写
* strtolower() 把字符串转换为大写
* ucfirst() 把单词的首字母转换成大写
* lcfirst() 把单词的首字母转换为小写
* ucwords() 把字符串中每个单词的首字母转换为大写
* 
* str_replace 子字符串替换
* str_ireplace  str_replace 的忽略大小写版本
* strrev 反转字符串
* trim  去除字符串首尾处的空白字符（或者其他字符）
* rtrim 删除字符串末端的空白字符（或者其他字符）
* substr 截取字符串的一部分，返回字符串的子串
* substr 截取字符串的一部分（中文，需要安装扩展mbstring），返回字符串的子串

[更多请参考官方文档](https://www.php.net/manual/zh/book.strings.php)

## 数组
* array() 创建数组
* count() 返回数组中元素的数量 sizeof() 同样效果
* array_push() 将一个或多个单元压入数组的末尾（入栈）
* array_pop() 弹出数组最后一个单元（出栈）
* array_unshift() 在数组开头插入一个或多个单元
* array_shift() 将数组开头的单元移出数组
* shuffle() 打乱数组
* reset — 将数组的内部指针指向第一个单元
* end — 将数组的内部指针指向最后一个单元
* current — 返回数组中的当前值
* list — 把数组中的值赋给一组变量
*
* array_slice() 从数组中取出一段
* array_splice() 去掉数组中的某一部分并用其它值取代
* array_merge() 合并一个或多个数组
* array_reverse() 返回单元顺序相反的数组
* array_reduce() 用回调函数迭代地将数组简化为单一的值

*
* in_array() 检查数组中是否存在某个值
* array_key_exists — 检查数组里是否有指定的键名或索引,别名key_exists()
* array_keys() 返回数组中部分的或所有的键名
* array_values() 返回数组中所有的值
* array_search() 在数组中搜索给定的值，如果成功则返回首个相应的键名
* array_unique() 移除数组中重复的值
* 
* max() 最大值
* min() 最小值
* array_sum() 对数组中所有值求和  

*
* sort() 对数组升序排序
* sort() 对数组升序排序
* rsort() 对数组降序排序
* krsort() 对数组按照键名逆向排序
* ksort() 对数组根据键名升序排序
* array_multisort() 对多个数组或多维数组进行排序
* natcasesort() 用“自然排序”算法对数组进行不区分大小写字母的排序
* natsort() 用“自然排序”算法对数组排序
* uasort() 使用用户定义的比较函数对数组进行排序并保持索引关联
* uksort() 使用用户自定义的比较函数对数组中的键名进行排序
* usort() 使用用户自定义的比较函数对数组中的值进行排序
  
[更多请参考官方文档](https://www.php.net/manual/zh/book.array.php)

## 时间日期
* time() 返回当前的 Unix 时间戳
* date() 格式化 Unix 时间戳
* gmdate() 格式化 GMT/UTC 日期／时间
* strtotime() 将任何英文文本日期时间描述解析为 Unix 时间戳
* mktime() 取得一个日期的 Unix 时间戳
* microtime() 返回当前 Unix 时间戳和微秒数
* date_create() 创建一个时间日期对象
* date_format() 将日期时间格式转化为制定字符串格式
* date_diff() 计算两个日期的差
* date_default_timezone_get() 取得脚本中所有日期/时间函数所使用的默认时区
* date_default_timezone_set() 设置脚本中所有日期/时间函数使用的默认时区
  
[更多请参考官方文档](https://www.php.net/manual/zh/book.datetime.php)  
[推荐包](https://carbon.nesbot.com/docs/)

## 数学扩展
* floor() 舍去法取整
* round() 对浮点数进行四舍五入
* ceil() 进一法取整
* abs() 绝对值

[更多请参考官方文档](https://www.php.net/manual/zh/book.math.php)  