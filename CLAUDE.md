# CLAUDE.md - LEAG Platform

> **Purpose:** Provide Claude Code with comprehensive context for the LEAG platform monorepo.
> **Last Updated:** December 10, 2025

---

## Ownership & Licensing

**This platform and all associated technology is proprietary to Stryder Labs LLC.**

- All source code, design systems, and architectural patterns are owned by Stryder Labs LLC
- The LEAG Platform is a multi-tenant SaaS product for youth sports organizations
- Tenant-specific implementations exist as subdirectories (e.g., `nj-stars/`)

**Founder:** Abraham Pena

---

## Platform Vision

**LEAG** (League Excellence & Growth) is a **multi-tenant SaaS platform** designed for youth sports organizations including AAU teams, travel leagues, recreational programs, and athletic academies.

### Core Value Proposition

- **For Organizations:** Turnkey digital presence with e-commerce, member management, and event registration
- **For Parents:** Single portal to manage children's sports activities, payments, and communications
- **For Coaches:** Tools for scheduling, attendance, and direct service offerings (private lessons)

### Business Model

| Phase | Model | Description |
|-------|-------|-------------|
| **Phase 1** | Revenue Share | Platform fee on transactions (15-20% events/merch, 5% coach services) |
| **Phase 2** | Subscription | Monthly SaaS fee based on tier/features + reduced transaction fees |
| **Phase 3** | Hybrid | Base subscription + usage-based pricing for high-volume tenants |

---

## Repository Structure

```
leag/
├── CLAUDE.md                    # This file - platform-level context
├── documents/                   # Business documents (LLC, agreements, etc.)
│   └── LLC_FORMATION_BRIEF.md   # Stryder Labs LLC formation materials
├── nj-stars/                    # First tenant implementation
│   ├── CLAUDE.md               # Tenant-specific context
│   ├── backend/                # Django + Wagtail API
│   ├── frontend/               # Next.js application
│   ├── documentation/          # Project docs
│   └── docker-compose.yml      # Development containers
├── [future-tenant]/            # Additional tenants will follow same structure
└── shared/                     # [FUTURE] Shared libraries and components
```

---

## Current Tenants

### 1. NJ Stars Elite AAU (First Tenant)

| Field | Value |
|-------|-------|
| **Directory** | `nj-stars/` |
| **Status** | Active Development (MVP ~90% complete) |
| **Owner** | Kenneth Andrade |
| **Domain** | njstarselite.com |
| **Revenue Model** | Phase 1 (20% platform fee) |
| **Notes** | Founding client with future equity consideration |

**See `nj-stars/CLAUDE.md` for tenant-specific technical details.**

---

## Tech Stack (Platform Standard)

All tenant implementations follow this standard stack:

| Layer | Technology | Notes |
|-------|------------|-------|
| **Backend** | Django 5.0+ | REST API framework |
| **CMS** | Wagtail 7.x | Content management |
| **Frontend** | Next.js 14+ (App Router) | React with TypeScript |
| **Styling** | Tailwind CSS + shadcn/ui | Consistent design system |
| **Database** | PostgreSQL 14+ | Primary data store |
| **Auth** | django-allauth + NextAuth.js | OAuth support |
| **Payments** | Stripe | Connect for multi-tenant |
| **Deployment** | Railway (backend) + Vercel (frontend) | PaaS for simplicity |
| **Containers** | Docker + Docker Compose | Local development |

---

## Multi-Tenant Architecture

### Current State (Single-Tenant per Instance)

Each tenant currently has a separate deployment:
- Separate database
- Separate frontend deployment
- Separate backend deployment
- Shared codebase structure

### Future State (True Multi-Tenant)

Roadmap for shared infrastructure:
1. **Tenant Model:** Central tenant registry with configuration
2. **Domain Routing:** `{tenant}.leag.com` or custom domains
3. **Shared Database:** Tenant-scoped queries with row-level security
4. **Shared Services:** Single backend serving multiple tenants
5. **Stripe Connect:** Platform account with connected accounts per tenant

