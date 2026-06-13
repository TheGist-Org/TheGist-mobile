# TheGist Mobile

Native iOS and Android app for the TheGist protocol — location-first Web3 client for posting and browsing anonymous hyperlocal gists.

Connect via Freighter mobile or WalletConnect. Post gists tied to your GPS location. Browse what people are saying around you right now.

---

## How It Works

The mobile app follows the same trustless flow as the web client — no server mediates your identity or your posts.

```
1. Connect wallet via Freighter mobile or WalletConnect v2
2. Compose a gist at your current GPS location
3. Content pinned to IPFS → CID returned
4. Transaction (CID + geohash + author pubkey) signed by wallet
5. Signed transaction submitted directly to Soroban GistRegistry
6. Map pulls nearby gists from GraphQL indexer (TheGist-API)
7. Gist appears on the map for anyone in range
```

---

## Stack

| Layer | Choice |
|-------|--------|
| Framework | Expo SDK 51 |
| Language | TypeScript |
| Navigation | Expo Router |
| Map | Mapbox React Native |
| Wallet | WalletConnect v2 / Freighter mobile |
| Blockchain | Stellar SDK |
| Data fetching | Apollo Client |
| GraphQL | TheGist-API |

---

## Key Features

- **GPS-based gist discovery** — the map auto-centers on your location and loads gists within configurable radius
- **Camera integration** — attach images to gists; photos are stored on IPFS, only the CID goes on-chain
- **Push notifications** — opt in to notifications for new gists near saved locations
- **Offline map caching** — map tiles cached locally so the map loads even with a weak connection
- **Anonymous by default** — no username or profile; author Stellar address visible on-chain only
- **24h expiry** — gists expire automatically; the map reflects current on-chain state

---

## Local Setup

### Prerequisites

- Node.js >= 20
- [Expo CLI](https://docs.expo.dev/get-started/installation/): `npm install -g expo-cli`
- iOS Simulator (Xcode, macOS only) or Android Emulator (Android Studio)
- OR [Expo Go](https://expo.dev/go) on a physical device for quick iteration

### 1. Clone and install

```bash
git clone https://github.com/TheGist-Org/TheGist-mobile.git
cd TheGist-mobile
npm install
```

### 2. Environment variables

```bash
cp .env.example .env
```

```env
EXPO_PUBLIC_SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
EXPO_PUBLIC_CONTRACT_ID=<your_deployed_gist_registry_contract_id>
EXPO_PUBLIC_MAPBOX_TOKEN=pk.eyJ1Ijoixxxxxxx...
EXPO_PUBLIC_GRAPHQL_URL=http://localhost:4000/graphql
```

Get a free Mapbox token at [mapbox.com](https://www.mapbox.com).

### 3. Start the dev server

```bash
expo start
```

### 4. Run on a simulator

```bash
# iOS Simulator (macOS only)
expo start --ios

# Android Emulator
expo start --android
```

### 5. Run on a physical device

Scan the QR code shown by `expo start` with **Expo Go** on your phone. Ensure your device and development machine are on the same network.

---

## Environment Variable Notes

- `EXPO_PUBLIC_*` variables are bundled into the app at build time and visible to the device — never put secrets here.
- For Android Emulator, use `http://10.0.2.2:4000` for localhost services. For iOS Simulator, `http://localhost:4000` works directly.

---

## Production Builds

```bash
# Install EAS CLI
npm install -g eas-cli
eas login

# Build for Android
eas build --platform android

# Build for iOS
eas build --platform ios
```

Use `--profile preview` for internal testing builds that skip store submission.

---

## Project Layout

```
TheGist-mobile/
├── app/                          # Expo Router file-based screens
│   ├── _layout.tsx               # Root layout (wallet provider, Apollo)
│   ├── (tabs)/
│   │   ├── map.tsx               # Map tab
│   │   └── nearby.tsx            # List tab — gists sorted by distance
│   └── gist/[id].tsx             # Gist detail screen
├── components/
│   ├── map/
│   │   ├── GistMap.tsx           # Mapbox wrapper
│   │   └── GistMapPin.tsx        # Individual map marker
│   ├── gists/
│   │   ├── GistCard.tsx          # Gist row card
│   │   └── PostGistSheet.tsx     # Bottom sheet: compose + sign
│   └── wallet/
│       └── ConnectSheet.tsx      # WalletConnect / Freighter connect
├── lib/
│   ├── soroban.ts                # Transaction building and submission
│   ├── ipfs.ts                   # IPFS pinning
│   └── graphql/                  # Apollo Client setup and queries
├── .env.example
├── app.json
├── app.config.ts
└── README.md
```

---

## Useful Scripts

| Command | Description |
|---------|-------------|
| `expo start` | Start Expo dev server |
| `expo start --ios` | Open in iOS Simulator |
| `expo start --android` | Open in Android Emulator |
| `npm run lint` | ESLint |
| `npm run type-check` | TypeScript check |
| `npm test` | Jest tests |

---

## Contributing

- All Soroban interactions go through `lib/soroban.ts` — no raw SDK calls in screen files.
- IPFS pinning lives in `lib/ipfs.ts`.
- Location access must always be requested via `expo-location` with a user-visible permission prompt.
- Coordinate any GraphQL query shape changes with TheGist-API.

For global contribution rules, see [CONTRIBUTING.md](https://github.com/TheGist-Org/theGist-Meta/blob/main/CONTRIBUTING.md).

---

## License

MIT
