``cleanlab``
============

A Python package for cleaning/fixing errors in dataset labels using
state-of-the-art algorithms for multiclass learning with noisy labels,
detection of label errors in massive datasets, latent noisy channel
estimation, latent prior estimation, and much more. This package
implements the theory and algorithms of the machine learning subfield
known as confident learning.

``cleanlab`` is:
----------------

1. fast - only two hours (on cpu-based laptop) to find the label errors
   in the 2012 ImageNet Validation set
2. robust - provable generalization and risk minimimzation guarantees
   even for imperfect probability estimation
3. general - works with any probablistic classifier, Faster R-CNN,
   logistic regression, LSTM, etc.
4. unique - the only general implementation for multiclass learning with
   noisy lables for ANY classifier (deep or not), for ANY
   amount/distribution of label noise distribution, with no information
   about the label noise needed a priori.

Check out these `examples <examples>`__.

Installation
------------

Python 2.7 and Python 3.5 are supported.

To install the ``cleanlab`` package with pip, just run:

::

   $ pip install git+https://github.com/cgnorthcutt/cleanlab.git

If you have issues, you can also clone the repo and install:

::

   $ conda update pip # if you use conda
   $ git clone https://github.com/cgnorthcutt/cleanlab.git
   $ cd cleanlab
   $ pip install -e .

Citations and Related Publications
----------------------------------

Although this package goes far beyond our 2017 publication, if you find
this repository helpful, please cite our paper
http://auai.org/uai2017/proceedings/papers/35.pdf. New papers will be
posted here when they are published.

