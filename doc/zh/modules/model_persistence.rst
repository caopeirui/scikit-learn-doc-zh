.. _model_persistence:

=================
模型持久化
=================

在训练完 scikit-learn 模型之后, 最好有一种方法来将模型持久化以备将来使用，而无需重新训练.
以下部分为您提供了有关如何使用 pickle 来持久化模型的示例.
在使用 pickle 序列化时，我们还将回顾一些安全性和可维护性方面的问题.


持久化示例
-------------------

可以通过使用 Python 的内置持久化模型将训练好的模型保存在 scikit 中, 它名为 `pickle <https://docs.python.org/2/library/pickle.html>`_::

  >>> from sklearn import svm
  >>> from sklearn import datasets
  >>> clf = svm.SVC()
  >>> iris = datasets.load_iris()
  >>> X, y = iris.data, iris.target
  >>> clf.fit(X, y)  # doctest: +NORMALIZE_WHITESPACE
  SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
      decision_function_shape='ovr', degree=3, gamma='auto', kernel='rbf',
      max_iter=-1, probability=False, random_state=None, shrinking=True,
      tol=0.001, verbose=False)

  >>> import pickle
  >>> s = pickle.dumps(clf)
  >>> clf2 = pickle.loads(s)
  >>> clf2.predict(X[0:1])
  array([0])
  >>> y[0]
  0

在这个 scikit 的特殊示例中，使用 joblib 来替换 pickle (``joblib.dump`` & ``joblib.load``) 可能会更有意思, 这对于内部带有大量数组的对象来说更为高效, 通常情况下适合 scikit-learn estimators（预估器）, but can only pickle to the disk and not to a string::

  >>> from sklearn.externals import joblib
  >>> joblib.dump(clf, 'filename.pkl') # doctest: +SKIP

之后你可以使用以下方式加载 pickled model（可能在另一个 Python 进程中）::

  >>> clf = joblib.load('filename.pkl') # doctest:+SKIP

.. note::

   ``joblib.dump`` 和 ``joblib.load`` 函数也接收类似 file 的对象而不是文件名.
   有关使用 Joblib 来持久化数据的更多信息可以参阅 `这里 <https://pythonhosted.org/joblib/persistence.html>`_.

.. _persistence_limitations:

安全性和可维护性的局限性
--------------------------------------

pickle (和通过扩展的 joblib), 在安全性和可维护性方面存在一些问题.
由于以下原因,

* 不要打开不受信任的数据, 因为它可能导致恶意代码在加载时执行.
* 虽然使用一个版本的 scikit-learn 保存的模型可能会在其他版本中加载，但这完全不受支持并且也不合适.
  还应该记住, 对这些数据执行的操作可能会产生不同和意想不到的结果.

为了用将来版本的 scikit-learn 来重构类似的模型, 额外的元数据应该随着 pickled model 一起被保存：

* 训练数据, 例如. 引用不可变的快照
* 用于生成模型更多 python 源代码
* scikit-learn 以及它的 dependencies 的版本
* 在训练数据的基础上获得的交叉验证得分

这样可以检查交叉验证得分是否与以前的范围相同.

由于模型内部表示可能在两种不同架构上不一样, 因此不支持在一个架构上转储模型并将其加载到另一个体系架构上.

如果您想要了解关于这些 issues 以及浏览其它可能的序列化方法的更多详情，请参阅这个
`Alex Gaynor 的演讲 <http://pyvideo.org/video/2566/pickles-are-for-delis-not-software>`_.
