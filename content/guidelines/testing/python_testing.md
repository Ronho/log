# Testing in Python

In this document you will find best practices, questions I had during testing and some examples.


## pytest vs unittest

The two most popular frameworks for testing in Python are pytest and unittest. I will try to give you some reasons why you should choose one over the other. But first, if you decide to use one or the other, try to stick to it as much as possible.

unittest is part of the standard Python library, which makes it a natural choice if you do not have too much experience with Python or testing itself. It provides everything you need to get started. It also follows a class-like structure for organising related tests based on setup and teardown methods.

Pytest is a third party library that provides some features such as test parameterisation, making it an ideal choice for larger projects. It is often used in a function style approach, but also supports classes and setup and teardown methods for grouping tests.

Since I believe most developers will use pytest, and I am most familiar with it, the following sections will focus on pytest.


## Concepts

In this section you will find some basic concepts to familiarise yourself with. As some of these are pytest-specific and already well explained by the pytest authors, I will only provide links to helpful resources.

The links are for version 8.0.X of pytest to avoid future changes, but you should have a look at the latest stable pytest release.

- [Fixtures](https://docs.pytest.org/en/8.0.x/explanation/fixtures.html#about-fixtures "Fixtures")
- [Conftest File](https://docs.pytest.org/en/8.0.x/how-to/fixtures.html#scope-sharing-fixtures-across-classes-modules-packages-or-session "Sharing Fixtures")
- [Fixture Factories](https://docs.pytest.org/en/8.0.x/how-to/fixtures.html#factories-as-fixtures "Fixture Factories")
- [Parameterized Fixtures](https://docs.pytest.org/en/8.0.x/how-to/fixtures.html#parametrizing-fixtures)
- [Parameterized Tests](https://docs.pytest.org/en/8.0.x/how-to/parametrize.html)
- [Mark Tests](https://docs.pytest.org/en/8.0.x/how-to/mark.html)

Additional links:
- [unittest mock](https://docs.python.org/3.11/library/unittest.mock.html "Python 3.11 unittest.mock")
- [Test Doubles in Python](https://blog.szymonmiks.pl/p/test-doubles-in-python-unit-tests/ "Test Doubles")

## Style Guide

### Folder Structure

Here, I will assume that you have some kind of `tests` folder within your project. The following structure incorporates the general testing hierarchy, thereby, enabling running each of these independently.
```
| tests/
    | __init__.py
    | component/
        | __init__.py
        | conftest.py
        | ...
    | integration/
        | __init__.py
        | conftest.py
        | database_connector/
            | __init__.py
            | conftest.py
            | test_one.py
            | ...
        | ...
    | unit/
        | __init__.py
        | conftest.py
        | test_single_example_test.py
        | example_module_one/
            | __init__.py
            | conftest.py
            | test_class_one.py
            | ...
        | ...
```

In the case of a monolithic architecture, `component` would be replaced with `system`. Note that there is no `end-to-end' test folder, as these will most likely be spread across multiple Python projects/repositories.

The files below the `unit`, `integration`, and `component` folders should follow the overall project file structure.

### Naming

#### Tests

Tests must start with `test_` followed by the name of the functionality, the state under test, and the expected behaviour - all in lower_snake_case.

:::{admonition} Example
:class: note
```python
def div(numerator: float, denominator: float) -> float:
    if denominator == 0:
        raise ValueError('Zero division!')
    return numerator/denominator
```
:::

:::{admonition} Yes
:class: tip

```python
def test_div_denominator_zero_raises_exception():
    ...
```

- name of the functionality: `div`
- state under test: `denominator_zero`
- expected behaviour: `raises_exception`
:::

:::{admonition} No
:class: error

```python
def div_throws_exception_test():
    ...
```
```python
def test_div():
    ...
```
:::

#### Parameterised Tests

In the case of parameterised tests, the state to be tested and the expected behaviour can be transferred to the test case id. The `pytest.param` parameter should be preferred over the `ids` parameter as it is closer to the parameters. In repetitive cases, the `ids` parameter can be used for templating, or the test name can represent the common behaviour of the tests.

:::{admonition} Example
:class: note
```python
def div(numerator: float, denominator: float) -> float:
    if denominator == 0:
        raise ValueError('Zero division!')
    return numerator/denominator
```
:::

:::{admonition} Yes
:class: tip

```python
@pytest.mark.parametrize(
    ('numerator', 'denominator', 'expected'),
    [
        pytest.param(8, 4, 2, id='numerator_8_denominator_4_returns_2'),
        pytest.param(9, 3, 3, id='numerator_9_denominator_3_returns_3'),
    ]
)
def test_div(numerator, denominator, expected):
    ...
```

```python
@pytest.mark.parametrize(
    ('numerator', 'denominator', 'expected'),
    [(8, 4, 2), (9, 3, 3)]
    ids=lambda n,d,e: f'numerator_{n}_denominator_{d}_returns_{e}'
)
def test_div(numerator, denominator, expected):
    ...
```

```python
@pytest.mark.parametrize(
    ('numerator', 'denominator', 'expected'),
    [(8, 4, 2), (9, 3, 3)]
)
def test_div_returns_expected_value(numerator, denominator, expected):
    ...

def test_div_denominator_zero_raises_exception():
    ...
```
:::

:::{admonition} No
:class: error

```python
@pytest.mark.parametrize(
    ('numerator', 'denominator', 'expected'),
    [(8, 4, 2), (9, 3, 3)]
    ids=lambda n,d,e: f'numerator_{n}_denominator_{d}_returns_{e}'
)
def test_div_returns_expected_value(numerator, denominator, expected):
    ...
```
:::

#### Fixtures

The name of a fixture should describe the value it provides. Fixtures should be documented. The structure of the documentation should follow the structure of the whole project.

:::{admonition} Example
:class: note
```python
class Animal():
    def __init__(self, name: str):
        self.name = name
```
:::

:::{admonition} Yes
:class: tip

```python
@pytest.fixture
def animal_tiger() -> Animal:
    return Animal('tiger')

@pytest.fixture
def animal_wolf() -> Animal:
    return Animal('wolf')
```
:::

:::{admonition} No
:class: error

```python
@pytest.fixture
def animal_a() -> Animal:
    return Animal('tiger')

@pytest.fixture
def animal_b() -> Animal:
    return Animal('wolf')
```
:::

### Structure

#### Separated Parameterised Tests

Parameterised tests should be preferred to single test cases. However, checks for exceptions, none or boolean values, and other types should be separated.

:::{admonition} Example
:class: note
```python
def do_something(a: int) -> int | None:
    if (a > 3):
        return None
    return a + 1
```
:::

:::{admonition} Yes
:class: tip

```python
@pytest.mark.parametrize(('a', 'expected'), [(4, 5), (81, 82)])
def test_do_something_returns_expected_value(a, expected):
    actual = do_something(a)
    assert actual == expected

@pytest.mark.parametrize('a', [(3), (1)])
def test_do_something_returns_none(a):
    actual = do_something(a)
    assert actual is None
```
:::

:::{admonition} No
:class: error

```python
@pytest.mark.parametrize(('a', 'expected'), [(4, 5), (81, 82), (3, None)])
def test_do_something_returns_expected_value(a, expected):
    actual = do_something(a)
    assert actual == expected
```
:::

#### Using Fixtures with Parameterised Tests

When using parameterised tests and object factories, use fixture factories or parameterised fixtures rather than functions.

#### Grouping Tests using Classes

Tests should be written in a functional format. Tests for the same method or function should be placed side by side. However, classes are allowed in the following cases:

1. Tests share the same parameterisation.
1. Logical grouping of tests, e.g. of different classes in the same file.

Instead of using xunit-style setup and teardown methods, fixtures should be used whenever possible, as they can be shared across files.

### Sharing between Files

#### Sharing Fixtures

If you need to share a fixture between two or more files, the lowest common `conftest` file should be used. Fixtures should not be shared between different test levels (unit, integration, component)!

### Execution

#### Marking Tests

Slow tests should be marked as such with `@pytest.mark.slow`.

#### Order

Tests should be performed in parallel or in random order.