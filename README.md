# MediVision AI

**An AI-powered medical image processing, enhancement and analysis platform.**

MediVision AI is a full-stack web application that brings Digital Image Processing (DIP), Computer Vision, Deep Learning and web development together in one workspace. Users can upload X-ray and MRI images, enhance and segment them, compare results, view explainability heatmaps and generate technical reports.

> **Disclaimer:** MediVision AI is an academic and research project. Its classifications, heatmaps and segmentation masks are **experimental**, have **not** been clinically validated, and must **not** be used for diagnosis, treatment or any medical decision. Use only public, de-identified research datasets.

---

## Table of contents

- [Features](#features)
- [Project status](#project-status)
- [Tech stack](#tech-stack)
- [Architecture](#architecture)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [API overview](#api-overview)
- [Image processing and AI](#image-processing-and-ai)
- [Datasets](#datasets)
- [Database design](#database-design)
- [Security and privacy](#security-and-privacy)
- [Testing](#testing)
- [Roadmap](#roadmap)
- [Author](#author)

---

## Features

**Imaging modalities:** X-ray and MRI.

- **Upload and view:** drag-and-drop upload, file validation, zoom, pan, rotate, flip, before/after slider.
- **Enhancement:** grayscale, Gaussian and median filtering, histogram equalization, CLAHE, sharpening.
- **Analysis tools:** Canny, Sobel and Laplacian edge detection, Otsu thresholding, morphology, contours, watershed.
- **X-ray classification:** ResNet18 transfer learning (Normal vs Pneumonia) with confidence score and Grad-CAM heatmap.
- **MRI segmentation:** 2D U-Net producing a mask and overlay, with region area statistics.
- **Comparison:** MSE, PSNR, SSIM and difference maps. A higher score does not always mean better medical image quality.
- **History and reports:** every operation is recorded; downloadable PDF reports include steps, parameters, images and metrics.
- **Accounts:** JWT authentication with per-user access to images and results.

## Project status

| Area | Status |
| --- | --- |
| Project blueprint and SRS | Done |
| Landing page and interactive demo (upload, filters, compare, history, report download) | Done, runs fully in the browser |
| React app (dashboard, workspace pages) | Planned |
| FastAPI backend, MongoDB, authentication | Planned |
| OpenCV processing engine | Planned |
| ResNet18 classifier, Grad-CAM | Planned |
| U-Net MRI segmentation | Planned |
| PDF reports (ReportLab) | Planned |

Update this table as you build.

## Tech stack

| Layer | Technology |
| --- | --- |
| Frontend | React.js (Vite), Tailwind CSS, Axios, React Router, Lucide React |
| Backend | Python, FastAPI, Uvicorn |
| Image processing | OpenCV, NumPy, SciPy, scikit-image, Pillow, SimpleITK, pydicom, NiBabel |
| Deep learning | PyTorch, torchvision, scikit-learn |
| Database | MongoDB (Motor / PyMongo) |
| Auth | JWT (python-jose), passlib (bcrypt) |
| Reports | ReportLab, Matplotlib |
| Testing | pytest, httpx |

## Architecture

Three layers:

1. **Presentation:** React single-page app (auth, dashboard, upload, viewer, processing workspace, AI analysis, reports, history).
2. **Application:** FastAPI services (authentication, image validation, processing engine, AI inference, segmentation, report generator, history manager).
3. **Processing and data:** OpenCV for classical processing, PyTorch for CNN, U-Net and Grad-CAM, MongoDB for metadata, private file storage for images, masks, heatmaps and reports.

Request flow: the user uploads an image from React, FastAPI validates it and stores the original, the processing or AI module runs, the result and history record are saved, and the response is returned to the UI. Image processing and AI inference live in separate Python modules so they can be tested independently and models can be replaced later.

## Project structure

```
medivision-ai/
├── frontend/
│   └── src/
│       ├── components/    # Navbar, Sidebar, ImageUploader, ImageViewer, BeforeAfter, ...
│       ├── pages/         # Home, Login, Dashboard, Upload, Processing, AIAnalysis, ...
│       ├── services/      # api.js, authService.js, imageService.js, reportService.js
│       ├── context/       # AuthContext.jsx
│       └── layouts/       # DashboardLayout.jsx
├── backend/
│   ├── app/
│   │   ├── main.py, config.py, database.py
│   │   ├── models/ schemas/ routes/ services/
│   │   ├── processing/    # preprocessing, enhancement, filtering, edge_detection, segmentation, comparison
│   │   ├── ai/            # classification, inference, gradcam, unet, model_loader
│   │   ├── utils/         # image_validation, dicom_utils, security
│   │   └── reports/       # pdf_generator.py
│   ├── storage/           # original, processed, masks, heatmaps, reports (private)
│   ├── trained_models/    # xray_classifier.pth, mri_segmenter.pth
│   ├── tests/
│   ├── requirements.txt
│   └── .env.example
├── docs/                  # architecture, API documentation, project report, SRS
├── .gitignore
└── README.md
```

Create folders as you implement their features; you do not need every file on day one.

## Getting started

### Prerequisites

- Python 3.10+
- Node.js 18+
- MongoDB (local or Atlas)
- Git
- Optional: CUDA GPU for model training

### 1. Clone

```bash
git clone https://github.com/<your-username>/medivision-ai.git
cd medivision-ai
```

### 2. Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # then fill in the values
uvicorn app.main:app --reload
```

Suggested `.env` values:

```
MONGODB_URI=mongodb://localhost:27017
DATABASE_NAME=medivision
JWT_SECRET=change-me
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
MAX_UPLOAD_MB=20
```

The API runs at `http://localhost:8000` and interactive docs at `http://localhost:8000/docs`.

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

The app runs at `http://localhost:5173`. Point the frontend API base URL at the backend (`/api/v1`).

### Backend dependencies

```
fastapi uvicorn[standard] python-multipart
opencv-python numpy pillow scikit-image scipy
torch torchvision scikit-learn
pymongo motor pydantic pydantic-settings
python-jose[cryptography] passlib[bcrypt]
pydicom SimpleITK nibabel
matplotlib reportlab pytest httpx
```

Pin versions after testing in your environment.

## API overview

All endpoints use the prefix `/api/v1` and require a Bearer JWT except registration and login.

| Group | Endpoints |
| --- | --- |
| Auth | `POST /auth/register`, `POST /auth/login`, `GET /auth/me`, `POST /auth/logout` |
| Images | `POST /images/upload`, `GET /images`, `GET /images/{id}`, `GET /images/{id}/original`, `DELETE /images/{id}` |
| Processing | `POST /processing/{denoise, clahe, sharpen, grayscale, edges, threshold}/{id}` |
| AI | `POST /ai/classify/{id}`, `POST /ai/segment/{id}`, `GET /ai/result/{id}`, `GET /ai/heatmap/{id}` |
| Comparison and history | `POST /compare`, `GET /history` |
| Reports | `POST /reports/generate/{id}`, `GET /reports`, `GET /reports/{id}/download` |

Example:

```http
POST /api/v1/processing/clahe/{image_id}
Authorization: Bearer <token>
Content-Type: application/json

{ "clip_limit": 2.0, "tile_grid_size": 8 }
```

```json
{
  "success": true,
  "operation": "CLAHE",
  "image_id": "<image_id>",
  "output_url": "/api/v1/processing/results/<processing_id>",
  "processing_time": 1.25
}
```

## Image processing and AI

| Category | Methods |
| --- | --- |
| Basic DIP | Grayscale, Gaussian blur, median filter, histogram equalization, CLAHE, sharpening |
| Advanced DIP | Canny, Sobel, Laplacian, Otsu, erosion, dilation, watershed, contours |
| Deep learning | ResNet18 classification, 2D U-Net segmentation, Grad-CAM explainability |
| Comparison | MSE, PSNR, SSIM, registration, difference mapping |

X-ray and MRI use **separate pipelines**:

- **X-ray:** validate, grayscale, denoise, CLAHE or histogram equalization, optional edges, optional CNN classification, original vs processed vs Grad-CAM.
- **MRI:** read dimensions and voxel spacing, normalize intensity, denoise and enhance, select 2D slices, run U-Net, mask and overlay, region measurements. Start with 2D slices for the MVP; add NIfTI and DICOM series later. Preserve original DICOM pixel data instead of converting everything to 8-bit JPEG.

## Datasets

| Task | Dataset | Model | Metrics |
| --- | --- | --- | --- |
| X-ray classification | Chest X-Ray Images (Pneumonia), Kaggle | ResNet18 | Accuracy, precision, recall, F1, confusion matrix, ROC-AUC |
| MRI segmentation | BraTS brain MRI | 2D U-Net | Dice, IoU, precision, recall, per-class metrics |

Download datasets yourself and respect their licences. Do not commit datasets or trained weights to Git (use Git LFS or release assets). Use patient-level splits where identifiers exist, and pay particular attention to sensitivity and false-negative rate.

## Database design

MongoDB collections: `users`, `medical_images`, `processing_history`, `ai_results`, `reports`.

- Use ObjectId references; store files on disk and only paths and metadata in MongoDB.
- Indexes: `users.email` (unique), `medical_images.user_id`, `processing_history.image_id`, `ai_results.image_id`, `reports.user_id`.

## Security and privacy

- Argon2 or bcrypt password hashing, JWT authentication, per-user resource ownership checks.
- MIME, extension and content validation; size and dimension limits; random server-generated filenames.
- Private storage that is never exposed directly; HTTPS in deployment; secrets in `.env`.
- Remove or protect identifying DICOM metadata; delete files and records together.
- Use public, de-identified datasets only. Never upload real patient records.

## Testing

```bash
cd backend
pytest
```

Test image processing (dimensions preserved, grayscale and colour inputs, noise reduction), AI (model loading, preprocessing, mask size, Grad-CAM, held-out performance) and the web app (auth, invalid and oversized uploads, unauthorized access, report generation, deletion, responsiveness).

## Roadmap

**MVP**

- [x] Project blueprint and SRS
- [x] Landing page and in-browser demo
- [ ] Login and registration
- [ ] X-ray and MRI upload with validation
- [ ] Enhancement, edge detection and thresholding API
- [ ] Image comparison (MSE, PSNR, SSIM)
- [ ] ResNet18 X-ray classification
- [ ] U-Net MRI segmentation
- [ ] PDF reports and history

**Advanced**

- [ ] Grad-CAM heatmaps
- [ ] 3D MRI volume viewer and NIfTI support
- [ ] DICOM series navigation
- [ ] Advanced segmentation and model evaluation dashboards
- [ ] Background jobs with Celery and Redis

Suggested 8-week plan: setup and UI, upload and viewer, image processing, MRI segmentation, X-ray classification, Grad-CAM and comparison, database and reports, testing and documentation.

## Author

**Aryan Tarush**, B.Tech CSE individual project.

## License

Add a `LICENSE` file (for example MIT) before publishing. Dataset and model licences are separate and must be respected.
