# Codex AIDP Workbench Plugin

Codex plugin packaging for Oracle AI Data Platform Workbench skills.

This plugin helps Codex operate Oracle AI Data Platform Workbench from natural
language. It gives Codex task-specific skills for common AIDP work: listing
workspaces and clusters, creating or checking compute clusters, exploring
catalogs and schemas, running Spark SQL, managing Delta Lake tables, working
with ingestion and pipelines, and handling governance or MLOps workflows.

The plugin provides Codex instructions and helper scripts. It does not include
Oracle credentials or the Oracle AIDP CLI itself. Each user connects it to their
own Oracle tenancy and AIDP instance.

## Attribution

This repository packages Oracle AIDP Workbench skill content from Oracle sample
materials for Codex usage. Original content is copyright Oracle and/or its
affiliates and is distributed with the included license and notice files.

This is not an official Oracle-published Codex plugin. It is a Codex packaging
adaptation.

## What Users Must Install

This plugin is not a standalone Oracle client. Each user must have local Oracle
tooling and access to their own AIDP environment.

Required:

- Codex with local plugin support.
- Python 3.9 or later. Download from
  <https://www.python.org/downloads/>. On Windows, enable **Add python.exe to
  PATH** during installation.
- `pip`, usually available as `python -m pip` after Python is installed.
- A working terminal where `python --version` and `python -m pip --version`
  succeed.
- Oracle `aidp` CLI installed and available on `PATH`. Download the AIDP CLI
  release assets from
  <https://github.com/oracle-samples/aidataplatform-sdk/releases/tag/v1.0.2>.
- OCI auth configured in `~/.oci/config`. For OCI config setup, see Oracle's
  SDK/CLI config docs:
  <https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm>.
- Permission to access the target Oracle AIDP instance and workspace.
- The user's own AIDP instance OCID, region, and workspace key.

For Spark SQL and notebook execution through `scripts/aidp_sql.py`, install the
Python helper dependencies:

```bash
python -m pip install -r scripts/requirements.txt
```

## Install The AIDP CLI

First verify Python and pip:

```bash
python --version
python -m pip --version
```

If either command fails, install Python 3.9+ first and make sure Python is on
`PATH`.

Download the Python CLI release assets from the Oracle AIDP SDK/CLI release:

https://github.com/oracle-samples/aidataplatform-sdk/releases/tag/v1.0.2

For Python CLI usage, download these two assets:

- `aidp-python-client-1.0.2.zip`
- `aidp-py-cli-1.0.2.zip`

Unpack both, then install both wheels:

```bash
python -m pip install --user \
  ./aidp-python-client/aidp_python_client-1.0.2-py3-none-any.whl \
  ./aidp-py-cli/aidp_cli-1.0.2-py3-none-any.whl
```

Verify:

```bash
aidp --help
```

If `aidp` is installed but not found, add your Python user scripts directory to
`PATH`. On Windows this is often similar to:

```text
C:\Users\<you>\AppData\Roaming\Python\Python314\Scripts
```

Windows PowerShell example:

```powershell
$scripts = "$env:APPDATA\Python\Python314\Scripts"
[Environment]::SetEnvironmentVariable(
  "Path",
  [Environment]::GetEnvironmentVariable("Path", "User") + ";$scripts",
  "User"
)
```

Close and reopen the terminal, then verify:

```bash
aidp --help
```

Git Bash temporary PATH example for the current terminal session:

```bash
export PATH="$HOME/AppData/Roaming/Python/Python314/Scripts:$PATH"
aidp --help
```

## Configure OCI Auth

Create or verify `~/.oci/config`.

API key profile example:

```ini
[DEFAULT]
user=ocid1.user.oc1..example
fingerprint=aa:bb:cc:dd:ee:ff
tenancy=ocid1.tenancy.oc1..example
region=us-phoenix-1
key_file=C:/Users/<you>/.oci/oci_api_key.pem
```

Do not quote `key_file`, and do not put inline comments after the path.

Alternatively, use session-token auth:

```bash
oci session authenticate --profile-name DEFAULT --region <region>
```

## Verify AIDP Access Before Using Codex

Run this outside Codex first:

```bash
aidp workspace list \
  --instance-id <aidp-instance-ocid> \
  --auth api_key \
  --profile DEFAULT \
  --region <region>
```

For a session-token profile, use:

```bash
aidp workspace list \
  --instance-id <aidp-instance-ocid> \
  --auth security_token \
  --profile DEFAULT \
  --region <region>
```

This should return `status: 200` and show at least one workspace.

## Enable This Plugin In Codex

Clone this repository:

```bash
git clone https://github.com/mariam-abdelgalil/codex-aidp-workbench-plugin.git
```

Install or enable the local plugin in Codex using the plugin installation
mechanism available in your Codex build. Common local-plugin flows look like:

```bash
codex plugin install /path/to/codex-aidp-workbench-plugin
```

or adding the folder through the Codex app/plugin UI.

After enabling the plugin, start a new Codex session so the skills are loaded.

Codex loads the skill files from `skills/`. When a user asks an AIDP task,
Codex reads the relevant `SKILL.md` file and uses the local `aidp` CLI or the
documented OCI REST fallback to perform the operation.

## Local Workspace Instructions

Do not commit real OCIDs or private key paths to this repository. Put local
environment details in your own private `AGENTS.md`, `.env`, or Codex workspace
instructions.

The plugin expects users to provide:

```text
AIDP instance OCID: <your-aidp-instance-ocid>
Region: <your-region>
Workspace key: <your-workspace-key>
Auth profile: DEFAULT or another OCI profile name
```

## Example Prompts

Once the plugin is enabled and the AIDP CLI works locally, ask Codex:

```text
List my AIDP workspaces and clusters.
```

```text
Create an AIDP Spark cluster named demo_cluster with defaults.
```

```text
Explore my AIDP catalog schemas and tables.
```

## How It Works

The skills use two execution paths:

- Control-plane operations prefer the official `aidp` CLI.
- Skills document `oci raw-request` fallbacks for operations where needed.
- Interactive Spark SQL and notebook cells use `scripts/aidp_sql.py`.

Relevant references:

- `references/aidp-cli-map.md`
- `references/oci-raw-request.md`
- `references/no-mcp-rest-map.md`
- `references/rest-endpoint-map.md`

## What Not To Commit

Never commit:

- `~/.oci/config`
- private keys such as `*.pem`
- real AIDP instance OCIDs
- real compartment OCIDs
- real workspace keys
- generated cluster payloads containing environment-specific values
- `.aidp/` grounding caches from private environments

## License

See `LICENSE` and `NOTICE`.
