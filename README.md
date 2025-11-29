# Campfire Detection Pipeline for Solar Orbiter EUI-hrieuv174 Data

**This pipeline detects and characterizes small-scale transient brightenings, called "campfires", in Solar Orbiter EUI data.**

![](https://github.com/sseso/solo-campfires/blob/main/results/report/showcase3_small.gif)

## Context 

Data from Solar Orbiter's perihelion campaigns offers the highest spatial resolution of the sun's surface to date. As a result, previously unknown, small transient brightenings in the corona (typical areas range from 0.5 Mm$^2$ to 10 Mm$^2$), called "campfires", were discovered in Solar Orbiter's first datasets from 2020. To this day, many statistical properties of these events remain unclear, however, campfires are believed to play a significant role in the coronal heating problem, which is why studying them is of highest interest.

## Goal 

The goal of this project was to gain experience with tools used in solar physics and image analysis by creating a simple detection pipeline for campfires in Solar Orbiter Extreme UV Imager Data (HRI$_{\text{EUV}}$-174 Å), high quality image and video exports showcasing the detections, as well as simple approaches for computing physical properties (lifetime, area, intensity, etc.).

## Methodology

The detections pipeline uses a multi-stage process, combining thresholding, connected-component labeling, morphological filtering, and DBSCAN-based spatiotemporal clustering (as precursor to more sophisticated Machine Learning-based feature detection, work in progress). Visualizations were implemented with FFMPEG encoding for video exports, as well as matplotlib/seaborn for plots and image exports. 

## Results

Along with the preview .gif attached above, the pipeline produced realistic datasets. Here's some sample plots.

![](https://github.com/sseso/solo-campfires/blob/main/results/report/Intensity_vs_Area_and_Lifetime.png)

![](https://github.com/sseso/solo-campfires/blob/main/results/report/lifetime_dist_20200530.png)
