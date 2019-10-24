.. _label_algorithms:


ALPINE Algorithms
^^^^^^^^^^^^^^^^^

As simulations move to exascale machines, I/O and storage become a major limitation to scientific knowledge discovery.  Exascale system concurrency is expected to grow by five or six orders of magnitude, yet I/O bandwidth is only expected to grow by two orders of magnitude.  On exascale systems, the increasing I/O bottleneck will make it necessary for most simulation data to be analyzed *in situ*, or on the supercomputer while the simulation is running.  Furthermore, to meet data bandwidth constraints, it will be necessary to sharply reduce the volume of data moved on the machine and especially the data that are exported to persistent storage. ALPINE provides a suite of in situ algorithms that will address these concerns by using adaptable data-driven techniques to downsample data, identify important features in the data, and move analysis of the data into the simulation process.

The current list of ALPINE algorithms is below.  This documentation is a work in progress and some algorithms may not yet be fully documented.. _label_introduction:.  ECP partner applications can find more information and contacts on the ECP ALPINE Confluence page.


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
   :maxdepth: 2
   :caption: Current ALPINE Algorithms

   sampling
   lagrangian
   topology
   moments
   task_based
   statistics
