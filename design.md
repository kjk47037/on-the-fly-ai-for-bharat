# OnTheFly Platform Design Document

## Executive Summary

This document describes the system architecture, data models, API contracts, and technical design decisions for the OnTheFly platform. The platform follows a microservices architecture with separate Node.js and Python backends, a React frontend, and integration with multiple AI/ML services.

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  React 18 + Vite Frontend (Port 5173)                    │  │
│  │  - Tailwind CSS + DaisyUI                                │  │
│  │  - React Router DOM                                      │  │
│  │  - Recharts for visualization                            │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                           │
│  ┌────────────────────┐  ┌────────────────────┐  ┌───────────┐ │
│  │  Express Server    │  │  Flask API Server  │  │  Flask ML │ │
│  │  (Port 5005)       │  │  (Port 5004)       │  │  (Port    │ │
│  │                    │  │                    │  │   5007)   │ │
│  │  - Content Gen     │  │  - Twitter API     │  │           │ │
│  │  - Brand Kit       │  │  - Instagram API   │  │  - Model  │ │
│  │  - Campaign Mgmt   │  │  - Social Auth     │  │    Train  │ │
│  │  - SEO Audit       │  │  - WhatsApp        │  │  - Predict│ │
│  └────────────────────┘  └────────────────────┘  └───────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      EXTERNAL SERVICES                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Firebase    │  │  AI Services │  │  Social Media APIs   │  │
│  │              │  │              │  │                      │  │
│  │  - Auth      │  │  - HuggingFace│ │  - Twitter API v2   │  │
│  │  - Firestore │  │  - ModelsLab │  │  - Instagram API    │  │
│  │  - Storage   │  │  - Gemini    │  │  - Twilio WhatsApp  │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Component Architecture

#### Frontend Architecture

```
src/
├── pages/              # Route components
│   ├── Dashboard.jsx
│   ├── ContentStudio.jsx
│   ├── BrandKitGenerator.jsx
│   ├── CampaignBuilder.jsx
│   ├── Analytics.jsx
│   ├── ReachbeeCompetitorAnalysis.jsx
│   └── SEOAudit.jsx
├── components/         # Reusable UI components
│   ├── Navbar.jsx
│   ├── Sidebar.jsx
│   └── ProtectedRoute.jsx
├── hooks/             # Custom React hooks
│   └── useAuth.js
├── firebase.js        # Firebase configuration
└── main.jsx          # Application entry point
```

#### Backend Architecture (Express - Port 5005)

```
backend/src/
├── routes/
│   ├── content.routes.js      # Content generation endpoints
│   ├── brand.routes.js        # Brand kit endpoints
│   ├── campaign.routes.js     # Campaign management
│   └── seo.routes.js          # SEO audit endpoints
├── services/
│   ├── content.service.js     # AI content generation logic
│   ├── brand.service.js       # Brand kit generation
│   ├── campaign.service.js    # Campaign operations
│   └── seo.service.js         # SEO analysis
├── middleware/
│   └── auth.middleware.js     # Firebase auth verification
└── server.js                  # Express app configuration
```

#### Python Backend Architecture (Flask - Port 5004)

```
backend/events/
├── app.py                     # Flask application
├── routes/
│   ├── twitter_routes.py      # Twitter API integration
│   ├── instagram_routes.py    # Instagram API integration
│   └── whatsapp_routes.py     # WhatsApp notifications
└── services/
    ├── twitter_service.py
    └── instagram_service.py
```

#### ML Service Architecture (Flask - Port 5007)

```
backend/ml/
├── app.py                     # ML Flask application
├── train.py                   # Model training script
├── predict.py                 # Prediction service
├── models/                    # Saved ML models
│   └── model_*.joblib
└── data/                      # Training data
    └── twitter_data.csv
```

## Data Models

### User Model (Firebase Auth)

```typescript
interface User {
  uid: string;                 // Firebase UID
  email: string;
  displayName?: string;
  photoURL?: string;
  emailVerified: boolean;
  createdAt: Timestamp;
  lastLogin: Timestamp;
}
```

### Content Model (Firestore)

