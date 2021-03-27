# leetcode两数之和变种（找出所有满足总和的两个数）

偶尔看到leetcode 的两数之和，但是之前遇到过两数之和的变种，之前一开始想不出来，后面看了别人的题解才想到解法，这里记录一下。


题目描述：

原leetcode题目描述
>给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
你可以按任意顺序返回答案。
示例1：
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
原leectcode题目链接：[https://leetcode-cn.com/problems/two-sum/description/](https://leetcode-cn.com/problems/two-sum/)

这里我们把每种输入只会对应一个答案这个假设去掉，既该数组内可能会有多个两个数的组合满足，输出所有满足条件的两数组合。


思路1：

直接暴力解法，两个for 循环遍历，时间复杂度为 O(n^2)，空间复杂度 O(1)  

思路2：

首先用一个map存储原数组的，key 为数组的值，value为该值出现的次数，然后遍历原数组，依次使用 目标sum减去当前遍历的值得到值结果，根据该值去map中查找。如果匹配，则输出，并且次数减一，如果不满足，则继续遍历，直到遍历完，就可以得到结果。时间复杂度 O(n)  ,空间复杂度 O(n)。(不过这种思路还没考虑到数组有序的情况，有序的情况应该能再减少时间或空间复杂度，但是我暂时没有想出来)


这里简单写一下第二种的实现 java 代码：

```java
// 20210321 20:38 开始
/**
 * 获取两数之和的所有结果
 *
 * @author zhangcanlong
 * @date 2021-03-21 20:38
 */
public Map<Integer, Integer> getAllTwoSumResult(int[] ints, int sum) {
    if (ints == null || ints.length <= 1) {
        return new HashMap<>(0);
    }
    Map<Integer, Integer> resultMap = new HashMap<>();
    Map<Integer, Integer> tempIntsMap = new HashMap<>();
    for (int i = 0; i < ints.length - 1; ++i) {
        int tempInt = tempIntsMap.getOrDefault(ints[i], 0);
        tempIntsMap.put(ints[i], ++tempInt);
    }

    // 注意这里：再次遍历原数组的一半(1/2之前，不能大于1/2)就可以输出满足结果
    for (int i = 0; i < (ints.length >> 1); ++i) {
        int suitSumInt = sum - ints[i];
        if (tempIntsMap.containsKey(suitSumInt)) {
            int suitSumIntCnt = tempIntsMap.get(suitSumInt);
            // 注意当前值的次数大于1，才算一个结果，否则不是
            if (suitSumIntCnt >= 1) {
                resultMap.put(ints[i], suitSumInt);
                tempIntsMap.put(suitSumInt, --suitSumIntCnt);
            }
        }
    }
    return resultMap;
}
// 2021-03-21 20:51 完成



```

注意：一开始写我是对着文本编辑器写出来的，大概写了10分钟，没用idea，发现map.containsKey 方法都记得不是很清楚😂，一开始写成map.contains，然后当时写着的时候发现思路没有考虑到一些东西，结合运行的时候，有以下要注意的：
1. 使用次数的如果为0了，则不进入统计
2. 如果遍历完整的数组获取到的结果都会有个重复值的组合，结合原数组是有序的情况下，应该遍历到 (length>>1)-1 的时候就停止。

当前的代码结果是修正好的结果了，还是要手写过一遍才记忆深刻🤣，一开始的时候，看的第一眼，我还以为这道算法会很难，如果一开始知道了使用map 和 利用 sum 的结果导推回去，就可能要思考很久。算作简单记录一下自己的做的一道算法题吧，太久没写算法了，今天坐着需求的时候，偶尔心血来潮想写一下算法练练手🙃。


>文章杂谈
>太久没练算法了，感谢自己的算法能力已经生疏很多了。另外上周周末加班了两天，真 997 😪，导致一篇技术文在写但是一直没有完成，工作真的有点忙，预计明天会发出来


![CrudBoys](https://img-blog.csdnimg.cn/20201114143851114.png)