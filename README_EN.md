[Русский](README.md) · **English**

<div align="center">

# Otklik

A lead-handling system for service businesses: the customer gets a reply within a minute, the owner gets a lead card in MAX and Telegram, and response time is tracked

![Python 3.14](https://img.shields.io/badge/Python-3.14-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.163-009688?logo=fastapi&logoColor=white)
![PostgreSQL 16](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![1104 tests](https://img.shields.io/badge/tests-1104_passing-2EA043)
![Lighthouse 100](https://img.shields.io/badge/Lighthouse-100-F44B21?logo=lighthouse&logoColor=white)

**[Live showcase](https://turtleshell2077.github.io/otklik-showcase/) · [Video](assets/demo.mp4) · [Contact](#contact)**

<img src="assets/demo.gif" width="800" alt="A customer completes the quiz on a renovation company website, and a second later the owner receives the lead card">

</div>

## What it does

Small service businesses (the first niche is home renovation in Russia) lose leads that arrive at night, on weekends or while the owner is on site. Otklik:

- **replies to the customer instantly** — lead number and a price range, or an honest "we'll call you tomorrow from 9:00" outside working hours;
- **collects leads from every channel** — website quiz, MAX and Telegram bots, Tilda/Marquiz and other forms, missed calls from a cloud PBX — into one card in the owner's messenger with "Take it / In 1 h / Not a lead" buttons;
- **tracks the reaction** — if nobody takes the lead in time, the team gets a reminder and then the owner is notified personally; the admin panel shows response time, night-time leads, sources and the funnel.

Screenshots and the full feature list are in the [Russian README](README.md#скриншоты).

## Architecture

```mermaid
flowchart LR
    subgraph SRC["Lead sources"]
        direction TB
        S1["Website quiz"]
        S2["Tilda, Marquiz,<br/>any form"]
        S3["MAX and Telegram<br/>bots"]
        S4["Cloud PBX:<br/>missed call"]
    end
    subgraph INST["One instance per client · Docker Compose on the client's server"]
        direction TB
        CADDY["Caddy · HTTPS"]
        WEB["FastAPI<br/>API, webhooks, admin panel"]
        CORE["Pure core, no I/O<br/>flow, pricing, business hours"]
        CFG["client.yaml"]
        DB[("PostgreSQL<br/>leads + outbox")]
        WRK["Worker and scheduler<br/>delivery, reminders, digests"]
        BAK["Encrypted backups"]
    end
    AI["LLM<br/>Yandex AI Studio, GigaChat"]
    OWN["Owner and team<br/>MAX · Telegram"]
    S1 & S2 & S3 & S4 --> CADDY
    CADDY --> WEB
    WEB -->|"lead + notification job<br/>in one transaction"| DB
    DB --> WRK
    WRK --> OWN
    CFG --> CORE
    WEB --- CORE
    WRK --- CORE
    WEB -.->|"text without names and phones"| AI
    DB --> BAK
```

- **Transactional outbox** — a saved lead always reaches the owner: the worker retries and survives restarts.
- **One instance per client** — client data never mixes; an outage or upgrade affects one client only.
- **YAML config with errors in Russian** — flow, prices, texts and hours change without a developer; a typo stops startup with a clear message.
- **Reaction control and escalation** — counted in business minutes: a reminder to the team first, then a personal alert to the owner.
- **Fallback messenger** — if MAX or Telegram is down, the card goes to the other one.
- **PII masking before the LLM** — the model never sees names or phone numbers; amounts and promises in its answers are checked against the price list and knowledge base.

Quality: 909 backend tests, 151 Vitest, 44 Playwright e2e, Lighthouse mobile 100/100/100/100, pyright strict for the core.

## Stack

| Layer | Technologies |
|---|---|
| Backend | Python 3.14, FastAPI, Pydantic 2, SQLAlchemy 2 (async), Alembic, structlog |
| Data | PostgreSQL 16, outbox queue on `FOR UPDATE SKIP LOCKED` |
| Messengers | MAX (own httpx client), Telegram (aiogram) |
| AI | Yandex AI Studio, GigaChat, OpenAI-compatible APIs |
| Websites | Astro, TypeScript, own design system and quiz, no external CDNs |
| Admin panel | server-rendered Jinja2, token-based CSS, strict CSP, PWA |
| Tests | pytest, hypothesis, Vitest, Playwright, Lighthouse |
| Deploy | Docker Compose, Caddy (HTTPS), encrypted rotated backups |

## Contact

**Alexey Malyshev** — I design and deploy Otklik for service businesses.

- MAX: [message me in MAX](https://max.ru/u/f9LHodD0cOKYgKYUB-79BnOZrfAqQjYPuFIh7r63PSzxH7JzxmgAK0zqOJw)
- Email: [turtleshell2077@gmail.com](mailto:turtleshell2077@gmail.com)

The source code is private; available on request.
