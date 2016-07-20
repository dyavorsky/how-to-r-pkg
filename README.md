# Notes from Hadley's R Packages Book
Dan Yavorsky  


# Minimum Package Creation with VCS and C++


### Create a Package

- To create a package from scratch, click "New Package" > "New Directory" > "R Package"
- To associate a new package with an existing R project, use `devtools::use_rstudio("path/to/package")`


### Connect to GitHub

First, track the package with git:

- When creating a package, check the "git" box
- Or run `git init` in a shell after you `cd` into the package's directory

Second, create a GitHub repo:

- On GitHub, create a new repo
- Give it the same title as the package name
- Use the "description" field for the package subtitle

Last, connect the local project to the GitHub repo:

- Follow the prompt on GitHub, it will look something like this

```
git remote add origin git@github.com:dyavorsky/HowToPkg.git  
git push -u origin master
```

To work with Git/GitHub:

- Add them to `.gitignore` to keep them untracked (can do this with a right-click in the Git pane of RStudio)
- **Ctrl-Alt-M** opens the commit window
- **Ctrl-Alt-D** diffs files

You'll want a markdown README file at the top level of the directory so that GitHub renders it on the repo's landing page. It's best to add the `README.Rmd` and `README.html` to `.gitignore` so there's no confusion on which gets rendered by GitHub.


### Package Pieces

- R code goes in the `R/` directory. You _cannot_ put subdirectories in there.
- C++ code goes in the `src/` directory
- Documentation goes in the `man/` directory (done automatically with roxygen comments)
- Datasets go in the `data/` directory
- Other files like `CITATION` or raw data (`extdata`) go in the `inst` (installed files) directory
- `DESCRIPTION` and `NAMESPACE` files live at the top level
- The top level is also a good place for a `README` file, especially if the package is on GitHub


### Package Development Workflow

To load the development version of the package, run `devtools::load_all()`. The easier way, however, is to press **Ctrl-Shift-L** in RStudio, which saves all open files and loads the package. 

You'll mostly be writing functions. That workflow is:

- Edit an R File
- Press **Ctrl-Shift-L**
- Use function at command line

To omit a directory or file when building a package, include a RegEx in the `.Rbuildignore` file. For this package, the RMarkdown version of this ReadMe file (`README.Rmd`) has been added to `.Rbuildignore`. The line reads `^ReadMe\.Rmd$`.

When you document functions with roxygen (see below), that workflow is:

- Add roxygen comments to your .R files
- Press **Ctrl-Shift-D** to convert roxygen comments to .Rd files
- Preview documentation with ?


### DESCRIPTION File

This file uses Debian Control Format (DCF), which means each line has a field name and a value, separate by a colon. When values span multiple lines, they need to be indented.

Title and Description appear on the CRAN download page:

- `Title:` is a one-line description (65 char max, no punctuation, no formatting)
- `Description:` is a one-paragraph description (80 char/line max, indent subsequent lines)

This file is where you put other package dependencies (not in a `library()` or `require()` statement somewhere in your code!)

- `Imports: dplyr, ggvis` means these packages _must_ be present for your package to work
- `Suggests: dplyr, ggvis` is weaker, see Hadley's _R Packages_ page 35

You can manually add information to the DESCRIPTION file, or you can use `devtools::use_package("dplyr")` to do it for you.

It's a good idea to require a minimum version of other packages if they're dependencies. So your DESCRIPTION file might look something like this 

```
Imports:
  dplyr (>= 0.3.0.1),
  ggvis (>= 0.2)
Suggests:
  MASS (>= 7.3.0)
```


### NAMESPACE File

This file control hows your package interacts with the rest of R, importantly making it self-contained. It controls _imports_, which define how a function in one package finds a function in another; and _exports_, which define which of your functions are available outside of your package.

We will not generate the NAMESPACE file by hand, but will instead use roxygen2, as we do for function/dataset documentation (see below). The NAMESPACE workflow will be the same as the documentation workflow:

- Add roxygen comments to your .R files
- Press **Ctrl-Shift-D** to convert roxygen comments to .Rd files
- Look at the NAMESPACE file and run tests to check that the specification is correct

For a function to be usable outside of your package, you must **export** it. Do this by putting `#' @export` in a roxygen block. Roxygen will handle the specific "export" code in the NAMESPACE file for you. (For datasets that live in `data/`, they don't use the usual namespace mechanism and don't need to be exported.)

