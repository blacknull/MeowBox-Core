# MeowBox-Core — Workspace Instructions for AI Agents

**MeowBox-Core** is an embedded music server for ESP32 devices, featuring a Go HTTP backend with user authentication, playlist management, device binding, and a React+Vite frontend.

## Quick Reference

| Aspect | Details |
|--------|---------|
| **Languages** | Go 1.25.0 (backend), JavaScript/React 18.2 (frontend), JSON configs |
| **Backend Structure** | Go package `main`, handlers in `*_go` files, stores in `files/` (JSON) |
| **Frontend** | React + Vite + Tailwind CSS, compiled to static files served by backend |
| **Entry Points** | Backend: `main.go` | Frontend: `frontend/index.tsx` or `frontend/src/main.jsx` |
| **Environment** | `.env` file with `PORT` setting (default 2233); uses `godotenv` |
| **Run Commands** | Backend: `go run .` | Frontend: `npm run dev` (Vite) | Full: `start.sh` or `start.bat` |

---

## Backend (Go)

### Architecture
- **HTTP Server**: Port 2233 (configurable via `PORT` env var)
- **Request Handling**: Standard `http.HandleFunc` pattern
- **Authentication**: Token-based middleware (`AuthMiddleware`)
- **Data Storage**: JSON files in `files/` directory (`users.json`, `user_playlists.json`, `playlists.json`)

### Key Modules
- `main.go` — Server setup, route registration, signal handling
- `api.go` — Music search/streaming API (`/stream_pcm`, `/api/search`)
- `user.go` — Authentication handlers (register, login, logout)
- `playlist.go` — User playlist management (create, add songs, delete)
- `device.go` — ESP32 device binding system (generate codes, verify tokens)
- `search.go` — Music search logic
- `struct.go` — Data structures (User, MusicItem, UserPlaylist, Device)
- `helper.go` — Utility functions
- `httperr.go` — HTTP error responses
- `file.go` — JSON file I/O operations
- `index.go` — Web UI serving

### Common Patterns
1. **Handlers** follow pattern: `func Handle**(w http.ResponseWriter, r *http.Request)`
2. **Error Handling**: Custom `WriteHTTPError()` function for consistent JSON responses
3. **JSON Encoding**: Use `json.NewEncoder(w).Encode(data)`
4. **Authentication**: Wrap protected endpoints with `AuthMiddleware(handler)`
5. **Configuration**: Use environment variables via `os.Getenv()`

### Dependencies
- `github.com/joho/godotenv` — Load `.env` file
- `golang.org/x/crypto` — Password hashing

### Build & Run
```bash
# Install dependencies
go mod tidy

# Run server
go run .

# Or use shell script
./start.sh  # Linux/macOS
start.bat   # Windows
```

---

## Frontend (React + Vite)

### Architecture
- **Framework**: React 18.2 with functional components
- **Build Tool**: Vite (fast HMR in dev, optimized production build)
- **Styling**: Tailwind CSS + PostCSS
- **Icons**: Lucide React
- **Entry**: Frontend source (likely under `frontend/src/`)

### Development
```bash
cd frontend
npm install
npm run dev      # Start development server (http://localhost:5173)
npm run build    # Build for production
npm run preview  # Preview production build
```

### Output
- Production build outputs to `frontend/dist/`
- Backend serves these static files (check `theme/` directory for HTML roots)
- Frontend communicates with backend via HTTP API (same server)

### Environment
- **API Base**: Backend runs on same server (configurable via env/build config)
- **Build Output**: Vite generates assets with hash-based naming for cache busting

---

## API Endpoints (Backend)

### Core Streaming
- **GET `/stream_pcm`** — Stream PCM audio (query: `song`, `singer`)
- **GET `/stream_live`** — Real-time live stream transcoding
- **GET `/api/search`** — Search music

### Authentication
- **POST `/api/auth/register`** — Register new user
- **POST `/api/auth/login`** — Login (returns token)
- **POST `/api/auth/logout`** — Logout
- **GET `/api/auth/me`** — Get current user (requires auth token)

### Playlists
- **GET `/api/user/playlists`** — List user playlists (requires auth)
- **POST `/api/user/playlists/create`** — Create playlist (requires auth)
- **POST `/api/user/playlists/add-song`** — Add song to playlist (requires auth)
- **POST `/api/user/playlists/remove-song`** — Remove song from playlist (requires auth)
- **POST `/api/user/playlists/delete`** — Delete playlist (requires auth)

