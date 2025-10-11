# Quick Start Guide

Get up and running with Gemini 2.0 Playground in 5 minutes!

## 🚀 5-Minute Setup

### Step 1: Get a Gemini API Key (30 seconds)

1. Go to [Google AI Studio](https://aistudio.google.com)
2. Sign in with your Google account
3. Click "Get API Key"
4. Copy your API key

### Step 2: Deploy (2 minutes)

Choose your preferred deployment method:

#### Option A: Deno Deploy (Recommended)

1. Fork [the repository](https://github.com/tech-shrimp/gemini-playground/fork)
2. Go to [Deno Deploy](https://dash.deno.com/new_project)
3. Select your forked repo
4. Set entrypoint: `src/deno_index.ts`
5. Click "Deploy Project"
6. Done! Your app is live at `https://YOUR_PROJECT_NAME.deno.dev`

#### Option B: Local Development

```bash
# Install Deno
curl -fsSL https://deno.land/install.sh | sh

# Run
deno run --allow-net --allow-read https://raw.githubusercontent.com/tech-shrimp/gemini-playground/main/src/deno_index.ts
```

Open http://localhost:8000

### Step 3: Start Using (1 minute)

1. Open your deployed URL
2. Paste your API key in the input field
3. Click "Connect"
4. Start chatting!

## 🎤 Using Audio Features

1. Click "Connect" to establish connection
2. Click the microphone button (🎙️)
3. Allow microphone access when prompted
4. Start speaking - the AI will respond with voice!

## 📹 Using Video Features

1. Click the camera button (📹)
2. Allow camera access
3. The AI can now see what you're showing!

**Try asking:**
- "What do you see?"
- "What color is this?"
- "Can you read this text?"

## 🖥️ Using Screen Sharing

1. Click the screen share button (🖥️)
2. Select which screen/window to share
3. The AI can now see your screen!

**Try asking:**
- "What's on my screen?"
- "Help me debug this code"
- "Review this design"

## 💬 Basic Chat Examples

### Simple Conversation

```
You: Hello!
AI: Hello! How can I help you today?

You: What can you do?
AI: I can chat with you, answer questions, help with tasks, 
    and even see and hear you! Try enabling your camera or 
    microphone to have a multimodal conversation.
```

### Using Tools

The AI can use built-in tools automatically:

```
You: What's the weather in London on October 15th?
AI: Let me check that for you...
    [Uses weather tool]
    The weather in London on October 15th will be 
    partly cloudy with a temperature of 18°C.
```

```
You: Search for the latest news about AI
AI: [Uses Google Search]
    Here are the latest developments in AI...
```

## ⚙️ Configuration

### Available Voices

Click the settings icon (⚙️) to access configuration.

Available voice options:
- **Aoede** (Default)
- **Charon**
- **Fenrir**
- **Kore**
- **Puck**

### Response Type

- **Audio**: AI responds with voice
- **Text**: AI responds with text only
- **Audio + Text**: Both voice and text

### Video Frame Rate

Adjust based on your needs:
- **1 FPS**: Basic, low bandwidth
- **5 FPS**: Balanced (recommended)
- **15 FPS**: Smooth, high bandwidth

### System Instruction

Customize how the AI behaves:

**Examples:**

```
"You are a helpful coding assistant specialized in Python."

"You are a friendly teacher who explains concepts simply."

"You are a professional translator between English and Spanish."
```

## 📱 Mobile Usage

The app is fully mobile-responsive!

### On iOS/Android:

1. Open the URL in your mobile browser
2. Add to home screen for app-like experience
3. Use front/back camera toggle button
4. Tap and hold chat messages to copy

### Mobile Tips:

- Use headphones for better audio quality
- Ensure good lighting for video
- Grant camera/microphone permissions
- Works best in landscape mode

## 🔍 Common Use Cases

### 1. Voice Assistant

```javascript
// Enable microphone and audio response
1. Connect to API
2. Click microphone button
3. Speak naturally
4. AI responds with voice
```

### 2. Visual Q&A

```javascript
// Show objects to camera
1. Enable camera
2. Point at object
3. Ask "What is this?"
4. AI describes what it sees
```

### 3. Code Review

```javascript
// Share your code screen
1. Enable screen share
2. Show your code
3. Ask for review
4. AI provides feedback
```

### 4. Language Practice

```javascript
// Practice speaking
1. Enable microphone
2. Set system instruction: "You are a language tutor"
3. Speak in target language
4. Get corrections and feedback
```

### 5. Real-time Translation

```javascript
// Translate conversations
1. Enable microphone
2. Set instruction: "Translate to [language]"
3. Speak
4. Get instant translation
```

## 🎯 Pro Tips

### Better Audio Quality

```
✅ Use headphones to prevent echo
✅ Speak clearly and not too fast
✅ Reduce background noise
✅ Check microphone permissions
```

### Better Video Quality

```
✅ Ensure good lighting
✅ Hold camera steady
✅ Get close to objects
✅ Reduce frame rate if laggy
```

### Faster Responses

```
✅ Use text-only mode
✅ Disable video when not needed
✅ Lower video frame rate
✅ Use stable internet connection
```

### Privacy

```
✅ Your API key is stored locally
✅ No data is logged on server
✅ Audio/video sent only when enabled
✅ Clear browser data to remove API key
```

## 🔧 Troubleshooting

### Connection Issues

```
Problem: "Could not connect" error
Solution:
1. Check API key is correct
2. Check internet connection
3. Try refreshing page
4. Check browser console for errors
```

### No Audio

```
Problem: Can't hear AI responses
Solution:
1. Check microphone permission
2. Ensure response type includes "audio"
3. Check browser volume
4. Try different browser
```

### No Video

```
Problem: Camera not working
Solution:
1. Check camera permission
2. Ensure HTTPS is used
3. Close other apps using camera
4. Try different browser
```

### Slow Performance

```
Problem: Laggy or slow
Solution:
1. Lower video frame rate
2. Disable video if not needed
3. Use text-only mode
4. Check internet speed
```

## 📚 Learning Resources

### Explore Features

1. **Audio Conversation**
   - Try different voices
   - Experiment with languages
   - Test interruption (start speaking during AI response)

2. **Video Analysis**
   - Show different objects
   - Test OCR (text reading)
   - Try facial expressions

3. **Screen Sharing**
   - Share code for review
   - Show presentations
   - Demonstrate problems

4. **Tools**
   - Ask about weather
   - Request web searches
   - Try calculations

### Advanced Usage

Check the [API Documentation](API.md) for:
- JavaScript API reference
- Custom tool creation
- Advanced configuration
- Integration examples

## 🚦 Next Steps

Now that you're up and running:

1. **Explore Features** - Try all the buttons!
2. **Customize** - Set your preferred voice and system instruction
3. **Integrate** - Use the REST API in your apps
4. **Extend** - Add custom tools and features
5. **Share** - Deploy your own customized version

## 💡 Example Prompts to Try

### Creative

```
"Write a short story about a robot"
"Generate creative names for a coffee shop"
"Help me brainstorm blog post ideas"
```

### Analytical

```
"Explain quantum computing simply"
"What are the pros and cons of solar energy?"
"Compare Python and JavaScript"
```

### Interactive

```
"Let's play 20 questions"
"Quiz me on world capitals"
"Help me practice my presentation"
```

### Practical

```
"Help me write a professional email"
"Review my resume (via screen share)"
"Suggest healthy meal ideas"
```

## 📞 Getting Help

- **Documentation**: Check [API.md](API.md) for detailed docs
- **Issues**: [GitHub Issues](https://github.com/tech-shrimp/gemini-playground/issues)
- **Examples**: See examples in documentation

## 🎉 You're Ready!

You now know the basics of using Gemini 2.0 Playground. Start experimenting and discover what's possible with multimodal AI!

---

**Happy chatting! 🚀**
