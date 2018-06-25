---
title: "基础算法-快速排序"
date: 2018-06-21T22:40:41+08:00
draft: true
---

递归是快速排序的基础，因此想要了解快速排序，就必须得知道递归，对于递归来说有两个重要的概念

1. 基线条件，函数结束调用自身的条件

1. 递归条件，函数调用自身的条件

一个递归函数，必须包含基线条件，否则递归将无法结束


举例说明，假设需要实现对一个元素为数字类型的数组进行求和，那么我们首先需要做的就是找出其基线条件和递归条件

- 基线条件，数组只包含一个元素或者数组为空

- 递归条件，数组元素个数大于一个

因此实现方式如下:

```php
<?php
function sum(array $arr): int {
    $arrCount = count($arr);

    // 基线条件
    if($arrCount == 0)
        return 0;

    if($arrCount == 1)
        return array_shift($arr);

    // 递归条件
    return array_shift($arr) + sum($arr);
}
```

那么考虑一下如何对一个数组进行排序,由于之前已经已经使用选择排序对数组进行排序，其时间复杂度为`O(n^2)`

因此这里对于一个数组使用快速排序的方式进行排序，首先介绍已给快速排序的步骤

1. 找出基准值,基准值就是从数组中找出一个元素作为基准

1. 分区， 分区就是以基准值作为参考，将素组分为三个区，小于等于基准值的元素集合， 基准值， 大于基准值的集合

1. 对分区进行快速排序，使用递归的方式对所分的区进行递归排序

因此，我们需要找出快速排序的基线条件以及递归条件,结束递归的条件就是数组为空，或者数组只有一个元素，那么这个时候，该数组是不需要进行排序的

```php
<?php

function fastSort(array $arr):array{

    // 基线条件
    if(count($arr) < 2)
        return $arr;

    // 基准值
    $pivot = $arr[0];

    $lessArr = [];
    $greeterArr = [];

    foreach ($arr as $key => $value) {

         if($key == 0)
            continue;
        
         // 分区
         if($value <= $pivot){

             $lessArr[] = $value;

         }else{
             
             $greeterArr[] = $value;
         }
    }

    // 递归条件
    return array_merge(fastSort($lessArr), [$pivot], fastSort($greeterArr));

}
```

