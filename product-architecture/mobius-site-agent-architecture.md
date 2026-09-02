# Mobius Site Agent — Product Architecture

> An agent that interviews you, learns your story, and builds you a personal website. Not a template filler — a conversational design partner that lives in your Mobius system.

---

## 1. Product Concept

**What it is:** A specialized Mobius agent that talks to you, gets to know your work and goals, analyzes your photos, and produces a personal brand website. It iterates with you conversationally until it's right.

**What it isn't:** A drag-and-drop builder, a template marketplace, or a one-shot generator. The value is the agent's judgment and the ongoing relationship.

**Positioning:** "Tell the agent your story. Get a site that actually sounds like you."

**Product name options:**
- **SiteWeaver** — agent that weaves your story into a site
- **Presence** — your online presence, built by an agent
- **Mobius Sites** — simple, fits the ecosystem
- **SiteCraft** — craft your site with an agent

---

## 2. Agent Interaction Design

### 2.1 Conversation Flow

The agent guides the user through a natural interview, not a form:

```
Agent: "Hey — what do you do? Not your job title, just... what do you actually do?"
User:  "I help people build hand-balancing and movement skills."
Agent: "Nice. Who's it for? People who've never tried, or folks with some experience?"
User:  "All levels — from first kick-up to freestanding handstand."
Agent: "What's the one thing you want someone to feel when they land on your site?"
User:  "Like this is something they can actually do — not magic, just practice."
```

The agent builds a **You Model** — a structured representation of the person — from the conversation.

### 2.2 Interaction Surfaces

| Surface | Use Case | Priority |
|---------|----------|----------|
| Telegram bot | Primary — conversational, always with you | P0 |
| Web chat | In-browser, side-by-side with live preview | P1 |
| Voice (Voice Room) | Natural conversation, discovery phase | P2 |

### 2.3 Approval Gates

The agent never deploys without explicit approval. Gates:

1. **You Model review** — "Here's what I learned about you. Right?"
2. **Design direction** — "Here are 3 directions. Which feels closest?"
3. **Content review** — "Here's the copy. Anything that doesn't sound like you?"
4. **Photo treatment** — "Here's how your photos will look. Crop adjustments?"
5. **Deploy** — "Ready to push live?"

Each gate is a conversation, not a checkbox.

### 2.4 Ongoing Relationship

The agent doesn't disappear after launch:
- "Your site's been up 2 weeks. Want me to add that testimonial from Sarah?"
- "I noticed your gallery could use a stronger finale shot. Want me to re-analyze your photos?"
- "You mentioned wanting to add online classes. Want me to mock up a 'Services' section?"

---

## 3. Module Architecture

The product is built as Mobius modules — each is independently testable and replaceable.

```
┌─────────────────────────────────────────────────────┐
│                  Mobius Site Agent                   │
│                                                       │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │  Discovery   │  │   Content    │  │   Design     │ │
│  │  Module      │  │   Module     │  │   Module     │ │
│  │             │  │              │  │              │ │
│  │ Interview   │  │ Headlines    │  │ Color/Type   │ │
│  │ You Model   │  │ About text   │  │ Layout       │ │
│  │ Goals       │  │ Project cards│  │ Variants     │ │
│  │ Audience    │  │ CTAs         │  │ Cropping     │ │
│  └──────┬──────┘  └──────┬───────┘  └──────┬──────┘ │
│         │                │                  │        │
│         └────────────────┼──────────────────┘        │
│                          │                           │
│                   ┌──────┴───────┐                   │
│                   │  Build Module │                   │
│                   │              │                   │
│                   │ Assemble HTML│                   │
│                   │ CSS/JS       │                   │
│                   │ Versioning   │                   │
│                   └──────┬───────┘                   │
│                          │                           │
│         ┌────────────────┼────────────────┐          │
│         │                │                │          │
│  ┌──────┴──────┐  ┌──────┴───────┐  ┌────┴───────┐  │
│  │   Photo     │  │   Deploy     │  │  Iterate   │  │
│  │   Module    │  │   Module     │  │  Module    │  │
│  │             │  │              │  │            │  │
│  │ Analyze     │  │ GitHub Pages │  │ Feedback   │  │
│  │ Crop        │  │ VPS nginx    │  │ Versions   │  │
│  │ Compose     │  │ Netlify      │  │ Changelog  │  │
│  └─────────────┘  └──────────────┘  └────────────┘  │
│                                                       │
└─────────────────────────────────────────────────────┘
```