```typescript
interface Content {
  id: string;
  userId: string;
  platform: 'instagram' | 'twitter' | 'linkedin';
  text: string;
  imageUrl?: string;
  hashtags: string[];
  createdAt: Timestamp;
  updatedAt: Timestamp;
  campaignId?: string;
  published: boolean;
  publishedAt?: Timestamp;
  mlPrediction?: {
    category: 'low' | 'medium' | 'high';
    confidence: number;
    features: Record<string, number>;
  };
  performance?: {
    likes: number;
    comments: number;
    shares: number;
    engagementRate: number;
  };
}
```

### Campaign Model (Firestore)

```typescript
interface Campaign {
  id: string;
  userId: string;
  name: string;
  objective: 'awareness' | 'engagement' | 'conversion' | 'traffic';
  platforms: string[];
  status: 'draft' | 'active' | 'paused' | 'completed';
  startDate: Timestamp;
  endDate: Timestamp;
  budget?: number;
  targetAudience: {
    ageRange?: string;
    gender?: string;
    interests?: string[];
    location?: string;
  };
  contentIds: string[];
  metrics: {
    impressions: number;
    clicks: number;
    conversions: number;
    spend: number;
  };
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

### Brand Kit Model (Firestore)

```typescript
interface BrandKit {
  id: string;
  userId: string;
  brandName: string;
  industry: string;
  personality: string[];
  logos: {
    url: string;
    style: string;
    format: string;
  }[];
  colorPalette: {
    primary: string;
    secondary: string;
    accent: string;
    neutral: string[];
    accessibilityScore: number;
  };
  typography: {
    headingFont: string;
    bodyFont: string;
    fontPairings: string[];
  };
  toneGuidelines: {
    voice: string;
    dosList: string[];
    dontsList: string[];
  };
  createdAt: Timestamp;
}
```

### Analytics Model (Firestore)

```typescript
interface Analytics {
  id: string;
  userId: string;
  platform: string;
  date: Timestamp;
  metrics: {
    followers: number;
    following: number;
    totalPosts: number;
    totalLikes: number;
    totalRetweets: number;
    totalReplies: number;
    engagementRate: number;
  };
  topPosts: {
    id: string;
    text: string;
    likes: number;
    retweets: number;
    replies: number;
  }[];
  topHashtags: {
    tag: string;
    count: number;
    avgEngagement: number;
  }[];
}
```

### ML Training Data Model

```typescript
interface TrainingData {
  text: string;
  hashtag_count: number;
  text_length: number;
  has_url: boolean;
  has_mention: boolean;
  hour_posted: number;
  day_of_week: number;
  is_weekend: boolean;
  sentiment_score: number;
  question_count: number;
  exclamation_count: number;
  emoji_count: number;
  capital_ratio: number;
  word_count: number;
  avg_word_length: number;
  unique_word_ratio: number;
  has_media: boolean;
  is_reply: boolean;
  is_retweet: boolean;
  engagement_category: 'low' | 'medium' | 'high';
}
```

## API Contracts

### Content Generation API

#### POST /api/content/generate

**Request:**
```json
{
  "prompt": "string",
  "platform": "instagram" | "twitter" | "linkedin",
  "tone": "professional" | "casual" | "humorous" | "inspirational",
  "includeHashtags": boolean,
  "includeImage": boolean,
  "brandContext": {
    "industry": "string",
    "targetAudience": "string"
  }
}
```

**Response:**
```json
{
  "data": {
    "content": "string",
    "hashtags": ["string"],
    "imageUrl": "string",
    "mlPrediction": {
      "category": "low" | "medium" | "high",
      "confidence": number,
      "featureImportance": {
        "hashtag_count": number,
        "text_length": number,
        "sentiment_score": number
      }
    }
  },
  "error": null,
  "metadata": {
    "generationTime": number,
    "model": "string"
  }
}
```

### ML Prediction API

#### POST /predict

**Request:**
```json
{
  "text": "string",
  "hashtag_count": number,
  "platform": "string",
  "hour_posted": number,
  "has_media": boolean
}
```

**Response:**
```json
{
  "prediction": "low" | "medium" | "high",
  "confidence": number,
  "feature_importance": {
    "hashtag_count": number,
    "text_length": number,
    "sentiment_score": number
  },
  "model_version": "string"
}
```

### Twitter Analytics API

#### GET /api/twitter/fetch-analytics

**Query Parameters:**
- `userId`: string (required)
- `timeRange`: "7d" | "30d" | "90d" (optional, default: "30d")

**Response:**
```json
{
  "data": {
    "profile": {
      "username": "string",
      "followers": number,
      "following": number,
      "totalTweets": number
    },
    "metrics": {
      "totalLikes": number,
      "totalRetweets": number,
      "totalReplies": number,
      "engagementRate": number
    },
    "topTweets": [
      {
        "id": "string",
        "text": "string",
        "likes": number,
        "retweets": number,
        "replies": number,
        "createdAt": "string"
      }
    ],
    "topHashtags": [
      {
        "tag": "string",
        "count": number,
        "avgEngagement": number
      }
    ]
  },
  "error": null
}
```

### Brand Kit Generation API

#### POST /api/brand/generate

**Request:**
```json
{
  "brandName": "string",
  "industry": "string",
  "personality": ["string"],
  "targetAudience": "string",
  "preferences": {
    "colorScheme": "vibrant" | "muted" | "monochrome",
    "logoStyle": "modern" | "classic" | "playful"
  }
}
```

**Response:**
```json
{
  "data": {
    "logos": [
      {
        "url": "string",
        "style": "string",
        "prompt": "string"
      }
    ],
    "colorPalette": {
      "primary": "#hexcode",
      "secondary": "#hexcode",
      "accent": "#hexcode",
      "neutral": ["#hexcode"],
      "accessibilityScore": number
    },
    "typography": {
      "headingFont": "string",
      "bodyFont": "string",
      "fontPairings": ["string"]
    },
    "toneGuidelines": {
      "voice": "string",
      "dosList": ["string"],
      "dontsList": ["string"]
    }
  },
  "error": null
}
```

### Campaign Management API

#### POST /api/campaigns/create

**Request:**
```json
{
  "name": "string",
  "objective": "awareness" | "engagement" | "conversion" | "traffic",
  "platforms": ["string"],
  "startDate": "ISO8601",
  "endDate": "ISO8601",
  "budget": number,
  "targetAudience": {
    "ageRange": "string",
    "gender": "string",
    "interests": ["string"],
    "location": "string"
  }
}
```

**Response:**
```json
{
  "data": {
    "campaignId": "string",
    "status": "draft",
    "createdAt": "ISO8601"
  },
  "error": null
}
```

### SEO Audit API

#### POST /api/seo/audit

**Request:**
```json
{
  "url": "string",
  "options": {
    "checkMobile": boolean,
    "checkSpeed": boolean,
    "checkAccessibility": boolean
  }
}
```

**Response:**
```json
{
  "data": {
    "overallScore": number,
    "categories": {
      "technical": {
        "score": number,
        "issues": ["string"],
        "recommendations": ["string"]
      },
      "content": {
        "score": number,
        "keywords": ["string"],
        "keywordDensity": number
      },
      "performance": {
        "score": number,
        "loadTime": number,
        "pageSize": number
      }
    },
    "recommendations": [
      {
        "title": "string",
        "description": "string",
        "impact": "high" | "medium" | "low",
        "effort": "high" | "medium" | "low"
      }
    ]
  },
  "error": null
}
```

## ML Model Architecture

### RandomForest Classifier

**Model Type:** scikit-learn RandomForestClassifier

**Features (18 total):**
1. `hashtag_count` - Number of hashtags in post
2. `text_length` - Character count of post text
3. `has_url` - Boolean indicating URL presence
4. `has_mention` - Boolean indicating @mention presence
5. `hour_posted` - Hour of day (0-23)
6. `day_of_week` - Day of week (0-6)
7. `is_weekend` - Boolean for weekend posts
8. `sentiment_score` - Sentiment analysis score (-1 to 1)
9. `question_count` - Number of question marks
10. `exclamation_count` - Number of exclamation marks
11. `emoji_count` - Number of emojis
12. `capital_ratio` - Ratio of capital letters
13. `word_count` - Number of words
14. `avg_word_length` - Average word length
15. `unique_word_ratio` - Ratio of unique words
16. `has_media` - Boolean for media attachment
17. `is_reply` - Boolean for reply tweets
18. `is_retweet` - Boolean for retweets

**Target Variable:**
- `engagement_category`: "low", "medium", "high"
  - Low: < 33rd percentile of engagement
  - Medium: 33rd-66th percentile
  - High: > 66th percentile

**Model Hyperparameters:**
```python
RandomForestClassifier(
    n_estimators=100,
    max_depth=10,
    min_samples_split=5,
    min_samples_leaf=2,
    random_state=42,
    class_weight='balanced'
)
```

**Training Process:**
1. Load historical Twitter data from CSV
2. Feature engineering and extraction
3. Train-test split (80/20)
4. StandardScaler for feature normalization
5. Model training with cross-validation
6. Save model and scaler with timestamp
7. Log performance metrics (accuracy, precision, recall, F1)

**Prediction Pipeline:**
1. Extract features from new content
2. Apply saved scaler transformation
3. Generate prediction with confidence scores
4. Return feature importance for explainability

## Integration Architecture

### AI Service Integration

#### HuggingFace Integration
- **Model:** meta-llama/Llama-3.1-8B-Instruct
- **Purpose:** Text generation for social media content
- **API:** HuggingFace Inference API
- **Rate Limit:** Handled with retry logic
- **Timeout:** 30 seconds per request

#### ModelsLab Integration
- **API:** text2video endpoint
- **Purpose:** Video generation from text prompts
- **Format:** MP4 output
- **Processing Time:** 60-120 seconds

#### Google Gemini Integration
- **Model:** gemini-pro
- **Purpose:** Competitor analysis insights
- **API Key:** Stored in environment variable
- **Rate Limit:** 60 requests per minute

### Social Media API Integration

#### Twitter API v2
- **Authentication:** OAuth 2.0 Bearer Token
- **Endpoints Used:**
  - GET /2/users/:id - User profile
  - GET /2/users/:id/tweets - User tweets
  - POST /2/tweets - Create tweet
- **Rate Limits:** 
  - User lookup: 300 requests/15min
  - Tweet creation: 200 requests/15min

#### Instagram API (RapidAPI)
- **Authentication:** RapidAPI Key
- **Endpoints Used:**
  - GET /user/info - Profile information
  - GET /user/posts - Recent posts
- **Rate Limits:** Based on RapidAPI subscription

#### Twilio WhatsApp
- **Purpose:** Campaign notifications
- **Authentication:** Account SID + Auth Token
- **Message Format:** WhatsApp template messages

### Firebase Integration

#### Authentication
- **Methods:** Email/Password, Google, Facebook
- **Session Management:** Firebase Auth tokens
- **Token Refresh:** Automatic via Firebase SDK

#### Firestore Database
- **Collections:**
  - `users` - User profiles
  - `content` - Generated content
  - `campaigns` - Marketing campaigns
  - `brandKits` - Brand identity assets
  - `analytics` - Performance metrics

#### Security Rules
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    match /content/{contentId} {
      allow read, write: if request.auth.uid == resource.data.userId;
    }
    match /campaigns/{campaignId} {
      allow read, write: if request.auth.uid == resource.data.userId;
    }
  }
}
```

