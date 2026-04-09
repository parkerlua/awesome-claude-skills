---
name: security-audit
description: Audit code for security vulnerabilities. Use when asked to review code for security issues, check for vulnerabilities, or harden an application.
---

You are doing a security audit. Think like an attacker — assume every input is malicious, every dependency is compromised, every secret is already leaked.

## Audit Order

Go through code in this order of severity:

1. **Injection** — SQL, command, HTML, template injection
2. **Authentication & Authorization** — who can do what, and is it enforced?
3. **Secrets & Credentials** — anything sensitive in the code?
4. **Dependencies** — outdated packages with known CVEs
5. **Data Exposure** — what's being logged, returned, or stored that shouldn't be?
6. **Cryptography** — weak algorithms, bad key management
7. **Input Validation** — is user input trusted anywhere it shouldn't be?

## Injection

### SQL Injection
```typescript
// 🚨 CRITICAL: raw string interpolation
const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`)

// ✅ Parameterized query
const user = await db.query('SELECT * FROM users WHERE email = $1', [email])
```

Look for: string template literals in queries, `.format()`, string concatenation into SQL.

### Command Injection
```typescript
// 🚨 CRITICAL: user input in shell commands
exec(`convert ${filename} output.png`)

// ✅ Use arrays, never shell strings with user input
execFile('convert', [filename, 'output.png'])
```

Look for: `exec`, `spawn`, `system`, `eval` with any user-controlled data.

### XSS
```typescript
// 🚨 Raw HTML injection
element.innerHTML = userContent

// ✅ Text content only
element.textContent = userContent
// Or sanitize: DOMPurify.sanitize(userContent)
```

## Authentication & Authorization

**Check every endpoint:**
- Is auth required? Is it actually enforced?
- Can user A access user B's data by changing an ID?
- Are admin routes protected beyond just checking a role in the frontend?

```typescript
// 🚨 Authorization only checked on frontend
// Anyone can call /api/admin/users directly

// ✅ Enforced on every API route
router.get('/admin/users', requireAuth, requireRole('admin'), handler)
```

**Common auth bugs:**
- JWT `alg: none` accepted
- Token not invalidated on logout
- Password reset tokens don't expire
- Account enumeration via different error messages for "wrong email" vs "wrong password"
- Missing rate limiting on login endpoint

## Secrets & Credentials

Scan for these patterns:
```
API keys: sk-..., pk-..., AKIA..., ghp_...
Passwords: password=, passwd=, pwd=
Tokens: token=, secret=, bearer
Private keys: -----BEGIN RSA PRIVATE KEY-----
Connection strings: mongodb://, postgres://, mysql://
```

```bash
# Quick scan
grep -rn "password\s*=\s*['\"]" --include="*.ts" --include="*.js" .
grep -rn "secret\s*=\s*['\"]" --include="*.ts" --include="*.js" .
```

**Never acceptable:**
- Secrets in source code (use env vars)
- Secrets in `.env` committed to git (use `.env.example`)
- Secrets in logs
- Secrets in error messages returned to clients

## Dependencies

```bash
# Check for known vulnerabilities
npm audit
pip-audit
bundle audit

# Check for outdated packages
npm outdated
```

Flag any package with a HIGH or CRITICAL CVE. Note the CVE number and whether a patched version exists.

## Data Exposure

**Check API responses — are you returning more than needed?**
```typescript
// 🚨 Returns password hash, internal IDs, all fields
return await db.query('SELECT * FROM users WHERE id = $1', [id])

// ✅ Explicit field selection
return await db.query(
  'SELECT id, name, email, created_at FROM users WHERE id = $1', [id]
)
```

**Check logs — are you logging sensitive data?**
```typescript
// 🚨 Logs password and token
console.log('Login attempt:', { email, password, token })

// ✅ Log only what's safe
console.log('Login attempt:', { email, userId })
```

## Cryptography

```typescript
// 🚨 Weak/broken algorithms
crypto.createHash('md5')
crypto.createHash('sha1')
createCipheriv('des', ...)

// ✅ Strong algorithms
crypto.createHash('sha256')
crypto.createHash('sha512')
createCipheriv('aes-256-gcm', ...)

// 🚨 Rolling your own crypto
// ✅ Use established libraries: bcrypt for passwords, libsodium for encryption
```

**Passwords must use bcrypt, scrypt, or argon2 — never plain SHA or MD5.**

## Output Format

```
## Security Audit Summary

**Risk Level:** CRITICAL / HIGH / MEDIUM / LOW

### Critical Issues (fix before deploy)
[CRITICAL] ...

### High Issues (fix this sprint)
[HIGH] ...

### Medium Issues (schedule soon)
[MEDIUM] ...

### Low / Informational
[LOW] ...

### What Looks Good
...
```

Always include what looks good — it builds trust and shows the audit was thorough.
