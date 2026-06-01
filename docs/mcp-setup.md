# MCP Setup — connecting Claude Code to your ABAP system

> **Placeholder.** This document will be filled in with step-by-step instructions
> for configuring the official SAP ABAP MCP Server (`SAPSE.adt-vscode`) against
> a BTP ABAP Environment tenant or an S/4HANA on-prem on-prem system.
>
> Until this is written, refer to the SAP documentation that ships with the
> `SAPSE.adt-vscode` VS Code extension.

## Will be covered

- Prerequisites (VS Code, ABAP extension version, Claude Code)
- Logon configuration for BTP ABAP Environment (service key + Cloud Connector)
- Logon configuration for S/4HANA on-prem on-prem (system landscape, SAP GUI, ADT logon)
- Verifying the MCP connection from Claude Code
- Confirming that Claude can read object source, write source, run ATC, and run ABAP Unit
- Troubleshooting authentication and SSL trust

## Contributing

This page is open for contributions — see `CONTRIBUTING.md`. Verified setup
walkthroughs against real BTP and S/4HANA systems are especially welcome.
