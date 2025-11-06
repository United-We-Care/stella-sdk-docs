# 🧩 UWC Chat SDK – Web Version (Development Guide)

This document provides a **structured guide** to build the **web-compatible version** of the UWC Chat SDK (`@united-we-care/stella-web-client-sdk`) as a standalone package.  
It mirrors the architecture of the React Native SDK but uses **browser-native APIs** for crypto, storage, and device information.

---

## 🏗️ Project Structure

```bash
stella-sdk-web/
├── src/
│   ├── managers/             # AuthManager, ChatManager, AvatarManager, RecommendationManager
│   ├── websocket/            # WebSocketManager (connection, events, heartbeat, reconnect)
│   ├── utils/                # logger, jwt, jwtGenerator, deviceInfo, secureSessionManager, apiService
│   ├── platform/             # Web platform abstraction (storage, crypto, device, file)
│   ├── types/                # Shared TypeScript types
│   ├── UWCChatSDK.ts         # Main SDK orchestrator (singleton)
│   └── index.ts              # Public exports
├── public/
│   └── demo.html             # Demo page
├── dist/                     # Build outputs (ESM/UMD/types)
├── docs/                     # Guides
├── package.json
├── tsconfig.json
├── rollup.config.js
└── README.md
```

---

## ⚙️ 1. Package Configuration (`package.json`)

```json
{
  "name": "@united-we-care/stella-web-client-sdk",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.umd.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "dev": "vite",
    "build": "rollup -c",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "preview": "vite preview",
    "lint": "eslint src/**/*.ts",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "tslib": "^2.8.1"
  },
  "devDependencies": {
    "axios": "^1.6.0",
    "@rollup/plugin-terser": "^0.4.4",
    "@rollup/plugin-typescript": "^11.1.2",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.0.0",
    "jsdom": "^27.0.0",
    "rollup": "^4.13.0",
    "typescript": "^5.4.2",
    "vite": "^5.2.0",
    "vitest": "^1.4.0"
  },
  "peerDependencies": {
    "axios": "^1.6.0",
    "react": ">=18.0.0"
  }
}
```

---

## 🧠 2. Platform Abstraction Layer (`src/platform/index.ts`)

Browser-native wrappers used by managers (storage/crypto/device/file).

```ts
export interface PlatformStorage {
  getItem(key: string): Promise<string | null> /* ... */;
}
export interface PlatformCrypto {
  sha256(input: string): Promise<string>;
  randomBytes(n: number): Uint8Array;
}
export interface PlatformDevice {
  locale: string;
  languages: string[];
  userAgent: string;
  platform: string;
  timeZone: string;
  screen: { width: number; height: number; colorDepth: number };
  online: boolean;
}
export interface PlatformFile {
  createObjectURL(file: File): string /* ... */;
}

export const platform = {
  storage: {
    /* localStorage get/set/remove/clear with try/catch */
  },
  crypto: {
    /* SHA-256 via Web Crypto; randomBytes via getRandomValues */
  },
  device: {
    /* getters for locale, languages, userAgent, platform, timeZone, screen, online */
  },
  file: {
    /* createObjectURL, revokeObjectURL, FileReader helpers */
  },
};
```

---

## 🌐 3. WebSocket Manager (`src/websocket/WebSocketManager.ts`)

Singleton with `connect(jwtToken, websocketUrl?)`, `setEventHandlers`, `sendMessage`, heartbeat ping, and exponential backoff reconnect.

```ts
const ws = WebSocketManager.getInstance(config);
ws.setEventHandlers({
  onConnected: (sessionId) => {
    /* ... */
  },
  onMessage: (msg) => {
    /* ... */
  },
  onConnectionChange: (ok) => {
    /* ... */
  },
  onError: (err) => {
    /* ... */
  },
});
await ws.connect(jwtToken, optionalCustomUrl);
ws.sendMessage({ type: "request" /* ... */ });
```

---

## 🔐 4. Auth & Secure Session (`src/managers/AuthManager.ts`, `src/utils/secureSessionManager.ts`)

