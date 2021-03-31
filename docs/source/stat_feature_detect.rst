.. _label_stat_feature_detect:

Statistical Feature Detection
=============================

Overview
^^^^^^^^

The Statistical feature detection algorithm is an in situ feature detection algorithm that processes three-dimensional (3D) particle fields in situ and transforms the data into a feature similarity field, which is stored in the disks for further post hoc analysis. Even though our current version of the algorithm works on a particle field, the algorithm can be easily applied to any regular-grid scalar data with minor modifications. By analyzing data in situ and detecting user-interested feature regions, the algorithm outputs a statistically summarized data set, which is significantly smaller in size compared to the raw particle data and such summarized data can be analyzed interactively in post hoc analysis for further feature analysis. This algorithm follows the feature-driven data reduction paradigm to achieve significant data reduction while preserving important information so that post hoc analysis can be done on the reduced data in a timely manner.

This algorithm works on an unstructured particle field and a feature is represented as a statistical probability distribution. Representing the feature in the form of distribution allows the application scientists to specify a descriptor of the features of interest without needing to precisely define it. In many application domains, a precise description of a feature is not readily available due to the complexity of the scientific data and so a statistical technique such as this is a promising solution for feature detection. In fact, an interactive user interface can be used where the users can move a cube object freely inside the data and put it in a region where they are interested. Next, a distribution representation (currently Gaussian distribution is used, but any other distribution model can be used) is created from the data points within that selected cube region and is used as the target feature descriptor that we want to find in the data. For our current target application, the algorithm takes a particle field as input and first transforms it into a regular grid particle density field, which is a scalar field and is used as an intermediate representation. Then the density field is passed through a 3D super voxel generating algorithm, called Simple Linear Iterative Clustering (SLIC) [2], that produces super voxels from the particle density field. Now, a (Gaussian) distribution is modeled for each super voxel using the density values from respective super voxels. Finally, we use a distribution similarity measure to compute the statistical similarity between each super voxel distribution and the user-provided target feature distribution. Using these distribution similarity values for each super voxel, the algorithm finally generates a new feature similarity scalar field where each data point in the regular grid indicates its similarity with the user-given target feature. This feature similarity field is stored into disks at each time step for further detailed interactive post hoc analysis. A detailed version of this algorithm can be found in [1].


This statistical feature detection algorithm is made available as a VTK-based  ParaView/Catalyst filter for the user community. We have also developed a VTKm-based filter of this algorithm, which will be made available soon and the VTKm version of the algorithm will be using accelerators to improve computational performance.


Getting Started
^^^^^^^^^^^^^^^

The statistical distribution-based in situ feature detection algorithm is available as a VTK filter and is deployed in situ through ParaView/Catalyst pipeline. The name of the VTK filter is "vtkPFeatureAnalysis". This filter generates a histogram-based density field from the particle data using distributed data communication. Then SLIC super voxel algorithm is applied to generate the super voxels (clusters) from the particle density field and finally, the statistical similarity field is computed and stored into the disks for post hoc analysis. The output field can be readily visualized in ParaView/VisIT.

This filter expects two input parameters from the users and can be specified in the Catalyst python script:

#. Size of the 3D super voxel/clusters ("ClusterBlockSize") - The expected X, Y, Z size of the super voxels to be generated.
#. Parameters for user specified target feature distribution ("FeatureGaussian") - Currently takes mean and standard deviation values for the target feature distribution.

Currently, the output dimensions of the feature similarity scalar field are set to 128X16X128. In our upcoming version, we plan to make it an input parameter.


Example Result - In situ bubble detection in MFiX-Exa simulation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The following image shows a representative result of this algorithm when applied on a time step of MFiX-Exa particle field. The step-by-step in situ processing of the algorithm is shown. The final bubble similarity field is stored in the disks for post hoc bubble dynamics analysis.

.. image:: FeatureDetect/algorithm_concept.png
      :align: center
      :width: 95%

Use Case Example
^^^^^^^^^^^^^^^^
To demonstrate the use of statistical distribution-based feature detection VTK filter, here is an example Python-based Catalyst script file:

