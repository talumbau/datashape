===============
DataShape Types
===============

In addition to `defining the grammar <grammar.rst>`_, datashape specifies
a standard set of types and some properties those types should have.
Type constructors can be classified as ``dimension`` or ``dtype``, and a
datashape is always composed of zero or more dimensions followed by
a dtype.

Dimension Types
===============

Fixed Dimension
---------------

``fixed[4]``

A dimension whose size is specified. This is the most common
dimension type used in Blaze, and ``4 * int32`` is syntactic sugar for
``fixed[4] * int32`` in datashape syntax.

Var Dimension
-------------

``var``

A dimension whose size may be different across instances.
A common use of this is a ragged array like ``4 * var * int32``.

Strided Dimension
-----------------

``strided``

A dimension whose size is fixed across the array, but whose
size is no known ahead of time. This is not commonly used in Blaze,
but is in DyND where it is how NumPy-like dimensions are created.
For example, a two dimensional strided array has type
``strided * strided * int32``, and does not bake in particular
dimension sizes like with fixed dimensions in ``10 * 15 * int32``.

Type Variables
--------------

``typevar['DimName']``

Constructs a type variable. ``DimName`` is syntactic sugar for
``typevar['DimName']``. This is used for pattern matching types,
particularly for function prototypes. For example the
datashape ``(M * N * int32) -> N * int32`` accepts an input
with two dimensions that are type variables, and returns a
one dimensional array using one of those dimension types.

Ellipsis
--------

``ellipsis``

Constructs an ellipsis for matching multiple broadcast dimensions.
``...`` is syntactic sugar for ``ellipsis``.

``ellipsis['DimVar']``

Constructs a named ellipsis for matching multiple broadcast dimensions.
``Dim...`` is syntactic sugar for ``ellipsis['Dim']``.

DTypes
======

Boolean Type
------------

``bool``

A boolean type which may take on two values, ``True`` and ``False``.
In Blaze and DyND, this is stored as a single byte which may take on
the values 1 and 0.

Default Integer
---------------

``int``

This is an alias for ``int32``.

Arbitrary-Precision Integer
---------------------------

``bignum`` or ``bigint``

An integer type which has no minimum or maximum value. This is not
implemented in Blaze or DyND presently and the final name for it hasn't
been locked down.

Signed Integer Types
--------------------

``int8``
``int16``
``int32``
``int64``
``int128``

Integer types whose behavior follows that of twos-complement integers
of the given size.

Unsigned Integer Types
----------------------

``uint8``
``uint16``
``uint32``
``uint64``
``uint128``

Integer types whose behavior follows that of unsigned integers of
the given size.

Platform-Specific Integer Aliases
---------------------------------

``intptr``
``uintptr``

Aliases for ``int##`` and ``uint##`` where ## is the size of a pointer type on
the platform.

Default Floating Point
----------------------

``real``

This is an alias for ``float64``.

Binary Floating Point
---------------------

``float16``
``float32``
``float64``
``float128``

Binary floating point types as defined by IEEE 754-2008. Each type
corresponds to the ``binary##`` type defined in the standard.

Note that ``float128`` is not a C/C++ ``long double``, except on such
platforms where they coincide. NumPy defines a ``float128`` on
some platforms which is not IEEE ``binary128``, and is thus different
from DataShape's type of the same name on those platforms.

TODO: Support for C/C++ ``long double``. This is tricky given that
      DataShape intends to be cross-platform, and maybe some inspiration
      can be taken from HDF5 for specifying them.

Decimal Floating Point
----------------------

``decimal32``
``decimal64``
``decimal128``

Decimal floating point types as defined by IEEE 754-2008. These are
not implemented in Blaze or DyND presently.

Default Complex
----------------------

``complex``

This is an alias for ``complex[float64]``.

Complex
-------

``complex[float32]``

Constructs a complex number type from a real number type.

Void
----

``void``

A type which can store no data. It is not intended to be constructed
in concrete arrays, but to allow for things like function prototypes
with ``void`` return type.

String
------

