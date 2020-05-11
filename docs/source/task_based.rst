.. _label_task_based:

Task Based Feature Detection
============================

Overview
^^^^^^^^

Getting Started
^^^^^^^^^^^^^^^

Use Case Examples
^^^^^^^^^^^^^^^^^

**Input and output**

This filter currently supports only uniform grids and scalar fields (with datatype float64).
The filter can generate a new field named "segment" containing a new uniform grid (equivalent to the input field's one) containing the segmentation values.

**Parameters**

The filter accepts the following set of parameters:

  * task: the name of the algorithm to execute, now only "pmt" is currently supported (default = "pmt")
  * field: name of the input field (scalar and float64)
  * fanin: defines the reduce/broadcast factor to use (e.g., fanin=2 will merge two merge trees at a time), it is preferrable to use 2 for 2D datasets and 8 for 3D datasets
  * threshold: the threshold for the merge tree construction, all value below this thershold will be discarded
  * gen_segment: generate (1) or not (0) a new field containing the segmentation named "segment"
  * in_ghosts: number of ghost cells used by the input domain decomposition (default = [1,1,1,1,1,1])
  * ugrid_select: if dataset contains multiple uniform grids, select the index of the grid to use for the analysis (default = 0)

**Example**

Here is an actions file which takes as input a scalar field "mag" and perform a topological segmentation using a threshold (e.g., 0.0025). The segmentation is stored by the filter in a new variable named "segment", in the following example pipeline this variable saved by relay extract to a blueprint file.

.. code-block:: yaml

  -
    action: "add_pipelines"
    pipelines:
      pl2:
        f1:
          type: "babelflow"
          params:
            task: "pmt"
            field: "mag"
            fanin: 2
            threshold: 0.0025
            gen_segment: 1
  -
    action: "add_extracts"
    extracts:
      e1:
        type: "relay"
        pipeline: "pl2"
        params:
          path: "seg0"
          protocol: "blueprint/mesh/hdf5"
          fields: 
            - "segment"

The equivaled pipeline in C++:

.. code-block:: C++

  Ascent a;
  conduit::Node ascent_opt;
  ascent_opt["mpi_comm"] = MPI_Comm_c2f(MPI_COMM_WORLD);
  ascent_opt["runtime/type"] = "ascent";
  a.open(ascent_opt);

  Node pipelines;
  pipelines["pl1/f1/type"] = "babelflow";
  pipelines["pl1/f1/params/task"] = "pmt";
  pipelines["pl1/f1/params/field"] = "mag";
  pipelines["pl1/f1/params/fanin"] = 2;
  pipelines["pl1/f1/params/threshold"] = 0.0025;
  pipelines["pl1/f1/params/gen_segment"] = 1;

  Node extract;
  extract["e1/type"] = "relay";
  extract["e1/pipeline"] = "pl1";
  extract["e1/params/path"] = "seg";
  extract["e1/params/protocol"] = "blueprint/mesh/hdf5";
  extract["e1/params/fields"].append() = "segment";

  Node actions;
  Node &add_extract = actions.append();
  add_extract["action"] = "add_extracts";
  add_extract["extracts"] = extract;

  actions.append()["action"] = "execute";

  Node &add_pipelines = actions.append();
  add_pipelines["action"] = "add_pipelines";
  add_pipelines["pipelines"] = pipelines;

  a.execute(actions);
  a.close();

Performance
^^^^^^^^^^^

Developers
^^^^^^^^^^
  * Peer-Timo Bremer (LLNL)
  * Xuan Huang (University of Utah)
  * Steve Petruzza (University of Utah)
  * Sergei Shudler (LLNL)


.. toctree::
   :maxdepth: 1
