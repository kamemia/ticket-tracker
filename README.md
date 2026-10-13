# Mini Ticket Tracker (Frontend)

Vue 3 + Vite + Vuetify client for the Mini Ticket Tracker API.

## Prerequisites

- Node.js 18+
- Git
- The Django API running locally (see the backend README)

## Setup

1. Clone the repository and enter the project directory:

```bash
   git clone https://github.com/kamemia/ticket-tracker.git
   cd ticket-tracker
```

2. Install dependencies:

```bash
   npm install
```

3. Copy the example env file and set the API URL:

```bash
   cp .env.example .env
```

```env
   VITE_BASE_API_URL=http://127.0.0.1:8000
```

4. Start the dev server:

```bash
   npm run dev
```

   The app is available at http://localhost:5173.

## Scripts

| Command           | Description                      |
| ----------------- | -------------------------------- |
| `npm run dev`     | Start the dev server             |
| `npm run build`   | Build for production to `dist/`  |
| `npm run preview` | Preview the production build     |

## Notes
Ensure the backend server is running.