## Deployment Architecture

### Development Environment
```
Frontend: localhost:5173 (Vite dev server)
Express Backend: localhost:5005
Flask API: localhost:5004
ML Service: localhost:5007
```

### Production Environment (AWS)
```
Frontend: S3 + CloudFront CDN
Express Backend: EC2 instance with PM2
Flask Services: EC2 instances with Gunicorn
Database: Firebase (managed)
Load Balancer: AWS ALB
SSL/TLS: AWS Certificate Manager
```

### Environment Variables

**Frontend (.env):**
```
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
VITE_GEMINI_API_KEY=
VITE_API_URL=http://localhost:5005
VITE_PYTHON_API_URL=http://localhost:5004
```

**Backend (.env):**
```
PORT=5005
HUGGINGFACE_API_KEY=
MODELSLAB_API_KEY=
TWITTER_BEARER_TOKEN=
INSTAGRAM_API_KEY=
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_WHATSAPP_NUMBER=
FIREBASE_SERVICE_ACCOUNT=./firebase-service-account.json
```

## Security Architecture

### Authentication Flow
1. User submits credentials to Firebase Auth
2. Firebase returns JWT token
3. Frontend stores token in localStorage
4. All API requests include Authorization header
5. Backend verifies token with Firebase Admin SDK
6. Request proceeds if token valid

