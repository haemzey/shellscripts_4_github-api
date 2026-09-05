# GitHub Repository Access Audit

A production-oriented Bash automation script for auditing GitHub repository collaborators and their access permissions through the **GitHub REST API**.

The project demonstrates practical DevOps shell scripting techniques including API integration, authentication, JSON processing with `jq`, error handling, input validation, environment-based secrets, and repository access auditing.

---

## Features

- Authenticate with the GitHub REST API using a Personal Access Token
- Retrieve repository collaborators
- Display collaborator usernames and roles
- Identify read-level access
- Identify write-level access
- Detect maintain/admin permissions
- Validate required dependencies
- Validate required environment variables
- Handle API failures and HTTP status codes
- Use strict Bash execution mode
- Generate clean, human-readable audit output
- Designed to be CI/CD friendly

---

## Architecture

```text
                GitHub Repository
                       │
                       │ REST API
                       ▼
              github-collaborators.sh
                       │
             ┌─────────┴─────────┐
             │                   │
           curl                 jq
             │                   │
             ▼                   ▼
       GitHub API            JSON Parser
             │                   │
             └─────────┬─────────┘
                       ▼
                Access Report
```

---

## Requirements

The following tools must be installed:

- Bash
- `curl`
- `jq`
- `mktemp`

Check the dependencies:

```bash
bash --version
curl --version
jq --version
```

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install -y curl jq
```

---

## GitHub Authentication

The script uses environment variables for authentication instead of storing credentials inside the source code.

Set your GitHub username:

```bash
export GITHUB_USER="HamzehMughal"
```

Set your GitHub Personal Access Token:

```bash
export GITHUB_TOKEN="YOUR_GITHUB_TOKEN"
```

### Security Recommendation

Never commit the token to Git.

Do **not** do this:

```bash
GITHUB_TOKEN="ghp_xxxxxxxxx"
```

inside the script or repository.

Prefer:

```bash
export GITHUB_TOKEN="..."
```

or inject the secret through a CI/CD secret-management system.

---

## Usage

Make the script executable:

```bash
chmod +x github-collaborators.sh
```

Run it:

```bash
./github-collaborators.sh <repository-owner> <repository-name>
```

Example:

```bash
./github-collaborators.sh haemzey aws_resource_tracker
```

Or provide credentials only for the command:

```bash
GITHUB_USER="HamzehMughal" \
GITHUB_TOKEN="$TOKEN" \
./github-collaborators.sh haemzey aws_resource_tracker
```

---

## Example Output

```text
[INFO] Starting GitHub repository access audit.
[INFO] Fetching repository collaborators...
[INFO] Repository: haemzey/aws_resource_tracker

USERNAME                  ROLE         ACCESS
------------------------- ------------ ------------
developer01               direct       write
developer02               direct       read
devops-admin              admin        admin

[INFO] Total collaborators: 3

Users with read-level access:
--------------------------------
developer02

Users with write-level access:
--------------------------------
developer01

[INFO] GitHub repository access audit completed.
```

---

## GitHub Permissions

The script evaluates the permissions returned by GitHub's collaborator API.

| GitHub Access | API Permission | Description |
|---|---|---|
| Read | `pull` | Can view and clone the repository |
| Write | `push` | Can push changes |
| Triage | `triage` | Can manage issues and pull requests |
| Maintain | `maintain` | Repository maintenance permissions |
| Admin | `admin` | Full repository administration |

---

## Script Structure

The script is organized into separate functions to keep API communication, validation, and presentation logic isolated.

```text
github-collaborators.sh
│
├── Configuration
│
├── Input Validation
│
├── Dependency Validation
│
├── GitHub API Request
│
├── Collaborator Retrieval
│
├── Collaborator Display
│
├── Read Access Filtering
│
├── Write Access Filtering
│
└── Main Execution
```

---

## Error Handling

The script uses strict Bash execution:

```bash
set -Eeuo pipefail
```

This provides:

- `-e` — exit when a command fails
- `-u` — detect unset variables
- `-o pipefail` — detect failures inside pipelines
- `-E` — preserve the `ERR` trap inside functions

API requests also validate the HTTP status code.

For example:

```text
HTTP 200
    ↓
