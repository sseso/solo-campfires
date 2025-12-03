# Campfire Detection Pipeline for Solar Orbiter EUI-HRI-EUV-174 Data

![Detection Showcase .gif](https://raw.githubusercontent.com/sseso/solo-campfires/main/results/report/showcase3_small.gif)

## Overview
### Summary
- This project automatically detects and characterizes small-scale transient brightenings, called "campfires", in Solar Orbiter EUI data.
- Approach: Sigma Thresholding, first connected-component labeling --> Masking --> Spatial Merging of fragmented events --> Spatiotemporal Clustering with scikit-learn (DBSCAN) + morphological filtering
  --> Output/Visualization (Event catalog CSV, annotated still frames, annotated .mp4 movies, plots)
- early-stage, but already reproduced some key findings from recent literature (positive correlation between Intensity/Area/Lifetime, power-law-like distribution of lifetimes)
### Folder Structure
```
solo-campfires/
├── notebooks/
├── data/
│   ├── raw/
│   ├── processed/
├── results/
│   ├── csv/
│   ├── video/
├── requirements.in
├── requirements.txt
├── README.md
├── LICENSE
└── Project_Summary_SolO_Campfires.pdf
```

## Usage
### Prerequisites
Python 3.8 or higher (lower versions may cause issues with sunpy or astropy).
A working installation of Jupyter (via pip or conda).
Access to the Solar Orbiter Archive (SOAR) for downloading FITS data files—note that these can be large (hundreds of MB per dataset), so ensure you have sufficient storage and bandwidth. The project assumes L2-level HRI-EUV-174 Å data; raw or L1 data won't work without preprocessing.
While the pipeline is notebook-based and straightforward, solar data handling can be finicky due to FITS format specifics and coordinate systems. Test on a small subset first to avoid surprises.

### Data Fetching
The pipeline uses real Solar Orbiter data, so you'll have to download it manually from the public SOAR archive (https://soar.esac.esa.int/soar/#search). Select the instrument "Extreme UV-Imager (EUI)", select processing level "L2" and the file name "hrieuv174". You'll find the dataset used in this project by searching between 2020-05-30T00:00:00 to 2020-05-30T23:59:59. (you can use other datasets, of course). Save the .fits files in a folder like "data/raw/yourdataset" and you're good to go.

### Installation
Clone the repository: 
```bash
git clone https://github.com/sseso/solo-campfires.git
cd solo-campfires
```
Install the dependencies. This project uses requirements.in for loose specs and requirements.txt for pinned versions (generated via pip-compile). For reproducibility:
```bash
pip install pip-tools  # If not already installed
pip-compile requirements.in  # Updates requirements.txt if needed
pip install -r requirements.txt
```

If you're in a hurry or testing, just run 
```bash
pip install -r requirements.txt
```
directly. Always use a fresh virtual environment (e.g., venv or conda) to avoid conflicts, sunpy might clash with system astropy installs.
Launch Jupyter (if not already set up): 
```bash
pip install jupyter  # Only if missing
```
### Running the notebooks

The project is entirely notebook-based, so execution is interactive and modular. The uploaded notebooks already contain the intended structure for usage, but every function has an extensive docstring for easier usage. First run the detection pipeline:
- import packages
- for detection pipeline: load dataset
```python
dataset = "20200530"
sequence = load_dataset(dataset)
```
  
- run sigma threshold for baseline detections
```python
# example usage
sigma = 3
df: pd.DataFrame[CampfireData] = sigma_threshold(sequence, sigma, per_frame=True)
```
- apply masks and spatial merging for fragmented events
```python
masks = build_masks(sequence, bright_pct=99, dilation=2, min_area=150)
df_masked = apply_masks(df, masks)
visualize_mask(sequence, 25, masks)

merged_df = merge_detections(df_masked, radius = 5)
```
- spatiotemporal clustering and morphological filtering (area, lifetime)
```python
event_df, detections_df = build_event_catalog(merged_df,
    spatial_eps=3,
    temporal_eps_frames=3,     
    min_samples=1,
    min_lifetime_seconds=5,
    max_lifetime_seconds=500,
    cadence_seconds=5.0,          
    max_area_Mm2=15,
)
```
- preview results with still frames (showcase_detections()) or movies (make_movie())
```python
# still frame
frame = 25
showcase_detections(sequence, frame, df=detections, factor=10, vmin_pct=0.1, vmax_pct=99.9, dpi=250)

# or make a movie
make_movie(sequence, df=detections, file_name="detections_video", fps = 7, vmin_pct=0.5, vmax_pct = 99.5, show_detections = True, dpi = 200)
```
- save to .csv if happy with the results
```python
save_to_csv(event_df, "file_name")
```

Then the event statistics:
- import packages
- load .csv file
```python
# set up paths
csv_dir = Path("../results/csv")
dataset = "20200530"

event_df = pd.read_csv(csv_dir / "detections.csv")
```

- run pre-written plots for sanity check, add new plots as desired

Runtime: 1-2 Minutes for detection pipeline, about 30/sec per 50 frames for movie rendering (with a sensible amount of detections), <30 sec for event statistics (based on tests with a powerful desktop computer and an average ThinkPad). 

### Troubleshooting
FITS errors: Ensure sunpy version matches requirements.txt, mismatched headers can crash loading.
Adjust baseline detections with sigma_threshold (values vary for quiet-sun vs. very active region datasets).
Clustering fails: DBSCAN params are tunable, tweak eps and min_samples if noise dominates..
For issues, check notebook logs or raise a GitHub issue with your data sample and error traceback.

This setup should get you 90% there, but test it end to end before trusting results; solar image analysis is prone to artifacts like cosmic rays or projection distortions.
## Context 

Data from Solar Orbiter's perihelion campaigns offers the highest spatial resolution of the sun's surface to date. As a result, previously unknown, small transient brightenings in the corona (typical areas range from 0.5 Mm^2 to 10 Mm^2), called "campfires", were discovered in Solar Orbiter's first datasets from 2020. To this day, many statistical properties of these events remain unclear, however, campfires are believed to play a significant role in the coronal heating problem, which is why studying them is of highest interest.

## Goal 

The goal of this project was to gain experience with tools used in solar physics and image analysis by creating a simple detection pipeline for campfires in Solar Orbiter Extreme UV Imager Data (HRI-EUV-174 Å), high quality image and video exports showcasing the detections, as well as simple approaches for computing physical properties (lifetime, area, intensity, etc.).

## Methodology

The detections pipeline uses a multi-stage process, combining thresholding, connected-component labeling, morphological filtering, and DBSCAN-based spatiotemporal clustering (as precursor to more sophisticated Machine Learning-based feature detection, work in progress). Visualizations were implemented with FFMPEG encoding for video exports, as well as matplotlib/seaborn for plots and image exports. 

## Results

Along with the preview .gif attached above, here's an image directly comparing an original frame with a frame + bounding boxes over the detections, as well as some plots.
![Original Frame vs. Detections](https://github.com/sseso/solo-campfires/blob/main/results/report/det_vs_orig.png)

![Scatterplot Intensity vs Area with colormap](https://github.com/sseso/solo-campfires/blob/main/results/report/Intensity_vs_Area_and_Lifetime.png)

![Histogram Lifetime distribution](https://github.com/sseso/solo-campfires/blob/main/results/report/lifetime_dist_20200530.png)

## Discussion

A critical evaluation of the pipeline confirms its core strengths: It reliably detects visually sensible events and filters out noise effectively (see fig. Detections) at rates (448–1294 per 50-frame sequence) that align well with published benchmarks (1467 events, Berghmans et al. 2021). Plotting Intensity vs. area shows a positive correlation (see fig. Intensity_vs_Area, Spearman's correlation coefficient of $\rho = 0.699$, $p = 5.59\cdot10^{-63}$), with bigger and brighter events tending to live longer (as implied by the colormap), which suggests a positive relationship between area/intensity and lifetime as well, consistent with expected trends (Narang et al. 2025).

### Limitations
#### **Fragmented Lifetime detection, binned histogram**
The detections capture a power-law-like or log-normal-like trend in lifetime distribution (see fig. Lifetime_Distribution), as observed in literature (Berghmans et al. 2021). However, the histogram is a bit front-heavy with a thinned out tail, which hints at fragmentation in lifetime detections, causing more shorter events. This is confirmed through visual inspection of the exported videos with bounding box overlays, where occasional gap frames become evident, explaining the bias towards shorter/truncated lifetime detections.Another crude assumption stems from the fact that equal-width binning in the lifetime histogram slightly distorts the short-tail dominance, though the overall shape (steep drop-off with long tail) holds true.

#### **Area distortion through surface curvature**
A similar power-law like distribution is cited in many papers for other properties such as area, however, plotting a histogram for the areas of detected events currently shows a very long tail, which signals that some detected areas are overestimated. This could be caused by the small-angle pixel-scale approximation used, which disregards solar curvature, leading to increasing errors for detections at the edge of a frame and overestimation of their area.

#### **Spatial tracking**
The biggest opportunity for improvement lies in spatial tracking, as the current implementation cannot recognize moving events (e.g., mass ejections), which means the pipeline works well for quiet-sun datasets with little moving components, but detects too many false positives on very active regions.

### Suggested Upgrades
These insights point to targeted upgrades, including:
- correct for solar curvature using heliocentric angle (angle between the local surface normal and the line of sight spacecraft);
- Computer vision based shape classifiers to refine area calculations and reject artifacts;
- ML-driven lifetime tracking for gap free detections;
- add centroid tracking to detect moving events;
- Rigorous statistics tools like Maximum likelihood estimation for power-law fits and Kolmogorov-Smirnov tests for literature comparisons;
- Scaling to larger, finer-cadence datasets available in the public SOAR-archive for truncation-free analysis and higher time resolution.

### Personal Insight
This project taught me the basics of python tools used in solar physics, such as sunpy, astropy and the unsupervised machine-learning tool DBSCAN, git/version control, FITS data handling and data/image analysis, gave me insight into ongoing research on solar campfires, and made me realize the challenges associated with analyzing and interpreting real physical data, such as handling noise and correct drawing of statistical conclusions from real datasets. With my current results and literature comparison, I compiled several actionable suggestions for improvement, which I will work on implementing in the future.

## References

- Narang, N., Verbeeck, C., Mierla, M., Berghmans, D., Auchère, F., Shestov, S., Delouille, V., Chitta, L. P., Priest, E., Lim, D., Dolla, L. R., & Kraaikamp, E. (2025). Extreme-ultraviolet transient brightenings in the quiet-Sun corona: Closest-perihelion observations with Solar Orbiter/EUI. *Astronomy & Astrophysics*, 699, A138. https://doi.org/10.1051/0004-6361/202554650
- Berghmans, D., Auchère, F., Long, D. M., Soubríe, E., Mierla, M., Zhukov, A. N., Schühle, U., Antolin, P., Harra, L., Parenti, S., Podladchikova, O., Aznar Cuadrado, R., Buchlin, E., Dolla, L., Verbeeck, C., Gissot, S., Teriaca, L., Haberreiter, M., Katsiyannis, A. C., Rodriguez, L., Kraaikamp, E., Smith, P. J., Stegen, K., Rochus, P., Halain, J. P., Jacques, L., Thompson, W. T., & Inhester, B. (2021). Extreme UV quiet Sun brightenings observed by Solar Orbiter/EUI. *Astronomy & Astrophysics*, 656, L4. https://doi.org/10.1051/0004-6361/202140380
