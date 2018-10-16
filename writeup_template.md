# Self-Driving Car Engineer Nanodegree


## Project: **Finding Lane Lines on the Road** 
***
This is a CV based lane-marker detection and tracking implementation using standard OpenCV packages and image processing functions. 

The entire pipeline is structured as follows and is implemented after some experimentation on robust color spaces to detect lane-marker lines under various occlusions (such as shadows). 

---
>**The Lane MarkerDetector Pipeline**
>1. *RoI Mask* -- limits further processing toi the expected lane line regions in captured image
>2. *Color Detect* -- identify white/yellow lane markers using HLS color-space transform
>3. *Gray Scale* -- translate to single channel gray-scale frame
>4. *Gaussian Blur* -- unsharp mask using Gaussian kernel before edge detection
>5. *Edge Detect* -- Canny edge detection
>6. *Hough Line Transform* -- lane line stitching & detection using Hough transform
>7. *L/R Detect* -- Identify left & right lane markers using slope thresholds
>8. *Line Average* -- Average detected lines to smooth single frame detection
>9. *Lane Space Detect* -- lane and drivable space identification using OpenCV functions

---

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

### Test & Evaluations

#### Color Space Selection

We experimented with some online database images of road markings before settling on a dual-threshold color mask in the HLS space. 
Data sets of road markings ([ROMA (ROad MArkings) image database](http://perso.lcpc.fr/tarel.jean-philippe/bdd/index.html)) used for this evaluation. 
The lane markings are robustly detceted by using both the L & S components from the transformed color space with appropriate thresholds for the luminance and saturation values. 

---

### Reflection

### 1. Pipeline Implementation

The staged CV based pipeline is structured to reduce workload on teh system allowing a more balanced trade-off between pixel processing and system utilization. 
[Original Frame]: ./Resources/orig_solidYellowCurve.jpg "Original Frame"

As a result, the very first step is the pipeline is to reduce the valid region-of-interest over which all succesive pixel processing is performed. This is implemented via the `ROI Mask` function which is tuned for the specific camera topology and car hood geometry.
[RoI Mask]: ./Resources/roi_solidYellowCurve.jpg "RoI Mask"

The following 2 stages, viz. `Color Detect` and `Gray Scale` isolate the right components of the RoI pixels that give the most relevant & robust information on the lane markers udner various environmental conditions. The yellow & whiote thresholds selected for the HSL color space are tuned to pick off the standard colored lane markers on US roads udner a broad range of lighting & occlusion conditions. 
[Color Detect]: ./Resources/whiteyellow_solidYellowCurve.jpg "Color Detect"
[Grayscale]: ./Resources/gray_solidYellowCurve.jpg "Gray Scale"

The next 3 stages correspond to the line detector functon that uses an unsahrp mask (`Gaussian Blur`) before Canny edge filtering (`Edge Detect`) that feed the Hough line transform (`Hough Line Transform`). Specifically for the Hough line detector, the gris resolution and the thresholds for minimum length & maximum line separation required some fair amount of manual tuning to give acceptable results on thetest images & videos. The settings have been hard-coded into the code to simplify the call structure and overall clarity of the pipeline implementation.   
[Gaussian Blur]: ./Resources/blur_solidYellowCurve.jpg "Gaussian Blur"
[Canny Edge]: ./Resources/edge_solidYellowCurve.jpg "Canny Edge"
[Hough Lines]: ./Resources/houghed_solidYellowCurve.jpg "Hough Lines"

The lane marker orientation and left/right detector `L\R Detect` allows separation of the markers based on a simple slope threshold check. This adds value by providing some meta information to handle left & right lane tracking differently as a potential future implementation. 
[Left/Right Lanes]: ./Resources/leftright_solidYellowCurve.jpg "Left/Right Lanes"

The left and right lane lines so detected are then averaged (`Line Average`) using slope-intercept values to average each detected line. 
This is immediately followed by overlaying the detected averaged left and right lane markers (`Lane Space Detect`) on the captured camera frame while identifying the drivable space as the region encompassed withing the detected lanes.  
[Drivable Space]: ./Resources/drivable_solidYellowCurve.jpg "Drivable Space"

### 2. Improving the Pipeline

The current pipeline emphasizes simplicity and speed versus providing too many tunign knobs to fine tune detector/tracker. A simple API can encapsulate this implementation and provide a parameter structure for users to tune setup for car geometries, camera setup and other environmental conditions. 

The CV pipeline itself relies on a rather naive assumption on appearance of both lines to track. The algorithm can be made more robust to only detect available lines and estimate drivable space. Some examples of road images from the ROMA database show rural roads with partially drawn or all-together mission road lines. 

A complete solution should be robust to driving conditions (includign poor visibility, low light, extreme glare on concrete surfaces etc.) as well as roads with no lanes (e.g. provide warning rather than drawing crazy lines :)). 

A few points that can help improve the robustness and stability of the detected lane lines:

1. Temporal information from successive video frames can be used to either restrict searching around last detected line or else applying a window averaging filter to reduce the frame-to-frame flicker of detected lane lines.   
2. A weighted averaging of lines based on the counts and/or lengths from the Hough transform can put more emphasis on strongly detected lines which would play a bigger role in identifying the dominant lane line.  