- `set.issubset`

- `np.random.choice`: `replace=True` by default

```python
with pytest.raises(AssertionError):
    q_gen.random()
```

# clean and create a directory in tensorflow

```python
if tf.gfile.Exists(FLAGS.log_dir):
    tf.gfile.DeleteRecursively(FLAGS.log_dir)
  tf.gfile.MakeDirs(FLAGS.log_dir)
```

# "remove" nodes in `graph_tool`

hide it by setting vertex_fitler, benefit:

- it's fast
- node indexes are not changed

# accessing `vertex_property` and `edge_property`

it's slow. use `dict` if possible

# custome object for `set`

change the following functions:

- `__eq__`
- `__ne__`
- `__hash__`

[source](https://stackoverflow.com/questions/5754440/performing-set-operations-on-custom-classes-in-python)

# `vertex_filter`

for graph with hidden hides, its `vertex_filter` will raise error if hidden hides are accessed. 

however, `vertex_filter.a` still have the same length as `g.num_vertices()`

# `graph_tool` installation

`./configure CXXFLAGS="-std=gnu++14"`


# `graph_tool` plot in `matplotlib`

remembe to add before plotting:

- `plt.swith_backend('cairo')

# `Graph.vertex` and `Graph.edge`

- slow to access the node and edge by integers

```
1755     26303      2891233    109.9     95.5          v = libcore.get_vertex(self.__graph, int(i), use_index)
```

```
  1776      3184       333105    104.6     34.9          s = self.vertex(int(s))
  1777      3184       324256    101.8     33.9          t = self.vertex(int(t))
  1778      3184       286766     90.1     30.0          edges = libcore.get_edge(self.__graph, int(s), int(t), all_edges)
```

`graph_python_interface.cc`: 

- `get_vertex`: `get_vertex_hard` (`O(n)`) and `get_vertex_soft` (`O(1)`)
- `get_edge` internally runs `get_edge_dispatch`, seems to iterate through out edges

`Graph.vertex(...)`: 

- if `use_index=True`: return the node with index `i`
- if `use_index=False`: return the `i`th vertex (can differ if the graph is filtered)

# quickly check if a node is in `g`

`g._Graph__filter_state['vertex_filter'][0].a[node]`

don't use `g.vertex`

# GraphView creation

```
  3104       404       529413   1310.4     36.8                      ef[0].fa = efilt.fa
```
setting edge filter is slow, takes `36.8\%` of the time. 


# `line_profiler` + `Ipyhton`

`c.InteractiveShellApp.extensions = ['line_profiler']`

or 

`>>> %load_ext line_profiler`

usage:

`%lprun -f my_method for i in range(1000): my_method()`

# real bottle neck: contracting graph

related to the slow part: `g.edge`

before optimizing:

```
   206     31252      4782729    153.0     38.7          new_g.add_edge(u, v)
   207
   208      1000        59560     59.6      0.5      new_weights = new_g.new_edge_property('float')
   209     32252        55232      1.7      0.4      for (u, v), w in e2w.items():
   210     31252      5200172    166.4     42.1          new_weights[new_g.edge(u, v)] = w
```

after optimzing:

```
   205      1000         1360      1.4      0.0      edges = []
   206     32261        34668      1.1      0.5      for u, v in e2w:
   207     31261      5101433    163.2     76.2          e = new_g.add_edge(u, v)
   208     31261        51667      1.7      0.8          edges.append(e)
   209
   210      1000        58566     58.6      0.9      new_weights = new_g.new_edge_property('float')
   211     32261        41056      1.3      0.6      for e, w in zip(edges, e2w.values()):
   212     31261       364971     11.7      5.5          new_weights[e] = w
   213
```

assigning weight takes only 11.7, instead of 166.4 now. 


# make `matplotlib.colormap` using `seaborn`

just wonderful!

https://stackoverflow.com/questions/37902459/how-do-i-use-seaborns-color-palette-as-a-colormap-in-matplotlib

- `ListedColormap`
- `sns.color_palette(colors).as_hex()`

# `graph_tool.cairo_draw`

seems to have normalized the colormap

# resize figure

`fig.set_size_inches(width, height, forward=True)`

# `jupyter-nbconvert`

instead of `jupyter nbconvert`

# tri-diagonal matrix

`np.diag(a, -1) + np.diag(b, 0) + np.diag(c, 1)`

# lu decomposition

```python
import scipy
import scipy.linalg

scipy.linalg.lu(A)
```

# catch multiple exceptions

```python
except (IDontLikeYouException, YouAreBeingMeanException) as e:
```


# tf debugging

- https://www.tensorflow.org/programmers_guide/debugger
  - including memory consumption
  - however, there is bug: https://github.com/tensorflow/tensorflow/issues/12059
- profiling: https://github.com/tensorflow/tensorflow/issues/1824


# tf get tensor from checkpoint

```pyton
import tensorflow as tf

checkpoint_file = 'runs/deepwalk/checkpoints/model-25000'

sess = tf.InteractiveSession()

saver = tf.train.import_meta_graph("{}.meta".format(checkpoint_file))
saver.restore(sess, checkpoint_file)

embedding_table = sess.graph.get_operation_by_name('embedding/table')

val = embedding_table.outputs[0].eval()

print(val)
```

https://gist.github.com/xiaohan2012/5456951471c6e7fd19fc796809ab303a

# `python setup.py`

note to the following:

- `packages=['name-of-the-package']`

```python
core_module = Extension(
    'pyedmond/_core',  # the path to save the compiled module
    include_dirs=['/usr/include/python3.5/'],
    libraries=['boost_python-py35', 'boost_graph'],
    library_dirs=['/usr/lib/x86_64-linux-gnu/'],
    extra_compile_args=['-std=c++11', '-O2', '-Wall'],
    extra_link_args=['-Wl,--export-dynamic'],
    sources=['pyedmond/_core.cpp']
)
```


adding test: https://docs.pytest.org/en/latest/goodpractices.html#integrating-with-setuptools-python-setup-py-test-pytest-runner

```python
os.environ["CC"] = "g++-4.7"
os.environ["CXX"] = "g++-4.7"
```

to specifiy enrionment variables such as compiler

authentication issue when doing `python setup.py sdist upload`:  https://github.com/pypa/setuptools/issues/941

- create a `.pypirc` file
