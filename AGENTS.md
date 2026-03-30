# MeowEmbeddedMusicServer (喵波音律) - Agent Guide

## Project Overview

MeowEmbeddedMusicServer (喵波音律 - "Meow Wave Music") is a music streaming service specifically designed for embedded devices (especially ESP32). It provides a complete user system, personal playlist management, and real-time music streaming capabilities.

**Project Name**: MeowEmbeddedMusicServer  
**Version**: 2.0  
**Primary Language**: Chinese (documentation and comments)  
**Repository**: https://github.com/OmniX-Space/MeowBox-Core

### Key Features
- 🎵 Music streaming for embedded devices (ESP32)
- 🔐 Complete user system (registration, login, token-based auth)
- 📱 Personal playlist management ("My Favorites", custom playlists)
- 🔗 ESP32 device binding and verification system
- 🔍 Online music search from multiple sources (Kuwo, NetEase, Migu, Baidu)
- ⚡ Real-time audio transcoding and streaming (FFmpeg)
- 🌐 Web interface (React + TailwindCSS)
- 💾 Local music cache and file storage

---

## Technology Stack

### Backend
- **Language**: Go 1.25+
- **HTTP Server**: Standard `net/http` package
- **Authentication**: bcrypt for passwords, token-based sessions
- **External Dependencies**:
  - `github.com/joho/godotenv` - Environment variable loading
  - `golang.org/x/crypto/bcrypt` - Password hashing

### Frontend
- **Framework**: React 18
- **Styling**: TailwindCSS
- **Build Tool**: Vite
- **Icons**: lucide-react

### Optional External Tools
- **FFmpeg**: Audio transcoding and compression (required for full functionality)
- **FFprobe**: Audio duration detection

### Data Storage
All data is stored in JSON files (no database required):
- `./files/users.json` - User accounts
- `./files/user_playlists.json` - User playlists
- `./files/playlists.json` - Legacy global playlists
- `./devices.json` - Device bindings
- `./cache/` - Cached music files

---

## Project Structure

```
MeowEmbeddedMusicServer/
├── main.go                 # Entry point, HTTP server setup
├── api.go                  # Music streaming API (/stream_pcm, /stream_live)
├── user.go                 # User authentication and management
├── playlist.go             # Playlist management (legacy + user playlists)
├── device.go               # ESP32 device binding and token verification
├── search.go               # Music search functionality
├── helper.go               # Utility functions, FFmpeg integration
├── yuafengfreeapi.go       # External music API integration (枫雨API)
├── file.go                 # Static file serving
├── index.go                # Web interface handler, default page renderer
├── httperr.go              # Error handling (404 page)
├── struct.go               # Data structures (structs)
├── go.mod                  # Go module definition
├── go.sum                  # Go dependency checksums
├── start.bat               # Windows startup script
├── start.sh                # Linux/macOS startup script
├── sources.json            # Local music source configuration
├── sources.json.example    # Example sources configuration
├── theme/                  # Frontend HTML files (pre-built)
│   ├── full-app.html      # Main React application
│   ├── music-app.html     # Music app interface
│   ├── index.html         # Classic interface
│   ├── device-bind.html   # Device binding page
│   └── ...
├── frontend/               # Frontend source (React)
│   └── package.json
├── files/                  # Runtime data storage (auto-created)
│   ├── users.json
│   ├── user_playlists.json
│   └── playlists.json
└── cache/                  # Music cache (auto-created)
```

---

## Build and Run Commands

### Prerequisites
- Go 1.19 or higher
- (Optional) FFmpeg for audio transcoding

### Quick Start

**Windows:**
```bash
# Double-click or run in terminal
start.bat
```

**Linux/macOS:**
```bash
# Add execute permission and run
chmod +x start.sh
./start.sh
```

### Manual Build

```bash
# Install dependencies
go mod tidy

# Run in development mode
go run .

# Build executable (Windows)
go build -o meow-music-server.exe

# Build executable (Linux/macOS)
go build -o meow-music-server

# Build with optimization
go build -ldflags="-s -w" -o meow-music-server
```

### Access Points
After starting the server:
- **Main App**: http://localhost:2233/app
- **Classic Interface**: http://localhost:2233/
- **Device Binding Page**: http://localhost:2233/device-bind

