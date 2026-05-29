---
title: 面试合集：HOT-100指南
date: 2026-05-26 15:25:39
tags:
---

{% note color:blue 前言 最近一直在忙于找暑假实习，但是一直苦于没有个持续、有效的刷题思路；偶然间在B站看到了个博主[刷题攻略](https://www.bilibili.com/video/BV1G4oSBMETC)，遂决定参考他的方案学习，并写了这篇博文作为学习笔记。%}

## 思路总览

### 1. 两数之和

> 给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。
>
> 你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。你可以按任意顺序返回答案。

这题的要点是最后<u>返回目标原始的索引数组</u>，不难想出一般解法是`O(N^2)`遍历；我想到的优化方案有：

* 预排序然后双指针从首/尾便利，但这种方法下寻找原始索引映射过于繁琐，故放弃；
* 采用哈希表缓存，空间换时间。这也是**推荐解法**，在JS实现中需要注意，最好使用`new Map()`代替`{}`，一是避免`Object`原形链的影响，二是`Map`有专属的哈希优化；

> **原形链查找顺序**：先查找自身自有属性（`Object.prototype.hasOwnProperty`）→沿着隐式原型`__proto__`构成的原型链逐层向上追溯，依次访问各级原型对象→直至原型链顶端`null`；未找到返回`undefined`；

### 49. 字母异位词分组

> 给你一个字符串数组，请你将「字母异位词」组合在一起。可以按任意顺序返回结果列表。

这题的要点是<u>字母类型有限<sup>26</sup></u>，这里先梳理一下该题型的常规解题思路：

* 对每个`word`做排序处理，然后用排序结果作为键值，所用时间复杂度为`O(L*C*log(C))`；
* 在不考虑空间复杂度的情况下，可以采用先计数，后生成键值的方法来维护哈希表；

我最初思考解法时，忽略了字符重复的场景，原本计划通过`BitMap`生成唯一键值。后续发现，即便存在字符重复问题，该思路依然可行——只需将原生 BitMap 升级为**带计数能力的 BitArray**，即可完美适配重复字符场景。这种实现方式下，最终`BigInt`键值的二进制长度公式为：`TYPE * Math.ceil(Math.log2(COUNT))`。

不过该方案存在明显短板：`BigInt`与`Number`类型转换、计数片段截取、`BigInt`位运算等操作，都会产生额外的性能与资源损耗。因此，想要进一步压缩空间、优化性能，可替换为`ArrayBuffer + DataView`实现轻量化计数数组。

以上都是工程实现层面的优化方案，这里不展开赘述。接下来，重点分享我解这道题时，新学到的几个JS知识点。

#### a. 字符串

字符集

码点`code point`

ascii(实际128, 编码1)

中文相关：GB2312、GBK(2b)、CJK

unicode：统一所以🤗

编码

UTF-32：统一长度4byte

UTF-8：可变长度编码，兼容ascii

Intl.Segment颗粒度（grahpeme、word、sentense）

码点与字形簇不是一一对应关系

#### b. TypedArray



### 128. 最长连续序列

> 给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
>
> 请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

最初我考虑使用并查集来解决问题，但随后发现需要额外维护一个缓存来处理边界情况。既然要额外维护边界缓存，那么计数逻辑其实也可以直接整合到该缓存中。最终得到了如下解法：

```ts
function longestConsecutive(nums: number[]): number {
  const left = new Map<number, number>();
  const right = new Map<number, number>();

  let max = 0;
  const uniqueNums = [...new Set(nums)];

  for (let i = 0; i < uniqueNums.length; i++) {
    const num = uniqueNums[i];

    const lCount = left.get(num - 1) ?? 0;
    const rCount = right.get(num + 1) ?? 0;
    left.delete(num - 1);
    right.delete(num + 1);

    const mergeCount = 1 + lCount + rCount;
    left.set(num + rCount, mergeCount);
    right.set(num - lCount, mergeCount);

    if (mergeCount > max) {
      max = mergeCount;
    }
  }
  return max;
}
```

该方法虽然复杂度为`O(n)`，但由于每一步循环中都包含大量`Map.prototype.set`操作，实际运行耗时较高。本题的最优解法如下：

* 利用哈希表的`O(1)`查询能力，**只从连续序列的起点开始扩展**，保证每个数字最多被访问一次，从而把时间复杂度优化到`O(n)`。

### 283. 移动零

> 给定一个数组`nums`，编写一个函数将所有`0`移动到数组的末尾，同时保持非零元素的相对顺序。
>
> **请注意** ，必须在不复制数组的情况下原地对数组进行操作。

我的解法是预处理值为`0`的目标索引，然后用队列+单指针实现非`0`值的顺序移动；这道题的最佳解法是采用双指针在遍历时就解决：

* 左指针指向**已处理**序列的末尾，右指针指向**未处理**序列的开头，同时维护一个当前的偏移量遍历`bias`；

### 11. 盛最多水的容器

> 给定一个长度为`n`的整数数组`height`。有`n`条垂线，第`i`条线的两个端点是`(i, 0)`和`(i, height[i])`。
>
> 找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。返回容器可以储存的最大水量。你不能倾斜容器。

注意，本题只考虑两条垂线。一般解法是遍历所有可能的垂线组合。为了找到更优的解法，不妨从一组已知的最优组合`<a, b>`出发，分析其他可能的最优组合：

- 盛水高度由两条垂线中较短者决定，因此其他可能的最优垂线只能出现在区间`(a, b)`内部；
- 在宽度缩小的前提下，只有比当前较短垂线更高的垂线，才有机会成为新的最优解。

基于以上两点，我们可以采用一种简化的遍历策略来高效搜索最优垂线。

### 15. 三数之和<sup>*</sup>

> 给你一个整数数组`nums`，判断是否存在三元组`[nums[i], nums[j], nums[k]]`满足`i != j`、`i != k` 且`j != k` ，同时还满足`nums[i] + nums[j] + nums[k] == 0`。请你返回所有和为`0`且不重复的三元组。
>
> **注意：**答案中不可以包含重复的三元组。

本题的一般解法是`O(n^3)`遍历，我最初的思路是：先确定三个数的边界值，记为`[neg, third, pos]`（分别代表负数、待定值、正数），然后在收缩边界的过程中逐步得到结果。但这种方法涉及较多判断逻辑，具体如下：

1. 计算`need = 0 - neg - pos`，如果`need`存在于`(neg_idx, pos_idx)`区间内，则记录该三元组。
2. 根据`need`的值更新边界：
   * 若`need > 0`/`need < 0`：需要在`(need, pos)`/`(neg, pos)`中继续查找其他可能的组合，然后移动`neg_idx`/`pos_idx`，或直接调用官方解法；
   * 若`need == 0`：同时移动`neg_idx`和`pos_idx`，在`(neg, pos)`中继续寻找候选边界。

实现中，我先使用双指针确定候选边界，再用二分查找寻找满足条件的`third`值，并通过不断收缩`neg`/`pos`来实现三元组的去重。

不过，力扣官方并未推荐这种包含复杂判断的解法，而是采用更简洁的方式：先遍历固定`neg`边界，再通过双指针确定剩余两个元素。

### 42. 接雨水<sup>*</sup>

> 给定`n`个非负整数表示每个宽度为`1`的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

典中典单调栈；但这道题其实有三种解法，下面来详细学习一下解法。

#### a. 单调栈

我们依次将序列元素压入一个递减的单调栈。在维护单调栈的过程中，实际上已经在计算以当前柱子为边界的容器的蓄水容量。

具体来说：当弹出栈中比当前元素小的柱子时，可以立即计算这部分产生的容量。之后，我们将这些被弹出的、高度较低的柱子用「水」填补，使得它们的等效高度等于当前栈顶柱子的高度。这样，在后续处理新柱子时，只需计算引入新柱子所带来的增量水容量即可。

```ts
function monotonicStack(heights: number[]): number {
  let contain = 0;
  type Col = { height: number, width: number };

  const stack: Array<Col> = [];
  for (let i = 0; i < heights.length; i++) {
    const height = heights[i];

    let lastCol: Col | null = null;
    let widths = 0;

    while (stack.length && stack[stack.length - 1].height <= height) {
      lastCol = stack.pop()!;
      contain += (height - lastCol.height) * lastCol.width;
      widths += lastCol.width;
    }

    // move min(height, leftHeight) out
    if (stack.length === 0) {
      // when start widths = 0, barrierHeight = 0;
      const barrierHeight = lastCol?.height ?? 0;
      contain -= widths * (height - barrierHeight);
    }
    stack.push({ height, width: widths + 1 });
  }

  return contain;
}
```

#### b. 双指针

单调栈方法逐个处理每个小水坑，而双指针法可以一次性计算整个容器的蓄水总量。其基本思路是：先假设只有最左和最右的两根柱子存在，然后逐步向中间加入新的柱子，并动态调整蓄水容量。

关键规则如下：对于两根高度不同的边界柱子，如果从较高的一侧向内添加新柱子，后续水位可能升高，导致之前已添加的柱子对应的水位需要重新计算。因此，我们始终从**较矮的一侧**向中间扩展，这样就能保证当前水位的确定性，无需回溯。

![image-20260528044728986](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/return-vue/image-20260528044728986.png)

#### c. 动态规划

如果说单调栈是以“水坑”为单位，双指针是将整个水库视为一个整体，那么接下来要介绍的方法则是**以柱子为单位**逐一计算。

每个柱子能够承接的水量，实际上只取决于它左右两侧的最高柱子。我们可以先通过预处理，得到每个位置`i`左侧的最高高度 $\operatorname{leftMax}[i]$ 和右侧的最高高度 $\operatorname{rightMax}[i]$ 。那么，该位置能蓄的水量为：
$$
\begin{align*}
\operatorname{water}[i] = \min(\operatorname{leftMax}[i],\; \operatorname{rightMax}[i]) - \operatorname{height}[i];
\end{align*}
$$

> 这里最终计算公式没有使用动态规划，但`leftMax`、`rightMax`的求取过程实际上是用了的`DP`；

### 3. 无重复字符的最长子串

> 给定一个字符串`s`，请你找出其中不含有重复字符的**最长子串**的长度。

本题的最佳解法是**滑动窗口**。题目说明字符串`s`由英文字母、数字、符号和空格组成，因此我们可以用一个哈希表来存储当前窗口内每个字符的最新位置。

我最初的思路如下：

* 从左到右扩展窗口，遇到重复字符时，记录当前窗口的长度（候选最长子串）。
* 然后从重复字符的下一个位置重新开始，清空哈希表并重建窗口，继续寻找更长的无重复子串。

对应的代码实现：

```ts
function lengthOfLongestSubstring(s: string): number {
  const table = new Map<number, number>(); // code, idx

  let start = 0;
  let maxLength = -1;

  for (let i = 0; i < s.length; i++) {
    const char = s.charCodeAt(i);
    if (table.has(char)) {
      const dup = table.get(char)!;

      // reset hash map
      table.clear();
      for (let j = dup + 1; j <= i; j++) {
        table.set(s.charCodeAt(j), j);
      }

      // update maxLength
      const subLength = i - start;
      if (subLength > maxLength) {
        maxLength = subLength;
      }
      start = dup + 1;
    } else {
      table.set(char, i);
    }
  }

  return Math.max(s.length - start, maxLength);
}
```

上述方法实际上是在寻找最大的一对相同字符之间的间隔。但每当遇到重复字符，就需要重建哈希表，这一步骤成为性能瓶颈，导致最坏情况下的时间复杂度退化为`O(n^2)`。

但对于这个过程，我们不必每次遇到重复字符都重建哈希表，而是直接**存储每个字符上一次出现的索引**。只需将窗口左边界移动到`max(当前左边界, 上一次出现位置 + 1)`，同时更新该字符的最新索引即可。整个过程只需一次遍历，哈希表的更新和查询都是`O(1)`，整体时间复杂度降为`O(n)`。

### 438. 找到字符串中所有字母异位词

> 给定两个字符串`s`和`p`，找到`s`中所有`p`的**异位词**的子串，返回这些子串的起始索引。不考虑答案输出的顺序。
>
> **提示:**
>
> - `1 <= s.length, p.length <= 3 * 104`
> - `s`和`p`仅包含小写字母

这道题类似于字符串匹配的弱化版本。我最初的思路是：滑动一个大小为`p.length`的窗口，逐一比较窗口内字符的计数是否与`p`中一致。但每次匹配失败后，窗口的调整方式可能涉及`O(p)`的计数重置开销，效率较低。

受上一题最优解启发，我们可以为每个字符维护一个定长队列，队列长度等于该字符在`p`中出现的次数。遍历`s`时，将每个字符的位置加入对应队列。当所有字符队列的**队首元素**与当前位置的距离都在`p.length`范围内时，说明找到了一个字母异位词的起始位置。

此外，我们还可以采用哈希表计数+字符队列的方式优化定长队列；

### 560. 和为 K 的子数组<sup>*</sup>

> 给你一个整数数组`nums`和一个整数`k`，请你统计并返回 *该数组中和为`k`的子数组的个数* 。
>
> 子数组是数组中元素的**连续非空**序列。

暴力解法的时间复杂度为`O(n^2)`，这里不再赘述，直接讨论更优的方法。

我最初尝试使用双指针，但由于数组中元素的范围是`-1000 <= nums[i] <= 1000`，前缀和在指针移动过程中并不单调变化，因此双指针无法直接适用。

本题的推荐解法是**前缀和**。通过预处理前缀和，问题可以转化为「**1. 两数之和**」的变种——即寻找两个前缀和之差等于目标值`k`的组合。

### 239. 滑动窗口最大值<sup>*</sup>

> 给你一个整数数组`nums`，有一个大小为`k`的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的`k`个数字。滑动窗口每次只向右移动一位。
>
> 返回 *滑动窗口中的最大值* 。

先讲讲**优先队列**的解法。一开始看到用优先队列的提示时，我还在疑惑：如何弹出已经移出窗口的无用元素？

但转念一想，超出窗口范围的最大值，要么出现在当前元素之前，要么出现在之后。我们可以通过**按需入队**来避免考虑窗口之后的元素，同时只需在取值时检查堆顶元素是否仍在当前窗口内——如果不在，就弹出，直到堆顶有效。这样一来，窗口滑动的问题就轻松解决了。

```ts
class MyPriorityQueue<Value> {
  #heap: Array<{ score: number; value: Value }> = [];

  constructor(compareFn?: (a: number, b: number) => number) {
    // default: max heap, big score has more priority
    this.#compare = compareFn ?? ((a, b) => a - b);
  }

  #compare: (a: number, b: number) => number;
  #swap(aIdx: number, bIdx: number) {
    [this.#heap[aIdx], this.#heap[bIdx]] = [this.#heap[bIdx], this.#heap[aIdx]];
  }

  emplace(val: Value, score: number) {
    this.#heap.push({ value: val, score });
    let idx = this.#heap.length - 1;

    // bubble up
    while (idx > 0) {
      // compute parent idx
      const parent = (idx - 1) >> 1;
      if (this.#compare(this.#heap[idx].score, this.#heap[parent].score) > 0) {
        this.#swap(parent, idx);
        idx = parent;
      } else {
        break;
      }
    }
  }

  pop() {
    if (this.#heap.length === 0) return undefined;

    this.#swap(0, this.#heap.length - 1);
    const popped = this.#heap.pop()!;
    const len = this.#heap.length;

    // bubble down
    let idx = 0;
    while (true) {
      // compute child idx
      const left = idx * 2 + 1;
      const right = idx * 2 + 2;
      let bestIdx = idx;

      if (left < len && this.#compare(this.#heap[left].score, this.#heap[bestIdx].score) > 0) {
        bestIdx = left;
      }
      if (right < len && this.#compare(this.#heap[right].score, this.#heap[bestIdx].score) > 0) {
        bestIdx = right;
      }
      if (bestIdx === idx) break;

      this.#swap(idx, bestIdx);
      idx = bestIdx;
    }

    return popped.value;
  }

  peek() {
    return this.#heap[0]?.value;
  }

  get size() {
    return this.#heap.length;
  }
}
```

### 76. 最小覆盖子串<sup>*</sup>

> 给定两个字符串`s`和`t`，长度分别是`m`和`n`，返回 s 中的**最短窗口**子串，使得该子串包含`t`中的每一个字符（**包括重复字符**）。如果没有这样的子串，返回空字符串`""`。
>
> 测试用例保证答案唯一。

又是字符相关的题目，能否借鉴「3. 无重复字符的最长子串」和「438. 找到字符串中所有字母异位词」的思路，采用「字符 + 索引哈希」的方法来解决？初步设想如下：

* 使用一个**总体队列**（用于维护窗口的起始位置）和**每个字符对应的索引队列**。根据 `t` 中字符的出现次数，维护当前窗口内各字符的计数，以及匹配所需的字符队列。
* 遍历时，每遇到一个新字符，根据其队列更新窗口的左边界。随后将该字符索引入队，并更新对应字符的计数匹配情况，按需更新**最短覆盖子串**。

此外，判断计数是否满足`t`的要求时，我们其实可以使用一个总计数器`flag`来标记已经达到所需数量的字符种类数，从而避免逐字符遍历哈希表。

### 53. 最大子数组和

> 给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> **子数组**是数组中的一个连续部分。

DP

$$max[i] = \max(max[i-1] + nums[i], \;nums[i])$$



### 56. 合并区间

> 以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

预排序

### 189. 转轮数组

> 给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

1. k长缓存
2. `an=bk`，`gcd(n, b)`
3. 分段reverse

### 238. 除了自身以外数组的乘积

> 给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除了 `nums[i]` 之外其余各元素的乘积 。
>
> 题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内。
>
> 请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题。

维护`prod_l[i]`/`prod_r[i]`

### 41. 缺失的第一个正数<sup>*</sup>

> 给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。
>
> 请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

这道题是思维题，不要被`nums[i]`的取值吓到，最小正数的取值范围一定是`[1, n+1]`，其中`n+1`是数组元素都是从`1-n`中取的`n`个数；



