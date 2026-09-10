# Music Recommendation System with K-Nearest Neighbors

A web-based music recommendation system that uses the K-Nearest Neighbors (KNN) algorithm to suggest similar songs based on audio features. This project demonstrates how machine learning can be used to build a music recommendation engine.

## Features

- Search for any song in the database
- Get song recommendations based on:
  - Similar tracks
  - Custom audio feature profiles (Optional; requires student implementation)
- Play previews of recommended songs (length may vary)
- Adjust algorithm settings (distance metrics, number of neighbors)
- Mobile-responsive web interface

## Quick Start

### Prerequisites

- Python 3.11 recommended (3.9, 3.10 also work)
- pip (Python package manager)

### Installation

2. **Create and activate a virtual environment**
   ```bash
   # On macOS/Linux (recommended)
   # Ensure Python 3.11 is installed (e.g., via Homebrew: brew install python@3.11)
   python3.11 -m venv .venv-music
   source .venv-music/bin/activate
   
   # On Windows
   # py -3.11 -m venv .venv-music
   # .venv-music\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Get a YouTube Data API key** (see detailed steps below)

5. **Configure the application** (env vars preferred)
   - The app reads config from environment variables in `utils/config.py`.
   - Create a `.env` file at the project root (same folder as `app.py`) based on `.env.example`:
     ```bash
     cp .env.example .env
     # Edit .env and set your values
     ```
   - Or export directly in your shell before running:
     - macOS/Linux
       ```bash
       export YOUTUBE_API_KEY="your_api_key"
       export SECRET_KEY="dev-secret"
       ```
     - Windows (PowerShell)
       ```powershell
       $env:YOUTUBE_API_KEY="your_api_key"
       $env:SECRET_KEY="dev-secret"
       ```

### Running the Application

```bash
# Start the development server
python app.py

# The application will be available at:
# http://localhost:5002
```

Note:
- If you change values in `.env` (e.g., update `YOUTUBE_API_KEY`), you must restart the server to pick up changes.
- Start the app from the project root (same directory as `app.py`) so `utils/config.py` can load `.env`.

### How to get a YouTube API key

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project (or select an existing one).
3. Go to "APIs & Services" → "Library" → enable "YouTube Data API v3".
4. Go to "APIs & Services" → "Credentials" → "Create credentials" → "API key".
5. (Recommended) Click the key to set restrictions:
   - Application restrictions: None (for local dev) or your machine's IP.
   - API restrictions: Restrict to "YouTube Data API v3".
6. Put the key in `.env` in the project root:
   ```env
   YOUTUBE_API_KEY=YOUR_REAL_KEY
   SECRET_KEY=dev-secret
   ```
7. Restart the Flask server.

## Project Structure

```
.
├── app.py                     # Main Flask application
├── utils/
│   ├── api_helpers.py         # Helper functions for API endpoints
│   ├── knn_recommender.py     # Core KNN recommendation algorithm
│   └── config.py              # Application configuration (loads .env)
├── requirements.txt      # Python dependencies
├── static/               # Frontend assets
│   └── knn-music-recommender.html  # Main HTML/JS/CSS
└── data/                 # Data files (not included in repo)
    ├── mergedFile.csv    # Main dataset with song information
    └── item_profile.csv  # Audio feature profiles for each song
