=====
Setup
=====


Requirements
------------

* Python>=2.7
* Django>=1.3
* Django REST Framework >= 3.0.0
* REST Framework >= 2.2.5
  (**only with** Django<1.8 since DRF<3 does not support Django1.8)

Installation
------------

Install from PyPI with ``pip``::

    pip install drf-bulk

Or Install from ``source code``::

    pip install -e git+https://github.com/101Loop/drf-bulk#egg=drf-bulk

    
To use ``drf-bulk`` in your Django project, just import and
use the views and mixins described in this documentation; there is no need to
modify your ``INSTALLED_APPS`` setting.
