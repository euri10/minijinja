<div align="center">
  <img src="https://github.com/mitsuhiko/minijinja/raw/main/artwork/logo.png" alt="" width=320>
  <p><strong>MiniJinja for Python: a powerful template engine for Rust and Python</strong></p>

[![Build Status](https://github.com/mitsuhiko/minijinja/workflows/Tests/badge.svg?branch=main)](https://github.com/mitsuhiko/minijinja/actions?query=workflow%3ATests)
[![License](https://img.shields.io/github/license/mitsuhiko/minijinja)](https://github.com/mitsuhiko/minijinja/blob/main/LICENSE)
[![Crates.io](https://img.shields.io/crates/d/minijinja.svg)](https://crates.io/crates/minijinja)
[![rustc 1.61.0](https://img.shields.io/badge/rust-1.61%2B-orange.svg)](https://img.shields.io/badge/rust-1.61%2B-orange.svg)
[![Documentation](https://docs.rs/minijinja/badge.svg)](https://docs.rs/minijinja)

</div>

`minijinja-py` is an experimental binding of
[MiniJinja](https://github.com/mitsuhiko/minijinja) to Python.  It has somewhat
limited functionality compared to the Rust version.  These bindings use
[maturin](https://www.maturin.rs/) and [pyo3](https://pyo3.rs/).

You might want to use MiniJinja instead of Jinja2 when the full feature set
of Jinja2 is not required and you want to have the same rendering experience
of a data set between Rust and Python.

With these bindings MiniJinja can render some Python objects and values
that are passed to templates, but there are clear limitations with regards
to what can be done.

To install MiniJinja for Python you can fetch the package [from PyPI](https://pypi.org/project/minijinja/):

```
$ pip install minijinja
```

## Basic API

The basic API is hidden behind the `Environment` object.  It behaves almost entirely
like in `minijinja` with some Python specific changes.  For instance instead of
`env.set_debug(True)` you use `env.debug = True`.  Additionally instead of using
`add_template` or attaching a `source` you either pass a dictionary of templates
directly to the environment or a `loader` function.

```python
from minijinja import Environment

env = Environment(templates={
    "template_name": "Template source"
})

# alternatively

def loader(template_name):
    if template_name == "template_name":
        return "Template source"
    return None

env = Environment(loader=loader)
```

To render a template you can use the `render_template` method:

```python
result = env.render_template('template_name', var1="value 1", var2="value 2")
print(result)
```

## Utility

MiniJinja attemps a certain level of compatibiliy with Jinja2, but it does not
try to achieve this at all costs.  As a result you will notice that quite a few
templates will refuse to render with MiniJinja despite the fact that they probably
look quite innocent.  It is however possible to write templates that render to the
same results for both Jinja2 and MiniJinja.  This raises the question why you might
want to use MiniJinja.

The main benefit would be to achieve the exact same results in both Rust and Python.
Additionally MiniJinja has a stronger sandbox than Jinja2 and might perform ever so
slightly better in some situations.  However you should be aware that due to the
marshalling that needs to happen in either direction there is a certain amount of
loss of information.

## Runtime Behavior

MiniJinja uses it's own runtime model which is not matching the Python
runtime model.  As a result there are clear gaps in beahvior between the
two and only limited effort is made to bridge them.  For instance you will
be able to call some methods of types, but for instance builtins such as
dicts and lists do not expose their methods on the MiniJinja side.  This
means that it's very intentional that if you pass a dictionary to MiniJinja,
the Python `.items()` method is unavailable.

Here is what this means for some basic types:

* Python dictionaries and lists (as well as other objects that behave as sequences)
  appear in the MiniJinja side as native lists.  They do not expose any specific
  other behavior and when they move back to the Python side they will appear as basic
  lists.  Specifically this means that a tuple (which does not exist in MiniJinja)
  when moving from Python to MiniJinja turns into a list and will remain a list when
  it moves back.
* Python objects are represented in MiniJinja similarly to dicts, but they retain all
  their meaningful Python APIs.  This means they stringify via `__str__` and they
  allow the MiniJinja code to call their non-underscored methods.  Note that there is
  no extra security layer in use at the moment so take care of what you pass there.
* MiniJinja's python binding understand what `__html__` is when it exists on a string
  subclass.  This means that a `markupsafe.Markup` object will appear as safe string in
  MiniJinja.  This information can also flow back to Python again.

## Sponsor

If you like the project and find it useful you can [become a
sponsor](https://github.com/sponsors/mitsuhiko).

## License and Links

- [Documentation](https://docs.rs/minijinja/)
- [Examples](https://github.com/mitsuhiko/minijinja/tree/main/examples)
- [Issue Tracker](https://github.com/mitsuhiko/minijinja/issues)
- [MiniJinja Playground](https://mitsuhiko.github.io/minijinja-playground/)
- License: [Apache-2.0](https://github.com/mitsuhiko/minijinja/blob/main/LICENSE)