### Device Binding (ESP32)
- **POST `/api/device/generate-code`** — Generate binding code (requires auth)
- **POST `/api/device/bind-direct`** — Web-based device binding
- **GET `/api/device/list`** — List user's devices
- **POST `/api/device/unbind`** — Unbind device
- **POST `/api/esp32/bind`** — ESP32 registers with binding code
- **POST `/api/esp32/verify`** — Verify device token
- **POST `/api/esp32/sync`** — Sync device token via MAC address

---

## Configuration

### Environment Variables (`.env` file)
```env
PORT=2233              # Server port (default: 2233)
# Add other config as needed
```

### Music Sources (`sources.json`)
JSON array of music items:
```json
[
  {
    "title": "Song Name",
    "artist": "Artist Name",
    "audio_url": "https://example.com/song.mp3",
    "audio_full_url": "https://example.com/full.mp3",
    "m3u8_url": "https://example.com/playlist.m3u8",
    "lyric_url": "https://example.com/lyrics.lrc",
    "cover_url": "https://example.com/cover.jpg",
    "duration": 240
  }
]
```

### Data Files (JSON in `files/`)
- `users.json` — User accounts
- `user_playlists.json` — User-created playlists
- `playlists.json` — System playlists (if any)

---

## Development Workflow

### Setting Up
1. Clone repository
2. Copy `.env.example` to `.env` (if needed) and configure `PORT`
3. Backend: `go mod tidy` then `go run .`
4. Frontend: `cd frontend && npm install && npm run dev`

### Making Changes
- **Backend**: Modify Go files, restart server with `go run .`
- **Frontend**: Changes trigger HMR automatically in dev mode
- **API Changes**: Update endpoint handlers in appropriate `*_go` file, document in this file

### Testing
- Check CI/CD workflow in `.github/workflows/test-and-build.yml` for automated test commands

### Building
- **Backend**: Go binary created during build
- **Frontend**: `npm run build` generates optimized assets in `frontend/dist/`

---

## Common Development Tasks

### Adding a New API Endpoint
1. Create handler function in appropriate module (e.g., `api.go` for music, `device.go` for devices)
2. Register route in `main.go` with `http.HandleFunc()`
3. Add middleware if authentication required: `AuthMiddleware(handler)`
4. Document endpoint in this file's API section

### Modifying User Data Structure
1. Update struct in `struct.go`
2. Update JSON file I/O in `file.go` if needed
3. Update related handlers
4. Handle backward compatibility in data loading

### Adding Frontend Features
1. Create React components in `frontend/src/components/`
2. Use Tailwind CSS for styling
3. Call backend API endpoints via fetch/axios
4. Test with `npm run dev`

### Configuring Music Sources
1. Edit `sources.json` with music metadata
2. Ensure URLs are accessible and in correct format
3. Restart backend to reload sources

---

## Conventions & Patterns

### Naming
- **Go Files**: `action_type.go` (e.g., `user.go`, `api.go`, `device.go`)
- **Go Functions**: `HandleAction()` or `Action()` depending on context
- **Go Structs**: `PascalCase` (e.g., `MusicItem`, `UserPlaylist`)
- **JSON Keys**: `snake_case` (configured via struct tags)
- **Environment Variables**: `SCREAMING_SNAKE_CASE`

### Error Handling
- Use custom `WriteHTTPError()` for consistent JSON error responses
- Include error context in logs with `TAG` prefix
- Log to stdout with `fmt.Printf("[Level] message")`

### Authentication
- Tokens are passed in request headers or body
- Protect endpoints with `AuthMiddleware()`
- Token validation in handler before processing

### Backward Compatibility
- Legacy `/api/favorite/*` endpoints maintained for compatibility
- URL redirects for common typos (e.g., `/device-bin` → `/device-bind`)

---

## Documentation & Resources

- **Chinese Guides**: See `快速开始.md`, `本地部署指南.md`, `新功能说明.md` in project root
- **Device Binding**: Refer to `DEVICE_BINDING_GUIDE.md`, `WEB_DEVICE_BIND_GUIDE.md`
- **User System**: See `USER_SYSTEM_README.md`
- **API Source**: Check handler functions in respective modules
- **Frontend Source**: React components in `frontend/src/`

---

## Support & Contact
QQ交流群: 865754861
