# Merge Sorted Arrays in Rust

|date | Description | Keywords |
|--------- |--------------------------------------------------- |----------- |
|2024-12-11| Leetcode - problem 88, merge sorted arrays in Rust |Leetcode, 88|

I wanted to start with some exercises that allow us to discuss functional problem solving exercises from platforms like Leetcode, Codility and Exercism.org. Among all this platforms, I would say leetcode is the most well known, but exercism is my favorite due to the very complete and transparent test cases. However, recently, I have been asked to live code in a mob session in Rust, Clojure and Go of leetcode problems, so this series is a representation of those sessions.

## Merge Sorted Arrays:

Leetcode gives us the following statement:

"_You are given two integer arrays `nums1` and `nums2`, **sorted** in **non-decreasing order**, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` **respectively**._

_Merge `nums1` and `nums2` into a single array sorted in non-decreasing order._
 
_The **final sorted array should not be returned by the function**, but instead be stored inside the array `nums1`. To accommodate this, `nums1` has a length of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a length of `n`._"

### Example cases:

* Example 1:

```yaml
Input: nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
Output: [1,2,2,3,5,6]
Explanation: The arrays we are merging are [1,2,3] and [2,5,6].
The result of the merge is [1,2,2,3,5,6] with the underlined 
elements coming from nums1.
```


* Example 2:

```yaml
Input: nums1 = [1], m = 1, nums2 = [], n = 0
Output: [1]
Explanation: The arrays we are merging are [1] and [].
The result of the merge is [1].
```

* Example 3:

```yaml
Input: nums1 = [0], m = 0, nums2 = [1], n = 1
Output: [1]
Explanation: The arrays we are merging are [] and [1].
The result of the merge is [1].
Note that because m = 0, there are no elements in nums1. The 0 is only
there to ensure the merge result can fit in nums1.
```


### Constraints

* `nums1.length == m + n`
* `nums2.length == n`
* `0 <= m, n <= 200`
* `1 <= m + n <= 200`
* `-109 <= nums1[i], nums2[j] <= 109`

## Naive Solution:

we can start by doing a simple for loop that iterates over the range of elements `(0..n)`, which means, all elements in `nums2`:


```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    for nums2_index in (0..n) {
        // ...
    }
}
```

Now, we know that we need to add each element of `nums2` to a position in `nums1`, starting in `m`:


```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    for nums2_index in (0..n) {
        nums1[(m + nums2_index) as usize] = nums2[nums2_index as usize];
    }
}
```

With this, our array contains 2 sorted sections of arrays in `nums1`, so we could just call a simple rust sort there:


```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    for nums2_index in (0..n) {
        nums1[(m + nums2_index) as usize] = nums2[nums2_index as usize];
    }
    nums1.sort();
}
```

A final style touch to this code is that we don't need to pass a mutable reference to a `Vec<i32>`, we can just pass `&mut &[i32]`. Also, style wise, the correct way to express a range in a for loop is without `()`, so we should rewrite the for loop as `for num2_index in 0..n`:

```rust
pub fn merge_naive(nums1: &mut [i32], m: i32, nums2: &mut [i32], n: i32) {
    for nums2_index in 0..n {
        nums1[(m + nums2_index) as usize] = nums2[nums2_index as usize];
    }
    nums1.sort();
}
```

According to benchmarks using criterion and memory-stats, for only the first example the execution time is around 7.8 ns and 40 KB of memory (all relative to my machine). For the largest example I have it has an execution time of 1 ms and a Memory consumption of 2.19 MB, not a bad start.

## Less procedural

Rust has a great function to solve this problem and to solve my problem with for loops, it is called `splice`. [Splice](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.splice) receives a mutable reference to the vector, a range of points to splice (`replace_with`) the vector, and an `IntoIterator` to loop. The following code represents the solving of this problem with this function: 


```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    let m = m as usize;
    let n = n as usize;
    nums1.splice(m..m+n, nums2.to_owned());
    nums1.sort();
}
```

One thing I don't like in this exact code is `m+n` range ending, as we could consider for this problem `n+m == nums1.len()`, so the whole function would look like:


```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    let m = m as usize;
    let _ = nums1.splice(m.., nums2.to_owned());
    nums1.sort();
}
```

Another take on this, would be to use the n variable with a `take` function, as well as simplify the use case of the `& mut Vec<i32>` where Vec is not needed:

```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut [i32], n: i32) {
    let m = m as usize;
    nums1.splice(m.., nums2.iter().copied().take(n as usize));
    nums1.sort();   
}
```

This is about 10% more memory efficient and 10% less performatic than the for loop, that consumed around  40 KB for the first example, while this approach consumes only around 36 KB, for the largest test case. Performance wise, this approach being about 10% less performatic with 8.5 ns. Another interesting addition of using `splice` is that if we `collect::<Vec<i32>>()` the splice, we can return the exactly elements that were replaced by paying the allocation cost of a new `Vec<i32>`.


"_But Julia! you are using std functions to solve this problem!_". Yes! young padawan, why recreate the wheel? But if you really want, here is a guide:

1. _Check if the first `m` elements are `0`, if they are, just replace them with `nums2`._
2. _Create an iterator for `nums2`._
3. _Loop over `nums1`, and if the `next` element in `nums2` iterator is less of equal to the current element in `nums1`, insert it in the list and pop the last element._
4. _Get the next element in `nums2` iterator._

> A pure functional approach in this problem is hard as we are dealing with mutability in `nums1`, but we could easily create a new vector from the originals ones and return it. This would be terrible memory wise as we are allocating a new vector full of new elements.

For the previous guide, a more functional approach can be using fold and collecting the elements in a new vector. A pure functional way would require returning that list and avoiding mutability. However, that would fail leetcode test cases, so it will need to be assigned to `nums1`. I would risk saying that `m` and  `n` are useless in a more functional Rust.

> If you need help, you can open an issue in the repo
>
> ## Benchmark dependencies:
> ```toml
> [dev-dependencies]
> criterion = "0.4"
> memory-stats = "1"
> ```
