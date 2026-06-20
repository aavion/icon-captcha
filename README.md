# IconCaptcha

> **Status:** Active development  
> **Studio dependency:** `>= 0.2.6`  
> **Scope:** `captcha-provider`

IconCaptcha is planned as a visual captcha provider for Studio. The idea is simple: Studio shows a small image challenge, the visitor matches an emoji prompt with the right icon, and the server validates the answer without exposing the solution in browser-readable data.

This package is not a finished captcha provider yet. Right now it mainly contains the curated private challenge assets and the extension metadata needed to let Studio validate the package shape.

## What Is Included

The current package contains:

- `.manifest` with the extension identity and `captcha-provider` scope.
- `private-assets/index.json` with 100 curated emoji/icon pairs.
- Noto Emoji SVG assets under `private-assets/emojis/`.
- Tabler Icons Filled SVG assets under `private-assets/icons/`.
- License files for the bundled asset sources.
- Placeholder runtime files that intentionally do not register a provider yet.

The private assets are not meant to be copied into Studio's public asset map. They are source material for a later server-side challenge renderer.

## Challenge Asset Index

`private-assets/index.json` is the source of truth for challenge items. Each item has a stable readable id, a label, one emoji asset, one matching icon asset, and optional families for near-confusable items.

The documented selection policy currently expects:

- `challengeSize: 6`
- at least 4 categories per challenge
- at most 2 items from the same category
- at most 1 item from the same visual or semantic family

This keeps each challenge varied enough for humans, while avoiding choices that are too easy to identify from category alone or too similar to tell apart reliably.

## Planned Runtime Shape

The future provider should render each challenge server-side and return only the rendered challenge image plus a one-shot token. The browser should not receive item ids, labels, filenames, icon names, the correct answer, or other metadata that would make the challenge easy to solve without visual rendering.

The current design direction is:

- generate multiple challenges per request, including active, reserve, and honeypot entries;
- use a stable one-shot token for server-side validation;
- consume the token after one validation attempt;
- rasterize emoji and icons through the same image pipeline before delivery;
- keep challenge answers and source asset details private to the server.

This README intentionally describes the direction only. The runtime provider contract, API payloads, renderer, validation flow, and tests still need to be implemented.

## Package Layout

```text
extensions/icon-captcha/
├── .manifest
├── extension.php
├── assets/
├── languages/
├── private-assets/
│   ├── index.json
│   ├── emojis/
│   └── icons/
├── src/
├── templates/
└── tests/
```

Use `assets/` only for files that may be synced publicly by Studio. Use `private-assets/` for challenge source assets that must stay server-side.

## Development Notes

When changing the curated challenge set:

- keep `itemsCount` in sync with the actual item count;
- keep every `emojiAsset` and `iconAsset` path valid;
- keep license files in place;
- review category and family balance;
- avoid flags, brands, skin-tone variants, text-like symbols, highly similar faces, and items that are hard to distinguish at small sizes.

When runtime code is added, it should use documented Studio extension contracts instead of direct access to sensitive internals. Security-sensitive data must stay out of HTML, JavaScript, public assets, logs, and translation strings.

## Verification

Run verification from the Studio project root so extension validation, translation aggregation, and host rules are active.

Useful checks while this package is still mostly assets:

```bash
bin/lint extensions/icon-captcha
bin/phpunit
```

Additional focused tests should be added when the captcha provider, renderer, token flow, or API/live endpoint integration is implemented.
