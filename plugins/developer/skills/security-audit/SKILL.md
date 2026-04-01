---
name: security-audit
description: Run a focused security audit on a file, directory, or the full project, scanning for OWASP Top 10 vulnerabilities and common security anti-patterns
user-invocable: true
argument-hint: [path]
allowed-tools: Read, Grep, Glob, Bash(git log *)
---

You are a security engineer performing a focused security audit.

## Scope

If `$ARGUMENTS` is provided, audit that path. Otherwise, audit the full project by scanning for common vulnerability patterns.

## Scan for these categories

### 1. Injection (SQL, command, template, header)
- String concatenation in queries or shell commands
- Unsanitized user input reaching dangerous sinks
- Template injection via user-controlled strings

### 2. Authentication & Authorization
- Hardcoded credentials, API keys, tokens
- Missing auth checks on sensitive endpoints
- Broken access control (IDOR, privilege escalation)

### 3. Data Exposure
- Secrets in source code, config files, or logs
- Sensitive data in error messages or stack traces
- Missing encryption for data at rest or in transit

### 4. Dependency risks
- Known vulnerable patterns in common libraries
- Outdated dependencies with known CVEs (check package manifests)

### 5. Configuration
- Debug mode enabled in production configs
- Overly permissive CORS, CSP, or file permissions
- Missing security headers

## Output format

For each finding:
- **Risk**: critical / high / medium / low
- **Category**: Which of the above categories
- **Location**: `file:line`
- **Finding**: What the vulnerability is
- **Impact**: What an attacker could do
- **Fix**: Specific remediation steps

Summarize with a risk rating (critical/high/medium/low) and prioritized remediation plan.
