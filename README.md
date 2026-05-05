# 🏃 Long Jump Athlete Tracking — Image Processing Pipeline


## 📌 Project Overview

This project builds a complete image processing pipeline to **detect and track a long jump athlete** across frames extracted from a sports video recording. The pipeline covers every step from raw video input through preprocessing, enhancement, segmentation, and CSRT-based tracking — producing a final annotated output video with bounding box overlays.

The sport of **long jump** was chosen because the athlete moves linearly down a runway, making it well-suited for tracking with fewer background distractions than team sports.

---

## 🎬 Video Source

| Property | Details |
|---|---|
| Sport | Long Jump |
| Video Source | YouTube |
| Video URL | https://youtu.be/mGjANZAx45k |
| Resolution | 1080p |
| Trimmed Segment | 1:34 – 1:43 (run-up and jump phase) |
| Editing Tool | [Online Video Cutter](https://online-video-cutter.com) |
| Total Frames | 492 (at 60 FPS) |
| Frames Extracted | 164 (every 3rd frame) |

---

## 📁 Project Structure

```
IP_Project/
│
├── videos/
│   └── match1.mp4                         # Trimmed long jump video (input)
│
├── frames/match1/                         # Raw extracted colour frames
├── grayscale_frames/match1/               # Grayscale-converted frames
├── denoised_frames/match1/                # NLM-denoised frames
├── enhanced_frames/match1/                # CLAHE-enhanced frames
├── roi_frames/match1/                     # ROI foreground-masked frames
├── tracked_frames/match1/                 # CSRT-tracked frames (grayscale)
├── Final_tracked_frames/match1/           # CSRT-tracked frames (colour)
│
├── output_videos/
│   └── match1_player_tracking.mp4         # Final annotated output video
│
├── 01_frame_extraction.ipynb
├── 02_grayscale_conversion.ipynb
├── 03_noise_analysis.ipynb
├── 04_gaussian_denoising.ipynb
├── 05_clahe_enhancement.ipynb
├── 06_enhancement_comparison.ipynb
├── 07_player_roi_detection.ipynb
├── 08_player_tracking_csrt.ipynb
├── 09_tracking_analysis.ipynb
└── 10_color_video_bordered_players.ipynb
```

---

## 🔄 Pipeline Stages

| # | Notebook | What It Does |
|---|---|---|
| 01 | `01_frame_extraction.ipynb` | Reads the trimmed MP4 and saves every 3rd frame as a JPG image (164 frames total) |
| 02 | `02_grayscale_conversion.ipynb` | Converts all colour frames to single-channel grayscale (pixel values 0–255) |
| 03 | `03_noise_analysis.ipynb` | Analyses frames for Gaussian, Salt-and-Pepper, Speckle, and Poisson noise using Std Dev, VMR, and impulse ratio |
| 04 | `04_gaussian_denoising.ipynb` | Compares Gaussian Blur, Median, Bilateral, and Non-Local Means filters; applies **NLM** to all frames |
| 05 | `05_clahe_enhancement.ipynb` | Applies **CLAHE** (clipLimit=2.0, tileGridSize=8×8) to boost local contrast across all frames |
| 06 | `06_enhancement_comparison.ipynb` | Side-by-side visual and histogram comparison of grayscale → denoised → CLAHE stages |
| 07 | `07_player_roi_detection.ipynb` | Crops a Region of Interest and runs **MOG2 background subtraction** to generate foreground masks |
| 08 | `08_player_tracking_csrt.ipynb` | User selects the player with a bounding box; **CSRT tracker** tracks the athlete across enhanced grayscale frames |
| 09 | `09_tracking_analysis.ipynb` | Reconstructs the player trajectory, computes frame-by-frame displacement, and checks tracking stability |
| 10 | `10_color_video_bordered_players.ipynb` | Re-runs CSRT tracking on original **colour** frames and exports the final annotated MP4 video |

---

## 🛠️ Technologies Used

- **Python 3.7+**
- **OpenCV** (`opencv-contrib-python`) — VideoCapture, CLAHE, MOG2 background subtraction, CSRT tracker
- **NumPy** — pixel statistics, noise metrics (Std Dev, VMR, impulse ratio)
- **Matplotlib** — frame visualisation and histogram plots
- **Jupyter Notebook** — interactive step-by-step execution environment

---

## ⚙️ Installation

**1. Clone the repository**
```bash
git clone https://github.com/aadhil-nisar/IP_PROJECT.git
cd IP_PROJECT
```

**2. Install required packages**
```bash
pip install opencv-contrib-python numpy matplotlib jupyter
```

> ⚠️ You must install **`opencv-contrib-python`** (not plain `opencv-python`). The CSRT tracker is only included in the contrib version.

**3. Launch Jupyter Notebook**
```bash
jupyter notebook
```

---

## 🚀 How to Run

> Run the notebooks **in order from 01 to 10**. Each notebook saves its output into a folder that the next notebook reads from.

### Step 1 — Video is already included

The trimmed video is already included in the repository at `videos/match1.mp4`. No downloading or trimming is needed — just clone the repo and it's ready to use.

### Step 2 — Update file paths

In each notebook, update the path variables to point to your local folders. Example from `01_frame_extraction.ipynb`:

```python
video_path    = r"C:\your\path\videos\match1.mp4"
output_folder = r"C:\your\path\frames\match1"
```

Repeat this for every notebook before running it.

### Step 3 — Run notebooks 01 through 07

These run fully automatically. Open each notebook and run all cells — no manual interaction needed.

### Step 4 — Interactive player selection (Notebooks 08 & 10)

When you run **notebook 08** or **notebook 10**, an OpenCV window will open showing the first video frame. Use your mouse to **draw a bounding box around the athlete**, then:

- Press **Enter** or **Space** to confirm the selection
- Press **C** to cancel and redraw

The CSRT tracker will then automatically track the selected athlete through all remaining frames.

### Step 5 — View the final output

The final annotated colour video is saved to:
```
output_videos/match1_player_tracking.mp4
```

---

## 📊 Key Techniques Explained

### Noise Analysis (Notebook 03)

Each frame is assessed using statistical metrics to classify noise type:

| Noise Type | Detection Condition | Result for This Video |
|---|---|---|
| Gaussian | Std Dev > 20, low impulse ratio, VMR not near 1 | ✅ Present (Std Dev = 28.37, VMR = 5.82) |
| Salt-and-Pepper | Impulse ratio ≥ 1% | ❌ Not Present (ratio = 0.0008) |
| Speckle | High variance + VMR > 2 + low impulse ratio | ❌ Not Present |
| Poisson | VMR close to 1 | ❌ Not Present |

### Denoising (Notebook 04)

Four methods were evaluated side-by-side on the same frame:
- Gaussian Blur
- Median Filter
- Bilateral Filter
- **Non-Local Means (NLM)** ← selected for final pipeline

NLM was chosen because it preserves fine edge detail (athlete outline) while effectively reducing the confirmed Gaussian noise.

### CLAHE Enhancement (Notebook 05)

Rather than equalizing the whole image at once, CLAHE applies histogram equalization to small local 8×8 tile patches. This boosts the contrast between the athlete and the track background while preventing noise amplification — especially important under the uneven outdoor stadium lighting conditions.

### CSRT Player Tracking (Notebooks 08 & 10)

The **Channel and Spatial Reliability Tracker (CSRT)** was selected for its high accuracy with fast-moving subjects, partial occlusions, and appearance changes — all common challenges in sports video. The tracker is manually initialised with a bounding box on the first frame and automatically propagated through all subsequent frames. A white rectangle is drawn on each tracked frame.

---

## 📽️ Output

The final output is a colour MP4 video (`match1_player_tracking.mp4`) with a **white bounding box** drawn around the tracked athlete in every frame, capturing the complete run-up phase of the long jump.

---

## 🔮 Future Improvements

- Apply deep learning detectors (e.g. YOLOv8) for automatic player detection, removing the need for manual bounding box initialisation
- Extend the system to multi-object tracking for team sports
- Add jump distance estimation by mapping pixel coordinates to real-world measurements
- Automate video trimming and folder path configuration

---

## 📋 Requirements Summary

```
opencv-contrib-python
numpy
matplotlib
jupyter
```

---

## 📚 References

1. OpenCV Documentation — https://opencv.org
2. Source Video — https://youtu.be/mGjANZAx45k
3. Gonzalez, R.C., Woods, R.E. — *Digital Image Processing*, Pearson Education
4. Online Video Cutter — https://online-video-cutter.com

---

## 👨‍💻 Author

**M.N.M Aadhil**  
Reg No: ASP/2023/061  
Department of Computing, Rajarata University of Sri Lanka  
GitHub: [@aadhil-nisar](https://github.com/aadhil-nisar)

