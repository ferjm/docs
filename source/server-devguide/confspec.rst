Configuration files specification
=================================

All Services applications need to use the same configuration file format.
This document specifies it.


Syntax
======

The configuration file is a ini-based file. (See
http://en.wikipedia.org/wiki/INI_file for more details.) Variable names can be
assigned values, and grouped into sections. 

A line that starts with "#" is commented out. Empty lines are also removed.

Example::

    [section1] 
    # comment 
    name = value 
    name2 = "other value" 

    [section2] 
    foo = bar


Ini readers in Python, PHP and other languages understand this syntax.
Although, there are subtle differences in the way they interpret values and in
particular if/how they convert them.


Values conversion
=================

Here are a set of rules for converting values:

- If value is quoted with " chars, it's a string. This notation is useful to
  include "=" characters in the value. In case the value contains a " 
  character, it must be escaped with a "\" character.

- When the value is composed of digits and optionally prefixed by "-", it's
  tentatively converted to an integer or a long depending on the language. If
  the number exceeds the range available in the language, it's left as a 
  string.

- If the value is "true" or "false", it's converted to a boolean, or 0 and 
  1 when the language does not have a boolean type.

- A value can be an environment variable : "${VAR}" is replaced by the value 
  of VAR if found in the environment. If the variable is not found, an error 
  must be raised.

- A value can contain multiple lines. When read, lines are converted into a
  sequence of values. Each new line for a multiple lines value must start 
  with at least one space or tab character.


Examples::

    [section1] 
    # comment 
    a_flag = True 
    a_number = 1 
    a_string = "other=value"
    another_string = other value 
    a_list = one 
            two 
            three 

    user = ${USERNAME}


Extending a file
================

An INI file can extend another file. For this, a "DEFAULT" section must contain
an "extends" variable that can point to one or several INI files which will be
merged into the current file by adding new sections and values. 

If the file pointed to in "extends" contains section/variable names that already
exist in the original file, they will not override existing ones.

file_one.ini::

    [section1] 
    name2 = "other value" 

    [section2] 
    foo = baz 
    bas = bar

file_two.ini::

    [DEFAULT] 
    extends = file_one.ini    

    [section2] 
    foo = bar

Result::

    [section1] 
    name2 = "other value" 

    [section2] 
    foo = bar 
    bas = bar


To point to several files, the multi-line notation can be used::

    [DEFAULT] 
    extends = file_one.ini 
              file_two.ini

When several files are provided, they are processed sequentially. So if the
first one has a value that is also present in the second, the second one will
be ignored. This means that the configuration goes from the most specialized to
the most common.


Implementations
===============

There's one implementation in the core package of the Python server, but it
could be moved to a standalone distribution if another project wants to use it.

http://bitbucket.org/tarek/sync-core/src/tip/services/config.py

