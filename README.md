# Notes from Hadley's R Packages Book
Dan Yavorsky  

### Create a Package

- To create a package from scratch, click "New Package" > "New Directory" > "R Package"
- To associate a new package with an existing R project, use `devtools::use_rstudio("path/to/package")`

### Connect to GitHub

First, track the pacakge with git:

- When creating a package, check the "git" box
- Or run `git init` in a shell after `cd`ing into the package's directory

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

### Working with Git/GitHub

- To keep files untracked, add them to `.gitignore` (can do this with a right-click in the Git pane of RStudio)
- **Ctrl-Alt-M** opens the commit window
- **Ctrl-Alt-D** diffs files

You'll want a markdown README file at the top level of the directory so that GitHub renders it on the repo's landing page. Best to add the `README.Rmd` and `README.html` to `.gitignore` so there's no confusion on which gets rendered by GitHub.

### Package Development Workflow

To load the development version of the package, run `devtools::load_all()`. The easier way, however, is to press **Ctrl-Shift-L** in RStudio, which saves all open files and loads the package. So the R code workflow is:

- Edit an R File
- Press **Ctrl-Shift-L**
- Explore code in console
- Repeat

R code goes in the `R/` directory. You _cannot_ put subdirectories in there.

### Build Ignore

To omit a directory or file when bundling a package, include a RegEx in the `.Rbuildignore` file. For this package, the RMarkdown version of this ReadMe file (`README.Rmd`) has been added to `.Rbuildignore`.

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

### Documentation Workflow with Roxygen2

The four basic steps are:

1. Add roxygen comments to your .R files
1. Run `devtools::document()` (or press **Ctrl-Shift-D** in RStuio) to covernt roxygen comments to .Rd files
1. Preview documentation with ?
1. Repeat

Documenting R code with roxygen2 involves putting the help documentation directly into the .R code files using roxygen comments, which start with `#'`. 

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



### 5 States of Packages

For package development, it's helpful to know about the various states that a package can be in. 

  1. _Source_: Just a directory of files and subdirectories (i.e., the development version of a package)
  1. _Bundled_: A compressed, single file (.tar.gz) of the package; not useful on its own, but a useful intermediary step
  1. _Binary_: Platform-specific; how packages are distributed via CRAN
  1. _Installed_: A decompressed binary package
  1. _In-Memory_: to use a package, you need to load it into memory

A note on loading and attaching packages:

  - R loads packages automatically when you use them
  - To use a package without providing its name (e.g., `breg()` instead of `bayesm::breg()`) you need to attach it to the search path
  - `library()` and `require()` load then attach an installed package, but these are not useful when developing a package; instead use `devtools::load_all()` or RStudio's "Build & Reload" feature (discussed later)
  - `.libPaths()` shows you where libraries (directories of packages) live on your computer. R searches these when you call `library()` or `require()`. The difference is that, if the requested package isn't in one of those libraries, `library()` thows an error whereas `require()` prints a message and returns `FALSE`.




The command line tool `R CMD INSTALL` powers all package installation. The R package `devtools` provides functions that are wrappers for `R CMD INSTALL` so that you can call this command from instide of R. 

- `devtools::install()` is a wrapper for `R CMD INSTALL`
- `devtools::build()` is a wrapper for `R CMD BUILD` that turns source packages into bundles
- `devtools::install_github()` downloads a source package from GitHub, runs `build()` to make vignettes, and then uses `R CMD INSTALL` to do the install
- `devtools::install_url()` and `devtools::install_bitbucket()` work similarly





