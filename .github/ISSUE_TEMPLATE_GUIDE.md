# GitHub Issue Template Guide

This document helps you choose the right issue template for your needs.

## Quick Reference

| Template | Use When | Label | Auto-Triage |
|----------|----------|-------|-------------|
| **Epic** | Large initiative with multiple stories/tasks | `epic` | No |
| **Story** | User-facing functionality to deliver | `story` | Yes (`needs-triage`) |
| **Enhancement** | Idea that needs validation/scoping | `enhancement` | Yes (`needs-triage`) |
| **Chore** | Technical work, no user-facing change | `chore` | No |
| **Bug** | Something broken that needs fixing | `bug` | Yes (`needs-triage`) |

## Detailed Guide

### 🎯 Epic
**When to use:** You're planning a large initiative that will involve multiple user stories or technical tasks.

**Examples:**
- Customer Self-Service Portal (containing login, dashboard, invoice viewing, etc.)
- ProxMox Automation Infrastructure (containing Terraform setup, Ansible configs, testing, etc.)
- Satellite Office Deployment System (containing network setup, VM provisioning, monitoring, etc.)

**Key characteristics:**
- Takes multiple sprints/iterations to complete
- Contains 5+ child stories or tasks
- Aligns with product vision and roadmap phases
- Has clear success metrics

---

### 📖 User Story
**When to use:** You're describing specific user-facing functionality that delivers value to users.

**Examples:**
- As a PM, I want to view project costs on my mobile device
- As a customer, I want to download invoices in PDF format
- As a PM, I want to receive alerts when project costs exceed budget

**Key characteristics:**
- Can be completed in one iteration/sprint
- Delivers value to a specific user type
- Has clear acceptance criteria with Gherkin scenarios
- Tests a hypothesis about user behavior or value
- Part of a larger Epic

---

### ✨ Enhancement / Feature Request
**When to use:** You have an idea that hasn't been validated or fully scoped yet.

**Examples:**
- What if we added a chat feature for PM-customer communication?
- Could we integrate with Slack for project notifications?
- I think customers would like to see project photos

**Key characteristics:**
- Still an idea, not yet validated
- Needs research or stakeholder input
- May become a Story or Epic after validation
- Gets `needs-triage` label for prioritization

**Lifecycle:** Enhancement → (after validation) → Story or Epic

---

### 🔧 Chore / Technical Task
**When to use:** You're doing technical work that doesn't directly change user-facing functionality.

**Examples:**
- Refactor authentication middleware for consistency
- Update dependencies to address security vulnerabilities
- Set up automated testing infrastructure
- Implement Loki logging for new service
- Configure ProxMox VM templates

**Key characteristics:**
- Technical debt reduction
- Infrastructure improvements
- Developer experience enhancements
- Refactoring or optimization
- No direct user-facing changes
- Enables future features or improves maintainability

---

### 🐛 Bug Report
**When to use:** Something that should work is broken or behaving incorrectly.

**Examples:**
- Login button doesn't respond on mobile devices
- Project costs showing outdated data
- Email notifications not being sent
- CloudFlare tunnel connection dropping

**Key characteristics:**
- Something that previously worked (or should work)
- Clear current vs expected behavior
- Can be reproduced with specific steps
- Impacts users negatively
- Should include regression test strategy

---

## Backlog Item Flow

```
┌─────────────────┐
│  Enhancement    │  ← Ideas, not yet validated
│  needs-triage   │
└────────┬────────┘
         │
         │ (After validation & scoping)
         ▼
    ┌─────────┐
    │  Epic   │  ← Large initiative
    └────┬────┘
         │
         │ (Broken down into)
         ▼
    ┌─────────┐
    │  Story  │  ← User-facing work
    │  Chore  │  ← Technical work
    └─────────┘
```

**Bug Reports** can be filed at any time and addressed independently of the above flow.

---

## Label System

### Issue Type Labels
- `epic` - Large initiative (parent of multiple issues)
- `story` - User-facing functionality
- `enhancement` - Feature request or idea
- `chore` - Technical task
- `bug` - Something broken

### Auto-Applied Labels
- `needs-triage` - Applied automatically to Stories, Enhancements, and Bugs
- Remove this label after prioritization and assignment to an Epic or sprint

### Additional Labels (applied manually)
- `priority:critical` - Urgent work
- `priority:high` - Important but not urgent
- `priority:medium` - Should do soon
- `priority:low` - Nice to have
- `blocked` - Waiting on something
- `help wanted` - Looking for contributors
- `good first issue` - Good for newcomers

---

## Shell Aliases for Quick Issue Creation

Add these to your `~/.zshrc` for quick issue creation from the terminal:

```bash
# Create bug report
bug() {
  gh issue create --label "bug,needs-triage" --title "🐛 $1"
}

# Create feature request
feature() {
  gh issue create --label "enhancement,needs-triage" --title "✨ $1"
}

# Create chore/task
chore() {
  gh issue create --label "chore" --title "$1"
}

# Create story
story() {
  gh issue create --label "story,needs-triage" --title "$1"
}

# Create epic
epic() {
  gh issue create --label "epic" --title "[Epic]: $1"
}
```

**Usage examples:**
```bash
bug "Login button not working on mobile"
feature "Add Slack integration for notifications"
chore "Refactor auth middleware"
story "PM can view costs on mobile"
epic "Customer Self-Service Portal"
```

---

## Hypothesis-Driven Development

All **Stories** and **Epics** should include a hypothesis to test:

**Format:**
```
We believe that [building this feature]
will result in [specific outcome].

We'll know we're right when:
- [Measurable outcome 1]
- [Measurable outcome 2]
- [Measurable outcome 3]
```

**Example:**
```
We believe that providing PMs with mobile access to real-time project costs
will result in faster customer responses and increased confidence.

We'll know we're right when:
- PMs respond to cost questions 80% faster
- PM satisfaction with tools increases by 20%
- 90% of PMs use mobile view at least weekly
```

This ensures we're building with intent and can validate whether our assumptions were correct.

---

## Questions?

- **Not sure which template to use?** Start with Enhancement and we'll help you refine it.
- **Need to break down an Epic?** Create the Epic first, then create Stories/Chores and link them.
- **Found a bug?** Use the Bug template and include reproduction steps.
- **Technical work?** Use Chore template and explain the value/hypothesis.

---

**Last Updated:** October 14, 2025  
**Maintained by:** Molabs DevOps Team
