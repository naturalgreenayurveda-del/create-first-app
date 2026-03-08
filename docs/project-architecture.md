# Complete Architecture: AI-Powered Fashion Stylist Shopping App

## 1) Complete Project Architecture

### High-level architecture

```text
Flutter Mobile App
   |
   | HTTPS + JWT
   v
Node.js API Gateway (Express/Nest)
   |-- Firebase Auth (identity)
   |-- Firestore (profiles, recommendations, sessions)
   |-- Firebase Storage (photo/video uploads)
   |-- Queue/Event Bus (Pub/Sub/Redis)
   |-- Product Aggregator APIs (Amazon, Myntra, Zalando, etc.)
   |
   v
Python AI Services (FastAPI)
   |-- Media Processing: OpenCV
   |-- Landmark/pose/face: MediaPipe
   |-- Feature Extraction (body shape, skin tone, face shape)
   |-- Recommendation Engine (rules + ML ranking)
   |-- Hairstyle/Accessories/Shoes matching
   |
   v
Recommendation Results + Buy Links
   |
   v
Flutter App Recommendation UI + Checkout Redirects
```

### Component responsibilities

- **Flutter app**: authentication, media capture/upload, loading states, recommendation visualization, product cards, external buy links.
- **Node.js backend**: session orchestration, business logic, auth verification, API aggregation, persistence, and secure signed upload URLs.
- **Firebase**:
  - **Auth** for signup/login.
  - **Firestore** for users, style profiles, analysis sessions, recommendations, clicks/conversions.
  - **Storage** for images/videos and thumbnails.
- **Python AI layer**:
  - Body and face feature extraction with MediaPipe landmarks.
  - Skin tone estimation with OpenCV color analysis.
  - Hybrid recommendation generation (rule engine + ranking model).
- **Commerce integrations**: normalized product catalog + affiliate tracking links.

### Non-functional architecture essentials

- **Scalability**: asynchronous analysis jobs via queue workers.
- **Performance**: split quick preview recommendations (<3s) vs deep recommendations (10–30s).
- **Security**: signed URLs, JWT auth, data encryption at rest, strict Firebase security rules.
- **Observability**: centralized logs, request IDs, metrics for latency and conversion.

---

## 2) Recommended Folder Structure

```text
create-first-app/
├── mobile_app/
│   └── flutter_stylist_app/
│       ├── lib/
│       │   ├── core/                 # theme, constants, networking, DI
│       │   ├── features/
│       │   │   ├── auth/
│       │   │   ├── onboarding/
│       │   │   ├── upload/
│       │   │   ├── analysis/
│       │   │   ├── recommendations/
│       │   │   ├── shopping/
│       │   │   └── profile/
│       │   ├── shared/               # reusable widgets/models
│       │   └── main.dart
│       ├── assets/
│       └── test/
│
├── backend/
│   └── node-api/
│       ├── src/
│       │   ├── modules/
│       │   │   ├── auth/
│       │   │   ├── users/
│       │   │   ├── media/
│       │   │   ├── analysis/
│       │   │   ├── recommendations/
│       │   │   ├── products/
│       │   │   └── links/
│       │   ├── middleware/
│       │   ├── integrations/
│       │   ├── queues/
│       │   └── app.ts
│       ├── tests/
│       └── package.json
│
├── ai-services/
│   ├── app/
│   │   ├── api/
│   │   ├── pipelines/
│   │   ├── models/
│   │   ├── features/
│   │   ├── recommenders/
│   │   └── utils/
│   ├── notebooks/
│   ├── tests/
│   └── requirements.txt
│
├── shared/
│   ├── contracts/                    # OpenAPI + JSON schemas
│   └── enums/                        # body shape types, face shapes, etc.
│
└── docs/
    ├── project-architecture.md
    └── development-roadmap.md
```

---

## 3) Backend API Design (Node.js)

### API domain model

- `User`
- `StyleProfile` (height, weight range optional, preferences, climate, budget)
- `AnalysisSession`
- `BodyAnalysisResult`
- `RecommendationBundle`
- `ProductItem`
- `AffiliateLink`

### Core endpoints

#### Authentication

- `POST /v1/auth/signup`
- `POST /v1/auth/login`
- `POST /v1/auth/logout`
- `GET /v1/auth/me`

> Use Firebase Auth tokens and verify in backend middleware.

#### User/Profile

- `GET /v1/users/profile`
- `PUT /v1/users/profile`
- `PUT /v1/users/preferences` (style, colors, brands, budget)

#### Media Upload

- `POST /v1/media/upload-url` → returns signed upload URL for image/video.
- `POST /v1/media/complete` → confirms upload and creates analysis session.
- `GET /v1/media/:mediaId`

#### AI Analysis

- `POST /v1/analysis/start` (async job trigger)
- `GET /v1/analysis/:sessionId/status`
- `GET /v1/analysis/:sessionId/result`

#### Recommendations

- `GET /v1/recommendations/:sessionId`
  - returns outfit, hairstyle, accessories, shoes + confidence score.
- `POST /v1/recommendations/:sessionId/feedback`

#### Product Search + Buy Links

