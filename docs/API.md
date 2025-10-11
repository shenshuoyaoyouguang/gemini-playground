# Gemini 2.0 Playground - API Documentation

## Table of Contents

1. [Overview](#overview)
2. [Server APIs](#server-apis)
   - [WebSocket API](#websocket-api)
   - [REST API Proxy](#rest-api-proxy)
3. [Client APIs](#client-apis)
   - [Core](#core)
   - [Audio](#audio)
   - [Video](#video)
   - [Tools](#tools)
   - [Utilities](#utilities)
4. [Configuration](#configuration)
5. [Examples](#examples)

---

## Overview

The Gemini 2.0 Playground provides a comprehensive set of APIs for interacting with Google's Gemini 2.0 Multimodal Live API. It supports:

- Real-time multimodal communication (text, audio, video)
- WebSocket-based bidirectional streaming
- OpenAI-compatible REST API
- Audio recording and playback
- Video/camera capture
- Screen sharing
- Extensible tool system

---

## Server APIs

### WebSocket API

The WebSocket API provides real-time bidirectional communication with the Gemini API.

#### Connection

**Endpoint:** `ws://your-domain/ws/google.ai.generativelanguage.v1alpha.GenerativeService.BidiGenerateContent?key={API_KEY}`

**Protocol:** WebSocket

**Query Parameters:**
- `key` (required): Your Gemini API key

#### Message Format

All messages are sent as JSON strings.

##### Setup Message

Initialize the connection with configuration:

```json
{
  "setup": {
    "model": "models/gemini-2.0-flash-exp",
    "generationConfig": {
      "responseModalities": "audio",
      "speechConfig": {
        "voiceConfig": {
          "prebuiltVoiceConfig": {
            "voiceName": "Aoede"
          }
        }
      }
    },
    "systemInstruction": {
      "parts": [{
        "text": "You are a helpful assistant."
      }]
    },
    "tools": []
  }
}
```

##### Client Content Message

Send user messages to the model:

```json
{
  "clientContent": {
    "turns": [{
      "role": "user",
      "parts": [
        { "text": "Hello, how are you?" }
      ]
    }],
    "turnComplete": true
  }
}
```

##### Realtime Input Message

Send audio or video frames:

```json
{
  "realtimeInput": {
    "mediaChunks": [{
      "mimeType": "audio/pcm;rate=16000",
      "data": "base64-encoded-audio-data"
    }]
  }
}
```

##### Tool Response Message

Respond to tool calls:

```json
{
  "toolResponse": {
    "functionResponses": [{
      "response": {
        "output": { "result": "data" }
      },
      "id": "tool-call-id"
    }]
  }
}
```

#### Server Messages

##### Setup Complete

```json
{
  "setupComplete": {}
}
```

##### Server Content

```json
{
  "serverContent": {
    "modelTurn": {
      "parts": [
        { "text": "Response text" },
        {
          "inlineData": {
            "mimeType": "audio/pcm",
            "data": "base64-audio-data"
          }
        }
      ]
    },
    "turnComplete": true
  }
}
```

##### Tool Call

```json
{
  "toolCall": {
    "functionCalls": [{
      "name": "get_weather_on_date",
      "args": {
        "location": "London",
        "date": "2025-10-11"
      },
      "id": "call-id"
    }]
  }
}
```

---

### REST API Proxy

The REST API provides OpenAI-compatible endpoints that proxy requests to Gemini API.

#### Authentication

All API requests require authentication via Bearer token:

```
Authorization: Bearer YOUR_GEMINI_API_KEY
```

#### List Models

**Endpoint:** `GET /v1/models`

**Description:** List all available models.

**Example Request:**

```bash
curl https://your-domain.com/v1/models \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY"
```

**Example Response:**

```json
{
  "object": "list",
  "data": [
    {
      "id": "gemini-2.0-flash-exp",
      "object": "model",
      "created": 0,
      "owned_by": ""
    }
  ]
}
```

#### Chat Completions

**Endpoint:** `POST /v1/chat/completions`

**Description:** Generate chat completions.

**Request Body:**

```json
{
  "model": "gemini-2.0-flash-exp",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "What is the weather like today?"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 1000,
  "stream": false
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Model identifier |
| `messages` | array | Yes | Array of message objects |
| `temperature` | number | No | Sampling temperature (0-2) |
| `max_tokens` | number | No | Maximum tokens to generate |
| `top_p` | number | No | Nucleus sampling parameter |
| `top_k` | number | No | Top-k sampling parameter |
| `stream` | boolean | No | Whether to stream responses |
| `stop` | string/array | No | Stop sequences |
| `frequency_penalty` | number | No | Frequency penalty |
| `presence_penalty` | number | No | Presence penalty |

**Example Request:**

```bash
curl https://your-domain.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY" \
  -d '{
    "model": "gemini-2.0-flash-exp",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
```

**Example Response:**

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "gemini-2.0-flash-exp",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "Hello! How can I help you today?"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30
  }
}
```

#### Streaming Response

When `stream: true`, responses are sent as Server-Sent Events:

```
data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","created":1234567890,"model":"gemini-2.0-flash-exp","choices":[{"index":0,"delta":{"role":"assistant","content":"Hello"},"finish_reason":null}]}

data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","created":1234567890,"model":"gemini-2.0-flash-exp","choices":[{"index":0,"delta":{"content":"!"},"finish_reason":null}]}

data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","created":1234567890,"model":"gemini-2.0-flash-exp","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

#### Embeddings

**Endpoint:** `POST /v1/embeddings`

**Description:** Create embeddings for text.

**Request Body:**

```json
{
  "model": "text-embedding-004",
  "input": ["Text to embed", "Another text"],
  "dimensions": 768
}
```

**Example Request:**

```bash
curl https://your-domain.com/v1/embeddings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY" \
  -d '{
    "model": "text-embedding-004",
    "input": "Hello world"
  }'
```

**Example Response:**

```json
{
  "object": "list",
  "data": [{
    "object": "embedding",
    "index": 0,
    "embedding": [0.1, 0.2, 0.3, ...]
  }],
  "model": "text-embedding-004"
}
```

---

## Client APIs

### Core

#### MultimodalLiveClient

The main client for interacting with the Gemini API via WebSocket.

**Import:**

```javascript
import { MultimodalLiveClient } from './core/websocket-client.js';
```

**Constructor:**

```javascript
const client = new MultimodalLiveClient();
```

**Methods:**

##### `connect(config, apiKey)`

Connects to the WebSocket server.

**Parameters:**
- `config` (Object): Configuration object
  - `model` (string): Model name
  - `generationConfig` (Object): Generation configuration
    - `responseModalities` (string): Response modalities ("audio", "text", or "audio,text")
    - `speechConfig` (Object): Speech configuration
  - `systemInstruction` (Object): System instruction
  - `tools` (Array): Additional tools
- `apiKey` (string): Gemini API key

**Returns:** `Promise<boolean>`

**Example:**

```javascript
const config = {
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'audio',
    speechConfig: {
      voiceConfig: {
        prebuiltVoiceConfig: {
          voiceName: 'Aoede'
        }
      }
    }
  },
  systemInstruction: {
    parts: [{
      text: 'You are a helpful assistant.'
    }]
  }
};

await client.connect(config, 'YOUR_API_KEY');
```

##### `disconnect()`

Disconnects from the WebSocket server.

**Returns:** `boolean`

**Example:**

```javascript
client.disconnect();
```

##### `send(parts, turnComplete)`

Sends a message to the server.

**Parameters:**
- `parts` (string|Object|Array): Message content
- `turnComplete` (boolean): Whether the turn is complete (default: true)

**Example:**

```javascript
// Send text
client.send('Hello!');

// Send structured content
client.send([
  { text: 'What is in this image?' },
  { inlineData: { mimeType: 'image/jpeg', data: base64Data } }
]);
```

##### `sendRealtimeInput(chunks)`

Sends real-time audio/video input.

**Parameters:**
- `chunks` (Array): Array of media chunks
  - `mimeType` (string): MIME type
  - `data` (string): Base64-encoded data

**Example:**

```javascript
client.sendRealtimeInput([{
  mimeType: 'audio/pcm;rate=16000',
  data: base64AudioData
}]);
```

##### `sendToolResponse(toolResponse)`

Sends a tool response.

**Parameters:**
- `toolResponse` (Object): Tool response object

**Example:**

```javascript
client.sendToolResponse({
  functionResponses: [{
    response: { output: { result: 'data' } },
    id: 'call-id'
  }]
});
```

**Events:**

The client extends EventEmitter and emits the following events:

- `open`: Connection opened
- `close`: Connection closed
- `audio`: Audio data received
- `content`: Content received
- `setupcomplete`: Setup completed
- `turncomplete`: Turn completed
- `interrupted`: Model interrupted
- `toolcallcancellation`: Tool call cancelled
- `error`: Error occurred
- `log`: Log message

**Example:**

```javascript
client.on('open', () => {
  console.log('Connected');
});

client.on('audio', (audioData) => {
  // Process audio data
});

client.on('content', (content) => {
  console.log('Received:', content);
});

client.on('error', (error) => {
  console.error('Error:', error);
});
```

---

### Audio

#### AudioRecorder

Records audio from the microphone.

**Import:**

```javascript
import { AudioRecorder } from './audio/audio-recorder.js';
```

**Constructor:**

```javascript
const recorder = new AudioRecorder(sampleRate = 16000);
```

**Methods:**

##### `start(onAudioData)`

Starts audio recording.

**Parameters:**
- `onAudioData` (Function): Callback for audio data chunks

**Returns:** `Promise<void>`

**Example:**

```javascript
const recorder = new AudioRecorder();
await recorder.start((base64Data) => {
  // Send audio data to server
  client.sendRealtimeInput([{
    mimeType: 'audio/pcm;rate=16000',
    data: base64Data
  }]);
});
```

##### `stop()`

Stops audio recording.

**Example:**

```javascript
recorder.stop();
```

#### AudioStreamer

Streams audio for playback.

**Import:**

```javascript
import { AudioStreamer } from './audio/audio-streamer.js';
```

**Constructor:**

```javascript
const audioContext = new AudioContext();
const streamer = new AudioStreamer(audioContext);
```

**Methods:**

##### `addWorklet(workletName, workletSrc, handler)`

Adds an audio worklet.

**Parameters:**
- `workletName` (string): Worklet name
- `workletSrc` (string): Worklet source URL
- `handler` (Function): Message handler

**Returns:** `Promise<AudioStreamer>`

**Example:**

```javascript
await streamer.addWorklet(
  'vumeter-out',
  'js/audio/worklets/vol-meter.js',
  (event) => {
    console.log('Volume:', event.data.volume);
  }
);
```

##### `addPCM16(chunk)`

Adds PCM16 audio data to the playback queue.

**Parameters:**
- `chunk` (Uint8Array): Audio data chunk

**Example:**

```javascript
client.on('audio', (data) => {
  streamer.addPCM16(new Uint8Array(data));
});
```

##### `stop()`

Stops audio playback.

##### `resume()`

Resumes audio playback.

**Returns:** `Promise<void>`

##### `complete()`

Marks the stream as complete.

---

### Video

#### VideoManager

Manages video capture with motion detection.

**Import:**

```javascript
import { VideoManager } from './video/video-manager.js';
```

**Constructor:**

```javascript
const videoManager = new VideoManager();
```

**Methods:**

##### `start(fps, onFrame)`

Starts video capture.

**Parameters:**
- `fps` (number): Frames per second
- `onFrame` (Function): Callback for each frame

**Returns:** `Promise<boolean>`

**Example:**

```javascript
const videoManager = new VideoManager();
await videoManager.start(15, (frameData) => {
  client.sendRealtimeInput([frameData]);
});
```

##### `stop()`

Stops video capture.

**Example:**

```javascript
videoManager.stop();
```

##### `flipCamera()`

Switches between front and back camera.

**Returns:** `Promise<void>`

**Example:**

```javascript
await videoManager.flipCamera();
```

#### VideoRecorder

Low-level video recorder.

**Import:**

```javascript
import { VideoRecorder } from './video/video-recorder.js';
```

**Constructor:**

```javascript
const recorder = new VideoRecorder({
  fps: 15,
  quality: 0.7,
  width: 640,
  height: 480
});
```

**Options:**
- `fps` (number): Frames per second (default: 1)
- `quality` (number): JPEG quality 0-1 (default: 0.6)
- `width` (number): Video width (default: 640)
- `height` (number): Video height (default: 480)
- `maxFrameSize` (number): Max frame size in bytes (default: 102400)

**Methods:**

##### `start(previewElement, facingMode, onVideoData)`

Starts video recording.

**Parameters:**
- `previewElement` (HTMLVideoElement): Preview video element
- `facingMode` (string): Camera facing mode ('user' or 'environment')
- `onVideoData` (Function): Callback for video frames

**Returns:** `Promise<void>`

**Example:**

```javascript
const recorder = new VideoRecorder({ fps: 15 });
const video = document.getElementById('preview');

await recorder.start(video, 'user', (base64Data) => {
  // Process video frame
});
```

##### `stop()`

Stops video recording.

##### `checkBrowserSupport()` (static)

Checks if browser supports video recording.

**Returns:** `boolean`

#### ScreenRecorder

Records screen content.

**Import:**

```javascript
import { ScreenRecorder } from './video/screen-recorder.js';
```

**Constructor:**

```javascript
const screenRecorder = new ScreenRecorder({
  fps: 2,
  quality: 0.8,
  width: 1280,
  height: 720
});
```

**Options:**
- `fps` (number): Frames per second (default: 2)
- `quality` (number): JPEG quality 0-1 (default: 0.8)
- `width` (number): Width (default: 1280)
- `height` (number): Height (default: 720)
- `maxFrameSize` (number): Max frame size in bytes (default: 204800)

**Methods:**

##### `start(previewElement, onScreenData)`

Starts screen recording.

**Parameters:**
- `previewElement` (HTMLVideoElement): Preview element
- `onScreenData` (Function): Callback for screen frames

**Returns:** `Promise<void>`

**Example:**

```javascript
const recorder = new ScreenRecorder();
const preview = document.getElementById('screen-preview');

await recorder.start(preview, (base64Data) => {
  client.sendRealtimeInput([{
    mimeType: 'image/jpeg',
    data: base64Data
  }]);
});
```

##### `stop()`

Stops screen recording.

##### `checkBrowserSupport()` (static)

Checks if browser supports screen sharing.

---

### Tools

#### ToolManager

Manages tool registration and execution.

**Import:**

```javascript
import { ToolManager } from './tools/tool-manager.js';
```

**Constructor:**

```javascript
const toolManager = new ToolManager();
```

**Methods:**

##### `registerTool(name, toolInstance)`

Registers a new tool.

**Parameters:**
- `name` (string): Tool name
- `toolInstance` (Object): Tool instance with `getDeclaration()` and `execute()` methods

**Example:**

```javascript
class MyTool {
  getDeclaration() {
    return [{
      name: 'my_tool',
      description: 'Description of my tool',
      parameters: {
        type: 'object',
        properties: {
          param1: { type: 'string', description: 'Parameter 1' }
        },
        required: ['param1']
      }
    }];
  }
  
  async execute(args) {
    return { result: 'data' };
  }
}

toolManager.registerTool('myTool', new MyTool());
```

##### `getToolDeclarations()`

Gets all tool declarations.

**Returns:** `Array<Object>`

##### `handleToolCall(functionCall)`

Handles a tool call from the API.

**Parameters:**
- `functionCall` (Object): Function call object

**Returns:** `Promise<Object>`

#### GoogleSearchTool

Built-in Google Search tool (handled server-side by Gemini).

**Import:**

```javascript
import { GoogleSearchTool } from './tools/google-search.js';
```

#### WeatherTool

Built-in weather forecast tool.

**Import:**

```javascript
import { WeatherTool } from './tools/weather-tool.js';
```

**Methods:**

##### `execute(args)`

Gets weather forecast.

**Parameters:**
- `args.location` (string): Location (city name)
- `args.date` (string): Date (YYYY-MM-DD)

**Returns:** `Promise<Object>`

**Example:**

```javascript
const weatherTool = new WeatherTool();
const forecast = await weatherTool.execute({
  location: 'London',
  date: '2025-10-11'
});
// Returns: { location, date, condition, temperature, humidity, windSpeed, forecast }
```

---

### Utilities

#### Logger

Logging utility with event emission.

**Import:**

```javascript
import { Logger } from './utils/logger.js';
```

**Methods:**

##### `debug(message, data)`

Logs a debug message.

**Parameters:**
- `message` (string): Log message
- `data` (Object): Optional data

**Example:**

```javascript
Logger.debug('Processing frame', { size: 1024 });
```

##### `info(message, data)`

Logs an info message.

##### `warn(message, data)`

Logs a warning message.

##### `error(message, data)`

Logs an error message.

##### `export()`

Exports logs as JSON file.

**Example:**

```javascript
Logger.export(); // Downloads logs-[timestamp].json
```

**Events:**

```javascript
const logger = Logger.getInstance();
logger.on('log', (logEntry) => {
  console.log(logEntry);
});
```

#### Error Handling

**Import:**

```javascript
import { ApplicationError, ErrorCodes } from './utils/error-boundary.js';
```

**ApplicationError Class:**

```javascript
throw new ApplicationError(
  'Error message',
  ErrorCodes.AUDIO_START_FAILED,
  { originalError: error }
);
```

**Error Codes:**

```javascript
ErrorCodes.AUDIO_DEVICE_NOT_FOUND
ErrorCodes.AUDIO_PERMISSION_DENIED
ErrorCodes.AUDIO_NOT_SUPPORTED
ErrorCodes.VIDEO_START_FAILED
ErrorCodes.WEBSOCKET_CONNECTION_FAILED
ErrorCodes.API_AUTHENTICATION_FAILED
// ... and more
```

#### Utility Functions

**Import:**

```javascript
import { blobToJSON, base64ToArrayBuffer } from './utils/utils.js';
```

##### `blobToJSON(blob)`

Converts Blob to JSON.

**Parameters:**
- `blob` (Blob): Blob to convert

**Returns:** `Promise<Object>`

**Example:**

```javascript
const json = await blobToJSON(blob);
```

##### `base64ToArrayBuffer(base64)`

Converts base64 string to ArrayBuffer.

**Parameters:**
- `base64` (string): Base64 string

**Returns:** `ArrayBuffer`

**Example:**

```javascript
const buffer = base64ToArrayBuffer(audioData);
```

---

## Configuration

### CONFIG Object

**Import:**

```javascript
import { CONFIG } from './config/config.js';
```

**Structure:**

```javascript
{
  API: {
    VERSION: 'v1alpha',
    MODEL_NAME: 'models/gemini-2.0-flash-exp'
  },
  SYSTEM_INSTRUCTION: {
    TEXT: 'You are my helpful assistant...'
  },
  AUDIO: {
    SAMPLE_RATE: 16000,
    OUTPUT_SAMPLE_RATE: 24000,
    BUFFER_SIZE: 2048,
    CHANNELS: 1
  }
}
```

**Usage:**

```javascript
// Access configuration
const modelName = CONFIG.API.MODEL_NAME;
const sampleRate = CONFIG.AUDIO.SAMPLE_RATE;

// Modify system instruction
CONFIG.SYSTEM_INSTRUCTION.TEXT = 'Custom instruction';
```

---

## Examples

### Example 1: Basic Chat

```javascript
import { MultimodalLiveClient } from './core/websocket-client.js';

const client = new MultimodalLiveClient();

// Connect
const config = {
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'text'
  },
  systemInstruction: {
    parts: [{ text: 'You are a helpful assistant.' }]
  }
};

await client.connect(config, 'YOUR_API_KEY');

// Listen for responses
client.on('content', (content) => {
  const text = content.modelTurn.parts
    .map(p => p.text)
    .join('');
  console.log('AI:', text);
});

// Send message
client.send('Hello, how are you?');
```

### Example 2: Audio Conversation

```javascript
import { MultimodalLiveClient } from './core/websocket-client.js';
import { AudioRecorder } from './audio/audio-recorder.js';
import { AudioStreamer } from './audio/audio-streamer.js';

const client = new MultimodalLiveClient();
const audioContext = new AudioContext();
const streamer = new AudioStreamer(audioContext);
const recorder = new AudioRecorder();

// Connect with audio support
await client.connect({
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'audio',
    speechConfig: {
      voiceConfig: {
        prebuiltVoiceConfig: { voiceName: 'Aoede' }
      }
    }
  },
  systemInstruction: {
    parts: [{ text: 'You are a helpful voice assistant.' }]
  }
}, 'YOUR_API_KEY');

// Play audio responses
client.on('audio', async (data) => {
  streamer.addPCM16(new Uint8Array(data));
});

// Start recording
await recorder.start((base64Data) => {
  client.sendRealtimeInput([{
    mimeType: 'audio/pcm;rate=16000',
    data: base64Data
  }]);
});

// Stop recording after 5 seconds
setTimeout(() => {
  recorder.stop();
}, 5000);
```

### Example 3: Video Analysis

```javascript
import { MultimodalLiveClient } from './core/websocket-client.js';
import { VideoManager } from './video/video-manager.js';

const client = new MultimodalLiveClient();
const videoManager = new VideoManager();

// Connect
await client.connect({
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'text'
  },
  systemInstruction: {
    parts: [{ text: 'Describe what you see in the video.' }]
  }
}, 'YOUR_API_KEY');

// Handle responses
client.on('content', (content) => {
  const text = content.modelTurn.parts
    .map(p => p.text)
    .join('');
  console.log('Description:', text);
});

// Start video capture
await videoManager.start(1, (frameData) => {
  client.sendRealtimeInput([frameData]);
});

// Ask about the video
client.send('What do you see?');
```

### Example 4: Using Tools

```javascript
import { MultimodalLiveClient } from './core/websocket-client.js';

const client = new MultimodalLiveClient();

await client.connect({
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'text'
  },
  systemInstruction: {
    parts: [{
      text: 'You can use tools to help answer questions.'
    }]
  }
}, 'YOUR_API_KEY');

// Tools are handled automatically by ToolManager
client.on('content', (content) => {
  const text = content.modelTurn.parts
    .map(p => p.text)
    .join('');
  console.log('AI:', text);
});

// Ask a question that requires a tool
client.send('What is the weather in London on 2025-10-15?');
```

### Example 5: Screen Sharing

```javascript
import { MultimodalLiveClient } from './core/websocket-client.js';
import { ScreenRecorder } from './video/screen-recorder.js';

const client = new MultimodalLiveClient();
const screenRecorder = new ScreenRecorder({ fps: 2 });

await client.connect({
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'text'
  },
  systemInstruction: {
    parts: [{ text: 'Analyze the screen content.' }]
  }
}, 'YOUR_API_KEY');

const preview = document.getElementById('screen-preview');

await screenRecorder.start(preview, (base64Data) => {
  client.sendRealtimeInput([{
    mimeType: 'image/jpeg',
    data: base64Data
  }]);
});

client.send('What is displayed on the screen?');
```

### Example 6: Using REST API (curl)

```bash
# List models
curl https://your-domain.com/v1/models \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY"

# Chat completion
curl https://your-domain.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY" \
  -d '{
    "model": "gemini-2.0-flash-exp",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Tell me a joke."}
    ],
    "temperature": 0.7
  }'

# Streaming chat
curl https://your-domain.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY" \
  -d '{
    "model": "gemini-2.0-flash-exp",
    "messages": [
      {"role": "user", "content": "Count to 10"}
    ],
    "stream": true
  }'

# Embeddings
curl https://your-domain.com/v1/embeddings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_GEMINI_API_KEY" \
  -d '{
    "model": "text-embedding-004",
    "input": "Hello world"
  }'
```

### Example 7: Using REST API (Python)

```python
import openai

# Configure client
client = openai.OpenAI(
    api_key="YOUR_GEMINI_API_KEY",
    base_url="https://your-domain.com/v1"
)

# List models
models = client.models.list()
print(models)

# Chat completion
response = client.chat.completions.create(
    model="gemini-2.0-flash-exp",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is AI?"}
    ]
)
print(response.choices[0].message.content)

