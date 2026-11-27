# Changelog

Every release of apogee-session, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

### Fixes

- A Redis connection shared by several threads (the cache, session and queue-dashboard stores, used by a multi-threaded server or queue worker) no longer mixes up replies or stalls: every command is one locked round trip (grav-redis 0.2.2). Uses the new releases of grav-redis.

## 0.2.2 (2026-11-25)

### Dependencies

- Uses apogee-http 0.3.0, apogee-routing 0.3.0, apogee-validation 0.2.1 (from their GitHub tags).

## 0.2.1 (2026-11-20)

### Requirements

- Uses the 0.2.1 release of grav-redis.

## 0.2.0 (2026-11-20)

### Requirements

- Needs grav 1.9.0 or later, and uses the 0.2.0 releases of the packages it
  depends on.

## 0.1.0 (2026-11-19)

The first tagged release. Needs grav 1.7.0 or newer.

Sessions for [Apogee](https://github.com/venomous-maker): what a browser keeps across requests through a signed cookie, with flash data that lives exactly one more request, old input and errors for forms, a CSRF token, and memory, file and Redis stores.