### 3.1 Discovery Module

**Purpose:** Build the You Model through conversation.

**Inputs:** User messages (text/voice), optional existing site URL, optional resume/LinkedIn.

**Outputs:** Structured You Model (JSON):
```json
{
  "name": "Alex Wright",
  "tagline": "Hand Balancing, Coordination & Movement Patterns",
  "what_you_do": "Help people build real physical capability through progressive coaching",
  "who_its_for": "All levels — from first kick-up to freestanding handstand",
  "vibe": ["grounded", "progressive", "honest", "non-gimmicky"],
  "goals": ["book consultations", "show credibility", "share progressions"],
  "audience": "Adults interested in movement practice",
  "content_sections": ["about", "teach", "process", "gallery", "contact"],
  "photo_notes": "Full-body action shots, minimal background clutter preferred"
}
```

**Key behaviors:**
- Asks open questions, not "fill in the blank"
- Reflects back what it heard for confirmation
- Detects vibe words and tone from how the person writes/speaks
- Handles "I don't know" gracefully — offers examples

### 3.2 Content Module

**Purpose:** Generate and refine site copy from the You Model.

**Inputs:** You Model, optional existing copy.

**Outputs:** Content blocks (headlines, about text, project/service cards, CTAs).

**Key behaviors:**
- Writes in the user's voice (learned from conversation)
- Offers 2–3 options for headlines, not 10
- Flags anything that sounds generic or AI-polished
- Keeps copy concise — respects that visitors skim

### 3.3 Design Module

**Purpose:** Produce design direction and layout variants.

**Inputs:** You Model, content blocks, photo analysis.

**Outputs:** 2–4 HTML prototypes (like the movement coach variants).

**Key behaviors:**
- Generates variants that differ meaningfully (not just color swaps)
- Crops photos based on composition analysis
- Tests name prominence, photo treatment, nav simplicity
- Can invoke Claude Code as a design reviewer (bounded contractor)

### 3.4 Photo Module

**Purpose:** Analyze and treat user photos.

**Inputs:** Photo files/URLs.

**Outputs:** Cropped variants, composition notes, recommendations.

**Key behaviors:**
- Uses vision analysis to understand composition
- Recommends crops that emphasize the subject
- Detects background clutter, poor framing
- Generates `object-position` and `aspect-ratio` values

### 3.5 Build Module

**Purpose:** Assemble the chosen design into a deployable site.

**Inputs:** Chosen variant, content blocks, photo treatments.

**Outputs:** Self-contained HTML/CSS/JS (and optional React/Vue if needed).

**Key behaviors:**
- Produces clean, semantic HTML
- Responsive by default
- Accessible (alt text, contrast, keyboard nav)
- Fast (no heavy frameworks unless needed)
- Versioned (each build is a commit)

### 3.6 Deploy Module

**Purpose:** Push the site live.

**Inputs:** Built site files, deploy target config.

**Outputs:** Live URL.

**Deploy targets (priority order):**
1. **GitHub Pages** — free, simple, versioned
2. **VPS nginx** — full control, custom domain
3. **Cloudflare Pages** — fast global CDN
4. **Netlify** — easy forms, analytics

**Key behaviors:**
- Never deploys without explicit approval
- Shows preview before going live
- Handles custom domain + SSL
- Keeps previous version for rollback

### 3.7 Iterate Module

**Purpose:** Ongoing refinement after launch.

**Inputs:** User feedback, analytics (optional), new content/photos.

