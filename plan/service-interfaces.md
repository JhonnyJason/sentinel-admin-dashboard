# Service Interface Specification

Interface contracts for the three remote services consumed by the Sentinel Admin Dashboard.
This document is intended for the development teams implementing these services.

---

## 1. Sentinel Access Manager

**Base URL:** `https://sentinel-access-manager.dotv.ee`
**Protocol:** HTTPS (POST, JSON, CORS)

### 1.1 General Request Pattern

All endpoints accept `POST` with `Content-Type: application/json`.
CORS must be enabled (`Access-Control-Allow-Origin` for the dashboard origin).

**Response conventions:**
- `200` with JSON body on success
- `204` (no body) on success where no data is returned
- Any other status: body is a plain-text error message

### 1.2 `POST /registerAdmin`

**Purpose:** Register a new admin key pair. Called once during initial key setup.

**Status: Not yet active.** The dashboard currently stubs this call (waits 3s, no network request). This is the first endpoint that needs to be implemented for the full auth flow to work.

**Request body:**
```json
{
  "publicKey":  "<hex string, 64 chars - curve25519 public key>",
  "otc":        "<hex string, 32 chars - one-time-code from invitation URL>",
  "secret":     "<hex string, 64 chars - sha256(otc + pin + pwdSalt)>",
  "timestamp":  "<string - Date.now() as string>",
  "signature":  "<hex string - ed25519 signature>"
}
```

**Signature construction:**
1. Build payload with `signature: ""`
2. `JSON.stringify()` the payload
3. Sign the JSON string with the admin's private key (ed25519 via `thingy-crypto-node`)
4. Set `signature` to the resulting hex string

**Server must:**
1. Verify `otc` is a valid, unused one-time-code
2. Verify `secret == sha256(otc + pin + pwdSalt)` — the server knows the PIN and salt
3. Verify `signature` against `publicKey` (re-serialize payload with `signature: ""` to get the signed content)
4. Verify `timestamp` is recent (prevent replay)
5. Store `publicKey` as an authorized admin key
6. Invalidate the `otc` (single-use)

**Success:** `204` (no body)
**Error:** non-200 status with plain-text error message

---

### 1.3 User-facing endpoints (inherited from Sentinel Dashboard)

These endpoints exist in the scimodule but are **not actively used** by the Admin Dashboard. They are from a shared codebase. Listed here for completeness — the Admin Dashboard uses key-based auth, not session-based auth.

| Endpoint | Request Body | Response |
|---|---|---|
| `POST /register` | `"email"` (plain JSON string) | `204` |
| `POST /login` | `{ "email": "<email>", "passwordSH": "<hex64>" }` | `200` + JSON |
| `POST /loginX` | `{ "email": "<email>", "passwordSHX": "<hex64>" }` | `200` + JSON |
| `POST /refreshSession` | `"<hex32>"` (authCode) | `200` + JSON |
| `POST /logout` | `"<hex32>"` (authCode) | `204` |
| `POST /requestPasswordReset` | `"email"` | `204` |
| `POST /updateEmail` | `{ "newEmail", "email", "passwordSH" }` | `200` + JSON |
| `POST /updatePassword` | `{ "newPasswordSH", "email", "passwordSH" }` | `200` + JSON |
| `POST /deleteAccount` | (not used) | (not used) |

---

## 2. Sentinel Backend

**Base URL:** `https://sentinel-backend.dotv.ee`
**Protocol:** WebSocket (upgrade from HTTPS)

The dashboard opens a single persistent WebSocket connection to the backend.

### 2.1 Wire Format

**Client -> Server:**
- Without payload: `"commandName"`
- With payload: `"commandName <JSON>"`
  - One space between command name and JSON
  - Example: `saveEntry {"name":"Experiment 1","snapshot":{...}}`

**Server -> Client:**
- Always JSON with a `type` field for routing
- Example: `{"type":"allMakroData","payload":{...}}`

### 2.2 Connection & Authorization Flow

```
1. Client opens WebSocket to backend URL
2. On open, client sends:   authorizeAdmin <authMessage>
3. Server validates, responds:  {"type":"authorizationApproved"}
4. Client sends:            getAllMakroData
5. Server responds:         {"type":"allMakroData","payload":{...}}
6. Client sends:            getAllHistory
7. Server responds:         {"type":"allHistory","entries":{...},"published":{...}}
8. Session established — mutation commands follow as needed
```

