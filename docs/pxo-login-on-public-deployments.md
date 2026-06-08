# PXO Analysis — Login Button on Public Deployments

> **Perspective:** the **non-logged-in user** (end-user / guest on a public agent)
> **Task:** Add a login (and logout) button to public deployments — Share Link, Embed, Live Chat, Website Copilot.

A guest may want to log in even on public agents in two cases: (1) when the account has **end-user limits** set, and (2) when the agent has **conversation history** available. There is currently no straightforward way to do so.

**Visibility conditions:** the icon is shown when the **agent is public** AND (`End-user conversation history` is **not** set to *hidden* OR the `End-user limit for Guests` is **not** *Unlimited*).

**Placement:** Share Link → top-right icon cluster. Embed / Live Chat / Website Copilot → top bar.
**States:** present on **all** chat states (with and without a conversation).

---

## Step 00 — Problem Hypothesis

**Refined problem statement**
On public deployments there is no clear way for an end-user to log in. This becomes a problem in two scenarios: (1) the account has end-user limits set, and (2) the agent persists conversation history. In both, a guest hits a wall (limit exhausted) or suffers a loss (history disappears) with no explanation and no exit.

**Impacted user segment**
End-user / guest on a public agent — someone opening a Share Link or interacting with an embed / Live Chat widget, with no CustomGPT account or no active session.

**Business risk**
- Guest hits the limit → abandons → lost conversion event (registration / lead).
- Guest loses history on refresh → perceives the product as broken → trust erodes for the agent owner (e.g. a city / institution).
- For the owner: inability to tie valuable conversations to a user identity.

**Metric at risk**
Guest return / continuation rate after hitting a limit, and the guest → logged-in end-user login conversion rate.

