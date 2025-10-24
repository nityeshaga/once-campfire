---
name: hotwire-decision-maker
description: Guide architectural decisions for building modern UIs with Hotwire (Turbo + Stimulus). Use when planning features, choosing between Turbo tools, or deciding on UI implementation approaches. Provides decision frameworks based on ease-of-use vs responsiveness trade-offs, real-world patterns from Campfire, and first-principles understanding of when to use Drive vs Frames vs Streams vs Morphing vs Stimulus.
---

# Hotwire Decision-Maker

## Overview

This skill helps make informed architectural decisions when building modern, responsive UIs with Hotwire. It provides decision frameworks for choosing between Turbo Drive, Turbo Frames, Turbo Streams, Turbo Morphing, and Stimulus based on the specific use case, trade-offs, and desired user experience.

## When to Use This Skill

Activate this skill when:
- **Planning a new feature's UI** - Determine the right Hotwire approach
- **Choosing between Turbo tools** - Decide Drive vs Frames vs Streams vs Morphing
- **Reviewing existing Hotwire code** - Identify anti-patterns and optimization opportunities
- **Debugging slow/janky UIs** - Understand what went wrong architecturally
- **Evaluating SSR+Hotwire vs React/Vue** - Make informed framework decisions
- **Implementing real-time features** - Choose the right broadcasting pattern

## The Core Decision Framework

DHH's **Ease vs Responsiveness** graph (Rails World 2023):

```
Responsiveness
       ↑
   High|  Turbo Stream Actions
       |  (Most powerful, most setup)
       |
       |           Turbo Frames
       |           (Medium effort, good responsiveness)
       |
       |                    Turbo 8 Morphing ★
       |                    (HIGH responsiveness, LOW effort)
       |
       |                    Turbo Drive
    Low|                    (Easiest, moderate responsiveness)
       |________________________→
        Hard              Easy
               Developer Happiness
```

**The Revolutionary Insight:** Turbo 8 Morphing gives you Turbo Streams-level responsiveness with Turbo Drive-level ease!

## The Four Tools and When to Use Them

### 1. Turbo Drive (Start Here)

**What it does:** Intercepts link clicks and form submissions, replaces `<body>` with AJAX, keeps `<head>` intact.

**Use when:**
- ✅ Building standard CRUD pages
- ✅ Simple navigation between pages
- ✅ You want zero setup
- ✅ Traditional SSR feel is acceptable

**Don't use when:**
- ❌ You need partial page updates
- ❌ You want to preserve scroll position during updates
- ❌ Multiple sections need independent navigation

**Example:**
```ruby
# Controller - nothing special needed!
def update
  @post.update(post_params)
  redirect_to @post
end
```

Result: Smooth navigation, no white flash, but whole `<body>` replaces.

---

### 2. Turbo Drive + Morphing (The Game Changer)

**What it does:** Same as Turbo Drive but diffs the DOM and surgically updates only what changed.

**Use when:**
- ✅ Form submissions that redirect back to same page
- ✅ You want to preserve scroll, selection, focus
- ✅ Real-time collaborative updates (via broadcasting)
- ✅ You want Turbo Streams power with Drive simplicity

**Enable it:**
```erb
<%= turbo_refreshes_with method: :morph, scroll: :preserve %>
```

**Example - Collaborative Task Manager:**
```ruby
# Model - ONE LINE for real-time updates!
class Project < ApplicationRecord
  broadcasts_refreshes
end

class Task < ApplicationRecord
  belongs_to :project, touch: true  # Triggers project refresh
end
```

When ANY user updates a task, ALL users see the update with their scroll preserved!

**The key win:** Replaces complex Turbo Stream templates with one simple broadcast.

---

### 3. Turbo Frames (Navigation Boundaries)

**What it does:** Creates an independent section with its own navigation context. Links/forms inside stay inside.

**Use when:**
- ✅ Modals, wizards, multi-step forms
- ✅ Tabs that load different content
- ✅ Sections with independent pagination
- ✅ Lazy-loaded content with `src` attribute
- ✅ You want to compose a page from multiple URLs

**Don't use when:**
- ❌ Just as a container for Turbo Streams (use plain div)
- ❌ You don't want navigation trapped inside
- ❌ Simple real-time updates (use Streams instead)