### API Security
- All endpoints require authentication (except /health)
- CORS configured for frontend origin only
- Rate limiting on all endpoints
- Input validation and sanitization
- SQL injection prevention (using Firestore)
- XSS prevention (React escaping)

### Data Security
- API keys stored in environment variables
- Firebase service account in gitignored file
- HTTPS enforced in production
- Firestore security rules enforce user isolation
- No sensitive data in client-side code

## Performance Optimization

### Frontend Optimization
- Code splitting with React.lazy()
- Image lazy loading
- Debounced search inputs
- Memoized expensive computations
- Optimistic UI updates

### Backend Optimization
- Response caching for analytics data
- Database query optimization
- Connection pooling
- Async/await for concurrent operations
- Compression middleware

### ML Optimization
- Model loaded once at startup
- Feature extraction optimized
- Batch prediction support
- Model versioning for A/B testing

## Monitoring & Logging

### Application Logging
- Winston for Node.js logging
- Python logging module for Flask
- Log levels: ERROR, WARN, INFO, DEBUG
- Structured JSON logs

### Performance Monitoring
- API response time tracking
- ML prediction latency monitoring
- Database query performance
- External API call tracking

### Error Tracking
- Try-catch blocks around all async operations
- Error responses with correlation IDs
- Failed API call retry logic
- User-friendly error messages