::

   @inproceedings{northcutt2017rankpruning,
    author={Northcutt, Curtis G. and Wu, Tailin and Chuang, Isaac L.},
    title={Learning with Confident Examples: Rank Pruning for Robust Classification with Noisy Labels},
    booktitle = {Proceedings of the Thirty-Third Conference on Uncertainty in Artificial Intelligence},
    series = {UAI'17},
    year = {2017},
    location = {Sydney, Australia},
    numpages = {10},
    url = {http://auai.org/uai2017/proceedings/papers/35.pdf},
    publisher = {AUAI Press},
   } 

Get started with easy, quick examples.
--------------------------------------

New to **cleanlab**? Start with:

1. `Visualizing confident
   learning <examples/visualizing_confident_learning.ipynb>`__
2. `A simple example of learning with noisy labels on the multiclass
   Iris dataset <examples/iris_simple_example.ipynb>`__.

These examples show how easy it is to characterize label noise in
datasets, learn with noisy labels, identify label errors, estimate
latent priors and noisy channels, and more.

.. raw:: html

   <!---

   ## Automatically identify ~50 label errors in MNIST with cleanlab. [[link]](examples/finding_MNIST_label_errors).
   ![Image depicting label errors in MNIST train set.](https://raw.githubusercontent.com/cgnorthcutt/cleanlab/master/img/mnist_training_label_errors24_prune_by_noise_rate.png)
   Label errors of the original MNIST **train** dataset identified algorithmically using the rankpruning algorithm. Depicts the 24 least confident labels, ordered left-right, top-down by increasing self-confidence (probability of belonging to the given label), denoted conf in teal. The label with the largest predicted probability is in green. Overt errors are in red.

   ![Image depicting label errors in MNIST test set.](https://raw.githubusercontent.com/cgnorthcutt/cleanlab/master/img/mnist_test_label_errors8.png)
    Selected label errors in the MNIST **test** dataset ordered by increasing self-confidence (in teal).

   ## Automatically identify ~5k (of 50k) validation set label errors in ImageNet. [[link]](examples/finding_ImageNet_label_errors).
   ![Image depicting label errors in ImageNet validation set.](https://raw.githubusercontent.com/cgnorthcutt/cleanlab/master/img/imagenet_validation_label_errors_96_prune_by_noise_rate.jpg)
   Label errors in the 2012 ImageNet validation dataset identified automatically with cleanlab using a pre-trained resnet18. Displayed are the 96 least confident labels. We see that ImageNet contains numerous multi-label images, although it is used widely by the machine learning and vision communities as a single-label benchmark dataset.

   --->

Use ``cleanlab`` with any model (Tensorflow, caffe2, PyTorch, etc.)
-------------------------------------------------------------------

All of the features of the ``cleanlab`` package work with **any model**.
Yes, any model. Feel free to use PyTorch, Tensorflow, caffe2,
scikit-learn, mxnet, etc. If you use a scikit-learn classifier, all
``cleanlab`` methods will work out-of-the-box. It’s also easy to use
your favorite model from a non-scikit-learn package, just wrap your
model into a Python class that inherets the
``sklearn.base.BaseEstimator``:

.. code:: python

   from sklearn.base import BaseEstimator
   class YourFavoriteModel(BaseEstimator): # Inherits sklearn base classifier
       def __init__(self, ):
           pass
       def fit(self, X, y, sample_weight = None):
           pass
       def predict(self, X):
           pass
       def predict_proba(self, X):
           pass
       def score(self, X, y, sample_weight = None):
           pass
           
   # Now you can use your model with `cleanlab`. Here's one example:
   from cleanlab.classification import RankPruning
   rp = RankPruning(clf=YourFavoriteModel())
   rp.fit(train_data, train_labels_with_errors)

Want to see a working example? `Here’s a compliant PyTorch MNIST CNN class <https://github.com/cgnorthcutt/cleanlab/blob/master/examples/models/mnist_pytorch.py#L28>`__
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As you can see
`here <https://github.com/cgnorthcutt/cleanlab/blob/master/examples/models/mnist_pytorch.py#L28>`__,
technically you don’t actually need to inherit from
``sklearn.base.BaseEstimator``, as you can just create a class that
defines .fit(), .predict(), and .predict_proba(), but inheriting makes
downstream scikit-learn applications like hyper-parameter optimization
work seamlessly. For example, the `RankPruning()
model <https://github.com/cgnorthcutt/cleanlab/blob/master/cleanlab/classification.py#L48>`__
is fully compliant.

Note, some libraries exists to do this for you. For pyTorch, check out
the ``skorch`` Python library which will wrap your ``pytorch`` model
into a ``scikit-learn`` compliant model.

Documentation by Example - Quick Tutorials
------------------------------------------

Many of these methods have default parameters that won’t be covered
here. Check out the method docstrings for full documentation.

Multiclass learning with noisy labels (in **3** lines of code):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**rankpruning** is a fast, general, robust algorithm for multiclass
learning with noisy labels. It adds minimal overhead, needing only
*O(nm2)* time for n training examples and m classes, works with any
classifier, and is easy to use.

.. code:: python

   from cleanlab.classification import RankPruning
   # RankPruning uses logreg by default, so this is unnecessary. 
   # We include it here for clarity, but this step is omitted below.
   from sklearn.linear_model import LogisticRegression as logreg

   # 1.
   # Wrap around any classifier. Yup, neural networks work, too.
   rp = RankPruning(clf=logreg()) 

   # 2.
   # X_train is numpy matrix of training examples (integers for large data)
   # train_labels_with_errors is a numpy array of labels of length n (# of examples), usually denoted 's'.
   rp.fit(X_train, train_labels_with_errors) 

   # 3.
   # Estimate the predictions you would have gotten by training with *no* label errors.
   predicted_test_labels = rp.predict(X_test)

Estimate the confident joint, the latent noisy channel matrix, *Ps \| y* and inverse, *Py \| s*, the latent prior of the unobserved, actual true labels, *p(y)*, and the predicted probabilities.:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

where *s* denotes a random variable that represents the observed, noisy
label and *y* denotes a random variable representing the hidden, actual
labels. Both *s* and *y* take any of the m classes as values. The
``cleanlab`` package supports different levels of granularity for
computation depending on the needs of the user. Because of this, we
support multiple alternatives, all no more than a few lines, to estimate
these latent distribution arrays, enabling the user to reduce
computation time by only computing what they need to compute, as seen in
the examples below.

Throughout these examples, you’ll see a variable called
*confident_joint*. The confident joint is an m x m matrix (m is the
number of classes) that counts, for every observed, noisy class, the
number of examples that confidently belong to every latent, hidden
class. It counts the number of examples that we are confident are
labeled correctly or incorrectly for every pair of obseved and
unobserved classes. The confident joint is an unnormalized estimate of
the complete-information latent joint distribution, *Ps,y*. Most of the
methods in the **cleanlab** package start by first estimating the
*confident_joint*.

Option 1: Compute the confident joint and predicted probs first. Stop if that’s all you need.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

   from cleanlab.latent_estimation import estimate_latent
   from cleanlab.latent_estimation import estimate_confident_joint_and_cv_pred_proba

   # Compute the confident joint and the n x m predicted probabilities matrix (psx),
   # for n examples, m classes. Stop here if all you need is the confident joint.
   confident_joint, psx = estimate_confident_joint_and_cv_pred_proba(
       X=X_train, 
       s=train_labels_with_errors,
       clf = logreg(), # default, you can use any classifier
   )

   # Estimate latent distributions: p(y) as est_py, P(s|y) as est_nm, and P(y|s) as est_inv
   est_py, est_nm, est_inv = estimate_latent(confident_joint, s=train_labels_with_errors)

Option 2: Estimate the latent distribution matrices in a single line of code.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

   from cleanlab.latent_estimation import estimate_py_noise_matrices_and_cv_pred_proba
   est_py, est_nm, est_inv, confident_joint, psx = estimate_py_noise_matrices_and_cv_pred_proba(
       X=X_train,
       s=train_labels_with_errors,
   )

Option 3: Skip computing the predicted probabilities if you already have them
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

   # Already have psx? (n x m matrix of predicted probabilities)
   # For example, you might get them from a pre-trained model (like resnet on ImageNet)
   # With the cleanlab package, you estimate directly with psx.
   from cleanlab.latent_estimation import estimate_py_and_noise_matrices_from_probabilities
   est_py, est_nm, est_inv, confident_joint = estimate_py_and_noise_matrices_from_probabilities(
       s=train_labels_with_errors, 
       psx=psx,
   )

Estimate label errors in a dataset:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With the ``cleanlab`` package, we can instantly fetch the indices of all
estimated label errors, with nothing provided by the user except a
classifier, examples, and their noisy labels. Like the previous example,
there are various levels of granularity.

.. code:: python

   from cleanlab.pruning import get_noise_indices
   # We computed psx, est_inv, confident_joint in the previous example.
   label_errors = get_noise_indices(
       s=train_labels_with_errors, # required
       psx=psx, # required
       inverse_noise_matrix=est_inv, # not required, include to avoid recomputing
       confident_joint=confident_joint, # not required, include to avoid recomputing
   )

Estimate the latent joint probability distribution matrix of the noisy and true labels, *Ps,y*:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are two methods to compute *Ps,y*, the complete-information
distribution matrix that captures the number of pairwise label flip
errors when multipled by the total number of examples as *n* Ps,y\*.

Method 1: Guarantees the rows of *Ps,y* correctly sum to *p(s)*, by first computing *Py \| s*.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This method occurs when hyperparameter prune_count_method =
‘inverse_nm_dot_s’ in RankPruning.fit() and get_noise_indices().

.. code:: python

   from cleanlab.util import value_counts
   # *p(s)* is the prior of the observed, noisy labels and an array of length m (# of classes)
   ps = value_counts(s) / float(len(s))
   # We computed est_inv (estimated inverse noise matrix) in the previous example (two above).
   psy = np.transpose(est_inv * ps) # Matrix of prob(s=l and y=k)

Method 2: Simplest. Compute by re-normalizing the confident joint. Rows won’t sum to *p(s)*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This method occurs when hyperparameter prune_count_method =
‘calibrate_confident_joint’ in RankPruning.fit() and
get_noise_indices().

.. code:: python

   from cleanlab.util import value_counts
   # *p(s)* is the prior of the observed, noisy labels and an array of length m (# of classes)
   ps = value_counts(s) / float(len(s))
   # We computed confident_joint in the previous example (two above).
   psy = confident_joint / float(confident_joint.sum()) # calibration, i.e. re-normalization

Generate valid, class-conditional, unformly random noisy channel matrices:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

   # Generate a valid (necessary conditions for learnability are met) noise matrix for any trace > 1
   from cleanlab.noise_generation import generate_noise_matrix_from_trace
   noise_matrix = generate_noise_matrix_from_trace(
       K = number_of_classes, 
       trace = float_value_greater_than_1_and_leq_K,
       py = prior_of_y_actual_labels_which_is_just_an_array_of_length_K,
       frac_zero_noise_rates = float_from_0_to_1_controlling_sparsity,
   )

   # Check if a noise matrix is valid (necessary conditions for learnability are met)
   from cleanlab.noise_generation import noise_matrix_is_valid
   is_valid = noise_matrix_is_valid(noise_matrix, prior_of_y_which_is_just_an_array_of_length_K)

Support for numerous *weak supervision* and *learning with noisy labels* functionalities:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

   # Generate noisy labels using the noise_marix. Guarantees exact amount of noise in labels.
   from cleanlab.noise_generation import generate_noisy_labels
   s_noisy_labels = generate_noisy_labels(y_hidden_actual_labels, noise_matrix)

   # This package is a full of other useful methods for learning with noisy labels.
   # The tutorial stops here, but you don't have to. Inspect method docstrings for full docs.

The Polyplex
------------

The key to learning in the presence of label errors is estimating the joint distribution between the actual, hidden labels ‘*y*’ and the observed, noisy labels ‘*s*’. Using ``cleanlab`` and theory of confident learning, we can completely characterize the trace of the latent joint distribution, *trace(Ps,y)*, given *p(y)*, for any fraction of label errors, i.e. for any trace of the noisy channel, *trace(Ps|y)*.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can check out how to do this yourself here: 1. `Drawing
Polyplices <examples/drawing_polyplices.ipynb>`__ 2. `Computing
Polyplices <cleanlab/polyplex.ipynb>`__