**Example - Modal:**
```erb
<turbo-frame id="modal">
  <%= form_with model: @user do |f| %>
    <!-- Form submits INSIDE modal -->
  <% end %>

  <%= link_to "Terms", terms_path %>
  <!-- Opens terms INSIDE modal -->
</turbo-frame>
```

**Example - Lazy Loading Dashboard:**
```erb
<div class="dashboard">
  <h1>Dashboard</h1>

  <!-- Loads in parallel after page loads -->
  <turbo-frame id="activity" src="/dashboard/activity">
    <%= render "skeleton" %>
  </turbo-frame>

  <turbo-frame id="stats" src="/dashboard/stats" loading="lazy">
    <!-- Only loads when scrolled into view -->
    <div class="skeleton animate-pulse"></div>
  </turbo-frame>
</div>
```

**Critical:** Frames TRAP navigation! Every link/form inside is scoped to the frame unless you use `data-turbo-frame="_top"`.

---

### 4. Turbo Streams (Surgical DOM Updates)

**What it does:** Sends specific DOM manipulation instructions (append, prepend, replace, update, remove, before, after).

**Two modes:**
1. **HTTP Response** - After form submission (only acting user sees update)
2. **WebSocket Broadcasting** - Real-time updates (all users see update)

**Use when:**
- ✅ Multiple DOM updates from one action
- ✅ Real-time chat/notifications
- ✅ Appending to lists (not replacing entire list)
- ✅ Need precise control over updates
- ✅ Complex form responses with multiple region updates

**Don't use when:**
- ❌ Simple form redirect (use Morphing instead)
- ❌ Only one region updates (use Frame instead)
- ❌ You'll forget to update a region (use Morphing for safety)

**Example - Form Response (HTTP):**
```ruby
def create
  @message = @room.messages.create(message_params)

  respond_to do |format|
    format.turbo_stream
  end
end
```

```erb
<%# create.turbo_stream.erb %>
<%= turbo_stream.append "messages", @message %>
<%= turbo_stream.update "count", @room.messages.count %>
<%= turbo_stream.replace "flash", partial: "layouts/flash" %>
```

**Example - Real-time Broadcasting:**
```erb
<%# View - Subscribe %>
<%= turbo_stream_from @chat %>
<div id="messages"></div>
```

```ruby
# Model - Broadcast
class Message < ApplicationRecord
  after_create_commit do
    broadcast_append_to chat, target: "messages"
  end
end
```

---

### 5. Stimulus (JavaScript Sprinkles)

**What it does:** Adds client-side interactivity without managing state in JavaScript.

**Use when:**
- ✅ Dropdowns, tooltips, modals (UI behavior)
- ✅ Form validation, character counters
- ✅ Drag and drop, copy to clipboard
- ✅ Keyboard shortcuts, command palettes
- ✅ Client-side filtering/sorting (no server needed)
- ✅ Any JavaScript behavior that doesn't need server data

**Don't use when:**
- ❌ You need to fetch data from server (use Turbo Frame/Stream)
- ❌ You're managing complex application state (that's the DOM's job)
- ❌ You're building data visualizations (unless simple)

**Example - Dropdown:**
```html
<div data-controller="dropdown">
  <button data-action="click->dropdown#toggle">Menu</button>
  <div data-dropdown-target="menu" class="hidden">
    <a href="/profile">Profile</a>
  </div>
</div>
```

```javascript
// dropdown_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["menu"]

  toggle() {
    this.menuTarget.classList.toggle("hidden")
  }
}
```

**Key insight:** State lives in the DOM (classes, attributes), not in JavaScript variables.

---

## Common Decision Scenarios

### Scenario 1: Updating a Counter

**Question:** User clicks "Like". Show new like count.

**Options:**
- Turbo Drive: ❌ Whole page refreshes, loses scroll
- Turbo Morphing: ✅ **BEST** - Preserves everything, updates count
- Turbo Frame: ❌ Overkill, creates navigation boundary
- Turbo Stream: ✅ Works, but more code than morphing
- Stimulus: ❌ No server data access

**Recommendation:** Use **Turbo Morphing** - simplest and preserves state.

---

### Scenario 2: Real-time Chat

**Question:** Show new messages from other users instantly.

