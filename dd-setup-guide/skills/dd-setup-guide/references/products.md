# Datadog Product — GitHub Repository Entry Points

Source repository: https://github.com/DataDog/documentation
All content lives under `content/en/` in the master branch.

## How to use this file

For each product selected in Phase 1, use the **Directory API URL** to browse what subdirectories and
files exist, then use the **Raw Entry Point** to read the top-level `_index.md`. Follow the directory
structure to drill into specific environments, runtimes, or features.

### URL patterns

**Directory listing (returns JSON with file/folder list):**
```
https://api.github.com/repos/DataDog/documentation/contents/content/en/<product>
```

**Raw markdown file:**
```
https://raw.githubusercontent.com/DataDog/documentation/master/content/en/<product>/<path>.md
```

Note: GitHub code search requires authentication — do not use `api.github.com/search/code`. Use
directory listing + targeted raw file fetches instead.

---

## Core Observability

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| APM & Distributed Tracing | `content/en/tracing/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/tracing | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/tracing/_index.md |
| Log Management | `content/en/logs/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/logs | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/logs/_index.md |
| Infrastructure Monitoring | `content/en/infrastructure/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/infrastructure | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/infrastructure/_index.md |
| Metrics | `content/en/metrics/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/metrics | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/metrics/_index.md |

## Frontend & Synthetics

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| Real User Monitoring (RUM) | `content/en/real_user_monitoring/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/real_user_monitoring | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/real_user_monitoring/_index.md |
| Session Replay | `content/en/real_user_monitoring/session_replay/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/real_user_monitoring/session_replay | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/real_user_monitoring/session_replay/_index.md |
| Synthetics | `content/en/synthetics/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/synthetics | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/synthetics/_index.md |

## Database & Network

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| Database Monitoring | `content/en/database_monitoring/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/database_monitoring | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/database_monitoring/_index.md |
| Network Performance Monitoring | `content/en/network_monitoring/performance/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/network_monitoring/performance | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/network_monitoring/performance/_index.md |
| Network Device Monitoring | `content/en/network_monitoring/devices/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/network_monitoring/devices | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/network_monitoring/devices/_index.md |

## Security

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| Cloud Security Management | `content/en/security/cloud_security_management/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/security/cloud_security_management | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/security/cloud_security_management/_index.md |
| Application Security Management | `content/en/security/application_security/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/security/application_security | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/security/application_security/_index.md |
| Cloud SIEM | `content/en/security/cloud_siem/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/security/cloud_siem | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/security/cloud_siem/_index.md |

## CI/CD & Testing

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| CI Visibility | `content/en/continuous_integration/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/continuous_integration | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/continuous_integration/_index.md |
| Test Optimization | `content/en/tests/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/tests | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/tests/_index.md |

## Serverless & Containers

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| Serverless Monitoring | `content/en/serverless/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/serverless | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/serverless/_index.md |
| Container Monitoring | `content/en/containers/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/containers | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/containers/_index.md |

## Agent & Platform

| Product | GitHub Path | Directory API | Raw Entry Point |
|---------|-------------|---------------|-----------------|
| Datadog Agent | `content/en/agent/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/agent | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/agent/_index.md |
| OpenTelemetry | `content/en/opentelemetry/` | https://api.github.com/repos/DataDog/documentation/contents/content/en/opentelemetry | https://raw.githubusercontent.com/DataDog/documentation/master/content/en/opentelemetry/_index.md |

---

## Known subdirectory patterns

These subdirectories appear consistently across products and are worth checking when building a guide:

| Subdirectory | What's in it |
|---|---|
| `_index.md` | Product overview, links to major sections |
| `trace_collection/` or `log_collection/` | Install and ingestion guides |
| `configuration/` | Agent and product config references |
| `compatibility/` | Supported versions, runtimes, OS |
| `guide/` | How-to articles, often environment-specific |
| `troubleshooting/` | Common issues and fixes |

### APM-specific paths worth knowing

| Path | Content |
|---|---|
| `content/en/tracing/trace_collection/dd_libraries/` | Per-language tracer install guides |
| `content/en/tracing/trace_collection/automatic_instrumentation/` | Auto-instrumentation by env |
| `content/en/tracing/trace_collection/compatibility/` | Per-language compatibility matrices |
| `content/en/tracing/trace_collection/library_config/` | Per-language tracer config reference |
| `content/en/tracing/trace_collection/single-step-apm/` | Single-step APM instrumentation |
