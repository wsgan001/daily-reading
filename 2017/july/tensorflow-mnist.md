- `tf.placeholder(dtype, shape)`,  where `shape` is a list `[num,..,num]`, `num` can be None, which denotes "any length"
- `reduce_sum` on matrices over certain axis using `reduction_indices=[..]`
- `reduce_mean`
- logits: Unscaled log probabilities.the `x` in `exp(x) / sum exp(x')`
- `with` statement with `tf.Session`
- `tf.argmax(y, 1)` to get the predicted/actual labels
- `tf.cast` to change datatype
- `tf.equal`
- `mnist.train.next_batch`
- `tf.global_variables_initializer().run()` to initialize variables