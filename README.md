# Jordanian ID Information Extractor

A computer-vision notebook project for:

- extracting Jordanian national ID text using OCR (Arabic + English), and
- verifying whether an external face image matches the ID holder using FaceNet embeddings and cosine similarity.

> Current implementation lives in a single notebook: `ID extraction.ipynb`.

---

## Overview

This project demonstrates an end-to-end identity-processing workflow built in Python/Colab:

1. Read a Jordanian ID image.
2. Run OCR with EasyOCR to detect ID-related text.
3. Crop the face region from the ID card using fixed ROI coordinates.
4. Generate face embeddings with `facenet-pytorch` (`MTCNN` + `InceptionResnetV1`).
5. Compare embeddings against a second face image using cosine similarity.
6. If matched, append the extracted ID number to a CSV database.

It also includes additional notebook cells for sample image visualization and extended OCR extraction (name, DOB, ID).

---

## Key Features

- **Arabic/English OCR** with confidence filtering.
- **Jordanian ID pattern extraction** via regex.
- **Face matching** with normalized FaceNet embeddings.
- **Cosine-similarity thresholding** (`threshold=0.6` by default).
- **CSV persistence** for verified IDs.
- **Colab-friendly workflow** (`cv2_imshow`, inline visualization).

---

## Repository Structure

Based on the current repository contents:

- `ID extraction.ipynb` — main implementation notebook (OCR + face verification pipeline).
- `README.md` — this documentation.
- `sample 22.png`, `sample 23.jpg`, `sample 24.jpg`, `sample 25.jpg`, `sample 26.jpg` — sample image assets used for visualization/testing in notebook cells.

---

## Technology Stack

- **Python 3** (notebook kernel)
- **OpenCV (`cv2`)** for image I/O and manipulation
- **EasyOCR** for OCR in Arabic and English
- **PyTorch** backend
- **facenet-pytorch**:
  - `MTCNN` for face detection/cropping
  - `InceptionResnetV1(pretrained="vggface2")` for embeddings
- **NumPy**, **Pillow**, **pandas**, **csv**, **regex (`re`)**
- **Google Colab display helper**: `google.colab.patches.cv2_imshow`

---

## Installation & Setup

### Option A: Google Colab (closest to current notebook behavior)

1. Upload/open `ID extraction.ipynb` in Colab.
2. Run dependency cells:

```python
!pip install easyocr
!pip install facenet_pytorch
```