### Design Principles

When building features, always consider:

1. **Tenant Isolation:** Data must be properly scoped
2. **Configurable Branding:** Colors, logos, text in database/CMS
3. **Feature Flags:** Tenant-specific feature availability
4. **Billing Boundaries:** Clear revenue tracking per tenant
5. **Admin Hierarchy:** Platform admins vs tenant admins

---

## Development Workflow

### Working on a Specific Tenant

```bash
# Navigate to tenant directory
cd nj-stars/

# Follow tenant-specific CLAUDE.md for commands
make up        # Start Docker services
make seed      # Seed test data
```

### Platform-Level Tasks

For work that affects the platform structure or shared components:

```bash
# From root /leag directory
# [Commands will be added as shared infrastructure develops]
```

---

## Business Documents

### `/documents/` Directory

Contains confidential business materials:

| Document | Purpose |
|----------|---------|
| `LLC_FORMATION_BRIEF.md` | Information for legal counsel to form Stryder Labs LLC |

**Note:** This directory should be added to `.gitignore` if the repository becomes shared or public.

---

## Key Business Terms

### Stryder Labs LLC

- **Entity:** Limited Liability Company (formation in progress)
- **Owner:** Abraham Pena (sole member)
- **Purpose:** Owns and operates the LEAG Platform

### Revenue Relationships

| Tenant | Current Terms | Future Terms |
|--------|--------------|--------------|
| NJ Stars | 20% platform fee | Subscription + equity consideration for Kenneth Andrade |
| [Future] | Negotiated per tenant | Standard subscription tiers |

### Equity Considerations

Kenneth Andrade (NJ Stars) has been offered a small equity stake in Stryder Labs LLC, contingent on:
- Successful transition from platform fee to subscription model
- Continued partnership as founding client
- Terms documented in legal agreements

---

## Getting Started (New Tenant Setup)

### Prerequisites

1. Docker and Docker Compose installed
2. Node.js 18+ and npm
3. Python 3.11+
4. PostgreSQL 14+ (or use Docker)

### Creating a New Tenant

1. Copy tenant template structure (based on `nj-stars/`)
2. Update configuration for new tenant
3. Set up deployment infrastructure
4. Configure Stripe Connect account
5. Customize branding and content

**Detailed guide:** `documentation/NEW_TENANT_SETUP.md` (to be created)

---

## Contacts

### Platform Owner

**Abraham Pena** - Stryder Labs LLC
- Role: Founder, Lead Developer
- Email: [EMAIL]

### Current Tenants

**Kenneth Andrade** - NJ Stars Elite AAU
- Role: Founding Client
- Email: [EMAIL]

---

## Related Documentation

| Location | Content |
|----------|---------|
| `nj-stars/CLAUDE.md` | NJ Stars technical documentation |
| `nj-stars/documentation/` | Detailed project docs, meeting notes |
| `documents/LLC_FORMATION_BRIEF.md` | LLC formation materials |

---

## Initializing CLAUDE.md for This Directory

If you're setting up a new clone or the parent `/leag` directory:

### Steps to Initialize

1. **Create the CLAUDE.md file:**
   ```bash
   # This file should exist at /leag/CLAUDE.md
   # Copy this template and customize as needed
   ```

2. **Create the documents directory:**
   ```bash
   mkdir -p documents
   ```

3. **Add to .gitignore (if needed):**
   ```bash
   # Add to /leag/.gitignore
   documents/  # Confidential business documents
   ```

4. **Link tenant CLAUDE.md files:**
   Each tenant subdirectory should have its own `CLAUDE.md` with technical specifics.

### CLAUDE.md Best Practices

- **Keep updated:** Update when major changes occur
- **Be specific:** Include concrete paths, commands, and configurations
- **Separate concerns:** Platform-level info here, tenant-specific in subdirectories
- **Mark sensitive info:** Use placeholders for credentials and personal data

---

*Last Updated: December 10, 2025*
