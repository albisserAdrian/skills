---
name: security
description: Security playbook organised as three tiers (always-do / ask-first / never-do) plus OWASP-shaped issue patterns and dependency triage. Use when building anything that handles user input, auth, sessions, sensitive data, file uploads, webhooks, or external integrations, or when reviewing security-sensitive code before merge.
---

# Security

**External input is hostile until proven otherwise. Secrets stay out of source. Every protected path checks authorization, not just authentication.**

Security is a constraint on every line that touches user data, auth, or external systems, not a phase you add later. Retrofitting is roughly 10× the cost of building it in.

When working in an unfamiliar area of the codebase, check the project's domain glossary for what counts as sensitive (PII, payment, internal-only) and respect ADRs that scope the threat model.

## Three tiers

### Always do

- **Validate external input at the system boundary** (route handlers, form handlers, webhook endpoints, message consumers).
- **Parameterise database queries.** Never concatenate user input into SQL / NoSQL filters / shell commands.
- **Encode output** to prevent XSS. Use the framework's auto-escaping; don't bypass it.
- **HTTPS everywhere** for external communication.
- **Hash passwords with bcrypt / scrypt / argon2.** Salt rounds ≥ 12 for bcrypt. Never store plaintext.
- **Set security headers**: CSP, HSTS, X-Frame-Options, X-Content-Type-Options.
- **Session cookies are `httpOnly`, `secure`, `sameSite`.** Never put auth tokens in localStorage.
- **Run a vulnerability scan** (`npm audit` / equivalent) before every release.

### Ask first

These touch security-critical surface area and need an explicit human call before you ship:

- New authentication flows or changes to auth logic.
- Storing a new category of sensitive data (PII, payment info, health data).
- New external service integrations.
- CORS configuration changes.
- File upload handlers.
- Rate-limit or throttle changes.
- Granting elevated permissions or roles.

### Never do

- **No secrets in version control.** API keys, passwords, tokens: environment only.
- **No sensitive data in logs.** No passwords, tokens, full card numbers, full SSNs.
- **No client-side validation as a security boundary.** It's UX, not enforcement.
- **No disabling security headers for convenience.** Fix the root cause instead.
- **No `eval` / `innerHTML` / `dangerouslySetInnerHTML`** with user-provided data.
- **No stack traces or internal error details exposed to users.** Log them server-side, return a generic message.

## OWASP-shaped issues

The named issues to actively guard against, with the canonical pattern for each.

### Injection (SQL, NoSQL, OS command, LDAP)

If you find yourself building a query string with `+` or template literals, stop.

```ts
// BAD: SQL injection via string concatenation
const q = `SELECT * FROM users WHERE id = '${userId}'`;

// GOOD: parameterised query
const u = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// GOOD: ORM with parameterised input
const u = await prisma.user.findUnique({ where: { id: userId } });
```

### Broken authentication

Strong password hashing, short-lived session tokens, expiring reset tokens, rate-limited login.

```ts
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const passwordHash = await hash(plaintext, SALT_ROUNDS);
const ok = await compare(attempt, passwordHash);

app.use(session({
  secret: process.env.SESSION_SECRET,  // env, not source
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,                    // not JS-accessible
    secure: true,                      // HTTPS only
    sameSite: 'lax',                   // CSRF protection
    maxAge: 24 * 60 * 60 * 1000,
  },
}));
```

### Cross-site scripting (XSS)

Let the framework auto-escape. If you must render user HTML, run it through a vetted sanitiser and never trust the result for sensitive contexts.

```ts
// BAD
element.innerHTML = userInput;

// GOOD: framework auto-escapes
return <div>{userInput}</div>;

// IF YOU MUST: sanitise first
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### Broken access control

Authentication answers "who is this". Authorization answers "is this person allowed to touch *this specific resource*". The most common access-control bug is checking the first and forgetting the second. Check ownership at fetch time, not just at action time.

```ts
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // Authorization: does the authenticated user own this resource?
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Not authorized' },
    });
  }

  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### Security misconfiguration

Restrictive CSP, locked-down CORS to known origins (no `*` with credentials), no debug mode in production.

