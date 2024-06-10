# MPI-Sintel 数据集简介

## DIRECTORY STRUCTURE

===================

This section describes the structure of the full MPI-Sintel dataset. Please note
that, depending on your choice of download, you might have only a subset of the
data. All data can be downloaded from [Link](http://sintel.is.tue.mpg.de).

1. training/

   Training set.

2. training/albedo/

    Albedo pass for all sequences in the training set.

3. training/clean/

   Clean pass for all sequences in the training set.

4. training/final/

    Final pass for all sequences in the training set.

5. training/flow/

    Ground truth optical flow for all sequences in the training set, encoded
    as .flo files (see description below). Given is the forward flow.

6. training/flow_viz/

    Visualization of Optical Flow. For color coding, see Baker et al., "A
    Database and Evaluation Methodology for Optical Flow", IJCV 2011.

7. training/occlusion/

    Occluded regions for all sequences in the training set. Given as intensity images, with pixel values = 0 denoting non-occluded regions.

8. training/invalid/

    Invalid pixel maps, given as intensity images.

9. test/

    Testing set. For this set, only the Clean and Final passes are given. All
    further data for the training set is withheld for evaluation purposes.

10. test/clean/

    Clean pass for all sequences in the testing set.

11. test/final/

    Final pass for all sequences in the testing set.

12. bundler/
    The Bundler application as binaries for Linux, Windows, and OSX.

13. flow_code/

    Example C and MATLAB code for reading/writing .flo files.

## DATA FORMAT

===========

1. Albedo pass, Clean pass, Final pass, flow visualization:

    Given as 8-bit RGB PNG images.

2. Occlusions, invalid pixels:

    Given as binary intensity images, with a value of 1 denoting occluded regions and invalid pixels, respectively.

3. Ground truth optical flow:

    Binary ".flo" file format, storing a 2-channel float image for horizontal (u) and vertical (v) flow components. Floats are stored in little-endian order.

    | offset | size | contents |
    | --- | --- | --- |
    | 0 | 4 | Tag: "PIEH" in ASCII, which in little endian happens to be the float 202021.25 (just a sanity check that floats are represented correctly)|
    | 4 | 4 | Width as an integer |
    | 8 | 4 | Height as an integer |
    | 12 | width\*height\*2\*4 | Data: float values for u and v, interleaved, in row order, i.e., u[row0,col0], v[row0,col0], u[row0,col1], v[row0,col1], ... |

    Example code (C and MATLAB) to read/write .flo files is included in the subdirectory flow_code/. The C code was written by Daniel Scharstein, the MATLAB code was written by Deqing Sun.

## DATA METRICS

===========

1. d10: <=10 pixels from an occulusion boundary
2. s10: speed <= 10ppf
