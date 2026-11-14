# Church Management System WeChat Mini Program Development Specification

## 1 Task for Codex

Develop the WeChat mini program for the Grace Church management system in the current code repository. Use the supplied HTML prototype as the UI reference and build with **standard uni-app, Vue 3, and TypeScript**. The mini program lives in `apps/member-client`. Follow the repository's existing structure and build conventions where applicable.

**Implement Phase A by default.** Deliver an interactive mini program demo that can be opened in WeChat DevTools on Windows 10. If the repository already has a usable backend, reuse its existing endpoints and retain a standalone local demo mode. Phases B and C guide subsequent development.

Before starting, check `AGENTS.md`, `README.md`, `package.json`, the lockfile, and any existing source code. If the repository is empty, initialize it using the guidance in this document. If a real AppID, test account, or backend URL is unavailable, continue with the work you can verify locally and identify the outstanding configuration at handoff.

## 2 Reference Files

| File | Purpose |
| --- | --- |
| `Grace+Church+·+教会管理+App+原型.html` | Visual design, UI copy, page order, and interaction entry points |
| `教会管理系统需求文档.docx` | Church business requirements and role definitions |
| `Church_Management_Architecture_V1.md` | Backend, permissions, data models, file storage, and API design |

Place the reference files in `docs/reference/` or attach them directly to the Codex task. Use the HTML prototype as the primary reference for visual design and navigation. Use the architecture document as the primary reference for data, identity, and permissions. Replace the prototype's direct DOM interactions, simulated counters, and pop-ups with Vue components and UI states that reflect the operation's result. Retain the Chinese and English copy and add missing status messages.

## 3 Pages and Visual Design

Keep the prototype's five bottom navigation tabs. Their order is **Home → Members → Groups → Resources → More**. Follow the prototype for the welcome screen and language selection. Administrators maintain global settings such as the church name, and the client reads that configuration. Home keeps the green header, event check-in card, shortcuts to common services, announcements, and any statistics the user is allowed to see.

| Design Token | Prototype Value |
| --- | --- |
| Primary color | `#316b58` |
| Page background | `#fbf9f3` |
| Main text | `#20372e` |
| Secondary text | `#718279` |
| Check-in accent | `#c99848` |
| Card | `#ffffff`, rounded corners, and a subtle shadow |

Centralize colors, spacing, and corner radii. Build the screens with uni-app components and platform APIs. Support different phone screen sizes, the bottom safe area, and large text. Store the Chinese and English translations in language resources and persist the user's language preference. Provide loading, empty, error with retry, access denied, and submitting states. When an entry leads to an unavailable feature, show a clear status message.

Use the suggested paths below for an empty repository. Keep existing names if routes are already in place.

| Page | Suggested Path | Phase A Behavior |
| --- | --- | --- |
| Welcome and language | `pages/welcome/index` | Let users choose a language on first launch before continuing to Home. They can switch languages from More. |
| Home | `pages/home/index` | Show the church, today's events, check-in, shortcuts to common services, and announcements. |
| Member sign-in | `pages/auth/index` | Start a session for a fixed demo member, then resume the action that prompted sign-in. |
| Event details and check-in | `pages/events/detail` | Show the session, time, and check-in status. Handle successful check-in and repeat submissions. |
| Guest registration | `pages/events/guest-checkin` | Include fields for name, contact details, and whether this is the guest's first visit. Use fictional data to demonstrate validation and check-in. |
| Members | `pages/members/index` | Show the signed-in member's own profile. Provide a sign-in entry for users without a session. |
| Groups | `pages/groups/index` | Show sample areas, meeting locations, and Bible reading progress within the user's access scope. |
| Resources | `pages/resources/index` | Browse sample resources by Old or New Testament, book of the Bible, and resource type. |
| Resource details | `pages/resources/detail` | Provide at least one test audio track that can be played and paused, with its playback state shown. |
| Announcements | `pages/notices/index`, `pages/notices/detail` | Open the list and individual announcements. |
| More | `pages/more/index` | Include language, account status, and sign-out. Preserve the prototype's entries for later features. |

Keep the prototype's entry locations for Announcements, Library, Giving, Partner Churches, and other features. Implement announcement viewing in Phase A. Implement giving summaries, library borrowing, and the partner church directory in Phase C.

## 4 Development Phases

### Phase A Local Mini Program Demo