If your package uses functions from another package, you must **import** them (i.e., that package must be loaded or loaded-and-attached). Here's the recommended way to do so:

- If you are using just a few functions from another package, put that package name in the `Imports:` field in the `DESCRIPTION` file, and then call those functions using `::` (e.g., `bayesm::breg()`)
- If you are using certain functions repeatedly, you can avoid `::` with `@importFrom pkg fun` as a roxygen comment
- If you use many functions from a package repeatedly, then as a last resort, use `@ import pkg` as a roxygen comment


## C++ 

To set up your package with Rcpp:

- Select "Package with Rcpp" when creating a new package with RStudio's dropdown
- Or run `devtools::use_rcpp()`

This does the following:

- Creates an `src/` directory to hold the `.cpp` files
- Adds `Rcpp` to the `LinkingTo` and `Imports` fields in the `DESCRIPTION` file
- Sets up a `.gitignore` file 
- Tells you the two roxygen tags you need to add to your package (`#' @useDynLib you-pkg-name` and `#' @importFrom Rcpp sourceCpp`) for the namespace imports

The workflow will be:

- Create/edit C++ files and generate necessary namespace modifications (**Ctrl-Shift-D**)
- Click the "Build & Reload" button or press **Ctrl-Shift-B**
- Use your function at the command line

"Build and Reload" does a lot of work behind the scenes. One of those things is to call `Rcpp::compileAttributes()`, which inspects `.cpp` functions looking for _attributes_ of the form `// [[Rcpp::export]]`. When it finds one, it generates the code needed to make the function available in R, and creates `src/RcppExports.cpp` and `R/RcppExports.R`. _Never_ modify these by hand.

Two important parts of each C++ file are:

1. The header

    `#include <Rcpp.h>`  
    `using namespace Rcpp;`
    
1. Making the function available in R

    `// [[Rcpp::export]]`




### A bit about package development

For package development, it's helpful to know about the 5 states that a package can be in. 

  1. _Source_: Just a directory of files and subdirectories (i.e., the development version of a package)
  1. _Bundled_: A compressed, single file (.tar.gz) of the package; not useful on its own, but a useful intermediary step
  1. _Binary_: Platform-specific; how packages are distributed via CRAN
  1. _Installed_: A decompressed binary package
  1. _In-Memory_: to use a package, you need to load it into memory

The command line tool `R CMD INSTALL` powers all package installation. The R package `devtools` provides functions that are wrappers for `R CMD INSTALL` so that you can call this command from inside of R. 

- `devtools::install()` is a wrapper for `R CMD INSTALL`
- `devtools::build()` is a wrapper for `R CMD BUILD` that turns source packages into bundles
- `devtools::install_github()` downloads a source package from GitHub, runs `build()` to make vignettes, and then uses `R CMD INSTALL` to do the install
- `devtools::install_url()` and `devtools::install_bitbucket()` work similarly






*****






# Additional Topics


### Documentation Workflow with Roxygen2

The four basic steps are:

1. Add roxygen comments to your .R files
1. Run `devtools::document()` (or press **Ctrl-Shift-D**^[This must be enabled in Package Options > Build Tools] in RStudio) to convert roxygen comments to .Rd files
1. Preview documentation with ?
1. Repeat

Documenting R code with roxygen2 involves putting the help documentation directly into the .R code files using roxygen comments, which start with `#'`. In C++ files, roxygen comments are `//'`

Roxygen comments come in blocks. A block is all the documentation for a specific function and it goes _before_ the function. Lines must wrap at 80 characters. Thus one .R file can have multiple documented functions. 

Each block is made up of an introduction and tags with the format `@tagname details`. The intro has a title, description, and (otionally) details. Then you include tags for documentation elements. 

**Common tags are:**

_Functions_
  
- `@param name description`
- `@examples`
- `@return description`
  
_Navigation_

- `@seealso`: point to other places 
    - on web (`\url{}`)
    - in your package (`\code{\link{functionname}}`)
    - in another package (`\code{\link[packagename]{functionname}`)
- `@family` when all functions in a family should link to each other

_Finding Documentation_

- `@aliases alias1 alias2` adds aliases which can be used with `?`
- `@keywords keyword1 keyword2` must be taken from a predefined list found in `file.path(R.home("doc"), "KEYWORDS")`

_Datasets_

- `@format` for providing an overview of a dataset
- `@source` to provide details from where you got a dataset

