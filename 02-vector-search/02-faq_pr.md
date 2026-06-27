## Question

In Lesson 4: Vector Search, why can `np.allclose(scores, scores_loop)` return `False` when comparing the matrix multiplication version of vector search with a Python for loop? Aren't they doing the same math?

## Answer

Yes, they are doing the same math. The difference comes from floating-point precision and the order in which numbers are added.

The matrix version `X.dot(v_query)` is handled by optimized low-level numerical code, while the Python loop adds values sequentially. That can produce tiny differences in the last decimal places, even when the results are effectively the same for search purposes.

Depending on the exact FAQ data and the vectors in the index, the scores may be close enough to pass `np.allclose()` or close enough to fail it if the tolerance is too strict. We can allow for a small tolerance and make it slightly more forgiving with `atol`:

```python
print(f"Loop matches matrix version: {np.allclose(scores, scores_loop, atol=1e-5)}")
```

With that adjustment, the comparison should be more robust across small numerical differences.