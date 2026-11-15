# apogee-session

Sessions for [Apogee](https://github.com/venomous-maker): what a browser keeps
across requests through a signed cookie, with flash data that lives exactly
one more request, old input and errors for forms, a CSRF token, and
memory, file and Redis stores.

```grav
import "std"
import "apogee-session"

let sessions = new SessionManager(new FileSessionStore("storage/sessions"), Config.getString("app.key"))
router.aliasInstance("session", new StartSession(sessions))
router.aliasInstance("csrf", new VerifyCsrfToken(["webhooks/*"]))
router.middlewareGroup("web", ["session", "csrf"])

let s = req.session as Session
s.put("cart", items)  s.get("cart")  s.push("cart", item)  s.increment("visits")  s.pull("coupon")
s.flash("status", "Saved!")      // the next request only
s.now("toast", "...")            // this request only;  s.reflash()  s.keep(["status"])
s.regenerate()                   // a new id on sign-in (no session fixation)
s.invalidate()                   // emptied, with a new id, on sign-out
s.token()                        // the CSRF token
```

## Stores

| | |
|---|---|
| `MemorySessionStore()` | this process, thread-safe (tests, development) |
| `FileSessionStore(dir)` | one file per session, written to a temp file and renamed; `gc` drops expired ones |
| `RedisSessionStore(conn, prefix = "session:")` | expiring by Redis's own TTL |

Your own: implement `SessionStore` (`read`, `write`, `destroy`, `gc`).
Sessions are stored as JSON, so their values are plain: null, bool,
numbers, strings, lists and maps.

## The middleware

`StartSession` reads the session its signed cookie names (an id the
server did not make, or a cookie that was changed, starts a new
session). It runs the request, remembers a page GET as the previous
URL, ages the flash data, saves the session, and sets the cookie:
HttpOnly, SameSite=Lax, Secure on HTTPS, for `lifetimeMinutes`, or until
the browser closes with `expireOnClose`. An error that renders itself (a
failed form) is rendered inside it, so the errors and old input it
flashed are saved. Expired sessions are collected on about one request
in `gcLottery`, at most once in `gcEverySeconds` (60), on a thread of its
own so no request waits. The file store stamps each file's modified time
with its expiry, so a sweep stats files instead of reading them.

`VerifyCsrfToken` requires the session's token on POST, PUT, PATCH and
DELETE, as a `_token` field, an `X-CSRF-TOKEN` header, or the
`X-XSRF-TOKEN` header that JavaScript clients copy from the `XSRF-TOKEN`
cookie it sets. It compares in constant time, and a mismatch is a 419.
Paths in `except` (globs) are skipped.

## Forms

A `ValidationException` (apogee-validation) flashes its errors and the old
input through the session, and redirects back. On the next request:
`s.error("email")`, `s.errors()`, `req.old("email")`.

## Tests

```bash
REDIS_URL=redis://127.0.0.1:6379/9 grav test tests/session.grav
```

MIT licensed.
