---
title: Frontend and React principles
id: frontend-react-principles
doc_type: concept
summary: Track D. Providers as a dependency graph, Atomic Design with one rule for where data fetching lives, design tokens across themes, the lowest owner for each kind of state, effects only for synchronising with the outside, one API client, the browser auth patterns ranked by security, conversational UI, accessibility, caching and delivery, and tests that query like a user.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [frontend, react, atomic-design, design-tokens, theming, state-management, hooks, effects, api-client, oauth, pkce, bff, conversational-ui, accessibility, wcag, performance, caching, testing]
aliases: [provider tree, component hierarchy, atoms molecules organisms, CSS custom properties, DTCG, server state, client state, useEffect, custom hooks, fetch wrapper, backend for frontend, token storage, chat UI, streaming UI, aria-live, Core Web Vitals, React Testing Library]
related: [engineering-principles-map.md, backend-api-principles.md, security-identity-principles.md, ai-agent-engineering.md]
group: Build
order: 40
html_twin: frontend-react-principles.html
---

# Frontend and React principles

## TL;DR
- Providers wrap from the outside in, and each may use only those outside it; gate the tree on runtime config before anything that needs it mounts.
- Atomic Design gives five levels; the rule that ends most debates is that atoms never fetch and pages own data and routing.
- Every colour, font, radius and duration is a token; components use tokens only, and every theme defines every token.
- Give each kind of state its lowest owner. Server state belongs to the server: cache it, do not copy it.
- One API client adds auth and handles 401; for sensitive apps the browser should not hold tokens at all (a backend for frontend).
- Notes are private study notes and carry no ref; Practice sources are public links.

## D1 App structure and the provider tree
- Priority: P1 core. Source: Lesson.
- Principle: providers form a dependency graph. A provider may use those outside it, never those inside it. Order them by what they need: config → theme → auth → features that need a user → router.
- Principle: gate the tree on runtime config. Show a loading state until config has loaded, so no provider starts with missing values.
- Principle: one source of truth for navigation, with the role each item needs; protected routes show a sign-in prompt, admin routes redirect.
- Exercise: draw your app's provider tree and mark what each provider reads from the ones outside it.
- Check: why must the config gate sit outside the auth provider?

