# Connector Recovery Guide

An open, zero-dependency browser tool produced by Jarvis. It classifies normalized connector failures without sending input anywhere.

A zero-dependency, local-first classifier for normalized connector failures. It does not make network requests, access credentials, or return/store the original failure payload.

## Classification contract

`classify(input)` returns only:

```json
{
  "category": "TRANSIENT",
  "explanation": "The failure appears temporary and may succeed when retried.",
  "reasonCode": "RETRYABLE_CONNECTOR_FAILURE",
  "nextAction": "Retry the connector request with backoff."
}
```

| Category | Typical signals | Reason code | Next action |
| --- | --- | --- | --- |
| `TRANSIENT` | HTTP 408, 425, 429, 5xx; timeout/rate-limit codes | `RETRYABLE_CONNECTOR_FAILURE` | Retry the connector request with backoff. |
| `RENEWAL_REQUIRED` | HTTP 401; expired/revoked authorization codes | `CONNECTOR_AUTHORIZATION_EXPIRED` | Reauthorize the connector. |
| `PERMISSION_MISMATCH` | HTTP 403; denied/missing permission or scope | `CONNECTOR_PERMISSION_DENIED` | Review and grant the required connector permissions. |
| `UNKNOWN` | Unrecognized, malformed, or conflicting input | `UNRECOGNIZED_CONNECTOR_FAILURE` | Review the normalized failure details locally. |

Every result includes a concise explanation, a stable machine-readable `reasonCode`, and one plain-language `nextAction`. `UNKNOWN` is deliberately explicit; the classifier does not guess when signals are unrecognized or point to multiple categories. Supported normalized fields are `status`, `statusCode`, `httpStatus`, `code`, `errorCode`, `message`, and nested `error.code` / `error.message`.

## Local CLI

Pipe one JSON object over stdin so sensitive input is not placed in command-line arguments:

```sh
echo '{"status":503}' | npm start
```

The diagnosis is printed as JSON and an allowlisted event is appended to `connector-events.jsonl` in the current directory. Select another local event file with:

```sh
echo '{"status":401}' | node src/cli.js --events ./diagnoses.jsonl
```

The JSONL event contains only a timestamp, event name, category, reason code, and recommended action. Raw input, messages, connector identifiers, tokens, and credentials are never recorded. Protect the input source and event-file directory according to local policy.

## Tests

```sh
npm test
```

## Browser version

Open `index.html` directly or visit the GitHub Pages deployment. Classification runs entirely in the browser. There are no accounts, cookies, trackers, analytics, forms, external scripts, or network requests.

This public repository intentionally contains only `index.html`, `app.js`, `styles.css`, `README.md`, and `robots.txt`. It contains no credentials, raw connector payloads, local event files, or private Jarvis data.
