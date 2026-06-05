# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

CalDavZAP is a pure-JavaScript CalDAV web client (calendar + tasks) licensed under AGPLv3. It is a static single-page application with no build system, no package manager, and no test suite. There is no server-side component beyond the optional PHP auth module in `auth/`.

## Architecture

The app loads as a single HTML page (`index.html`) with HTML5 application cache (`cache.manifest`). JavaScript files are loaded in dependency order via `<script>` tags — the ordering in `index.html` lines 34-57 is the dependency chain.

**Core data flow:**

1. `main.js` initializes the app, sets up the event loop, and manages global state
2. `webdav_protocol.js` handles all HTTP/WebDAV/CalDAV communication (PROPFIND, REPORT, PUT, DELETE, MKCALENDAR, OPTIONS preflight for CORS)
3. `resource.js` manages CalDAV collection discovery and resource lifecycle
4. `vcalendar.js` (`EventList` class) and `vtodo.js` parse/serialize iCalendar VEVENT and VTODO data
5. `vcalendar_rfc_regex.js` provides RFC 5545 regex patterns used by the parsers
6. `data_process.js` handles recurrence expansion, date manipulation, sorting, and alarm processing
7. `interface.js` renders the UI (calendar views, event lists) using jQuery and FullCalendar
8. `forms.js` handles the event/todo editor dialog forms
9. `localization.js` contains all i18n strings; `timezones.js` (~1.5MB) is the Olson timezone database

**Libraries in `lib/`:** jQuery 2.1.4, jQuery UI 1.11.4, FullCalendar, Spectrum (color picker), rrule.js (recurrence expansion), jshash (SHA-256), IE base64 shim, and jQuery plugins (autosize, browser, quicksearch, placeholder).

## No build/CI infrastructure

This project has no `package.json`, bundler, linter, test framework, or CI pipeline. Development consists of editing static files and reloading them in a browser. There is nothing to install, build, or run locally — the files are served directly by a web server (Apache with mod_mime, mod_headers, mod_expires recommended).

## Post-change cache update

After editing any file, run `./cache_update.sh` (or manually modify the second line of `cache.manifest`) so browsers pick up the changes. The HTML5 app cache otherwise serves stale files.

## Configuration

All configuration is in `config.js`. Three mutually exclusive setup modes exist:

- **`globalAccountSettings`** — hardcoded credentials, no login screen. For intranet/home use only.
- **`globalNetworkCheckSettings`** — login screen, user enters credentials. Standard setup.
- **`globalNetworkAccountSettings`** — delegates auth to the `auth/` PHP module, which returns XML config. Avoids browser auth popups on wrong credentials.

Key config variables beyond setup mode: `globalTimeZone`, `globalSyncResourcesInterval`, `globalActiveCalendarCollections`, `globalActiveTodoCollections`, `globalInterfaceLanguage`, `globalBackgroundSync`.

## Auth module (`auth/`)

An optional PHP backend (requires PHP >= 5.3, basic auth only). The flow:

1. `auth/index.php` loads a plugin (configured via `auth/config.inc` → `$config['auth_method']`, default: `generic`)
2. The plugin proxies authentication against the real CalDAV server
3. On success, returns XML config equivalent to `globalAccountSettings`
4. CORS headers are added by `auth/cross_domain.inc`

Plugin files live in `auth/plugins/`. The `generic` plugin uses `auth/plugins/generic_conf.inc`.

## Key constraints

- **CORS:** Cross-domain setups require the CalDAV server to return proper `Access-Control-Allow-*` headers. See `readme.txt` section 3 or `misc/config_davical.txt` for Apache config to inject these headers.
- **Digest auth:** Unreliable across browsers. Basic auth over HTTPS is strongly recommended.
- **Self-signed certs:** JavaScript cannot prompt users to accept them. Users must manually visit the server URL in the browser first.
