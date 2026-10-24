# BrandLens

**Intelligent Brand Governance — Review Content Before It Goes Live**

BrandLens is a B2B SaaS platform that reviews marketing and product content *before publication* against approved brand guidelines, facts, and assets. It detects inconsistent tone, banned terminology, unsupported claims, off-brand visuals, and accessibility issues — and explains every finding with a citation to the specific rule it violates.

Built for teams that publish through multiple employees, agencies, and AI tools, BrandLens prevents brand inconsistency before it damages credibility — while keeping humans fully in control of every decision.

---

## ✨ Why BrandLens

Growing organisations publish content through multiple employees, agencies and AI tools, making it difficult to maintain a consistent and trustworthy brand identity. Incorrect claims, outdated information, inconsistent language and off-brand visuals can damage credibility before anyone notices the problem.

BrandLens solves this by:
- **Detecting** issues before publication, not after
- **Explaining** every finding with a citation to the exact rule or approved fact
- **Suggesting** corrections without overriding important human decisions
- **Auditing** every action for traceability and compliance
- **Learning** over time through governance insights

---

## 🎯 Key Features

### Brand Knowledge Base
The single source of truth for all brand rules:
- **Guidelines** — versioned documents with effective dates and source references
- **Rules** — typed by category (`TONE`, `TERMINOLOGY`, `FACTUAL`, `CLAIM`, `VISUAL`, `ACCESSIBILITY`) with severity levels
- **Approved Facts** — verified statements with source citations
- **Terminology** — preferred and banned terms with suggested replacements
- **Approved Assets** — logos, images, and documents served via signed URLs

### Content Review
- Submit text, images, or documents for review
- Review runs a **hybrid pipeline**: deterministic checks first (banned terms, fact matching, missing alt text, contrast, logo usage), then LLM-assisted checks (tone, unsupported claims, messaging)
- Every finding includes: category, severity, flagged snippet, **the specific rule ID violated**, plain-language explanation, suggested correction, and **AI disclosure** (if applicable)

### Human Review Workflow
- **Roles**: Creator → Manager → Authorised Reviewer → Admin (RBAC enforced on every API route)
- **Per-finding actions**: Accept suggestion, Edit manually, Reject/Dismiss with required reason, Comment
- **State machine**: `Draft → Submitted → In Review → Changes Requested → Approved → Published`
- **Revision history** and **immutable audit trail** on every action

### Readiness Score & Blockers
- Live readiness score reflecting open findings
- Publication blocked until critical findings are resolved
- Suggestions applied inline with one click

### Governance Insights
- Recurring issue types and trends
- Most-violated guidelines
- **Ambiguity Radar** — flags guidelines with unusually high rejection rates (indicating unclear rules)
- Exportable CSV summaries

### Security & Trust
- Multi-tenant data isolation (org-scoped at the query layer)
- RBAC on every API route
- Signed, expiring asset URLs
- Immutable audit log for all sensitive actions
- AI/provider disclosure visible per finding
- Confidential material protection settings

---

## 🖥 Screens

| Screen | Purpose |
|---|---|
| **Dashboard** | Overview of in-progress work, governance audit stream, and core principles |
| **Inbox (Review Workspace)** | Side-by-side content editor with highlighted findings and action sidebar |
| **Brand Kit** | Admin view of guidelines, terminology, and approved assets |
| **Approvals Queue** | Role-based approval with self-approval prevention |
| **Insights** | First-pass yield, findings trends, ambiguity radar |
| **Settings** | Confidentiality controls, AI disclosure preferences |
| **Audit Log** | Full governance trail (slide-over panel) |

---

## 🏗 Architecture

