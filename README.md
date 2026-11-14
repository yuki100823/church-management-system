# Grace Church WeChat Mini Program

This project is located in `apps/member-client` and uses uni-app with Vue 3 and TypeScript. Phase A provides a standalone local demo with member and guest check-in, personal profiles, small groups, announcements, and Bible resources. The Giving, Library, Partner Churches, and Permissions Management entries indicate that those features are not yet available.

## Run on Windows 10

1. Install Node.js 24 LTS (which includes npm) and WeChat DevTools. You can edit the source code in VS Code.
2. Open PowerShell in the repository root and run the following commands.

   ```powershell
   cd apps/member-client
   node --version
   npm.cmd ci
   npm.cmd run type-check
   npm.cmd run dev:mp-weixin
   ```

3. After the build succeeds, select **Import Project** in WeChat DevTools and choose `apps/member-client/dist/dev/mp-weixin`. The demo uses the built-in test AppID `touristappid`. Sign in if prompted, then select the test AppID option and run the simulator.
4. Keep the development command running to watch for source changes. Press `Ctrl+C` when you finish the demo.

### Troubleshoot Installation

- `node --version` should show `v24.x`. If it still shows an older version, switch to Node.js 24, open a new PowerShell window, and check again.
- If `npm.cmd ci` fails with `EPERM` and the log points to an `esbuild.exe` process that is in use, press `Ctrl+C` in the terminal running the project's development command. Then run `npm.cmd ci` again.
- If `uni` or `vue-tsc` cannot be found, check that `npm.cmd ci` finished successfully. The project dependencies provide both commands.

## Try the Demo

1. **Welcome screen.** Select Chinese or English, then tap **Get Started**. Change languages from the top-right corner of Home or from **More → App Settings**. The app saves your language preference locally.
2. **Member check-in.** On Home, select **Existing Member Login**, then tap **Sign In as a Demo Member**. The app resumes check-in for the current session and shows the result and attendance counts by category. Return to Home and check in again to view the existing record.
3. **Guest check-in.** On Home, open **New Guest Registration**. Submitting an empty form shows which fields are required. Use **Fill In Demo Details** to populate a fictional name, email address, and first-visit selection. Submit the form to see the check-in result. The demo saves only the check-in credential. The name and contact details entered in the form are discarded.
4. **Access permissions.** **Members** displays only the signed-in member's own profile. After sign-in, **Groups** shows the White Rock group, and the **Notification Center** shows a sample prayer announcement. Sign out from **More → Account** to clear restricted content from these screens.
5. **Audio playback.** Go to **Resources → Old Testament**, choose **Psalms** and **Bible Audio**, then open **Quiet Listening**. This is an original instrumental track about 22 seconds long. You can play, pause, and drag the progress bar to seek. Leaving the detail screen stops playback and releases the audio player.

The app generates demo sessions using the current date in `America/Vancouver`. Check-in stays open throughout the local day. Each session ID includes its date and time slot. The app deduplicates member check-ins using a fixed demo account and guest check-ins using a device-local demo identity. Completed check-ins remain visible when you reopen the app. To reset the local demo, use **Clear Cache → Clear Data Cache** in WeChat DevTools.

## Build and Environment Configuration

| Command | Output Directory | Data Mode |
| --- | --- | --- |
| `npm.cmd run dev:mp-weixin` | `dist/dev/mp-weixin` | Local demo by default. Can be switched to API mode. |
| `npm.cmd run build:mp-weixin` | `dist/build/mp-weixin` | API mode. The release package excludes demo code and audio. |

Use `apps/member-client/.env.example` as a template. Copy it to `.env.development.local` or `.env.production.local` in the mini program project directory.

- **`VITE_DATA_MODE`**. Set this to `mock` or `api` for development builds. Release builds always use API mode.
- **`VITE_API_BASE_URL`**. Set the service origin. The client appends `/api/v1`. Release builds require HTTPS. On a phone, `localhost` points to the phone itself. For integration testing, use an address reachable from the phone and configure the allowed domains for API requests and audio in the WeChat administration console.
- **`WECHAT_APP_ID`**. Enter the actual mini program AppID. The build writes it to `project.config.json`. Previewing on a physical device requires a real AppID and the corresponding developer permissions.

If a request fails in HTTP mode, the app displays an error and lets the user retry. This repository currently has no NestJS API. Phase B requires a real AppID, a reachable API URL, a test member account and a method for linking it, and updates to the contracts in `src/services/contracts.ts` to match the actual OpenAPI definition. Real guest registration requires an event credential issued by the server. Open the page using `pages/events/guest-checkin?id=<session>&pass=<credential>`. The server manages the WeChat AppSecret, database credentials, and session signing keys.

## Source Code

- `src/pages` and `src/components` contain pages and interactions.
- `src/styles/app.css` and `src/i18n` contain shared design variables and bilingual strings.
- `src/services` contains shared service contracts, session handling, and the HTTP implementation.
- `src/mocks` and `demo-assets` provide local data, check-in behavior, and sample audio.

## Verification

Dependencies were reinstalled, types were checked, and development and release builds were run using Node.js 24.22.2. The built-in WeChat DevTools compiler passed the WXML and WXSS checks. The development build was used to verify sign-in and access scope, duplicate check-in submissions, restoration of check-in records, guest form validation, and language preference persistence.

Capturing the DevTools window timed out. Simulator interactions, audio playback, and testing on a physical device still need to be verified using the steps above. You can import the build through the DevTools interface without enabling the command-line service port.
