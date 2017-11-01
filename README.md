## Single-Cell Chip Image Analysis

## Repo Structure:

### [Typhoon Image Analysis](https://github.com/)
 Contains scripts for determining well fluorescence for each fluorophore

  - TyphoonCrop: Prepares raw images for analysis, including identifying and performing necessary rotation and crop
  - AnalyzeFluorescence: Analyzes the fluorescence per well of the images modified by TyphoonCrop

### [Olympus Imaging Analysis](https://github.com/)
Contains scripts for determining cell counts in each well from images taken via an Olympus scope with an automated stage

  - alignOlympusImages:  Prepares raw images for analysis
  - analyzeOlympus:  Analyze the number of cells per well

### [Raw Data](https://github.com/)
Contains an example data set used in our publication

### [Processed Data](https://github.com/)
Contains all processed data from an example analysis of this data set as described in our publication.

# Before running any analysis:
Clone repo from Github.

If this is your first time analyzing these data on this computer, do the following steps first:

1. Install ImageJ

    - Link:  [https://imagej.nih.gov/ij/download.html](https://imagej.nih.gov/ij/download.html)

2. Download the plugin "Linearize_GelData.class"

    - Link:  [https://imagej.nih.gov/ij/plugins/linearize-gel-data.html](https://imagej.nih.gov/ij/plugins/linearize-gel-data.html)

    - The Typhoon saves files in a .gel format, which is unique to the instrument type.  Typhoons register pixel intensity on a 100,000-value scale which cannot be easily compressed to traditional image filetypes that are 8-bit (256-value) or 16-bit (65536-value).  In order to save files in a .gel format, which are 16-bit images, the instrument uses a nonlinear (exponential) compression algorithm.  As a result, unless one is using General Electric’s custom Typhoon analysis software, all .gel files must be "linearized" and converted to .tifs in order to obtain interpretable data.  This is accomplished with ImageJ’s “Linearize Gel Data” plugin, which reverses the nonlinear compression algorithm and applies a linear scaling factor (the user can choose the value but we use the default).

3. Drag "Linearize_GelData.class" to the "plugins" subfolder of ImageJ

4. Close and re-open ImageJ


## Analysis Quick Start Protocol
1. Open ImageJ
2. In the navigation bar, go to: (Plugins -> Macros -> Run...), go to "TyphoonImageAnalysis" and select "TyphoonCrop.txt".  
3. In the navigation bar, go to: (Plugins -> Macros -> Run...), go to "OlympusImagingAnalysis" and select "alignOlympusImages.txt".
13. **Data Integration in R:**  DataReduction.Rmd
   - To use this R markdown file, you will need to make sure that the code is being evaluated in the console, NOT inline (click on the gear icon above the Rmd and choose "Chunk Output in Console" NOT "Chunk Output Inline").  



## ImageJ Analysis Details
### TyphoonCrop.txt
  - Rotates, crops and linearizes the .gel files (raw data).
  - Values for the specific scan to be analyzed will need to be set for the following variables in this macro due to placement of the arrays on the Typhoon scanner being different every time.  These variables will set the rotation angle required to orient the image for downstream processing (array 1, the first array filled always on the top, and inlets on the left), and the size of the crop area for the image stack.  
  - Run-specific Variables to be set:
    - rotationAngle
    - xStart
    - yStart
  - This will create a set of rotated and cropped images in a directory in ProcessedData called "TyphoonImages" and save metadata about the process in the Metadata directory.  

### AnalyzeFluorescence.txt
  - Thresholds the images based on the probes listed in "runInfo.txt".
  - Overlays a Region of Interest grid based on user input via dialog boxes.
  - Measures %Area and Integrated Density of each region of interest.
  - Saves fluorescence data and metadata.

  - Run-specific Variables to be set:
    - X, Y, Width, Height for each image's ROI Grid

  - Creates its output folder (TyphoonCSVs).
  - Creates a mask of droplet locations in each fluorophore channel using a probe specific, experimentally determined threshold.
  - Uses this mask to set all non-droplet pixels in the FAM, HEX and Cy5 images from a scan to zero, and leaves the droplet pixel intensities as is.
  - For each image:
    - Calls the ROIGrid function to define regions of interest (ROIs) over which to integrate.
    - Measures the droplet areas (%Fill) and ROI Integrated Density (IntDen).
    - Outputs a csv containing the ROI identifier and the variables measured to the TyphoonCSVs folder, with one csv generated per fluorophore.
  - Save metadata to the Metadata folder in ProcessedData.


### alignOlympusImages.txt
  - Calls the findRotation and findCrop functions on an image set.
  - Saves to results folder.
  - Creates its output folder (./ProcessedData/OlympusImages).

For each image in ./RawData/OlympusImages, the following processes occur:  

  - Calls the percentileThreshold function to account for exposure differences
  - Calls the findRotation function on the side of the image closer to the interior of the array
  - Determines a uniform rotation value for all images in that array and applies that value
  - Calls the findCrop function on the top, left and right sides of the image
  - Determines two uniform top boundaries, one each for images in the lower and upper halves of the array
  - Applies each image's left and right crops, as well as the consensus top crop for images in its half (upper/lower) of the array
  - Outputs a rotated and cropped image.  These images will be used by analyzeOlympus.txt
  
##### Olympus imaging order and numbering scheme per array:
	 1   2   3   4   5   6  
	12  11  10   9   8   7


### analyzeOlympus.txt
 - Calls the roiGrid and countCells functions on an image set.
 - Saves to results folder.
 - Creates its output folder (./ProcessedData/OlympusCSVs).

For each image in ./ProcessedData/OlympusImages, the following processes occur:

  - Calls the roiGrid function to define ROIs over which to analyze particles.
  - Calls the countCells function to analyze the particles in each well, then adjust the count based on the size and circularity of the particles, which denote a clump rather than a single cell
  - Outputs a CSV containing cell counts for that image.  These CSVs will be compiled by DataReduction.Rmd