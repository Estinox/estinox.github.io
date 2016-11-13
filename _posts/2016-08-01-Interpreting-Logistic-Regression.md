---
layout: post
title: Interpreting Coefficients of Logistic Regressions
tags:
    - python
    - notebook
    - statistics
---
![png]({{ site.url }}/assets/20160801_logistic_regression/kiwi.jpg)

Giggles: Key + Wii = Kiwi; Math!

Dug out this relatively notebook from a while ago when I was learning about
logistic regression. We all know that the coefficients of a linear regression
relates to the response variable linearly, but the answer to how the logistic
regression coefficients related was not as clear.

If you're also wondering the same thing, I've worked through a practical example
using Kaggle's Titanic dataset and validated it against Sklearn's logistic
regression library.

<!--more-->

## Table of Content

* [Quick Primer](#Quick-Primer)
* [Titanic Example](#Titanic-Example)
  * [Coefficient of a Single Dichotomous Feature](#Coefficient-of-a-Single-
Dichotomous-Feature)
    * [Fitting Against Sklearn](#Fitting-Against-Sklearn)
    * [Survival for Males](#Survival-for-Males)
    * [Survival for Females](#Survival-for-Females)
* [Where to Go from Here](#Where-to-Go-from-Here) 
 
## Quick Primer
Logistic Regression is commonly defined as:

$$h_\theta(x) = \frac{1}{1+e^{\theta^Tx}}$$

You already know that, but with some algebriac manipulation, the above equation
can also be interpreted as follows

$$log\left(\frac{h(x)}{1-h(x)}\right) = \theta^Tx$$

Notice how the linear combination, $$\theta^T x$$, is expressed as the *log odds
ratio* (logit) of $$h(x)$$, and let's elaborate on this idea with a few examples. 
 
## Titanic Example

Kaggle is a great platform for budding data scientists to get more practice. I'm
currently working through the Titanic dataset, and we'll use this as our case
study for our logistic regression.

Let's load some python libraries to boot. 

**In [1]:**

{% highlight python %}
import pandas as pd
import matplotlib.pylab as plt
import numpy as np
%matplotlib inline
{% endhighlight %}
 
Read in our data set 

**In [2]:**

{% highlight python %}
train = pd.read_csv('train.csv')
train[['PassengerId', 'Survived', 'Name', 'Sex', 'Age']].head()
{% endhighlight %}




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>male</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>female</td>
      <td>38.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1</td>
      <td>Heikkinen, Miss. Laina</td>
      <td>female</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>1</td>
      <td>Futrelle, Mrs. Jacques Heath (Lily May Peel)</td>
      <td>female</td>
      <td>35.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>0</td>
      <td>Allen, Mr. William Henry</td>
      <td>male</td>
      <td>35.0</td>
    </tr>
  </tbody>
</table>
</div>


 
###  Coefficient of a Single Dichotomous Feature

Dichotomous just means the value can only be either 0 or 1, such as the field
Sex in our titanic data set. In this section, we'll explore what the
coefficients mean when regressing against only one dichotomous feature.

Let's map males to 0, and female to 1, then feed it through sklearn's 
[logistic regression](http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.Logist
icRegression.html){:target="blank"} function to get the coefficients out,
$$\theta_0$$ for the bias, $$\theta_1$$ for the logistic coefficient for sex. Then
we'll manually compute the coefficients ourselves to convince ourselves of
what's happening. 

**In [3]:**

{% highlight python %}
train.Sex = train.Sex.apply(lambda x: 0 if x == 'male' else 1)

y_train = train.Survived
x_train = train.drop('Survived', axis=1)
{% endhighlight %}
 
#### Fitting Against Sklearn

Sklearn applies automatic regularization, so we'll set the parameter $$C$$ to a
large value to emulate no regularization.

 

**In [4]:**

{% highlight python %}
from sklearn.linear_model import LogisticRegression

clf = LogisticRegression(C=1e10) # some large number for C

feature = ['Sex']
clf.fit(x_train[feature], y_train)
{% endhighlight %}




    LogisticRegression(C=10000000000.0, class_weight=None, dual=False,
              fit_intercept=True, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)


 
Now that we've fitted the logistic regression function, we can ask sklearn to
give us the two terms in $$\theta$$, namely the intercept and the coefficient 

**In [5]:**

{% highlight python %}
print('intercept:', clf.intercept_)
print('coefficient:', clf.coef_[0])
{% endhighlight %}

    intercept: [-1.45707193]
    coefficient: [ 2.51366047]

 
Cool, so with our newly fitted $$\theta$$, now our logistic regression is of the
form:

$$h(survived | x) = \frac{1}{1 + e^{(\theta_0 + \theta_1x)}} = \frac{1}{1 +
e^{(-1.45707 + 2.51366x)}}$$

or

$$log\left(\frac{h(x)}{1-h(x)}\right) = -1.45707 + 2.51366x$$ 

#### Survival for Males
So, when $$x = 0$$, meaning $$x = male$$, our equation boils down to:
$$log\left(\frac{h(survived|x=male)}{1-h(survived|x=male)}\right) =
log\left(\frac{h(survives|x=male)}{h(\overline{survive}|x=male)}\right) =
-1.45707 + 2.51366(0) = -1.45707$$

Exponentiating both sides gives us:
$$\frac{h(survived|x=male)}{h(\overline{survived}|x=male)} = exp(-1.45707) =
0.232917$$

Now let's verify this ourselves via python. 

**In [6]:**

{% highlight python %}
survived_by_sex = train[train.Survived == 1].groupby(train.Sex).count()[['Survived']]
survived_by_sex['Total'] = train.Survived.groupby(train.Sex).count()
survived_by_sex['NotSurvived'] = survived_by_sex.Total - survived_by_sex.Survived
survived_by_sex['OddsOfSurvival'] = survived_by_sex.Survived / survived_by_sex.NotSurvived
survived_by_sex['ProbOfSurvival'] = survived_by_sex.Survived / survived_by_sex.Total
survived_by_sex['Log(OddsOfSurvival)'] = np.log(survived_by_sex.OddsOfSurvival)

survived_by_sex
{% endhighlight %}




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Survived</th>
      <th>Total</th>
      <th>NotSurvived</th>
      <th>OddsOfSurvival</th>
      <th>ProbOfSurvival</th>
      <th>Log(OddsOfSurvival)</th>
    </tr>
    <tr>
      <th>Sex</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>109</td>
      <td>577</td>
      <td>468</td>
      <td>0.232906</td>
      <td>0.188908</td>
      <td>-1.457120</td>
    </tr>
    <tr>
      <th>1</th>
      <td>233</td>
      <td>314</td>
      <td>81</td>
      <td>2.876543</td>
      <td>0.742038</td>
      <td>1.056589</td>
    </tr>
  </tbody>
</table>
</div>


 
As you can see, for males, we had 109 men who survived, but 468 did not survive.
The odds of survival for men is: 

$$\frac{109}{468} = 0.232906$$

And if we logged our odds of survival for men:

$$log(0.232906) = -1.457$$

Which is almost identical to what we have also gotten from skearn's fitting. In
essence, **the intercept term from the logistic regression is the log odds of
our base reference term**, which is men who has survived. 
 
#### Survival for Females

Now that we understand what the bias coefficient means in the logistic
regression. Naturally, adding $$\theta_1$$ gives us the survival probability if
female.

Don't take my words for it yet, we'll verify that $$\theta_1 = 2.513$$ for
ourselves.

$$ log \left( \frac{h(survived | x = female)}{h(\overline{survived} | x =
female)} \right) = -1.45707 + 2.51366 = 1.05659$$ 

**In [7]:**

{% highlight python %}
survived_by_sex # same table as above
{% endhighlight %}




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Survived</th>
      <th>Total</th>
      <th>NotSurvived</th>
      <th>OddsOfSurvival</th>
      <th>ProbOfSurvival</th>
      <th>Log(OddsOfSurvival)</th>
    </tr>
    <tr>
      <th>Sex</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>109</td>
      <td>577</td>
      <td>468</td>
      <td>0.232906</td>
      <td>0.188908</td>
      <td>-1.457120</td>
    </tr>
    <tr>
      <th>1</th>
      <td>233</td>
      <td>314</td>
      <td>81</td>
      <td>2.876543</td>
      <td>0.742038</td>
      <td>1.056589</td>
    </tr>
  </tbody>
</table>
</div>


 
Out of 314 female on board, 233 survived, but 81 didn't. So the odds of survival
for females is:

$$ \frac{233}{81} = 2.876$$

Then taking the log both sides gives us:

$$ log(2.876) = 1.056$$

And this is also what sklearn gave us as its $$\theta^Tx$$ from above.

Voila, nothing too ground breaking. Although it's interesting to understand the
relationship between $$h(\theta^Tx)$$ and $$\theta$$. 
 
## Where to Go from Here


If you're still trying to make more connections to how the logistic regression
is derived, I would point you in the direction of the Bernoulli distribution,
how the bernoulli can be expressed as part of the exponential family, and how
Generalized Linear Model can produces a learning algorithm for all members of
the exponential family. 

Thank you Sebastien for the wonderful cover art, more of his work can be found [here](http://www.deviantart.com/art/Math-297430290){:target="blank"}.