```
┌────────────────────────────────────────────────────────┐
│  UI (Next.js App Router + Tailwind)                    │
│  Dashboard · Inbox · Brand Kit · Approvals · Insights  │
├────────────────────────────────────────────────────────┤
│  API Layer (RBAC middleware + tenant scoping)          │
├────────────────────────────────────────────────────────┤
│  Review Orchestrator (jobs queue)                      │
│  Submission → Engine → Findings → Persist              │
├────────────────────────────────────────────────────────┤
│  Engine                                               │
│  ├─ Deterministic checkers (always run)               │
│  └─ LLM checkers (optional, disclosed, degrade-safe)  │
├────────────────────────────────────────────────────────┤
│  Data Layer (Postgres + Prisma)                        │
│  Org · User · Membership · Guideline · Rule ·          │
│  ApprovedFact · Term · Asset · Submission · Finding ·  │
│  ReviewAction · AuditLog · InsightSnapshot             │
├────────────────────────────────────────────────────────┤
│  Infra                                                 │
│  Postgres · S3-compatible storage · JWT auth · Secrets │
└────────────────────────────────────────────────────────┘
```

### Design Principles

1. **Explainability over cleverness** — every finding traceable to a rule and a source
2. **Human control over automation** — nothing auto-applied or auto-approved
3. **Rules-first, LLM-second** — deterministic checks are the backbone; AI augments
4. **Security is a feature** — tenant isolation, RBAC, signed URLs, audit logs are not optional
5. **Graceful degradation** — the app works without an LLM configured

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| UI Primitives | Custom shadcn-style components |
| Icons | Font Awesome 6 |
| Fonts | Inter (sans), JetBrains Mono (mono) |
| Database | PostgreSQL |
| ORM | Prisma |
| Auth | JWT sessions (bcrypt + jsonwebtoken) |
| Storage | S3-compatible (AWS S3 / MinIO) |
| Validation | Zod |
| LLM | Provider-agnostic abstraction (deterministic-only mode supported) |

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- PostgreSQL 14+ (or Docker)
- Optional: S3-compatible storage (AWS S3 or MinIO) for asset uploads

### 1. Clone & Install

```bash
git clone https://github.com/your-username/brandlens.git
cd brandlens
npm install
```

### 2. Configure Environment

```bash
cp .env.example .env
```

Edit `.env`:

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/brandlens?schema=public"
JWT_SECRET="replace-with-a-random-string-at-least-32-chars-long"

# Storage (optional — needed for asset uploads)
S3_ENDPOINT="http://localhost:9000"
S3_REGION="us-east-1"
S3_ACCESS_KEY_ID="minioadmin"
S3_SECRET_ACCESS_KEY="minioadmin"
S3_BUCKET="brandlens-assets"
S3_FORCE_PATH_STYLE="true"

# LLM (optional — leave blank for deterministic-only mode)
LLM_PROVIDER=""
LLM_API_KEY=""

NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

### 3. Start Postgres

```bash
docker run -d --name brandlens-pg \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=brandlens \
  -p 5432:5432 postgres:16
```

### 4. (Optional) Start MinIO for Asset Uploads

```bash
docker run -d --name brandlens-minio \
  -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  minio/minio server /data --console-address ":9001"
```

Open http://localhost:9001 → login `minioadmin` / `minioadmin` → create bucket `brandlens-assets`.

### 5. Set Up Database

```bash
npx prisma generate
npx prisma migrate dev --name init
```

### 6. Run

```bash
npm run dev
```

Open **http://localhost:3000** → register → start creating your Brand Kit.

---

## 🧪 Demo Flow

The fastest way to see BrandLens in action:

1. **Register** an account (creates an organisation + admin membership)
2. **Brand Kit** → add a few rules across all 6 categories, some banned terms, and approved facts
3. **New Content** → submit a paragraph that includes a banned term + an unsupported claim + a missing alt-text image
4. **Review Workspace** → see findings highlighted, each citing its rule ID; **accept** one, **reject** one with a reason, **edit** one
5. **Submit for Review** → the document moves to the Approvals Queue
6. **Approvals** → switch to Reviewer persona → approve → watch the audit trail
7. **Insights** → see recurring findings and the Ambiguity Radar flag a frequently-rejected rule

That single flow demonstrates every core capability.

---

## 📸 Screenshots

