Object Oriented Field Guide
================
Ian Flores

> DISCLAIMER: This is going to be more of a worksheet/live coding session than a presentation by itself.

> DISCLAIMER: Most of the code here is taken directly from the Advanced R book or is a modification of code inside that book. The book is available at this link: <http://adv-r.had.co.nz/OO-essentials.html>

Object-Oriented Programming
===========================

Let's get started with an example
---------------------------------

-   Think of a blueprint of a building
    -   How about a specific building?
        -   What does this building does?
        -   What does this building have?

Can you think of another hierarchy example?

How does this fit into OO Programming
-------------------------------------

-   Think of a blueprint of a building - **Class**
    -   How about a specific building? - **Object**
        -   What does this building does? - **Methods**
        -   What does this building have? - **Attributes**

Object-Oriented Systems in R
============================

R has (3) three OO systems:

-   S3
-   S4
-   Reference Classes

Base System
-----------

-   Every R object is a C `struct`
-   Base types are not an object system
-   To determine an object's base type, use the `typeof` function.

``` r
typeof(1)
```

    ## [1] "double"

``` r
typeof('x')
```

    ## [1] "character"

-   Functions are of type closure.

``` r
f <- function() {}
typeof(f)
```

    ## [1] "closure"

S3
--

The majority of objects used in R are S3 objects. How can we know if the object we are using is in the S3 system?

``` r
library(pryr)
```

    ## 
    ## Attaching package: 'pryr'

    ## The following object is masked _by_ '.GlobalEnv':
    ## 
    ##     f

``` r
df <- data.frame(num = 1:10, letter = letters[1:10])
otype(df)
```

    ## [1] "S3"

``` r
tib <- tibble::as.tibble(df)
otype(tib)
```

    ## [1] "S3"

However in S3, the methods belong to the functions, not to the objects.

``` r
## You do
dplyr::filter(tib, num == 1)
```

    ## # A tibble: 1 x 2
    ##     num letter
    ##   <int> <fct> 
    ## 1     1 a

``` r
## You DON'T do
tib.filter(num == 1)
```

### Function Generics

When you call a function such as `mean` you are calling a generic which will then match the object to the corresponding method.

If you want to see all the methods a function has, you can use:

``` r
methods('mean')
```

    ## [1] mean.Date     mean.default  mean.difftime mean.POSIXct  mean.POSIXlt 
    ## see '?methods' for accessing help and source code

What if you have an object and you want to see which functions you can apply to it?

``` r
methods(class = 'data.frame')
```

    ##  [1] [             [[            [[<-          [<-           $            
    ##  [6] $<-           aggregate     anyDuplicated as.data.frame as.list      
    ## [11] as.matrix     by            cbind         coerce        dim          
    ## [16] dimnames      dimnames<-    droplevels    duplicated    edit         
    ## [21] filter        format        formula       head          initialize   
    ## [26] intersect     is.na         Math          merge         na.exclude   
    ## [31] na.omit       Ops           plot          print         prompt       
    ## [36] rbind         row.names     row.names<-   rowsum        setdiff      
    ## [41] setequal      show          slotsFromS3   split         split<-      
    ## [46] stack         str           subset        summary       Summary      
    ## [51] t             tail          transform     type.convert  union        
    ## [56] unique        unstack       within       
    ## see '?methods' for accessing help and source code

### Creating our own objects

``` r
nums <- structure(list(x = 1:10), class = 'custom_num')
class(nums)
```

    ## [1] "custom_num"

How do I test if an object is of a certain class?

``` r
inherits(nums, 'custom_num')
```

    ## [1] TRUE

However, if using already created class is better to use the default way as converting between classes does not always work.

``` r
as.data.frame(nums)
```

``` r
data.frame(x = 1:10)
```

    ##     x
    ## 1   1
    ## 2   2
    ## 3   3
    ## 4   4
    ## 5   5
    ## 6   6
    ## 7   7
    ## 8   8
    ## 9   9
    ## 10 10

Let's see if we can create a new data\_scientist object, with class `Person` with at least 2 attributes.

``` r
## Your Code Here
data_scientist <- NULL
if (inherits(data_scientist, 'Person')){
    print("Success!")
}
```

S4
--

S4 objects are not that common in the R ecosystem. However, it is a rich OO system for development in R.

How to identify S4 objects?

``` r
library(stats4)
y <- c(1:10)
nLL <- function(lambda) - sum(dpois(y, lambda, log = TRUE))
fit <- mle(nLL, start = list(lambda = 5), nobs = length(y))
```

``` r
## From the base package
isS4(fit)
```

    ## [1] TRUE

How to identify from which class inherits an object?

``` r
is(fit)
```

    ## [1] "mle"

``` r
is(fit, 'mle')
```

    ## [1] TRUE

### Creating our own S4 objects

S4 objects have 3 key properties: \* A name \* A named list of slots \* A string giving the class inheritance

