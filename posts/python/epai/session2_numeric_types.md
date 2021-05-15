# Understanding Python Numeric Types

#### int data-type

* Internally, Integers are represented using base-2 (binary) digits.
* It can store signed and unsigned numbers
* Here `int` object uses a variable number of bits, hence it can use 4 bytes, 8 bytes, 12 bytes, etc. But it is limited to the amount of memory available

#### float data-type

* Float data is represented in IEEE 754 double-precision binary float format. It uses fixed number of bytes and hence are rational numbers. They can be represented as base-10 integers and fractions
* Float can be compared to be equal (i.e equality test) using a `math` library's `isclose` function. This function gives us the flexibility to compare based on a tolerance value. It can either be compared using `abs_tol` i.e. absolute tolerance or using `rel_tol` i.e. relative tolerance
* `abs_tol` is a tolerance value calculated using absolute difference of two floats. For example, if `a` & `b` are two floats then: `tol = abs(a) - abs(b)` . So if absolute difference is less than `tol`, then those two floats are close and hence `is_close` returns `True`
* `rel_tol` is a tolerance value calcuated using `rel_tol` percentage of max of two floats (absolute). For example, if `a` & `b` are two floats then: `tol = rel_tol/100 * max(abs(a), abs(b))`. So if absolute difference is less than  `tol`, then those two floats are close and hence `is_close` returns `True`

#### Base conversion

* Python provides in-built functions like `bin()`, `hex()`, `oct()` to convert decimal numbers to binary, hexadecimal, octal formats respectively
* A binary number is prefixed with `0b`, hexadecimal number is prefixed with `0x` and octal number with `0o`

#### Math functions

* Python provides various math functions like `round()` , `floor()`, `ceil()`, `trunc()`
* `round()` function rounds off a decimal to a the closest integer. It also takes precision as an argument to control the decimal values. In case of ties, we would expect numbers to round away from zero. But this doesn't happen. Python doesn bankers rounding i.e. rounds to the nearest value, with ties rounded to the nearest value with an even least significant digit
* `trunc()` function truncates a floating point number to just the integer part. For example `trunc(1.5)` will be perform truncation on`1.5` to `1`
* `floor()` function converts a floating point number to closest integer away from zero (But this is not always the case in python!)
* `ceil()` function is opposite of floor, where floating point number is converted to closest integer closer to zero

#### Fractions & Decimals

* Because of intricacies of representing floating point number in IEEE format, they are no very accurately represented
* Hence python has Fractions and Decimals libraries where floating point can be very closely represented in accurate format