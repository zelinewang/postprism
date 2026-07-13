# Contributing to PostPrism

PostPrism is a hackathon prototype: a React + TypeScript front end (built with
Bun) and an experimental Flask + Agent-S computer-use backend. Most
contributions touch the front end, which needs no API keys.

## Front-end setup

Requires [Bun](https://bun.sh/).

```bash
git clone https://github.com/zelinewang/postprism.git
cd postprism
bun install
bun run build     # production build — a good CI-equivalent sanity check
bun run dev       # http://localhost:8080
```

Lint before pushing:

```bash
bun run lint
```

## Backend (optional, credentialed)

The Flask backend under `backend/` orchestrates ORGO VMs and computer-use agents
and needs ORGO + model credentials. See `README.md` → Setup. Never commit `.env`
files or keys — `env.example.txt` documents the variables.

## Pull requests

- Branch off `main`; open one focused PR per change.
- Keep the hosted demo credential-free — front-end changes must still build and
  run in demo mode with no keys.
- Conventional-ish commit subjects (`fix:`, `feat:`, `docs:`…). PRs are
  squash-merged, so the PR title becomes the commit message.
- Run `bun run build` and `bun run lint` before opening the PR.

## License

By contributing, you agree that your contributions are licensed under the
project's [MIT License](LICENSE).