> _Add your screenshots here once you've deployed._
>
> - `docs/screenshots/dashboard.png` — Governance Dashboard
> - `docs/screenshots/inbox.png` — Review Workspace with findings
> - `docs/screenshots/brand-kit.png` — Brand Knowledge Base
> - `docs/screenshots/insights.png` — Governance Insights

---

## 🗺 Roadmap

**v1.0 (current)** — Core review loop: knowledge base, submission, findings, human workflow, approvals, insights, audit trail.

**v1.1** — LLM-assisted tone and messaging checks, richer visualization on insights.

**v1.2** — Browser extension for reviewing content directly inside Google Docs, Notion, and WordPress.

**v1.3** — CMS integrations (Contentful, Webflow, Sanity) for automated pre-publish gates.

**v2.0** — Multi-tenant white-label deployments, SSO/SAML, marketplace of guideline templates.

---

## 🧱 Project Structure

```
brandlens/
├── prisma/
│   └── schema.prisma               # DB schema
├── public/                         # Static assets
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx                # Landing
│   │   ├── login/
│   │   ├── register/
│   │   ├── dashboard/
│   │   │   ├── page.tsx            # Dashboard
│   │   │   ├── kb/                 # Brand Knowledge Base
│   │   │   ├── inbox/              # Review Workspace
│   │   │   ├── queue/              # Approvals
│   │   │   ├── insights/
│   │   │   └── settings/
│   │   └── api/
│   │       ├── auth/               # Register / login / logout / me
│   │       ├── kb/                 # Guidelines / rules / facts / terms / assets
│   │       ├── submissions/
│   │       ├── findings/
│   │       └── health/
│   ├── components/
│   │   └── ui/                     # Button, Input, Label, Card, Table, Badge
│   └── lib/
│       ├── prisma.ts
│       ├── auth.ts                 # JWT + sessions
│       ├── storage.ts              # S3 client + signed URLs
│       ├── validators.ts           # Zod schemas
│       ├── api-helpers.ts          # withSession, requireAdmin
│       └── kb.ts                   # Knowledge base repository
├── .env.example
├── .gitignore
├── next.config.mjs
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## 🔐 Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | ✅ | Postgres connection string |
| `JWT_SECRET` | ✅ | Secret for signing session tokens (min 32 chars) |
| `JWT_EXPIRES_IN` | — | Session lifetime (default `7d`) |
| `S3_ENDPOINT` | — | S3/MinIO endpoint (blank = AWS S3) |
| `S3_REGION` | — | AWS region |
| `S3_ACCESS_KEY_ID` | — | Access key |
| `S3_SECRET_ACCESS_KEY` | — | Secret key |
| `S3_BUCKET` | — | Bucket name |
| `S3_FORCE_PATH_STYLE` | — | `true` for MinIO, `false` for AWS |
| `LLM_PROVIDER` | — | e.g. `openai`, `anthropic` — blank = deterministic-only |
| `LLM_API_KEY` | — | Provider API key |
| `NEXT_PUBLIC_APP_URL` | — | Public URL for links |

---

## 🧪 Testing the Core Loop

Quick sanity check once you're running:

```bash
# Health check — should return { ok: true, db: "up", ... }
curl http://localhost:3000/api/health

# Register
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@demo.test","password":"password123","name":"Admin","organisationName":"Demo Corp"}' \
  -c cookies.txt

# Verify session
curl http://localhost:3000/api/auth/me -b cookies.txt
```

Then in the browser: `/register` → `/dashboard` → `/dashboard/kb` → create a rule → create a submission → see findings.

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m "Add your feature"`)
4. Push (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

MIT — see [LICENSE](./LICENSE) for details.

---

## 🙏 Acknowledgements

- Design inspired by modern developer tools (Linear, Vercel, Stripe)
- Fonts: [Inter](https://rsms.me/inter/), [JetBrains Mono](https://www.jetbrains.com/lp/mono/)
- Icons: [Font Awesome](https://fontawesome.com/)

---

## 💬 Contact

**Project:** BrandLens — Intelligent Brand Governance
**Author:** Your Name ([@yourhandle](https://github.com/yourhandle))
**Email:** you@example.com

---

_Protect the brand before it ships._
