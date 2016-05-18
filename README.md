This R package encapsulates a toy workflow. I created it to test if the [`remake`](https://github.com/richfitz/remake) package plays nicely with workflows implemented as R packages. Use `devtools::install_github()` to install.

# The workflow

The workflow generates some data, processes the data, and makes a plot. In terms of functions in R/code.R, the workflow does the following.

```
wrapper_function() # calls generate_data()
processed <- process_data("data.csv")
pdf("plot.pdf")                    
myplot(processed)
dev.off()
```

I would like to run the workflow using [`remake`](https://github.com/richfitz/remake). I use the following `remake.yml`

```
sources:
  - remakeLoadPackge.R

targets:
  all:
    depends: plot.pdf

  data.csv:
    command: wrapper_function()

  processed:
    command: process_data("data.csv")

  plot.pdf:
    command: myplot(processed)
    plot: true

```

with `remakeLoadPackage.R` below

```
library(remakeInPackage)
```

which I would like to keep short since I want to rely on my package to define the elements of the workflow.

# The results

After installing the package and putting `remake.yml` and `remakeLoadPackage.R` in my current working directory, a call to `remake::make()` completes normally.

```
> library(remake)
> make()
[  LOAD ] 
[  READ ]           |  # loading sources
<  MAKE > all
[ BUILD ] data.csv  |  wrapper_function()
[  READ ]           |  # loading packages
[ BUILD ] processed |  processed <- process_data("data.csv")
[  PLOT ] plot.pdf  |  myplot(processed) # ==> plot.pdf
[ ----- ] all
```

 But when I modify the contents of `remakeInPackage::generate_data()`, rebuild and resintall `remakeInPackage`, open a new R session, and and call `remake::make()`, the output files are not remade.
 
```
> make() 
<  MAKE > all
[    OK ] data.csv
[    OK ] processed
[    OK ] plot.pdf
[ ----- ] all
```

I would like a remake to be triggered.