- `GET /v1/products/search?category=&style=&color=&budget=`
- `GET /v1/products/:id`
- `GET /v1/products/:id/buy-links`
- `POST /v1/links/click` (analytics for conversions)

### Sample recommendation response

```json
{
  "sessionId": "sess_123",
  "analysis": {
    "bodyShape": "athletic",
    "estimatedHeightBand": "170-175cm",
    "skinTone": "warm-medium",
    "faceShape": "oval",
    "confidence": 0.89
  },
  "recommendations": {
    "outfits": [
      {"name": "Slim Fit Blazer + Tapered Trousers", "score": 0.93}
    ],
    "hairstyles": [
      {"name": "Textured Quiff", "score": 0.9}
    ],
    "accessories": [
      {"type": "watch", "name": "Minimal Steel Watch", "score": 0.88}
    ],
    "shoes": [
      {"name": "White Leather Sneakers", "score": 0.91}
    ]
  },
  "products": [
    {
      "id": "prod_001",
      "title": "Navy Slim Fit Blazer",
      "price": 89.99,
      "currency": "USD",
      "store": "ExampleStore",
      "buyUrl": "https://..."
    }
  ]
}
```

---

## 4) AI Processing Pipeline

### Pipeline stages

1. **Ingestion**
   - Receive media reference from backend (`image` or `short video`).
   - Validate quality (resolution, brightness, full-body visibility).

2. **Preprocessing**
   - Frame extraction for video (e.g., 1 frame/second).
   - Background normalization and subject detection.
   - Pose alignment and crop generation.

3. **Feature Extraction**
   - **MediaPipe Pose**: body keypoints for body-shape heuristics.
   - **MediaPipe Face Mesh**: face geometry for face shape.
   - **OpenCV**: skin tone sampling from robust facial/neck regions.
   - Height estimation using keypoint proportions + optional user-entered baseline.

4. **Attribute Classification**
   - Body shape class (`rectangle`, `triangle`, `inverted triangle`, `oval`, `athletic`).
   - Face shape class (`oval`, `round`, `square`, `heart`, `diamond`).
   - Skin undertone (`warm`, `cool`, `neutral`) + depth (`light`, `medium`, `deep`).

5. **Recommendation Engine**
   - Rule-based candidate generation from style knowledge graph.
   - ML ranking model (XGBoost/LightGBM/Neural ranker) using:
     - body/face/skin features,
     - user preferences,
     - budget,
     - context (occasion, climate).

6. **Product Matching**
   - Map abstract style items to real products via commerce APIs.
   - Re-rank by availability, rating, shipping, and price fitness.

7. **Response Packaging**
   - Return top-N outfits + hairstyle + accessories + shoes.
   - Include confidence and explanation bullets.

### Model improvement loop

- Capture explicit feedback (`like/dislike`, `too expensive`, `not my style`).
- Train periodic personalization models (nightly/weekly).
- A/B test ranking variants for CTR and conversion lift.

---

## 5) Mobile UI Structure (Flutter)

### App navigation

- **Splash** → **Auth** → **Home Upload** → **Processing** → **Recommendations** → **Product Detail**.

### Key screens

1. **Auth Screens**
   - Signup/login (email, Google, Apple).
   - Lightweight onboarding questionnaire (style, budget, occasions).

2. **Upload Screen**
   - Camera capture or gallery/video picker.
   - Full-body framing guide overlay.
   - Consent and privacy notice.

3. **Analysis Screen**
   - Progress timeline (uploading, analyzing body, generating looks).
   - Skeleton loading and expected wait time.

4. **Recommendations Screen**
   - Tabs: `Outfit`, `Hairstyle`, `Accessories`, `Shoes`.
   - AI insights chips (Body: Athletic, Face: Oval, Tone: Warm-Medium).
   - “Why this works” explainability card.

5. **Shopping Screen**
   - Product cards with image, store, price, rating.
   - Filters (price, brand, color, size availability).
   - Buy button opens e-commerce affiliate link in in-app browser.

6. **Profile & History**
   - Saved looks, previous sessions, preferences, privacy controls.

### UI architecture recommendation

- State management: `Riverpod` or `Bloc`.
- Routing: `go_router`.
- Design system:
  - neutral palette,
  - large cards,
  - modern typography,
  - bottom navigation,
  - subtle motion animations.

---

## 6) Development Roadmap (summary)

### Phase 1: Foundation (Week 1–2)
- Repo setup, CI/CD, Firebase project, auth integration.
- Flutter skeleton + Node.js baseline APIs.
- Media upload flow with signed URLs.

### Phase 2: AI MVP (Week 3–5)
- MediaPipe + OpenCV feature extraction.
- Initial rule-based recommendation logic.
- First recommendation results in app.

### Phase 3: Shopping Integration (Week 6–7)
- Product search connectors.
- Buy link redirection + click analytics.
- Filter/sort UI and product details.

### Phase 4: Personalization (Week 8–10)
- Feedback loop and ranking model.
- Session history + saved looks.
- Performance tuning for faster inference.

### Phase 5: Production Hardening (Week 11–12)
- Security review, privacy and data retention policy.
- Observability dashboards and alerting.
- Beta rollout, A/B tests, and launch readiness.
