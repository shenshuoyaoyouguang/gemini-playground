# Architecture Overview

## System Architecture

The Gemini 2.0 Playground is a full-stack application with the following architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser (Client)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐ │
│  │   Audio   │  │   Video   │  │   Tools   │  │   Core   │ │
│  │Components │  │Components │  │  Manager  │  │  Client  │ │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └────┬─────┘ │
│        └────────────┬──┴──────────────┴─────────────┘       │
│                     │  MultimodalLiveClient                 │
│                     │  (WebSocket Client)                   │
└─────────────────────┼─────────────────────────────────────────┘
                      │
                   WebSocket
                      │
┌─────────────────────┼─────────────────────────────────────────┐
│                     │         Proxy Server                    │
│              ┌──────┴──────┐                                  │
│              │  WebSocket  │                                  │
│              │    Proxy    │                                  │
│              └──────┬──────┘                                  │
│                     │                                         │
│              ┌──────┴──────┐                                  │
│              │  REST API   │                                  │
│              │    Proxy    │                                  │
│              │ (OpenAI ↔   │                                  │
│              │  Gemini)    │                                  │
│              └──────┬──────┘                                  │
├─────────────────────┼─────────────────────────────────────────┤
│  Deployment Options: Cloudflare Workers / Deno Deploy        │
└─────────────────────┼─────────────────────────────────────────┘
                      │
                   HTTPS
                      │
┌─────────────────────┼─────────────────────────────────────────┐
│                     ▼                                         │
│              Gemini 2.0 API                                   │
│         (generativelanguage.googleapis.com)                   │
└───────────────────────────────────────────────────────────────┘
```

## Deployment Architectures

### Cloudflare Workers Deployment

```
User Request
    ↓
Cloudflare Edge Network (CDN)
    ↓
Worker Script (src/index.js)
    ├── Static Assets (__STATIC_CONTENT)
    ├── WebSocket Handler
    └── API Proxy (worker.mjs)
    ↓
Gemini API
```

**Key Features:**
- Global edge deployment
- Automatic HTTPS
- DDoS protection
- Serverless execution
- Custom domain support required for China

### Deno Deploy Deployment

```
User Request
    ↓
Deno Deploy Edge Network
    ↓
Deno Handler (src/deno_index.ts)
    ├── Static File Server
    ├── WebSocket Handler
    └── API Proxy (worker.mjs)
    ↓
Gemini API
```

**Key Features:**
- TypeScript native
- Global edge deployment
- Automatic HTTPS
- Built-in domain
- Works directly in China

## Data Flow

### Multimodal Conversation Flow

```
┌─────────┐
│  User   │
└────┬────┘
     │ 1. Text input / Enable mic/camera
     ▼
┌─────────────────────────┐
│  UI Event Handlers      │
│  (main.js)              │
└────┬────────────────────┘
     │ 2. Capture audio/video
     ▼
┌─────────────────────────┐
│  Audio/Video Recorders  │
│  - AudioRecorder        │
│  - VideoManager         │
│  - ScreenRecorder       │
└────┬────────────────────┘
     │ 3. Encode to base64
     ▼
┌─────────────────────────┐
│ MultimodalLiveClient    │
│ - send()                │
│ - sendRealtimeInput()   │
└────┬────────────────────┘
     │ 4. WebSocket message
     ▼
┌─────────────────────────┐
│  Proxy Server           │
│  - WebSocket proxy      │
└────┬────────────────────┘
     │ 5. Forward to Gemini
     ▼
┌─────────────────────────┐
│  Gemini 2.0 API         │
│  - Process multimodal   │
│  - Generate response    │
└────┬────────────────────┘
     │ 6. Response (text/audio)
     ▼
┌─────────────────────────┐
│  Proxy Server           │
│  - Forward response     │
└────┬────────────────────┘
     │ 7. WebSocket message
     ▼
