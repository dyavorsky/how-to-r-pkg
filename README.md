# README
Dan Yavorsky  

### Create a Package

- To create a package from scratch, click "New Package" > "New Directory" > "R Package"
- To associate a new package with an existing R project, use `devtools::use_rstudio("path/to/package")`

### Build Ignore

To omit a directory or file when bundling a package, include a RegEx in the `.Rbuildignore` file. For this package, the RMarkdown version of this ReadMe file (`README.Rmd`) has been added to `.Rbuildignore`.

### Connecting to GitHub

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





