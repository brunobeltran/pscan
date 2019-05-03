PScan.Scan
==========

A module whose sole purpose is to facilitate parameter scans via a
generic framework for specifying parameter values to scan.

Combinatorial parameter sweeps, joint parameter sweeps, "scientific"
parameter sweeps, and advanced options for how many times to repeat
simulations are available.

This module was written after the author got tired of writing the same
boiler plate code every time he wanted to sweep parameters in his
simulations.

The impetus for the design of the module was the realization that there
are basically only four types of "parameter sweeps" that ever really
need to be done.

1. run the same parameters many times (e.g. stoch simulation)
2. vary certain parameters jointly (e.g. (i,j) = (1,2), (2,3), (3,4),
   ... )
3. vary parameters combinatorially (e.g. (i,j) = (1,1), (1,2), (2,1),
   (2,2))
4. vary "scientifically" (e.g. (i0,j),(i,j0) for fixed i0,j0, varying
   i,j)

All three of these can be specified declaratively (e.g. in English
sentences) relatively easily, but usually each requires coding an
easy-to-goof up nested for loop. This module turns a simple listing of
which parameters into an iterable you can query to get the next set of
parameters in your scan. If a Scan is stopped midway, it will remember
its state, allowing you to load it back up, change the scan parameters,
and continue the run. A Scan is *not* useful if your code needs to
regularly generate the next parameters based on previous simulations.

If you have several "jointly" varying parameters, each "group" of them
will interact combinatorially. More abstractly, you should think of each
set of jointly varying parameters as being the same as a "single"
parameter.

Here's an example that uses all of the features of the package. Say we
have two parameters, ``a`` and ``b``, that we want to vary jointly, and
another parameter, ``c``, that we want to to vary for each value of
``(a,b)``. To further complicate things, we want to run the simulation 2
times for each set of ``(a,b,c)``, except that if 'c' is too big, we run
less repeats to save time. This would normally involve a lot of
carefully-coded boilerplate, but instead we can just tell ``PScan.Scan``
what we want:

::

    >>> from pscan import Scan
    >>> import numpy as np
    >>> def f(a,b,c):
    >>>     print('a = ', a, ', b = ', b, ', c = ', c)
    >>> p = {}
    >>> p['a'] = np.linspace(0, 10, 5)
    >>> p['b'] = np.linspace(0.1, -0.5, 5)
    >>> p['c'] = np.linspace(10, 20, 3)
    >>> default_count = lambda p: 2
    >>> # None specifies to default to previous value
    >>> big_c_count = lambda p: 1 if p['c'] >= 14 else None
    >>>
    >>> s = Scan.from_dict(p, joint_lists=[['a','b']])
    >>> s.add_count(default_count) # these will be called
    >>> s.add_count(big_c_count) # in the order they were added
    >>> s.run_scan(f) # verify scan validity and run

        a =  0.0 , b =  0.1 , c =  10.0
        a =  0.0 , b =  0.1 , c =  10.0
        a =  2.5 , b =  -0.05 , c =  10.0
        a =  2.5 , b =  -0.05 , c =  10.0
        a =  5.0 , b =  -0.2 , c =  10.0
        a =  5.0 , b =  -0.2 , c =  10.0
        a =  7.5 , b =  -0.35 , c =  10.0
        a =  7.5 , b =  -0.35 , c =  10.0
        a =  10.0 , b =  -0.5 , c =  10.0
        a =  10.0 , b =  -0.5 , c =  10.0
        a =  0.0 , b =  0.1 , c =  15.0
        a =  2.5 , b =  -0.05 , c =  15.0
        a =  5.0 , b =  -0.2 , c =  15.0
        a =  7.5 , b =  -0.35 , c =  15.0
        a =  10.0 , b =  -0.5 , c =  15.0
        a =  0.0 , b =  0.1 , c =  20.0
        a =  2.5 , b =  -0.05 , c =  20.0
        a =  5.0 , b =  -0.2 , c =  20.0
        a =  7.5 , b =  -0.35 , c =  20.0
        a =  10.0 , b =  -0.5 , c =  20.0

