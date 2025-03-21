To run the shell scripts inside experiments folder, we need sudo rights.


```python
1. changed all "from . " to ""
2. changed all "from ." to "from "
so that we can run without the module (-m) easily in pyCharm; much easier to debug

files.py has two versions; (overloaded) one inside the dataset/files.py; Other inside the python/files.py 

```

The changed version is [https://github.com/dblalock/bolt/compare/master...abcnishant007:boltfork:master](https://github.com/dblalock/bolt/compare/master...abcnishant007:boltfork:master) 
```
Some parts of the code are in python2.7, 
Those have been changed; But kmc2 is only available for python2.7 in pip. Now, the algo doesn't seem to run when we use the "subspaces" option.   File "/Users/nishant/Documents/GitHub/boltfork/python/bolt/bolt_api.py", line 91, in kmeans
    raise ValueError("K must be a square number if init='subspaces'")
ValueError: K must be a square number if init='subspaces' 

So, now we seem to have hit a wall! 

Based on the "subspaces" library, we hit recursion depth! :( 
  File "mtrand.pyx", line 244, in numpy.random.mtrand.RandomState.seed
  File "_mt19937.pyx", line 166, in numpy.random._mt19937.MT19937._legacy_seeding
RecursionError: maximum recursion depth exceeded while calling a Python object 


```

```
Python 3.7.16 (default, Jan 17 2023, 09:28:58) 
[Clang 14.0.6 ] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
```

```
numpy was downgraded for numba to work
```

```
The kmc2 library https://github.com/obachem/kmc2 does not work. This initialisation has been removed and instead, we have just used the other initialisation mechanism.
```

Requirements.txt is shown below
in order to run the `python -m python.amm_main` and this should be run inside the experiments folder.
```bash
charset-normalizer==3.4.1
cycler==0.11.0
fonttools==4.38.0
idna==3.10
importlib-metadata==6.7.0
joblib==1.3.2
kiwisolver==1.4.5
llvmlite==0.39.1
matplotlib==3.5.3
nose==1.3.7
numba==0.56.4
numpy==1.18.0
packaging==24.0
pandas==1.3.5
Pillow==9.5.0
pyparsing==3.1.4
python-dateutil==2.9.0.post0
pytz==2025.1
requests==2.31.0
scikit-learn==1.0.2
scipy==1.7.3
seaborn==0.12.2
six==1.17.0
threadpoolctl==3.1.0
torch==1.13.1
```



OS
```
Mac 2020 pro: 14.6.1 Sonoma
```
**Special notes for experiments**
- The fig2 implies everything else apart from cifar 10;
- The fig1 implies cifar 10.

- "brew install bazel"-- doesnt work; seems homebrew has been removed by mac due to it being marked too unsafe.
- conda install bazel seems to work;
- pip install swif if needed later on




  
<p align="center">
  <img src="https://github.com/dblalock/bolt/blob/master/assets/bolt.jpg?raw=true" alt="Bolt" width="611px" height="221px"/>
  <!-- <img src="https://github.com/dblalock/bolt/blob/master/assets/bolt.jpg?raw=true" alt="Bolt" width="685px" height="248px"/> -->
</p>

Bolt is an algorithm for compressing vectors of real-valued data and running mathematical operations directly on the compressed representations.

If you have a large collection of mostly-dense vectors and can tolerate lossy compression, Bolt can probably save you 10-200x space and compute time.

Bolt also has [theoretical guarantees](https://github.com/dblalock/bolt/blob/master/assets/bolt-theory.pdf?raw=true) bounding the errors in its approximations.

EDIT: this repo now also features the source code for [MADDNESS](https://arxiv.org/abs/2106.10860), our shiny new algorithm for approximate matrix multiplication. MADDNESS has no Python wrapper yet, and is referred to as "mithral" in the source code. Name changed because apparently I'm the only who gets Lord of the Rings references. MADDNESS runs ridiculously fast and, under reasonable assumptions, requires zero multiply-adds. Realistically, it'll be most useful for speeding up neural net inference on CPUs, but it'll take another couple papers to get it there; we need to generalize it to convolution and write the CUDA kernels to allow GPU training. See also the [poster](https://github.com/dblalock/bolt/blob/master/assets/blalock-maddness-poster.png) and [slides](https://github.com/dblalock/bolt/blob/master/assets/snn-maddness.pdf). <!-- (it's lightweight, but still full strength! Get it? Guys...?). -->

EDIT2: Looking for a research project? See our [list of ideas](https://github.com/dblalock/bolt/tree/master/experiments).

EDIT3: See [Build.md](https://github.com/dblalock/bolt/blob/master/BUILD.md) for a working dockerfile that builds and runs Bolt, contributed by @mneilly.

**NOTE: All below code refers to the Python wrapper for Bolt and has nothing to do with MADDNESS.** It also seems to be [no longer building](https://github.com/dblalock/bolt/issues/4) for many people. If you want to use MADDNESS, see the [Python Implementation](https://github.com/dblalock/bolt/blob/45454e6cfbc9300a43da6770abf9715674b47a0f/experiments/python/vq_amm.py#L273) driven by [amm_main.py](https://github.com/dblalock/bolt/blob/45454e6cfbc9300a43da6770abf9715674b47a0f/experiments/python/amm_main.py) or [C++ implementation](https://github.com/dblalock/bolt/blob/45454e6cfbc9300a43da6770abf9715674b47a0f/cpp/src/quantize/mithral.cpp). All code is ugly, but Python code should be pretty easy to add new AMM methods/variations to.

<!-- NOTE: All the code, documentation, and results associated with Bolt's KDD paper can be found in the `experiments/` directory. See the README therein for details. A cleaned-up version of the paper is available [here](https://github.com/dblalock/bolt/blob/master/assets/bolt.pdf?raw=true). -->

## Installing

#### Python

```bash
  $ brew install swig  # for wrapping C++; use apt-get, yum, etc, if not OS X
  $ pip install numpy  # bolt installation needs numpy already present
  $ git clone https://github.com/dblalock/bolt.git
  $ cd bolt && python setup.py install
  $ pytest tests/  # optionally, run the tests
```

If you run into any problems, please don't hesitate to mention it [in the Python build problems issue](https://github.com/dblalock/bolt/issues/4).

#### C++

Install [Bazel](https://bazel.build), Google's open-source build system. Then
```bash
  $ git clone https://github.com/dblalock/bolt.git
  $ cd bolt/cpp && bazel run :main
```

The `bazel run` command will build the project and run the tests and benchmarks.

If you want to integrate Bolt with another C++ project, include `cpp/src/include/public.hpp` and add the remaining files under `cpp/src` to your builds. You should let me know if you're interested in doing such an integration because I'm hoping to see Bolt become part of many libraries and thus would be happy to help you. <!-- Note that the `BoltEncoder` object you'll interact with presently needs something else to feed it k-means centroids-see `python/bolt/bolt_api.py` for an example. -->

#### Notes

Bolt currently only supports machines with [AVX2 instructions](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#Advanced_Vector_Extensions_2), which basically means x86 machines from fall 2013 or later. Contributions for ARM support [are welcome](https://github.com/dblalock/bolt/issues/2). Also note that the Bolt Python wrapper is currently configured to require Clang, since GCC apparently [runs into issues](https://github.com/dblalock/bolt/issues/4).

## How does it work?

Bolt is based on [vector quantization](https://en.wikipedia.org/wiki/Vector_quantization). For details, see the [Bolt paper](https://arxiv.org/abs/1706.10283) or [slides](https://github.com/dblalock/bolt/blob/master/assets/bolt-slides.pdf?raw=true).

## Benchmarks

Bolt includes a thorough set of speed and accuracy benchmarks. See the `experiments/` directory. This is also what you want if you want to reproduce the results in the paper.

Note that all of the timing results use the raw C++ implementation. At present, the Python wrapper is slightly slower due to Python overhead. If you're interested in having a full-speed wrapper, let me know and I'll allocate time to making this happen.

## Basic usage
```python
X, queries = some N x D array, some iterable of length D arrays

# these are approximately equal (though the latter are shifted and scaled)
enc = bolt.Encoder(reduction='dot').fit(X)
[np.dot(X, q) for q in queries]
[enc.transform(q) for q in queries]

# same for these
enc = bolt.Encoder(reduction='l2').fit(X)
[np.sum((X - q) * (X - q), axis=1) for q in queries]
[enc.transform(q) for q in queries]

# but enc.transform() is 10x faster or more
```

## Example: Matrix-vector multiplies

```python
import bolt
import numpy as np
from scipy.stats import pearsonr as corr
from sklearn.datasets import load_digits
import timeit

# for simplicity, use the sklearn digits dataset; we'll split
# it into a matrix X and a set of queries Q
X, _ = load_digits(return_X_y=True)
nqueries = 20
X, Q = X[:-nqueries], X[-nqueries:]

enc = bolt.Encoder(reduction='dot', accuracy='lowest') # can tweak acc vs speed
enc.fit(X)

dot_corrs = np.empty(nqueries)
for i, q in enumerate(Q):
    dots_true = np.dot(X, q)
    dots_bolt = enc.transform(q)
    dot_corrs[i] = corr(dots_true, dots_bolt)[0]

# dot products closely preserved despite compression
print("dot product correlation: {} +/- {}".format(
    np.mean(dot_corrs), np.std(dot_corrs)))  # > .97

# massive space savings
print(X.nbytes)  # 1777 rows * 64 cols * 8B = 909KB
print(enc.nbytes)  # 1777 * 2B = 3.55KB

# massive time savings (~10x here, but often >100x on larger
# datasets with less Python overhead; see the paper)
t_np = timeit.Timer(
    lambda: [np.dot(X, q) for q in Q]).timeit(5)        # ~9ms
t_bolt = timeit.Timer(
    lambda: [enc.transform(q) for q in Q]).timeit(5)    # ~800us
print "Numpy / BLAS time, Bolt time: {:.3f}ms, {:.3f}ms".format(
    t_np * 1000, t_bolt * 1000)

# can get output without offset/scaling if needed
dots_bolt = [enc.transform(q, unquantize=True) for q in Q]
```

## Example: K-Nearest Neighbor / Maximum Inner Product Search
```python
# search using squared Euclidean distances
# (still using the Digits dataset from above)
enc = bolt.Encoder('l2', accuracy='high').fit(X)
bolt_knn = [enc.knn(q, k_bolt) for q in Q]  # knn for each query

# search using dot product (maximum inner product search)
enc = bolt.Encoder('dot', accuracy='medium').fit(X)
bolt_knn = [enc.knn(q, k_bolt) for q in Q]  # knn for each query
```

## Miscellaneous

Bolt stands for "Based On Lookup Tables". Feel free to use this exciting fact at parties.

<!-- 2) If you use Bolt, let me know and I'll link to your project/company. -->