``string``

A unicode string that can be arbitrarily sized. In Blaze and DyND, this
is a UTF-8 encoded string.

``string[16]``

A unicode string in a UTF-8 fixed-sized buffer. The string is
zero-terminated, but as in NumPy, all bytes may be filled with character
data so the buffer is not valid as a C-style string.

``string['utf16']``

A unicode string that can be arbitrarily sized, using the specified
encoding. Valid values for the encoding are ``'ascii'``, ``'utf8'``,
``'utf16'``, ``'utf32'``, ``'ucs2'``, and ``'cp###'`` for valid
code pages.

``string[16, 'utf16']``

A unicode string in a fixed-size buffer of the specified number of bytes,
encoded as the requested encoding.  The string is
zero-terminated, but as in NumPy, all bytes may be filled with character
data so the buffer is not valid as a C-style string.

Character
---------

``char``

A value which contains a single unicode code point. Typically stored as
a 32-bit integer.

Bytes
-----

``bytes``

An arbitrarily sized blob of bytes. This like ``bytes`` in Python 3.

``bytes[16]``

A fixed-size blob of bytes. This is not zero-terminated as in the
``string`` case, it is always exactly the specified number of bytes.

Categorical
-----------

``categorical[type=string, values=['low', 'medium', 'high']]``

Constructs a type which stores values compactly as small integers
that are indexes into the list of values provided.

JSON
----

``json``

A unicode string which is known to contain values represented as JSON.

Records
-------

``struct[['name', 'age', 'height'], [string, int, real]]``

Constructs a record type with the given field names and types.
``{name: string, age: int}`` is syntactic sugar for
``struct[['name', 'age'], [string, int]]``.

Tuples
------

``tuple[[string, int, real]]``

Constructs a tuple type with the given types. ``(string, int)``
is syntactic sugar for ``tuple[[string, int]]``.

Function Prototype
------------------

``funcproto[[string, int], bool]``

Constructs a function prototype with the given argument and return types.
``(string, int) -> bool`` is syntactic sugar for
``funcproto[[string, int], bool]``.

Type Variables
--------------

``typevar['DTypeName']``

Constructs a type variable. ``DTypeName`` is syntactic sugar for
``typevar['DTypeName']``. This is used for pattern matching types,
particularly for function prototypes. For example the
datashape ``(T, T) -> T`` accepts any types as input, but requires
they have the same types.

Option/Missing Data
-------------------

``option[float32]``

Constructs a type based on the provided type which may have missing
values.

Pointer
-------

``pointer[target=2 * 3 * int32]``

Constructs a type whose value is a pointer to values of the target type.

Date, Time, and DateTime
------------------------

``date``

A type which represents a single date in the Gregorian calendar.
In DyND and Blaze, it is represented as a 32-bit signed integer offset
from the date ``1970-01-01``.

``time``
``time[tz='UTC']``

Represents a time in an abstract day (no time zone), or a day
with the specified time zone.

Stored as a 64-bit integer offset from midnight,
stored as ticks (100 ns units).

``datetime``
``datetime[tz='UTC']``

Represents a moment in time in an abstract time zone if no time
zone is provided, otherwise stored as UTC but representing time
in the specified time zone.

Stored as a 64-bit signed integer offset from
``0001-01-01T00:00:00`` in ticks (100 ns units), the "universal
time scale" from the ICU library. Follows the POSIX convention
of ignoring leap seconds.

http://userguide.icu-project.org/datetime/universaltimescale

``units['second', int64]``

A type which represents a value with the units and type specified.
Initially only supporting time units, to support the datetime
functionality without adding a special "timedelta" type.

Initial valid units are: '100*nanosecond' (ticks as in the datetime storage),
'microsecond', 'millisecond', 'second', 'minute', 'hour', 'day'.
Need to decide on valid shortcuts in a context with more physical units,
probably by adopting conventions from a good physical units library.

``timetz``
``datetimetz``

Represents a time/datetime with the time zone attached to the data. Not
implemented in Blaze/DyND.