┌─────────────────────────┐
│ MultimodalLiveClient    │
│ - Emit events           │
└────┬────────────────────┘
     │ 8. Process response
     ├──► Text: Display in UI
     └──► Audio: AudioStreamer → Speakers
```

### Tool Execution Flow

```
User: "What's the weather in London?"
    ↓
MultimodalLiveClient.send()
    ↓
WebSocket → Gemini API
    ↓
Gemini analyzes and decides to use tool
    ↓
WebSocket ← toolCall event
    ↓
MultimodalLiveClient.handleToolCall()
    ↓
ToolManager.handleToolCall()
    ↓
WeatherTool.execute({ location: 'London', date: '2025-10-11' })
    ↓
{ temperature: 15, condition: 'cloudy', ... }
    ↓
MultimodalLiveClient.sendToolResponse()
    ↓
WebSocket → Gemini API
    ↓
Gemini generates natural language response
    ↓
"The weather in London on October 11th will be cloudy with a temperature of 15°C"
    ↓
Display in UI
```

## Component Architecture

### Core Layer

```
┌──────────────────────────────────────────────┐
│           MultimodalLiveClient               │
│  (EventEmitter, WebSocket Management)        │
├──────────────────────────────────────────────┤
│  Properties:                                 │
│  - ws: WebSocket                             │
│  - config: Config                            │
│  - toolManager: ToolManager                  │
│                                              │
│  Methods:                                    │
│  - connect(config, apiKey)                   │
│  - disconnect()                              │
│  - send(message)                             │
│  - sendRealtimeInput(chunks)                 │
│  - sendToolResponse(response)                │
│                                              │
│  Events:                                     │
│  - open, close, error                        │
│  - audio, content, log                       │
│  - setupcomplete, turncomplete               │
│  - interrupted, toolcallcancellation         │
└──────────────────────────────────────────────┘
```

### Audio Layer

```
┌────────────────────┐     ┌─────────────────────┐
│  AudioRecorder     │     │   AudioStreamer     │
├────────────────────┤     ├─────────────────────┤
│ INPUT PIPELINE:    │     │ OUTPUT PIPELINE:    │
│                    │     │                     │
│ Microphone         │     │ WebSocket           │
│      ↓             │     │      ↓              │
│ getUserMedia       │     │ PCM16 Data          │
│      ↓             │     │      ↓              │
│ AudioContext       │     │ Float32 Convert     │
│      ↓             │     │      ↓              │
│ AudioWorklet       │     │ Audio Queue         │
│      ↓             │     │      ↓              │
│ Float32→Int16      │     │ Buffer Schedule     │
│      ↓             │     │      ↓              │
│ Base64 Encode      │     │ AudioWorklet        │
│      ↓             │     │      ↓              │
│ Callback           │     │ Speakers            │
└────────────────────┘     └─────────────────────┘
```

### Video Layer

```
┌──────────────────────────────────────────────┐
│            VideoManager                      │
│  (High-level, motion detection)              │
└────────────────┬─────────────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
┌───▼─────────────┐  ┌────────▼──────────┐
│ VideoRecorder   │  │ ScreenRecorder    │
│                 │  │                   │
│ Camera          │  │ Screen            │
│      ↓          │  │      ↓            │
│ getUserMedia    │  │ getDisplayMedia   │
│      ↓          │  │      ↓            │
│ Video Element   │  │ Video Element     │
│      ↓          │  │      ↓            │
│ Canvas Draw     │  │ Canvas Draw       │
│      ↓          │  │      ↓            │
│ JPEG Encode     │  │ JPEG Encode       │
│      ↓          │  │      ↓            │
│ Base64          │  │ Base64            │
│      ↓          │  │      ↓            │
│ Callback        │  │ Callback          │
└─────────────────┘  └───────────────────┘
```

### Tool Layer

```
┌──────────────────────────────────────────────┐
│            ToolManager                       │
│  (Registry, Router, Handler)                 │
├──────────────────────────────────────────────┤
│  tools: Map<string, Tool>                    │
│                                              │
│  registerTool(name, instance)                │
│  getToolDeclarations()                       │
│  handleToolCall(functionCall)                │
└────────┬─────────────────────────────────────┘
         │
    ┌────┴────┬─────────────────┬──────────────┐
    │         │                 │              │