**Outputs:** Updated site, version history.

**Key behaviors:**
- Tracks what changed and why
- Suggests improvements based on goals
- Handles "add a section" / "change this photo" / "rewrite this" naturally
- Maintains a changelog the user can review

---

## 4. Data Model

### 4.1 You Model (per user)

```json
{
  "user_id": "uuid",
  "created_at": "2026-09-02T...",
  "updated_at": "2026-09-02T...",
  "name": "Alex Wright",
  "tagline": "...",
  "what_you_do": "...",
  "who_its_for": "...",
  "vibe": ["grounded", "progressive"],
  "goals": ["book consultations"],
  "audience": "...",
  "content_sections": ["about", "teach", "gallery", "contact"],
  "photo_notes": "...",
  "voice_samples": ["quote1", "quote2"],
  "preferences": {
    "color_warmth": "warm",
    "layout_density": "airy",
    "name_prominence": "high"
  }
}
```

### 4.2 Site Config (per site)

```json
{
  "site_id": "uuid",
  "user_id": "uuid",
  "name": "Alex Wright — Movement Coach",
  "domain": "alexwright.example.com",
  "deploy_target": "github_pages",
  "deploy_url": "https://mobiusframeworks.github.io/alex-wright/",
  "repo_url": "https://github.com/mobiusframeworks/alex-wright",
  "selected_variant": "c",
  "color_bg": "#faf9f6",
  "color_ink": "#1a1a1a",
  "color_accent": "#b5651d",
  "font_heading": "system-ui",
  "font_body": "system-ui",
  "created_at": "2026-09-02T...",
  "updated_at": "2026-09-02T..."
}
```

### 4.3 Content Blocks (per site)

```json
{
  "block_id": "uuid",
  "site_id": "uuid",
  "section": "hero",
  "type": "headline",
  "content": "Build strength, control & awareness through hand balancing and movement",
  "variant": "final",
  "created_at": "2026-09-02T..."
}
```

### 4.4 Versions (per site)

```json
{
  "version_id": "uuid",
  "site_id": "uuid",
  "commit_sha": "abc123",
  "message": "Tighten photo crop, simplify nav to 4 items",
  "deployed": true,
  "deployed_at": "2026-09-02T...",
  "created_at": "2026-09-02T..."
}
```

---

## 5. Technical Stack

### 5.1 Agent Runtime

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Agent framework | Hermes Agent (existing) | Already running in Mobius, proven |
| Profile | `repo-siteagent` (new) | Scoped to site agent work |
| Conversation | Telegram bot + web chat | Primary surfaces |
| Memory | Hermes memory + You Model JSON | Persistent user understanding |
| Vision | Hermes `vision_analyze` | Photo analysis |
| Code execution | Sandboxed Python/Node | Build assembly |

### 5.2 Web Chat Frontend (if not just Telegram)

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Framework | Preact or Alpine.js | Lightweight, fast |
| Styling | Tailwind CSS | Rapid prototyping |
| Live preview | iframe + postMessage | Real-time site preview |
| WebSocket | For live updates | Agent → frontend streaming |

### 5.3 Storage

| Component | Choice | Rationale |
|-----------|--------|-----------|
| You Model + Site Config | SQLite (dev) / PostgreSQL (prod) | Simple, reliable |
| Photos | Local filesystem + S3-compatible | Fast access, backup |
| Site builds | Git repos | Versioned, deployable |
| Agent state | Hermes session + JSON files | Fits existing patterns |

### 5.4 VPS Deployment

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Container | Docker Compose | Simple, reproducible |
| Agent service | Hermes profile in container | Runs the SiteAgent |
| Web chat | nginx-served static + WebSocket | Lightweight |
| Site hosting | nginx static sites | Fast, simple |
| SSL | Let's Encrypt (certbot) | Free, automated |
| Domain | Custom per user or subdomain | alex.siteagent.io or custom |

---

## 6. VPS Architecture

