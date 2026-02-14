# 🎵 Spleeter Audio Separation API

FastAPI-based REST API for separating vocals from music using AI (Spleeter by Deezer).

## Features

- 🎤 **Vocal Isolation**: Extract vocals from any audio track
- 🎸 **Instrumental Extraction**: Get music without vocals
- ⚡ **Fast Processing**: Optimized for quick separation
- 🌐 **CORS Enabled**: Works with Chrome extensions
- 📦 **Docker Ready**: Easy deployment with containers

## API Endpoints

### 1. Health Check
```bash
GET /
GET /health
```

### 2. Separate Audio (Single Output)
```bash
POST /api/separate
```

**Parameters:**
- `file`: Audio file (mp3, wav, ogg, flac, m4a, webm)
- `return_vocals`: boolean (default: true)
  - `true` = returns vocals only
  - `false` = returns instrumental only

**Example:**
```bash
curl -X POST "http://localhost:8000/api/separate?return_vocals=true" \
  -F "file=@song.mp3" \
  --output vocals.wav
```

### 3. Separate Audio (Both Outputs)
```bash
POST /api/separate-both
```

Returns JSON with download URLs for both vocals and instrumental.

**Example:**
```bash
curl -X POST "http://localhost:8000/api/separate-both" \
  -F "file=@song.mp3"
```

**Response:**
```json
{
  "request_id": "abc-123-def",
  "status": "success",
  "files": {
    "vocals": "/api/download/abc-123-def/vocals",
    "instrumental": "/api/download/abc-123-def/accompaniment"
  }
}
```

### 4. Download Separated Track
```bash
GET /api/download/{request_id}/{track_type}
```

- `track_type`: `vocals` or `accompaniment`

### 5. Cleanup Files
```bash
DELETE /api/cleanup/{request_id}
```

## Installation & Deployment

### Option 1: Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Install ffmpeg (required)
# Ubuntu/Debian:
sudo apt-get install ffmpeg

# macOS:
brew install ffmpeg

# Run the server
python main.py
# Or with uvicorn:
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Access API at: `http://localhost:8000`

### Option 2: Docker

```bash
# Build image
docker build -t spleeter-api .

# Run container
docker run -d -p 8000:8000 --name spleeter-api spleeter-api
```

### Option 3: Deploy to Railway

1. Create account at [Railway.app](https://railway.app)
2. Click "New Project" → "Deploy from GitHub repo"
3. Connect your repository
4. Railway will auto-detect and deploy
5. Get your deployment URL (e.g., `https://your-app.railway.app`)

### Option 4: Deploy to Render

1. Create account at [Render.com](https://render.com)
2. New → Web Service
3. Connect your GitHub repo
4. Settings:
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `uvicorn main:app --host 0.0.0.0 --port $PORT`
5. Deploy!

### Option 5: Deploy to Heroku

```bash
# Install Heroku CLI
# Create Procfile
echo "web: uvicorn main:app --host 0.0.0.0 --port \$PORT" > Procfile

# Add buildpacks
heroku buildpacks:add --index 1 heroku/python
heroku buildpacks:add --index 2 https://github.com/jonathanong/heroku-buildpack-ffmpeg-latest.git

# Deploy
git init
heroku create your-app-name
git add .
git commit -m "Initial commit"
git push heroku main
```

## Chrome Extension Integration

Update your extension to call this API:

```javascript
// In content.js or background.js
async function separateAudio(audioBlob) {
    const formData = new FormData();
    formData.append('file', audioBlob, 'audio.mp3');
    
    const response = await fetch('YOUR_API_URL/api/separate?return_vocals=true', {
        method: 'POST',
        body: formData
    });
    
    const vocalsBlob = await response.blob();
    return vocalsBlob;
}
```

## Configuration

### Environment Variables

Create `.env` file (optional):

```env
PORT=8000
HOST=0.0.0.0
ALLOWED_ORIGINS=*
```

### CORS Settings

Update `main.py` to restrict origins in production:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["chrome-extension://your-extension-id"],  # Restrict in production
    # ...
)
```

## System Requirements

- Python 3.8+ (3.10 recommended)
- FFmpeg
- 2GB+ RAM
- CPU: Multi-core recommended (separation is CPU-intensive)

## Supported Audio Formats

- MP3
- WAV
- OGG
- FLAC
- M4A
- WebM

## Performance Notes

- First request downloads Spleeter model (~30MB) - takes extra time
- Separation takes ~10-30 seconds per song depending on length and CPU
- Consider adding queue system for high traffic

## Troubleshooting

### Error: "ffmpeg not found"
Install ffmpeg:
```bash
# Ubuntu/Debian
sudo apt-get install ffmpeg

# macOS
brew install ffmpeg
```

### Error: "TensorFlow not found"
```bash
pip install tensorflow==2.13.0
```

### Out of Memory
Increase server RAM or process shorter audio clips.

## License

MIT License

## Credits

- **Spleeter** by Deezer Research
- **FastAPI** by Sebastián Ramírez

---

**Ready to use!** 🚀 Deploy and share your API URL with the Chrome extension.
