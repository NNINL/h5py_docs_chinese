.. _quick:

快速入门指南
=================

安装
-------

使用 `Anaconda <http://continuum.io/downloads>`_ 或
`Miniconda <http://conda.pydata.org/miniconda.html>`_::

    conda install h5py


如果在您的平台(mac, linux, windows on x86)上有 wheels (*.whl) 并且不需要MPI，你可以通过pip安装 ``h5py`` ::

  pip install h5py

对于 `Enthought Canopy <https://www.enthought.com/products/canopy/>`_, 
使用GUI包管理器 或::

    enpkg h5py

从源代码安装 参阅: :ref:`install`.

核心概念
-------------

HDF5里有两种对象的容器 `datasets` (类似数组的数据集) 与 `groups`(类似文件夹),使用h5py时要记住的最基本的一点是：

    **Groups 像字典一样工作,  datasets 像NumPy数组一样工作**

假设有人向您发送了一个HDF5文件, :code:`mytestfile.hdf5`. (要创建这样的文件，请阅读 `Appendix: Creating a file`_.) 您第一步需要做的事, 是打开该文件 进行读取::

    >>> import h5py
    >>> f = h5py.File('mytestfile.hdf5', 'r')

这个文件 :ref:`File object <file>` 是您的起点. 此文件中存储了什么内容? 记住 :py:class:`h5py.File` 文件就像Python字典，因此我们可以检查键，

    >>> list(f.keys())
    ['mydataset']

根据我们的观察, 他有一个数据集(datasets), :code:`mydataset` 在文件里.
让我们像字典一样检查 :ref:`Dataset <dataset>` 对象

    >>> dset = f['mydataset']

我们获得的对象不是数组, 是一个:ref:`HDF5 dataset <dataset>`.
像 NumPy 中的 arrays, dataset 具有数据形式(shape/形状/长度)和数据类型(data type)：

    >>> dset.shape
    (100,)
    >>> dset.dtype
    dtype('int32')

它们还支持array式(numpy.ndarray)的切片。下面是如何读取和写入文件中的(dataset)数据集::

    >>> dset[...] = np.arange(100)
    >>> dset[0]
    0
    >>> dset[10]
    10
    >>> dset[0:100:10]
    array([ 0, 10, 20, 30, 40, 50, 60, 70, 80, 90])

更多, 详看 :ref:`file` and :ref:`dataset`.

附录：创建文件
+++++++++++++++++++++++++

此时, 你可能会想知道是怎么创建 :code:`mytestdata.hdf5` 的.
我们可以通过设置 :code:`mode`  :code:`w` 写入模式 初始化File对象. 其他一些模式包括 :code:`a`
(用于 读/写/创建 访问), 和
:code:`r+` (用于 读/写 访问).
完整的 文件访问模式及其含义 列表位于 :ref:`file`. ::

    >>> import h5py
    >>> import numpy as np
    >>> f = h5py.File("mytestfile.hdf5", "w")

文件中 :ref:`File object <file>` 有几个方法看起来很有趣. 其中之一是 ``create_dataset``(创建dataset数据集), 顾名思义，数据集(dataset)创建时 需要设置 形状(shape) 和 数据类型(dtype) ::

    >>> dset = f.create_dataset("mydataset", (100,), dtype='i')

File()对象是上下文管理器；所以下面的代码也起作用(使用with) ::

    >>> import h5py
    >>> import numpy as np
    >>> with h5py.File("mytestfile.hdf5", "w") as f:
    >>>     dset = f.create_dataset("mydataset", (100,), dtype='i')


组(Groups)和层级组织(hierarchical organization)
------------------------------------

"HDF" 代表 "分层数据格式(Hierarchical Data Format)".  HDF5文件中的每一个对象，都有一个名字, 它们以 POSIX风格 结构排列，带有``/``分隔符::

    >>> dset.name
    '/mydataset'

我们创建的File对象本身就是一个组:ref:`groups <group>`.  我们创建的 ``File`` 对象本身就是一个组(groups), 在本实例中是"根目录(root group)" ``/``:

    >>> f.name
    '/'

创建子组是通过方法 ``create_group``完成的. 追加('a')”模式打开文件（如果文件存在，则用读取/写入，否则创建文件） ::

    >>> f = h5py.File('mydataset.hdf5', 'a')
    >>> grp = f.create_group("subgroup")

所有 ``Group`` 对象都有 ``create_*`` 的方法,在"文件夹"里可以继续创建"文件夹/数据集"：::

    >>> dset2 = grp.create_dataset("another_dataset", (50,), dtype='f')
    >>> dset2.name
    '/subgroup/another_dataset'

顺便说一句，您不必手动创建所有[中间组]。指定完整路径就会创建路径中的中间组了：::

    >>> dset3 = f.create_dataset('subgroup2/dataset_three', (10,), dtype='i')
    >>> dset3.name
    '/subgroup2/dataset_three'

组(Groups)支持大多数Python的字典风格。使用索引语法检索文件中的对象::

    >>> dataset_three = f['subgroup2/dataset_three']

循环出 组(Groups)里的成员的名称::

    >>> for name in f:
    ...     print(name)
    mydataset
    subgroup
    subgroup2

组(Groups)也可以使用成员资格测试, "xx"是否在组里::

    >>> "mydataset" in f
    True
    >>> "somethingelse" in f
    False

您甚至可以使用完整路径名,"xx/yyy/z"是否在组里::

    >>> "subgroup/another_dataset" in f
    True

还有使用熟悉的 ``keys()``, ``values()``, ``items()`` 和
``iter()`` 方法, 以及 ``get()``.

由于遍历一个组(Group)只会输出直接在组下的成员(组下的组下成员不会输出),如果要遍历整个文件下的所有成员,使用``Group`` 里的方法:``visit()`` 和 ``visititems()``,它可以接受一个 回调函数::

    >>> def printname(name):
    ...     print(name)
    >>> f.visit(printname)
    mydataset
    subgroup
    subgroup/another_dataset
    subgroup2
    subgroup2/dataset_three

有关更多信息，请参阅:ref:`group`.

属性
----------

HDF5 最佳功能之一就是, 您可以将元数据(metadata)存储在它描述的数据旁边。所有组(Groups)和数据集(Datasets)都支持附加的自定义数据(.attrs["xxx"]=自定义数据)，该数据存储在属性(attributes)里。

通过 ``attrs`` 代理对象访问属性, 代理对象(attrs)有像字典风格的接口::

    >>> dset.attrs['temperature'] = 99.5
    >>> dset.attrs['temperature']
    99.5
    >>> 'temperature' in dset.attrs
    True

有关更多信息，请参阅 :ref:`attributes`.