## D2 Component design and Atomic Design
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: Atomic Design has five levels (https://atomicdesign.bradfrost.com/chapter-2/):
  - Atoms — basic elements that stop being useful if split: a button, an input, a label.
  - Molecules — small groups of atoms that work as one unit: a search field.
  - Organisms — sections of an interface: a header, a card list, a chat composer.
  - Templates — page layouts that show the content skeleton.
  - Pages — templates filled with real content; in React, a route.
- Principle: design tokens sit below atoms, and a level uses only the levels below it.
- Principle, the rule that ends most debates: atoms hold no business logic and never fetch; organisms may use context and hooks; pages own data fetching and routing. "Molecule or organism?" is one question: does it own state or fetch data?
- Lesson: classifying an existing codebase finds misfiled components: a "shared primitive" that fetches through two services is an organism, and inline markup repeated in a layout is a missing component.
- Needs team input: the design system and component library, if one exists [unverified].
- Exercise: classify ten components; find each one that fetches data and say which level it really is.
- Check: where should data fetching live, and why?

## D3 Design tokens and theming
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: every colour, font, spacing step, radius and motion duration is a named token (a CSS custom property); components reference tokens only, never raw values.
- In practice: the W3C Design Tokens Community Group publishes a vendor-neutral JSON format for tokens; its first stable version is 2025.10 (https://www.designtokens.org/).
- Principle: switch themes with attributes on `<html>` (`data-theme`, `data-brightness`), so CSS alone re-themes the page without a React re-render.
- Lesson, rules that prevented regressions:
  - Every new token gets a value in every theme; a missing value falls back silently and looks like a bug in one theme only.
  - No raw colour literals in components; derive variants from tokens.
  - Override stylesheets load last, because cascade order decides.
  - A small script applies the stored theme before the first paint; its default must equal the app's default, or the page flashes on load.
  - `var()` cannot reach inside an SVG data URI, so an icon drawn that way needs one copy per theme.
  - Transitions use duration and easing tokens, so motion can be tuned, or turned off, in one place.
- Exercise: add one token correctly in every theme and brightness.
- Check: why can a CSS variable not colour an SVG inside a data URI?

## D4 State: give each kind its lowest owner
- Priority: P2 depth. Sources: Practice, Lesson.
- Principle: name the kind of state, then give it the lowest owner:

| Kind | Owner |
|---|---|
| Local UI | `useState` in the component |
| Shared app values that change rarely | Context: config, auth, theme |
| Server state | The server is the truth; a cache with invalidation, not a copy |
| URL state | Route and query parameters |
| Persisted preferences | Browser storage, validated on read |

- In practice: server state lives remotely, can be changed by others and goes stale; caching, deduplication, background refresh and staleness detection are the hard parts a server-state library solves (https://tanstack.com/query/latest/docs/framework/react/overview).
- In practice, structure state so impossible combinations cannot exist: group values that change together, replace related booleans with one status value, and never store what you can compute during render (https://react.dev/learn/choosing-the-state-structure).
- Principle: fast-changing values in a broad context re-render every consumer; many related `useState` calls in one page call for a reducer or a store.
- Lesson: a client-side cache of running jobs drifted from the server; deleting it and resuming from a server feed removed a class of bugs (`backend-api-principles`, C5).
- Lesson: migrate stored values safely. When an option is removed, a stored value that names it must fall back to a valid default on read.
- Exercise: sketch a reducer or store for one page that has many `useState` calls.
- Check: why is "a status value" safer than two booleans `isSending` and `isSent`?

## D5 Effects and hooks
- Priority: P2 depth. Sources: Practice, Lesson.
- Principle: an Effect synchronises with something outside React (the network, a timer, a non-React widget). If no external system is involved, you probably do not need one (https://react.dev/learn/you-might-not-need-an-effect).
- Principle: compute derived values during render; put logic caused by a user action in the event handler, not in an Effect (https://react.dev/learn/you-might-not-need-an-effect).
- Principle: every Effect that fetches has a cleanup that ignores stale responses, or a slow early response overwrites a later one (https://react.dev/learn/you-might-not-need-an-effect).
- Principle: put reusable behaviour (polling, voice input, a confirm flow) in a custom hook with its own cleanup; pair each provider with a hook that reads it.
- Lesson: never hand-roll a polling loop in a component; one shared hook with cleanup and backoff replaced several that leaked timers.
- Exercise: extract a polling loop from a component into a hook with cleanup.
- Check: what goes wrong when a fetch Effect has no cleanup?

## D6 One API client
- Priority: P1 core. Source: Lesson.
- Principle: one API client module. It adds `Authorization: Bearer <token>`, turns a 401 into one app-wide "session expired" event, maps errors to one shape, and reads the base URL from runtime config, never from the build.
- Principle: components never call `fetch` directly; they call a service function that uses the client. Tests then mock the service module.
- Lesson: serving the API under the same origin as the app (a hosting rewrite or a CDN path) removes CORS preflights entirely.
- Exercise: add a service call through the client, with a test that mocks the service module.
- Check: why is one client safer than `fetch` in each component?

## D7 Auth in the browser
- Priority: P1 core. Sources: Practice, Lesson; theory in `security-identity-principles` G2.
- In practice, three patterns in decreasing order of security (https://www.rfc-editor.org/rfc/rfc10017):
  - Backend for frontend (BFF): a server-side confidential client keeps the tokens and gives the browser a `Secure`, `HttpOnly` session cookie; strongly recommended for business apps and apps that handle personal data.
  - Token-mediating backend: the server gets tokens and hands the access token to the browser; refresh tokens stay on the server.
  - Browser-based client: the browser is a public OAuth client and holds the tokens itself.
- In practice: a public client must use authorization code with PKCE, and its refresh tokens must be sender-constrained or rotated (https://www.rfc-editor.org/rfc/rfc9700).
- Lesson: an OAuth `state` check written as "compare if both values exist" is skipped when the stored value is missing. Reject the callback when either value is absent.
- Lesson: refresh tokens on demand shortly before expiry, not only on a timer; background tabs throttle timers.
- Lesson: an identity SDK can add hundreds of kilobytes; calling the provider's REST endpoints keeps the bundle small, at the cost of harder federated sign-in and MFA.
- Exercise: walk the PKCE redirect in the browser's dev tools and name each parameter.
- Check: what does the `state` parameter protect against?

## D8 Conversational and streaming UI
- Priority: P2 depth. Sources: Lesson, Practice.
- Principle: the UI makes a slow, fallible model feel responsive and safe:
  - Stream the answer; users can type ahead into a serial submit queue.
  - Show staged progress labels on the client, not invented model "thoughts".
  - Answer the greeting and help commands without a model call.
  - Show a confirm panel for every action the model proposes; the user approves, then the normal API runs (`ai-agent-engineering`, H3).
  - Offer slash commands for frequent actions (`/help`, `/clear`, `/reset`).
- Lesson: only an explicit "feature disabled" error should switch users back to the classic form UI; a network blip should show a retry, not change the mode.
- Lesson: one conversational surface, not two; users shown two chat boxes did not know which one could act.
- In practice: announce streamed or late-arriving text with a polite live region, `aria-live="polite"`; reserve `assertive` for urgent interruptions (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-live).
- Exercise: add a slash command with a test.
- Check: why does a confirm step belong in the UI rather than in the prompt?

## D9 Accessibility and motion
- Priority: P2 depth. Sources: Practice, Lesson.
- In practice: WCAG 2.2 is a W3C Recommendation with three conformance levels, A, AA and AAA; normal text needs a contrast ratio of at least 4.5:1 (https://www.w3.org/TR/WCAG22/).
- In practice, a modal dialog moves focus inside on open, keeps Tab inside, closes on Escape, and returns focus to the element that opened it (https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/).
- Principle: semantic HTML first; every path usable by keyboard; contrast checked in every theme and brightness.
- Lesson: honour `prefers-reduced-motion` in every theme, including decorative transitions between themes.
- Lesson: visual checks across every theme and brightness are easy to skip when manual; automate at least a screenshot per combination.
- Needs team input: the accessibility standard (WCAG level) and how it is audited [unverified].
- Exercise: tab through one page and fix the first place focus gets lost.
- Check: what must happen to focus when a modal closes?

## D10 Performance and delivery
- Priority: P2 depth. Sources: Practice, Lesson.
- In practice: the Core Web Vitals are LCP (good at 2.5 s or less), INP (200 ms or less) and CLS (0.1 or less), measured at the 75th percentile of page loads (https://web.dev/articles/vitals).
- In practice: content-hashed files get `Cache-Control: max-age=31536000`; unversioned HTML gets `no-cache`, so the browser revalidates it every time (https://web.dev/articles/http-cache).
- Lesson: a cached `index.html` points at asset files the deploy deleted: the classic blank page after a release.
- In practice: split routes with `React.lazy` and `Suspense`, declared at module top level (https://react.dev/reference/react/lazy).
- Principle: count every SDK's bundle cost; memoise only what a measurement shows is slow.
- Exercise: measure the main bundle and list the three largest dependencies.
- Check: why does a cached `index.html` cause a blank page?

## D11 Testing the frontend
- Priority: P1 core. Sources: Practice, Lesson.
- In practice: tests should resemble how users use the software; query by role first, then label, placeholder and text, and by test id last (https://testing-library.com/docs/queries/about/).
- Principle: mock service modules, not the network or `fetch`.
- Lesson: wrapping an update that does nothing in `act()` fails the lint rule `testing-library/no-unnecessary-act`; flush async work explicitly instead.
- Principle: run a small smoke suite against the deployed environment after every deploy.
- Exercise: test a component that polls a job.
- Check: why query by role rather than by class name?

## D12 Shared primitives and file size
- Priority: P2 depth. Source: Lesson.
- Principle: use shared primitives (modal, button, spinner) instead of new one-offs; past a file-length cap, extract a hook or a component.
- Lesson: three modal implementations with different focus handling became one shared modal, which fixed focus and styling bugs in every caller at once.
- Exercise: replace one one-off spinner or modal with the shared primitive.
- Check: when is a one-off justified, and where do you record it?

## D13 Angular, if you meet it
- Priority: P3 reference. Source: Notes.
- Principle: modern Angular has standalone components, signals and a new control-flow syntax; experience from older versions is dated and should be said so.
- Needs team input: where Angular is used today [unverified].

## Invariants
- Components never call `fetch` directly (D6).
- Every token has a value in every theme (D3).
- Stored values are validated on read (D4).

## Gotchas
- An OAuth `state` check that runs only when both values exist can be skipped (D7).
- A pre-paint theme default that differs from the app default makes the page flash (D3).
- A fetch Effect without cleanup lets a slow, older response win (D5).

## Glossary
- Atomic Design — atoms, molecules, organisms, templates, pages.
- Design token — a named design value such as `--color-accent`.
- Server state — data whose truth lives on the server.
- BFF — backend for frontend: a server that holds the tokens for a browser app.
- PKCE — proof key for code exchange.
- Provider tree — nested React context providers.

## Open questions
- The team's design system, component library and WCAG level [unverified].

## Sources
- https://atomicdesign.bradfrost.com/chapter-2/ — Atomic Design.
- https://www.designtokens.org/ — the design tokens format.
- https://react.dev/learn/choosing-the-state-structure — structuring state.
- https://react.dev/learn/you-might-not-need-an-effect — when not to use an Effect.
- https://tanstack.com/query/latest/docs/framework/react/overview — server state.
- https://www.rfc-editor.org/rfc/rfc10017 — OAuth 2.0 for browser-based applications.
- https://www.w3.org/TR/WCAG22/ — WCAG 2.2.
- https://web.dev/articles/vitals — Core Web Vitals.
- https://testing-library.com/docs/queries/about/ — query priority.
