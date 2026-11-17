# MCP Security Checklist: tick-box version

Copy this into a pull request or review ticket. Tick an item only after you ran its test. The full text, tests, sources and the list of what was not verified are in the [README](README.md). Sections 1 to 3 are blockers. Sections 4 to 6 are hardening.


## 1. Tool descriptions and metadata (tool poisoning)

- [ ] **1.1** I read the full raw `tools/list` output, including descriptions and parameter descriptions, not only tool names.
- [ ] **1.2** No description contains instructions aimed at the model (for example "ignore previous", "always call X first", "do not tell the user").
- [ ] **1.3** No description contains hidden text such as zero-width characters or very long whitespace runs.
- [ ] **1.4** The client treats tool annotations as untrusted unless the server is trusted.

## 2. Tool changes after approval (rug pull)

- [ ] **2.1** The client stores a hash of each approved tool definition (name, description, schema) and re-checks it each session.
- [ ] **2.2** A changed definition requires re-approval.
- [ ] **2.3** Edits to an already-approved MCP config file require re-approval.
- [ ] **2.4** Server versions are pinned. No `latest`, no floating tags.
- [ ] **2.5** The tool set does not change per connection or as a side effect of other requests.

## 3. Authentication and authorization

- [ ] **3.1** The server publishes RFC 9728 Protected Resource Metadata.
- [ ] **3.2** The client sends the RFC 8707 `resource` parameter on both the authorization and token requests.
- [ ] **3.3** The server rejects a token issued for a different audience with HTTP 401.
- [ ] **3.4** No access token appears in a URI query string.
- [ ] **3.5** The server never accepts or forwards a token that was not issued for it (no token passthrough).
- [ ] **3.6** Possession of a session ID or state handle alone does not authenticate a request.
- [ ] **3.7** MCP proxy servers show a per-client consent page and require an exact `redirect_uri` string match.
- [ ] **3.8** Local servers bind to `127.0.0.1`, not `0.0.0.0`.

## 4. Client-side execution safety

- [ ] **4.1** The client allows only `http://` and `https://` authorization URLs and rejects `javascript:`, `data:`, `file:` and `vbscript:`.
- [ ] **4.2** The client never opens URLs through a shell.
- [ ] **4.3** One-click local server install shows the full, untruncated command and needs explicit approval.
- [ ] **4.4** MCP Inspector or proxy tooling is 0.14.1 or later and requires authentication between UI and proxy.

## 5. Server-side tool safety

- [ ] **5.1** Tool inputs are validated against their schema.
- [ ] **5.2** File paths are contained to allowed roots, including symlinks and sibling-prefix paths.
- [ ] **5.3** Tool arguments cannot inject CLI flags.
- [ ] **5.4** Tool invocations are rate limited and outputs are sanitized.
- [ ] **5.5** Tool output is treated as untrusted input to the model.

## 6. Supply chain and logging

- [ ] **6.1** The package comes from the publisher you expect. Check the exact name, not a lookalike.
- [ ] **6.2** Dependencies are locked and scanned.
- [ ] **6.3** Install scripts and post-install hooks were read.
- [ ] **6.4** Every tool call is logged with tool name, arguments, caller and result status, and logs hold no secrets.
- [ ] **6.5** One server or one tool can be disabled quickly, and a named owner exists for incidents.

Result: ____ of 31 items ticked. Reviewer: ________  Date: ________