1. Set up the WeChat mini program build target, page routes, design tokens, and Chinese and English language switching.
2. Prepare fictional church, member, session, group, announcement, and resource data. Generate demo sessions for the current date in the church's time zone so check-in remains available as the date changes.
3. Serve demo data through a shared data access layer. Pages should call the same service methods when switching to HTTP so they can be reused.
4. Let demo members sign in and check in, and let guests register and check in. Persist demo state locally so users can see their check-in results after reopening the app. Count repeat submissions for the same demo identity and session only once.
5. Include at least one original or authorized test audio track. Handle loading, playback, pause, errors, and cleanup when the user leaves the page.
6. Generate the WeChat mini program build output. Document dependency installation, compilation, import, and demo steps on Windows 10.

The development configuration can enable `mock` mode. It uses fictional data and runs without PostgreSQL, AWS, or actual WeChat secrets. Use the development build for the Phase A demo and document how to enable it in the README. In live API mode, display request errors and avoid silently falling back to demo data. Release builds must disable demo sign-in and demo data mode.

### Phase B Live Sign-In and Check-In Integration

1. Use the existing NestJS API. If there is no backend yet, start with sign-in, member profiles, event sessions, and check-ins in `apps/api`, including PostgreSQL migrations and development seed data. The iOS app, Android app, and admin portal will share this API.
2. The mini program calls `wx.login` or the uni-app WeChat sign-in API and sends the temporary `code` to the server. The server uses its WeChat credentials to exchange the code for an identity and return an application session. Keep the WeChat AppSecret, session keys, and database credentials on the server.
3. Distinguish guests, signed-in users with no linked member profile, and users with a linked profile. Link profiles through a validated invitation or by verifying an existing account. A WeChat identity and an email sign-in may map to the same internal account. Avoid automatically merging records by name or using the last four digits of a phone number as a password.
4. Connect Home, personal profiles, groups, and event check-in to the live API. Use a test database to verify that repeat requests for the same member and session produce a single check-in record.
5. Test sign-in, allowed domains, HTTPS, and network error messages in both WeChat DevTools and on a physical device. Obtain the WeChat AppID and access settings from the project configuration.

### Phase C Resources and Remaining Features

Connect live announcements, Bible resources, and media playback. Administrators upload audio through the admin portal. The server checks access and issues a short-lived playback URL. Then implement giving summaries, assets, library borrowing, and the partner church directory. Start with in-app notifications. For WeChat subscription messages, use verified templates and obtain user consent. If class bookings, online giving, or support for multiple churches is requested, define the corresponding workflows and permissions first.

## 5 Business Rules

### Identity and Visibility

- `users` represents sign-in accounts and `members` represents church member profiles. An account can link multiple sign-in methods.
- Home may return information approved for public display. Regular members can see only their own profile by default. Group leaders can see only the fields they are authorized to view for their own group. Finance, intercessory prayer, data export, and administration require separate permissions.
- Use client session state to control what the UI shows. The server authorizes each request based on the account, church, and resource, and identifies the person making the request from the authenticated session.
- When the user signs out or the session expires, clear the session and cached data for restricted pages. Fetch the data again after sign-in.

### Check-In

- Use `eventSessionId` to distinguish occurrences of the same event on different dates or in different time slots. Display times in the church's time zone. Use `America/Vancouver` for development data for now.
- For member check-in, the server identifies the member and verifies the session and check-in window. A database uniqueness constraint on the session and participant ensures retries and concurrent requests count as one check-in.
- For repeat check-ins, return the existing record and a clear message. Allow users to retry after a timeout or connection loss, then update the page using the server's result.
- Guest registration collects only the information needed for the session. In a live environment, protect registration with an event credential and rate limiting. The demo deduplicates by a device-local identity. Cross-device deduplication requires a real participant identifier.
- Store the participant's category at the time of check-in. If a guest later becomes a member, keep the historical attendance categories for past sessions. Regular users can view only the check-in results or totals they are authorized to access.

### Audio

- The resource list includes category, title, type, language, and visibility. Request an authorized playback URL from the backend when the user starts playback.
- Support pausing, progress, error handling, and cleanup when leaving the page. If the signed URL expires, request a new one and resume from the previous position where possible.
- Development may use local test files. Use private object storage in the live environment. The client receives only an authorized playback URL, and the database stores file identifiers and metadata.

## 6 API Conventions

The endpoints below come from the architecture plan and include the fields the pages need. For an existing API, use its actual OpenAPI contract. If there is no contract yet, define TypeScript types and use those same types in demo mode.

