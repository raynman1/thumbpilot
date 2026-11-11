<p align="center"><img src="assets/banner.svg" alt="ThumbPilot" width="720"></p>

# ThumbPilot

**Give your coding agent thumbs.** Unlock, switch profiles, tap and drive apps on a real rooted Android phone, from Claude, ChatGPT, Grok or any agent.

![license](https://img.shields.io/badge/license-MIT-a6e3a1?style=flat-square) ![android](https://img.shields.io/badge/Android-rooted-a6e3a1?style=flat-square) ![agents](https://img.shields.io/badge/agents-Claude%20%7C%20ChatGPT%20%7C%20Grok-89b4fa?style=flat-square) ![version](https://img.shields.io/badge/guide-v1.5.0-cba6f7?style=flat-square)

> Paste this into your coding agent:
>
> `Read GUIDE.md from this repo. Use my values from config.env. Act by default. First, run a harmless test: open Settings and stop. Tell me only the outcome.`

[Quick start](#quick-start) | [How it works](#how-it-works) | [Rules in one minute](#rules-in-one-minute) | [FAQ](#faq) | [Contributing](#contributing)

## Quick start

1. Root a phone and install [FIRERPA Lamda](https://github.com/firerpa/lamda). Enable USB debugging.
2. `git clone` this repo, then `cp config.example.env config.env` and fill in your own values. Never commit `config.env`.
3. Pick a route: USB (`adb forward`) or a small SSH gateway you host.
4. Paste the prompt above into your agent. See [QUICKSTART.md](QUICKSTART.md).

## How it works

```
your agent -> GUIDE.md rules -> route (USB or SSH gateway) -> controller -> Lamda -> your phone
```

One guide, one source of truth. The agent unlocks the phone, switches Android profiles, reads the screen, taps, types and verifies. Holds are puzzles with fixes in a table, not dead ends.

## Rules in one minute

- Act by default. Routine app work needs no permission.
- Ask the owner only for money, sending messages, secure screens (PIN, OTP, bank), wrong-PIN risk, or a reboot.
- Never type a wrong PIN twice. Never store secrets in this repo.

## FAQ

**Is this safe?** You control a real phone. Use it only on devices and accounts you own. Secrets stay in your password manager.
**Which agents work?** Any agent that can run shell commands and read a file.
**Do I need a gateway?** No. USB alone works.
**Status?** Early. This repo is the operating guide. The reference controller it describes (the `doctor`, `observe`, `run`, `act` commands) is not published yet, so today you pair the guide with your own controller or wait for the code release. Issues and ideas welcome.

## Contributing

Open an issue or a pull request. Improve `GUIDE.md`: a hold and its fix, a timing, an app procedure. Never include secrets, real addresses, serials or job logs.

## License

MIT. See [LICENSE](LICENSE).
