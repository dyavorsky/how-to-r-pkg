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





