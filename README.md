# UWC Chat SDK Web

A comprehensive web-compatible SDK for integrating AI-powered chat functionality into React and vanilla JavaScript applications with enterprise-grade security and privacy protection.

## 🚀 Quick Start

### Installation

You can install the SDK from GitHub Packages (private feeds).

#### GitHub Packages (private)

1. Create a `.npmrc` in your project root (and/or in your user home) with the scope registry and token:

```ini
@united-we-care:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
always-auth=true
```

2. Provide `GITHUB_TOKEN` (or `NPM_TOKEN`) with at least `read:packages` scope.

- Local development (macOS/Linux):

```bash
export GITHUB_TOKEN=ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXX
npm install @united-we-care/stella-web-client-sdk@latest
```

- CI (GitHub Actions):

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

steps:
  - uses: actions/checkout@v4
  - name: Install deps
    run: npm ci
```

### Basic Usage

```typescript
import { UWCChatSDK } from "@united-we-care/stella-web-client-sdk";

// 1) Initialize the SDK (singleton)
const sdk = await UWCChatSDK.initialize({
  partnerApiKey: "your-partner-api-key",
  environment: "prod", // "prod" | "stage" | "dev"
  baseUrl: "https://cauth.unitedwecare.com", // optional override
  enableLogging: true,
});

// 2) Create a user session (register/login)
const user = await sdk.createUserSession({
  partnerUserId: "external-user-id",
  email: "user@example.com",
  firstName: "Jane",
  lastName: "Doe",
  country: "United States",
});

// 3) (Optional) Fetch avatars/personas and select one
await sdk.fetchAvatarsData();
const avatars = sdk.getAvailableAvatars();
if (avatars.length) sdk.setAvatar(avatars[0]);

// 4) Wire up chat event handlers
sdk.setEventHandlers({
  onConnected: (sessionId) => console.log("Connected:", sessionId),
  onMessage: (message) => console.log("New message:", message),
  onConnectionChange: (connected) => console.log("WS:", connected),
  onRecommendations: (recs) => console.log("Recs:", recs),
  onSmartSuggestions: (sugs) => console.log("Suggestions:", sugs),
  onThinking: (cog) => console.log("Thinking:", cog?.heading),
  addSelfMessage: (msg) => console.log("You:", msg.message),
  onError: (err) => console.error("Chat error:", err),
});