``` r
setClass('r_programmer', slots = list(name = 'character', packages = 'character'))
```

``` r
(ian <- new('r_programmer', name = 'Ian', packages = c('ggplot2', 'leaflet')))
```

    ## An object of class "r_programmer"
    ## Slot "name":
    ## [1] "Ian"
    ## 
    ## Slot "packages":
    ## [1] "ggplot2" "leaflet"

``` r
## Create your own S4 class and an object. 
```

How do we access the package `leaflet`?

``` r
ian
```

    ## An object of class "r_programmer"
    ## Slot "name":
    ## [1] "Ian"
    ## 
    ## Slot "packages":
    ## [1] "ggplot2" "leaflet"

``` r
ian@packages
```

    ## [1] "ggplot2" "leaflet"

``` r
ian@packages[2]
```

    ## [1] "leaflet"

#### Brief parentheses about inheritance

If an S4 object contains an S3 class or a base type, it will have a `.Data` slot.

``` r
setClass('RangedNumeric', contains = 'numeric', slots = list(min = 'numeric', max = 'numeric'))
```

``` r
(rn <- new('RangedNumeric', 1:5, min = 1, max = 5))
```

    ## An object of class "RangedNumeric"
    ## [1] 1 2 3 4 5
    ## Slot "min":
    ## [1] 1
    ## 
    ## Slot "max":
    ## [1] 5

``` r
rn@.Data
```

    ## [1] 1 2 3 4 5

S4 comes with a `standardGeneric()` function to assign the generic call of a function.

Hadley Wickham added some reources in his book to go more in depth into the S4 system.

Reference Classes
-----------------

\*\* Very similar to classes in `Python` \*\*

If we want to create a Reference class we use the `setRefClass()` function

``` r
vehicle <- setRefClass("vehicle")
vehicle$new()
```

    ## Reference class object of class "vehicle"

How about if we want to add attributes?

``` r
(vehicle <- setRefClass("vehicle", 
                       fields = list(tires = "numeric", doors = "numeric")))
```

    ## Generator for class "vehicle":
    ## 
    ## Class fields:
    ##                       
    ## Name:    tires   doors
    ## Class: numeric numeric
    ## 
    ## Class Methods: 
    ##      "field", "trace", "getRefClass", "initFields", "copy", "callSuper", 
    ##      ".objectPackage", "export", "untrace", "getClass", "show", 
    ##      "usingMethods", ".objectParent", "import"
    ## 
    ## Reference Superclasses: 
    ##      "envRefClass"

#### Let's create some vehicle objects

``` r
bike <- vehicle$new(tires = 2, doors = 0)
car <- vehicle$new(tires = 4, doors = 4)
```

What if I want to create a copy of bike?

``` r
bike_copy <- bike$copy()
```

However, if you modify the original object, the copy will not be affected as well.

``` r
bike$tires <- 3
bike
```

    ## Reference class object of class "vehicle"
    ## Field "tires":
    ## [1] 3
    ## Field "doors":
    ## [1] 0

``` r
bike_copy
```

    ## Reference class object of class "vehicle"
    ## Field "tires":
    ## [1] 2
    ## Field "doors":
    ## [1] 0

As compared to S3 and S4 methods are part of objects, just as in Python or C++.

``` r
vehicle <- setRefClass("Vehicle", 
                       fields = list(tires = 'numeric', doors = 'numeric'),
                       methods = list(
                           steal_all_tires = function(x) {
                               tires <<- 0
                           },
                           seal_one_door = function(x) {
                               doors <<- doors - 1
                           }, 
                           convert_vehicle = function(x){
                               tires <<- tires + 1
                           }
                       ))
```

``` r
bike <- vehicle$new(tires = 2, doors = 0)
bike
```

    ## Reference class object of class "Vehicle"
    ## Field "tires":
    ## [1] 2
    ## Field "doors":
    ## [1] 0

``` r
bike$convert_vehicle()
bike
```

    ## Reference class object of class "Vehicle"
    ## Field "tires":
    ## [1] 3
    ## Field "doors":
    ## [1] 0

How can I inherit from a previous class?

``` r
buses <- setRefClass("Buses",
                     contains = "Vehicle",
                     fields = list(people_onboard = 'numeric'),
                     methods = list(
                         add_people = function(x){
                             people_onboard <<- people_onboard + x
                         }
                             )
                    )
```

``` r
bus_480 <- buses$new(tires = 8, doors = 2, people_onboard = 25)
bus_480
```

    ## Reference class object of class "Buses"
    ## Field "tires":
    ## [1] 8
    ## Field "doors":
    ## [1] 2
    ## Field "people_onboard":
    ## [1] 25

How can we test if `bus_480` is a Reference Class?

``` r
(is(bus_480, "refClass"))
```

    ## [1] TRUE

``` r
otype(bus_480)
```

    ## [1] "RC"
