Description
-----------

Automatically update LyX and LaTeX tables

Background
----------

I found the original idea for this code when working with
[GSLab](https://github.com/gslab-econ). [The original tablefill](https://github.com/gslab-econ/gslab_python/blob/master/gslab_fill/tablefill.py)
was made to automatically update LyX files with Stata or Matlab output.
The workflow for the original file is as follows:

- Pre-fill tagged tables in LyX with placeholders.
- Output Stata matrices to file, with entries corresponding to placeholders.
- Run Python to update placeholders one by one with Stata output.

This project initially simply aimed to modify the script so it could
fill analogous templates written in LaTeX. However, over time it added
enough features that it became distinct. Several notes:

- The input tables can be generated by any program, as long as they
  write missing values as blank, a dot, or NA (the "." and "NA" are
  exceptions made for Stata and R).
- The format is to print a tag on the line prior to the table,
  `<tab:table_label>`, and then print a tab-delimited table (typically a
  matrix, but it can have rows or varying length).
- For compatibility with scripts using GSLab's tablefill, the script can
  be run from the command line or imported as a python module.

Features
--------

- Several error and consistency checks
    - Checks inputs are correct (names and type)
    - Checks if input files exist
    - Soft warning when placeholder outside proper environment
    - Soft warning when placeholder in environment with no label
    - Soft warning when placeholder in environment with unmatched label
    - Soft warning when more placeholders than table entries
- Can fill LaTeX templates
    - There can be several placeholders in one line
    - However, there must be at most one table line per code line
    - Environment must be a table environment, not tabular
    - Placeholders can be either `#` or `\#` (note the former is a
      special character in LaTeX so while the filled output will
      compile, the template will not).
    - Labels in LaTeX can be anywhere in the table environment
    - Can have several matches of the pattern in the same line
- Can be run from the command line directly or imported to a python script.
- Can choose whether to fill commented out lines.
- Adds placeholder modifiers % and ||; added placeholder type `#*#`
    - Adding the % before closing a placeholder causes tablefill to
      interpret it as a percentage (i.e. multiplies the number by 100).
    - Enclosing the contents of a placeholder in pipes, `||`, causes
      tablefill to take the absolute value of the input.
    - The placeholder `#*#` causes the entry to be interpreted as a
      p-value. The user can specify the p-value levels and symbols
      to replace them with (anything that LaTeX will compile can be
      passed). Default is `* 0.1, **0.05, ***0.01`.
- Basic LaTeX compilation via the `--compile` and/or `--bibtex` flags.
- Added XML-based ad-hoc table combination engine.
    - The user can combine arbitrary entries of existing tables in
      the provided templates to create new tables. This is useful if
      generating new tables is costly. It is also useful to quote
      table statistics in a summary elsewhere in the text (the LaTeX
      engine can parse the placeholders as long as they are in a table
      environment, which can be regular text).

Future releases will document the function's various features.

Setup
-----

```bash
git clone https://github.com/mcaceresb/tablefill
cd tablefiil
./tablefill.py --help
```

This was created specifically to run in a server that only had Python
2.6 available. The function should be compatible with Python 2.6, Python
2.7, and Python 3.X without requiring the installation of any additional
packages (the `--numpy-syntax` option is available if the function can
successfully run `import numpy`; however, `numpy` is not required for
normal use).

Documentation
-------------

### Main function

_**Forthcomming...**_

### XML combination engine

The following code in the `.tex` template will be interpreted by `tablefill.py`
```xml
% <tablefill-python tag = 'newtagname'>
%     tagname[rows1][subentries1],
%     tagname[rows2][subentries2],
%     othertagname[rows3][subentries3]
% </tablefill-python>
%
% <tablefill-python tag = 'othernewtagname' type = 'float'>
%     tagname[row1][subentry1] / tagname[row2][subentry2],
%     othertagname[rows3][subentries3]
% </tablefill-python>
```

The above is parsed as xml and will create 2 new tags: First it creates
`newtagname` with `[rows1][subentries1]` and `[rows2][subentries2]`
from `tagname` and `[rows3][subentries3]` from `othertagname`.
Second it creates `othernewtagname` with the result of the operation
`tagname[row1][subentry1] / tagname[row2][subentry2]` followed by the
entries from `othertagname[rows3][subentries3]`. Only scalar operations
are supported, and the type must be set to `float` or the parsing will exit
with error.

The syntax for `[rows][sub]` is python syntax for nested
lists: See [here](http://stackoverflow.com/questions/509211#509295).
Note that python uses 0-based indexing and that the combine engine uses
the raw matrices (i.e. before missing entries are stripped). Each matrix
is parsed as a list of lists, so
```html
1  2  3    [[1,  2,  3],
. -1 -2 --> [., -1, -2],
.  0  .     [.,  0,  .]]

[0]  or [-1]  --> [.,  0,  .]
[1]  or [-2]  --> [., -1, -2]
[2]  or [-3]  --> [1,  2,  3]
[1:] or [-2:] --> [.,  0,  .]
[1][1:3] or [-2][-2:] --> [-1, -2]
```

It is also possible to specify tables in a separate `.xml` file and
pass it to tablefill (there should be no leading `%` in this case) via
`--xml-tables` in the command line or `xml_tables` in a function call.

TODO
----

- [ ] Finish writing the documentation for the project.
- [ ] Update unit testing (tests are mostly copy/paste from the original tablefill).
- [ ] Build into a proper Python package.
- [ ] Look into doxygen.

License
-------

MIT
