# Jordanian ID Information Extractor

A computer-vision notebook project for extracting data from Jordanian national ID images and verifying identity with face similarity.

This repository currently contains a single Jupyter notebook (`ID extraction.ipynb`) that demonstrates:
- OCR-based ID text extraction with **EasyOCR** (Arabic + English)
- Face-region extraction from an ID image using **OpenCV** cropping
- Face embedding + comparison using **facenet-pytorch** (**MTCNN** + **InceptionResnetV1**)
- Conditional storage of verified IDs in a CSV file

---

## Project Structure

```text
.
├── ID extraction.ipynb
├── README.md
├── sample 22.png
├── sample 23.jpg
├── sample 24.jpg
├── sample 25.jpg
└── sample 26.jpg
```

> There are no standalone Python modules, CLI scripts, or `requirements.txt` file in this repository at this time.

---

## What the Notebook Does

### 1) Jordanian ID OCR Extraction
The notebook loads an ID image (`id sample.png` in the example), runs EasyOCR with:

- Languages: `['ar', 'en']`
- Confidence filtering: `conf > 0.5`
- Regex pattern for ID: `\d{10}[ء-ي]`

If matched, it prints the extracted Jordanian ID value.

### 2) Face Extraction from the ID Card
The notebook extracts a face region by cropping a fixed ROI from the ID image dimensions:

- Vertical range: `36%` to `95%` of image height
- Horizontal range: `5%` to `35%` of image width

The cropped face is saved as `extracted_face.png`.

### 3) Face Verification Workflow
The pipeline compares:
- Cropped face from ID (`extracted_face.png`)
- External image (`image.png` in the example)

Using:
- `MTCNN` for face detection/cropping
- `InceptionResnetV1(pretrained="vggface2")` for embeddings
- Cosine similarity with default threshold `0.6`

If similarity is above threshold, the pipeline treats them as the same person.

### 4) CSV Registration Step
After a successful match, the notebook attempts to add the extracted ID to a CSV database.

---

## Additional Notebook Cells

The notebook also contains:
- Sample image visualization cells (`sample 22/24/25`)
- A helper `face(img)` function for ROI visualization
- A second OCR example that attempts to extract:
  - National ID (`\d{10}`)
  - Name (keyword-based)
  - Date of birth (`dd/mm/yyyy` pattern)

---

## Setup / Installation

Because the project is notebook-based, you can run it in **Google Colab** or locally in Jupyter.

### Option A: Google Colab
The notebook already includes:

```python
!pip install easyocr
!pip install facenet_pytorch
```

Open `ID extraction.ipynb` in Colab and run cells in order.

### Option B: Local Jupyter Environment
1. Create and activate a Python virtual environment.
2. Install dependencies:

```bash
pip install easyocr facenet_pytorch opencv-python pillow numpy pandas matplotlib torch torchvision
```

3. Launch Jupyter:

```bash
jupyter notebook
```

4. Open and run:
- `ID extraction.ipynb`

> Note: `google.colab.patches.cv2_imshow` is Colab-specific. For local runs, use Matplotlib or `cv2.imshow` alternatives.

---

## Inputs and Outputs

### Expected Inputs (as referenced by notebook cells)
- `id sample.png` (Jordanian ID card image)
- `image.png` (face image to compare against ID face)

You can change these paths in notebook variables:
- `id_image_path`
- `other_face_path`

### Generated/Used Outputs
- `extracted_face.png` (cropped ID face)
- CSV database file for stored IDs (see note below)

### Important Current Behavior
In the current notebook code, `add_to_database` defines:
- `def add_to_database(id_number, path='database_csv')`

So, if called without a path argument, it uses the literal filename `database_csv` rather than `verified_ids.csv`. If you rely on `verified_ids.csv`, pass the path explicitly when calling the function.

---

## Model and Runtime Requirements

- OCR: EasyOCR with Arabic + English recognition
- Face detector: MTCNN
- Face embedding model: InceptionResnetV1 (`vggface2` pretrained weights)
- Runtime: CPU or CUDA-enabled GPU (`torch.device("cuda" if available else "cpu")`)

First-time model downloads may take time depending on network speed.

---

## Limitations

- Notebook workflow is prototype-style and not packaged as a production API/service.
- Face ROI extraction is fixed-percentage cropping and may fail on differently formatted ID images.
- OCR quality depends on image resolution, blur, rotation, and lighting.
- Regex-based extraction may miss edge cases in real IDs.
- Face verification threshold (`0.6`) may require calibration for your dataset and risk tolerance.

---

## Privacy and Security Considerations

This project processes highly sensitive data (national IDs + face images). You should:

- Obtain consent and follow applicable legal/privacy regulations.
- Avoid committing real identity/biometric data to Git repositories.
- Encrypt stored outputs where possible.
- Restrict file and notebook access to authorized users only.
- Define retention/deletion policies for ID and biometric records.

---

## Troubleshooting

- **`Could not load image at ...`**
  - Check image paths and working directory.
- **`Could not detect face in one of the images.`**
  - Use clearer frontal faces and higher-resolution images.
- **No ID extracted**
  - Improve image quality; inspect OCR results and regex assumptions.
- **`ModuleNotFoundError`**
  - Reinstall missing dependencies in the active environment.
- **`cv2_imshow` import error (local environment)**
  - Replace Colab display calls with Matplotlib display code.

---

## Contributing

Contributions are welcome. Recommended approach:

1. Open an issue describing the improvement.
2. Keep changes focused and small.
3. Include clear reproduction/usage notes for notebook changes.
4. Validate notebook cells after updates.

Potential contribution areas:
- Improve OCR robustness (preprocessing, orientation handling)
- Replace fixed ROI cropping with face detection on full ID image
- Add reproducible dataset-free tests for utility logic
- Refactor notebook logic into reusable Python modules

---

## Disclaimer

This repository appears to be a research/prototype workflow and is not a certified identity verification system. Use responsibly and validate performance before any real-world deployment.
