.. _label_zfp_compression:

zfp Compression
^^^^^^^^^^^^^^^

zpf is a floating-point array primitive using very high-speed, lossy (but optionally error-bounded) compression to significantly reduce data volumes. zfp reduces I/O time and off-line storage requirements by 1-2 orders of magnitude depending on accuracy requirements, as dictated by user-set error tolerances. Unique among data compressors, zfp also supports constant-time read/write random access to individual array elements from compressed storage. zfp's compressed arrays can often replace conventional arrays in existing applications with minimal code changes, allowing, for example, the user to store tables of floating-point data in compressed form that otherwise would not fit in memory. When used in numerical computations, zfp arrays provide a fine-grained knob on precision while achieving accuracy comparable to IEEE floating point at half the storage, reducing both memory usage and bandwidth.

More information can be found on zfp compression on the LLNL `zfp website`_ or `zfp GitHub`_ page.


.. _ParaView : https://www.paraview.org/
.. _VisIt : https://wci.llnl.gov/simulation/computer-codes/visit
.. _Ascent : https://alpine-dav.github.io/ascent/
.. _Data Science at Scale Team : https://dsscale.org
.. _Los Alamos National Laboratory :  https://www.lanl.gov
.. _Lawrence Livermore National Laboratory : https:/www.llnl.gov
.. _Lawrence Berkeley National Laboratory : https://www.lbl.gov/
.. _Oak Ridge National Laboratory : https://www.ornl.gov/
.. _Sandia National Laboratories : https://www.sandia.gov/
.. _Argonne National Laboratory : https://www.anl.gov/
.. _University of Oregon : https://www.uoregon.edu/
.. _University of Utah : https://www.utah.edu/
.. _University of Leeds : http://www.leeds.ac.uk/
.. _Kitware : https://www.kitware.com/
.. _Exascale Computing Project : https://www.exascaleproject.org/
.. _zfp GitHub : https://github.com/LLNL/zfp
.. _zfp website : https://computing.llnl.gov/projects/floating-point-compression

.. toctree::
   :maxdepth: 1
