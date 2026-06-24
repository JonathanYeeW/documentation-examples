# PB&J Mobile

The PB&J Machine Co. consumer app for ordering sandwiches from a machine in your home or school. Pick your bread, spread, and jelly — your sandwich is ready in under two minutes.

Built with Expo and React Native. Runs on iOS and Android.

---

## Setup

### Prerequisites

- Node.js 18+
- Expo CLI (`npm install -g expo-cli`)
- For iOS: Xcode 14+ and an Apple Developer account
- For Android: Android Studio with an emulator or a physical device
- A running instance of the [PB&J Machine API](https://github.com/pbj-machine-co/pbj-api)

### Install

```bash
git clone https://github.com/pbj-machine-co/pbj-mobile
cd pbj-mobile
npm install
```

### Configure

Create a `.env` file at the project root:

```bash
EXPO_PUBLIC_API_URL=http://localhost:3000
```

### Run

```bash
# Start the Expo dev server
npx expo start

# Run on iOS simulator
npx expo run:ios

# Run on Android emulator
npx expo run:android
```

---

## Features

**Order Flow** — A guided four-step wizard: bread → spread → jelly → confirm. One choice per screen, under 20 seconds to place an order.

**Live Progress** — Real-time assembly tracking from the moment you confirm to the moment your sandwich is ready. Chime notification on completion.

**Order History** — Every sandwich you've ordered, with timestamps and selections. Reorder in one tap.

**Dietary Flags** — Set nut-free or gluten-free preferences on your account. The app filters available ingredients automatically.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for local development setup, navigation conventions, and how to run the app against the machine simulator.
