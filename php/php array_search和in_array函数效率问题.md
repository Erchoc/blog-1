### 问题
array_search 查找数组中的元素的 key 时，效率随着数组变大，耗时增加。特别是多次查找大数组时，非常耗时。在函数 in_array 也有这个问题。

### 解决办法
采用 array_flip 翻转后，用 isset 代替 in_array 函数，用 $array[key] 替代 array_search, 这样能解决大数组超时耗时问题.

### 注意：
1、这种优化只适用于无重复数据的大数组且多次查找时有效，原因见下面。
2、array_search、in_array、array_flip 原理都是数组遍历。array_search、in_array是发现便终止，array_flip遍历整个数组
3、因array_flip遍历整个数组，相比array_search和in_array，array_flip更耗时，只有多次使用array_search和in_array时才能显示出array_flip效果
4、isset 走的 hash 表,查找时间复杂度为 O(1),更高效。
5、使用场景不多，基本用处不大  只是提供一种思路


实例：
```
<?php
$array = array();

for ($i=0; $i<200000; $i++){
    ##随机字符串
    $array[$i] = get_rand().$i;
}

$str = $array[150000];
$time1 = microtime(true);
array_search($str, $array);
$time2 = microtime(true);
echo '原始方法:'.($time2-$time1)."\n";

$time3 =  microtime(true);
$new_array = array_flip($array);
isset($new_array[$str]);
$time4 = microtime(true);
echo '新方法:'.($time4-$time3);


$array = array();

for ($i=0; $i<200000; $i++){
    ##随机字符串
    $array[$i] = get_rand().$i;
}

$str = $array[199999];
$time1 = microtime(true);
for ($i=0; $i<5000; $i++){
    array_search($str, $array);
}
$time2 = microtime(true);

echo '原始方法:'.($time2-$time1)."\n";

$time3 =  microtime(true);
$new_array = array_flip($array);
for ($i=0; $i<5000; $i++){
    isset($new_array[$str]);
}
$time4 = microtime(true);

echo '新方法:'.($time4-$time3);
```

转自： https://learnku.com/laravel/t/34826