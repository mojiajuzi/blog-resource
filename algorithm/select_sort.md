---
title: "基础算法-选择排序"
date: 2018-06-01T22:51:07+08:00
draft: true
---


1. 原理

    选择排序就是将一个有序的元素集合，将其第一个元素于后面的元素进行比对，
    找出符合条件的的元素，并将符合条件的元素从该集合中剔除，存储到新的集合中
    重复执行以上操作，直到集合中的元素全部转移到新的集合中



1. 时间复杂度

    对于选择排序来说其时间复杂度用大O表示法来表示为`O(n^2)`,
    虽然需要比对的元素逐渐减少，但是平均每一次需要比对的元素个数为`n/2`
    总共需要比对的次数为`n`,因此时间复杂度为`O(n * 1/2 * n)`,考虑到大O表示法中，忽略常数项，
    因此选择排序的时间复杂度为`O(n ^ 2)`

1. 代码示例

    ```php
        <?php

        /**
        * 获取数组中的最小元素
        */
        function getMinItem(array $list):int{

            // 由于php没有没有Python中类数以arr.pop之类的函数，因此需要将数组的第一个元素作为最小索引
            $i = 0;

            foreach($list as $k => $v){

                // 将第一个元素的索引作为最小索引
                if($i == 0){
                    $minKey = $k;
                    $i++;

                    continue;
                }

                // 逐个两两比对，并将较小的元素索引作为最小元素的索引
                if($v <= $list[$minKey])
                    $minKey = $k;
            }
            return $minKey;

        }

        function orderItem(array $list):array{
            $mewArr=  [];

            $listLength = count($list); //计算总的元素个数

            for ($i=0; $i < $listLength; $i++) { // 循环次数为元素的个数，因为每一次只能获取一个最小元素的索引
                $minKey = getMinItem($list); // 每一次从数组中获取最小元素的索引
                array_push($mewArr, $list[$minKey]); // 将获取到的元素添加到新数组中
                unset($list[$minKey]); //从旧有的元素中删除掉当次比对所获取的最小元素
            }
            return $mewArr;
        }

        $a = [1,4,6,5,23,2];

        var_dump(orderItem($a));
    ```

    以上会打印出
    
    ```
        array(6) {
            [0]=>
            int(1)
            [1]=>
            int(2)
            [2]=>
            int(4)
            [3]=>
            int(5)
            [4]=>
            int(6)
            [5]=>
            int(23)
        }
    ```