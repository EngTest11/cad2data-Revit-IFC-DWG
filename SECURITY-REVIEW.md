# Security Review

Reviewed: 2026-10-28

## Findings

### High: Unescaped CAD paths in generated PowerShell

**File:** [`DDC_n8n_Workflows&Pipelines/n8n_3_CAD-BIM-Batch-Converter-Pipeline.json`](DDC_n8n_Workflows%26Pipelines/n8n_3_CAD-BIM-Batch-Converter-Pipeline.json), line 390 (file discovery at line 203)  
**Confidence:** 8/10

The workflow discovers CAD files and interpolates path values into PowerShell source, placing them in single-quoted strings without escaping apostrophes. A crafted filename in the configured input folder could break out of the string and alter or execute commands when the workflow runs, under the n8n process account.

**Recommendation:** Avoid constructing PowerShell source from interpolated paths. Pass paths as arguments or structured input and validate them before use.

### Medium: Qdrant listens on all interfaces without authentication

**File:** [`DDC_LINUX_Converters/ddc-qdrant/root/etc/qdrant/config.yaml`](DDC_LINUX_Converters/ddc-qdrant/root/etc/qdrant/config.yaml), line 9  
**Related file:** `DDC_LINUX_Converters/ddc-qdrant/root/DEBIAN/postinst`, lines 23-24  
**Confidence:** 9/10

The packaged Qdrant service binds to `0.0.0.0` on ports 6333 and 6334 without an API key, and its post-install script enables and starts the service. On a host reachable from other machines, the unauthenticated API may expose database data and write operations.

**Recommendation:** Bind to loopback by default or require API-key authentication, and restrict network access to the service ports.

### Low: API token checked into dashboard configuration

**File:** [`DDC_LINUX_Converters/dashboard/wrangler.toml`](DDC_LINUX_Converters/dashboard/wrangler.toml), line 11  
**Related file:** `DDC_LINUX_Converters/dashboard/functions/api/_middleware.ts`, lines 27-28  
**Confidence:** 9/10

A literal `API_TOKEN` value is present in the checked-in deployment variables and is used by the middleware to authorize dashboard API requests. Unless replaced during deployment, repository readers can use the token to access the analytics API. The exposed data appears limited to aggregate download and geographic statistics.

**Recommendation:** Remove the token from checked-in variables, provision it as a deployment secret, and rotate the current value.

## Scope and limitations

The review covered agent/prompt instructions, workflow prompts and execution nodes, integration code, Linux service and package configuration, deployment scripts, and dependency manifests in this directory. The package-host worker records download metadata, including a daily hashed IP, location/organization fields, and user agent; this appeared to be server-side download analytics rather than covert upload of local model files. Cloud transfers in the Python integration appeared to be explicit options, and semantic search documents its OpenAI use.

This was a content review of the checkout, not a feature-branch diff. Workflows and binaries were not executed; deployed Cloudflare settings were not inspected; no CVE database audit or reverse-engineering of bundled/proprietary binaries was performed.
