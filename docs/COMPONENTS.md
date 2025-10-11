# Component Documentation

This document provides detailed information about each component in the Gemini 2.0 Playground.

## Table of Contents

1. [Audio Components](#audio-components)
2. [Video Components](#video-components)
3. [Core Components](#core-components)
4. [Tool Components](#tool-components)
5. [Utility Components](#utility-components)
6. [Audio Worklets](#audio-worklets)

---

## Audio Components

### AudioRecorder

**File:** `src/static/js/audio/audio-recorder.js`

**Purpose:** Captures audio from the user's microphone and processes it into PCM16 format for transmission.

**Key Features:**
- Configurable sample rate (default: 16000 Hz)
- Real-time audio processing using Web Audio API
- Audio worklet-based processing for low latency
- Base64 encoding for network transmission

**Technical Details:**
- Uses `getUserMedia()` API for microphone access
- Employs `AudioWorkletNode` for efficient processing
- Converts Float32 audio to Int16 PCM format
- Buffers audio data before sending to reduce overhead

**Configuration:**
```javascript
const recorder = new AudioRecorder(16000); // 16kHz sample rate
```

**State Management:**
- `isRecording`: Boolean flag for recording state
- `stream`: MediaStream from microphone
- `audioContext`: Web Audio API context
- `processor`: AudioWorkletNode for processing

**Error Handling:**
- Throws `ApplicationError` with specific error codes
- Handles microphone permission denial
- Validates browser support for required APIs

---

### AudioStreamer

**File:** `src/static/js/audio/audio-streamer.js`

**Purpose:** Manages playback of audio data received from the Gemini API.

**Key Features:**
- Queue-based audio playback
- Support for audio worklets (volume meter, effects)
- Smooth scheduling to prevent gaps
- Automatic buffer management

**Technical Details:**
- Maintains internal audio queue
- Schedules buffers ahead of time (200ms look-ahead)
- Converts PCM16 to Float32 for playback
- Supports dynamic worklet attachment

**Audio Processing Pipeline:**
```
PCM16 Data → Float32 Conversion → Audio Queue → Buffer Scheduling → Audio Worklets → Speaker Output
```

**Configuration:**
```javascript
const context = new AudioContext();
const streamer = new AudioStreamer(context);

// Add volume meter
await streamer.addWorklet(
  'vumeter-out',
  'js/audio/worklets/vol-meter.js',
  (event) => {
    console.log('Volume:', event.data.volume);
  }
);
```

**Performance Characteristics:**
- Buffer size: 7680 samples
- Initial buffer time: 100ms
- Schedule ahead time: 200ms
- Sample rate: 24000 Hz (configurable)

---

## Video Components

### VideoManager

**File:** `src/static/js/video/video-manager.js`

**Purpose:** High-level manager for video capture with intelligent frame processing.

**Key Features:**
- Motion detection to reduce bandwidth
- Frame preview with enlargement option
- Camera flip support (front/back)
- Configurable frame rate
- Automatic frame optimization

**Motion Detection Algorithm:**
```javascript
// Compares pixel differences between frames
// Only sends frames when significant changes detected
// Threshold: 10 (configurable)
// Force send: Every 10th frame regardless of motion
```

**Frame Processing:**
1. Capture frame from camera
2. Detect motion vs. previous frame
3. Skip low-motion frames (unless 10th frame)
4. Resize for preview (50% scale)
5. Send to callback if significant

**Configuration:**
```javascript
const manager = new VideoManager();
await manager.start(15, (frameData) => {
  // frameData: { mimeType: "image/jpeg", data: base64String }
});
```

**DOM Integration:**
- `video-container`: Main container
- `preview`: Live video preview element
- `frame-preview`: Canvas for processed frames
- `flip-camera`: Button for camera switching

---

### VideoRecorder

**File:** `src/static/js/video/video-recorder.js`

**Purpose:** Low-level video recording with frame extraction.

**Key Features:**
- Configurable FPS, quality, resolution
- Frame validation
- Base64 encoding
- JPEG compression
- Aspect ratio preservation

**Technical Details:**
- Uses `getUserMedia()` for camera access
- Canvas-based frame capture
- Interval-based frame extraction
- Automatic resolution adjustment

**Options:**
```javascript
{
  fps: 15,           // Frames per second
  quality: 0.6,      // JPEG quality (0-1)
  width: 640,        // Desired width
  height: 480,       // Desired height
  maxFrameSize: 102400  // Max size in bytes
}
```

**Frame Processing:**
```
Video Stream → Canvas Draw → JPEG Encode → Base64 → Validate → Send
```

---

### ScreenRecorder

**File:** `src/static/js/video/screen-recorder.js`

**Purpose:** Records screen content for sharing with the AI.

**Key Features:**
- Screen capture via `getDisplayMedia()`
- Lower FPS optimized for screen content
- Higher quality settings
- Automatic cleanup on stream end

**Differences from VideoRecorder:**
- Higher default quality (0.8 vs 0.6)
- Lower default FPS (2 vs 15)
- Larger resolution (1280x720 vs 640x480)
- Larger max frame size (200KB vs 100KB)

**Use Cases:**
- Code review
- Presentation analysis
- UI/UX feedback
- Technical support

---

## Core Components

### MultimodalLiveClient

**File:** `src/static/js/core/websocket-client.js`

**Purpose:** Main client for WebSocket communication with Gemini API.

**Architecture:**
```
Application
    ↓
MultimodalLiveClient (EventEmitter)
    ↓
WebSocket Connection
    ↓
Gemini API
```

**Event Flow:**
1. Application sends message via `send()` or `sendRealtimeInput()`
2. Client serializes and transmits via WebSocket
3. Server responds with events
4. Client emits events for application to handle

**Message Types:**

**Outgoing:**
- `setup`: Initial configuration
- `clientContent`: Text messages
- `realtimeInput`: Audio/video data
- `toolResponse`: Tool execution results

**Incoming:**
- `setupComplete`: Setup acknowledged
- `serverContent`: Text/audio responses
- `toolCall`: Tool execution request
- `toolCallCancellation`: Tool cancelled

**Connection Lifecycle:**
```
Created → connect() → Setup → setupcomplete → Active → disconnect() → Closed
```

**Error Recovery:**
- Automatic event emission on errors
- Connection state tracking
- Graceful disconnection
- Error boundary integration

---

### WorkletRegistry

**File:** `src/static/js/core/worklet-registry.js`

**Purpose:** Manages registration and lifecycle of audio worklets.

**Features:**
- Prevents duplicate worklet registration
- Supports multiple handlers per worklet
- Context-specific registration
- URL creation from source code

**Usage Pattern:**
```javascript
// Register once per AudioContext
const worklets = registeredWorklets.get(audioContext);
if (!worklets || !worklets[workletName]) {
  // Register new worklet
}
```

---

## Tool Components

### ToolManager

**File:** `src/static/js/tools/tool-manager.js`

**Purpose:** Manages tool registration and execution.

**Architecture:**
```
ToolManager
  ├── GoogleSearchTool
  ├── WeatherTool
  └── [Custom Tools...]
```

**Tool Lifecycle:**
1. Tool registered with `registerTool()`
2. Declaration added to API config
3. API calls tool by name
4. ToolManager routes to correct tool
5. Tool executes and returns result
6. ToolManager formats response

**Tool Interface:**
```javascript
class CustomTool {
  getDeclaration() {
    // Returns tool schema for API
    return [{
      name: 'tool_name',
      description: 'Tool description',
      parameters: { /* JSON schema */ }
    }];
  }
  
  async execute(args) {
    // Executes tool logic
    return { result: 'data' };
  }
}
```

---

### GoogleSearchTool

**File:** `src/static/js/tools/google-search.js`

**Purpose:** Enables Google Search capability (handled server-side by Gemini).

**Implementation:**
- Returns empty declaration object
- Gemini API handles actual search
- No client-side execution needed

**Why Empty?**
The Gemini API has built-in Google Search integration. By including an empty Google Search declaration, we signal to the API that search functionality should be enabled.

---

### WeatherTool

**File:** `src/static/js/tools/weather-tool.js`

**Purpose:** Provides weather forecast information (mock implementation).

**Features:**
- Deterministic pseudo-random data
- Consistent results for same location/date
- Realistic weather patterns
- Temperature ranges based on conditions

**Data Generation:**
```javascript
// Generates consistent weather from location + date hash
const seed = hashString(location + date);
const condition = conditions[seed % conditions.length];
const temperature = tempRanges[condition].min + (seed % range);
```

**Weather Conditions:**
- sunny, partly cloudy, cloudy
- light rain, heavy rain, thunderstorm
- windy, snow, foggy

---

## Utility Components

### Logger

**File:** `src/static/js/utils/logger.js`

**Purpose:** Centralized logging with event emission and export capability.

**Features:**
- Multiple log levels (debug, info, warn, error)
- In-memory log storage (last 1000 entries)
- Event-based log distribution
- JSON export functionality

**Log Entry Structure:**
```javascript
{
  timestamp: "2025-10-11T12:00:00.000Z",
  level: "info",
  message: "Log message",
  data: { /* optional data */ }
}
```

**Singleton Pattern:**
```javascript
// Always returns same instance
const logger = Logger.getInstance();
```

**Usage Pattern:**
```javascript
// Direct logging
Logger.info('Operation completed', { duration: 123 });

// Event subscription
Logger.getInstance().on('log', (entry) => {
  // Send to external service
});

// Export logs
Logger.export(); // Downloads JSON file
```

---

### Error Boundary

**File:** `src/static/js/utils/error-boundary.js`

**Purpose:** Standardized error handling across the application.

**Error Codes:**
```javascript
// Audio
AUDIO_DEVICE_NOT_FOUND
AUDIO_PERMISSION_DENIED
AUDIO_NOT_SUPPORTED
AUDIO_INITIALIZATION_FAILED
AUDIO_RECORDING_FAILED
AUDIO_STOP_FAILED
AUDIO_CONVERSION_FAILED

// Video
VIDEO_DEVICE_NOT_FOUND
VIDEO_PERMISSION_DENIED
VIDEO_NOT_SUPPORTED
VIDEO_START_FAILED
VIDEO_STOP_FAILED

// WebSocket
WEBSOCKET_CONNECTION_FAILED
WEBSOCKET_MESSAGE_FAILED
WEBSOCKET_CLOSE_FAILED

// API
API_AUTHENTICATION_FAILED
API_REQUEST_FAILED
API_RESPONSE_INVALID

// General
UNKNOWN_ERROR
INVALID_STATE
INVALID_PARAMETER
```

**ApplicationError Class:**
```javascript
class ApplicationError extends Error {
  constructor(message, code, details) {
    // Includes timestamp, code, details
    // Preserves stack trace
    // JSON serializable
  }
}
```

**Best Practices:**
```javascript
try {
  await operation();
} catch (error) {
  throw new ApplicationError(
    'Descriptive message',
    ErrorCodes.SPECIFIC_CODE,
    { originalError: error }
  );
}
```

---

### Utils

**File:** `src/static/js/utils/utils.js`

**Purpose:** Common utility functions.

**Functions:**

#### `blobToJSON(blob)`
```javascript
// Converts Blob to JSON object
// Used for WebSocket message parsing
const json = await blobToJSON(messageBlob);
```

#### `base64ToArrayBuffer(base64)`
```javascript
// Converts base64 string to ArrayBuffer
// Used for audio data processing
const buffer = base64ToArrayBuffer(audioData);
```

---

## Audio Worklets

### AudioProcessingWorklet

**File:** `src/static/js/audio/worklets/audio-processing.js`

**Purpose:** Processes audio input for recording.

**Processing Steps:**
1. Receive Float32 audio samples
2. Convert to Int16 PCM format
3. Buffer samples (2048 samples)
4. Send complete buffers via message port

**Conversion Algorithm:**
```javascript
// Float32 (-1.0 to 1.0) → Int16 (-32768 to 32767)
const int16 = Math.max(-32768, Math.min(32767, 
  Math.floor(float32 * 32768)
));
```

**Message Format:**
```javascript
{
  event: 'chunk',
  data: {
    int16arrayBuffer: ArrayBuffer
  }
}
```

---

### VUMeterProcessor

**File:** `src/static/js/audio/worklets/vol-meter.js`

**Purpose:** Calculates real-time audio volume (RMS).

**Algorithm:**
```javascript
// 1. Calculate sum of squares
sum = Σ(sample²)

// 2. Calculate RMS
rms = √(sum / sampleCount)

// 3. Apply smoothing
volume = max(rms, previousVolume * 0.95)
```

**Update Frequency:**
- Default: Every 25ms
- Configurable via `_updateIntervalInMS`

**Message Format:**
```javascript
{
  volume: 0.0 to 1.0  // Normalized volume level
}
```

**Use Cases:**
- Audio visualizers
- Voice activity detection
- Level meters
- Debug/monitoring

---

## Component Interactions

### Audio Recording Flow

```
Microphone
    ↓
getUserMedia()
    ↓
AudioContext
    ↓
AudioWorkletNode (AudioProcessingWorklet)
    ↓
PCM16 Chunks
    ↓
Base64 Encoding
    ↓
MultimodalLiveClient
    ↓
WebSocket
    ↓
Gemini API
```

### Audio Playback Flow

```
Gemini API
    ↓
WebSocket
    ↓
MultimodalLiveClient (audio event)
    ↓
AudioStreamer
    ↓
PCM16 → Float32 Conversion
    ↓
Audio Queue
    ↓
Buffer Scheduling
    ↓
AudioWorkletNode (VUMeterProcessor)
    ↓
Speaker Output
```

### Video Capture Flow

```
Camera
    ↓
getUserMedia()
    ↓
Video Element
    ↓
VideoRecorder (setInterval)
    ↓
Canvas Draw
    ↓
JPEG Encoding
    ↓
Base64
    ↓
VideoManager (Motion Detection)
    ↓
MultimodalLiveClient
    ↓
WebSocket
    ↓
Gemini API
```

### Tool Execution Flow

```
User Message
    ↓
MultimodalLiveClient.send()
    ↓
WebSocket → Gemini API
    ↓
Gemini decides to use tool
    ↓
WebSocket ← toolCall event
    ↓
MultimodalLiveClient.handleToolCall()
    ↓
ToolManager.handleToolCall()
    ↓
Tool.execute()
    ↓
Result
    ↓
MultimodalLiveClient.sendToolResponse()
    ↓
WebSocket → Gemini API
    ↓
Gemini generates response
    ↓
WebSocket ← serverContent
    ↓
MultimodalLiveClient (content event)
    ↓
UI Display
```

---

## Performance Considerations

### Audio
- **Sample Rate**: 16kHz for input, 24kHz for output
- **Buffer Size**: 2048 samples for recording
- **Latency**: ~100-200ms total (network + processing)

### Video
- **Frame Rate**: 15 FPS for camera, 2 FPS for screen
- **Quality**: 0.6-0.8 JPEG compression
- **Resolution**: Auto-adjusted to maintain aspect ratio
- **Bandwidth**: ~50-100 KB per frame

### WebSocket
- **Message Size**: Typically 1-100 KB per message
- **Frequency**: Continuous for audio/video
- **Protocol**: Binary (Blob) for efficiency

### Memory
- **Audio Queue**: Dynamically sized, cleared on playback
- **Video Frames**: Not stored, processed immediately
- **Logs**: Last 1000 entries kept in memory

---

## Browser Compatibility

### Required APIs
- WebSocket
- Web Audio API (AudioContext, AudioWorklet)
- MediaDevices (getUserMedia, getDisplayMedia)
- Canvas API
- ES6+ (async/await, classes, modules)

### Supported Browsers
- Chrome/Edge: 89+
- Firefox: 88+
- Safari: 14.1+
- Opera: 75+

### Not Supported
- Internet Explorer
- Older mobile browsers

---

## Testing Components

Each component should be tested for:

1. **Initialization**: Proper setup and configuration
2. **Core Functionality**: Primary operations work correctly
3. **Error Handling**: Graceful failure on errors
4. **Resource Cleanup**: Proper disposal of resources
5. **Browser Compatibility**: Works across supported browsers

Example test structure:
```javascript
describe('AudioRecorder', () => {
  it('should initialize with default sample rate', () => {});
  it('should start recording when start() called', () => {});
  it('should throw error without microphone permission', () => {});
  it('should stop cleanly', () => {});
  it('should encode audio to base64', () => {});
});
```

---

## Extending Components

### Adding a New Tool

```javascript
// 1. Create tool class
class MyTool {
  getDeclaration() {
    return [{
      name: 'my_tool',
      description: 'Description',
      parameters: { /* schema */ }
    }];
  }
  
  async execute(args) {
    // Implementation
    return result;
  }
}

// 2. Register tool
toolManager.registerTool('myTool', new MyTool());
```

### Adding a New Audio Worklet

```javascript
// 1. Create worklet file
class MyWorklet extends AudioWorkletProcessor {
  process(inputs, outputs) {
    // Processing logic
    return true;
  }
}
registerProcessor('my-worklet', MyWorklet);

// 2. Add to streamer
await streamer.addWorklet(
  'my-worklet',
  'path/to/worklet.js',
  (event) => {
    // Handle messages
  }
);
```

### Adding a New Feature

1. Create component file in appropriate directory
2. Import required utilities and dependencies
3. Implement error handling with ApplicationError
4. Add logging with Logger
5. Emit events for important state changes
6. Document public API
7. Add examples
8. Update this documentation

---

## Maintenance

### Code Organization
- Components grouped by functionality
- Clear separation of concerns
- Minimal coupling between components
- Event-driven communication

### Dependencies
- EventEmitter3 (for events)
- No other external dependencies
- Native Web APIs only

### Updates
- Follow semantic versioning
- Document breaking changes
- Provide migration guides
- Keep examples up to date

---

This component documentation provides a comprehensive overview of each piece of the application, its purpose, implementation details, and how it interacts with other components.
