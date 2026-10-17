# Privacy Policy

Last updated: 18 October 2026

This policy covers the Production Readiness Auditor skill and plugin ("the skill"), published by Ali Parandeh.

## What the skill collects

Nothing. The skill is a set of instructions and reference files. It has no server, no analytics, no telemetry and no tracking, and the author receives no data of any kind when you install or use it.

## How your code is handled

The skill runs entirely inside your own Claude session. Claude reads files in the repository you point it at, using the file tools your Claude client already provides, and writes the audit report (`PRODUCTION_READINESS_AUDIT-{timestamp}.md`) to that repository's root. Your code and the report stay on your machine or in your Claude workspace, and are handled under the terms and privacy policy of the Claude product you are using. The skill does not send them anywhere else.

## Optional O'Reilly MCP

If you choose to connect the O'Reilly MCP yourself, the skill uses it to look up book content and reading recommendations. In that case it sends short topic queries to O'Reilly, for example "Docker non-root container security". The skill is instructed never to include source code, file contents, file names, configuration, secrets, logs, or anything that identifies your repository, company or product in those queries.

Data sent to O'Reilly is handled under O'Reilly Media's privacy policy: https://www.oreilly.com/privacy.html

If you don't connect the O'Reilly MCP, the skill uses its bundled reference files and makes no outside requests.

## Retention

The skill stores nothing, so there is nothing to retain or delete. Audit reports written to your repository are yours to keep or remove.

## Changes

Changes to this policy are published in this file, and the history is visible in the repository's commit log.

## Contact

Questions about this policy: open an issue at https://github.com/Ali-Parandeh/production-readiness-auditor/issues