---

## Code Organization and Module Divisions

### Core Modules

1. **main.go** - HTTP server setup and route registration
   - Initializes UserStore, PlaylistManager, DeviceManager
   - Registers all HTTP handlers
   - Handles graceful shutdown

2. **user.go** - User authentication system
   - `UserStore` singleton with mutex-protected access
   - Password hashing with bcrypt
   - Session token generation and validation
   - Support for multiple auth methods (Bearer, Cookie, X-Device-Token)

3. **playlist.go** - Playlist management
   - Legacy playlist support (backward compatible)
   - User-specific playlist system
   - "我喜欢" (Favorites) playlist auto-creation
   - CRUD operations for playlists and songs

4. **device.go** - ESP32 device management
   - `DeviceManager` singleton
   - 6-digit binding code generation (5-minute expiry)
   - Device token verification
   - MAC address-based device tracking

5. **api.go** - Music streaming endpoints
   - `/stream_pcm` - JSON metadata API
   - `/stream_live` - Real-time audio streaming with transcoding

6. **search.go** - Music search
   - Multi-source search (local, API)
   - Source priority: Kuwo > NetEase > Migu > Baidu

7. **yuafengfreeapi.go** - External API integration
   - 枫雨API (Yuafeng API) integration
   - Multiple API host fallback
   - Async background processing for downloads
   - Lyrics fetching from backup sources

8. **helper.go** - Utility functions
   - FFmpeg transcoding functions
   - File download utilities
   - Cache management
   - Duration detection with FFprobe

9. **file.go** - File serving
   - Static file handler
   - Remote URL proxying (`/url/` prefix)
   - Live stream transcoding when files not ready
   - Content-Type detection

10. **index.go** - Web interface
    - Route-based page serving
    - Default HTML page generation
    - Internationalization support (Chinese/English)

11. **struct.go** - Data structures
    - `MusicItem` - Music metadata
    - `User` - User account
    - `UserPlaylist` - User playlist
    - `LoginRequest`/`RegisterRequest` - Auth requests

---

## API Endpoints

### Music Streaming
- `GET /stream_pcm?song=<name>&singer=<name>` - Get music metadata (JSON)
- `GET /stream_live?song=<name>&singer=<name>` - Real-time streaming audio

### Search
- `GET /api/search?query=<keyword>` - Search music from all sources

### Authentication
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `GET /api/auth/me` - Get current user info

### User Playlists (Requires Authentication)
- `GET /api/user/playlists` - Get all user playlists
- `POST /api/user/playlists/create` - Create new playlist
- `POST /api/user/playlists/add-song?playlist_id=<id>` - Add song to playlist
- `DELETE /api/user/playlists/remove-song?playlist_id=<id>&title=<t>&artist=<a>` - Remove song
- `DELETE /api/user/playlists/delete?playlist_id=<id>` - Delete playlist

### Legacy Favorites (Backward Compatible)
- `POST /api/favorite/add` - Add to favorites
- `POST /api/favorite/remove` - Remove from favorites
- `GET /api/favorite/list` - Get favorites list
- `GET /api/favorite/check?title=<t>&artist=<a>` - Check if favorite

### Device Management (ESP32)
- `POST /api/device/generate-code` - Generate binding code (auth required)
- `POST /api/device/bind-direct` - Direct device binding (auth required)
- `GET /api/device/list` - List user devices (auth required)
- `POST /api/device/unbind` - Unbind device (auth required)
- `POST /api/esp32/bind` - ESP32 bind with code
- `GET /api/esp32/verify` - Verify device token
- `POST /api/esp32/sync` - Sync device token by MAC

---

## Code Style Guidelines

### Go Code
- Use standard Go formatting (`gofmt`)
- Follow Go naming conventions:
  - `CamelCase` for exported functions/structs
  - `camelCase` for unexported functions/variables
- Comments in Chinese for project-specific logic
- Log messages with `[Info]`, `[Error]`, `[Warning]`, `[API]`, `[Web Access]` prefixes

