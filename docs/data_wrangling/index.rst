Handling neuroimaging data for graph theory analysis
====================================================

Most graph theoretic analyses of fMRI data use either ROI-to-ROI or voxel-to-voxel connectivity.
In ROI-to-ROI connectivity analyses, time series from different regions of interest across the brain are extracted and some measure of association (e.g., correlation) is calculated between each pair of ROIs in order to produce an association matrix. In voxel-to-voxel connectivity analysis, the time series from individual voxels are extracted and correlated, resulting in a much larger association matrix. While Pearson correlation is generally used, other association metrics may be substituted, including partial correlation.

With the use of ``nibabel`` and ``nilearn``, it is very easy to extract these time series and to produce the association matrix you want to analyze. In fact, nilearn has a great user guide that goes over

Reading in and extracting fMRI data
-----------------------------------

Nilearn contains many functions of resampling and masking data. After extracting time series from a series of ROIs, we can just use ``numpy`` to perform ROI-to-ROI correlations in order to generate an association matrix.

  >>> import numpy as np
  >>> from nilearn import datasets
  >>> from nilearn.image import resample_to_img
  >>> from nilearn.input_data import NiftiLabelsMasker
  >>>
  >>> # Load an atlas
  >>> dataset = datasets.fetch_atlas_harvard_oxford('cort-maxprob-thr25-2mm')
  >>> atlas_filename = dataset.maps
  >>> labels = dataset.labels
  >>> masker = NiftiLabelsMasker(labels_img=atlas_filename, standardize=True)
  >>>
  >>> # Download and resample some data
  >>> subjects = datasets.fetch_adhd(n_subjects=1)
  >>> func_img = resample_to_img(subjects.func[0], atlas_filename)
  >>> time_series = masker.fit_transform(func_img, confounds=subjects.confounds[0])
  >>> corr_mat = np.corrcoef(time_series.T)

Preparing association matrices for analysis
-------------------------------------------

Association matrices require some additional processing before we can calculate graph theory metrics on them. For standard correlation matrices, the diagonal is going to be 1s (i.e., each ROI's time series is correlated with itself). However, for most graph theoretic analyses, we aren't interested in self-connectivity, so we zero out the diagonal as part of the transformation from association matrix to *adjacency matrix*.

Additionally, it is generally assumed that most low values in the matrix reflect noise rather than signal, so it is common to threshold the adjacency matrix. There are two common thresholding methods: absolute and proportional. Absolute thresholding selects a threshold to apply to all matrices being analyzed. Proportional thresholding uses the zeros out the X% lowest values in each matrix, so that all of the matrices have the same number of edges preserved.
