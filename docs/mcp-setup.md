# MCP Setup — connecting Claude Code to your ABAP system

This guide covers the shortest path from a fresh Claude Code install plus the
official `SAPSE.adt-vscode` extension to a working ABAP MCP connection.

This repo assumes the MCP server is provided by the VS Code extension. Nothing
in `claude-abap-skills` starts or implements the server itself.

## What you will finish with

At the end of this setup:

- `claude mcp list` shows `abap-adt` as connected
- `mcp__abap-adt__abap_list_destinations` returns at least one configured
  destination
- `mcp__abap-adt__abap_generators-list_generators` returns the three RAP
  generators

If you want to use the RAP generator skill, launch Claude Code with a longer
timeout:

```bash
MCP_TIMEOUT=600000 claude
```

## Prerequisites

Install and confirm these first:

1. `Claude Code`
2. Visual Studio Code
3. The official SAP extension:
   [`SAPSE.adt-vscode`](https://marketplace.visualstudio.com/items?itemName=SAPSE.adt-vscode)
4. Access to one supported target:
   - SAP BTP ABAP Environment
   - SAP S/4HANA on-prem, developed in the ABAP Cloud development model

The extension supports different connection types depending on the landscape.
For BTP ABAP Environment you normally connect over HTTPS. For on-prem or private
landscapes, your team may instead expose the system through RFC or a private
network path managed by Basis.

## Before you register Claude Code

In VS Code, confirm the extension can already reach your ABAP system through
ADT. The MCP server sits on top of that working ADT configuration.

Use this order:

1. Create or import the ADT destination in the extension
2. Log on successfully
3. Only then enable the MCP server and register it with Claude Code

If ADT logon is not working yet, stop and fix that first. Claude Code will not
be able to compensate for a broken ADT destination.

## Path A: SAP BTP ABAP Environment

### 1. Collect the system details

From your BTP subaccount and ABAP environment, collect:

- The ABAP service endpoint / host used for ADT
- The user you will use from ADT
- The password or other credential required by your landscape
- A service key, if your team uses one to discover the endpoint and landscape
  details

For a trial system this is usually direct internet access over HTTPS. In a
production landscape, private connectivity or corporate routing rules may still
apply.

### 2. Cloud Connector is not part of this path

Cloud Connector is **not** required for workstation-to-BTP ADT access. SAP
Cloud Connector is an on-prem agent that opens a tunnel from your on-prem
network to a BTP subaccount, so BTP-hosted applications can reach on-prem
systems. The direction is BTP → on-prem, not workstation → BTP.

Your VS Code / Claude Code workstation reaches BTP ABAP Environment directly
over HTTPS using the ADT endpoint from the service key. Cloud Connector only
becomes relevant if your BTP ABAP Environment itself needs to call back into
on-prem systems — which is out of scope for this setup guide.

### 3. Create the ADT logon in VS Code

In VS Code:

1. Open the SAP ABAP extension UI
2. Create a new system / destination
3. Enter the BTP ABAP Environment endpoint details
4. Save the destination
5. Log on once and confirm the connection succeeds

Use the same endpoint and credentials your team already uses for ADT. Do not
invent a second set of connection details just for Claude Code.

### 4. Enable the MCP server in the extension

Open VS Code settings and enable:

```text
adt.mcpServer.enabled = true
```

After enabling it, retrieve the bearer token exposed by:

```text
adt.mcpServer.token
```

Treat that token like a local secret. Anyone with the token can talk to the MCP
server running on your machine.

### 5. Register the MCP server in Claude Code

Run this in a terminal:

```bash
claude mcp add --transport http abap-adt http://localhost:2236/mcp --header "Authorization: Bearer <TOKEN>"
```

Replace `<TOKEN>` with the value from `adt.mcpServer.token`.

If you rotate the token later, remove and re-add the MCP entry or update the
stored header so Claude Code stops using the old bearer token.

## Path B: S/4HANA on-prem in the ABAP Cloud development model

### 1. Confirm the system is in scope

This repo only supports:

- SAP S/4HANA on-prem releases that support the ABAP Cloud development model
- Development performed in the ABAP Cloud development model

Do not use this setup guide for ECC or classic non-Cloud ABAP development.

### 2. Confirm the workstation prerequisites

For on-prem access you usually need:

- Network access to the ABAP system from your workstation
- Any required corporate VPN or private route
- SAP GUI or other landscape tooling already used by your team for SSO or trust
- SNC configured on your workstation — for now this setup requires it
- A working ADT user with the required authorizations

The exact transport path is Basis-dependent. The important requirement for this
guide is simple: ADT in VS Code must already work before you involve Claude
Code.

### 3. Create and test the ADT logon in VS Code

In VS Code:

1. Create the S/4HANA destination
2. Enter the host, port, and client details provided by your team
3. Log on with the same user you use for ADT development
4. Verify the connection succeeds

If your landscape uses certificates, reverse proxies, or corporate SSL
inspection, finish that trust setup here first.

### 4. Enable the MCP server and capture the token

Enable this setting:

```text
adt.mcpServer.enabled = true
```

Then copy the value from:

```text
adt.mcpServer.token
```

### 5. Register the MCP server in Claude Code

Run:

```bash
claude mcp add --transport http abap-adt http://localhost:2236/mcp --header "Authorization: Bearer <TOKEN>"
```

This registration step is the same for BTP and S/4HANA because Claude Code is
connecting to the local MCP server exposed by VS Code, not directly to your
backend ABAP host.

## Verify the connection

Run these checks in order.

### 1. Check Claude Code sees the MCP

```bash
claude mcp list
```

You want to see `abap-adt` with a connected status.

### 2. Check the destination list

In Claude Code, call:

```text
mcp__abap-adt__abap_list_destinations
```

Expected result: the destination you configured in VS Code appears in the
response.

### 3. Check the RAP generators are exposed

In Claude Code, call:

```text
mcp__abap-adt__abap_generators-list_generators
```

Expected result: three generators are listed:

- `x-ui-service`
- `ui-service`
- `webapi-service`

If those checks succeed, the MCP path is working end-to-end.

## Troubleshooting

### Authentication failures

Symptoms:

- `claude mcp list` shows the server but not as connected
- MCP calls fail with unauthorized or forbidden responses

Checks:

- Confirm `adt.mcpServer.enabled` is still `true`
- Copy a fresh value from `adt.mcpServer.token`
- Re-run the `claude mcp add ... --header "Authorization: Bearer <TOKEN>"`
  command with the new token
- Make sure ADT logon itself still works in VS Code

Common cause: the local bearer token was rotated but Claude Code still has the
old one stored.

### SSL trust issues

Symptoms:

- ADT logon in VS Code fails before Claude Code is involved
- Certificates are rejected for on-prem or proxied systems

Checks:

- Fix certificate trust in the workstation / corporate environment first
- Re-test ADT in VS Code before re-testing Claude Code

Claude Code depends on the local VS Code extension path. If ADT cannot establish
trust, the MCP layer will not recover on its own.

### Port conflicts on `2236`

Symptoms:

- The MCP server does not start
- Claude Code cannot reach `http://localhost:2236/mcp`

Checks:

- Close any stale local process already bound to port `2236`
- Restart VS Code after freeing the port
- Re-check `claude mcp list`

### Long-running generator calls time out

For RAP generator workflows, start Claude Code with a longer MCP timeout:

```bash
MCP_TIMEOUT=600000 claude
```

Without this, the local MCP server can be healthy while Claude Code still gives
up too early on valid long-running generator operations.

## Recommended first smoke test

After the connection works, use a minimal sequence:

1. `claude mcp list`
2. `mcp__abap-adt__abap_list_destinations`
3. `mcp__abap-adt__abap_generators-list_generators`

Do not start with a large RAP generation flow. Prove the transport, token, and
destination wiring first.

## What this guide intentionally does not cover

This page stays narrow on purpose. It does not attempt to document:

- General VS Code usage
- Full ADT feature coverage
- ABAP development outside the supported scope of this repo
- Classic ABAP or ECC setup

Once the MCP connection works, return to the repo README and install the plugin
or plugins you need.
