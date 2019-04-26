# .Rprofile -- commands to execute at the beginning of each R session
#
# Use this file to set options, load packages, etc
#
# NOTE: changes in this file won't be reflected until after you quit
# and start a new session
#
# Blogpost worth reading:
# http://kevinushey.github.io/blog/2015/02/02/rprofile-essentials/
#
# This .Rprofile is based on different .Rprofiles from
# https://github.com/kevinushey/etc/blob/master/dotfiles/.Rprofile
# https://github.com/jimhester/dotfiles/blob/master/R/Rprofile
# https://gist.github.com/GarrettMooney/9472d66a7c4dddc989447056a15d93a3


.First = function() {

  # only run in interactive mode
  if (!interactive())
    return()

  # Name and Email
  GIVEN = intToUtf8(
    c(67L, 104L, 114L, 105L, 115L, 116L, 111L, 112L, 104L)
  )

  FAMILY = intToUtf8(
    c(83L, 116L, 101L, 112L, 112L, 101L, 114L)
  )

  EMAIL = intToUtf8(
    c(99L, 104L, 114L, 105L, 115L, 116L, 111L, 112L, 104L,
      46L,
      115L, 116L, 101L, 112L, 112L, 101L, 114L,
      64L,
      103L, 102L, 107L,
      46L,
      99L, 111L, 109L)
  )

  # ensure Rtools on PATH for Windows
  isWindows <- Sys.info()[["sysname"]] == "Windows"
  if (isWindows) {
    PATH <- Sys.getenv("PATH", unset = "")
    paths <- strsplit(PATH, .Platform$path.sep, fixed = TRUE)[[1]]
    hasRtools <- any(file.exists(file.path(paths, "Rtools.txt")))
    if (!hasRtools) {

      # construct candidate Rtools paths
      rMajor <- as.numeric(getRversion()[1, 1])
      rMinor <- as.numeric(getRversion()[1, 2])
      candidates <- c(
        sprintf("C:\\RBuildTools\\%s", paste(rMajor, rMinor + 1, sep = ".")),
        "C:\\Rtools"
      )

      discovered <- FALSE
      for (candidate in candidates) {
        if (file.exists(file.path(candidate, "Rtools.txt"))) {

          bitness <- .Machine$sizeof.pointer * 8
          entries <- paste(normalizePath(Filter(file.exists, c(
            file.path(candidate, "bin"),
            if (getRversion() < "3.3.0")
              file.path(candidate, "gcc-4.6.3/bin")
            else
              file.path(candidate, sprintf("mingw_%s/bin", bitness))
          ))), collapse = .Platform$path.sep)

          Sys.setenv(PATH = paste(entries, PATH, sep = .Platform$path.sep))
          discovered <- TRUE
          break
        }
      }

      if (!discovered)
        warning("Failed to discover Rtools on the PATH")
    }
  }

  # prefer source packages for older versions of R
  pkgType <- getOption("pkgType")
  if (getRversion() < "3.3")
    pkgType <- "source"

  # repositories (add local drat repo)
  local({
    r <- getOption("repos")
    r["GIdrat"] <- "file://gfk.com/data/001483/geom/MDR/GIdev/06_software/R/GIpackages/DRAT/"
    r["CRAN"] <- "https://cran.rstudio.com/"
    options(repos = r)
  })

  # options
  options(

    # Ncpus: The number of parallel processes to use for a parallel install
    # of more than one source package.
    Ncpus = 6L,

    # unicode encoding
    # cli.unicode = TRUE,

    # set RStudio Viewer to NULL (not always working)
    viewer = NULL,

    # max lines to print in console
    max.print = 1000,

    # penalty for printing numeric values in scientific notation
    scipen = 10,

    # no fancy quotes (makes copy + paste a pain)
    useFancyQuotes = FALSE,

    # don't treate newlines as synonym to 'n' in browser
    browserNLdisabled = TRUE,

    # no tcltk
    menu.graphics = FALSE,

    # # show only two lines in traceback
    # deparse.max.lines = 2 ,

    #  error handling
    error = quote(rlang::entrace()),
    rlang_backtrace_on_error = "collapse", # or "branch" or "full"

    # warn on partial matches
    # warnPartialMatchArgs = TRUE, # too disruptive
    warnPartialMatchAttr = TRUE,
    warnPartialMatchDollar = TRUE,

    # warn right away
    # warn = 2, # all warnings are turned into errors
    warn = 1, # warnings are printed as they occur

    # truncation limit for warning messages
    warning.length = 8170, # max warning length

    # source packages for older R
    pkgType = pkgType,

    # keep source code as-is for package installs
    keep.source = TRUE,
    keep.source.pkgs = TRUE,

    # devtools/usethis related options
    # useful for usethis::create_package / usethis::use_description
    usethis.protocol  = "https",
    usethis.full_name = paste(GIVEN, FAMILY, sep = " "),
    usethis.description = list(
      `Authors@R` = paste('person("', GIVEN, '", "', FAMILY, '", email = "',
                          EMAIL, '", role = c("aut", "cre"))',
                          sep = ""),
      License = "MIT + file LICENSE",
      Version = "0.0.0.9000"
    ),

    # devtools options for backwards compatibility
    devtools.name = paste(GIVEN, FAMILY, sep = " "),
    devtools.desc = list(
      Author = paste(GIVEN, FAMILY, sep = " "),
      Maintainer = paste(GIVEN, " ", FAMILY,  " <", EMAIL, ">", sep = ""),
      License = "MIT + file LICENSE",
      Version = "0.0.1"
    )

  )

  # always run Rcpp tests
  Sys.setenv(RunAllRcppTests = "yes")

  # auto-completion of package names in `require`, `library`
  utils::rc.settings(ipck = TRUE)

  # ensure package development tools are loaded
  suppressPackageStartupMessages(require(devtools))
  suppressPackageStartupMessages(require(testthat))
  suppressPackageStartupMessages(require(usethis))


  # add notification for long running tasks
  invisible({
    # https://github.com/gaborcsardi/notifier
    # devtools::install_version("notifier")
    notify_long_running <- function(second_cutoff = 20) {
      last <- proc.time()[1]
      function(expr, value, ok, visible) {
        duration <- proc.time()[1] - last
        if (duration > second_cutoff) {
          notifier::notify(msg = paste0(collapse = " ", deparse(expr)), title = sprintf("Completed in %.02f (s)", duration))
        }
        last <<- proc.time()[1]
        TRUE
      }
    }

    addTaskCallback(notify_long_running())
  })

  # Lookup R full function definitions, including compiled code, S3 and S4 methods.
  # devtools::install_github(jimhester/lookup)
  suppressPackageStartupMessages(require(lookup))

  # change prompt to promptr
  # require("promptr")
  # set_prompt(prompt_memuse)

  # display startup message(s)
  msg <- if (length(.libPaths()) > 1)
    "Using libraries at paths:\n"
  else
    "Using library at path:\n"
  libs <- paste("-", .libPaths(), collapse = "\n")
  message(msg, libs, sep = "")


}

