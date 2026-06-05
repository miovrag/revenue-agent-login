# Revenue Agent — Login Edition

A single-file chat widget for [CustomGPT.ai](https://customgpt.ai), deployed as a standalone landing page. This is an improved copy of the original **Revenue Agent** experiment, adding an end-user **login / logout** control to the chat header.

Live: **[revenue-agent-login.vercel.app](https://revenue-agent-login.vercel.app)**

---

## What's new in this edition

### Login / Logout control

There are two cases where a visitor may want to be signed in, even on a **public** agent, with no straightforward way to do so today:

1. The account has **end-user limits** set (guests have a limited message allowance).
2. The agent has **conversation history** available.

This edition adds a login/logout icon to the chat header so signing in is one tap away.

- **Login icon** sits in the top-right icon row, alongside Share / Download / Reset / Close.
- Present in **every chat state** — with and without an active conversation.
- Clicking **login** opens CustomGPT auth (`https://app.customgpt.ai/login`) in a new tab and switches the control to its **logout** state.
- Clicking **logout** clears the local end-user session.

### Visibility conditions

The control is only shown when **both** hold (matching the deployment settings on Share Link / Embed / Live Chat / Website Copilot):

- Agent is **public**, **and**
- *End-user conversation history* is **not** hidden **OR** the *End-user limit for Guests* is **not** Unlimited.

These are driven by `authConfig` at the top of the `<script>` block:

```js
const authConfig = {
  agentIsPublic:        true,   // Agent is public
  endUserHistoryHidden: false,  // "End-user conversation history" set to hidden?
  guestLimitUnlimited:  false,  // End-user limit for Guests = Unlimited?
};
```

Flip these to preview when the control appears or hides.

---

## Features (inherited)

- **AI chat** powered by the CustomGPT.ai REST API (streaming responses)
- **Preset questions**, **sign-up card**, **pricing card**, **feedback questionnaire**
- **Glass morphism input** with backdrop blur; hides on scroll, reappears on scroll stop
- **Entrance animation**, **ping sound**, **cursor glow**, **light / dark mode**, **mobile fullscreen**

## Stack

| Layer | Technology |
|---|---|
| Markup & logic | Single HTML file |
| Styles | Tailwind CSS (CDN) + inline styles |
| Typography | Manrope (Google Fonts) |
| Icons | Material Symbols Outlined |
| AI backend | CustomGPT.ai REST API |
| Hosting | Vercel |

## Project structure

```
revenue-agent-login/
├── index.html       # entire app — HTML, CSS, JS
├── bg.webp          # light mode background
├── bg-dark.webp     # dark mode background
├── avatar.png       # Alden's photo (bot avatar)
└── logo.svg         # CustomGPT.ai logo
```

## Configuration

Constants at the top of the `<script>` block in `index.html`:

```js
const CGPT_KEY = 'YOUR_API_KEY';
const CGPT_PID = 77058;                                  // CustomGPT project ID
const CGPT_LOGIN_URL = 'https://app.customgpt.ai/login'; // auth target for login
```

## Deploy

```bash
npm i -g vercel   # if needed
vercel --prod
```

Or connect the GitHub repo to a Vercel project for automatic deploys on push.