// 5) Connect and chat
await sdk.startChat();
await sdk.sendMessage("Hello!");
```

## 🏗️ Architecture

The web SDK is built with a modular architecture that mirrors the React Native version:

- **Core SDK**: Main orchestrator class
- **Managers**: Specialized modules for auth, chat, avatars, and recommendations
- **WebSocket**: Real-time communication layer
- **Platform Abstraction**: Browser-native API implementations
- **Utilities**: Logging, JWT, device info, and secure storage

## 🔒 Privacy & Security

The SDK is designed for minimal, transparent data handling and secure, browser-side storage of session information.

### What the SDK stores (locally, encrypted)

- ✅ Session data in encrypted storage (AES‑GCM via Web Crypto):
  - `userId`, `partnerId`, `partnerUserId`
  - `email`, `firstName`, `lastName`
  - `sessionToken`, `expiresAt`
  - Stored under the key `uwc_session_data`, never in cookies
- ✅ In‑memory runtime state (not persisted across reloads):
  - Chat history for the current session
  - Selected avatar/persona

### What the SDK sends to UWC services

- ✅ Session creation (HTTPS): `partnerUserId`, `email`, `firstName`, `lastName`
- ✅ WebSocket (WSS) metadata includes device info (locale, languages, userAgent, platform, timeZone, screen, online) and selected avatar for proper routing and UX
- ✅ Chat content (messages, options, suggestions) is sent over WSS; not cached locally beyond in‑memory state

### What the SDK does NOT store or send

- ❌ No passwords or payment data
- ❌ No third‑party analytics/trackers
- ❌ No persistent storage of chat transcripts beyond current runtime memory
- ❌ No cross‑site cookies; tokens are never placed in cookies

### Cryptography & storage details

- AES‑GCM encryption via Web Crypto API for session payloads
- A non‑exported key is derived at runtime; encrypted blob stored in `localStorage` (through a platform abstraction)
- Session validity is enforced using token `expiresAt` and an SDK safety window (~24h), after which session data is cleared

### Operational guidance

- Set `enableLogging: false` in production to avoid verbose console logs
- Treat `partnerUserId` as a stable pseudonymous identifier
- To clear local data, call `sdk.destroy()` which disconnects chat and wipes encrypted session storage

## 🎯 Key Features

- 🤖 **Real-time AI Chat** - WebSocket-based communication
- 👤 **Avatar Management** - Multiple AI personalities
- 📋 **Smart Recommendations** - Personalized content suggestions
- 🔐 **Secure Session** - Partner key-based registration and encrypted local storage
- 🔒 **Privacy by Design** - No cookies; session data encrypted at rest in browser
- 🌐 **Web Ready** - Built for modern browsers
- 🎯 **TypeScript Support** - Full type safety
- 📁 **File Upload** - Support for images and documents
- 🔄 **Auto-reconnection** - Automatic reconnection handling

## 📖 API Reference

### Core Methods

- `initialize(config)` - Initialize the SDK singleton
- `createUserSession(request)` - Register/authenticate user and store session
- `refreshSession()` - Refresh the stored session
- `loadExistingSession()` - Restore cached session if valid
- `isAuthenticated()` / `isSessionValid()` - Session state helpers
- `getCurrentUser()` - Get current authenticated user
- `startChat()` - Open WebSocket connection
- `sendMessage(text, files?)` - Send a chat message
- `sendOptionsClick(option, options)` - Send option selection
- `sendSmartSuggestion(suggestion)` - Send smart suggestion
- `sendRecommendation(recommendation)` - Handle recommendation click
- `setEventHandlers(handlers)` - Register chat event handlers
- `getChatState()` / `getChatHistory()` / `clearChatHistory()`
- `updateDeviceInfo(country?, timeZone?)`
- `setLoggingEnabled(enabled)`
- `disconnect()` / `destroy()`

### Avatar Management

- `getAvailableAvatars()` - Get all available avatars
- `getSelectedAvatar()` - Get current avatar
- `setAvatar(avatar)` - Set selected avatar (also informs chat)
- `fetchAvatarsData()` - Fetch Avatars data for current user
- `refreshAvatarsData()` - Refresh Avatars data for current user

### Recommendations

- `getUserAssessments(userId, page, pageSize)` - Get user assessments
- `getUserPrograms(userId, country, page, pageSize)` - Get user programs

## 🛠️ Development

### Building the SDK

```bash
# Install dependencies
npm install

# Build the SDK
npm run build

# Development mode
npm run dev

# Run tests
npm test

# Preview demo
npm run preview
```

### Project Structure

```
src/
├── managers/           # Business logic modules
├── websocket/          # WebSocket handling
├── utils/              # Utility functions
├── platform/           # Web platform abstraction
├── types/              # TypeScript definitions
└── UWCChatSDK.ts      # Main SDK class
```

## 🌐 Browser Support

- **Modern Browsers**: Chrome 80+, Firefox 75+, Safari 13+, Edge 80+
- **Web APIs**: Web Crypto API, WebSocket, LocalStorage
- **ES2020**: Modern JavaScript features

## 📦 Bundle Information

- **ESM**: `dist/index.esm.js` - Modern module format
- **UMD**: `dist/index.umd.js` - Universal module definition
- **Types**: `dist/index.d.ts` - TypeScript definitions
- **Size**: ~50KB gzipped (without dependencies)

## 🧪 Testing

The SDK includes comprehensive testing:

```bash
# Run unit tests
npm test

# Run tests with UI
npm run test:ui

# Type checking
npm run type-check
```


## 🔄 Migration from React Native SDK

If you're migrating from the React Native SDK, the API is nearly identical:

```typescript
// React Native SDK
import { UWCChatSDK } from "@uwc/chat-sdk";

// Web SDK
import { UWCChatSDK } from "@united-we-care/stella-web-client-sdk";

// Same API, different platform!
```

## 🚀 Deployment

### Production Checklist

- [ ] **API keys** secured
- [ ] **Environment** configured
- [ ] **Logging** disabled in production
- [ ] **Error handling** implemented
- [ ] **Performance** optimized
- [ ] **Security** validated

### Environment Variables (example)

```bash
# Production
PARTNER_API_KEY=your-partner-api-key
BASE_URL=https://cauth.unitedwecare.com
ENVIRONMENT=prod

# Development
PARTNER_API_KEY=your-dev-partner-api-key
BASE_URL=https://cauth.unitedwecare.com
ENVIRONMENT=dev
```

---

**Built with ❤️ by the United We Care team**
