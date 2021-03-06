[![Build Status](https://travis-ci.org/pierre-vigier/Perl6-Math-Matrix.svg?branch=master)](https://travis-ci.org/pierre-vigier/Perl6-Math-Matrix)
[![Build status](https://ci.appveyor.com/api/projects/status/github/pierre-vigier/Perl6-Math-Matrix?svg=true)](https://ci.appveyor.com/project/pierre-vigier/Perl6-Math-Matrix/branch/master)

NAME
====

Math::Matrix - create, compare, compute and measure 2D matrices

VERSION
=======

0.2.0

SYNOPSIS
========

Matrices are tables with rows and columns (index counting from 0) of numbers (Numeric type): 

    transpose, invert, negate, add, multiply, dot product, tensor product, determinant, 
    rank, trace, norm, 15 boolean properties, decompositions, submatrix, splice, map, reduce and more

DESCRIPTION
===========

Because the list based, functional toolbox of Perl 6 is not enough to calculate matrices comfortably, there is a need for a dedicated data type. The aim is to provide a full featured set of structural and mathematical operations that integrate fully with the Perl 6 conventions.

Matrices are readonly - all operations and functions do create new matrix objects. All methods return readonly data or deep clones - also the constructor does a deep clone of provided data. In that sense the library is thread safe.

All computation heavy properties will be calculated lazily and all below listed operators will be exported automatically.

This module is pure perl and we plan to use native shaped arrays one day.

METHODS
=======

  * constructors: new, new-zero, new-identity, new-diagonal, new-vector-product

  * accessors: cell, row, column, diagonal, submatrix

  * conversion: Bool, Numeric, Str, Array, list, list-rows, list-columns, gist, full, perl

  * boolean properties: is-zero, is-identity, is-square, is-diagonal, is-diagonally-dominant, is-upper-triangular, is-lower-triangular, is-invertible, is-symmetric, is-antisymmetric, is-unitary, is-self-adjoint, is-orthogonal, is-positive-definite, is-positive-semidefinite

  * numeric properties: cell-type, size, density, trace, determinant, rank, kernel, norm, condition

  * derived matrices: transposed, negated, conjugated, inverted, reduced-row-echelon-form

  * decompositions: decompositionLUCrout, decompositionLU, decompositionCholesky

  * list like ops: elems, cont, equal, map, map-row, map-column, map-cell, reduce, reduce-rows, reduce-columns

  * structural ops: move-row, move-column, swap-rows, swap-columns, splice-rows, splice-columns

  * matrix math ops: add, subtract, add-row, add-column, multiply, multiply-row, multiply-column, dotProduct, tensorProduct

  * operators: MM, +, -, *, **, ⋅, dot, ÷, ⊗, x, | |, || ||

Constructors
------------

Methods that create a new Math::Matrix object. The .new is optimized for speed and uniformity - the other are there for convenience and to optimize property calculation.

### new( [[...],...,[...]] )

The default constructor, takes arrays of arrays of numbers os only required parameter. Each second level array represents a row in the matrix. That is why their length has to be the same.

    say Math::Matrix.new( [[1,2],[3,4]] ) :

    1 2
    3 4

    Math::Matrix.new([<1 2>,<3 4>]); # does the same, WARNING: doesn't work with complex numbers
    Math::Matrix.new( [[1]] );       # one cell 1*1 matrix (exception where you don't have to mind comma)
    Math::Matrix.new( [[1,2,3],] );  # one row 1*3 matrix, mind the trailing comma
    Math::Matrix.new( [$[1,2,3]] );  # does the same, if you don't like trailing comma
    Math::Matrix.new( [[1],[2]] );   # one column 2*1 matrix

    use Math::Matrix :MM;            # tag :ALL works too
    MM [[1,2],[3,4]];                # shortcut

### new( "..." )

Alternatively you can define the matrix from a string, which makes most sense while using heredocs.

    Math::Matrix.new("1 2 \n 3 4"); # our demo matrix as string
    Math::Matrix.new: q:to/END/;    # indent as you wish
        1 2
        3 4
      END

    use Math::Matrix :ALL;          # 
    MM '1';                         # this case begs for a shortcut

### new-zero

This method is an constructor that returns an zero (sometimes empty) matrix (as checked by is-zero) of the size given in parameter. If only one parameter is given, the matrix is quadratic. All the cells are set to 0.

    say Math::Matrix.new-zero( 3, 4 ) :

    0 0 0 0
    0 0 0 0
    0 0 0 0

    say Math::Matrix.new-zero( 2 ) :

    0 0
    0 0

### new-identity

This method is a constructor that returns an identity matrix (as checked by is-identity) of the size given in the only and required parameter. All the cells are set to 0 except the top/left to bottom/right diagonale is set to 1.

    say Math::Matrix.new-identity( 3 ) :
      
    1 0 0
    0 1 0
    0 0 1

### new-diagonal

This method is a constructor that returns an diagonal matrix (as checked by is-diagonal) of the size given by count of the parameter. All the cells are set to 0 except the top/left to bottom/right diagonal, set to given values.

    say Math::Matrix.new-diagonal( 2, 4, 5 ) :

    2 0 0
    0 4 0
    0 0 5

### new-vector-product

This method is a constructor that returns a matrix which is a result of the matrix product (method dotProduct, or operator dot) of a column vector (first argument) and a row vector (second argument). It can also be understood as a tensor product of row and column.

    say Math::Matrix.new-vector-product([1,2,3],[2,3,4]) :

    2  3  4     1*2  1*3  1*4
    4  6  8  =  2*2  2*3  2*4
    6  9 12     3*2  3*3  3*4

Accessors
---------

Methods that return the content of selected elements (cells).

### cell

Gets value of element in row (first parameter) and column (second parameter). (counting always from 0)

    say Math::Matrix.new([[1,2],[3,4]]).cell(0,1) : 2

### row

Gets values of specified row (first required parameter) as a list.

    say Math::Matrix.new([[1,2],[3,4]]).row(0) : (1, 2)

### column

Gets values of specified column (first required parameter) as a list.

    say Math::Matrix.new([[1,2],[3,4]]).column(0) : (1, 3)

### diagonal

Gets values of diagonal elements as a list. 

    say Math::Matrix.new([[1,2],[3,4]]).diagonal : (1, 4)

### submatrix

Matrix built by a subset of cells of a given matrix.

The first and simplest usage is by choosing a cell (by coordinates like .cell()). Row and column of that cell will be removed. The remaining cells form the submatrix.

    say $m:

    1 2 3 4
    2 3 4 5
    3 4 5 6

    say $m.submatrix(1,2) :

    1 2 4
    3 4 6

If you provide two pairs of coordinates (row1, column1, row2, column2), these will be counted as left upper and right lower corner of and area inside the original matrix, which will the resulting submatrix.

    say $m.submatrix(1,1,1,3) :

    3 4 5

When provided with two lists of values (first for the rows - second for columns) a new matrix will be created with that selection of the old rows and columns in that new order.

    $m.submatrix((1,2),(3,2)):

    5 4
    6 5

In that example we have second and third row in previous order, but selcted only third and fourth column in reversed order.

Type Conversion And Output Flavour
----------------------------------

Methods that convert a matrix into other types or allow different views on the overall content.

### Bool

Conversion into Bool context. Returns False if matrix is zero (all cells equal zero as in is-zero), otherwise True.

    $matrix.Bool
    ? $matrix           # alias
    if $matrix          # matrix in Bool context too

### Numeric

Conversion into Numeric context. Returns number (amount) of cells (as .elems). Please note, only prefix a prefix + (as in: + $matrix) will call this Method. A infix (as in $matrix + $number) calls .add($number).

    $matrix.Numeric
    + $matrix           # alias op

### Str

All values separated by one whitespace, rows by horizontal line. It is called implicitly by put and print.

    say Math::Matrix.new([[1,2],[3,4]]).Str : 1 2 | 3 4

### Array

All cells as an array of arrays (basically what was put into new(...)).

    say Math::Matrix.new([[1,2],[3,4]]).Array : [[1 2] [3 4]]

### list

Same as $matrix.list-rows.flat.list : (1 2 3 4)

### list-rows

Returns a list of lists, reflecting the row-wise content of the matrix.

    say Math::Matrix.new( [[1,2],[3,4]] ).list-rows      : ((1 2) (3 4))
    say Math::Matrix.new( [[1,2],[3,4]] ).list-rows.flat : (1 2 3 4)

### list-columns

Returns a list of lists, reflecting the row-wise content of the matrix.

    say Math::Matrix.new( [[1,2],[3,4]] ).list-columns : ((1 3) (2 4))

### gist

Limited tabular view, optimized for shell output. Just cuts off excessive columns that do not fit into standard terminal and also stops after 20 rows. Several dots will hint that something is missing. It is implicitly called when:

    say $matrix;      # output when matrix has more than 100 cells

    1 2 3 4 5 ..
    3 4 5 6 7 ..
    ...

### full

Full tabular view (all rows and columns) for the shell or file output.

    say $matrix.full;

    1 2 3 4 5 11 12 13 14 15 21 22 23 24 25
    3 4 5 6 7 13 14 15 16 17 23 24 25 26 27
    5 6 7 8 9 15 16 17 18 19 25 26 27 28 29

### perl

Conversion into String that can reevaluated into the same object later using default constructor.

    my $clone = eval $matrix.perl;       # same as: $matrix.clone

Boolean Properties
------------------

These are mathematical properties a matrix can have or not.

### is-square

True if number of rows and colums are the same.

### is-zero

True if every cell (element) has value of 0.

### is-identity

True if every cell on the diagonal (where row index equals column index) is 1 and any other cell is 0.

    Example:    1 0 0
                0 1 0
                0 0 1

### is-upper-triangular

True if every cell below the diagonal (where row index is greater than column index) is 0.

    Example:    1 2 5
                0 3 8
                0 0 7

### is-lower-triangular

True if every cell above the diagonal (where row index is smaller than column index) is 0.

    Example:    1 0 0
                2 3 0
                5 8 7

### is-diagonal

    True if only cells on the diagonal differ from 0.
    .is-upper-triangular and .is-lower-triangular would also be True.

     Example:    1 0 0
                 0 3 0
                 0 0 7

### is-diagonally-dominant

    True if cells on the diagonal have a bigger (strict) or equal absolute value than the
    sum of the other absolute values in the column or row.

    if $matrix.is-diagonally-dominant {
    $matrix.is-diagonally-dominant(:!strict)   # same thing (default)
    $matrix.is-diagonally-dominant(:strict)    # diagonal elements (DE) are stricly greater (>)
    $matrix.is-diagonally-dominant(:!strict, :along<column>) # default
    $matrix.is-diagonally-dominant(:strict,  :along<row>)    # DE > sum of rest row
    $matrix.is-diagonally-dominant(:!strict, :along<both>)   # DE >= sum of rest row and rest column

### is-symmetric

True if every cell with coordinates x y has same value as the cell on y x. In other words: $matrix and $matrix.transposed (alias T) are the same.

    Example:    1 2 3
                2 5 4
                3 4 7

### is-antisymmetric

Means the transposed and negated matrix are the same.

    Example:    0  2  3
               -2  0  4
               -3 -4  0

### is-self-adjoint

A Hermitian or self-adjoint matrix is equal to its transposed and complex conjugated.

    Example:    1   2   3+i
                2   5   4
                3-i 4   7

### is-invertible

Is True if number of rows and colums are the same (.is-square) and .determinant is not zero. All rows or colums have to be independent vectors. Please use this method before $matrix.inverted, or you will get an exception.

### is-orthogonal

An orthogonal matrix multiplied (dotProduct) with its transposed derivative (T) is an identity matrix or in other words: .transposed and .inverted matrices are equal.

### is-unitary

An unitery matrix multiplied (dotProduct) with its concjugate transposed derivative (.conj.T) is an identity matrix, or said differently: the concjugate transposed matrix equals the inverted matrix.

### is-positive-definite

True if all main minors or all Eigenvalues are strictly greater zero.

### is-positive-semidefinite

True if all main minors or all Eigenvalues are greater equal zero.

Numeric Properties
------------------

Matrix properties that are expressed with a single number.

### size

List of two values: number of rows and number of columns.

    say $matrix.size();
    my $dim = min $matrix.size();

### density

Density is the percentage of cell which are not zero.

    my $d = $matrix.density( );

### trace

The trace of a square matrix is the sum of the cells on the main diagonal. In other words: sum of cells which row and column value is identical.

    my $tr = $matrix.trace( );

### determinant, alias det

If you see the columns as vectors, that describe the edges of a solid, the determinant of a square matrix tells you the volume of that solid. So if the solid is just in one dimension flat, the determinant is zero too.

    my $det = $matrix.determinant( );
    my $d = $matrix.det( );             # same thing
    my $d = |$matrix|;                  # operator shortcut

### rank

Rank is the number of independent row or column vectors or also called independent dimensions (thats why this command is sometimes calles dim)

    my $r = $matrix.rank( );

### kernel

Kernel of matrix, number of dependent rows or columns (rank + kernel = dim).

    my $tr = $matrix.kernel( );

### norm

    my $norm = $matrix.norm( );           # euclidian norm (L2, p = 2)
    my $norm = ||$matrix||;               # operator shortcut to do the same
    my $norm = $matrix.norm(1);           # p-norm, L1 = sum of all cells
    my $norm = $matrix.norm(p:<4>,q:<3>); # p,q - norm, p = 4, q = 3
    my $norm = $matrix.norm(p:<2>,q:<2>); # Frobenius norm
    my $norm = $matrix.norm('max');       # maximum norm - biggest absolute value of a cell
    $matrix.norm('row-sum');              # row sum norm - biggest abs. value-sum of a row
    $matrix.norm('column-sum');           # column sum norm - same column wise

### condition

Condition number of a matrix is L2 norm * L2 of inverted matrix.

    my $c = $matrix.condition( );

### cell-type

Matrix cells can be of type (Int), (Rat), (Num) or (Complex). The widest type of any cell will returned as type object.

Derivative Matrices
-------------------

Single matrices that can be computed with only our original matrix as input.

### transposed, alias T

Returns a new, transposed Matrix, where rows became colums and vice versa.

    Math::Matrix.new([[1,2,3],[3,4,6]]).transposed

    Example:   [1 2 3].T  =  1 4       
               [4 5 6]       2 5
                             3 6

### negated

    my $new = $matrix.negated();    # invert sign of all cells
    my $neg = - $matrix;            # works too

### conjugated, alias conj

    my $c = $matrix.conjugated();    # change every value to its complex conjugated
    my $c = $matrix.conj();          # works too (official Perl 6 name)

### inverted

Matrices that have a square form and a full rank can be inverted (see .is-invertible). Inverse matrix regarding to matrix multiplication. The dot product of a matrix with its inverted results in a identity matrix (neutral element in this group).

### reduced-row-echelon-form, alias rref

Return the reduced row echelon form of a matrix, a.k.a. row canonical form

    my $rref = $matrix.reduced-row-echelon-form();
    my $rref = $matrix.rref();

Decompositions
--------------

Methods that return lists of matrices, which in their product or otherwise can be recombined to the original matrix. In case of cholesky only one matrix is returned, becasue the other one is its transposed.

### decompositionLU

    my ($L, $U, $P) = $matrix.decompositionLU( );
    $L dot $U eq $matrix dot $P;         # True
    my ($L, $U) = $matrix.decompositionLUC(:!pivot);
    $L dot $U eq $matrix;                # True

$L is a left triangular matrix and $R is a right one Without pivotisation the marix has to be invertible (square and full ranked). In case you whant two unipotent triangular matrices and a diagonal (D): use the :diagonal option, which can be freely combined with :pivot.

    my ($L, $D, $U, $P) = $matrix.decompositionLU( :diagonal );
    $L dot $D dot $U eq $matrix dot $P;  # True

### decompositionLUCrout

    my ($L, $U) = $matrix.decompositionLUCrout( );
    $L dot $U eq $matrix;                # True

$L is a left triangular matrix and $R is a right one This decomposition works only on invertible matrices (square and full ranked).

### decompositionCholesky

This decomposition works only on symmetric and definite positive matrices.

    my $D = $matrix.decompositionCholesky( );  # $D is a left triangular matrix
    $D dot $D.T eq $matrix;                    # True

List Like Matrix Operations
---------------------------

Selection of methods that are also provided by Lists and Arrays and make also sense in context a 2D matrix. 

### elems

Number (count) of elements.

    say $matrix.elems();
    say +$matrix;                       # same thing

### cont

Asks if certain value is containted in cells (treating the matrix like a baggy set), or if there is one value within a cetain range.

    Math::Matrix.new([[1,2],[3,4]]).cont(1) :   True
    Math::Matrix.new([[1,2],[3,4]]).cont(5) :   False
    Math::Matrix.new([[1,2],[3,4]]).cont(3..7): True

    MM [[1,2],[3,4]] (cont) 1           # True too

### equal

Checks two matrices for equality. They have to be of same size and every element of the first matrix on a particular position has to be equal to the element (on the same position) of the second matrix.

    if $matrixa.equal( $matrixb ) {
    if $matrixa ~~ $matrixb {

### map

Like the built in map it iterates over all elements, running a code block. The results for a new matrix.

    say Math::Matrix.new([[1,2],[3,4]]).map(* + 1) :

    2 3
    4 5

### map-row

Map only specified row (row number is first parameter).

    say Math::Matrix.new([[1,2],[3,4]]).map-row(1, {$_ + 1}) :

    1 2
    4 5

### map-column

    say Math::Matrix.new([[1,2],[3,4]]).map-column(1, {0}) :

    1 0
    3 0

### map-cell

Changes value of one cell on row (first parameter) and column (second) with code block (third, $_ or $^... is previous value).

    say Math::Matrix.new([[1,2],[3,4]]).map-cell(0, 1, {$_  * 3}); :

    1 6
    3 4

### reduce

Like the built in reduce method, it iterates over all elements and joins them into one value, by applying the given operator or method to the previous result and the next element. I starts with the cell [0][0] and moving from left to right in the first row and continue with the first cell of the next row.

    Math::Matrix.new([[1,2],[3,4]]).reduce(&[+]): 10
    Math::Matrix.new([[1,2],[3,4]]).reduce(&[*]): 10

### reduce-rows

Reduces (as described above) every row into one value, so the overall result will be a list. In this example we calculate the sum of all cells in a row:

    say Math::Matrix.new([[1,2],[3,4]]).reduce-rows(&[+]): (3, 7)

### reduce-columns

Similar to reduce-rows, this method reduces each column to one value in the resulting list:

    say Math::Matrix.new([[1,2],[3,4]]).reduce-columns(&[*]): (3, 8)

Structural Matrix Operations
----------------------------

Methods that reorder the rows and columns, delete some or even add new. The accessor .submatrix is also useful for that purpose.

### move-row

    Math::Matrix.new([[1,2,3],[4,5,6],[7,8,9]]).move-row(0,1);  # move row 0 to 1
    Math::Matrix.new([[1,2,3],[4,5,6],[7,8,9]]).move-row(0=>1); # same

    1 2 3           4 5 6
    4 5 6    ==>    1 2 3
    7 8 9           7 8 9

### move-column

    Math::Matrix.new([[1,2,3],[4,5,6],[7,8,9]]).move-column(2,1);
    Math::Matrix.new([[1,2,3],[4,5,6],[7,8,9]]).move-column(2=>1); # same

    1 2 3           1 3 2
    4 5 6    ==>    4 6 5
    7 8 9           7 9 8

### swap-rows

    Math::Matrix.new([[1,2,3],[4,5,6],[7,8,9]]).swap-rows(2,0);

    1 2 3           7 8 9
    4 5 6    ==>    4 5 6
    7 8 9           1 2 3

### swap-columns

    Math::Matrix.new([[1,2,3],[4,5,6],[7,8,9]]).swap-columns(0,2);

    1 2 3           3 2 1
    4 5 6    ==>    6 5 4
    7 8 9           9 8 7

### splice-rows

Like the splice for lists: the first two parameter are position and amount (optional) of rows to be deleted. The third and alos optional parameter will be an array of arrays (line .new would accept), that fitting row lengths. These rows will be inserted before the row with the number of first parameter. The third parameter can also be a fitting Math::Matrix.

    Math::Matrix.new([[1,2],[3,4]]).splice-rows(0,0, Math::Matrix.new([[5,6],[7,8]]) ); # aka prepend
    Math::Matrix.new([[1,2],[3,4]]).splice-rows(0,0,                  [[5,6],[7,8]]  ); # same result

    5 6
    7 8 
    1 2
    3 4

    Math::Matrix.new([[1,2],[3,4]]).splice-rows(1,0, Math::Matrix.new([[5,6],[7,8]]) ); # aka insert
    Math::Matrix.new([[1,2],[3,4]]).splice-rows(1,0,                  [[5,6],[7,8]]  ); # same result

    1 2
    5 6
    7 8
    3 4

    Math::Matrix.new([[1,2],[3,4]]).splice-rows(1,1, Math::Matrix.new([[5,6],[7,8]]) ); # aka replace
    Math::Matrix.new([[1,2],[3,4]]).splice-rows(1,1,                  [[5,6],[7,8]]  ); # same result

    1 2
    5 6 
    7 8

    Math::Matrix.new([[1,2],[3,4]]).splice-rows(2,0, Math::Matrix.new([[5,6],[7,8]]) ); # aka append
    Math::Matrix.new([[1,2],[3,4]]).splice-rows(2,0,                  [[5,6],[7,8]]  ); # same result
    Math::Matrix.new([[1,2],[3,4]]).splice-rows(-1,0,                 [[5,6],[7,8]]  ); # with negative index

    1 2 
    3 4     
    5 6
    7 8

### splice-columns

Same as splice-rows, just horizontally.

    Math::Matrix.new([[1,2],[3,4]]).splice-columns(2,0, Math::Matrix.new([[5,6],[7,8]]) ); # aka append
    Math::Matrix.new([[1,2],[3,4]]).splice-columns(2,0,                  [[5,6],[7,8]]  ); # same result
    Math::Matrix.new([[1,2],[3,4]]).splice-columns(-1,0,                 [[5,6],[7,8]]  ); # with negative index

    1 2  ~  5 6  =  1 2 5 6
    3 4     7 8     3 4 7 8

Matrix Math Operations
----------------------

Matrix math methods on full matrices and also parts (for gaussian table operations).

### add

    Example:    1 2  +  5    =  6 7 
                3 4             8 9

                1 2  +  2 3  =  3 5
                3 4     4 5     7 9

    my $sum = $matrix.add( $matrix2 );  # cell wise addition of 2 same sized matrices
    my $s = $matrix + $matrix2;         # works too

    my $sum = $matrix.add( $number );   # adds number from every cell 
    my $s = $matrix + $number;          # works too

### subtract

Works analogous to add - it's just for convenance.

    my $diff = $matrix.subtract( $number );   # subtracts number from every cell (scalar subtraction)
    my $sd = $matrix - $number;               # works too
    my $sd = $number - $matrix ;              # works too

    my $diff = $matrix.subtract( $matrix2 );  # cell wise subraction of 2 same sized matrices
    my $d = $matrix - $matrix2;               # works too

### add-row

Add a vector (row or col of some matrix) to a row of the matrix. In this example we add (2,3) to the second row. Instead of a matrix you can also give as parameter the raw data of a matrix as new would receive it.

    Math::Matrix.new([[1,2],[3,4]]).add-row(1,[2,3]);

    Example:    1 2  +       =  1 2
                3 4    2 3      5 7

### add-column

    Math::Matrix.new([[1,2],[3,4]]).add-column(1,[2,3]);

    Example:    1 2  +   2   =  1 4
                3 4      3      3 7

### multiply

In scalar multiplication each cell of the matrix gets multiplied with the same number (scalar). In addition to that, this method can multiply two same sized matrices, by multipling the cells with the came coordinates from each operand.

    Example:    1 2  *  5    =   5 10 
                3 4             15 20

                1 2  *  2 3  =   2  6
                3 4     4 5     12 20

    my $product = $matrix.multiply( $number );   # multiply every cell with number
    my $p = $matrix * $number;                   # works too

    my $product = $matrix.multiply( $matrix2 );  # cell wise multiplication of same size matrices
    my $p = $matrix * $matrix2;                  # works too

### multiply-row

Multiply scalar number to each cell of a row.

    Math::Matrix.new([[1,2],[3,4]]).multiply-row(0,2);

    Example:    1 2  * 2     =  2 4
                3 4             3 4

### multiply-column

Multiply scalar number to each cell of a column.

    Math::Matrix.new([[1,2],[3,4]]).multiply-row(0,2);

    Example:    1 2   =  2 2
                3 4      6 4
            
               *2

### dotProduct

Matrix multiplication of two fitting matrices (colums left == rows right).

    Math::Matrix.new( [[1,2],[3,4]] ).dotProduct(  Math::Matrix.new([[2,3],[4,5]]) );

    Example:    2  3
           *    4  5

         1 2   10 13  =  1*2+2*4  1*3+2*5
         3 4   22 29     3*2+4*4  3*3+4*5

    my $product = $matrix1.dotProduct( $matrix2 )
    my $c = $a dot $b;              # works too as operator alias
    my $c = $a ⋅ $b;                # unicode operator alias

    A shortcut for multiplication is the power - operator **
    my $c = $a **  3;               # same as $a dot $a dot $a
    my $c = $a ** -3;               # same as ($a dot $a dot $a).inverted
    my $c = $a **  0;               # created an right sized identity matrix

### tensorProduct

The tensor product between a matrix a of size (m,n) and a matrix b of size (p,q) is a matrix c of size (m*p,n*q). All matrices you get by multiplying an element (cell) of matrix a with matrix b (as in $a.multiply($b.cell(..,..)) concatinated result in matrix c. (Or replace in a each cell with its product with b.)

    Example:    1 2  *  2 3   =  1*[2 3] 2*[2 3]  =  2  3  4  6
                3 4     4 5        [4 5]   [4 5]     4  5  8 10
                                 3*[2 3] 4*[2 3]     6  9  8 12
                                   [4 5]   [4 5]     8 15 16 20

    my $c = $matrixa.tensorProduct( $matrixb );
    my $c = $a x $b;                # works too as operator alias
    my $c = $a ⊗ $b;                # unicode operator alias

Operators
=========

The Module overloads or uses a range of well and less known ops. +, -, * and ~~ are commutative.

The only exception is MM, a shortcut to create a matrix. That has to be importet explicitly with the tag :MM or :ALL.

    my $a   = +$matrix               # Num context, amount (count) of cells
    my $b   = ?$matrix               # Bool context, True if any cell has a none zero value
    my $str = ~$matrix               # String context, matrix content as data structure

    $matrixa ~~ $matrixb             # check if both have same size and they are cell wise equal

    my $sum =  $matrixa + $matrixb;  # cell wise sum of two same sized matrices
    my $sum =  $matrix  + $number;   # add number to every cell

    my $dif =  $matrixa - $matrixb;  # cell wise difference of two same sized matrices
    my $dif =  $matrix  - $number;   # subtract number from every cell
    my $neg = -$matrix               # negate value of every cell

    my $p   =  $matrixa * $matrixb;  # cell wise product of two same sized matrices
    my $sp  =  $matrix  * $number;   # multiply number to every cell

    my $dp  =  $a dot $b;            # dot product of two fitting matrices (cols a = rows b)
    my $dp  =  $a ⋅ $b;              # dot product, unicode alias
    my $dp  =  $a ÷ $b;              # alias to $a dot $b.inverted

    my $c   =  $a **  3;             # $a to the power of 3, same as $a dot $a dot $a
    my $c   =  $a ** -3;             # alias to ($a dot $a dot $a).inverted
    my $c   =  $a **  0;             # creats an right sized identity matrix

    my $tp  =  $a x $b;              # tensor product 
    my $tp  =  $a ⊗ $b;              # tensor product, unicode alias

     | $matrix |                     # determinant
    || $matrix ||                    # Euclidean (L2) norm

    MM [[1]]                         # a new matrix

Author
======

  * Pierre VIGIER

  * Herbert Breunung

Contributors
============

License
=======

Artistic License 2.0

