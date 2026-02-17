# GistPin Mobile

The mobile client for GistPin — a location-first interface for posting and browsing hyperlocal gists on iOS and Android.

---

## What This Repo Does

- Detects the user's current location and shows nearby gists on a map and list view
- Lets users post a new gist at their current (or pinned) location
- Communicates exclusively with `gistpin-backend` — no direct blockchain calls from the app

This app is a **pure client**. All IPFS pinning and Soroban transaction logic lives in the backend.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | React Native via **Expo** (SDK 51+) |
| Language | TypeScript |
| Navigation | Expo Router (file-based, built on React Navigation) |
| Map | `react-native-maps` (via `expo-maps` or direct) |
| Data fetching | TanStack Query (React Query v5) |
| HTTP client | Axios |
| Forms | React Hook Form + Zod |
| Location | `expo-location` |
| Storage (local) | `expo-secure-store` / `@react-native-async-storage/async-storage` |
| Testing | Jest + React Native Testing Library |

---

## Project Layout

```
gistpin-mobile/
├── app/                         # Expo Router file-based routing
│   ├── _layout.tsx              # Root layout (providers, navigation)
│   ├── index.tsx                # Home screen — shows MapScreen
│   ├── (tabs)/
│   │   ├── _layout.tsx          # Tab navigator layout
│   │   ├── map.tsx              # Map tab
│   │   └── list.tsx             # List tab (nearby gists)
│   └── gist/
│       └── [id].tsx             # Gist detail screen
├── components/
│   ├── map/
│   │   ├── GistMap.tsx          # MapView with gist markers
│   │   └── GistMapMarker.tsx    # Individual marker component
│   ├── gists/
│   │   ├── GistCard.tsx         # Single gist row / card
│   │   └── CreateGistSheet.tsx  # Bottom sheet for creating a gist
│   └── ui/                      # Shared primitives (Button, Input, etc.)
├── lib/
│   ├── api/
│   │   └── client.ts            # Typed Axios instance for gistpin-backend
│   ├── hooks/
│   │   ├── useGists.ts          # TanStack Query hooks
│   │   └── useCurrentLocation.ts # expo-location wrapper
│   └── utils/
│       └── geo.ts               # Geo helpers
├── types/
│   └── gist.ts                  # Shared TypeScript types (mirror of web)
├── constants/
│   └── config.ts                # API base URL, map defaults, etc.
├── assets/                      # Images, fonts, icons
├── .env.example                 # Environment template
├── app.json                     # Expo config
├── app.config.ts                # Dynamic Expo config (reads .env)
├── babel.config.js
├── tsconfig.json
├── package.json
└── README.md
```

---

## Prerequisites

- **Node.js** >= 20 — [nodejs.org](https://nodejs.org)
- **pnpm** (recommended) or npm/yarn
- **Expo Go** app on your phone — [iOS](https://apps.apple.com/app/expo-go/id982107779) / [Android](https://play.google.com/store/apps/details?id=host.exp.exponent)
- OR an iOS Simulator (Xcode, macOS only) / Android Emulator (Android Studio)
- A running instance of `gistpin-backend` (see that repo's README)

---

## Local Development

### 1. Clone and install

```bash
git clone https://github.com/gistpin/gistpin-mobile.git
cd gistpin-mobile
pnpm install
```

### 2. Environment variables

```bash
cp .env.example .env
```

Open `.env` and fill in:

```env
# For emulator on the same machine as backend:
API_BASE_URL=http://10.0.2.2:3000   # Android emulator
# API_BASE_URL=http://localhost:3000  # iOS simulator

# For a real device on the same LAN as your backend:
# API_BASE_URL=http://192.168.x.x:3000

# Default map center (lon, lat)
DEFAULT_LAT=5.6037
DEFAULT_LNG=-0.1870
```

> **Tip:** On Android Emulator, `localhost` on your machine maps to `10.0.2.2`. On iOS Simulator, `localhost` works directly.

### 3. Start the Expo dev server

```bash
pnpm start
```

Then:
- Press **`a`** to open in Android Emulator
- Press **`i`** to open in iOS Simulator
- Scan the QR code with **Expo Go** on a real device

---

## Key Patterns

### API client

All network calls go through `lib/api/client.ts`. Never call `fetch` or `axios` directly from components.

```ts
// lib/api/client.ts
import axios from 'axios';
import { API_BASE_URL } from '@/constants/config';
import type { Gist, CreateGistDto, GistQueryParams, GistsResponse } from '@/types/gist';

export const api = axios.create({ baseURL: API_BASE_URL });

export const gistsApi = {
  getNearby: (params: GistQueryParams) =>
    api.get<GistsResponse>('/gists', { params }),

  create: (data: CreateGistDto) =>
    api.post<Gist>('/gists', data),
};
```

### Location permissions

Always request permission before accessing location. Expo handles this cleanly:

```ts
// lib/hooks/useCurrentLocation.ts
import * as Location from 'expo-location';
import { useEffect, useState } from 'react';

export function useCurrentLocation() {
  const [location, setLocation] = useState<Location.LocationObject | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    (async () => {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        setError('Location permission denied');
        return;
      }
      const loc = await Location.getCurrentPositionAsync({});
      setLocation(loc);
    })();
  }, []);

  return { location, error };
}
```

### Map with gist markers

```tsx
// components/map/GistMap.tsx (simplified)
import MapView, { Marker } from 'react-native-maps';
import { useGists } from '@/lib/hooks/useGists';
import { useCurrentLocation } from '@/lib/hooks/useCurrentLocation';

export default function GistMap() {
  const { location } = useCurrentLocation();
  const lat = location?.coords.latitude ?? 5.6037;
  const lon = location?.coords.longitude ?? -0.1870;
  const { data } = useGists(lat, lon);

  return (
    <MapView
      style={{ flex: 1 }}
      initialRegion={{ latitude: lat, longitude: lon, latitudeDelta: 0.01, longitudeDelta: 0.01 }}
    >
      {data?.data.map(gist => (
        <Marker
          key={gist.gistId}
          coordinate={{ latitude: gist.lat, longitude: gist.lon }}
          title={gist.text.slice(0, 60)}
        />
      ))}
    </MapView>
  );
}
```

---

## Expo Router Notes

This app uses **Expo Router** (file-based routing). The `app/` directory maps directly to screens:

| File | Route / Screen |
|---|---|
| `app/index.tsx` | Initial screen (redirects to map tab) |
| `app/(tabs)/map.tsx` | Map tab |
| `app/(tabs)/list.tsx` | List tab |
| `app/gist/[id].tsx` | Gist detail — navigated to via `router.push('/gist/42')` |

---

## Building for Production

Expo uses EAS (Expo Application Services) for production builds:

```bash
# Install EAS CLI
npm install -g eas-cli

# Log in to Expo
eas login

# Build for Android
eas build --platform android

# Build for iOS
eas build --platform ios
```

For dev/internal testing, use `eas build --profile preview` which skips store submission.



## Contribution Guidelines

- Keep screen components focused — no direct API calls inside screen files.
- Centralise all networking in `lib/api/`.
- Use TanStack Query for all remote data (no raw `useEffect` + `setState` for fetching).
- When changing the API contract, coordinate with `gistpin-backend` first.

For global contribution rules: [gistpin-meta/CONTRIBUTING.md](https://github.com/gistpin/gistpin-meta/blob/main/CONTRIBUTING.md)
