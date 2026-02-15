# On the Fly -  GenAI Content Creation & Marketing Platform

## Overview

OnTheFly is an intelligent marketing automation platform that helps small businesses and content creators generate, optimize, and manage social media content across multiple platforms using AI and machine learning.

## Key Features

- **Content Studio** - AI-powered content generation with platform-specific optimization for Instagram, Twitter/X, and LinkedIn. Includes real-time ML predictions for engagement performance.
- **Email Campaigns** - Automated email marketing with customizable templates and audience segmentation.
- **Brand Kit Generator** - Automated brand identity creation including logos, color palettes, typography recommendations, and brand guidelines.
- **Campaign Builder** - Multi-platform campaign management with scheduling, audience targeting, and performance tracking.
- **Analytics Dashboard** - Real-time analytics aggregation from connected social platforms with AI-generated insights.
- **SEO Audit** - Website analysis for search engine optimization with actionable recommendations.
- **Competitor Analysis** - Instagram competitor analysis with AI-powered insights using Gemini API.
- **WhatsApp Integration** - Automated engagement alerts via Twilio integration.
- **Event-Based Triggers** - Automated actions triggered by user behavior, time-based events, or engagement milestones.
- **Exit Intent Popup** - Smart popups that detect when users are about to leave and display targeted offers or CTAs.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND LAYER                                  │
│                     React 18 + Vite + Tailwind + DaisyUI                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            API GATEWAY LAYER                                 │
│                         Express.js REST API (:5005)                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                 ┌────────────────────┼────────────────────┐
                 ▼                    ▼                    ▼
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│  Content Service    │  │   Flask Server      │  │   ML Server         │
│  (Node.js)          │  │   (:5004)           │  │   (:5007)           │
│                     │  │                     │  │                     │
│  • Text Generation  │  │  • Twitter API v2   │  │  • Random Forest    │
│  • Image Generation │  │  • Instagram API    │  │  • Engagement       │
│  • Video Scripts    │  │  • Social Analytics │  │    Prediction       │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
         │                                                  │
         ▼                                                  ▼
┌─────────────────────┐                          ┌─────────────────────┐
│   AI/ML Services    │                          │   Data Storage      │
│                     │                          │                     │
│  • Llama 3.1-8B     │                          │  • Firebase         │
│    (HuggingFace)    │                          │    Firestore        │
│  • FLUX.1-schnell   │                          │  • SQLite           │
│    (Wavespeed)      │                          │    (Events)         │
│  • Gemini 2.5 Flash │                          │                     │
└─────────────────────┘                          └─────────────────────┘
```

## Tech Stack

### Frontend
- **Framework**: React 18 with Vite
- **Styling**: Tailwind CSS + DaisyUI
- **Routing**: React Router DOM
- **Charts**: Recharts, Chart.js
- **Icons**: React Icons, FontAwesome, Lucide

### Backend
- **Node.js Server**: Express.js (port 5005)
- **Python Server**: Flask (port 5004) - Twitter/Instagram APIs
- **ML Server**: Flask (port 5007) - Engagement predictions

### AI/ML Services
- **Text Generation**: HuggingFace Router (Llama-3.1-8B-Instruct)
- **Image Generation**: FLUX.1-schnell via Wavespeed API
- **Post Analysis**: Google Gemini 2.5 Flash
- **Engagement Prediction**: Custom Random Forest classifier (18 features)

### Database & Auth
- **Database**: Firebase Firestore
- **Authentication**: Firebase Auth
- **Event Storage**: SQLite

### External APIs
- **Twitter API v2**: Analytics and posting
- **RapidAPI Instagram**: Competitor analysis
- **Twilio**: WhatsApp notifications


## Built With

This project was developed using [Kiro IDE](https://kiro.dev) - an AI-powered development environment that helped with spec-driven development, code generation, and iterative feature building.

## ML Model

The engagement prediction model uses a Random Forest classifier trained on 18 features:
- Content metrics (word count, hashtag count, emoji count, etc.)
- Timing features (hour, day of week)

- Platform-specific optimizations
- Historical performance data

<img width="1024" height="1024" alt="logo2" src="https://github.com/user-attachments/assets/4b74be9f-e4bd-4d05-8d9c-1390f42484ca" />
