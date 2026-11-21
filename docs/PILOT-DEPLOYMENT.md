# KinaBot feedback pilot

This setup runs the React frontend and the local speech-analysis backend together on one paid Render service. The speech model runs on the server; participant audio is not sent to AWS or Gemini by this mode. Gemini insights are unavailable. The service has a single worker and is intended for a small invited pilot, not general public use.

## Before deployment

1. Review the changes and commit them in the `backend` and `frontend` repositories, then commit their updated submodule references and the root files in the main repository. Render must be able to access all three repositories, including private submodules. Never commit `.env`, local runtime files, recordings, or account data. Check the repository's existing tracked files as well as ignored files before pushing.
2. Create or sign in to a Render account and connect the main repository. Set a spending limit or monitor usage and check the current price for the **Standard** instance and 10 GB persistent disk before creating them. This is an initial resource estimate; model build size, memory use and transcription speed must be tested.
3. Create a Blueprint from the main repository's `render.yaml`. Render prompts for `PILOT_INVITE_CODE`: use a long random value of at least 16 characters and share it privately only with testers. Render generates `JWT_SECRET` automatically. The first container build downloads the speech model and may take time.
4. Once deployed, check `/api/health`, then create two test accounts and verify each can only see its own recordings, including when both upload the same file. Verify signup rejects an invalid invitation code, feedback appears on disk, and deletion removes the selected recording. Test microphone use, uploading and analysis on the browsers testers will use.

The Blueprint stores pilot data at `/var/data` on a persistent disk. Render disks preserve only the mounted directory and are limited to one running instance; deploys cause a brief interruption. A disk is not a complete backup or retention policy. Plan secure backups, access restrictions, and how to handle deletion from backups before collecting personal recordings. Do not enable public signup or scale this file-backed service to multiple instances.

## Running the pilot

- Share the HTTPS URL and invitation code with testers separately. The code is shared, so anyone who receives it can sign up; rotate it if it is exposed. Existing accounts continue to work.
- Make sure participants understand that their account details, audio, transcripts and results are stored on the pilot server and accessible to the service operator. Explain the pilot purpose, contact information, retention period, and deletion process before collecting data. Do not claim medical diagnostic capability.
- Feedback is stored in `/var/data/feedback.jsonl` with an account ID and timestamp. Review it through Render's authenticated service shell; do not post or share it publicly. Use the account ID in `/var/data/users.json` to identify who requested follow-up or account deletion. Testers can delete an individual recording and its analysis in Settings.
- Account deletion is a manual operator request through Feedback. To process one, set `PILOT_MAINTENANCE=true` in Render and redeploy; wait until the new deployment is healthy so all app requests are blocked. In the service shell run `node backend/scripts/delete-pilot-account.js 'tester@example.com'` using the account's email. The script removes the live account, recordings, analyses and feedback. Then remove the maintenance variable and redeploy. Handle any backups under the retention policy separately. Before the pilot, decide who responds to requests. Password reset is not automated.
- Keep access to Render restricted, review service logs and storage usage, and have a plan to respond to failures. Avoid collecting real participant data until the deployment, access controls and deletion process have been reviewed and tested for your study's requirements.

## Local development

The existing local development mode is unchanged unless `PILOT_MODE=true` is set. For an isolated pilot test, use a separate empty directory for `PILOT_DATA_DIR`, set `LOCAL_AUTH_ONLY=true`, `LOCAL_AUTH_STORAGE=true`, `JWT_SECRET` to a random value of at least 32 characters and `PILOT_INVITE_CODE` to a random value of at least 16 characters. Configure the frontend with `VITE_PILOT_MODE=true`; never put either secret in a frontend environment variable.
