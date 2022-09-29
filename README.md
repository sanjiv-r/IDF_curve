### Obtaining global Intensity-Duration-Frequency curves for rainfall using map_blocks() in xarray
----

This notebook shows how to leverage the powerful [map_blocks](https://docs.xarray.dev/en/stable/generated/xarray.map_blocks.html) function in [Xarray](https://docs.xarray.dev/en/stable/index.html) to calculate the Intensity-Duration-Frequency curves (IDF) for precipitation, by parallelizing calculations on a **global high-resolution** [**CAM-SE**](https://journals.sagepub.com/doi/10.1177/1094342011428142) dataset with an unstructured grid. The **atmospheric resolution** of this dataset is about **2.9-11.5 times** higher than that in the [CMIP6](https://www.wcrp-climate.org/wgcm-cmip/wgcm-cmip6) suite depending on the individual model while the corresponding **ocean resolution** is approximately **10 times finer**. The **IDF curves** plot the precipitation intensity (mm/hr) as a function of the duration for different return periods and are a critical tool in developing strategies for climate resilience. The present exercise is inspired by this set of [videos](https://www.youtube.com/watch?v=FItPMwK4K1o) showing how to compute the IDF curves at a single location by fitting **Generalized Extreme Value** distributions using `numpy` and `scipy`, but operates within an `Xarray` framework. The primary contribution of this notebook is the use of `map_blocks` as looping through the locations using a `for` loop is not a feasible option for a dataset of this size.

For purposes of illustration, the present analysis uses a **6-hourly timeseries** of precipitation from a     high-resolution **CESM pre-industrial "control" simulation**. The analysis covers a duration of **52 years** (1970-2021). Here is the  outline of the workflow:
1. The unstructured grid organizes two-dimensional variables in `(ncol,time)` format where `ncol` is a linear index that runs through all the `(lon,lat)`      values. 
2. The `ncol` dimension (~ 7.7 million) is chopped up into 7 roughly equal chunks.
3. First, the IDF curves at a single location (fixed `ncol` value) are confirmed to look reasonable.
4. We then create a function that generates the IDF curves and will be passed as an argument to `map_blocks.` The call to `map_blocks` will efficiently execute the function in parallel within each of the 7 chunks  (step 2). Nothing has been executed  yet.
5. We make 7 duplicates of the notebook in step 4 and run them simultaneously to cover the entire range of `ncol`, which spans the entire globe. At the end of step 5, we have 7 files which together contain the IDF curves for every point on the globe, covering all 7 chunks. (It is also possible to run `map_blocks` on the entire range of `ncol` without splitting beforehand in step 2 but I found the present strategy to be faster.) 
6. We combine the 7 chunks using `xr.concat` to generate a single file which has the IDF curves for the entire globe. The result is still using the linear `ncol`     index which corresponds to the unstructured grid and is not ideal for plotting.  
7. We regrid the data on the unstructured grid to a regular `lon/lat` grid using [NCL](https://www.ncl.ucar.edu/) tools to generate the final global plots.
