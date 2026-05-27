# kees-server2

A simple Express.js server application.

## Installation

```bash
npm install
```

## Running the Server

### Development
```bash
npm run dev
```

### Production
```bash
npm start
```

The server will run on `http://localhost:3000` by default.

## API Endpoints

- `GET /` - Welcome message
- `GET /api/health` - Server health check

## Environment Variables

You can set the `PORT` environment variable to run on a different port:

```bash
PORT=5000 npm start
```
