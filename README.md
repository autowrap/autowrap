autowrap
========

Generates Python Extension modules from Cythons PXD files.

This module uses the Cython "header" .pxd files to automatically generate
Cython input (.pyx) files. It does so by parsing the header files and possibly
annotations in the header files to generate correct Cython code. For an
example, please have a look at `examples/int_holder.h` and
`example/int_holder.pxd` which together form the input to the program.

Simple example
---------------------

Assuming you want to wrap the following C++ class


    class IntHolder {
        public:
            int i_;
            IntHolder(int i): i_(i) { };
            IntHolder(const IntHolder & i): i_(i.i_) { };
            int add(const IntHolder & other) {
                return i_ + other.i_;
            }
    };


you could generate the following .pyd file and run autowrap


    cdef extern from "int_holder.hpp":
        cdef cppclass IntHolder:
            int i_
            IntHolder(int i)
            IntHolder(IntHolder & i)
            int add(IntHolder o)


which will generate Cython code that allows direct access to the public
internal variable `i_` as well as to the two constructors.

Compiling 
-------------

To compile the above examples to .pyx and .cpp files change the directory
to the folder containing `int_holder.hpp` and `int_holder.pxd` and run

    $ autowrap --out py_int_holder.pyx int_holder.pxd

which will generate files `py_int_holder.pyx` and `py_int_holder.cpp`
which you can compile using the following file `setup.py` which we
provide in the `examples` folder:


    from distutils.core import setup, Extension

    import pkg_resources

    data_dir = pkg_resources.resource_filename("autowrap", "data_files")


    from Cython.Distutils import build_ext

    ext = Extension("py_int_holder", sources = ['py_int_holder.cpp'], language="c++",
            extra_compile_args = [],
            include_dirs = [data_dir],
            extra_link_args = [],
            )

    setup(cmdclass = {'build_ext' : build_ext},
        name="py_int_holder",
        version="0.0.1",
        ext_modules = [ext]
        )

You can build the final Python extension module by running

    $ python setup.py build_ext --inplace

And you can use the final module running

    >>> import py_int_holder
    >>> ih = py_int_holder.IntHolder(42)
    >>> print ih.i_
    42
    >>> print ih.add(ih)
    84

Further docs can be found in 'docs/' folder.