## Correctness Properties

### CP1: Authentication Integrity
**Property:** All protected routes SHALL require valid Firebase authentication token

**Verification:**
- Middleware checks token on every request
- Invalid tokens return 401 Unauthorized
- Expired tokens trigger re-authentication

### CP2: Data Isolation
**Property:** Users SHALL only access their own data

**Verification:**
- All database queries filter by userId
- Firestore security rules enforce isolation
- API endpoints verify ownership before operations

### CP3: ML Prediction Consistency
**Property:** Same input SHALL produce same prediction (deterministic)

**Verification:**
- Model uses fixed random_state
- Feature extraction is deterministic
- Scaler transformation is consistent

### CP4: Content Generation Quality
**Property:** Generated content SHALL meet platform requirements

**Verification:**
- Twitter: ≤ 280 characters
- Instagram: ≤ 2200 characters
- LinkedIn: ≤ 3000 characters
- Hashtags properly formatted with #

### CP5: API Rate Limit Compliance
**Property:** System SHALL respect all third-party API rate limits

**Verification:**
- Rate limit tracking per API
- Exponential backoff on 429 responses
- Request queuing when approaching limits

### CP6: Data Persistence
**Property:** All user-generated content SHALL be persisted to Firestore

**Verification:**
- Successful write confirmation before UI update
- Retry logic for failed writes
- Transaction support for multi-document updates

### CP7: Real-time Analytics Accuracy
**Property:** Analytics data SHALL reflect actual social media metrics

**Verification:**
- Direct API calls to Twitter/Instagram
- No data transformation that alters values
- Timestamp tracking for data freshness

## Testing Strategy

### Unit Testing
- Jest for React components
- Pytest for Python services
- Mock external API calls
- Test coverage > 70%

### Integration Testing
- API endpoint testing with Supertest
- Database integration tests
- ML pipeline end-to-end tests

### Performance Testing
- Load testing with Artillery
- ML prediction latency benchmarks
- Database query performance tests

### Security Testing
- Authentication bypass attempts
- SQL injection tests (N/A for Firestore)
- XSS vulnerability scanning
- API rate limit testing

## Future Enhancements

### Phase 2 Features
- Video content generation
- Multi-account management
- Advanced scheduling with optimal timing
- A/B testing for content variations
- Sentiment analysis dashboard

### Phase 3 Features
- Mobile app (React Native)
- Team collaboration features
- White-label solution
- Advanced ML models (deep learning)
- Real-time collaboration

### Scalability Roadmap
- Microservices architecture
- Kubernetes orchestration
- Redis caching layer
- Message queue (RabbitMQ/SQS)
- CDN for generated assets

## Conclusion

This design document provides a comprehensive technical blueprint for the OnTheFly platform. The architecture balances simplicity with scalability, leveraging managed services (Firebase) while maintaining flexibility through microservices. The ML-powered prediction system provides unique value, and the multi-platform integration enables centralized social media management.

