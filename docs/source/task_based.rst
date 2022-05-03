.. _label_task_based:

Task Based Feature Detection
============================

Overview
^^^^^^^^

Segmentations of the domain derived from merge trees have been shown to efficiently encode threshold-based features. For example, segmented merge trees can be used to extract extinction regions defined as areas of high scalar dissipation in turbulent combustion simulations or also burning cells in turbulent combustion, eddies in the oceans, and bubbles in Raleigh-Taylor instabilities. Segmented merge-trees provide two key advantages over traditional threshold-based segmentation techniques: 1) they efficiently encode a wide range of possible segmentations, and 2) they allow for the selection of localized thresholds.

We implemented our feature extraction algorithm using a task based approach where we define the algorithm using an abstract task graphs that can be execute on multiple runtimes (using the BabelFlow library). The computation is composed of four types of tasks: local computation, join, correction and segmentation. The local computation at the leaf nodes (of the task graph) receive the input data and produces two outputs: a local tree and a boundary tree. These are sent to the correction and join tasks, respectively. All but the leaf tasks of the reduction perform the join of two (or more) boundary trees and send the other boundary tree to the next join and an augmented boundary tree to as many correction stages as there are leaves in the subtree of the join. The correction uses the augmented boundary tree and the local tree to update the local tree and sends it to the next correction stage. Once all joins have been passed to a correction, each local tree is passed to a final segmentation.


Getting Started
^^^^^^^^^^^^^^^

The task based feature segmentation is usable through Ascent when built via Spack/uberenv enabling the variant "+babelflow". The filter is called "bflow_pmt" and it accepts the following set of parameters:

  * task: the name of the algorithm to execute, now only "bflow_pmt" is currently supported (default = "bflow_pmt")
  * field: name of the input field (scalar and float64)
  * fanin: defines the reduce/broadcast factor to use (e.g., fanin=2 will merge two merge trees at a time), it is preferrable to use 2 for 2D datasets and 8 for 3D datasets
  * threshold: the threshold for the merge tree construction, all value below this thershold will be discarded
  * gen_segment: generate (1) or not (0) a new field containing the segmentation named "segment"

Some optional parameters are:

  * in_ghosts: number of ghost cells used by the input domain decomposition (default = [1,1,1,1,1,1])
  * ugrid_select: if dataset contains multiple uniform grids, select the index of the grid to use for the analysis (default = 0)

**Task-based compositing**

Another feature of BabelFlow is task-based, parallel compositing of images. This feature is available as Ascent extract called "bflow_comp". The extract is designed to be used with a Devil Ray filter called "dray_project_colors_2d". This filter renders images of local data and passes them down the Ascent's pipeline as Blueprint uniform mesh data with two fields. One field, called "color_field", is the pixel data and the other one, called "depth_field", is the depth data. This designs allows us to add future renderers and support other potential compositing patterns. The compositing extract accepts the following set of parameters:

 * pipeline: the name of the pipeline on which to apply this extract
 * color_field: the name of the field with the pixel data
 * depth_field: the name of the field with depth data
 * image_prefix: the prefix for the output image name
 * compositing: specifies which compositing technique to use, 0 means single reduce tree, 1 means binary swap, and 2 means use the radix-k algorithm 
 * fanin: defines the reduce factor to use when image fragments are reduced into one image in the end of the Radix-k exchange pattern; for higher process counts it is recommended to use higher fanin such as 4 or 8 to reduce the number of levels in the reduce tree
 * radices: array of radices for radix-k algorithm

**Relevance computation**

One extension on top of the feature segmentation filter is computing the relevance field. Relevance, which is a value in the range [0.0, 1.0], is computed from an existing merge tree and aims to rank points according to their relative distance in function space from the most dominant point in their neighborhood. Essentially, it reduces the potetially wide dynamic range of the original data into a fixed range that allows highlighting both local and global features. 

The dataflow graph for the relevance computations is constructed by composing the PMT dataflow with a dataflow to compute the global merge tree and a layer to compute the relevance field. For the relevance computation the user needs to use the same "bflow_pmt" type as for the plain PMT filter, but there is an additional parameter called "rel_field" that needs to be switched on.

**Streaming statistics**

Streaming statistics is an approach to compute statistical metrics such as average, median, or kurtosis in a streaming fashion, that is updating the metric after each data point become available. In the context of PMT, we are interested in computing streaming statistics per feature as identified by the merge tree. In other words, look at segmented regions and compute statistics on these regions. 

We use a standalone, easy-to-use library called `STREAMSTAT <https://github.com/LLNL/STREAMSTAT>`_. The library allows users to compute a number of statistic metrics on a stream of data. Supported metrics are: count, minimum, maximum, sum, mean (1st moment), variance (2nd moment), skewness (3rd moment), and kurtosis (4th moment). Each time a new value is available a statistics metric is updated and only a constant amount of information is retained for further updates.

The dataflow graph is constructed by composing the PMT dataflow with a Radix-k subgraph that essentially performs streaming statistics compositing. In the first phase of the compositing, per-feature statistics are computed on the local part of the segmentation. Then the statistics are split according to the location of each of the features. In the second layer, all incoming statistics are merged using the streaming capability in STREAMSTAT and are further split. In the final layer, the statistics are merged and passed to the next subgraph. The subgraph that follows the Radik-k compositing is a k-way reduction dataflow that merges the per feature statistics and the local merge trees such that eventually we write both the global tree (i.e., features) and the statistics to the output.