```ts
import helmet from 'helmet';

app.use(helmet());

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],   // tighten if possible
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### Sensitive data exposure

Strip secret/internal fields before serialising responses. Don't return the password hash, reset token, or internal IDs the client doesn't need.

```ts
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');
```

### Server-side request forgery (SSRF)

Validate any URL the server fetches against an allowlist. Reject internal IPs and cloud metadata endpoints (`169.254.169.254`, `localhost`, RFC1918 ranges).

### Insecure deserialisation

Don't deserialise untrusted data into typed objects without schema validation first. The validation step *is* the protection.

## Validate at the boundary

External data (request bodies, query params, webhook payloads, file uploads, config from disk, third-party API responses) gets validated **once** at the boundary, against an explicit schema. Past that boundary, the data is trusted and typed. This is the single highest-leverage habit on the list.

```ts
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid input',
        details: result.error.flatten(),
      },
    });
  }
  // result.data is typed and validated, trusted past this point
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### File upload safety

Cap size, allow-list mime types, and check magic bytes (not the extension) when the file content is security-critical. Store uploads outside the web root or behind a separate domain.

```ts
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('File type not allowed');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('File too large (max 5MB)');
  }
  // Don't trust the extension. Check magic bytes for security-critical uploads
}
```

## Rate limiting

Stricter limits on auth endpoints than general API. Headers go on so clients can back off.

```ts
import rateLimit from 'express-rate-limit';

app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000,   // 15 min
  max: 100,                    // per IP per window
  standardHeaders: true,
  legacyHeaders: false,
}));

app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,                     // 10 attempts / 15 min
}));
```

## Secrets management

```
.env.example   → committed, placeholder values only
.env           → NOT committed, real local secrets
.env.local     → NOT committed, local overrides
```

`.gitignore` must cover `.env`, `.env.local`, `.env.*.local`, `*.pem`, `*.key`.

Quick pre-commit check for accidentally staged secrets:

```bash
git diff --cached | grep -iE 'password|secret|api[_-]?key|token|bearer'
```

If a secret has ever been committed, **rotate it.** Removing it from history doesn't undo the leak. Assume it's compromised.

## Triaging vulnerability-scan results

Not every finding is a fire. Use this decision tree to keep the bar honest:

```
Scan reports a vulnerability
├── Severity: critical / high
│   ├── Is the vulnerable code reachable in your app?
│   │   ├── YES → fix immediately (update, patch, or replace)
│   │   └── NO  (dev-only dep, dead code path) → fix soon, not a blocker
│   └── Fix available?
│       ├── YES → update to the patched version
│       └── NO  → workaround, replace dep, or allowlist with a review date
├── Severity: moderate
│   ├── Reachable in production? → next release cycle
│   └── Dev-only?               → backlog
└── Severity: low → fold into regular dependency updates
```

When you defer a fix, write down the reason and a review date. Otherwise "later" becomes "never".

## Review checklist

```markdown
### Authentication
- [ ] Passwords hashed with bcrypt / scrypt / argon2 (salt rounds ≥ 12)
- [ ] Session tokens are httpOnly, secure, sameSite
- [ ] Login endpoint rate-limited
- [ ] Password reset tokens expire

### Authorization
- [ ] Every protected endpoint checks user permissions
- [ ] Users can only access their own resources (ownership verified at fetch time)
- [ ] Admin actions verify admin role

### Input
- [ ] All external input validated at the boundary against an explicit schema
- [ ] DB queries parameterised
- [ ] Output encoded / escaped
- [ ] File uploads: type allowlist, size cap, magic-byte check if critical

### Data
- [ ] No secrets in code, logs, or git history
- [ ] Sensitive fields stripped from API responses
- [ ] PII encrypted at rest where applicable

### Infrastructure
- [ ] Security headers configured (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- [ ] CORS restricted to known origins (no `*` with credentials)
- [ ] Dependencies scanned, no unaddressed critical/high findings
- [ ] Error responses don't expose internals (no stack traces, no DB error text)
```

## Honesty

- **No security theatre.** A check that doesn't actually enforce anything is worse than no check. It lies to the next reader.
- **No "internal tool, doesn't matter".** Internal tools get compromised; attackers go for the weakest link.
- **No "the framework handles it".** Frameworks give you tools, not guarantees. You still have to use them correctly.
- **Quantify the risk when you can.** "This grants every authenticated user read access to other users' tasks" beats "this might be a problem".
- **If in doubt, escalate.** Security-sensitive uncertainty is a "stop and ask", not a "guess and ship".