┌───▼───┐ ┌──▼────────┐ ┌──────▼─────┐ ┌─────▼──────┐
│Google │ │ Weather   │ │   Custom   │ │  Future    │
│Search │ │   Tool    │ │    Tool    │ │   Tools    │
└───────┘ └───────────┘ └────────────┘ └────────────┘
```

## State Management

### Application State

```javascript
// Global state (main.js)
{
  isRecording: boolean,      // Microphone state
  isConnected: boolean,      // WebSocket state
  isVideoActive: boolean,    // Camera state
  isScreenSharing: boolean,  // Screen share state
  isUsingTool: boolean,      // Tool execution state
  
  // Component instances
  client: MultimodalLiveClient,
  audioRecorder: AudioRecorder,
  audioStreamer: AudioStreamer,
  videoManager: VideoManager,
  screenRecorder: ScreenRecorder
}
```

### Connection States

```
┌──────────┐
│  Created │
└────┬─────┘
     │ connect()
     ▼
┌──────────┐
│Connecting│
└────┬─────┘
     │ 'open' event
     ▼
┌──────────┐
│  Setup   │
└────┬─────┘
     │ 'setupcomplete'
     ▼
┌──────────┐
│  Active  │◄─┐
└────┬─────┘  │ Messages
     │        │
     │ disconnect()
     ▼
┌──────────┐
│  Closed  │
└──────────┘
```

## Security Architecture

### API Key Handling

```
┌─────────────────────────────────────────────┐
│  Client (Browser)                           │
│  - API key stored in localStorage           │
│  - Sent via query parameter                 │
│  - Never exposed in code                    │
└────────────────┬────────────────────────────┘
                 │
                 │ ?key=API_KEY
                 ▼
┌─────────────────────────────────────────────┐
│  Proxy Server                               │
│  - Forwards key to Gemini                   │
│  - No storage or logging                    │
│  - HTTPS enforced                           │
└────────────────┬────────────────────────────┘
                 │
                 │ x-goog-api-key: API_KEY
                 ▼
┌─────────────────────────────────────────────┐
│  Gemini API                                 │
│  - Validates key                            │
│  - Rate limiting                            │
│  - Usage tracking                           │
└─────────────────────────────────────────────┘
```

### Content Security

```
- HTTPS only for production
- WebSocket Secure (WSS) for connections
- CORS headers properly configured
- No sensitive data in logs
- Base64 encoding for data transmission
- No client-side data persistence (except API key)
```

## Performance Optimizations

### Audio Optimization

```
┌─────────────────────────────────────┐
│ Audio Recording Optimizations      │
├─────────────────────────────────────┤
│ 1. AudioWorklet (vs ScriptProcessor)│
│    - Lower latency                  │
│    - Better performance             │
│    - No main thread blocking        │
│                                     │
│ 2. Buffering                        │
│    - 2048 sample buffer             │
│    - Reduces message frequency      │
│    - Balances latency vs overhead   │
│                                     │
│ 3. Sample Rate                      │
│    - 16kHz input (voice optimized)  │
│    - Reduced bandwidth              │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ Audio Playback Optimizations       │
├─────────────────────────────────────┤
│ 1. Queue Management                 │
│    - Dynamic queue sizing           │
│    - Look-ahead scheduling          │
│    - Gap prevention                 │
│                                     │
│ 2. Buffer Scheduling                │
│    - 200ms look-ahead               │
│    - Smooth playback                │
│    - Minimal latency                │
└─────────────────────────────────────┘
```

### Video Optimization

```
┌─────────────────────────────────────┐
│ Video Capture Optimizations        │
├─────────────────────────────────────┤
│ 1. Motion Detection                 │
│    - Skip static frames             │
│    - 10-20x bandwidth reduction     │
│    - Threshold-based                │
│                                     │
│ 2. Frame Rate Adaptation            │
│    - 15 FPS camera (interactive)    │
│    - 2 FPS screen (static content)  │
│                                     │
│ 3. Compression                      │
│    - JPEG quality 0.6-0.8           │
│    - Resolution scaling             │
│    - Max frame size limiting        │
│                                     │
│ 4. Force Frame Interval             │
│    - Every 10th frame sent          │
│    - Ensures periodic updates       │
└─────────────────────────────────────┘
```

### Network Optimization

```
┌─────────────────────────────────────┐
│ WebSocket Optimizations             │
├─────────────────────────────────────┤
│ 1. Binary Protocol                  │
│    - Blob messages vs JSON strings  │
│    - Reduced overhead               │
│                                     │
│ 2. Message Batching                 │
│    - Audio: buffered chunks         │
│    - Video: motion-based sending    │
│                                     │
│ 3. Compression                      │
│    - Base64 for binary data         │
│    - JPEG for images                │
│    - PCM16 for audio                │
└─────────────────────────────────────┘
```

## Scalability Considerations

### Horizontal Scaling

```
Edge Deployment (Cloudflare/Deno)
├── Automatic global distribution
├── Each edge handles connections independently
├── No central state
└── Stateless architecture
```

### Vertical Scaling

```
Client-side Processing
├── Worklets for audio processing
├── Canvas for video processing
├── No server-side processing needed
└── Scales with user's device
```

## Error Handling Strategy

### Error Propagation

```
Low-level Error (Browser API)
    ↓
