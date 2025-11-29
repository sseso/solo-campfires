# Campfire Detection Pipeline for Solar Orbiter EUI-hrieuv174 Data

**This pipeline detects and characterizes small-scale transient brightenings, called "campfires", in Solar Orbiter EUI data.**

![](https://github.com/sseso/solo-campfires/blob/main/results/report/showcase3_small.gif)

## Context 

Data from Solar Orbiter's perihelion campaigns offers the highest spatial resolution of the sun's surface to date. As a result, previously unknown, small transient brightenings in the corona (typical areas range from 0.5 Mm^2 to 10 Mm$^2$), called "campfires", were discovered in Solar Orbiter's first datasets from 2020. To this day, many statistical properties of these events remain unclear, however, campfires are believed to play a significant role in the coronal heating problem, which is why studying them is of highest interest.

## Goal 

The goal of this project was to gain experience with tools used in solar physics and image analysis by creating a simple detection pipeline for campfires in Solar Orbiter Extreme UV Imager Data (HRI-EUV-174 Å), high quality image and video exports showcasing the detections, as well as simple approaches for computing physical properties (lifetime, area, intensity, etc.).

## Methodology

The detections pipeline uses a multi-stage process, combining thresholding, connected-component labeling, morphological filtering, and DBSCAN-based spatiotemporal clustering (as precursor to more sophisticated Machine Learning-based feature detection, work in progress). Visualizations were implemented with FFMPEG encoding for video exports, as well as matplotlib/seaborn for plots and image exports. 

## Results

Along with the preview .gif attached above, here's an image directly comparing an original frame with a frame + bounding boxes over the detections.
![](https://github.com/sseso/solo-campfires/blob/main/results/report/det_vs_orig.png)

The pipeline also produced realistic datasets, here's some sample plots.
![](https://github.com/sseso/solo-campfires/blob/main/results/report/Intensity_vs_Area_and_Lifetime.png)

![](https://github.com/sseso/solo-campfires/blob/main/results/report/lifetime_dist_20200530.png)

## Discussion

A critical evaluation of the pipeline confirms its core strengths: It reliably detects visually sensible events and filters out noise effectively (see fig. Detections) at rates (448–1294 per 50-frame sequence) that align well with published benchmarks (Berghmans et al. 2021). Plotting Intensity vs. area shows a positive correlation (Spearman's correlation coefficient of $\rho = 0.657$), similar to positive trends in area-lifetime correlations ($\rho$=0.71; Narang et al. 2025). The brightest and biggest events tending to living longer (as shown by the colormap), which hints at a positive relationship between area/intensity and lifetime as well. The detections also capture a power-law-like or log-normal trend in lifetime distribution, as observed in literature (Berghmans et al. 2021). However, the histogram is a bit front-heavy with a thinned out tail, which hints at fragmentation in lifetime detections, causing more shorter events. This is confirmed through visual inspection of the exported videos with bounding box overlays, where occasional gap frames become evident, explaining the bias towards shorter/truncated lifetime detections. Another crude assumption stems from the fact that equal-width binning in the lifetime histogram slightly distorts the short-tail dominance, though the overall shape (steep drop-off with long tail) holds true. A similar power-law like distribution is cited in many papers for other properties such as area, however, plotting a histogram for the areas of detected events currently shows a very long tail, which signals that some detected areas are overestimated. This could be caused by the small-angle pixel-scale approximation used, which disregards solar curvature, leading to increasing errors for detections at the edge of a frame and overestimation of their area.

These insights point to targeted upgrades, including:

- correct for solar curvature using heliocentric angle (angle between the local surface normal and the line of sight spacecraft)
- Computer vision based shape classifiers to refine area calculations and reject artifacts;
- ML-driven lifetime tracking for gap free detections;
- add spatial tracking to detect moving events;
- Rigorous statistics tools like Maximum Likelihood Estimation for power-law fits and Kolmogorov-Smirnov tests for literature comparisons;
- Scaling to finer-cadence, multi-sequence datasets available in the public SOAR-archive for truncation-free analysis and higher time resolution.

This project taught me python tools used in solar physics such as sunpy and astropy, FITS data handling and data/image analysis, gave me insight into ongoing research on solar campfires, and made me realize the challenges associated with analyzing and interpreting real physical data, such as handling noise and correct drawing of conclusions from datasets. With my current results and literature comparison, I compiled several actionable suggestions for improvement, which I will work on implementing in the future.

## References

- Narang, N., Verbeeck, C., Mierla, M., Berghmans, D., Auchère, F., Shestov, S., Delouille, V., Chitta, L. P., Priest, E., Lim, D., Dolla, L. R., & Kraaikamp, E. (2025). Extreme-ultraviolet transient brightenings in the quiet-Sun corona: Closest-perihelion observations with Solar Orbiter/EUI. *Astronomy & Astrophysics*, 699, A138. https://doi.org/10.1051/0004-6361/202554650
- Berghmans, D., Auchère, F., Long, D. M., Soubríe, E., Mierla, M., Zhukov, A. N., Schühle, U., Antolin, P., Harra, L., Parenti, S., Podladchikova, O., Aznar Cuadrado, R., Buchlin, E., Dolla, L., Verbeeck, C., Gissot, S., Teriaca, L., Haberreiter, M., Katsiyannis, A. C., Rodriguez, L., Kraaikamp, E., Smith, P. J., Stegen, K., Rochus, P., Halain, J. P., Jacques, L., Thompson, W. T., & Inhester, B. (2021). Extreme UV quiet Sun brightenings observed by Solar Orbiter/EUI. *Astronomy & Astrophysics*, 656, L4. https://doi.org/10.1051/0004-6361/202140380
