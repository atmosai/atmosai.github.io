# Use statsmodels to Perform Linear Regression in Python



Often times, linear regression is associated with *machine learning* – a hot topic that receives a lot of attention in recent years. And so, in this tutorial, I’ll show you how to perform a linear regression in Python using *statsmodels.* I’ll use a simple example about the stock market to demonstrate this concept.

Here are the topics to be covered:

- Background about Linear Regression
- Review of an Example with the Full Dataset
- The Python Code
- Interpreting the Regression Results
- Making Predictions based on the Regression Results

To start, I’ll share some background about linear regression.

## About Linear Regression

Linear regression is used as a predictive model that assumes a linear relationship between the dependent variable (which is the variable we are trying to predict/estimate) and the independent variable/s (input variable/s used in the prediction).

For example, you may use linear regression to predict the price of the stock market (your dependent variable) based on the following Macroeconomics input variables:

- Interest Rate
- Unemployment Rate

Under Simple Linear Regression, only *one* independent/input variable is used to predict the dependent variable. It has the following structure:

<center>Y = C + M*X</center>

- Y = Dependent variable (output/outcome/prediction/estimation)
- C = Constant (Y-Intercept)
- M = Slope of the regression line (the effect that X has on Y)
- X = Independent variable (input variable used in the prediction of Y)