.. code-block:: python

  #--------------------------------------------------------------

  # Global timestep output options
  timeStepToStartOutputAt=0
  forceOutputAtFirstCall=False

  # Global screenshot output options
  imageFileNamePadding=0
  rescale_lookuptable=False

  # Whether or not to request specific arrays from the adaptor.
  requestSpecificArrays=False

  # a root directory under which all Catalyst output goes
  rootDirectory=''

  # makes a cinema D index table
  make_cinema_table=False

  #--------------------------------------------------------------
  # Code generated from cpstate.py to create the CoProcessor.
  # paraview version 5.8.1-1-g40b41376db
  #--------------------------------------------------------------

  from paraview.simple import *
  from paraview import coprocessing

  # ----------------------- CoProcessor definition -----------------------

  def CreateCoProcessor():
    def _CreatePipeline(coprocessor, datadescription):
      class Pipeline:
        # state file generated using paraview version 5.8.1-1-g40b41376db

        # ----------------------------------------------------------------
        # setup views used in the visualization
        # ----------------------------------------------------------------

        # trace generated using paraview version 5.8.1-1-g40b41376db

        #### disable automatic camera reset on 'Show'
        paraview.simple._DisableFirstRenderCameraReset()

        # get the material library
        materialLibrary1 = GetMaterialLibrary()

        # Create a new 'Render View'
        renderView1 = CreateView('RenderView')
        renderView1.ViewSize = [716, 352]
        renderView1.AxesGrid = 'GridAxes3DActor'
        renderView1.CenterOfRotation = [0.0020009127283896633, 0.001999172356439815, 0.0019999934867559444]
        renderView1.StereoType = 'Crystal Eyes'
        renderView1.CameraPosition = [0.036644586455975216, 0.21705065997450937, 0.0662849960547136]
        renderView1.CameraFocalPoint = [0.054171492461448235, -0.2147265944903157, 0.06031747111213411]
        renderView1.CameraViewUp = [0.9991579942858331, 0.04046464155513918, 0.0067760784031188894]
        renderView1.CameraParallelScale = 0.11422577498907434      
        renderView1.BackEnd = 'OSPRay raycaster'
        renderView1.OSPRayMaterialLibrary = materialLibrary1

        # register the view with coprocessor
        # and provide it with information such as the filename to use,
        # how frequently to write the images, etc.
        coprocessor.RegisterView(renderView1,
            filename='RenderView1_%t.png', freq=1, fittoscreen=0, 
            magnification=1, width=716, height=352, cinema={}, compression=5)
        renderView1.ViewTime = datadescription.GetTime()

        SetActiveView(None)

        # ----------------------------------------------------------------
        # setup view layouts
        # ----------------------------------------------------------------

        # create new layout object 'Layout #1'
        layout1 = CreateLayout(name='Layout #1')
        layout1.AssignView(0, renderView1)

        # ----------------------------------------------------------------
        # restore active view
        SetActiveView(renderView1)
        # ----------------------------------------------------------------

        # ----------------------------------------------------------------
        # setup the data processing pipelines
        # ----------------------------------------------------------------

        # create a new 'AMReX/BoxLib Particles Reader'
        # create a producer from a simulation input
        dEM07_plt00050 = coprocessor.CreateProducer(datadescription, 'inputparticles')

        # create a new 'Feature Analysis' filter
        featureAnalysis1 = FeatureAnalysis(Input=dEM07_plt00050)
        featureAnalysis1.ClusterBlockSize = [3, 3, 3]
        featureAnalysis1.FeatureGaussian = [2.0, 10.0]

        # create a new 'Resample To Image'
        resampleToImage1 = ResampleToImage(Input=featureAnalysis1)
        resampleToImage1.SamplingBounds = [1.9417382859558333e-06, 0.003984164024624505, 
        1.748457857490837e-08, 0.003865203601139793, 2.523888219899246e-06, 0.003966030690060129]

        # ----------------------------------------------------------------
        # setup the visualization in view 'renderView1'
        # ----------------------------------------------------------------

        # show data from resampleToImage1
        resampleToImage1Display = Show(resampleToImage1, renderView1, 'UniformGridRepresentation')

        # get color transfer function/color map for 'similarity'
        similarityLUT = GetColorTransferFunction('similarity')
        similarityLUT.ApplyPreset('Cold and Hot', True) ####
        similarityLUT.InvertTransferFunction() ####
        similarityLUT.ScalarRangeInitialized = 1.0

        # get opacity transfer function/opacity map for 'similarity'
        similarityPWF = GetOpacityTransferFunction('similarity')i
        similarityPWF.Points = [0.0, 0.0, 0.5, 0.0, 0.37078651785850525, 0.02139037474989891, 
        0.5, 0.0, 0.7387640476226807, 0.04812834411859512, 0.5, 0.0, 0.8904494643211365, 0.27272728085517883, 0.5, 0.0, 1.0, 1.0, 0.5, 0.0]
        ##similarityPWF.Points = [0.0, 1.0, 0.5, 0.0, 0.25070422887802124, 0.3375000059604645, 
        0.5, 0.0, 0.490140825510025, 0.0, 0.5, 0.0, 1.0, 0.0, 0.5, 0.0]
        similarityPWF.ScalarRangeInitialized = 1

        # trace defaults for the display properties.
        resampleToImage1Display.Representation = 'Volume'
        resampleToImage1Display.ColorArrayName = ['POINTS', 'similarity']
        resampleToImage1Display.LookupTable = similarityLUT
        resampleToImage1Display.OSPRayScaleArray = 'similarity'
        resampleToImage1Display.OSPRayScaleFunction = 'PiecewiseFunction'
        resampleToImage1Display.SelectOrientationVectors = 'None'
        resampleToImage1Display.ScaleFactor = 0.0003982222286338549
        resampleToImage1Display.SelectScaleArray = 'None'
        resampleToImage1Display.GlyphType = 'Arrow'
        resampleToImage1Display.GlyphTableIndexArray = 'None'
        resampleToImage1Display.GaussianRadius = 1.9911111431692744e-05
        resampleToImage1Display.SetScaleArray = ['POINTS', 'similarity']
        resampleToImage1Display.ScaleTransferFunction = 'PiecewiseFunction'
        resampleToImage1Display.OpacityArray = ['POINTS', 'similarity']
        resampleToImage1Display.OpacityTransferFunction = 'PiecewiseFunction'
        resampleToImage1Display.DataAxesGrid = 'GridAxesRepresentation'
        resampleToImage1Display.PolarAxes = 'PolarAxesRepresentation'
        resampleToImage1Display.ScalarOpacityUnitDistance = 6.888499664772514e-05
        resampleToImage1Display.ScalarOpacityFunction = similarityPWF
        resampleToImage1Display.SliceFunction = 'Plane'
        resampleToImage1Display.Slice = 49

        # init the 'PiecewiseFunction' selected for 'ScaleTransferFunction'
        resampleToImage1Display.ScaleTransferFunction.Points = [0.0, 0.0, 0.5, 0.0, 0.9267358779907227, 1.0, 0.5, 0.0]

        # init the 'PiecewiseFunction' selected for 'OpacityTransferFunction'
        resampleToImage1Display.OpacityTransferFunction.Points = [0.0, 0.0, 0.5, 0.0, 0.9267358779907227, 1.0, 0.5, 0.0]

        # init the 'Plane' selected for 'SliceFunction'
        resampleToImage1Display.SliceFunction.Origin = [0.0019930528814552304, 0.0019326105428591842, 0.0019842772891400145]

        # setup the color legend parameters for each legend in this view

        # get color legend/bar for similarityLUT in view renderView1
        similarityLUTColorBar = GetScalarBar(similarityLUT, renderView1)
        similarityLUTColorBar.Title = 'similarity'
        similarityLUTColorBar.ComponentTitle = ''
        similarityLUTColorBar.Orientation = 'Vertical' ##
        similarityLUTColorBar.Position = [0.862998295615945, 0.13773147335584612] ##
        similarityLUTColorBar.ScalarBarLength = 0.33000000000000007 ##

        # set color bar visibility
        similarityLUTColorBar.Visibility = 1

        # show color legend
        resampleToImage1Display.SetScalarBarVisibility(renderView1, True)

        # ----------------------------------------------------------------
        # setup color maps and opacity mapes used in the visualization
        # note: the Get..() functions create a new object, if needed
        # ----------------------------------------------------------------

        # ----------------------------------------------------------------
        # finally, restore active source
        SetActiveSource(resampleToImage1)
        # ----------------------------------------------------------------

        # Now any catalyst writers
        xMLPImageDataWriter1 = servermanager.writers.XMLPImageDataWriter(Input=resampleToImage1)
        coprocessor.RegisterWriter(xMLPImageDataWriter1, filename='ResampleToImage1_%t.pvti', 
        freq=1, paddingamount=0, DataMode='Appended', HeaderType='UInt64', 
        EncodeAppendedData=False, CompressorType='None', CompressionLevel='6')

      return Pipeline()

    class CoProcessor(coprocessing.CoProcessor):
      def CreatePipeline(self, datadescription):
        self.Pipeline = _CreatePipeline(self, datadescription)

    coprocessor = CoProcessor()
    # these are the frequencies at which the coprocessor updates.
    freqs = {'inputparticles': [1]}
    coprocessor.SetUpdateFrequencies(freqs)
    if requestSpecificArrays:
      arrays = [['cpu', 0], ['density', 0], ['dragx', 0], ['dragy', 0], 
      ['dragz', 0], ['id', 0], ['mass', 0], ['omegax', 0], ['omegay', 0], 
      ['omegaz', 0], ['omoi', 0], ['phase', 0], ['radius', 0], 
      ['state', 0], ['velx', 0], ['vely', 0], ['velz', 0], ['volume', 0]]
      coprocessor.SetRequestedArrays('inputparticles', arrays)
    coprocessor.SetInitialOutputOptions(timeStepToStartOutputAt,forceOutputAtFirstCall)

    if rootDirectory:
        coprocessor.SetRootDirectory(rootDirectory)

    if make_cinema_table:
        coprocessor.EnableCinemaDTable()

    return coprocessor


  #--------------------------------------------------------------
  # Global variable that will hold the pipeline for each timestep
  # Creating the CoProcessor object, doesn't actually create the ParaView pipeline.
  # It will be automatically setup when coprocessor.UpdateProducers() is called the
  # first time.
  coprocessor = CreateCoProcessor()

  #--------------------------------------------------------------
  # Enable Live-Visualizaton with ParaView and the update frequency
  coprocessor.EnableLiveVisualization(False, 1)

  # ---------------------- Data Selection method ----------------------

  def RequestDataDescription(datadescription):
      "Callback to populate the request for current timestep"
      global coprocessor

      # setup requests for all inputs based on the requirements of the
      # pipeline.
      coprocessor.LoadRequestedData(datadescription)

  # ------------------------ Processing method ------------------------

  def DoCoProcessing(datadescription):
      "Callback to do co-processing for current timestep"
      global coprocessor

      # Update the coprocessor by providing it the newly generated simulation data.
      # If the pipeline hasn't been setup yet, this will setup the pipeline.
      coprocessor.UpdateProducers(datadescription)

      # Write output data, if appropriate.
      coprocessor.WriteData(datadescription);

      # Write image capture (Last arg: rescale lookup table), if appropriate.
      coprocessor.WriteImages(datadescription, rescale_lookuptable=rescale_lookuptable,
          image_quality=0, padding_amount=imageFileNamePadding)

      # Live Visualization, if enabled.
      coprocessor.DoLiveVisualization(datadescription, "localhost", 22222)


Performance
^^^^^^^^^^^
The VTK filter currently runs on CPUs in the distributed environment. The integration pipeline has been tested at OLCF Summit. We will soon make the VTKm-based filter of this algorithm available for the users, which will be using accelerators and will further improve performance over the current VTK filter.

Developers
^^^^^^^^^^
Soumya Dutta (sdutta@lanl.gov), Li-Ta Lo, Berk Geveci, Dan Lipsa, and Patrick O'Leary.  

References
^^^^^^^^^^^
#. Soumya Dutta, Jonathan Woodring, Han-Wei Shen, Jen-Ping Chen, and James Ahrens, "Homogeneity Guided Probabilistic Data Summaries for Analysis and Visualization of Large-Scale Data Sets", 2017 IEEE Pacific Visualization Symposium (PacificVis), Seoul, Korea (South), 2017, pp. 111-120.

#. Radhakrishna Achanta, Appu Shaji, Kevin Smith, Aurelien Lucchi, Pascal Fua, and Sabine Süsstrunk, "SLIC Superpixels Compared to State-of-the-art Superpixel Methods", in IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 34, no. 11, pp. 2274-2282, Nov. 2012, doi: 10.1109/TPAMI.2012.120.