# Streaming chat
stream = client.chat.completions.create(
    model="gemini-2.0-flash-exp",
    messages=[{"role": "user", "content": "Count to 5"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end='')

# Embeddings
embeddings = client.embeddings.create(
    model="text-embedding-004",
    input="Hello world"
)
print(embeddings.data[0].embedding[:10])  # First 10 dimensions
```

### Example 8: Custom Tool

```javascript
import { ToolManager } from './tools/tool-manager.js';
import { MultimodalLiveClient } from './core/websocket-client.js';

// Define custom tool
class CalculatorTool {
  getDeclaration() {
    return [{
      name: 'calculate',
      description: 'Performs mathematical calculations',
      parameters: {
        type: 'object',
        properties: {
          expression: {
            type: 'string',
            description: 'Mathematical expression to evaluate'
          }
        },
        required: ['expression']
      }
    }];
  }
  
  async execute(args) {
    try {
      // Simple evaluation (use a proper math parser in production)
      const result = eval(args.expression);
      return { result, expression: args.expression };
    } catch (error) {
      throw new Error(`Invalid expression: ${error.message}`);
    }
  }
}

// Register tool
const client = new MultimodalLiveClient();
client.toolManager.registerTool('calculator', new CalculatorTool());

// Use with client
await client.connect({
  model: 'models/gemini-2.0-flash-exp',
  generationConfig: {
    responseModalities: 'text'
  },
  systemInstruction: {
    parts: [{
      text: 'You can use the calculator tool to perform math.'
    }]
  }
}, 'YOUR_API_KEY');

client.send('What is 25 * 47?');
```

---

## Best Practices

### Error Handling

Always wrap API calls in try-catch blocks:

```javascript
try {
  await client.connect(config, apiKey);
} catch (error) {
  if (error instanceof ApplicationError) {
    console.error(`${error.code}: ${error.message}`);
  } else {
    console.error('Unexpected error:', error);
  }
}
```

### Resource Management

Always clean up resources:

```javascript
// Stop recording before disconnecting
if (audioRecorder) {
  audioRecorder.stop();
}

if (videoManager) {
  videoManager.stop();
}

// Disconnect client
client.disconnect();
```

### Performance Optimization

For video, use motion detection to reduce bandwidth:

```javascript
const videoManager = new VideoManager();
// VideoManager automatically uses motion detection
// Only significant frames are sent
```

Adjust FPS based on use case:
- High FPS (15-30): Interactive applications, gaming
- Medium FPS (5-10): General video analysis
- Low FPS (1-2): Static content, screen sharing

### Security

- Never expose API keys in client-side code
- Use environment variables for sensitive data
- Implement rate limiting on server side
- Validate all user inputs

---

## Troubleshooting

### WebSocket Connection Issues

```javascript
client.on('error', (error) => {
  console.error('Connection error:', error);
  // Retry logic
  setTimeout(() => {
    client.connect(config, apiKey);
  }, 5000);
});
```

### Audio Issues

```javascript
// Ensure AudioContext is resumed
const audioContext = new AudioContext();

// Resume on user interaction
document.addEventListener('click', async () => {
  if (audioContext.state === 'suspended') {
    await audioContext.resume();
  }
}, { once: true });
```

### Video Permission Errors

```javascript
try {
  await videoManager.start(fps, onFrame);
} catch (error) {
  if (error.code === ErrorCodes.VIDEO_PERMISSION_DENIED) {
    console.error('Camera permission denied');
    // Show user instructions to enable camera
  }
}
```

---

## Support

For issues and questions:
- GitHub: https://github.com/tech-shrimp/gemini-playground
- Documentation: See README.md
- Examples: See examples directory

---

## License

MIT License - See LICENSE file for details
