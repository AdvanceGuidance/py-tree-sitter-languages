==================================================
Python Bindings for Tree Sitter with All Languages
==================================================

Binary Python wheels for all tree sitter languages.

`py-tree-sitter`_ is a fantastic library that provides Python bindings for the
even more fantastic `tree-sitter`_ parsing library.

`py-tree-sitter-languages`_ provides binary Python wheels for all tree sitter
languages. The binary wheels remove the need to download and compile support
for individual languages.

.. _`py-tree-sitter-languages`: https://github.com/grantjenks/py-tree-sitter-languages

Status
===========

This library is unmaintained, as an alternative you can use .. _`tree-sitter-language-pack`: https://github.com/Goldziher/tree-sitter-language-pack

Install
=======

::

   pip install tree_sitter_languages

Source installs are not supported. To see how the binary wheels are built, look
at:

1. `setup.py` — Python package setup.

2. `repos.txt` — Text file that contains a list of included language repositories and their commit hashes.

3. `build.py` — Python script to download and build the language repositories.

4. `.github/workflows/release.yml` — GitHub action to invoke `cibuildwheel`_ and
   release to PyPI.

.. _`cibuildwheel`: https://github.com/pypa/cibuildwheel


Usage
=====

.. code:: python

   from tree_sitter_languages import get_language, get_parser

   language = get_language('python')
   parser = get_parser('python')

That's the whole API!

Refer to `py-tree-sitter`_ for the language and parser API. Notice the
``Language.build_library(...)`` step can be skipped! The binary wheel includes
the language binary.

.. _`py-tree-sitter`: https://github.com/tree-sitter/py-tree-sitter


Demo
====

Want to know something crazy? Python lacks multi-line comments. Whhaaa!?!

It's really not such a big deal. Instead of writing

.. code:: python

   """
   My awesome
   multi-line
   comment.
   """

Simply write

.. code:: python

   # My awesome
   # multi-line
   # comment.

So multi-line comments are made by putting multiple single-line comments in
sequence. Amazing!

Now, how to find all the strings being used as comments?

Start with some example Python code

.. code:: python

   example = """
   #!shebang
   # License blah blah (Apache 2.0)
   "This is a module docstring."

   a = 1

   '''This
   is
   not
   a
   multiline
   comment.'''

   b = 2

   class Test:
       "This is a class docstring."

       'This is bogus.'

       def test(self):
           "This is a function docstring."

           "Please, no."

           return 1

   c = 3
   """

Notice a couple things:

1. Python has module, class, and function docstrings that bare a striking
   resemblance to the phony string comments.

2. Python supports single-quoted, double-quoted, triple-single-quoted, and
   triple-double-quoted strings (not to mention prefixes for raw strings,
   unicode strings, and more).

Creating a regular expression to capture the phony string comments would be
exceedingly difficult!

Enter `tree-sitter`_

.. code:: python

   from tree_sitter_languages import get_language, get_parser

   language = get_language('python')
   parser = get_parser('python')

Tree-sitter creates an abstract syntax tree (actually, a `concrete syntax
tree`_) and supports queries

.. code:: python

   tree = parser.parse(example.encode())
   node = tree.root_node
   print(node.sexp())

.. _`concrete syntax tree`: https://stackoverflow.com/q/1888854/232571

Look for statements that are a single string expression

.. code:: python

   stmt_str_pattern = '(expression_statement (string)) @stmt_str'
   stmt_str_query = language.query(stmt_str_pattern)
   stmt_strs = stmt_str_query.captures(node)
   stmt_str_points = set(
       (node.start_point, node.end_point) for node, _ in stmt_strs
   )
   print(stmt_str_points)

Now, find those statement string expressions that are actually module, class,
or function docstrings

.. code:: python

   doc_str_pattern = """
       (module . (comment)* . (expression_statement (string)) @module_doc_str)

       (class_definition
           body: (block . (expression_statement (string)) @class_doc_str))

       (function_definition
           body: (block . (expression_statement (string)) @function_doc_str))
   """
   doc_str_query = language.query(doc_str_pattern)
   doc_strs = doc_str_query.captures(node)
   doc_str_points = set(
       (node.start_point, node.end_point) for node, _ in doc_strs
   )

With the set of string expression statements and the set of docstring
statements, the locations of all phony string comments is

.. code:: python

   comment_strs = stmt_str_points - doc_str_points
   print(sorted(comment_strs))


License
=======

Copyright 2022-2023 Grant Jenks

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License.  You may obtain a copy of the
License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

The project also includes the following other projects distributed in binary
form:

* https://github.com/tree-sitter/tree-sitter — licensed under the MIT License.

* https://github.com/WhatsApp/tree-sitter-erlang — licensed under
  the Apache License, Version 2.0.

* https://github.com/Azganoth/tree-sitter-lua — licensed under the MIT
  License.

* https://github.com/Wilfred/tree-sitter-elisp — licensed under the MIT
  License.

* https://github.com/alemuller/tree-sitter-make — licensed under the MIT
  License.

* https://github.com/camdencheek/tree-sitter-dockerfile — licensed under the
  MIT License.

* https://github.com/camdencheek/tree-sitter-go-mod — licensed under the MIT
  License.

* https://github.com/elixir-lang/tree-sitter-elixir — licensed under the
  Apache License, Version 2.0.

* https://github.com/elm-tooling/tree-sitter-elm — licensed under the MIT
  License.

* https://github.com/fwcd/tree-sitter-kotlin — licensed under the MIT License.

* https://github.com/ganezdragon/tree-sitter-perl — licensed under the MIT
  License.

* https://github.com/ikatyang/tree-sitter-markdown — licensed under the MIT
  License.

* https://github.com/ikatyang/tree-sitter-yaml — licensed under the MIT
  License.

* https://github.com/jiyee/tree-sitter-objc — licensed under the MIT License.

* https://github.com/m-novikov/tree-sitter-sql — licensed under the MIT
  License.

* https://github.com/r-lib/tree-sitter-r — licensed under the MIT License.

* https://github.com/rydesun/tree-sitter-dot — licensed under the MIT License.

* https://github.com/slackhq/tree-sitter-hack — licensed under the MIT
  License.

* https://github.com/theHamsta/tree-sitter-commonlisp — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-bash — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-c — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-c-sharp — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-cpp — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-css — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-embedded-template — licensed
  under the MIT License.

* https://github.com/tree-sitter/tree-sitter-go — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-haskell — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-html — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-java — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-javascript — licensed under the
  MIT License.

* https://github.com/tree-sitter/tree-sitter-jsdoc — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-json — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-julia — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-ocaml — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-php — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-python — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-ql — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-regex — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-ruby — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-rust — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-scala — licensed under the MIT
  License.

* https://github.com/dhcmrlchtdj/tree-sitter-sqlite - licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-toml — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-tsq — licensed under the MIT
  License.

* https://github.com/tree-sitter/tree-sitter-typescript — licensed under the
  MIT License.

* https://github.com/stsewd/tree-sitter-rst - licensed under the MIT License.

* https://github.com/MichaHoffmann/tree-sitter-hcl - licensed under the
  Apache License, Version 2.0.

* https://github.com/stadelmanma/tree-sitter-fortran - licensed under the MIT
  License.

* https://github.com/stadelmanma/tree-sitter-fixed-form-fortran - licensed under
  the MIT License.

* https://github.com/yutaro-sakamoto/tree-sitter-cobol - licensed under
  the MIT License.

.. _`tree-sitter`: https://tree-sitter.github.io/
