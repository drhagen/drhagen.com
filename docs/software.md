# Software

## Parsita

[Parsita](https://parsita.drhagen.com/) is a parser combinator library for Python. It is [hosted](https://github.com/drhagen/parsita) on Github and [available](https://pypi.python.org/pypi/parsita) on PyPI. Its main goal is a maximally simple interface for parsing. The API is inspired by Scala's parser combinator library, but the operators were carefully selected to have intuitive meaning and precedence in Python. A unique feature of Parsita is that it uses metaclass magic to allow for forward declarations of values by converting names not found during class body execution into forward declaration objects resolved during class instantiation.

## Tabeline

[Tabeline](https://tabeline.drhagen.com/) is a data table and data grammar library for Python. It is inspired by [dplyr](https://dplyr.tidyverse.org/) in R and uses [Polars](https://pola-rs.github.io/polars-book/user-guide/index.html) underneath. It provides and easy-to-use interface for doing data manipulations by providing the standard verbs (like `select`, `filter`, and `mutate`) as methods on its central `DataTable` class and having those methods accept strings (like `table.filter("t < 24")`) which are parsed and executed in the context of the table.

## Tensora

[Tensora](https://tensora.drhagen.com/) is a sparse and dense tensor algebra library for Python. It is based on the [Tensor Algebra Compiler](http://tensor-compiler.org/) (TACO). The user can create tensors whose formats are varying combinations of dense and sparse dimensions. The tensor can be populated from a variety of sources, such as dictionaries of keys, list of tuples, tuples of lists, NumPy arrays, and SciPy sparse matrices. While basic mathematical operators, such as `__add__`, `__sub__`, `__matmul__`, are defined, the most important feature is the `evaluate` function, which takes a string like `"y(i) = A(i,j) * x(j)"` and executes that expression on the given tensors. Under the hood, `evaluate` parses the tensor algebra expression, validates it, turns it into C code, invokes a C compiler on that code, executes the code on the given tensors, and returns the result to the user. All in all, it provides an easy-to-use interface to a very powerful tool.

## Serialite

[Serialite](https://github.com/drhagen/serialite) is serialization/deserialization library for Python. It is similar to [Pydantic](https://pydantic-docs.helpmanual.io/), but is much more strict. It's main abstract base class anticipates `to_data` and `from_data` methods. It provides the `serializable` decorator, which can be applied to data classes to generate those methods from the data class fields. It also provides the `abstract_serializable` decorator, which can be applied to sealed classes to generate those methods using a `"_type"` discriminator which switches on the `__subclasses__` of the base class. Serialite implements enough of the Pydantic interface so that it can be used by FastAPI.
