#  Best Time to Buy and Sell Stock in Rust

|date | Description | Keywords |
|--------- |------------------------------------------------------------------------- |----------------- |
|2024-12-14| Leetcode - problems 121 and 122, Best Time to Buy and Sell Stock in Rust |Leetcode, 121, 122|


Leetcode has two best time to sell problems that I have solved, one easy and one medium. Here we will take a look at both, starting from the easy then going to the medium.

##  Best Time to Buy and Sell Stock (Easy/121):

Leetcode gives us the following statement:

"_You are given an array prices where `prices[i]` is the **price of a given stock on the ith day**._

_You want to **maximize your profit** by choosing a single day to buy one stock and choosing a different day in the future to sell that stock._

_Return the maximum profit you can achieve from this transaction. If you cannot achieve any profit, return `0`._"

### Example cases:

* Example 1:

```yaml
Input: prices = [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.
```

* Example 2:

```yaml
Input: prices = [7,6,4,3,1]
Output: 0
Explanation: In this case, no transactions are done and the max profit = 0.
```

### Constraints

* `1 <= prices.length <= 105`
* `0 <= prices[i] <= 104`

## My Solution:

This is a pretty straight forward problem, you have a vector of elements, where all elements are larger or equal to zero and you need to identify the maximum and the minimum of this vector. You can achieve this by iterating over all elements and preserving the minimum and the maximum difference for each element. In a procedural approach, this would mean:

```rust
pub fn max_profit(prices: Vec<i32>) -> i32 {
    let mut min = i32::MAX;
    let mut max_profit = 0;

    for value in prices {
        min = value.min(min);
        max_profit = max_profit.max(value - min);
    }

    max_profit
}
```

So, we create two variables to control `min`, which starts as anything above `104` (second constraint, but `i32::MAX` works beyond that constraint) and `max_profit`, that should start as zero in case we do not find any maximum value. Then we loop over all elements and reassign `min` and `max_profit` for the current `value`. Definitvely works, and is quite uncomplex solution with constant value allocation and a simple loop over `n` (`prices.len()`). Now we need to transition this to a functional solution, which in this case is have an **iterator consumer** and **immutable data**:

```rust
pub fn max_profit(prices: Vec<i32>) -> i32 {
    prices
        .iter()
        .fold((i32::MAX, 0), |(min, max_profit), &value| {
            (value.min(min), max_profit.max(value - min))
        })
        .1
}
```


To begin, we create an iterator, `prices.iter()`, then we consume this iterator with a `fold`, where the `accumulator` is a tuple consisting of the minimal value, same as before, and a maximum profit starting as `0`. Then, in the `fold` loop, we already return a tuple containing the new minimum, `value.min(min)`, as the first element, and the new maximum profit `max_profit.max(value - min)`, as the second element. After we have consumed all the iterator, just return the second element of the resulting tuple.

> Benchmark wise both solutions were equivalent in execution time and memory using criterion and memory-stats crates.

##  Best Time to Buy and Sell Stock (Medium/122):

"_You are given an integer array prices where `prices[i]` is the price of a given stock on the ith day._

_On each day, you may decide to buy and/or sell the stock. You can only **hold at most one share of the stock at any time**. However, you can buy it then immediately sell it on the **same day**_.

_Find and return the **maximum profit** you can achieve._"

### Example cases:

* Example 1:

```yaml
Input: prices = [7,1,5,3,6,4]
Output: 7
Explanation: Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = 5-1 = 4.
Then buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = 6-3 = 3.
Total profit is 4 + 3 = 7.
```

* Example 2:

```yaml
Input: prices = [1,2,3,4,5]
Output: 4
Explanation: Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
Total profit is 4.
```


* Example 3:

```yaml
Input: prices = [7,6,4,3,1]
Output: 0
Explanation: There is no way to make a positive profit, so we never buy the stock to achieve the maximum profit of 0.
```

### Constraints

* `1 <= prices.length <= 3 * 104`
* `0 <= prices[i] <= 104`

## My Solution:

This problem is a bit more flexible as we can buy and sell in the same day, which means we can use a method that traverses the vector two by two, `windows`. This method allows us to traverse the following array `[1, 2, 3, 4]`, in the following manner `[[1, 2], [2, 3], [3,4]]`. This means that for the example `[7,1,5,3,6,4]`, we get the following results `[[7,1], [1,5], [5,3], [3,6], [6,4]]`, if we filter out the results where the second element is smaller than the first we get `[[1,5], [3,6]]`, which means a maximum profit of `(5 - 1) + (6 - 3)`. This works well even with se quentially increasing number like `[1, 2, 3, 4, 5, 6, 7]`. Considering this, the trivial answer would be:


```rust
pub fn max_profit(prices: Vec<i32>) -> i32 {
    prices.windows(2)
        .filter(|price| price[0] < price[1])
        .map(|price| price[1] - price[0])
        .sum()
}
```

Where we iterate over prices two by two, filtering out all pairs where the second element is smaller than the first, so we map their difference and sum all the values. Note that sum starts at `zero`. Even if this is a good solution, I figured there is an even better approach to this, which results in one less line of code:

```rust
pub fn max_profit(prices: Vec<i32>) -> i32 {
    prices.windows(2)
        .map(|price| 0.max(price[1] - price[0]))
        .sum()
}
```

In this new solution, we replace the `filter` with a `map` operation that gets the maximum value between `zero` and the difference between prices (`price[1] - price[0]`). This means that if `price[0] >= price[1]`, the map will return `0`, allowing us to remove the `filter` line.