### 2.3 Authorization Message (`authorizeAdmin`)

The client sends `authorizeAdmin <JSON>` immediately on socket open.

**Payload structure:**
```json
{
  "randomHex":  "<hex string, 96 chars - random nonce>",
  "timestamp":  "<validatable timestamp string>",
  "publicKey":  "<hex string, 64 chars - admin's public key>",
  "signature":  "<hex string - ed25519 signature>"
}
```

**Signature construction:**
1. Build payload with `signature: ""`
2. `JSON.stringify()` the payload
3. Sign with admin's private key
4. Replace `signature:""` with `signature:"<sig>"` in the string

**Server must:**
1. Verify `publicKey` is a registered admin key (registered via Access Manager)
2. Verify `timestamp` is recent (prevent replay)
3. Verify `signature` against `publicKey`
4. Respond with `{"type":"authorizationApproved"}` on success
5. On failure: close the socket (or send an error and close)

### 2.4 `getAllMakroData`

**Command:** `getAllMakroData` (no payload)
**Response type:** `allMakroData`

**Response:**
```json
{
  "type": "allMakroData",
  "payload": {
    "<areaKey>": {
      "infl": <number>,
      "inflMeta": { "dataSet": "<string>", "source": "<string>", "date": "<string>" },
      "mrr": <number>,
      "mrrMeta": { "dataSet": "<string>", "source": "<string>", "date": "<string>" },
      "gdpg": <number>,
      "gdpgMeta": { "dataSet": "<string>", "source": "<string>", "date": "<string>" },
      "cot6": <number>,
      "cot36": <number>
    }
  }
}
```

**Area keys (all required):**
`eurozone`, `usa`, `japan`, `uk`, `canada`, `australia`, `switzerland`, `newzealand`

**Field descriptions:**

| Field | Type | Description |
|---|---|---|
| `infl` | number | Inflation rate |
| `inflMeta` | object | Source metadata for inflation |
| `mrr` | number | Main refinancing rate / key interest rate |
| `mrrMeta` | object | Source metadata for interest rate |
| `gdpg` | number | GDP growth rate |
| `gdpgMeta` | object | Source metadata for GDP growth |
| `cot6` | number | COT value (6-week) |
| `cot36` | number | COT value (36-week) |

Meta objects are optional (the dashboard handles their absence). The numeric values are required.

### 2.5 `getAllHistory`

**Command:** `getAllHistory` (no payload)
**Response type:** `allHistory`

**Response:**
```json
{
  "type": "allHistory",
  "entries": {
    "<experimentName>": [ <snapshot_v0>, <snapshot_v1>, ... ]
  },
  "published": { "name": "<experimentName>", "version": <int> } | null
}
```

- `entries`: map of experiment name -> ordered array of snapshots (index = version number)
- `published`: which experiment+version is currently active for end users. `null` if nothing published.
- If no experiments exist yet, `entries` should be `{}` (empty object).

### 2.6 Snapshot Structure

Every snapshot (stored in version arrays, sent in create/save commands) has this shape:

```json
{
  "areaParams": {
    "<areaKey>": {
      "infl": { "a": <number>, "b": <number>, "c": <number> },
      "mrr":  { "f": <number>, "n": <number>, "c": <number>, "s": <number> },
      "gdpg": { "a": <number>, "b": <number>, "c": <number> },
      "cot":  { "n": <number>, "e": <number> }
    }
  },
  "globalParams": {
    "diffCurves": {
      "infl": { "b": <number>, "d": <number> },
      "mrr":  { "b": <number>, "d": <number> },
      "gdpg": { "b": <number>, "d": <number> },
      "cot":  { "b": <number>, "d": <number> }
    },
    "finalWeights": {
      "st": { "i": <number>, "l": <number>, "g": <number>, "c": <number>, "f": <number> },
      "ml": { "i": <number>, "l": <number>, "g": <number>, "c": <number>, "f": <number> },
      "lt": { "i": <number>, "l": <number>, "g": <number>, "c": <number>, "f": <number> }
    }
  }
}
```

Area keys in `areaParams`: same 8 areas as in `allMakroData`.

The backend stores snapshots as opaque JSON blobs — it does not need to interpret the parameter values. It only needs to store and retrieve them faithfully.

### 2.7 Mutation Commands

All mutation commands follow the same response pattern:

