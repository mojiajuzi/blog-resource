---
title: "算法-二分查找法"
date: 2018-05-29T23:38:46+08:00
draft: true
---


 前提，被查找的列表必须是有序的,对于二分查找而言，其时间的复杂度为对数时间(`O(log n)`)

```php
<?php
function binary_search(array $list, int $item):int
{
    $low = 0;
    $high = count($list) - 1;

    // 判断数组是否为空
    if($high < 0)
        return -1;

    // 只有当范围缩小到一位时才停止循环
    while ($low <= $high) {

        $mid = floor(($low + $high) / 2); //获取中间值，使用向下取整

        $guess = $list[$mid];

        if($guess == $item)
            return $mid;
        
        if($guess > $item)
            $high = $mid - 1;

        if($guess < $item)
            $low = $mid + 1;
    }
    return -1;
}

$a = [1, 3,5,7,9];

echo binary_search($a, 20);
```