**Current user alternatives**
None in-context today. The guest either abandons, manually hunts for a login on the main site (which effectively doesn't exist for public deployments), or opens a new incognito session to bypass the limit.

**Why now**
Two new mechanisms (end-user limits + conversation history) implicitly assume an identity, but the UI offers no way to establish that identity on public surfaces. The capability exists under the hood; the entry point is missing.

**Validated hypothesis**
> If we add a visible login/logout icon to public deployments (when the agent has limits or history), for **guests**, then **return/continuation rate and login conversion** will rise, **because** the guest gets a clear, contextual exit when they hit a limit or want to save/continue history.

**Risk level: Medium**
The change is additive (a new icon) and doesn't break existing flows. Risk is medium because: the auth flow on a third-party embed (cookies, popup blockers, cross-origin) is technically sensitive, and incorrect icon visibility may confuse guests who have no account.

---

## Step 01 — Empathy Engine

**Think**
- "Why did my conversation disappear when I refreshed?"
- "It says I hit a limit — but who's limiting me? I never even signed in."
- "Do I really need an account for this? I just wanted to find one decision in the Gazette."

**Feel**
- Confusion (no context for why a limit appeared).
- Frustration (lost work / history).
- Suspicion / distrust (a sudden login wall reads like a bait or a paywall).

**Say**
- "Where's the button to sign in?"
- "I lost everything I asked."
- "Do I have to register? I just want an answer."

**Do**
- Refresh the page → lose history → abandon.
- Look for "login" in the interface, don't find it → leave.
- Open incognito to get a fresh guest limit.

**Hidden frictions**
- Guest doesn't know whether the limit is tied to them, their IP, or the browser session.
- The phrase "log in" on a local institution's agent (City of Niš) is confusing — log into *what*?

**Emotional risks**
- Feeling "punished" by a login wall with no warning.
- Loss of trust in the owning institution, not in CustomGPT.

**Trust barriers**
- A cross-origin login popup feels like phishing if it isn't branded.
- Unclear what logging in actually grants (persistent history? higher limit? both?).

---

## Step 02 — JTBD Extraction

**Core functional job**
"When I'm using a public agent and hit a limit or want to save a conversation, I want to log in quickly without leaving the page, so I can continue where I left off."

**Emotional job**
Feel in control and safe — that my work won't vanish and that I know why something happened.

**Social job**
Look competent (e.g. a clerk using the agent in front of a citizen must not lose the conversation) and make the product look reliable to the institution I represent.

**Job steps (chronological)**
1. Guest opens the public agent.
2. Asks one or more questions.
3. Hits the guest limit OR wants to save/continue history.
4. Notices the login icon.
5. Clicks → authenticates (minimal flow).
6. Returns to the same conversation, limit raised / history persists.
7. (Later) Recognizes themselves as logged in and logs out if needed.

**Success definition**
Guest logs in in ≤2 steps, stays in the same conversation context, and continues with no data loss.

**Failure definition**
Guest can't find login, or login kicks them out of the conversation / loses history, or the auth popup is blocked / fails with no recovery.

---

## Step 03 — Constraint Mapping

**Technical**
- **Cross-origin** on Embed / Website Copilot: login must go through a popup or redirect; popup blockers and third-party cookie restrictions (Safari ITP, Chrome) are real obstacles.
- The Live Chat widget often lives in an iframe → the session/token must survive parent navigation.
- Share Link is first-party on `*.customgpt-agents.com` → the easiest case.
- Auth state must reflect across all chat states (empty + with conversation) with no reload.

**Data**
- Guest (anon) history must be migrated/merged with the logged-in end-user's history after login (merge anon session → identified user).
- End-user limits: distinguish the Guest limit from the Identified limit; the UI must know both.

**Legal / Compliance**
- GDPR: before login the guest is anonymous; login introduces PII → needs consent / privacy link.
- The agent owner (e.g. a city) may have its own data policies.

**Organizational**
- Icon visibility depends on agent settings (public + history ≠ hidden OR guest limit ≠ unlimited) — the logic must be consistent across all 4 deployment types.
- Branding of the login screen: whose identity is it (CustomGPT end-user identity vs. owner SSO)?

**Edge cases**
- Agent public, but history hidden AND guest limit unlimited → the icon is **NOT** shown.
- Guest already logged in in another tab.
- Login mid-stream while an answer is rendering.
- Guest on a mobile embed where popups behave poorly.
- Login then immediate logout — must return to guest state without breaking.

---

## Step 04 — Persona Roles

**Naive (first-time)**
A citizen who clicked the "Official Gazette of the City of Niš" link from Google. Doesn't know what CustomGPT is, doesn't expect a login. For them the icon must be unobtrusive until needed, and when needed, accompanied by an explanation of what they gain.

**Medium (regular user)**
Someone returning to the same agent (e.g. a clerk who searches the gazette daily). Wants their history to persist and to continue quickly. Recognizes the value of logging in the first time they lose history.

**Expert (power user)**
A user juggling multiple public agents, possibly already holding a CustomGPT end-user account. Expects login/logout in a predictable spot (top-right), friction-free, with state synced across deployments.

---

## Step 05 — Behavioral Trigger Design

**Activation trigger**
The login icon appearing in a predictable corner (top-right for Share Link; top bar for the rest) — passively present from the start, on all states.

**Aha moment**
Guest hits a limit OR refreshes and sees "Log in to save your conversation" → realizes login directly solves their loss. A contextual prompt next to the icon, not the icon alone.

**Competence moment**
After login, the conversation is intact, the limit is raised, and a subtle confirmation appears ("You're logged in — history is saved"). The user feels they mastered the system.

**Control moment**
A visible logout icon/menu — the user can sign out at any time and return to the anonymous state, which builds trust (no "lock-in").

---

## Step 06 — Information Architecture

```
Public Agent Chat (guest)
├── Header / Top area
│   ├── [Share Link]  → icons top-right: [🔊 sound] [💻 display] ... [👤 Login]
│   └── [Embed/LiveChat/Copilot] → top bar: [agent name] ............ [👤 Login]
│       └── (logged in) [👤▾]
│            ├── Identity (email/name)
│            ├── "History is saved" status
│            └── Logout
├── Chat body
│   ├── Empty state (starter questions)
│   └── Conversation state
│       └── Limit-reached banner → inline [Log in for more]
└── Composer (input)
```

**Naive task flow**
Open link → ask → hit limit → see banner "Log in to continue" → click → minimal login → continue.

**Medium task flow**
Return to agent → click login icon top-right → authenticate → history loads → continue where left off.

**Expert task flow**
Notice 👤 in the corner → if not logged in, click → SSO/account → state syncs → later 👤▾ → Logout as needed.

---

## Step 07 — Failure Mode Design

**Failure 1 — Popup blocked (cross-origin embed)**
Scenario: guest clicks login, browser blocks the popup.
Recovery: detect the block → fall back to a full-page redirect or inline modal with "Allow the popup or [log in here]".

**Failure 2 — Guest history not merged after login**
Scenario: guest had an anonymous conversation, logs in, and the conversation "disappears".
Recovery: save the anon session ID before login → merge into the account after login; if the merge fails, show "Your previous conversation: [continue]".

**Failure 3 — Login mid-stream**
Recovery: disable the login button / set it "busy" while streaming, or finish the stream before redirecting; never interrupt an answer mid-way.

**Failure 4 — Icon shown to a guest with no account (no SSO)**
Scenario: guest clicks but doesn't know how to create an account.
Recovery: the login screen also offers a "Create account" path; microcopy explains the benefit.

**Failure 5 — Wrong visibility (icon shown when it shouldn't be)**
Recovery: strict conditional logic — if `history=hidden` AND `guestLimit=unlimited` → don't render the icon at all.

**Graceful degradation**
If the auth service is down: the icon goes to a disabled state with a tooltip "Login temporarily unavailable — continue as guest", and the guest flow stays fully functional.

---

## Step 08 — Metric Definition

**Primary metric**
Guest → logged-in conversion rate (% of guests who click login and successfully authenticate), measured especially among those who hit a limit.

**Secondary metrics**
- Continuation rate after hitting a limit (% who continue instead of leaving).
- History retention satisfaction — % of sessions where history persisted through login.
- Logout rate / return-to-guest (a trust control signal).

**Early signal (7-day)**
Login icon CTR on public deployments and the successful-auth rate (free of popup / cross-origin errors).

**Long-term signal (30-day)**
Increase in returning logged-in end-users and a drop in abandonment at the limit-reached point.

**Leading indicators**
- Share of sessions reaching the limit banner.
- Time from login click to return-to-conversation (target < 15s).
- Share of popup-fallback events (a technical-friction signal).

---

## Step 09 — Low-Fi Specification

**Layout structure**
- **Share Link**: icons in the top-right header row, next to existing ones (🔊, 💻). Login 👤 as the rightmost.
- **Embed / Live Chat / Website Copilot**: top bar, right-aligned, next to the agent name.

**Priority hierarchy**
1. The conversation/composer (primary).
2. Limit/history context (banner when relevant).
3. Login icon (passive until needed, but always visible on all states).

**CTA logic**
- Default: unobtrusive 👤 icon (no label on narrow space; label "Log in" where space allows).
- Escalated: on hitting a limit → inline banner with an expanded CTA "Log in to continue".

**State logic**
- **Logged-out**: 👤 icon + tooltip "Log in".
- **Logged-in**: 👤▾ with initials/avatar, opens a menu (identity + Logout).
- **Loading (during auth)**: spinner in place of the icon.
- **Error**: icon + tooltip "Login failed, try again".
- **Disabled (service down)**: dimmed icon + tooltip.

**Error states**
Popup blocked → inline message + fallback link. Auth fail → toast "Login failed".

**Empty states**
On the empty chat state (starter questions) the icon is present in the corner, with no banner — just a passive entry point.

---

## Step 10 — Heuristic & Cognitive Audit

| Heuristic | Violation | Improvement |
|---|---|---|
| Visibility of system status | Guest can't tell whether they're logged in | Logged-in/out state must be visually unambiguous (avatar vs. empty icon) |
| Match with the real world | "Log in" on an institution's agent is unclear | Use microcopy "Log in to save your conversation" instead of a bare "Login" |
| User control & freedom | No exit from the login wall | Always offer "continue as guest" and a visible Logout |
| Consistency & standards | Login could drift across deployments | Same spot (top-right) across all 4 deployment types and all chat states |
| Error prevention | Icon could appear where pointless | Strict conditional visibility (history hidden + unlimited guest → no icon) |
| Recognition over recall | Guest must remember a login exists | Contextual banner at the limit point instead of relying on memory |
| Flexibility & efficiency | One-size flow | Expert gets fast top-right access; naive gets a guided banner |
| Minimalist design | Header clutter on narrow embeds | Keep it an icon (no text button) on narrow embeds |
| Recover from errors | Auth dead-ends | Every auth fail has a clear message and retry/fallback, never a dead-end |
| Help & documentation | Unclear value | "Why log in?" / privacy link next to the login screen |

**Cognitive load** — Don't push login aggressively up front; reveal value only when it becomes relevant (progressive disclosure).

**Accessibility (WCAG 2.1)** — Icon needs an `aria-label` ("Log in" / "User menu"), contrast ≥ 4.5:1 (note: a white icon on a red background like Niš is fine), visible focus, keyboard-accessible menu, touch target ≥ 44×44px.

---

## Step 11 — High-Fi Execution

**Component inventory (all states)**
- `LoginIconButton`: states = `logged-out`, `loading`, `logged-in`, `error`, `disabled`.
- `UserMenu` (popover): identity, history status, Logout.
- `LimitReachedBanner`: inline, with a CTA.
- `AuthModal/Popup`: auth flow + fallback.

**Interaction details**
- Hover on 👤 → tooltip (white bg, shadow `0 1px 2.2px rgba(0,0,0,0.25)` per the design rule, never dark).
- Click logged-out → opens auth (popup → fallback redirect).
- Click logged-in → popover menu.
- Esc closes the menu/modal; focus trap inside the modal.

**Motion logic**
- Icon → spinner: cross-fade 150ms.
- Popover: fade + slide 120ms ease-out.
- Banner: slide-down 200ms when the limit is reached, with no layout jump of the composer.

**Microcopy** (English; localizable to the agent's language)
- Tooltip (out): "Log in"
- Tooltip (in): "User menu"
- Banner title: "You've reached the guest limit"
- Banner CTA: "Log in to continue"
- History prompt: "Log in to save your conversation"
- Auth fail toast: "Login failed. Please try again."
- Popup fallback: "Allow the popup or log in here."
- Menu logout: "Log out"
- Logged-in confirm: "You're logged in — history is saved"

**Visual hierarchy**
The icon follows the header color (on branded backgrounds — e.g. white on the City of Niš red); never a primary color with a colored shadow (per the "no blue shadow" rule).

**Edge case rendering**
- Narrow header (mobile embed): icon only, no label.
- Long agent name: truncate the name, icon keeps a fixed right position.
- RTL / Cyrillic: tooltip and menu alignment follow localization.

**Responsive behavior**
- Desktop: icon + optional label.
- Mobile / narrow widget: icon only, menu as a bottom-sheet if Live Chat is full-screen.
- The CustomGPT platform is desktop-only for admin, but the **end-user surface (embed) must work on mobile** — this is user, not admin, space.

---

## Step 12 — Validation Loop

**Before development**
- **Cognitive load test**: does a naive guest understand what 👤 does without clicking? (5-second test on 5 users).
- **Job completion speed**: measure click-login → return-to-conversation time on the prototype (target < 15s, ≤ 2 steps).
- **Risk perception**: test whether guests perceive login as a threat/paywall — pilot microcopy variants ("Log in" vs. "Save conversation").
- **Prototype first**: the Share Link case (first-party, least technical friction) as a baseline, then the embed cross-origin flow.

**After release**
- **Behavior to monitor**: login icon CTR per deployment type; drop in abandonment at the limit banner; merge success rate anon → identified history.
- **Metric thresholds**: if login conversion < 5% among limit-hits → revisit microcopy/visibility; if popup-fallback > 20% → solve cross-origin access.
- **Unexpected failure modes**: guest loop (login → logout → fresh guest-limit abuse), Safari third-party cookie session loss, duplicate histories after merge.
- **Iteration triggers**: high CTR but low completion → flow is tried but frustrating (fix the auth step); low visibility → move/emphasize the icon or strengthen the contextual banner.

---

## Executive Summary

From the **non-logged-in guest's** perspective, current public deployments have a hidden gap: end-user limits and conversation history were introduced, but there's no entry point for a guest to identify themselves — so hitting a limit and losing history read as a malfunction, not an invitation to sign in. The fix is a **passive, predictable login/logout icon** (top-right for Share Link, top bar for the rest), present on **all** chat states, paired with a **contextual banner** that reveals the value of logging in exactly at the moment of frustration (limit-reached / "save your conversation"). The key risks aren't visual but **technical and trust-related**: cross-origin popup blocks, merging anonymous history into an account, and the danger of login feeling like a forced wall — so the flow must always offer "continue as guest", a visible logout, and graceful degradation. Strict conditional visibility (show only if the agent is public AND a limit or history exists) prevents the icon from appearing pointlessly. Success is measured by guest → logged-in conversion and continuation rate at the limit point, with Share Link as the first, cleanest prototype before moving to the more sensitive embed case.
