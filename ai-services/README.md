# Python AI Services Blueprint

## Stack

- FastAPI for inference endpoints.
- OpenCV for image preprocessing and color analysis.
- MediaPipe for pose and face landmarks.
- Scikit-learn / XGBoost / PyTorch for ranking models.

## Pipelines

1. Media validation and preprocessing.
2. Body/face landmark extraction.
3. Skin tone and geometry feature extraction.
4. Attribute classification (body + face + tone).
5. Recommendation candidate generation and ranking.
6. Product feature matching metadata output.

## Service endpoints (internal)

- `POST /internal/analyze`
- `POST /internal/recommend`
- `GET /internal/health`
