# MCP Security Checklist

A one-page, copy-paste checklist for reviewing a Model Context Protocol (MCP) server or client before you connect an agent to it. Every item is a yes/no question you can answer in a few minutes, with a quick way to test it.

Use it as a pull-request gate, a pre-install review, or a security-review template.

> Scope: this is a practical review aid, not a standard. Items marked with a MITRE ATLAS ID map to techniques in the ATLAS knowledge base (data v5.6.0 at time of writing).

## How this differs from other checklists

Other MCP checklists exist and are worth reading, for example [SlowMist's MCP Security Checklist](https://github.com/slowmist/MCP-Security-Checklist). This one is built as a review gate:

- Every item is a yes/no question with a way to test it.
- Sections 1 to 3 are blockers and 4 to 6 are hardening, so you get a pass/fail result.
- Items map to MITRE ATLAS technique IDs where one exists.

## How to use it

1. Copy the checklist below into an issue or PR description.
2. Tick each box only after you tested it, not after you read the docs.
3. Any unchecked item in sections 1 to 3 is a blocker. Sections 4 to 6 are hardening.

## 1. Tool descriptions and metadata (tool poisoning)

Tool descriptions are text the model reads as trusted context, so hidden instructions in them can steer the agent. ATLAS: `AML.T0110`.

- [ ] I read the full raw `tools/list` output, including descriptions, parameter descriptions and defaults, not only tool names.
- [ ] No description contains instructions aimed at the model (for example "ignore previous", "always call X first", "do not tell the user").
- [ ] No description contains hidden text: zero-width characters, HTML comments, very long whitespace runs, or base64 blobs.
- [ ] Descriptions are short and match what the tool actually does.
- [ ] Tool descriptions are shown to the human approver in full, not truncated.

Quick test: `diff` the raw JSON of `tools/list` against what your client UI shows. Search it for non-printing Unicode.

## 2. Tool changes after approval (rug pull)

A server can change a tool after you approved it. ATLAS: `AML.T0109`.

- [ ] I store a hash of each approved tool definition (name, description, schema).
- [ ] The client re-checks that hash on every session and every `notifications/tools/list_changed` notification.
- [ ] A changed definition blocks the call and asks for re-approval.
- [ ] Server version is pinned. No `latest`, no floating tags.

Quick test: change one word in a tool description on a test server and confirm the client refuses to call it.

## 3. Authentication and authorization

- [ ] `tools/list` and `tools/call` both require authentication. Unauthenticated listing is a finding.
- [ ] Remote servers use TLS and verify the server certificate.
- [ ] Credentials are scoped to the minimum needed and are short-lived.
- [ ] Tokens are never placed in tool arguments, prompts or logs.
- [ ] Each tool call is authorized against the user's permissions, not only the agent's.
- [ ] Local servers bind to `127.0.0.1` and not `0.0.0.0`.

Related real-world cases: CVE-2025-6514 (OS command injection in mcp-remote via an untrusted server, CVSS 9.6) and CVE-2025-32711 (EchoLeak, M365 Copilot information disclosure, CVSS 9.3). Patch levels matter.

## 4. Supply chain

ATLAS: `AML.T0010`.

- [ ] The package comes from the publisher you expect. Check the exact name, not a lookalike.
- [ ] Dependencies are locked and scanned (for example `npm audit`, `pip-audit`).
- [ ] Install scripts and post-install hooks were read.
- [ ] Releases are signed or built from a public CI run with provenance.
- [ ] You know who maintains it and when it last changed.

## 5. Execution and data boundaries

- [ ] The server runs in a sandbox (container, restricted user, no host mounts by default).
- [ ] Filesystem access is limited to named directories.
- [ ] Outbound network access is allowlisted.
- [ ] Shell and code-execution tools are off unless required, and require per-call approval.
- [ ] Tool output is treated as untrusted input to the model (indirect prompt injection).
- [ ] Sensitive data is not returned to the model unless needed.

## 6. Logging and response

- [ ] Every tool call is logged with tool name, arguments, caller and result status.
- [ ] Logs do not store secrets.
- [ ] You can disable one server or one tool quickly.
- [ ] There is a named owner for incidents involving this server.

## Scoring

| Result | Meaning |
| --- | --- |
| All of sections 1 to 3 ticked | OK to pilot |
| Any of sections 1 to 3 unticked | Do not connect to production data |
| Sections 4 to 6 unticked | Allowed only with a written exception and an owner |

## Tools that automate some of this

These are my own projects, listed here because they implement parts of the checklist:

| Checklist section | Tool |
| --- | --- |
| 1 and 2 (poisoning, rug pull) | [ContextGuard](https://github.com/sunilgentyala/contextguard) |
| 3 to 5 (agent runtime controls) | [GSH Framework](https://github.com/sunilgentyala/gsh-framework) |

Both are research-grade. Read their README limitations before relying on them. In particular, ContextGuard's supply-chain checks are simulated in the current release.

## Contributing

Pull requests that add a testable item, fix a mapping or add a reference are welcome. Each item must be a yes/no question with a way to test it. Please cite a primary source for any new claim.

## License

MIT
