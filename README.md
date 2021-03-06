![Python 3.5](https://img.shields.io/badge/python-3.5-blue.svg)

# Wine-Quality
Coursework from [Harvard CS 109](http://nbviewer.jupyter.org/github/cs109/2014/blob/master/homework/HW5.ipynb)

Predict wine quality based on the chemical properties of the wine.

A **binary classification** problem, whether the wine is good/bad.

## Data
### Data Source
Wine quality dataset from [UCI website](https://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/)

Focus on the red wine only.

### Data Preview
**11 chemical properties**, including the concentration of sugar, citric acid, alcohol, pH etc.

The **wine qualities** are represented as integers scaling from **1 to 10**.
<p align="justify">
   <img src="https://github.com/xuwenyihust/Wine-Quality/blob/master/graphs/head.JPG" width="900"/>
</p>
```python
>>> print(wine.shape)
(1599, 12)
```

### Distribution of Wine Qualities
<p align="justify">
   <img src="https://github.com/xuwenyihust/Wine-Quality/blob/master/graphs/ratings_of_wines.png" width="500"/>
</p>

## Process

* **Data Wrangling**
    * Pick out the **feature matrix X & response vector y**, transform them into Numpy arrays
    * Convert 0~10 ratings to good(1)/bad(0) **binary classifications** (Threshold = 7)

* **Model Construction** 
    * Choose **random forests** method to do the classification
    * Iterate from 1 to 40 to choose the best **'number of decision trees'** parameter
    * For each value of 'number of decision trees', use **cross_val_score** to evaluate model with different train/test splittings
    * Need to consider about the **trade-off** between **classification accuracy & cost of fitting additional trees** 

* **Handle Class Imbalance**
    * The classes are highly unbalanced
    ```python
    >>> good_ratio = sum(y) / len(y)
    >>> bad_ratio = 1 - good_ratio
    >>> print('Ratio of good wine: ', good_ratio)
    Ratio of good wine:  0.135709818637
    >>> print('Ratio of bad wine: ', bad_ratio)
    Ratio of bad wine:  0.864290181363
    ```
    * Choose **F1** rather than **accuracy** to be less sensitive to imbalance
    * Use F1 to reevaluate model using different values of 'number of decision trees'
    * Set the number of decision trees to be 15
    * Get a **mean F1 score of 0.47**
    <p align="justify">
      <img src="https://github.com/xuwenyihust/Wine-Quality/blob/master/graphs/num_of_trees.png" width="500"/>
    </p>
   
* **Probability Calibration**
    * Have the predicted labels now, also want to get the probability of the respective label
    * Construct prediction based on probabilities
    * Due to **class imbalance**, look for the suitable **cutoff value** rather than simply set it to 0.5
    * Evaluate different cutoff values under different train/test splittings using **cross-validation**
    * Choose the cutoff value to be **0.2**

* **Compare the Importance of Different Features**
    * The more the accuracy drops when the feature is permuted, the more "important" we can conclude the feature is
    * Random forests Classifier provides a **importance list**
    ```python
    >>> clf = RandomForestClassifier(n_estimators=15)
    >>> clf.fit(X,Y)
    >>> importance_list = clf.feature_importances_
    >>> print(importance_list[:10])
    [ 0.07893465  0.1145243   0.08640124  0.07171019  0.0802987   0.06206291 0.07979788  0.11030425  0.0517296   0.13133903]
    ```
    * Use a **bar chart** to visualize the importance list
    <p align="justify">
      <img src="https://github.com/xuwenyihust/Wine-Quality/blob/master/graphs/relative_importance.png" width="500"/>
    </p>

## Libraries Used
* [pandas](http://pandas.pydata.org/)
* [matplotlib](http://matplotlib.org/)
* [sklearn](http://scikit-learn.org/stable/)
* [numpy](http://www.numpy.org/)
* [seaborn](https://stanford.edu/~mwaskom/software/seaborn/)

## Appendix
### 1.How to construct a random forest?
> **Construct a multitude of decision trees** at training time and output the class that is the mode of the classes (classification) or mean prediction (regression) of the individual trees.

### 2.Advantages of random forests over decision trees?
> Random decision forests correct for decision trees' habit of **overfitting** to their training set.

### 3.Sources of randomness in the process that are used to build a diverse set of decision trees?
> The random forest adds randomness in two ways. 

> First, it randomly resamples the data with replacement, so each decision tree is being fit on a **slightly different set of data**. 

> Secondly, for each split in each decision tree, the random forests algorithm only considers **a random subset of variables to split on**. All trees are trained independently of each other. To make predictions, all trees are queried independently and the majority vote wins.

### 4.How to use cross_val_score to help evaluating models?
scores = sklearn.cross_validation.cross_val_score(model, X=..., y=..., **cv=n**, ...) *(cv: short for cross-validation)*

Evaluate a model by splitting the data, fitting a model and computing the score n consecutive times (with different splits each time).

By default, the score computed at each CV iteration is the score method of the estimator.

### 5.How to deal with class imbalance?
* Undersampling the majority class
* **Oversampling the minority class**
* Use a **asymmetric cost function** to artificially balance the training process

    (assign a higher misclassification penalty on the minority class)
* **Better performance metric**
    * **F1 Score**

## Resources
* [Harvard CS 109 HW5](http://nbviewer.jupyter.org/github/cs109/2014/blob/master/homework/HW5.ipynb)
* [UCI wine-quality dataset](https://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/)
* [Wikipedia Random Forests](https://en.wikipedia.org/wiki/Random_forest)
* [Hastie, Trevor; Tibshirani, Robert; Friedman, Jerome (2008). The Elements of Statistical Learning (2nd ed.). Springer. ISBN 0-387-95284-5.](http://statweb.stanford.edu/~tibs/ElemStatLearn/)
* [Cross-validation: evaluating estimator performance](http://scikit-learn.org/stable/modules/cross_validation.html#cross-validation)
* [Class imbalance in Supervised Machine Learning](http://stats.stackexchange.com/questions/131255/class-imbalance-in-supervised-machine-learning)