We use a topology oriented format called *TopologyFileReader* for the output. It is part of `TALASS <https://bitbucket.org/cedmav/talass/src/develop>`_. It stands for Topological Analysis of Large-Scale Simulations, a joint University of Utah and LLNL project. The TopologyFileReader format stores per-feature statistics along with the merge tree itself. The format is used in other projects, such as NIF target analysis, and was also used in previous work focused on combustion analysis. In order to use the TopologyFileReader from within Ascent, we introduced various changes that facilitated this integration. This format is not parallel; therefore, we needed to merge all the statistics into one structure so that a single task could write it to the output. 

Use Case Examples
^^^^^^^^^^^^^^^^^

**Input and output**

*Limitations*: This filter currently supports only uniform grids and scalar fields (with datatype float64).
The filter can generate a new field named "segment" containing a new uniform grid (equivalent to the input field's one) containing the segmentation values.

**Example**

Here is an actions file which takes as input a scalar field "mag" and perform a topological segmentation using a threshold (e.g., 0.0025). The segmentation is stored by the filter in a new variable named ``segment``, in the following example pipeline this variable saved by relay extract to a blueprint file.

To enable the relevance computation, the user needs to set the flag ``rel_field`` to 1. For streaming statistics, ``rel_field`` has no meaning and ``gen_segment`` should be 0 as we do not want to add the segmentation into the output Blueprint node. To enable streaming statistics, users should specify the list of statistics to be computed ``stream_stat_types``. The codes in the list are as follows:

1. Count
2. Mean
3. Maximum
4. Minimum
5. Sum
6. Variance
7. Skewness
8. Kurtosis

Both the relevance computation and streaming statistics use the Radix-k dataflow. In both cases, the ``radices`` list can be specified with the radices to use in the dataflow. This gives users the option to adjust the Radix-k algorithm according to their needs.

.. code-block:: yaml

  -
    action: "add_pipelines"
    pipelines:
      pl2:
        f1:
          type: "bflow_pmt"
          params:
            field: "mag"
            fanin: 2
            threshold: 0.0025
            gen_segment: 1
            rel_field: 1   # Optional parameter to enable relevance computation
            radices: [2, 2]  # Optional list of radices for the Radix-k dataflow
            stream_stat_types: [1, 3, 4]  # Optional list of streaming statistic metrics (in this example: count, min, sum)
  -
    action: "add_extracts"
    extracts:
      e1:
        type: "relay"
        pipeline: "pl2"
        params:
          path: "seg"
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
  pipelines["pl1/f1/type"] = "bflow_pmt";
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

**Task-based compositing**

Below is the actions file for the compositing pipeline with Devil Ray renderer and BabelFlow compositing:

.. code-block:: yaml

  -
    action: "add_pipelines"
    pipelines:
      pl1:
        f1:
          type: "dray_project_colors_2d"
          params:
            field: "density"
            color_table:
              name: "cool2warm"
            camera:
              azimuth: -30
              elevation: 35
  -
    action: "add_extracts"
    extracts:
      e1:
        type: "bflow_comp"
        pipeline: pl1
        params: 
          color_field: "colors"
          depth_field: "depth"
          image_prefix: "comp_img"
          fanin: 2
          compositing: 2
          radices: [2, 4]

The equivaled pipeline in C++:

.. code-block:: C++

  Ascent a;
  conduit::Node ascent_opt;
  ascent_opt["mpi_comm"] = MPI_Comm_c2f(MPI_COMM_WORLD);
  ascent_opt["runtime/type"] = "ascent";
  a.open(ascent_opt);

  std::vector<int64_t> radices({2, 4});

  Node pipelines;
  pipelines["pl1/f1/type"] = "dray_project_colors_2d";
  pipelines["pl1/f1/params/field"] = "density";
  pipelines["pl1/f1/params/color_table/name"] = "cool2warm";
  pipelines["pl1/f1/params/camera/azimuth"] = -30;
  pipelines["pl1/f1/params/camera/elevation"] = 35;

  Node extract;
  extract["e1/type"] = "bflow_comp";
  extract["e1/pipeline"] = "pl1";
  extract["e1/params/color_field"] = "colors";
  extract["e1/params/depth_field"] = "depth";
  extract["e1/params/image_prefix"] = "comp_img";
  extract["e1/params/fanin"] = 2;
  extract["e1/params/compositing"] = 2;
  extract["e1/params/radices"].set_int64_vector(radices);

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

Our implementation using a task based abstraction layer (BabelFlow) allows for easy portability of analytics over multiple runtimes (i.e., the current implementation uses MPI). To avoid large global communications and improve overall performance reduction and broadcast communications are defined in the task graph as k-way reduction/broadcast trees (where k in the number of input/output nodes at each level of the tree). 


Developers
^^^^^^^^^^
  * Peer-Timo Bremer (LLNL)
  * Xuan Huang (University of Utah)
  * Steve Petruzza (University of Utah)
  * Sergei Shudler (LLNL)


.. toctree::
   :maxdepth: 1
