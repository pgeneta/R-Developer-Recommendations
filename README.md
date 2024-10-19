# Recommendations for R Developers

## Introduction

There are useful standards available for R developers to adopt but unless you have been pointed in their direction it's not necessarily obvious how they fit together.
This document makes some recommendations to make life easier for R developers who want to make their code more robust.
I will refer mostly to existing standards that have been developed by others, in particular RStudio.
I also add some advice based upon my experience.

The document shouldn't be understand as a set of rules; if any of these standards don't work for you then it's fine to develop your own.
What's most important is consistency; when working with others and also for your future self.

## Structure

### Package-Based Development

It's best to structure your code as R packages; they could be:

- Collections of functions that fit a theme
- Analyses and functions that support them
- A Shiny application
- A Plumber API

Hadley Wickham has written about R packages and how to structure them in the book here:

https://r-pkgs.org/

Further documentation can be found here:

https://cran.r-project.org/doc/manuals/r-devel/R-exts.html

#### Shiny Apps

When writing Shiny applications I recommend using `golem` standards.
Documentation for the `golem` package can be found here:

https://golemverse.org/packages/golem/

There is far more information on writing Shiny apps with `golem` in the following book:

https://engineering-shiny.org/index.html

### RStudio Projects

If you're using RStudio then you should create a single Project per R package.
A Project will have a file named identically to the project but with the `.Rproj` extension.
Using this will simplify managing, checking and testing your R package.
Find some details below:

https://support.posit.co/hc/en-us/articles/200526207-Using-RStudio-Projects

### Secrets

Passwords, keys, and other secrets should never be hardcoded into the package & never committed to git.
A simple way to manage this is to use `.Renviron` files to store your secrets.
The `.Renviron` file should also be added to your `.gitignore` file to avoid it being committed.
Some information on the `.Renviron` file is included in the link below:

https://support.posit.co/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf

It's possible (and desirable) to have `.Renviron` files in multiple places.
I recommend having a single `.Renviron` file at the user level to contain personal secrets such as your GitHub PAT & project level `.Renviron` files to manage environment variables for each package separately.
This will avoid any problems with environment variables names clashing (in the example of the same environment variable being used for different values across 2+ projects).

## Readability

There are actions we can take as developers to make our code easier for others & our future selves to read and understand.

### Coding Style

Code should be written to a regular format with easily understandable rules.
The Tidyverse Style Guide serves as a comprehensive foundation for any team coding style:

https://style.tidyverse.org/

### Referring to Functions from Other Packages

All of the packages linked to functions that are called directly in the code should be listed in the `DESCRIPTION` file.
The `library` function should almost **never** be called in an R package (excluding in some cases in Rmarkdown documents).
Those functions should then (nearly) always be called with the following format:

`package_name::function_name`

There are 3 reasons for this:

- It improves comprehension of the code because the function is always called with it's package name next to it
- There will never be any confusion over 2+ functions with the same name from different packages
- There is no need to use the `@importFrom` tag in the `roxygen2` documentation

There are some exceptions to the rule:

- Functions from `base` R should just call `function_name` and not `base::function_name`
- The `.data` pronoun from Tidyverse should always be added with the `@importFrom` tag
- If a function needs to be optimized for performance then importing functions with `@importFrom` tag is permissible

### Documentation

Use the `roxygen2` package for documenting your R functions.
This will help ensure your package passes various CRAN checks.
Documentation can be found here:

https://roxygen2.r-lib.org/

## Performance

### Use Native Pipes

Since R version 4.1 the native R pipe `|>` has been available.
If it is possible I would recommend using version 4.1 or higher to take advantage of the new pipe.
The new pipe has a speed improvement over the `magrittr` pipe `%>%`.

## Managing Change

### Version Control

I recommend using git to keep track of the versions of your code.
It's possible to use a monorepo-style format where many packages are kept in one single git repository.
However, a difficulty with this is that it's not simple to then load those packages into R from GitHub or similar.
I recommend one R package per git repository as it's then a simple matter to load the package in with `renv`.

If you are using Posit the top right panel contains a tab that assists in running simple git commands.
It's useful for simple commit, revert, pull and push but it's not sufficient for all circumstances.
Some knowledge of the git command line commands will still be necessary.

### Version Numbers

There is information on how to use version numbers meaningfully in the R Packages book by Hadley Wickham but an outline of Semantic Versioning can be found here:

https://semver.org/

### Package Dependencies

Use `renv` to manage your dependencies on R packages.
This allows the use of multiple versions of the same packages across different projects.
Find information about `renv` here:

https://rstudio.github.io/renv/

I recommend ignoring the current package (the package the `renv` is managing dependencies for) by setting `renv::settings$ignore.packages("your_package_name")`.
This can avoid problems where `renv` fails because it tries to find the current package from CRAN or other external source.
Information on the setting can be found here:

https://rstudio.github.io/renv/reference/settings.html#ignored-packages

## Testing

### CRAN Check

Packages that get uploaded to CRAN have to pass a number of checks before they are permitted.
These are useful checks to take into consideration even if you are not making your packages public.
Information about the checks is contained in an appendix to Hadley Wickham's R Packages book:

https://r-pkgs.org/R-CMD-check.html

When using an RStudio project to manage your package you can find a button to perform the Check process in the top right panel.

### Data Validation

When writing code to transform data you may want to consider adding data validation rules to ensure the output matches your expectations.
For example, if you wrote an Rmarkdown document that runs on a schedule & updates a file, you could check the transformed data before writing / saving.
There is a package called `validate` that allows the simple creation of rules.
A book explaining how to use the package can be found here:

https://cran.r-project.org/web/packages/validate/vignettes/cookbook.html

### Unit Testing of Functions

When it comes to unit testing of functions there is the `testthat` package which easily integrates into R packages;

https://testthat.r-lib.org/

### Testing Applications

For testing of shiny applications you may refer to the following documentation of the `shinytest2` package:

https://rstudio.github.io/shinytest2/
