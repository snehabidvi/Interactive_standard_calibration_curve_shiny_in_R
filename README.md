## Interactive shiny app dashboard of standard calibration curve

This is an interactive Shiny app dashboard for a standard calibration curve and prediction of data points fromm fitted linear model provides a versatile and powerful platform for construction and analysis of calibration curves, with several advanced features that enhance both usability and accuracy. This dashboard allows users to define the concentration range of the analyte under study, facilitating customization for different colorimetric assays with particular concentration detection range. Users can input absorbance values either as single measurements or in triplicates where the three absorbance values are separated by comma, with the app automatically calculating the mean and standard deviations to display error bars for the triplicate readings, also the R squared value, p value and line equation is displayed on the plot to reflect measurement variability and enhance the scientific rigor of the output. 

## Standard callibration curve

A calibration curve, also called as a standard curve, is a general method of determining the concentration of a substance in an unknown sample by comparing the unknown to a set of standard samples of known concentration. The data - the concentrations of the analyte and the instrument response for each standard concentration - can be fit to a straight line, with the use of linear regression analysis.

This R script is to plot standard callibration curve (not interactive).

## Loading necessary libraries


```{r}
library(dplyr)
library(tidyr)
library(ggplot2)

```

## Input data

The concentration and absorbance data can be given as an input in a dataframe format or can be imported in rstudio from .csv, .excel, .xlsx file.

```{r}
# Concentration range from 0 to 1000 (you can add concentration values as per your assay)

conc <- seq(0,1000,by=200)


# Three sets of assay (Absorbance values measured at specific wavwlength as per assay)

rep1 <- c(0.000,0.098,0.201,0.570,0.612,0.671)

rep2 <- c(0.000,0.145,0.356,0.558,0.615,0.774)

rep3 <- c(0.000,0.211,0.340,0.593,0.643,0.702)


df <- data.frame(conc,rep1,rep2,rep3)


# Importing data from comma separated values (.csv) file 

df <- read.csv("file name.csv")


```

## Wide data converted into long data form 

pivot_longer() function "lengthens" data, increasing the number of rows and decreasing the number of columns.

```{r}
library(tidyr)
library(dplyr)

df_long <- df %>% pivot_longer(names_to="groups",values_to="abs",cols=2:4)

```

Arguments -

    1. data	:- A data frame to pivot.

    2. cols	:- <tidy-select> Columns to pivot into longer format.

    3. ... :- Additional arguments passed on to methods.

    4. cols_vary :-	When pivoting cols into longer format, how should the output rows be arranged relative to their original row number?

    "fastest", the default, keeps individual rows from cols close together in the output. This often produces intuitively ordered output when you have at least one key column from data that is not involved in the pivoting process.

    "slowest" keeps individual columns from cols close together in the output. This often produces intuitively ordered output when you utilize all of the columns from data in the pivoting process.

    5. names_to	:- A character vector specifying the new column or columns to create from the information stored in the column names of data specified by cols.

    If length 0, or if NULL is supplied, no columns will be created.

    If length 1, a single column will be created which will contain the column names specified by cols.

    If length >1, multiple columns will be created. In this case, one of names_sep or names_pattern must be supplied to specify how the column names should be split. There are also two additional character values you can take advantage of:

    NA will discard the corresponding component of the column name.

    ".value" indicates that the corresponding component of the column name defines the name of the output column containing the cell values, overriding values_to entirely.

    6. names_prefix	:- A regular expression used to remove matching text from the start of each variable name.

    7.names_sep, names_pattern :-	If names_to contains multiple values, these arguments control how the column name is broken up.

    names_sep takes the same specification as separate(), and can either be a numeric vector (specifying positions to break on), or a single string (specifying a regular expression to split on).

    names_pattern takes the same specification as extract(), a regular expression containing matching groups (⁠()⁠).

    If these arguments do not give you enough control, use pivot_longer_spec() to create a spec object and process manually as needed.

    8. names_ptypes, values_ptypes :- Optionally, a list of column name-prototype pairs. Alternatively, a single empty prototype can be supplied, which will be applied to all columns. A prototype (or ptype for short) is a zero-length vector (like integer() or numeric()) that defines the type, class, and attributes of a vector. Use these arguments if you want to confirm that the created columns are the types that you expect. Note that if you want to change (instead of confirm) the types of specific columns, you should use names_transform or values_transform instead.

    9. names_transform, values_transform :- Optionally, a list of column name-function pairs. Alternatively, a single function can be supplied, which will be applied to all columns. Use these arguments if you need to change the types of specific columns. For example, names_transform = list(week = as.integer) would convert a character variable called week to an integer.

    If not specified, the type of the columns generated from names_to will be character, and the type of the variables generated from values_to will be the common type of the input columns used to generate them.

    10. names_repair :-	What happens if the output has invalid column names? The default, "check_unique" is to error if the columns are duplicated. Use "minimal" to allow duplicates in the output, or "unique" to de-duplicated by adding numeric suffixes. See vctrs::vec_as_names() for more options.

    11. values_to	:- A string specifying the name of the column to create from the data stored in cell values. If names_to is a character containing the special .value sentinel, this value will be ignored, and the name of the value column will be derived from part of the existing column names.

    12. values_drop_na :- If TRUE, will drop rows that contain only NAs in the value_to column. This effectively converts explicit missing values to implicit missing values, and should generally be used only when missing values in data were created by its structure.