Component catches error
    ↓
Wrap in ApplicationError with code
    ↓
Log error with Logger
    ↓
Emit error event
    ↓
UI displays user-friendly message
```

### Error Recovery

```
┌─────────────────────────────────────┐
│ Automatic Recovery                  │
├─────────────────────────────────────┤
│ WebSocket disconnect                │
│   → Emit close event                │
│   → UI shows "Reconnect" button     │
│                                     │
│ Audio/Video device lost             │
│   → Stop gracefully                 │
│   → Release resources               │
│   → UI shows error message          │
│                                     │
│ Tool execution failure              │
│   → Return error response           │
│   → Model generates fallback        │
│   → Continue conversation           │
└─────────────────────────────────────┘
```

## Monitoring & Debugging

### Logging Architecture

```
┌─────────────────────────────────────┐
│  Application Events                 │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Logger.log(level, message, data)   │
├─────────────────────────────────────┤
│  1. Console output (browser devtools)│
│  2. In-memory storage (last 1000)   │
│  3. Event emission (log listeners)  │
│  4. Export functionality (JSON)     │
└─────────────────────────────────────┘
```

### Debug Tools

```javascript
// Enable debug logging
Logger.getInstance().on('log', console.log);

// Export logs
Logger.export(); // Downloads JSON

// Monitor WebSocket
client.on('log', (log) => {
  console.log(`${log.type}:`, log.message);
});

// Check component state
console.log({
  recording: isRecording,
  connected: isConnected,
  video: isVideoActive
});
```

## Future Architecture Enhancements

### Potential Improvements

1. **State Management**
   - Redux/Zustand for complex state
   - Persistent state across sessions
   - Undo/redo functionality

2. **Testing**
   - Unit tests for components
   - Integration tests for flows
   - E2E tests with Playwright

3. **Performance**
   - WebRTC for peer-to-peer
   - WebCodecs for video encoding
   - Shared workers for background processing

4. **Features**
   - Multi-user sessions
   - Session recording/playback
   - Custom voice training
   - Advanced video effects

5. **Infrastructure**
   - Database for session storage
   - Authentication system
   - Rate limiting
   - Analytics

---

This architecture document provides a comprehensive overview of how the system is structured and how components interact. It serves as a reference for understanding the application's design decisions and for planning future enhancements.