### Key Patterns
1. **Singleton Pattern**: Used for `UserStore`, `PlaylistManager`, `DeviceManager`
2. **Mutex Protection**: All data stores use `sync.RWMutex` for concurrent access
3. **File-based Storage**: JSON files for persistence, loaded on init, saved after modifications
4. **Async Processing**: Music downloading/transcoding runs in goroutines

### Logging Conventions
```go
// Info logging
fmt.Printf("[Info] Starting music server at port %s\n", port)

// Error logging
fmt.Printf("[Error] Failed to marshal devices: %v\n", err)

// API logging
fmt.Printf("[API] Register request received from %s\n", r.RemoteAddr)

// Web access logging
fmt.Printf("[Web Access] Handling request for %s\n", r.URL.Path)
```

---

## Testing

### Manual Testing Checklist
After making changes, verify:
1. Server starts without errors: `go run .`
2. Can access http://localhost:2233/app
3. User registration works
4. User login works
5. Music search returns results
6. Can add songs to favorites
7. Can create custom playlists
8. Device binding flow works (if ESP32 related)

### No Automated Tests
This project currently does not have automated unit tests. All testing is manual through the web interface or API clients.

---

## Security Considerations

### Password Security
- Passwords are hashed with bcrypt before storage
- Never log or expose passwords

### Authentication
- Session tokens are 32-byte random hex strings
- Tokens can be passed via:
  - `Authorization: Bearer <token>` header
  - `X-Device-Token` header (for ESP32 devices)
  - `session_token` cookie (for web sessions)

### Data Protection
- User passwords never exposed in API responses (JSON tag: `json:"-"`)
- File path traversal is prevented in fileHandler
- Token verification required for all playlist modification endpoints

### Recommended for Production
1. Change default port (2233) in `.env`
2. Use HTTPS in production
3. Implement rate limiting for API endpoints
4. Regular backups of `files/` directory

---

## Configuration

### Environment Variables (Optional .env file)
```env
PORT=2233
WEBSITE_NAME_CN=我的音乐服务器
WEBSITE_NAME_EN=My Music Server
WEBSITE_URL=http://localhost:2233
```

### sources.json Format
```json
[
    {
        "title": "Song Name",
        "artist": "Artist Name",
        "audio_url": "https://example.com/audio.mp3",
        "audio_full_url": "https://example.com/audio_full.mp3",
        "m3u8_url": "",
        "lyric_url": "https://example.com/lyric.lrc",
        "cover_url": "https://example.com/cover.jpg",
        "duration": 180
    }
]
```

---

## Deployment

### Linux systemd Service
Create `/etc/systemd/system/meow-music.service`:
```ini
[Unit]
Description=Meow Music Server
After=network.target

[Service]
Type=simple
User=your_username
WorkingDirectory=/path/to/MeowEmbeddedMusicServer
ExecStart=/path/to/MeowEmbeddedMusicServer/meow-music-server
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable meow-music
sudo systemctl start meow-music
```

### Data Backup
Regularly backup the `files/` directory:
```bash
# Windows
xcopy /E /I files files_backup

# Linux/macOS
cp -r files files_backup
```

---

## Troubleshooting

### Common Issues

**Port already in use:**
```bash
# Windows
netstat -ano | findstr :2233
taskkill /PID <PID> /F

# Linux/macOS
lsof -i :2233
kill -9 <PID>
```

**Go dependency download fails (China users):**
```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

**FFmpeg not found:**
Install FFmpeg and ensure it's in PATH. The server can run without FFmpeg but audio transcoding features won't work.

---

## Documentation References

- `README.md` / `README_zh-CN.md` - Project overview
- `快速开始.md` - Quick start guide (3-minute deployment)
- `本地部署指南.md` - Detailed deployment guide
- `USER_SYSTEM_README.md` - User system API documentation
- `新功能说明.md` - New features overview
- `DEVICE_BINDING_GUIDE.md` - Device binding guide
- `WEB_DEVICE_BIND_GUIDE.md` - Web device binding guide
- `部署歌单功能指南.md` - Playlist deployment guide

---

## Community

- **QQ Group**: 865754861 (喵波音律-音乐家园)
- **GitHub**: https://github.com/OmniX-Space/MeowBox-Core

---

*This guide is for AI coding agents working on the MeowEmbeddedMusicServer project. For human contributors, see README.md files.*