_Other_

- `@section title` for long documentation that require section breaks

**Rd character formatting**

_Character_

- `\emph{italics}`
- `\strong{bold}`
- `\code{rfunction}`
    
_Links_

- `\code{\link{function}}`
- `\code{\link[package]{function}}`
- `\link[destination]{name}`
- `\url{http://google.com}`
- `\href{http://google.com}{google}`
- `\email{dyavorsky@gmail.com}`

_Lists_

- `\enumerate{}`
- `\itemize{}`
- `\describe{}`

_Math_

- `\eqn{}`
- `\deqn{}`

So an example might be:

```
#' Sum of vector elements.
#'
#' \code{sum} returns the sum of all the values present in its arguments.
#'
#' This is a generic function: methods can be defined for it directly
#' or via the \code{\link{Summary}} group generic. For this to work properly,
#' the arguments \code{...} should be unnamed, and dispatch is on the
#' first argument.
#'
#' @param ... Numeric, complex, or logical vectors.
#' @param na.rm A logical scalar. Should missing values (including NaN)
#'   be removed?
#' @return If all inputs are integer and logical, then the output
#'   will be an integer. If integer overflow
#'   \url{http://en.wikipedia.org/wiki/Integer_overflow} occurs, the output
#'   will be NA with a warning. Otherwise it will be a length-one numeric or
#'   complex vector.
#'
#'   Zero-length vectors have sum 0 by definition. See
#'   \url{http://en.wikipedia.org/wiki/Empty_sum} for more details.
#' @examples
#' sum(1:10)
#' sum(1:5, 6:10)
#' sum(F, F, F, T, T)
#'
#' sum(.Machine$integer.max, 1L)
#' sum(.Machine$integer.max, 1)
#'
#' \dontrun{
#' sum("a")
#' }
#'
#' @section Warning:
#' Do not operate heavy machinery within 8 hours of using this function.
#'
#' @family aggregate functions
#' @seealso \code{\link{prod}} for products, \code{\link{cumsum}} for cumulative
#'   sums, and \code{\link{colSums}}/\code{\link{rowSums}} marginal sums over
#'   high-dimensional arrays.
#'
sum <- function(..., na.rm = TRUE) {}
```


### Vignettes

Common vignette browsing commands:

- `browseVignettes()` to see all installed vignettes
- `browseVignettes(packagename)` to see vignettes for a specific package
- `vignette()` to read a vignette
- `edit(vignette())` to see the vignette's code

To create a vignette with devtools: `devtools::use_vignette("my-vignette")`, which does 3 things:

1. Creates a `vignettes/` directory
1. Adds the necessary dependencies to the `DESCRIPTION` file (adds `knitr` to the `Suggests` and `VignetteBuilder` fields)
1. Drafts a template vignette, `vignettes/my-vignette.Rmd`

Workflow:

- Modify the vignette
- Run individual code chunks with **Ctrl-Alt-C**
- Run all code chunks with **Ctrl-Alt-R**
- Knit to preview output with **Ctrl-Shift-K**

CRAN Notes

You build vignettes locally. CRAN only receives the output (html/pdf) and the source code. CRAN does not rebuild the vignette; it only checks that the code is runnable (by running it).


### Testing

[**ADD TESTING CHAPTER NOTES**]


### Datasets

You can include data in your package. `.RData` datasets go in `data/`; raw datasets go in `inst/extdata/`.

- Put datasets as `.RData` files in the `/data` directory with the same name as the object once it's loaded into R's workspace 
- Include `LazyData: true` in the `DESCRIPTION` file, so that datasets don't occupy memory until used
- Exported datasets must be documented; do so by documenting the _name_ of the dataset and use the two roxygen tags `@format` and `@source`:

```
#' Prices of 50,000 round cut diamonds.
#'
#' A dataset containing the prices and other attributes of almost 54,000
#' diamonds.
#'
#' @format A data frame with 53940 rows and 10 variables:
#' \describe{
#'   \item{price}{price, in US dollars}
#'   \item{carat}{weight of the diamond, in carats}
#'   ...
#' }
#' @source \url{http://www.diamondse.info/}
"diamonds"
```

For CRAN, datasets should be less than 1MB and compressed.

- Run `tools::checkRdaFiles()` to determine the best compression for each file
- Rerun `devtools::use_data()` with `compress` set to that optimal value