**Success:** `{ "type": "<command>Result", "ok": true }`
**Error:** `{ "type": "<command>Result", "ok": false, "message": "<error description>" }`

The dashboard has a **10-second timeout** on all commands. If no response arrives within 10s, the client treats it as a failure.

#### `createEntry`

Create a new named experiment with its initial snapshot (version 0).

**Command:** `createEntry {"name":"<string>","snapshot":<snapshot>}`

**Server must:**
- Store the snapshot as version 0 under the given name
- Reject if an experiment with that name already exists

#### `saveEntry`

Append a new version to an existing experiment.

**Command:** `saveEntry {"name":"<string>","snapshot":<snapshot>}`

**Server must:**
- Append snapshot as the next version to the named experiment
- Reject if the experiment does not exist

#### `publishEntry`

Mark a specific experiment+version as the published (active) configuration.

**Command:** `publishEntry {"name":"<string>","version":<int>}`

**Server must:**
- Set the published pointer to the given name+version
- Reject if the experiment or version does not exist
- Only one experiment+version can be published at a time

#### `renameEntry`

Rename an existing experiment.

**Command:** `renameEntry {"oldName":"<string>","newName":"<string>"}`

**Server must:**
- Rename the experiment, preserving all versions
- Update the published pointer if it referenced the old name
- Reject if `oldName` does not exist or `newName` is already taken

### 2.8 Future: Server-Push Updates

Not yet consumed by the dashboard, but the data contract reserves this pattern for future use:

```json
{ "type": "areaData", "key": "<areaKey>", "data": { "infl": <number>, ... } }
```

This would allow the backend to push updated makro data without the client re-requesting.

---

## 3. Sentinel Datahub

**Base URL:** `https://sentinel-datahub.dotv.ee`
**Protocol:** HTTPS (POST, JSON, CORS)

### 3.1 `POST /getEODHLCData`

**Purpose:** Retrieve historical end-of-day HLOC data for a symbol.

**Status:** Endpoint is defined and validated but **not actively called** by the Admin Dashboard (the import of `getAuthCode` is commented out). It is used by the public Sentinel Dashboard.

**Request body:**
```json
{
  "authCode":  "<hex string, 32 chars - session auth code>",
  "dataKey":   "<non-empty string - symbol identifier>",
  "yearsBack": <number or null - how many years of history>
}
```

**Expected response (`200`):**
```json
{
  "meta": {
    "startDate": "<string>",
    "endDate": "<string>",
    "interval": "1d",
    "historyComplete": <boolean>
  },
  "data": [ <array of HLOC entries> ]
}
```

### 3.2 Symbol Search (not yet implemented server-side)

The dashboard has a `getSymbolOptions` function that currently returns mock data instead of calling the server. When implemented:

**Endpoint:** TBD (no URL defined yet)

**Request body:**
```json
{
  "authCode": "<hex string, 32 chars>",
  "query":    "<non-empty string - search term>",
  "limit":    <number - max results>
}
```

**Expected response:** Array of symbol option objects (exact schema TBD).

---

## Summary: What Each Service Needs to Implement

### Access Manager — Priority: HIGH
| Endpoint | Status | Action Required |
|---|---|---|
| `POST /registerAdmin` | **Stubbed** | Implement admin key registration with OTC+secret verification and signature validation |
| Other user endpoints | Existing | Already implemented for public dashboard |

### Backend — Priority: HIGH
| Feature | Status | Action Required |
|---|---|---|
| WebSocket server | **Needed** | Accept WebSocket connections on base URL |
| `authorizeAdmin` | **Needed** | Validate admin auth message (publicKey + signature verification against registered keys) |
| `getAllMakroData` | **Needed** | Return current makro data for all 8 economic areas |
| `getAllHistory` | **Needed** | Return full experiment store (all names, all versions, published pointer) |
| `createEntry` | **Needed** | Store new experiment with initial snapshot |
| `saveEntry` | **Needed** | Append version to existing experiment |
| `publishEntry` | **Needed** | Set published pointer |
| `renameEntry` | **Needed** | Rename experiment, update published pointer if affected |

### Datahub — Priority: LOW (not used by Admin Dashboard)
| Endpoint | Status | Action Required |
|---|---|---|
| `POST /getEODHLCData` | Existing | Already implemented for public dashboard |
| Symbol search | **Not defined** | Future - endpoint URL and response schema TBD |