3. Upload required images to the Colab runtime (see [Inputs](#expected-inputs)).

### Option B: Local Jupyter

If running locally, install dependencies in your environment:

```bash
pip install easyocr facenet_pytorch opencv-python pillow torch torchvision pandas matplotlib
```

> Note: `cv2_imshow` is Colab-specific. In local Jupyter, replace with `matplotlib` or `cv2.imshow`-compatible display code.

---

## End-to-End Workflow (Implementation-Aligned)

In `ID extraction.ipynb`, the main pipeline cell does the following:

1. Initializes OCR and face models.
2. Loads paths:
   - `id_image_path = 'id sample.png'`
   - `other_face_path = 'image.png'`
   - `extracted_face_path = 'extracted_face.png'`
3. Extracts ID text from OCR output with:
   - confidence filter `conf > 0.5`
   - regex pattern `\d{10}[ء-ي]`
4. Crops the face from ID card with fixed ROI:
   - rows: `0.36 * height` to `0.95 * height`
   - cols: `0.05 * width` to `0.35 * width`
5. Saves cropped face to `extracted_face.png`.
6. Compares cropped ID face vs `image.png` using cosine similarity.
7. If similarity is above threshold (`0.6`), stores the ID value in CSV.

---

## Usage

Run the notebook cells in order.

### Minimal pipeline execution

Set image paths in the main pipeline cell:

```python
id_image_path = 'id sample.png'
other_face_path = 'image.png'
extracted_face_path = 'extracted_face.png'
database_csv = 'verified_ids.csv'
```

Then execute the cell containing:

- `compare_faces(...)`
- `add_to_database(...)`
- OCR + ID extraction logic

### Example console output

```text
Extracted Jordanian ID Number: 1234567890ا
Cosine similarity: 0.7421
Same person
ID number 1234567890ا added to database.
```

or

```text
Jordanian ID Number not found.
```

or

```text
Cosine similarity: 0.4123
Different person. ID not added.
```

---

## Expected Inputs

The current notebook expects:

1. **ID image** (`id sample.png` by default):
   - should contain readable ID text,
   - should include the person face in approximately expected card location.
2. **Reference face image** (`image.png` by default):
   - image of the claimed ID owner for verification.

Supported formats depend on OpenCV/Pillow readers (PNG/JPG work in provided examples).

---

## Expected Outputs

- Printed OCR/verification status in notebook output.
- `extracted_face.png` (cropped face from ID image).
- CSV file with header `ID Number` and accepted IDs.

---

## Configuration & Model Notes

- Device auto-selection:

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

- OCR languages are fixed to `['ar', 'en']`.
- Face encoder model is fixed to `InceptionResnetV1(pretrained="vggface2")`.
- Face comparison threshold default is `0.6` in `compare_faces(..., threshold=0.6)`.

---

## Face Verification and Cosine Similarity Considerations

The notebook:

1. extracts one face embedding per image,
2. L2-normalizes both embeddings,
3. computes cosine similarity via dot product,
4. accepts a match when `similarity >= threshold`.

Practical guidance:

- **Higher threshold** → fewer false accepts, more false rejects.
- **Lower threshold** → fewer false rejects, more false accepts.
- Tune threshold using representative Jordanian ID + selfie datasets from your deployment context.
- Ensure consistent lighting, pose, and image sharpness for stable similarity scores.

---

## OCR & Extraction Limitations

Current implementation constraints:

- ID extraction regex is strict (`\d{10}[ء-ي]`) and may miss valid variants.
- OCR confidence cutoff (`> 0.5`) can drop partially readable fields.
- Face crop uses a **fixed ROI**; unusual card alignment/scaling can break extraction.
- Multi-face scenarios default to the first detected face embedding.
- Additional name/DOB extraction cells are heuristic and keyword-based.

---

## Privacy, Security, and Responsible Use

This project processes sensitive identity and biometric data. You should:

- Obtain explicit legal/user consent before processing ID or face images.
- Minimize stored data (store only what is necessary).
- Protect images/CSV outputs with encryption and strict access controls.
- Avoid committing real IDs, face images, or generated biometric artifacts to source control.
- Define retention/deletion policies for identity data.
- Validate compliance with local regulations and organizational security policy.

---

## Troubleshooting

### `FileNotFoundError: Could not load image at: ...`
Check that `id_image_path`/`other_face_path` point to files present in the current runtime working directory.

### `Could not detect face in one of the images.`
Use clearer frontal images and verify that the ID face region is visible and not heavily occluded.

### OCR fails to find ID
Try higher-resolution images, better lighting, and preprocessing (denoise/contrast) before `readtext`.

### CSV filename mismatch
The pipeline defines `database_csv = 'verified_ids.csv'`, but `add_to_database` currently defaults to the literal `'database_csv'` unless a path is passed explicitly. If needed, call:

```python
add_to_database(id_number, database_csv)
```

---

## Contributing

Contributions are welcome. Recommended process:

1. Fork the repository.
2. Create a feature/fix branch.
3. Keep changes focused and reproducible.
4. Validate notebook cells from top to bottom.
5. Open a PR describing:
   - what changed,
   - why it changed,
   - sample input/output evidence.

When contributing, prioritize improvements in:

- OCR robustness for Jordanian IDs,
- dynamic face localization (instead of fixed ROI),
- configurable thresholds and structured outputs,
- stronger privacy-preserving data handling.