Process response

HTTP 401
    ↓
Authentication failure

HTTP 403
    ↓
Permission/rate-limit issue

HTTP 404
    ↓
Repository/resource not found
```

---

## API Endpoint

The script communicates with the GitHub REST API:

```text
GET /repos/{owner}/{repo}/collaborators
```

The API response contains collaborator information including:

```json
{
  "login": "developer01",
  "permissions": {
    "pull": true,
    "push": true,
    "admin": false
  }
}
```

The script uses `jq` to extract the relevant fields.

---

## Why `jq`?

GitHub returns structured JSON rather than plain text.

Instead of attempting to parse JSON using `grep`, `sed`, or `awk`, the script uses `jq`:

```bash
jq -r '
    .[] |
    [
        .login,
        .role_name,
        (
            if .permissions.admin then "admin"
            elif .permissions.maintain then "maintain"
            elif .permissions.push then "write"
            elif .permissions.triage then "triage"
            elif .permissions.pull then "read"
            else "unknown"
            end
        )
    ] |
    @tsv
'
```

This makes the JSON processing predictable and significantly safer than treating JSON as plain text.

---

## Security Considerations

### Credentials

Credentials are supplied through environment variables:

```bash
GITHUB_TOKEN
GITHUB_USER
```

They are not stored in the script.

### Token Permissions

Use the minimum GitHub token permissions required for the operation.

For read-only auditing, avoid granting unnecessary administrative privileges.

### Git History

Never commit:

```text
.env
tokens
PATs
credentials
private keys
```

Recommended `.gitignore`:

```gitignore
.env
*.secret
*.token
credentials/
```

---

## CI/CD Usage

The script can be integrated into GitHub Actions or another CI/CD platform.

Example:

```yaml
- name: Audit repository collaborators
  env:
    GITHUB_USER: ${{ secrets.GITHUB_USER }}
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    chmod +x github-collaborators.sh
    ./github-collaborators.sh haemzey aws_resource_tracker
```

For production pipelines, store credentials in the CI/CD platform's secret-management system rather than exposing them in workflow files.

---

## Operational Use Cases

This automation can be used for:

### Repository Access Auditing

```text
Repository
    ↓
Get collaborators
    ↓
Evaluate permissions
    ↓
Generate access report
```

### Security Reviews

Identify users with:

```text
Admin
Write
Maintain
```

permissions.

### Offboarding

Before removing an employee's repository access:

```text
User
 ↓
Search repositories
 ↓
Check permissions
 ↓
Remove unnecessary access
```

### Compliance

Regularly execute the script through a scheduled CI/CD job and retain the generated audit output.

---

## Future Improvements

Possible extensions for this project:

- Implement complete API pagination
- Add JSON output mode
- Add CSV reporting
- Add organization-wide repository auditing
- Add collaborator removal automation
- Add collaborator invitation automation
- Add branch protection auditing
- Add repository security-settings auditing
- Add GitHub Actions integration
- Add exit codes based on security policy violations
- Add automated reporting to Slack/Teams
- Add scheduled access audits
- Add rate-limit monitoring

---

## Example Project Expansion

The repository can eventually become a GitHub administration toolkit:

```text
github-devops-toolkit/
│
├── README.md
│
├── scripts/
│   ├── list-collaborators.sh
│   ├── list-read-users.sh
│   ├── list-write-users.sh
│   ├── add-collaborator.sh
│   ├── remove-collaborator.sh
│   ├── list-repositories.sh
│   ├── list-branches.sh
│   ├── branch-protection.sh
│   └── repository-audit.sh
│
├── lib/
│   └── github-api.sh
│
└── .github/
    └── workflows/
        └── security-audit.yml
```

This structure allows common GitHub API functionality to be reused instead of duplicating authentication and error-handling logic across every script.

---

## DevOps Skills Demonstrated

This project demonstrates practical knowledge of:

- Bash scripting
- Linux shell automation
- REST APIs
- GitHub REST API
- `curl`
- `jq`
- JSON processing
- HTTP status handling
- Authentication
- Environment variables
- Secret management
- Input validation
- Error handling
- Logging
- CI/CD integration
- Repository security auditing
- Automation design