```ts
// AuthManager.createUserSession(request)
const response = await axios.post(`${baseUrl || default}/api/partners/session`, request, {
  headers: { 'Content-Type': 'application/json', 'x-partner-api-key': partnerApiKey }
});
// Build User and persist via SecureSessionManager
await secureSession.storeSessionData({ userId, partnerId, partnerUserId, email, firstName, lastName, sessionToken, expiresAt });
```

```ts
// SecureSessionManager (AES-GCM encryption via Web Crypto)
await platform.storage.setItem(
  "uwc_session_data",
  await encrypt(JSON.stringify(localUserData))
);
const data = JSON.parse(
  await decrypt(await platform.storage.getItem("uwc_session_data")!)
);
```

---

## 🧩 5. Entry Point (`src/index.ts`)

```ts
export { UWCChatSDK } from "./UWCChatSDK";
export {
  AuthManager,
  ChatManager,
  AvatarManager,
  RecommendationManager,
} from "./managers";
export { WebSocketManager } from "./websocket/WebSocketManager";
export {
  Logger,
  JWTManager,
  JWTGenerator,
  DeviceInfoManager,
  SecureSessionManager,
  APIService,
} from "./utils";
export { platform } from "./platform";
export type {} from /* device, chat, avatar, recommendation, file, api, error types */ "./types";
```

---

## 📦 6. Rollup Build Config (`rollup.config.js`)

```js
import typescript from "@rollup/plugin-typescript";
import terser from "@rollup/plugin-terser";

export default {
  input: "src/index.ts",
  output: [
    {
      file: "dist/index.esm.js",
      format: "esm",
      exports: "named",
      sourcemap: true,
    },
    {
      file: "dist/index.umd.js",
      format: "umd",
      name: "UWCChatSDK",
      exports: "named",
      globals: { axios: "axios" },
      sourcemap: true,
    },
  ],
  plugins: [typescript({ tsconfig: "./tsconfig.json" }) /* , terser() */],
  external: ["axios"],
};
```

---

## 🧰 7. TypeScript Config (`tsconfig.json`)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "Node",
    "lib": ["DOM", "ES2020"],
    "outDir": "dist",
    "declaration": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

---

## 🧪 8. Local Testing (Optional)

```bash
npm run build
npm run preview
```

Then open `public/demo.html` to test the full flow with the built ESM bundle.

---

## 🚀 9. Future Migration to Monorepo

When the Web SDK stabilizes, merge into a single `uwc-chat-sdk/` repo:

```
uwc-chat-sdk/
├── packages/
│   ├── core/
│   ├── rn-adapter/
│   └── web-adapter/
```

**Migration steps:**

1. Move reusable logic from `src/` → `packages/core/`
2. Keep browser-specific code in `packages/web-adapter/`
3. Add `"exports"` in root `package.json`:
   ```json
   {
     "exports": {
       ".": {
         "react-native": "./packages/rn-adapter/dist/index.js",
         "browser": "./packages/web-adapter/dist/index.js",
         "default": "./packages/core/dist/index.js"
       }
     }
   }
   ```

---

## ✅ Key Development Principles

| Area             | Rule                                            |
| ---------------- | ----------------------------------------------- |
| **Structure**    | Match RN SDK layout exactly                     |
| **Dependencies** | Avoid React, use browser APIs                   |
| **Storage**      | Secure AES-GCM in localStorage (Web Crypto)     |
| **Crypto**       | Use `crypto.subtle`                             |
| **Device Info**  | Use `navigator`, `Intl` via `DeviceInfoManager` |
| **WebSocket**    | Native browser WebSocket                        |
| **Testing**      | Use Vitest (unit) + Playwright (integration)    |
| **Build**        | Rollup ESM + UMD output (axios external)        |

---

## 🧭 Summary

- Start with **`@uwc/chat-sdk-web`** as an independent package
- Keep structure **symmetrical** with React Native SDK
- Abstract platform differences early
- Migrate easily to **monorepo** when ready

---

> 🧑‍💻 **Cursor Tip:**  
> You can use this markdown as a **Cursor Project Context** file.
>
> - Open this file in Cursor → "Add as Context"
> - Then ask:
>   > “Refactor AuthManager for token refresh flow”  
>   > or  
>   > “Add reconnect logic with exponential backoff in WebSocketManager”

---