```

## How It Works

1. **Data Processing**
   - The system loads song data from CSV files
   - Audio features are normalized for comparison
   - Features include: danceability, energy, key, loudness, etc.

2. **Recommendation Engine**
   - Uses K-Nearest Neighbors algorithm
   - Supports multiple distance metrics:
     - Cosine similarity
     - Euclidean distance
   - (Optional/Future work) Feature weighting can be added if desired

3. **Frontend**
   - Built with vanilla HTML, CSS, and JavaScript
   - Communicates with the Flask backend via REST API
   - Uses YouTube for audio previews

## OPTIONAL: What students need to implement (brief)

To enable the "Find Songs With This Vibe" feature (profile-based recommendations), complete these minimal tasks:

- __Implement profile vector recommendations__ in `utils/student_adapter.py` within `KNNRecommender`.
  - Add a method that takes a feature vector and returns nearest neighbors, optionally filtered by selected features.
  - Skeleton:
    ```python
    class KNNRecommender:
        # ... existing code ...
        def recommend_from_vector(self, query_vector, n_recommendations=None,
                                  distance_metric='cosine', selected_features=None):
            # 1) choose feature indices (selected_features or all)
            # 2) slice self.features_matrix and query_vector to those indices
            # 3) compute distances (cosine or euclidean) to every row
            # 4) take top-k indices, map to track ids, build DataFrame with 'distance'
            # 5) return sorted DataFrame by 'distance'
            pass
    ```

- __Wire profile route to student recommender__ in `utils/api_helpers.py`.
  - In `get_recommendations()`, handle `way == 'fromProfile'` when `STUDENT_IMPLEMENTATION_AVAILABLE`:
    ```python
    if kwargs.get('way') == 'fromProfile' and STUDENT_IMPLEMENTATION_AVAILABLE:
        recs_df = student_recommender.recommend_from_vector(
            query_vector=kwargs.get('query_vector'),
            n_recommendations=kwargs.get('k', 10),
            distance_metric=kwargs.get('distance_metric', 'cosine').lower(),
            selected_features=kwargs.get('selected_features')
        )
        # convert rows to API response using get_information_from_id()
        # and include an optional similarity_score derived from distance
        return { 'success': True, 'data': {/* ... */} }
    ```

- __Feature columns to use__ (consistent across code):
  - `['energy','danceability','acousticness','valence','tempo','instrumentalness','loudness','liveness','speechiness']`

- __Tips__:
  - Ensure `fit()` stores `self.feature_columns`, `self.features_matrix`, and an `id` index map.
  - When computing cosine distance, guard against zero norms.
  - You can start without scaling; add a simple standard scaler later if desired.

Once these are in place, the POST `/api/recommend/profile` path will return results and the UI will render them.

---

## Common Issues & Solutions

### 1. Port Already in Use
```
Address already in use
Port 5002 is in use by another program
```
**Solution:**
```bash
# Find and kill the process using port 5002
lsof -i :5002
kill -9 <PID>

# Or use a different port by modifying app.py
if __name__ == "__main__":
    app.run(port=5003)  # Change to an available port
```

### 2. YouTube API Quota Exceeded
```
API key not valid. Please pass a valid API key.
```
**Solution:**
- Check your YouTube Data API quota in Google Cloud Console
- Consider enabling billing for higher quotas
- Cache responses to reduce API calls

### 3. Audio Playback Issues
```
ERROR: Signature extraction failed
```
**Solution:**
```bash
# Update yt-dlp to the latest version
pip install --upgrade yt-dlp
```

Additional note:
- Some videos only expose HLS streams (`.m3u8`). Chrome does not natively support HLS; you may see “no supported source” errors. Try another track or a browser with HLS support, or modify the backend to avoid HLS fallbacks.

### Verifying the Backend Quickly

```bash
# Check featured endpoint is working
curl -i http://127.0.0.1:5002/api/featured

# Example: get audio URL for a known track id
curl -i "http://127.0.0.1:5002/api/getmp3url?songid=<TRACK_ID>"
```

### 4. Missing Dependencies
```
ModuleNotFoundError: No module named 'flask'
```
**Solution:**
```bash
# Make sure you're in the virtual environment
source .venv-music/bin/activate  # or .venv-music\Scripts\activate on Windows

# Then install requirements
pip install -r requirements.txt
```

## Learning Resources

- [K-Nearest Neighbors Algorithm](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm)
- [Spotify Audio Features](https://developer.spotify.com/documentation/web-api/reference/get-audio-features)
- [Flask Web Framework](https://flask.palletsprojects.com/)
- [YouTube Data API](https://developers.google.com/youtube/v3)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Dataset: [Spotify Dataset 1921-2020, 600k+ Tracks](https://www.kaggle.com/yamaerenay/spotify-dataset-19212020-600k-tracks)
- Icons: [Material Design Icons](https://material.io/resources/icons/)
- Color Scheme: Inspired by Spotify's design language
- Inspiration: The project draws conceptual inspiration from [RUMusic](https://github.com/vraj152/RUMusic), while our implementation and website have significantly diverged.
