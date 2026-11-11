# Quickstart

1. Root your phone and install FIRERPA Lamda (https://github.com/firerpa/lamda). Enable USB debugging.
2. Clone this repo and copy `config.example.env` to `config.env` (never commit it). Fill in your own values.
3. Pick a route: USB (adb forward `<LOCAL_PORT>` to `<DEVICE_PORT>`) or a restricted SSH gateway you host.
4. Give your AI agent `GUIDE.md` and this prompt:

   > Read GUIDE.md. Act by default. Use my values from config.env. Run a harmless test first: open Settings and stop. Tell me only the outcome.

5. Test: unlock, switch profile, open an app. Improve the guide with a pull request.

Contributing: open an issue or PR. Never include secrets, real addresses, serials or job logs.
