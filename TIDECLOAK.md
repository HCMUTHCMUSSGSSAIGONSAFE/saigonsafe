# MindSafe × TideCloak — integration plan

MindSafe is built **as though TideCloak protects it**. In this demo, sign-in, roles and the network's access checks are **simulated in the browser**. Journal text is really encrypted with AES-256-GCM through Web Crypto, but that key stands in for Tide's ORK-held key.

The plan follows Tide's agent pack (Raziel MCP, `tide-foundation/raziel-mcp`). We used:
- scenario `relationship-attestation`;
- playbooks `configure-e2ee-roles-and-policies` and `setup-forseti-e2ee`;
- canon `ux-states`;
- template `react-vite-internal-dashboard`.

## Why this scenario

The discriminating question is: *"Must access to one record's data be limited to a named user, group or role, enforced at decryption?"* For MindSafe the answer is yes. One counselor may read one student's journal, read-only, for a limited time, and the admin must never read it.

So the model is the ORK's built-in contract **`RelationshipAttestation:1`**:
- one policy per counselor and record;
- signed by a `tide-realm-admin` in the Tide enclave;
- carrying an expiry.

## Realm `mindsafe` — roles

| Role | Kind | Holder | Purpose |
|---|---|---|---|
| `_tide_enabled` | realm (default) | everyone | Tide account |
| `student` | realm | students | UI and API gating of student routes |
| `counselor` | realm | counselors | UI and API gating of the counselor console |
| `researcher` | realm | research team | aggregates only, no decryption |
| `app-admin` | realm | IT admin | runs the system; **no** `_tide_decryptor` |
| `_tide_encryptor` | realm (fixed name) | students | needed to write to a record |
| `_tide_decryptor` | realm (fixed name) | students, counselors | necessary but **not sufficient** to read |
| `tide-realm-admin` | client role on `realm-management` | Wellbeing Board (threshold 2 of 3) | approves relationship policies in the enclave; cannot read |

Two rules from the pack:
- Do **not** put `_tide_encryptor` or `_tide_decryptor` in the default-roles composite.
- A role alone opens nothing, and a policy alone opens nothing. A reader needs both.

## Policies

| Field | Value |
|---|---|
| Contract | `RelationshipAttestation:1` (built in; nothing to deploy) |
| Model | `PolicyEnabledEncryption:1` / `PolicyEnabledDecryption:1` |
| `subjectType` / `subjectId` | `User` / the counselor's `tideuserkey` claim |
| `assignmentId` | the record id (e.g. `journal-sv204`), which is also the **only** tag on every item |
| `roles` | `["_tide_decryptor"]` for a counselor (read-only); both roles for the owner |
| Expiry | 7 days (2 minutes for the demo), so access ends on its own |

## Flows (what the demo simulates)

1. **Sign-in.** The user signs in on TideCloak's hosted login. The app receives a doken containing `realm_access.roles` and `tideuserkey`. The demo shows this through "View token".
2. **Write.** The student encrypts `[{ data, tags: [recordId] }]` using their owner policy.
3. **Consent.** The student picks a counselor and a duration, and the app records the consent. This happens either after the anonymity reminder or when the student approves a counselor's request.
4. **Grant.** A `tide-realm-admin` reviews the policy JSON and approves it in the Tide enclave. The network signs it and the server stores the signed bytes.
5. **Read.** The counselor fetches the policy and calls decrypt. On every call the network checks:
   - the caller holds `_tide_decryptor`;
   - the policy names the caller;
   - the tag equals `assignmentId`;
   - the policy permits decrypt;
   - the policy has not expired.

   The demo shows this checklist live.
6. **Admin attempt.** The admin has no policy and no `_tide_decryptor`, so the network refuses. The refusal is logged and visible to the student.
7. **Revocation.** The app stops serving the stored policy, and the expiry is the hard backstop. Per the pack, there is no instant per-record revocation for a `User` subject. Short expiries plus re-grants are the documented pattern.

## Swapping the simulation for the real SDK

- The app's `useTideCloak()` hook has the same shape as `@tidecloak/react` (`authenticated`, `login`, `logout`, `hasRealmRole`, `hasClientRole`, `isInitializing`).
- Wrap the app in `TideCloakContextProvider` with the exported adapter (`tidecloak.json`) and `useDPoP: { mode: 'strict', alg: 'ES256' }`.
- Replace `encryptText`/`decryptText` with `IAMService.doEncrypt` / `IAMService.doDecrypt` (`[{ data, tags: [recordId] }]`, signedPolicyBytes).
- Verify the JWT and DPoP proof on every API route with `@tidecloak/verify`. UI role checks are gating only and do not protect the API.
- Render the SDK's UX states (`isInitializing`, `sessionExpired`, `isOffline`), because encryption cannot work offline.

## What is not done yet (honest scope)

- There is no live TideCloak realm or ORK network behind this deployment, so all checks run in the browser.
- The enclave approval is a 1.4 s simulated step, and the threshold is shown as 2 of 3 but not enforced.
- Data lives in the browser's localStorage, not on a server.