**Options:**
- Turbo Drive: ❌ No real-time capability
- Turbo Morphing: ❌ Requires page refresh trigger
- Turbo Frame: ❌ Wrong tool for broadcasts
- Turbo Stream: ✅ **BEST** - WebSocket broadcasting
- Stimulus: ❌ No server connection

**Recommendation:** Use **Turbo Streams** with broadcasting.

```ruby
class Message < ApplicationRecord
  after_create_commit -> { broadcast_append_to chat }
end
```

---

### Scenario 3: Modal with Multi-step Form

**Question:** Edit user profile in a modal with multiple steps.

**Options:**
- Turbo Drive: ❌ Navigates whole page
- Turbo Morphing: ❌ Can't scope navigation to modal
- Turbo Frame: ✅ **BEST** - Perfect for scoped navigation
- Turbo Stream: ❌ Overly complex for this
- Stimulus: ❌ Can't handle server-side steps

**Recommendation:** Use **Turbo Frame** for the modal.

```erb
<turbo-frame id="modal">
  <!-- All navigation stays inside -->
</turbo-frame>
```

---

### Scenario 4: Dashboard with Slow Widgets

**Question:** Dashboard has 5 widgets, 2 are slow. Don't block page load.

**Options:**
- Turbo Drive: ❌ Sequential loading blocks page
- Turbo Morphing: ❌ Still sequential
- Turbo Frame: ✅ **BEST** - Parallel loading with `src`
- Turbo Stream: ❌ Wrong use case
- Stimulus: ❌ Can't fetch from server initially

**Recommendation:** Use **Turbo Frames** with `src` for parallelization.

```erb
<turbo-frame id="fast-widget">
  <%= render @fast_data %>  <!-- Renders immediately -->
</turbo-frame>

<turbo-frame id="slow-widget" src="/widgets/slow">
  <%= render "skeleton" %>  <!-- Loads in parallel -->
</turbo-frame>

<turbo-frame id="very-slow" src="/widgets/very-slow" loading="lazy">
  <!-- Only loads when scrolled into view -->
</turbo-frame>
```

---

### Scenario 5: Filter Search Results

**Question:** User types in search box. Filter list without server round-trip.

**Options:**
- Turbo Drive: ❌ Full page reload per keystroke
- Turbo Morphing: ❌ Requires server
- Turbo Frame: ❌ Requires server
- Turbo Stream: ❌ Requires server
- Stimulus: ✅ **BEST** - Pure client-side filtering

**Recommendation:** Use **Stimulus** for client-side filtering.

```html
<div data-controller="filter">
  <input data-action="input->filter#search" data-filter-target="input">

  <div data-filter-target="item">Item 1</div>
  <div data-filter-target="item">Item 2</div>
</div>
```

---

## Critical Anti-Patterns to Avoid

### Anti-Pattern 1: Using Frames Just as Stream Targets

**❌ WRONG:**
```erb
<turbo-frame id="notifications">
  <div id="notification-list"></div>
</turbo-frame>

<%= turbo_stream.append "notification-list", @notification %>
```

**Problem:** The frame adds no value and TRAPS navigation in notifications!

**✅ RIGHT:**
```erb
<div id="notification-list"></div>

<%= turbo_stream.append "notification-list", @notification %>
```

Turbo Streams can target ANY element with an ID. No frame needed!

---

### Anti-Pattern 2: Complex Stream Templates Instead of Morphing

**❌ WRONG (Before Turbo 8):**
```ruby
after_update_commit do
  broadcast_replace_to project, target: "task_#{id}"
  broadcast_update_to project, target: "progress_bar"
  broadcast_update_to project, target: "complete_count"
  # Easy to forget regions!
end
```

**✅ RIGHT (With Turbo 8):**
```ruby
class Project < ApplicationRecord
  broadcasts_refreshes  # One line updates everything!
end

class Task < ApplicationRecord
  belongs_to :project, touch: true
end
```

---

### Anti-Pattern 3: Frame Boundaries Too Large

**❌ WRONG:**
```erb
<turbo-frame id="entire-dashboard">
  <!-- 500 lines of HTML -->
</turbo-frame>
```

**Problem:** Large frames = more HTML to swap = slower updates.

**✅ RIGHT:**
```erb
<turbo-frame id="user-stats">...</turbo-frame>
<turbo-frame id="activity">...</turbo-frame>
<turbo-frame id="notifications">...</turbo-frame>
```

