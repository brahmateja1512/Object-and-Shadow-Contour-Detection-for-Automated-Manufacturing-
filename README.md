# Object and Shadow Contour Detection for Automated Manufacturing

This repository provides tools and notes to detect object and shadow contours from camera images and export the contours as DXF files for downstream CAD/CAM or cutting workflows.

Summary
- Input: photo from a fixed camera (object + shadow).
- Output: one or more closed polylines saved in a DXF file.
- Approach (example script): preprocess → edge/threshold → find contours → filter & simplify → export DXF.

Prerequisites
- Python 3.8+
- Basic command-line usage knowledge
- Recommended packages (see requirements.txt):
  - opencv-python
  - numpy
  - ezdxf
  - matplotlib (optional, for visualization)

Quick Install
1. Clone the repository:
   git clone https://github.com/brahmateja1512/Object-and-Shadow-Contour-Detection-for-Automated-Manufacturing-.git
   cd Object-and-Shadow-Contour-Detection-for-Automated-Manufacturing-
2. Create and activate a virtual environment:
   python -m venv venv
   source venv/bin/activate   # macOS / Linux
   venv\Scripts\activate      # Windows
3. Install dependencies:
   pip install -r requirements.txt

Step-by-step procedure to run contour extraction and produce a DXF
1. Prepare images
   - Use a fixed camera position and consistent lighting for repeatability.
   - Prefer a plain background and place the object so its contour is distinct from the background.
   - Save images into an `images/` folder (create if missing).

2. Basic run (example)
   - Example command:
     python main.py --input images/sample.jpg --output out/contours.dxf
   - Expected result: `out/contours.dxf` containing one or more LWPolylines representing detected contours.

3. Inspect intermediate output (recommended when tuning)
   - Visualize the preprocessed image (grayscale/blurred), edges or binary threshold, and detected contours.
   - You can add temporary cv2.imshow(...) or cv2.imwrite(...) calls to `main.py` to save these images.
   - Key images to inspect:
     - preprocessed gray/blurred image
     - canny edge map or binary threshold image
     - contour overlay on original image

4. Tune parameters (common flags)
   - --method [canny|threshold]
     - canny: good when object edges are high-contrast
     - threshold: better for uniform lighting or when you prefer global segmentation
   - --min-area <pixels>
     - Filters out small spurious contours. Set relative to image resolution.
   - --approx-epsilon <fraction>
     - Controls how much a contour is simplified (smaller → more points).
   - --blur <odd integer>
     - Smoothing before edge/threshold; helps remove noise but blurs small features.

5. Convert pixel contours to real-world units (optional)
   - If you need DXF in millimeters or another unit, determine pixels→mm scale:
     - Place a reference object of known size in the scene and measure its pixel length.
     - Scale contour coordinates before exporting (multiply x,y by mm_per_pixel).
   - For precise manufacturing, perform camera calibration to remove lens distortion.

Common problems and suggested fixes
- "Could not read input image" or blank image
  - Check the input path and filename; ensure correct relative vs absolute path.
  - Confirm the image file is not corrupted and OpenCV can read its format.
  - Permission error when saving: ensure output directory exists or you have write permission.

- No contours detected / "No contours detected with the current parameters."
  - Try switching methods: --method threshold vs canny.
  - Lower the --min-area value to allow smaller contours.
  - Adjust canny thresholds (if using canny) — lower the low threshold for more edges.
  - Improve lighting or change the background to increase contrast.
  - Try a larger or smaller blur to reduce noise or preserve edges.
  - If shadows and object merge, consider taking a second image with different illumination or using morphological ops to separate.

- Too many small noisy contours
  - Increase --min-area so tiny blobs are discarded.
  - Use morphological opening (erosion followed by dilation) on the binary image.
  - Increase blur to smooth high-frequency noise.

- DXF appears scaled incorrectly in CAD software
  - Confirm whether your CAD program expects units; DXF stores coordinates as raw numbers.
  - Apply pixel-to-unit scaling before export (recommended).
  - Make sure your CAD imports the correct units (mm vs inches).

- Import/installation errors: "No module named cv2" or "No module named ezdxf"
  - Run: pip install -r requirements.txt
  - For headless servers (no display), consider installing opencv-python-headless instead of opencv-python.

- ezdxf save errors or corrupted DXF
  - Ensure you create a valid modelspace and add polylines as closed lists.
  - Verify coordinates are finite numbers and not NaN/Inf.
  - Test saving a minimal DXF with ezdxf to verify installation.

Tips for better detection
- Use uniform backlighting or place a light source so the shadow is consistent if you need to detect both object and shadow separately.
- If you need to separate object contour from shadow contour:
  - Capture two images with different illumination (one strong overhead to minimize shadow, one with oblique lighting to highlight shadow), then subtract/compare results.
- Use morphological operations to close small gaps in contours.
- For repeatable production, fix the camera and create a calibration step to compute physical units and correct distortion.

Extending the pipeline
- Add image capture code for cameras (OpenCV VideoCapture) to run in production.
- Add a configuration file (JSON/YAML) to store tuned parameters per camera/environment.
- Add unit tests and example images with expected DXF outputs.
- Add a GUI or web interface for interactive parameter tuning and previewing results.

Reporting issues & contributing
- If you find a bug or need help tuning parameters, open an issue with:
  - a sample input image
  - command used and parameters
  - the output DXF (if produced) and any logs/errors
- Contributions are welcome (fork → feature branch → PR). See CONTRIBUTING.md.

License
- The project includes an MIT license. See LICENSE for details.

Contact / Notes
- The repository also contains `LAM shadow contour detection with dfx.txt` with algorithm notes — consult it for additional context and background.
- If you'd like, I can prepare a tuned configuration for your specific camera and sample images — provide a representative image and the desired output scale (mm/inches).
