# Python client

This client follows the rules in `SKILL.md`. It uses the standard library, so
it needs no package. Adapt names & style to the project. Keep the rules.

## Contents

- Client
- Usage
- Smoke test

## Client

```python
import json
import os
import random
import time
import urllib.error
import urllib.parse
import urllib.request
import uuid

BASE_URL = "https://xquik.com/api/v1"
REQUEST_TIMEOUT_S = 30
RETRY_BUDGET_S = 300
MAX_BACKOFF_S = 30
DEFAULT_POLL_S = 2
RESTART_CODES = {"coverage_cursor_gone", "invalid_coverage_cursor"}


class XquikError(Exception):
    def __init__(self, status, code, message, retry_after=None):
        super().__init__(message)
        self.status, self.code, self.retry_after = status, code, retry_after


class _RefuseRedirects(urllib.request.HTTPRedirectHandler):
    def redirect_request(self, *args, **kwargs):
        return None


_OPENER = urllib.request.build_opener(_RefuseRedirects)


def _api_key():
    key = os.environ.get("XQUIK_API_KEY", "").strip()
    if not key:
        raise RuntimeError("Set the XQUIK_API_KEY environment variable.")
    return key


def _encode(value):
    return str(value).lower() if isinstance(value, bool) else value


def _send(method, path, query=None, body=None, idempotency_key=None):
    """Send 1 request. `path` starts after `/api/v1`, or is a `statusUrl`."""
    url = urllib.parse.urljoin(BASE_URL, path) if path.startswith("/api/v1/") else BASE_URL + path
    if not url.startswith("https://xquik.com/"):
        raise RuntimeError("Refusing to send the key to another host.")
    pairs = {k: _encode(v) for k, v in (query or {}).items() if v is not None}
    if pairs:
        url += "?" + urllib.parse.urlencode(pairs)
    headers = {"x-api-key": _api_key()}
    data = None
    if body is not None:
        data = json.dumps(body).encode()
        headers["content-type"] = "application/json"
    if idempotency_key:
        headers["idempotency-key"] = idempotency_key
    request = urllib.request.Request(url, data=data, headers=headers, method=method)
    try:
        with _OPENER.open(request, timeout=REQUEST_TIMEOUT_S) as response:
            status, raw, reply = response.status, response.read(), response.headers
    except urllib.error.HTTPError as error:
        status, raw, reply = error.code, error.read(), error.headers
    except (urllib.error.URLError, OSError):
        raise XquikError(0, "network_error", f"{method} {path} did not complete.") from None
    try:
        payload = json.loads(raw)
    except ValueError:
        payload = None
    if 200 <= status < 300 and isinstance(payload, dict):
        return payload
    code = payload.get("error") if isinstance(payload, dict) else None
    code = code if isinstance(code, str) else "invalid_response"
    retry_after = reply.get("Retry-After", "")
    raise XquikError(
        status,
        code,
        f"{method} {path} failed with {status} {code}.",
        float(retry_after) if retry_after.isdigit() and int(retry_after) > 0 else None,
    )


def _is_retryable(error):
    return (
        error.status in (0, 429)
        or error.status >= 500
        or error.code == "coverage_cursor_unavailable"
        or (error.code == "invalid_response" and error.status < 300)
    )


def xquik_get(path, query=None):
    """GET with retries. `path` starts after `/api/v1`, such as `/x/tweets/search`."""
    deadline = time.monotonic() + RETRY_BUDGET_S
    attempt = 0
    while True:
        try:
            return _send("GET", path, query=query)
        except XquikError as error:
            if not _is_retryable(error):
                raise
            backoff = min(MAX_BACKOFF_S, 2**attempt) * random.uniform(0.5, 1.0)
            wait = error.retry_after or backoff
            if time.monotonic() + wait > deadline:
                raise
            attempt += 1
            time.sleep(wait)


def paginate(path, query, rows_key, max_pages):
    """Yield each row once. The ceiling is the page size times `max_pages`, in rows."""
    seen_ids, seen_cursors = set(), set()
    cursor, restarted = None, False
    for _ in range(max_pages):
        try:
            body = xquik_get(path, {**query, "cursor": cursor})
        except XquikError as error:
            if error.code not in RESTART_CODES or cursor is None or restarted:
                raise
            cursor, restarted = None, True
            seen_cursors.clear()
            continue
        rows = body.get(rows_key)
        for row in rows if isinstance(rows, list) else []:
            row_id = row.get("id") if isinstance(row, dict) else None
            if isinstance(row_id, str) and row_id not in seen_ids:
                seen_ids.add(row_id)
                yield row
        cursor = body.get("next_cursor")
        if body.get("has_next_page") is not True or not isinstance(cursor, str) or not cursor:
            return
        if cursor in seen_cursors:
            raise XquikError(0, "repeated_cursor", f"{path} repeated a cursor.")
        seen_cursors.add(cursor)


def xquik_write(method, path, body, idempotency_key=None):
    """Send 1 write & poll it to its final state. Reuse the key only to resend it."""
    key = idempotency_key or str(uuid.uuid4())
    action = _send(method, path, body=body, idempotency_key=key)
    deadline = time.monotonic() + RETRY_BUDGET_S
    while action.get("terminal") is not True and time.monotonic() < deadline:
        status_url = action.get("statusUrl")
        if not isinstance(status_url, str):
            break
        poll_ms = action.get("pollAfterMs")
        time.sleep(poll_ms / 1000 if isinstance(poll_ms, (int, float)) else DEFAULT_POLL_S)
        action = xquik_get(status_url)
    return action
```

## Usage

```python
query = {"q": "solar panels", "language": "en", "retweets": "exclude", "limit": 100}
for tweet in paginate("/x/tweets/search", query, "tweets", max_pages=10):
    save_tweet(tweet)

action = xquik_write("POST", "/x/tweets", {"account": "my_brand", "text": "Hello"})
if action.get("status") != "success":
    raise RuntimeError(f"The write ended as {action.get('status')}.")
```

That search reads at most 100 rows times 10 pages. So it reads at most 1,000
tweets, which cost $0.15.

## Smoke test

The developer runs this smoke test. It proves the endpoint & the field names.
It prints counts & names, never tweet text. This one reads at most 5 tweets,
which cost $0.00075.

```python
page = xquik_get("/x/tweets/search", {"q": "from:nasa", "limit": 5})
rows = page.get("tweets") or []
print({
    "rows": len(rows),
    "fields": sorted(rows[0]) if rows else [],
    "has_next_page": page.get("has_next_page"),
})
```