```
┌─────────────────────────────────────────────────────┐
│                      VPS                             │
│                                                       │
│  ┌──────────────────────────────────────────────┐   │
│  │              Docker Compose                    │   │
│  │                                                │   │
│  │  ┌─────────────┐    ┌──────────────────────┐  │   │
│  │  │  SiteAgent   │    │    Web Chat          │  │   │
│  │  │  (Hermes)    │    │    (Preact + WS)     │  │   │
│  │  │             │    │                      │  │   │
│  │  │  Discovery  │◄──►│  Chat UI             │  │   │
│  │  │  Content    │    │  Live Preview        │  │   │
│  │  │  Design     │    │  Approval Gates      │  │   │
│  │  │  Build      │    │                      │  │   │
│  │  │  Deploy     │    └──────────────────────┘  │   │
│  │  │  Iterate    │                              │   │
│  │  └──────┬──────┘                              │   │
│  │         │                                      │   │
│  │         ▼                                      │   │
│  │  ┌─────────────┐    ┌──────────────────────┐  │   │
│  │  │  Git Repos   │    │    nginx              │  │   │
│  │  │  (per site)  │───►│    (static sites)     │  │   │
│  │  └─────────────┘    │    alex.siteagent.io  │  │   │
│  │                      │    bob.siteagent.io   │  │   │
│  │                      └──────────────────────┘  │   │
│  │                                                │   │
│  └──────────────────────────────────────────────┘   │
│                                                       │
│  ┌─────────────┐    ┌──────────────────────┐        │
│  │  PostgreSQL  │    │    certbot            │        │
│  │  (You Models │    │    (SSL)              │        │
│  │   Sites)     │    │                      │        │
│  └─────────────┘    └──────────────────────┘        │
│                                                       │
└─────────────────────────────────────────────────────┘
```

---

## 7. MVP Scope

### Phase 1: Foundation (Week 1–2)
- [ ] New Hermes profile: `repo-siteagent`
- [ ] Discovery Module: conversation flow → You Model JSON
- [ ] Content Module: generate headlines + about text from You Model
- [ ] Design Module: produce 3 HTML variants (reuse existing prototype workflow)
- [ ] Photo Module: analyze + crop via vision_analyze
- [ ] Build Module: assemble chosen variant into deployable HTML
- [ ] Deploy Module: push to GitHub Pages
- [ ] Telegram bot as primary interaction surface

### Phase 2: Polish (Week 3–4)
- [ ] Web chat frontend with live preview
- [ ] Approval gates (You Model review, design choice, deploy)
- [ ] Iterate Module: feedback → update → redeploy
- [ ] Version history + rollback
- [ ] VPS deployment with Docker Compose

### Phase 3: Product (Week 5–6)
- [ ] Custom domain support
- [ ] Multi-user (separate You Models, separate sites)
- [ ] Analytics integration (optional, privacy-respecting)
- [ ] Pricing/packaging exploration

---

## 8. What Makes This a Product, Not a Tool

| Tool | Product (SiteAgent) |
|------|---------------------|
| You fill out a form | The agent interviews you |
| You pick a template | The agent learns your vibe |
| You write the copy | The agent writes in your voice |
| You crop photos | The agent analyzes composition |
| You hit publish | The agent asks "ready?" |
| You're done | The agent suggests improvements |
| One-time transaction | Ongoing relationship |

The moat is the You Model — the more the agent knows you, the better the site gets. Switching cost is the accumulated understanding.

---

## 9. Open Questions

1. **Name:** SiteWeaver / Presence / Mobius Sites / SiteCraft / other?
2. **Pricing:** Free for personal, paid for pro (custom domain, analytics, priority)?
3. **Multi-tenancy:** One VPS per user, or shared with isolated sites?
4. **Framework:** Stick with Hermes agent, or custom agent loop for more control?
5. **Frontend:** Telegram-only for MVP, or web chat from day one?
6. **Open source:** Like Groundwork, or proprietary?

---

*Architecture version 0.1 — 2026-09-02*
