# Basics for coding optimization problems
This document will give the reader an introduction to numerical optimization and the various trade-offs associated with the most common optimization algorithms. We will also cover how to code and evaluate an optimization problem in R.

## Basics of functional optimization

- All statistical estimators are either minima of some error (aka "cost") function or maxima of some likelihood function
    * Examples:
    * linear regression (OLS)
    * logistic regression
    * naive Bayes classification
    * Multi-Layer Perceptron (MLP) aka artifical neural networks (ANN)
    * support vector machine (SVM)
    * ... 
    * many others
- Some estimators (e.g. OLS, Generalized Method of Moments) have **closed form** solutions for the extremum value
    * we covered this in our previous lecture
- Other estimators (e.g. MLE, some non-linear least squares) require **numerical methods** to find the extremum value
    * we also showed this in the previous lecture (the case of logistic regression)
- There are a number of different algorithms to numerically find extrema ("guess-and-check")
- You can use these in R, Python, Julia, Matlab, ...

## Graph of the optimization problem

<img width="450" src="http://earlelab.rit.albany.edu/gallery/LogLVOacac.jpg" alt="Likelihood function">

This is a graph of a likelihood function. The parameters are on the `x` and `y` axes and the likelihood is on the `z` axis.

## How numerical optimization works
- User provides an **objective function** and a **starting value**
- The optimizer then calculates the magnitude and direction of the *gradient vector* at the starting value
- Given these values of the gradient vector, it decides 
    * in which direction to move to get closer to the optimum
    * how far it should move to get closer to the optimum
- Given the magnitude and direction of the gradient vector at the new point, it repeats these steps until convergence is achieved 

## How to determine convergence? (How do we know when we've "arrived"?)
There are generally three ways to determine if a numerical optimization algorithm has reached the optimum
1. The gradient vector (or Jacobian matrix) is numerically close to 0
    - Intuition: we may be at an extremum if the derivative is 0
2. The location of the previous guess is numerically close to the location of the updated guess
    - Intuition: The optimizer didn't move very far in updating the guess, so we're probably close to the answer
3. The value of the objective function at the previous guess is numerically close to the value of the objective function at the updated guess
    - Intuition: The objective function is flat in the area of the previous and updated guesses, which is kind of like having a gradient of zero

Note that "numerically close" generally refers to being within 10^-6


## Local or global minimum?
- Note that in the previous slide, no mention of second-order conditions was ever made
- Hence, it is unclear if the answer our optimizer gave is a local optimum, a saddle point, or a global optimum
- Luckily, most common statistical estimators have been proven to have **globally concave** objective functions
- If you are unsure whether or not your objective function is globally concave, you can try different sets of starting values and see how much the answers differ

*Note:* "globally concave" simply means that the function has only one "hump" or only one "dip" which means we will eventually get to the only extremum.

## Most commonly used optimization algorithms

- Gradient descent
- Stochastic gradient descent
- BFGS/L-BFGS
- Nelder-Mead (gradient-free simplex methods)
- Gauss-Newton 

## When should I use which algorithm?

- It turns out that optimization can often be more of an "art" than a "science"
- Often times your choice of objective function will dictate which optimization algorithm you should choose
- We'll talk more about these considerations a bit later today 

## How to use these in R? 
R has several packages for calling these commonly used optimizers, as well as many others. In a future problem set, you will get experience with the packages `optimr` and `nloptr`.

Sadly, R is not the best language to be using if you need to optimize your own functions. I recommend Python or (especially) Julia!


## Gradient descent
What is gradient descent?

- Pretend you are on a mountain and there's a valley below that you're trying to get to
- There's a fog over the whole mountain, such that you can't see down the mountain
- You want to get down the mountain as quickly as possible
- You have a special compass that will tell you the steepness of the hill in every direction
- It's costly to look at the compass
- You need to decide how often to look at the compass (i.e. how far to walk before checking the compass and potentially changing direction)

The best way down the mountain is to follow the path of steepest descent while taking the minimum number of stops to look at the compass

## The math behind Gradient descent

- Start with some initial value (call it x0) and some step size (call it &gamma;)
- The new guess of the solution is equal to

x_n = x0 - &gamma;&nabla;f(x0)

where &nabla;f(x0) is the gradient vector of our objective function f(&middot;)

Repeat this process until convergence (i.e. until &nabla;f(x0) is arbitrarily close to 0, until x doesn't change across iterations, or until f(x) doesn't change across iterations

## Famous quote
"Victory will never be found by taking the line of least resistance." -- Winston Churchill

- But in actuality, with gradient descent, victory *is* found by taking the line of (steepest descent)!


## Gradient descent in R
Suppose we want to find the minimum of `f(x) = x^4 - 3*x^3 + 2`

```
# set up a stepsize
alpha = 0.003

# set up a number of iteration
iter = 500

# define the gradient of f(x) = x^4 - 3*x^3 + 2
gradient = function(x) return((4*x^3) - (9*x^2))

# randomly initialize a value to x
set.seed(100)
x = floor(runif(1)*10)

# create a vector to contain all xs for all steps
x.All = vector("numeric",iter)

# gradient descent method to find the minimum
for(i in 1:iter){
        x = x - alpha*gradient(x)
        x.All[i] = x
        print(x)
}

# print result and plot all xs for every iteration
print(paste("The minimum of f(x) is ", x, sep = ""))
```

## How to write a function in R 
How do you write a function in R? The function has four distinct elements:

1. Name
2. Arguments (aka inputs)
3. Statements (i.e. code)
4. Outputs

```
myfunction <- function(arg1, arg2, ... ){
statements
return(object)
}
```

Note the braces!

## Calling a function
Once you have your function written, you call it as follows:

```
output <- myfunction(arg1, arg2, ...)
```

where `output` can be any name you'd like, and `arg1` etc. are the specific arguments you're interested in

## Example of a simple function
```
customMean <- function(x){
mean <- sum(x)/length(x)
return(mean)
}

# Now let's call the function

sample.average <- customMean(iris$Sepal.Width)
print(sample.average)

# Check that it's correct
mean(iris$Sepal.Width)
```

## Function scope

- It doesn't matter what you name your variables when you call the function
- i.e. when we called `customMean()` above, we sent it `iris$Sepal.Width` even though we called the input `x` in the function definition
- This is all to say that when a function is *evaluated*, its new workspace is whatever is sent to the function
- This is important to keep in mind when trying to optimize your own objective function

<img width="450" src="functionScope.png" alt="Scope of functions">

## Using `nloptr`
Now that we know how to write a function, let's optimize the same function we did before (using gradient descent) but instead now let's use one of the optimizers in the `nloptr` package.


