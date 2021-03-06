GCSFS
=====

A pythonic file-system interface to `Google Cloud Storage`_.

This software is beta, use at your own risk.

Please file issues and requests on github_ and we welcome pull requests.

.. _github: https://github.com/dask/gcsfs/issues

Installation
------------

The GCSFS library can be installed using ``conda`` or ``pip``:

.. code-block:: bash

   conda install gcsfs
   or
   pip install gcsfs

or by cloning the repository:

.. code-block:: bash

   git clone https://github.com/dask/gcsfs/
   cd gcsfs/
   pip install .

Examples
--------

Locate and read a file:

.. code-block:: python

   >>> import gcsfs
   >>> fs = gcsfs.GCSFileSystem(project='my-google-project')
   >>> fs.ls('my-bucket')
   ['my-file.txt']
   >>> with fs.open('my-bucket/my-file.txt', 'rb') as f:
   ...     print(f.read())
   b'Hello, world'

(see also ``walk`` and ``glob``)

Read with delimited blocks:

.. code-block:: python

   >>> fs.read_block(path, offset=1000, length=10, delimiter=b'\n')
   b'A whole line of text\n'

Write with blocked caching:

.. code-block:: python

   >>> with fs.open('mybucket/new-file', 'wb') as f:
   ...     f.write(2*2**20 * b'a')
   ...     f.write(2*2**20 * b'a') # data is flushed and file closed
   >>> fs.du('mybucket/new-file')
   {'mybucket/new-file': 4194304}

Because GCSFS faithfully copies the Python file interface it can be used
smoothly with other projects that consume the file interface like ``gzip`` or
``pandas``.

.. code-block:: python

   >>> with fs.open('mybucket/my-file.csv.gz', 'rb') as f:
   ...     g = gzip.GzipFile(fileobj=f)  # Decompress data with gzip
   ...     df = pd.read_csv(g)           # Read CSV file with Pandas

Credentials
-----------

Two modes of authentication are supported:

    - if ``token=None`` (default), GCSFS will attempt to use your default gcloud
      credentials or, attempt to get credentials from the google metadata
      service, or fall back to anonymous access. This will work for most
      users without further action. Note that the default project may also
      be found, but it is often best to supply this anyway (only affects bucket-
      level operations).

    - if ``token='cloud'``, we assume we are running within google (compute
      or container engine) and fetch the credentials automatically from the
      metadata service.

    - you may supply a token generated by the
      gcloud_ utility; this is either a python dictionary, or the name of a file
      containing the JSON returned by logging in with the gcloud CLI tool (e.g.,
      ``~/.config/gcloud/application_default_credentials.json`` or
      ``~/.config/gcloud/legacy_credentials/<YOUR GOOGLE USERNAME>/adc.json``)
      or any value google ``Credentials`` object.

    - you can also generate tokens via oauth2 in the browser, with ``token='browser'``,
      which, once done, will be saved in a local cache for future use.

The acquired session tokens are *not* preserved when serializing the instances, so
it is safe to pass them to worker processes on other machines if using in a
distributed computation context. If credentials are given by a file path, however,
then this file must exist on every machine.

Connection with Dask and Zarr
-----------------------------

Importing gcsfs will make this file-system backend available to dask_ for
parallel data ingestion using URLs
something like ``gcs://mybucket/myfiles/*.csv``; both ``gcs:`` and ``gs:`` work.

Similarly, ``GCSMap`` is a valid mutable-mapping, which can be used with zarr_.

Contents
========

.. toctree::
   api
   developer
   fuse
   :maxdepth: 2


.. _Google Cloud Storage: https://cloud.google.com/storage/docs/

.. _gcloud: https://cloud.google.com/sdk/docs/

.. _dask: http://dask.pydata.org/en/latest/remote-data-services.html

.. _zarr: http://zarr.readthedocs.io/en/latest/tutorial.html#storage-alternatives

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
