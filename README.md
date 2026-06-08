# EmoSound — AI-Powered Emotion-Based Music Discovery

A machine learning platform that detects emotions from text and voice input, then recommends music that matches your current mood using Spotify's audio feature API and transformer-based NLP.

> **Status**: Complete — v1.0.0 | Last updated: September 2025

---

## Overview

Most music platforms recommend based on listening history. EmoSound uses real-time emotion detection to understand how you're feeling *right now* and surfaces tracks that actually fit that state.

The core pipeline: user input (text or audio) → DistilRoBERTa emotion classification → Spotify audio feature matching → ranked recommendations with feedback loop.

---

## Features

**Emotion Detection**
- Text-based emotion analysis using DistilRoBERTa transformer
- Voice input via speech-to-text with emotional tone analysis
- Audio file upload (WAV, MP3, M4A) for offline recordings
- Confidence scoring on every prediction
- Supports 10 emotion classes: Happy, Sad, Angry, Excited, Calm, Anxious, Romantic, Energetic, Melancholic, Confident

**Music Recommendations**
- Spotify Web API integration with audio feature analysis
- Hybrid ranking: content-based (40%) + collaborative filtering (30%) + popularity (20%) + learned preferences (10%)
- Feature matching across valence, energy, danceability, acousticness, tempo
- Like/dislike feedback that updates the recommendation model per interaction

**User System**
- Auth with bcrypt password hashing and session management
- Spotify OAuth for personalized access
- Listening history, mood logs, and preference tracking
- Analytics dashboard with Plotly visualizations

---

## Tech Stack

| Layer | Tools |
|---|---|
| Frontend / Backend | Streamlit 1.28.0 |
| Language | Python 3.11+ |
| NLP Model | DistilRoBERTa (Hugging Face Transformers) |
| Speech Recognition | Google Speech API |
| Audio Analysis | Librosa, MFCC, Spectral Features |
| Music API | Spotify Web API (Spotipy) |
| Database | SQLite + SQLAlchemy ORM |
| Visualization | Plotly Express |
| Auth | Bcrypt, Python-Dotenv |

---

## ML Architecture

### Text Emotion Pipeline
User Input → Preprocessing → Tokenization → DistilRoBERTa → Emotion Class + Confidence Score

**Model specs:**
- Architecture: DistilRoBERTa (distilled RoBERTa)
- Parameters: 82M
- Layers: 6 transformer encoder layers, 12 attention heads
- Hidden size: 768
- Accuracy: ~89% on emotion classification
- Pre-trained on 160GB text, fine-tuned on emotion-labeled datasets

### Audio Pipeline
Audio Input → Speech-to-Text → Text Emotion Analysis
→ Feature Extraction (MFCCs, Spectral Centroid, ZCR, Tempo, Pitch)
→ Confidence Adjustment

### Recommendation Algorithm
Hybrid Score = 0.4 × content_similarity
+ 0.3 × collaborative_score
+ 0.2 × popularity_rank
+ 0.1 × learned_preference

Preference update per feedback:
new_preference = current_preference + (song_feature − current_preference) × learning_rate
Learning rate decays with interaction count to stabilize over time.

---

## Performance

| Metric | Value |
|---|---|
| Emotion Detection Accuracy | 89% |
| Average Response Time | 1.2s |
| Recommendation Relevance | 78% |
| Model Load Time | 3.5s |
| API Latency | 450ms |

---

## Installation

**Prerequisites:** Python 3.11+, pip, Spotify Developer account, 4GB RAM minimum

```bash
# Clone
git clone https://github.com/Aaryan-Lunis/EmoSound_Streamlit_Version.git
cd EmoSound_Streamlit_Version

# Virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Dependencies
pip install -r requirements.txt

# CPU-only PyTorch (optional, smaller download)
pip install torch --index-url https://download.pytorch.org/whl/cpu

# Initialize database
python database/init_db.py
```

First run will auto-download DistilRoBERTa (~500MB). To download manually:

```bash
python -c "
from transformers import AutoTokenizer, AutoModel
AutoTokenizer.from_pretrained('j-hartmann/emotion-english-distilroberta-base')
AutoModel.from_pretrained('j-hartmann/emotion-english-distilroberta-base')
"
```

---

## Configuration

Create a `.env` file in the root directory:

```env
DATABASE_URL=sqlite:///emosound.db

SPOTIFY_CLIENT_ID=your_client_id
SPOTIFY_CLIENT_SECRET=your_client_secret
SPOTIFY_REDIRECT_URI=http://localhost:8501/callback

SECRET_KEY=your_secret_key
JWT_SECRET_KEY=your_jwt_secret_key

DEBUG=True
LOG_LEVEL=INFO
```

Generate secure keys:
```python
import secrets
print(secrets.token_urlsafe(32))
```

Spotify setup: create an app at [developer.spotify.com](https://developer.spotify.com/dashboard), add `http://127.0.0.1:8501` as a redirect URI, copy your Client ID and Secret.

---

## Usage

```bash
streamlit run app.py
```

Opens at `http://localhost:8501`.

**Text detection:** Describe how you're feeling → Analyze → view emotion + confidence → browse recommendations.

**Audio upload:** Upload WAV/MP3/M4A (max 10MB) → AI transcribes and classifies → get matched tracks.

**Feedback:** Like/dislike any recommendation to train your personal model. Changes take effect immediately.

---

## Project Structure
emosound/
├── app.py                          # Entry point
├── requirements.txt
├── auth/
│   └── authentication.py           # Bcrypt auth logic
├── database/
│   ├── models.py                   # SQLAlchemy models
│   ├── database.py                 # Queries and operations
│   └── init_db.py
├── emotion/
│   ├── text_emotion.py             # DistilRoBERTa pipeline
│   ├── audio_emotion.py            # Audio processing
│   └── emotion_utils.py
├── api/
│   ├── spotify_api.py              # Spotify wrapper
│   └── spotify_ml_recommender.py   # Recommendation logic
├── ui/
│   ├── components.py
│   ├── pages.py
│   └── styles.py
└── utils/
└── helpers.py

---

## Database Schema
Users ──────── EmotionLogs ──────── Emotions
│
└─────────── UserSongHistory ──── Songs

- **Users**: credentials, Spotify OAuth tokens
- **Emotions**: 10 predefined classes with color codes
- **EmotionLogs**: full detection history with confidence scores and timestamps
- **Songs**: Spotify metadata cached for performance
- **UserSongHistory**: interaction log with like/dislike for model training

---

## Troubleshooting

**Spotify auth error** — verify Client ID/Secret in `.env`, confirm redirect URI is exactly `http://localhost:8501/callback`.

**Audio processing fails** — install FFmpeg:
```bash
# macOS
brew install ffmpeg
# Linux
sudo apt-get install ffmpeg
# Windows: download from ffmpeg.org
```

**Database errors** — reset with:
```bash
rm emosound.db && python database/init_db.py
```

**Duplicate widget key errors** — run `streamlit cache clear` and restart.

---

## Contributing

Open issues for bugs or feature requests. PRs welcome — follow PEP 8, include type hints, write tests for new functionality.

```bash
git checkout -b feature/your-feature-name
# make changes
git push origin feature/your-feature-name
```

---

## Citation

```bibtex
@software{emosound2025,
  title   = {EmoSound: AI-Powered Emotion-Based Music Discovery},
  author  = {Manav Bhuta and Aaryan Lunis},
  year    = {2024},
  url     = {https://github.com/Aaryan-Lunis/EmoSound_Streamlit_Version},
  version = {1.0.0}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
