# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

_____

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| profiles | id (matches auth.users.id), role, is_internal | One profile per authenticated user. RLS restricts profile reads to id = auth.uid(); users cannot change their own role or internal status. |
| invites | id, user_id, sender_email, recipient_email, status, created_at, updated_at, accepted_at | Each row is owned by user_id, which must match the authenticated session. Allowed statuses are sent, accepted, declined, and cancelled; invitation summary views use these records. |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

Data: Profiles. Who can see it: The matching user only (profiles.id = auth.uid()). Who can modify it: The user may update allowed profile fields, but cannot change their own role or is_internal status.
Data: User preferences, sessions, events, experiment assignments, action state. Who can see it: The matching user only (user_id = auth.uid()). Who can modify it: The matching user, subject to row-level policies; event inserts must reference their own session.
Data: Invites. Who can see it: The owner only (invites.user_id = auth.uid()). Who can modify it: The owner can create and cancel invites. The addressed recipient can accept or decline through the status RPC, which returns success rather than the invite record.
Data: Shared content (actions, quotes, metrics, widgets, experiments, takeaways, priority metrics). Who can see it: Any authenticated user. Who can modify it: Internal editors only.
Data: Pivot decisions. Who can see it: Internal users only. Who can modify it: Internal users only.
Data: Invite summary and recent views. Who can see it: The authenticated owner’s invite records only. Who can modify it: Read-only views; changes go through the invite table/RPC.

Auth boundaries
1) The app requires email/password authentication. Legacy anonymous sessions are discarded; unconfirmed signups must confirm by email before logging in when confirmation is enabled.
2) Supabase RLS uses the authenticated JWT’s auth.uid() as the ownership check. Hiding UI controls is not an access boundary; RLS is.
3) The Supabase anon/publishable key is used in the browser and is not a secret. Never expose the service_role key in frontend code; it bypasses RLS.
4) Recipient actions are checked against the email claim in the authenticated JWT. Recipients can change invite status through the RPC but cannot read the sender-owned invite record.

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | The dashboard displayed hard-coded sample metrics, so a first-run user did not see a genuine empty state. | The invite dashboard queries Supabase. With no invite rows, counts show zero, rates without a denominator show “—”, and a Get Started CTA opens the invite form. |
| Bad / malicious input | There was no email login or invite input flow; telemetry used anonymous sessions. | Signup/login validate required email and an 8-character minimum password; invite email input uses browser email validation and is normalized. RLS restricts invite rows to user_id = auth.uid(). The status RPC authorizes the sender or recipient. Direct API writes still rely on the migrations’ RLS and database constraints; email-format validation is not duplicated as a database constraint. |
| Failure / offline | The dashboard’s hard-coded values kept displaying, so it could not reveal a data connection failure. | Failed invite-summary or recent-list queries show inline errors with Retry. During loading, skeletons appear. There is no offline cache for invite data, so unavailable data is not presented as current counts. Invalid or expired auth sends the user back through login. |

## Stress test results

_What you threw at it, and what held / broke._

_____