| Method and Path | Request and Response Details |
| --- | --- |
| `POST /api/v1/auth/wechat` | Accept the temporary WeChat `code`. Return an application session and membership linking status. |
| `POST /api/v1/auth/link-wechat` | Verify the current account and WeChat identity, then link them. |
| `GET /api/v1/me` | Return the current identity, member profile summary, and permitted actions. |
| `GET /api/v1/home` | Return church display information, today's event sessions, visible announcements, and permitted statistics. |
| `GET /api/v1/event-sessions` | Filter sessions by date or event. Return session IDs, times, locations, and check-in status. |
| `POST /api/v1/event-sessions/{id}/checkins` | Return the check-in record ID, timestamp, and whether the participant had already checked in. Identify the caller from the session. |
| `POST /api/v1/event-sessions/{id}/guest-checkins` | Accept guest registration details and the event credential. Return the check-in result. |
| `GET /api/v1/groups` | Return groups, areas, meeting locations, and reading progress visible to the user. |
| `GET /api/v1/notices` | Return published announcements visible to the user. Add a detail endpoint if needed. |
| `GET /api/v1/resources` | Return categories, resource types, languages, and visible metadata. |
| `POST /api/v1/resources/{id}/playback` | Return the authorized playback URL and its expiration time. |

Centralize the API URL, session authentication headers, timeouts, error codes, and retries. Use stable error codes such as `CHECKIN_WINDOW_CLOSED` to display Chinese and English messages. Handle unauthenticated and unauthorized requests separately. Keep tokens, signed URLs, and members' personal information out of logs.

## 7 Project Structure and Windows 10 Setup

| Location | Contents |
| --- | --- |
| `apps/member-client` | uni-app project, page routes, app configuration, and local build scripts |
| `apps/member-client/src/pages` | Pages and page-level interactions |
| `apps/member-client/src/components` | Reusable cards, buttons, status components, and audio controls |
| `apps/member-client/src/services` | Business requests, HTTP client, and session interfaces |
| `apps/member-client/src/mocks` | Demo data and implementations for local development only |
| `apps/member-client/src/i18n` | Chinese and English strings and language preferences |
| `apps/api` | NestJS service used during backend integration |

Use Git, VS Code, Node.js, HBuilderX, and WeChat DevTools for Windows 10 development. Set up PostgreSQL when connecting a real local backend. Follow the repository's configured Node.js major version and package manager. For an empty repository, start with Node.js 24 LTS as the proposed version, then verify the uni-app template and dependencies before pinning a compatible version.

For a CLI project, provide the following scripts or document the repository's equivalent commands in the README. Run them from the mini program project directory or the workspace specified by the repository.

```powershell
npm.cmd ci
npm.cmd run dev:mp-weixin
npm.cmd run build:mp-weixin
```

Use `npm.cmd ci` for npm projects that already have a `package-lock.json`. Generate a lockfile during initial setup. If the project uses another package manager, keep its existing lockfile. For an HBuilderX workflow, document how to import the project and launch it in WeChat DevTools. Specify the actual build output directory for import into WeChat DevTools. Record the simulator's development configuration and the AppID required for physical-device testing.

Centralize environment mode, API URL, and public AppID in configuration and commit a sample file with no secrets. Configure the API URL separately for DevTools and phones as needed. On a phone, `localhost` points to the phone itself. For integration testing, use a development API the phone can reach and follow WeChat's requirements for HTTPS and allowed-domain checks.

## 8 Acceptance and Handoff

| Scenario | Phase A Acceptance Criteria |
| --- | --- |
| Build | The project generates WeChat mini program build output that can be imported into WeChat DevTools. |
| Navigation | After choosing a language on first launch, the user can open Home. The five bottom tabs appear in the same order as the prototype. |
| Bilingual UI | Navigation and forms remain readable after switching languages. The app retains the language preference after reopening. |
| Member check-in | A signed-in demo member can check in for the current session. Repeated taps and reopening the app leave only one check-in record. |
| Guest check-in | The form validates required fields and displays the result for the current session after submission. |
| Personal profile | Member details are hidden when signed out. A demo member can see only their own profile. |
| Groups and announcements | Populated, empty, and error states are distinguishable. Users can open individual announcements. |
| Resources | Test audio can play and pause. Playback behaves as designed when the user leaves the page. |
| Documentation | A new developer can follow the README to reproduce the build on Windows 10 and find the directory to import. |

In Phase B, verify live WeChat sign-in, member profile linking, session expiration, unauthorized access, closed check-in windows, and concurrent duplicate check-ins. In Phase C, verify authorization for private audio, resuming playback after a URL expires, and denial when the user lacks access.

At handoff, report the changed paths, commands actually run, verification results, and configuration the user still needs to provide. Record build, simulator, and physical-device verification separately. Include a dependency lockfile for reproducible builds, sample configuration, and demo instructions.