---

### Anti-Pattern 4: Using Streams to Replace Entire Lists

**❌ WRONG:**
```ruby
turbo_stream.replace "emails", render(@emails)  # Re-renders ALL emails
```

**✅ RIGHT:**
```ruby
turbo_stream.prepend "emails", @new_email  # Adds just the new one
```

---

## The Mental Models

### Turbo Drive + Morphing
**Think:** "I want traditional SSR navigation, but preserve screen state"
**State:** Server owns all state, DOM is kept in sync

### Turbo Frames
**Think:** "This section is a mini-app with its own navigation"
**State:** Server owns state, frame creates boundary

### Turbo Streams
**Think:** "I'm giving surgical DOM instructions to the browser"
**State:** Server pushes state changes to specific DOM elements

### Stimulus
**Think:** "I'm adding behavior to HTML, not managing state"
**State:** DOM owns state (as attributes), JavaScript just manipulates it

---

## Decision Flowchart

```
Start: I need to implement a UI feature
│
├─ Is it pure client-side behavior? (dropdown, tooltip, keyboard shortcut)
│  └─ YES → Use Stimulus
│
├─ Does it need server data?
│  │
│  ├─ Is it a standard page navigation/form submission?
│  │  └─ YES → Start with Turbo Drive + Morphing
│  │
│  ├─ Does it need independent navigation context? (modal, wizard, tabs)
│  │  └─ YES → Use Turbo Frame
│  │
│  ├─ Is it real-time updates from other users?
│  │  └─ YES → Use Turbo Streams with broadcasting
│  │
│  ├─ Does it update multiple scattered page regions?
│  │  │
│  │  ├─ Same user action?
│  │  │  └─ Consider Turbo Morphing first (easier)
│  │  │
│  │  └─ Real-time from others?
│  │     └─ Use Turbo Streams with broadcasting
│  │
│  └─ Do you need lazy loading or parallel loading?
│     └─ YES → Use Turbo Frames with `src` and `loading` attributes
```

---

## Reference Materials

This skill includes:

1. **hotwire-aha-moments.md** - Comprehensive first-principles learning notes covering:
   - Why Hotwire exists (the false SSR vs SPA dichotomy)
   - Detailed explanations of each tool
   - Real-world scenarios and examples
   - Common pitfalls and anti-patterns

2. **Campfire Codebase** (in `.claude/skills/dhh-rails-reviewer/references/`):
   - Real production examples from 37signals
   - Turbo Frame patterns in views
   - Stimulus controllers in `app/javascript/controllers/`
   - Broadcasting patterns in models
   - Turbo Stream templates

3. **Official Hotwire Docs** (add to `references/` as needed):
   - Turbo handbook
   - Stimulus handbook
   - API references

---

## When Reviewing Code

Ask these questions:

1. **Is the simplest tool being used?**
   - Don't use Streams if Morphing works
   - Don't use Frames if just targeting with Streams

2. **Is navigation correctly scoped?**
   - Links in Frames should stay in Frames
   - Use `data-turbo-frame="_top"` to break out when needed

3. **Is state in the right place?**
   - Server state → Server
   - UI state → DOM (not JavaScript variables)

4. **Could parallelization help?**
   - Slow page loads → Consider Frames with `src`
   - Heavy widgets → Use `loading="lazy"`

5. **Is morphing preserving what matters?**
   - Scroll position → ✅ Should preserve
   - Text selection → ✅ Should preserve
   - Form focus → ✅ Should preserve

---

## Summary

**The Hotwire Philosophy:**
- Start simple (Turbo Drive + Morphing)
- Add boundaries when needed (Turbo Frames)
- Use surgical updates sparingly (Turbo Streams)
- Sprinkle JavaScript behavior (Stimulus)

**The Key Insight:**
Most web apps are "information apps" (GitHub, Basecamp, Shopify), not "interaction apps" (Figma, Google Docs). Information apps don't need complex JavaScript frameworks - they need smart HTML delivery.

**When in Doubt:**
1. Can Turbo Morphing handle it? → Use that (easiest)
2. Need navigation boundaries? → Use Frames
3. Need real-time or multi-region updates? → Use Streams
4. Need client-side behavior? → Use Stimulus

The goal is **maximum user experience with minimum developer complexity**.
