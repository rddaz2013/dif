dif.py -- DIF file handler in Python
Copyright 2001, 2013 Chris Gonnerman
All Rights Reserved

Redistribution and use in source and binary forms, with or without 
modification, are permitted provided that the following conditions
are met:

Redistributions of source code must retain the above copyright
notice, this list of conditions and the following disclaimer. 

Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution. 

Neither the name of the author nor the names of any contributors
may be used to endorse or promote products derived from this software
without specific prior written permission. 

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
AUTHOR OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  
-----------------------------------------------------------------------------

This package includes the module dif.py, which implements a basic reader for the "Navy DIF" data file format. The current version is effectively read-only; there is no feature for saving, remodelling, etc. the DIF object data.

After importing dif, open the DIF file and pass the file handle to the DIF class like so:

> `diffile = dif.DIF(filehandle)`

If there are major formatting errors, `dif.DIFError` will be raised (minor errors including sanity checks are ignored). Otherwise, diffile will be an instance of the DIF class. It has the following data members:

| `.header`  | Dictionary of header entities. Names have been shifted to lowercase for ease of handling. |
| `.data`  | List of tuples. In this case, a Python tuple is equivalent to a DIF tuple. |
| `.vectors`  | List of vector names (actually column names). Only LABEL entries which refer to row #0 will be listed, and a default set is created when the VECTORS entry is processed. As a sanity check (of sorts), a LABEL with a vector number which is out of range results in an IndexError. |

Each .header value is either a tuple (indicating that the entity appeared exactly once) or a list of tuples (indicating multiple entities of the same name, such as LABEL or COMMENT). Each .header tuple is of the form

> `(n, n, "s")`

in other words, the two numeric values and the string value.

The DIF object also acts as a (read-only) sequence. When indexed directly the returned object is a DIF Tuple (not exactly the same as a Python tuple) converted to a dictionary, with column (vector) names as keys and column values as values. Remember that you can get the DIF Tuple as a tuple by indexing the .data member.

At the bottom of this README is the information on the DIF file format which I have used to create this module.<a name="installation"></a>

* * *

**Installation**

dif.py has a built-in distutils installer. Just do this:

> `python dif.py install`

to install it. You may need to be root or have other privileges depending on your OS.

* * *

**The DIF File Structure**

_NOTE! The information below was pulled by me from wotsit.org's list of data file formats. I can't identify the original author, and the data in the file I received was mangled; I have reformatted it and cleaned it up. If this has resulted in the omission of important data, I apologize; but the information below is the basis for the dif.py module file._

DIF (Data Interchange Format) is a program-independent method of storing data. DIF files are ASCll text files. The format uses a brief line length to make the files as universally compatible as possible with application software, languages, operating systems and computer hardware.

A DIF file is oriented towards row-and-column data, such as a spreadsheet or data-base manager might produce. Because individual programs may "rotate" the rows and columns, DIF uses the terms vector and tuple. You may generally interpret vector as column and tuple as row. DIF files contain two sections: a file header and a data section.

**The DIF Header**

There are four required entries in the DIF header, and a number of optional entries. The format of all header entries is

| `< topic >` |
| `< vector # > , < numerical value >` |
| `" < string value > "` |

where

> `< topic >`  is a "token," generally 32 characters or fewer.
> 
> `< vector # >`  is O if specifying the entire file.
> 
> `< numerical value >`  is O unless a value is specified.
> 
> `< string value >`  is "" (double quotations with no space between) if it is not used.

The first required item in a DIF file is the title. For a typical spreadsheet, this would look like:

| `TABLE` |
| `0, < version # >` |
| `" < title > "` |

where

> `< version # >` is 1.
> 
> `< title >`  is the title of the table.

The next required item is the vector count. This specifies the number of vectors (columns). Its format is

| `VECTORS` |
| `0, < count >` |
| `""` |

where

> `< count > ` is the number of vectors. This entry may appear anywhere in the header, but must appear before any entries that specify vector numbers.

The third required item is the tuple count. This specifies the length of the vectors (the number of rows). Its format is

| `TUPLES` |
| `0, < count >` |
| `""` |

where

> `< count >` is the number of tuples.

The final required header item is DATA, which specifies the division of the header information from the data proper. DATA must be the last header item. Its format is:

| `DATA` |
| `0,0` |
| `""` |

**Optional Header Items**

Other header entries are optional. DIF Clearinghouse has included optional entries. Some are "standard" as a result of their being used in particular software products. The optional header entry items are: label, comment, field size, time series, significant values, and measure.

**COMMENT** - Permits enhanced description of a vector

| `COMMENT` |
| `< vector # > , < line # >` |
| `" < comment > "` |

where

> `< vector # >`  is the commented vector.
> 
> `< line # >`  may refer to more than one line, i.e. multiline comments.
> 
> `< comment >`  is the comment string.

**LABEL** - Labels a specific < vector # > , < line # >

| `LABEL` |
| `< vector # > , < line # >` |
| `" < label > "` |

where

> `< vector # >` is the label
> 
> `< line # >` allows for labeling more than one
> 
> `< label >` is the label string.

**SIZE** - Allocates fixed field sizes for each vector

| `SIZE` |
| `< vector # > , < # bytes >` |
| `""` |

where

> `< vector # >` is the vector being sized.
> 
> `< # bytes >` is the size.

**TRUELENGTH** - Indicates the portion of a vector that contains significant values:

| `TRUELENGTH` |
| `< vector # > , < length >` |
| `""` |

where

> `< vector # >` is the specified vector.
> 
> `< length >` is the length of that vector that contains significant values.

**UNITS** - Units of measure for a given vector

| `UNITS` |
| `< vector # > ,0` |
| `" < name > "` |

where

> `< vector # >` is the specified vector.
> 
> `< name >` is the name string of the units to be applied.

**DISPLAYUNITS** - Units in which a given vector should be displayed:

| `DISPLAYUNITS` |
| `< vector # >,0` |
| `" < name > "` |

where

> `< vector # >` is the specified vector.
> 
> `< name >`  is the name string of the units used to display the vector. (This may be different from the units used to measure the vector.)

**DIF Data Section**

The data section is organized in a series of tuples. Data within each tuple is organized in vector sequence. Essentially, using a spreadsheet as a data model, this means one data entry to a cell, in ascending column position, then by ascending row position.

There are two "special data values," BOT (Beginning of Tuple) and EOD (End of Data). BOT marks the start of each tuple. EOD terminates the DIF file.

Each data entry is organized in the following manner:

| `< type indicator >, < numerical value >` |
| `< string value >` |

where

| `< type indicator >`  | is one of three different indicators: |

| -1 | special data value |
 `< numeric value >`  | is O |
 `< string value>`  | is BOT, EOD |
| O | numeric data (signed decimal number) |
 `< numeric value >`  | is numeric data |
 `< string value >`  | is one of the Value Indicators (see below) |
| 1 | string data |
 `< numeric value >`  | is O |
 `< string value >`  | is string data |

**Value Indicator**

There are five value indicators to use as the < string value > when the <type indicator> = 0:

| `V`  | value |
| `NA`  | not available; | < numeric value > must be O |
| `ERROR`  | error condition; | < numeric value > must be O |
| `TRUE`  | < numeric value > is 1 |
| `FALSE`  | < numeric value > is O |


