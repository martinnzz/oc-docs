## Application Structure

- `example-http-server.ts`: exports the `ExampleServer` class (Express app + routes + start/stop lifecycle).
- `index.ts`: bootstrap file. Creates an `ExampleServer` instance, conditionally starts it (skips when `NODE_ENV=test` or `AUTO_START=false`), and handles graceful shutdown signals.