In reality, a relationship may exist between the dependent variable and *multiple* independent variables. For these types of models (assuming linearity), we can use [Multiple Linear Regression](https://datatofish.com/multiple-linear-regression-python/) with the following structure:

<center>Y = C + M1*X1 + M2*X2 + …</center>

## An Example (with the Dataset to be used)

For illustration purposes, let’s suppose that you have a fictitious economy with the following parameters:

| **Year** | **Month** | **Interest_Rate** | **Unemployment_Rate** | **Stock_Index_Price** |
| -------- | --------- | ----------------- | --------------------- | --------------------- |
| 2017     | 12        | 2.75              | 5.3                   | 1464                  |
| 2017     | 11        | 2.5               | 5.3                   | 1394                  |
| 2017     | 10        | 2.5               | 5.3                   | 1357                  |
| 2017     | 9         | 2.5               | 5.3                   | 1293                  |
| 2017     | 8         | 2.5               | 5.4                   | 1256                  |
| 2017     | 7         | 2.5               | 5.6                   | 1254                  |
| 2017     | 6         | 2.5               | 5.5                   | 1234                  |
| 2017     | 5         | 2.25              | 5.5                   | 1195                  |
| 2017     | 4         | 2.25              | 5.5                   | 1159                  |
| 2017     | 3         | 2.25              | 5.6                   | 1167                  |
| 2017     | 2         | 2                 | 5.7                   | 1130                  |
| 2017     | 1         | 2                 | 5.9                   | 1075                  |
| 2016     | 12        | 2                 | 6                     | 1047                  |
| 2016     | 11        | 1.75              | 5.9                   | 965                   |
| 2016     | 10        | 1.75              | 5.8                   | 943                   |
| 2016     | 9         | 1.75              | 6.1                   | 958                   |
| 2016     | 8         | 1.75              | 6.2                   | 971                   |
| 2016     | 7         | 1.75              | 6.1                   | 949                   |
| 2016     | 6         | 1.75              | 6.1                   | 884                   |
| 2016     | 5         | 1.75              | 6.1                   | 866                   |
| 2016     | 4         | 1.75              | 5.9                   | 876                   |
| 2016     | 3         | 1.75              | 6.2                   | 822                   |
| 2016     | 2         | 1.75              | 6.2                   | 704                   |
| 2016     | 1         | 1.75              | 6.1                   | 719                   |

The goal here is to predict/estimate the stock index price based on two Macroeconomics variables: the Interest rate and the Unemployment rate.

We will use [pandas DataFrame](https://datatofish.com/create-pandas-dataframe/) to capture the above data in Python. Before we dive into the Python code, make sure that both the *statsmodels* and *pandas* packages are installed. You may use the [PIP method](https://datatofish.com/install-package-python-using-pip/) to install those packages.



## The Python Code using statsmodels

The following Python code includes an example of [Multiple Linear Regression](http://datatofish.com/multiple-linear-regression-python/), where the input variables are:

- Interest_Rate
- Unemployment_Rate

These two variables are used in the prediction of the dependent variable of *Stock_Index_Price*.

Alternatively, you can apply a Simple Linear Regression by keeping only one input variable within the code. For example, if you just want to use the interest rate as the input variable, simply use the following syntax of *X = df[‘Interest_Rate’]*, instead of using X = df[[‘Interest_Rate’,’Unemployment_Rate’]]

 

```python
from pandas import DataFrame
import statsmodels.api as sm

Stock_Market = {'Year': [2017,2017,2017,2017,2017,2017,2017,2017,2017,2017,2017,2017,2016,2016,2016,2016,2016,2016,2016,2016,2016,2016,2016,2016],
                'Month': [12, 11,10,9,8,7,6,5,4,3,2,1,12,11,10,9,8,7,6,5,4,3,2,1],
                'Interest_Rate': [2.75,2.5,2.5,2.5,2.5,2.5,2.5,2.25,2.25,2.25,2,2,2,1.75,1.75,1.75,1.75,1.75,1.75,1.75,1.75,1.75,1.75,1.75],
                'Unemployment_Rate': [5.3,5.3,5.3,5.3,5.4,5.6,5.5,5.5,5.5,5.6,5.7,5.9,6,5.9,5.8,6.1,6.2,6.1,6.1,6.1,5.9,6.2,6.2,6.1],
                'Stock_Index_Price': [1464,1394,1357,1293,1256,1254,1234,1195,1159,1167,1130,1075,1047,965,943,958,971,949,884,866,876,822,704,719]        
                }

df = DataFrame(Stock_Market,columns=['Year','Month','Interest_Rate','Unemployment_Rate','Stock_Index_Price']) 

X = df[['Interest_Rate','Unemployment_Rate']] # here we have 2 variables for multiple regression. If you just want to use one variable for simple linear regression, then use X = df['Interest_Rate'] for example.Alternatively, you may add additional variables within the brackets
Y = df['Stock_Index_Price']

X = sm.add_constant(X) # adding a constant

model = sm.OLS(Y, X).fit()
predictions = model.predict(X) 

print_model = model.summary()
print(print_model)
```



This is the result that you’ll get once your run the Python code:



![](https://ws1.sinaimg.cn/large/006tNbRwly1fxcidznf8gj30m30cvdh7.jpg)



## Interpreting the Regression Results

I highlighted several important components within the results:

1. <font color='red'><strong>Adjusted. R-squared</strong></font>:  reflects the fit of the model. R-squared values range from 0 to 1, where a higher value generally indicates a better fit, assuming certain conditions are met.
2. <font color='green'><strong>const coefficient</strong></font>:  is your Y-intercept. It means that if both the Interest_Rate and Unemployment_Rate coefficients are zero, then the expected output (i.e., the Y) would be equal to the const coefficient.
3. <font color='blue'><strong>Interest_Rate coefficient</strong></font>:  represents the change in the output Y due to a change of one unit in the interest rate (everything else held constant)
4. **Unemployment_Rate coefficient** represents the change in the output Y due to a change of one unit in the unemployment rate (everything else held constant)
5. **std err** reflects the level of accuracy of the coefficients. The lower it is, the higher is the level of accuracy
6. **P >|t|** is your *p-value*. A p-value of less than 0.05 is considered to be statistically significant
7. **Confidence Interval** represents the range in which our coefficients are likely to fall (with a likelihood of 95%)



## Making Predictions based on the Regression Results

Recall that the equation for Multiple Linear Regression is:

<center>Y = C + M1*X1 + M2*X2 + …</center>

So for our example, it would look like this:

<center>Stock_Index_Price = (<font color='green'>const coef</font>) + (<font color='blue'>Interest_Rate coef</font>)*X1 + (Unemployment_Rate coef)*X2</center>

And this is how our equation would look like once we plug the coefficients:

<center>Stock_Index_Price = (<font color='green'>1798.4040</font>) + (<font color='blue'>345.5401</font>)*X1 + (-250.1466)*X2</center>

Let’s suppose that you want to predict the stock index price, where you just collected the following values for the first month of 2018:

- Interest Rate = 2.75 (i.e., X1= 2.75)
- Unemployment Rate = 5.3 (i.e., X2= 5.3)

When you plug those numbers you’ll get:

<center>Stock_Index_Price = (1798.4040) + (345.5401)*X1 + (-250.1466)*X2</center>

<center>Stock_Index_Price = (1798.4040) + (345.5401)*(2.75) + (-250.1466)*(5.3) = 1422.86</center>

The predicted/estimated value for the Stock_Index_Price in January 2018 is therefore 1422.86.

The predicted value can eventually be compared with the actual value to check the level of accuracy. If, for example, the actual stock index price for that month turned out to be 1435, then the prediction would be off by 1435 – 1422.86 = 12.14

Disclaimer: this example should not be used as a predictive model for the stock market. It was based on a fictitious economy for illustration purposes only.

You may want to check the following tutorial that includes an [example of multiple linear regression](https://datatofish.com/multiple-linear-regression-python/) using both sklearn and statsmodels.

Finally, for further information about the *statsmodels* module*,* please refer to the statsmodels [documentation](https://www.statsmodels.org/stable/index.html).



[Here is original article.](https://datatofish.com/statsmodels-linear-regression/)