## Calculating mean and standard deviation of absorbance grouped by concentration

group_by() function groups one or more variables. The summarise() function creates a new data frame. It returns one row for each combination of grouping variables; if there are no grouping variables, the output will have a single row summarising all observations in the input. It will contain one column for each grouping variable and one column for each of the summary statistics that you have specified.

```{r}
df1 <- df_long %>% group_by(conc) %>% summarise(mean_abs = mean(abs),sd_abs = sd(abs))

```


Arguments -

    1. .data :- A data frame, data frame extension (e.g. a tibble), or a lazy data frame (e.g. from dbplyr or dtplyr).
    
    2. ... :- Name-value pairs of summary functions. The name will be the name of the variable in the result. 
    
    The value can be:
    A vector of length 1, e.g. min(x), n(), or sum(is.na(y))
    A data frame, to add multiple columns from a single expression
    

## Plotting standard callibration curve

ggplot() function initializes a ggplot object. It can be used to declare the input data frame for a graphic and to specify the set of plot aesthetics intended to be common throughout all subsequent layers unless specifically overridden.

```{r}
p <- ggplot(df1,aes(conc,mean_abs))+
  geom_point()+
  geom_errorbar(aes(ymin=mean_abs - sd_abs,ymax = mean_abs + sd_abs),width=0.2)+
  geom_smooth(method = "lm",se=0,formula = y~x - 1)+
  labs(title = "Reducing sugar estimation by DNSA method",
       subtitle = "Standard callibration curve",x="Concentration in mcg/ml",y="Absorbance #at 540 nm")+
  annotate(geom = "text",x=360,y=0.75,label="Absorbance = 0.0007*conc, R-squared = 0.98,p-value <0.05")+
  theme_classic()

p

```


Arguments -

    1. data	:- Default dataset to use for plot. If not already a data.frame, will be converted to one by fortify(). If not specified, must be supplied in each layer added to the plot.

    2. mapping :- Default list of aesthetic mappings to use for plot. If not specified, must be supplied in each layer added to the plot.


## Fitting linear regression model

lm() function is used to fit linear models, including multivariate ones. It can be used to carry out regression, single stratum analysis of variance and analysis of covariance. Fitting linear model with formula absorbance is dependent on concentration of the analyte being studied. In the formula, mean_abs ~ conc - 1, the - 1 is added to allow for the regression line pass through the origin.

```{r}
mod <- lm(mean_abs ~ conc - 1, df1, method="lm")

summary(mod)

```

Arguments -

    1. formula :- An object of class "formula" (or one that can be coerced to that class): a symbolic description of the model to be fitted. 

    2. data	:- An optional data frame, list or environment (or object coercible by as.data.frame to a data frame) containing the variables in the model. If not found in data, the variables are taken from environment(formula), typically the environment from which lm is called.

    3. subset	:- An optional vector specifying a subset of observations to be used in the fitting process. 

    4. weights :-	An optional vector of weights to be used in the fitting process. Should be NULL or a numeric vector. If non-NULL, weighted least squares is used with weights weights (that is, minimizing sum(w*e^2)); otherwise ordinary least squares is used. 

    5. na.action :- function which indicates what should happen when the data contain NAs. The default is set by the na.action setting of options, and is na.fail if that is unset. The ‘factory-fresh’ default is na.omit. Another possible value is NULL, no action. Value na.exclude can be useful.

    6. method	:- The method to be used; for fitting, currently only method = "qr" is supported; method = "model.frame" returns the model frame (the same as with model = TRUE, see below).

    7. model, x, y, qr :- Logicals. If TRUE the corresponding components of the fit (the model frame, the model matrix, the response, the QR decomposition) are returned.

    8. singular.ok :-	Logical. If FALSE (the default in S but not in R) a singular fit is an error.

    9. contrasts :-	An optional list. See the contrasts.arg of model.matrix.default.

    10. offset :- This can be used to specify an a priori known component to be included in the linear predictor during fitting. This should be NULL or a numeric vector or matrix of extents matching those of the response. One or more offset terms can be included in the formula instead or as well, and if more than one are specified their sum is used.


## Prediction of unknown

Fittting inverse model with formula concentration is dependent on absorbance to predict unknown concentration of the analyte being studied from absorbance measured.

```{r}

mod1 <- lm(conc ~ mean_abs,df1,method = "lm")

summary(mod1)

uk = data.frame(mean_abs = c(0.292,0.347))

uk$pred = predict(mod1,newdata = uk)

```

Arguments -

    1. object	:- A model object for which prediction is desired.

    2. ... :- Additional arguments affecting the predictions produced.


## Adding predicted unknown data points

```{r}
p +
  geom_point(data=uk,aes(pred,mean_abs,color = "red",size=3))+
  geom_text(data=uk,aes(pred,mean_abs,label=paste("uk",1:2,"=",round(pred,2)))
            ,vjust=1.0,hjust=-0.1)+
  theme(legend.position = "none")

```
