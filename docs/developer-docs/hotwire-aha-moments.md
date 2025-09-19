# Hotwire Aha Moments

## Table of Contents

- [1. The Fundamental Insight](#1-the-fundamental-insight)
  - [DHH's Revolutionary Insight](#dhhs-revolutionary-insight)
  - [The Three "Impossible" Things SSR Couldn't Do (Pre-Hotwire)](#the-three-impossible-things-ssr-couldnt-do-pre-hotwire)
- [2. Turbo Frames: The "Impossible" Partial Update](#2-turbo-frames-the-impossible-partial-update)
  - [The Old World Problem](#the-old-world-problem)
  - [The Turbo Frame Solution](#the-turbo-frame-solution)
  - [Turbo Streams: Always Sends HTML Chunks](#turbo-streams-always-sends-html-chunks)
  - [Turbo Frames: Can Send Full Page OR Chunks](#turbo-frames-can-send-full-page-or-chunks)
  - [Turbo Frames: Parallelization & Lazy Loading](#turbo-frames-parallelization--lazy-loading)
  - [Frames with `src`: Composable, Parallel-Loading Pages](#frames-with-src-composable-parallel-loading-pages)
  - [Aside: Why Your Hotwire App Might Feel Slow Even With Turbo Frames](#aside-why-your-hotwire-app-might-feel-slow-even-with-turbo-frames)
- [3. Turbo Streams Deep Dive: Two Modes, One Format](#3-turbo-streams-deep-dive-two-modes-one-format)
  - [Mode 1: HTTP Response (Form Submissions)](#mode-1-http-response-form-submissions)
  - [Mode 2: WebSocket Broadcasting (Real-time)](#mode-2-websocket-broadcasting-real-time)
  - [The Golden Rules of Turbo Stream Targeting](#the-golden-rules-of-turbo-stream-targeting)
  - [What You CAN'T Do](#what-you-cant-do)
  - [Broadcasting Patterns](#broadcasting-patterns)
  - [The Implementation Magic](#the-implementation-magic)
  - [When to Use Which Mode?](#when-to-use-which-mode)
  - [Turbo Streams makes WebSockets feel Easy:](#turbo-streams-makes-websockets-feel-easy)
- [3.5 Turbo Frames vs Turbo Streams: When to Use Which?](#35-turbo-frames-vs-turbo-streams-when-to-use-which)
  - [Critical Distinction: Frames vs Streams (Don't Mix Them Up!)](#critical-distinction-frames-vs-streams-dont-mix-them-up)
- [4. Stimulus: The JavaScript Sprinkles for SSR](#4-stimulus-the-javascript-sprinkles-for-ssr)
  - [The Problem Stimulus Solves](#the-problem-stimulus-solves)
  - [Enter Stimulus: HTML-First JavaScript](#enter-stimulus-html-first-javascript)
  - [The Bridge Concept](#the-bridge-concept)
  - [The Three Core Concepts](#the-three-core-concepts)
  - [The Underlying Power: DOM Events](#the-underlying-power-dom-events)
  - [Real-World Patterns](#real-world-patterns)
  - [The Magic: Stimulus Works With Turbo](#the-magic-stimulus-works-with-turbo)
  - [Stimulus vs React/Vue: The Philosophical Divide](#stimulus-vs-reactvue-the-philosophical-divide)
  - [The Complete Picture](#the-complete-picture)
  - [Working with External Resources](#working-with-external-resources)
  - [When to Use Each Tool for Loading Content](#when-to-use-each-tool-for-loading-content)
  - [The Key Philosophy](#the-key-philosophy)
  - [Where JavaScript Frameworks Actually Win](#where-javascript-frameworks-actually-win)
- [Why People Don't Realize Hotwire's Power](#why-people-dont-realize-hotwires-power)
- [Case Study: Campfire by 37Signals](#case-study-campfire-by-37signals)
  - [1. Why does "HTML over the wire" matter?](#1-why-does-html-over-the-wire-matter)
  - [2. Why is DOM-as-state-container revolutionary?](#2-why-is-dom-as-state-container-revolutionary)
  - [3. What does "locality of behavior" mean?](#3-what-does-locality-of-behavior-mean)
  - [4. Why do Turbo Frames have `src`?](#4-why-do-turbo-frames-have-src)
  - [The Revolutionary Realizations from Campfire](#the-revolutionary-realizations-from-campfire)
  - [Why This All Matters](#why-this-all-matters)

## 1. The Fundamental Insight

Before Hotwire, web developers were stuck in a false dichotomy:

1. Traditional SSR (Ruby on Rails, Django, PHP):
  - Server sends full HTML pages
  - Every click = white flash + lost scroll position
  - Felt "old" and "slow"

2. SPA Frameworks (React, Vue, Angular):
  - Client renders everything with JavaScript
  - Smooth interactions but...
  - Massive complexity (Redux, webpack, hydration issues)
  - Poor SEO, slow initial loads
  - Two codebases (API + frontend)

They thought:
- **Want partial updates?** → Must use JavaScript/AJAX
- **Want server-side simplicity?** → Accept full page reloads

**JavaScript frameworks assumed SSR was dead, so they rebuilt everything client-side.**

**Hotwire proves SSR was never the problem - the lack of interactivity was!**

By adding:
- Smooth navigation (Turbo Drive)
- Partial updates (Turbo Frames)  
- Real-time capability (Turbo Streams)
- Sprinkles of interactivity (Stimulus)

**You get SPA-like experience with SSR simplicity!**

### DHH's Revolutionary Insight

DHH realized everyone was solving the wrong problem. The issue wasn't that servers were sending HTML - it was that browsers were
tearing down the entire page to display it!

**DHH's breakthrough:** "What if we just... sent HTML instead of JSON?"

#### HTML Is Data, Not Just Presentation

**The fundamental realization:** HTML is already a data format - it's structured, tagged text that describes content.

**Traditional thinking:**
- Server: Database → JSON (data)
- Client: JSON → HTML (presentation)

**DHH's insight:**
- Server: Database → HTML (data AND presentation)
- Client: Just display it

#### The Round Trip Is The Real Bottleneck

**What's actually expensive:**
- Network round trip: ~270ms (DNS, TCP, SSL, latency)
- Payload difference: JSON (0.5KB) vs HTML (1KB) = ~1ms

**The false optimization:** Everyone optimized payload size when the round trip was 270x more expensive.

#### Information Apps vs Interaction Apps: The Core Distinction

**Information Apps (99% of web apps):**
- GitHub, Basecamp, Shopify, Airbnb
- Display and manipulate server data
- Perfect for HTML over the wire

**Interaction Apps (1% of web apps):**
- Figma, Google Docs, Games
- Heavy client-side computation
- Need JavaScript frameworks

**DHH's insight:** Most web apps are information apps, not interaction apps. The SPA revolution solved a problem that didn't exist for 99% of applications.

### The Three "Impossible" Things SSR Couldn't Do (Pre-Hotwire)

1. **Partial page updates** → Solved by Turbo Frames
2. **Smooth page transitions** → Solved by Turbo Drive
3. **Real-time updates** → Solved by Turbo Streams

Hotwire toolkit for web development has 4 key tools:

1. Turbo Drive = "Keep the page alive during navigation"
2. Turbo Frames = "Only swap the parts that changed"
3. Turbo Streams = "Surgical DOM updates from the server"
4. Stimulus = "JavaScript sprinkles that survive DOM swaps"

## The Turbo Decision Framework (DHH's Rails World 2023)

DHH presented this as a **graph with two axes** to help decide which Turbo tool to use:

**X-axis: Developer Happiness (Ease of Use)**
**Y-axis: Responsiveness (Native Feel)**

```
Responsiveness
       ↑
   High|  Turbo Stream
       |  Actions
       |
       |           Turbo
       |           Frames
       |
       |                    [Turbo 8 Morphing]
       |                    (NEW!)
       |
       |                    Turbo Drive
    Low|                    (Full <body>)
       |________________________→
        Hard              Easy

               Developer Happiness
```

**The Tradeoff:**
- **Turbo Drive** (bottom right): Maximum ease (zero setup), moderate responsiveness
- **Turbo Frames** (middle): Medium effort, good responsiveness
- **Turbo Streams** (top left): Most setup required, maximum responsiveness
- **NEW: Turbo 8 Morphing** (top right): Turbo Drive ease + Turbo Streams responsiveness

**Decision Framework:**
1. **Start with Turbo Drive** - Works everywhere, zero setup
2. **Upgrade to Turbo Frames** - When you need partial updates with navigation boundaries
3. **Add Turbo Streams** - When you need surgical DOM updates or real-time features
4. **Use Turbo Morphing** - When you want responsiveness without complexity

## Turbo Morphing (The Game Changer)

### The Core Problem: Lost Screen State

**Traditional Turbo Drive limitations:**
When you submit a form and get redirected back (a "page refresh"):
- Entire `<body>` gets replaced
- Scroll position resets to top
- Text selection is lost
- Form focus disappears
- CSS transitions restart
- Open dropdowns close

**Example:** You're halfway down a long task list, select some text to copy, then check a checkbox. The page refreshes, you lose your scroll position, lose the text selection, and have to find where you were. Frustrating!

### The Solution: Morphing

**Turbo 8 Morphing** uses intelligent DOM diffing to:
- Calculate the difference between current and new HTML
- Update ONLY the elements that actually changed
- Preserve everything else (scroll, selection, focus)
- **All with just 2 lines of configuration!**

### How to Enable Morphing

```erb
<!-- In your layout or specific pages -->
<%= turbo_refreshes_with method: :morph, scroll: :preserve %>

<!-- Or manually in HTML -->
<head>
  <meta name="turbo-refresh-method" content="morph">
  <meta name="turbo-refresh-scroll" content="preserve">
</head>
```

That's it! Your app now feels native-like.

### Scenario 1: Form Submissions (Same User Updates)

Jorge's task manager demo - checking off tasks without losing position:

**Without Morphing:**
```ruby
# Controller
def update
  @task.update(task_params)
  redirect_to project_path(@project)  # Page jumps to top!
end
```
Result: Screen flashes, scroll lost, selection lost

**With Morphing (exact same controller!):**
```ruby
# Controller - NO CHANGES!
def update
  @task.update(task_params)
  redirect_to project_path(@project)  # Smooth update!
end
```
Result: Only the task row updates, everything else preserved

**What's happening:**
1. Turbo detects a "page refresh" (redirecting to current URL)
2. Fetches the new HTML
3. Diffs it against current DOM
4. Surgically updates only what changed

### Scenario 2: Broadcasting (Other Users' Updates)

Jorge's collaborative demo - changes on one screen update another:

**The Old Way - Complex Turbo Streams:**
```ruby
# Model - had to specify every update
after_update_commit do
  broadcast_replace_to project, target: "task_#{id}"
  broadcast_replace_to project, target: dom_id(project, :progress)
  broadcast_update_to project, target: "complete_count"
  # Easy to forget regions!
end

# Different templates for each action
# create.turbo_stream.erb
# update.turbo_stream.erb
# destroy.turbo_stream.erb
```

**The New Way - Broadcasting with Morphing:**
```ruby
# Model
class Project < ApplicationRecord
  broadcasts_refreshes  # That's it!
end

class Task < ApplicationRecord
  belongs_to :project, touch: true  # Triggers project refresh
end

# View - subscribe to updates
<%= turbo_stream_from @project %>
```

Now when ANYONE updates a task:
1. Task touches its project (updates timestamp)
2. Project broadcasts a refresh signal
3. All subscribed clients morph their page
4. Everyone sees the update with their scroll preserved!

### The Magic: One Signal Updates Everything

**Before morphing:** You had to manually track every region:
- Update the task row ✓
- Update the progress bar ✓
- Update the counter ✓
- Update the delete button state ✓
- Oh wait, forgot the header badge... 🐛

**With morphing:** One refresh signal updates ALL regions automatically:
```html
<!-- Jorge's demo - ALL these update automatically -->
<div class="progress-bar">75%</div>     <!-- Updates ✓ -->
<span class="task-count">3 remaining</span>  <!-- Updates ✓ -->
<tr class="task completed">...</tr>          <!-- Updates ✓ -->
<button data-confirm="...">Delete</button>   <!-- Updates ✓ -->
```

### Excluding Regions from Morphing

Sometimes you want to keep certain elements untouched:

```html
<!-- This modal stays open during morphing -->
<div data-turbo-permanent id="upload-modal">
  <!-- Content here won't be morphed -->
</div>
```

### Turbo Frames with Morphing

For frames that load additional content (like pagination):

```erb
<turbo-frame id="comments" refresh="morph" src="/comments?page=2">
  <!-- Frame content morphs instead of replaces -->
</turbo-frame>
```

### When Screen State is Preserved

**Preserved during morphing:**
- Vertical and horizontal scroll position
- Text selection
- Form field focus and cursor position
- Textarea content being typed (if unchanged by server)
- CSS animations/transitions in progress
- Elements marked with `data-turbo-permanent`

**Still replaced (intentionally):**
- The actual content that changed
- CSRF tokens (security)
- Any server-side updates

### Where Morphing Fits in the Responsiveness Spectrum

```
Responsiveness
       ↑
   High|  Turbo Stream Actions
       |  (Manual but perfect control)
       |
       |           Turbo Frames
       |           (Scoped updates)
       |
       |                    Turbo Drive + Morphing
       |                    (NEW! Automatic precision)
       |
       |                    Turbo Drive
    Low|                    (Full <body> replace)
       |________________________→
        Hard              Easy
               Developer Happiness
```

**The key insight:** Morphing gives you Turbo Streams-level responsiveness with Turbo Drive-level simplicity!

### The Conceptual Compression

This is what DHH calls "conceptual compression" - complex becomes simple:

**Without morphing:** 100+ lines of stream templates
**With morphing:** 1 line (`broadcasts_refreshes`)

You no longer think about:
- Which regions need updating
- Creating multiple turbo_stream templates
- Coordinating updates across different actions
- Missing an update region and causing bugs

**The breakthrough:** You can build Notion-quality UX with redirect_to!

## 2. Turbo Frames: The "Impossible" Partial Update

### The Old World Problem

You have an email inbox. User clicks "Archive" on one email. What happens?

**Traditional SSR:**
```erb
# Click archive → Full page reload
# Server re-renders ENTIRE page including:
# - Navigation bar (unchanged)
# - Sidebar (unchanged)
# - Email list (only 1 email changed)
# - Footer (unchanged)
```
Result: Screen flashes white, scroll position lost, slow feeling.

**JavaScript Solution:**
```javascript
// Intercept click, prevent default
$('#archive-btn').click(function(e) {
  e.preventDefault()
  $.post('/emails/123/archive', function(response) {
    $('#email-123').fadeOut()
  })
})
```
Result: Works, but now you're writing JavaScript for every interaction.

### The Turbo Frame Solution

```erb
<!-- Each email is a frame -->
<turbo-frame id="email_123">
  <div class="email">
    <h3>Important Meeting</h3>
    <%= button_to "Archive", archive_email_path(email), method: :post %>
  </div>
</turbo-frame>
```

When user clicks "Archive":
1. Browser intercepts the click (Turbo's JavaScript does this automatically)
2. Makes request to server
3. Server responds with JUST the frame content
4. Browser swaps ONLY that frame

**The magic:** Server-side rendering, but only the part that changed!

#### The REAL Performance Bottleneck (The Biggest Aha!)

Everyone thought the problem was network/payload size. **WRONG!**

**What We Thought Was Slow**
- "Sending too much HTML over the wire"
- "Need smaller JSON payloads"

**What's ACTUALLY Slow (The Real Bottleneck)**
```
Browser page teardown → 200ms
DOM reconstruction → 150ms
CSS recalculation → 100ms
JavaScript re-execution → 100ms
Layout reflow → 50ms
= 600ms of BROWSER WORK
```

**The Numbers That Matter**
```
Traditional SSR:
- Network (50KB HTML): 30ms
- Browser teardown/rebuild: 600ms
- Total: 630ms + WHITE FLASH

Turbo Frames (even sending full page):
- Network (50KB HTML): 30ms
- Extract & swap frame: 30ms
- Total: 60ms, NO FLASH

Turbo Frames (optimized, just chunk):
- Network (2KB HTML): 5ms
- Swap frame: 30ms
- Total: 35ms, NO FLASH
```

**The revelation:** Even sending "too much" HTML with Turbo is 10x faster than traditional SSR because we avoid the teardown/rebuild cycle!

**The network was never the problem. The browser teardown was.**

### Turbo Streams: Always Sends HTML Chunks
- Server sends targeted HTML fragments
- Used for real-time updates, form responses
- Example: `turbo_stream.append "messages", partial: "message"`

### Turbo Frames: Can Send Full Page OR Chunks
- Server CAN send full page (Turbo extracts the frame it needs)
- Server CAN optimize and send just the frame
- Both work! The key is: **no page teardown/rebuild**

The revolutionary insight wasn't just about chunk size - it was about keeping the browser page alive!

### Turbo Frames: Parallelization & Lazy Loading

#### The Problem: Sequential Page Generation

Traditional Rails loads everything sequentially:

```ruby
def dashboard
  @user = User.find(params[:id])          # 50ms
  @notifications = fetch_notifications     # 100ms
  @activities = fetch_activities          # 150ms
  @stats = calculate_stats                # 100ms
  # Total: 400ms before user sees ANYTHING
end
```

#### The Solution: Parallel Frame Loading

With Turbo Frames, each segment loads independently:

```erb
<!-- Main page loads in 50ms with skeleton -->
<div class="dashboard">
  <h1>Dashboard</h1>
  
  <!-- These all load in PARALLEL -->
  <turbo-frame id="notifications" src="/dashboard/notifications">
    <div class="skeleton">Loading notifications...</div>
  </turbo-frame>
  
  <turbo-frame id="activity" src="/dashboard/activity">
    <div class="skeleton">Loading activity...</div>
  </turbo-frame>
  
  <turbo-frame id="stats" src="/dashboard/stats">
    <div class="skeleton">Loading stats...</div>
  </turbo-frame>
</div>
```

**Timeline comparison:**
```
Traditional: [----User----][----Notifs----][----Activities----][----Stats----]
             0            50              150                  300           400ms

Turbo Frames: [----User----]
              [----Notifs----]     (parallel)
              [----Activities----] (parallel)  
              [----Stats----]       (parallel)
              0            50                          150ms total!
```

#### Lazy Loading with `loading` Attribute

```erb
<!-- Loads immediately when page loads -->
<turbo-frame id="urgent" src="/urgent" loading="eager">
  <div>Loading urgent items...</div>
</turbo-frame>

<!-- Loads when about to enter viewport (like scrolling down) -->
<turbo-frame id="archived" src="/archived" loading="lazy">
  <div>Archived items will load when you scroll here</div>
</turbo-frame>
```

#### Skeleton UI in Rails (Yes, It's Possible!)

You don't need Next.js for skeleton UI! Here's how to do it in Rails:

```erb
<!-- The skeleton shows immediately while content loads -->
<turbo-frame id="revenue-chart" src="/charts/revenue" loading="lazy">
  <!-- This skeleton UI shows immediately -->
  <div class="animate-pulse">
    <div class="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
    <div class="h-4 bg-gray-200 rounded w-1/2 mb-4"></div>
    <div class="flex space-x-2">
      <div class="h-20 bg-gray-200 rounded w-1/6"></div>
      <div class="h-24 bg-gray-200 rounded w-1/6"></div>
      <div class="h-16 bg-gray-200 rounded w-1/6"></div>
      <div class="h-28 bg-gray-200 rounded w-1/6"></div>
      <div class="h-20 bg-gray-200 rounded w-1/6"></div>
      <div class="h-32 bg-gray-200 rounded w-1/6"></div>
    </div>
  </div>
</turbo-frame>

<!-- For a more realistic skeleton -->
<turbo-frame id="email-list" src="/emails">
  <div class="space-y-3">
    <% 5.times do %>
      <div class="border rounded p-4 animate-pulse">
        <div class="flex items-center space-x-3">
          <div class="rounded-full bg-gray-200 h-10 w-10"></div>
          <div class="flex-1">
            <div class="h-4 bg-gray-200 rounded w-1/4 mb-2"></div>
            <div class="h-3 bg-gray-200 rounded w-3/4"></div>
          </div>
        </div>
      </div>
    <% end %>
  </div>
</turbo-frame>
```

#### Tailwind CSS Skeleton Animation

```css
/* This is built into Tailwind! */
.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: .5; }
}
```

##### When to Use Each Pattern

**Use Eager Loading (`loading="eager"` or no attribute) for:**
- Above-the-fold content
- Critical user information
- Primary navigation
- Anything needed for SEO

**Use Lazy Loading (`loading="lazy"`) for:**
- Below-the-fold content
- Expensive computations (charts, analytics)
- Optional features (comments, related items)
- Archive/historical data
- Heavy images or media galleries

**Use Skeleton UI for:**
- Any frame that takes >100ms to load
- Lists of items (emails, posts, products)
- Data visualizations
- Dashboard widgets
- User-generated content

#### Real World Example: Email Dashboard

```erb
<div class="email-dashboard">
  <!-- Critical: Load immediately -->
  <div class="header">
    <h1>Inbox</h1>
    <span><%= current_user.unread_count %> unread</span>
  </div>
  
  <!-- Important: Load eagerly with skeleton -->
  <turbo-frame id="urgent-emails" src="/emails/urgent" loading="eager">
    <%= render "emails/skeleton", count: 3 %>
  </turbo-frame>
  
  <!-- Less important: Load lazily -->
  <turbo-frame id="promotional" src="/emails/promotional" loading="lazy">
    <div class="text-gray-500">
      Promotional emails will load when you scroll down
    </div>
  </turbo-frame>
</div>
```

#### The Performance Win

- **First Contentful Paint:** 50ms (vs 400ms traditional)
- **Time to Interactive:** 50ms (vs 400ms traditional)  
- **Complete page load:** 150ms (vs 400ms traditional)
- **Perceived performance:** Instant (users see skeletons immediately)

**The secret:** Rails can handle frame requests across multiple Puma workers/threads in parallel, while traditional rendering is sequential.

#### Pro Tips

1. **Always provide meaningful placeholders** - Not just "Loading..."
2. **Match skeleton dimensions** to real content to avoid layout shift
3. **Use `loading="lazy"` aggressively** for below-fold content
4. **Combine with Turbo Streams** for real-time updates after initial load
5. **Set up different endpoints** for each frame to maximize parallelization


### Frames with `src`: Composable, Parallel-Loading Pages

When a frame has a `src`, it automatically fetches that URL and extracts the matching frame:

```erb
<!-- This frame loads content from /emails/set_aside automatically -->
<turbo-frame id="set_aside_tray" src="/emails/set_aside">
  <!-- Optional: Show something while loading -->
  <img src="/icons/spinner.gif">
</turbo-frame>
```

#### How It Works Step-by-Step

##### 1. Initial Page Load
```erb
<!-- /inbox renders -->
<body>
  <h1>Inbox</h1>
  
  <div id="emails">
    <!-- Main emails load immediately with the page -->
    <%= render @emails %>
  </div>
  
  <!-- These frames appear but trigger background requests -->
  <turbo-frame id="set_aside_tray" src="/emails/set_aside">
    <div class="spinner">Loading...</div>
  </turbo-frame>
  
  <turbo-frame id="reply_later_tray" src="/emails/reply_later">
    <div class="spinner">Loading...</div>
  </turbo-frame>
</body>
```

##### 2. Automatic Parallel Requests
As soon as the page loads, Turbo automatically makes parallel requests:
```
GET /emails/set_aside     → For first frame
GET /emails/reply_later   → For second frame
```

##### 3. The Response
```erb
<!-- /emails/set_aside returns this FULL PAGE -->
<body>
  <h1>Set Aside Emails</h1>
  <p>These are emails you've set aside</p>
  
  <turbo-frame id="set_aside_tray">
    <!-- Only THIS part gets extracted and used -->
    <div id="emails">
      <div id="email_1">
        <a href="/emails/1">My important email</a>
      </div>
    </div>
  </turbo-frame>
</body>
```

##### 4. Turbo Extracts and Replaces
Turbo finds `<turbo-frame id="set_aside_tray">` in the response and replaces just that frame's content!

#### The Brilliant Part: Dual-Purpose Pages

The same URL works in TWO ways:

**As a Frame (embedded in inbox):**
```
User visits: /inbox
Sees: Just the emails inside the frame
```

**As a Standalone Page:**
```
User visits: /emails/set_aside directly
Sees: Full page with header, description, and emails
```

#### Real-World Dashboard Example

```erb
<!-- app/views/dashboard/show.html.erb -->
<div class="dashboard">
  <h1>Dashboard</h1>
  
  <!-- Main content loads immediately -->
  <div class="stats">
    <%= @quick_stats %>  <!-- 50ms -->
  </div>
  
  <!-- These load in parallel after page loads -->
  <turbo-frame id="recent_activity" src="/dashboard/activity">
    <%= render "shared/skeleton_list" %>
  </turbo-frame>
  
  <turbo-frame id="notifications" src="/dashboard/notifications">
    <div class="text-gray-400">Loading notifications...</div>
  </turbo-frame>
  
  <turbo-frame id="analytics" src="/dashboard/analytics" loading="lazy">
    <!-- This one only loads when scrolled into view -->
    <div class="chart-skeleton animate-pulse"></div>
  </turbo-frame>
</div>
```

```erb
<!-- app/views/dashboard/activity.html.erb -->
<!-- This page works standalone AND as a frame source -->
<% if turbo_frame_request? %>
  <!-- Minimal response for frame -->
  <turbo-frame id="recent_activity">
    <%= render @activities %>
  </turbo-frame>
<% else %>
  <!-- Full page when accessed directly -->
  <div class="container">
    <h1>Recent Activity</h1>
    <p>Your activity from the last 30 days</p>
    
    <turbo-frame id="recent_activity">
      <%= render @activities %>
    </turbo-frame>
  </div>
<% end %>
```

#### The Performance Timeline

**Without frames (sequential):**
```
1. Request /dashboard
2. Query ALL data (stats + activity + notifications + analytics)
3. Render everything
4. Send response
Total: 400ms before user sees ANYTHING
```

**With frames (parallel):**
```
1. Request /dashboard (50ms - just quick stats)
2. Page visible! User sees structure
3. Parallel background requests:
   - /dashboard/activity (100ms)
   - /dashboard/notifications (50ms)
   - /dashboard/analytics (when scrolled)
4. Frames populate as responses arrive
Total: 50ms to first meaningful paint!
```

#### Key Patterns

##### Pattern 1: Spinner/Skeleton
```erb
<turbo-frame id="data" src="/expensive/data">
  <!-- Shows while loading -->
  <div class="animate-pulse">
    <div class="h-4 bg-gray-200 rounded"></div>
  </div>
</turbo-frame>
```

##### Pattern 2: Silent Loading
```erb
<!-- Loads in background without placeholder -->
<turbo-frame id="optional" src="/optional/feature"></turbo-frame>
```

##### Pattern 3: Lazy Load Below Fold
```erb
<turbo-frame id="comments" src="/comments" loading="lazy">
  <!-- Only loads when user scrolls down -->
  <p>Comments will load when you scroll here</p>
</turbo-frame>
```

##### Pattern 4: Graceful Degradation
```erb
<turbo-frame id="weather" src="/external/weather">
  <div>Weather widget loading...</div>
  <!-- If request fails, user still sees placeholder -->
</turbo-frame>
```

#### The "Composable Pages" Architecture

Each section of your page becomes:

1. **Independently loadable** - Has its own URL
2. **Standalone viewable** - Can be accessed directly
3. **Embeddable** - Can be included in any page via frames
4. **Separately cacheable** - Different cache strategies per section
5. **Failure resilient** - One section fails, others still work
6. **Progressively enhanced** - Page usable before all sections load

#### Performance Best Practices

1. **Load critical content immediately** (no `src`, render with page)
2. **Use `src` for secondary content** (activity feeds, recommendations)
3. **Use `loading="lazy"` for below-fold** (comments, related items)
4. **Provide meaningful placeholders** (skeletons, not just "Loading...")
5. **Design URLs to work standalone** (dual-purpose pages)

#### Example: Email App Architecture

```erb
<!-- Inbox page composes multiple independent sections -->
<div class="inbox">
  <!-- Critical: Render immediately -->
  <div id="urgent">
    <%= render @urgent_emails %>
  </div>
  
  <!-- Important: Load eagerly in background -->
  <turbo-frame id="today" src="/emails/today">
    <%= render "skeleton" %>
  </turbo-frame>
  
  <!-- Secondary: Load eagerly but lower priority -->
  <turbo-frame id="newsletters" src="/emails/newsletters">
    <p class="text-gray-400">Loading newsletters...</p>
  </turbo-frame>
  
  <!-- Optional: Only load if user scrolls -->
  <turbo-frame id="archived" src="/emails/archived" loading="lazy">
    <p>Scroll down to see archived emails</p>
  </turbo-frame>
</div>
```

Each section (`/emails/today`, `/emails/newsletters`, etc.) is a fully functional page on its own, but also works embedded in the inbox. This is **architectural gold** - modular, cacheable, resilient!

##### The Mental Model

Think of frames with `src` as **\<iframe\>s done right**:
- No security sandbox issues
- No style isolation problems  
- Automatic integration with your app
- Progressive enhancement built-in
- Perfect for micro-frontends in a monolith

### Aside: Why Your Hotwire App Might Feel Slow Even With Turbo Frames

#### 1. Turbo Frame Boundaries Too Large

```erb
<!-- BAD: Frame includes too much -->
<turbo-frame id="entire-dashboard">
  <!-- 500 lines of HTML -->
</turbo-frame>

<!-- GOOD: Granular frames -->
<turbo-frame id="user-stats">...</turbo-frame>
<turbo-frame id="recent-activity">...</turbo-frame>
<turbo-frame id="notifications">...</turbo-frame>
```

**Why it matters:** Large frames = more HTML to swap = slower updates

#### 2. Missing Lazy Loading

```erb
<!-- BAD: Loading everything upfront -->
<%= render @emails %>  <!-- 1000 emails -->

<!-- GOOD: Lazy load with frames -->
<turbo-frame id="emails" src="/emails" loading="lazy">
  <div>Loading emails...</div>
</turbo-frame>
```

#### 3. Not Using Turbo Streams for Lists

```erb
<!-- BAD: Replacing entire list for one new item -->
<turbo-frame id="emails">
  <%= render @emails %> <!-- Re-renders ALL emails -->
</turbo-frame>

<!-- GOOD: Surgical updates with Streams -->
<%= turbo_stream.prepend "emails", @new_email %>
```

#### 4. Database Queries in Frame Updates

```ruby
# BAD: N+1 queries in partial updates
def update
  @email = Email.find(params[:id])
  @email.update(email_params)
  # This partial triggers tons of queries
  render partial: "email_with_all_associations"
end

# GOOD: Preload what you need
def update
  @email = Email.includes(:sender, :labels).find(params[:id])
  # ...
end
```

## 3. Turbo Streams Deep Dive: Two Modes, One Format

### Mode 1: HTTP Response (Form Submissions)
When YOU do something, server responds with updates just for YOU:

```ruby
# Controller
def create
  @message = Message.create(message_params)
  
  respond_to do |format|
    format.turbo_stream {
      # Renders create.turbo_stream.erb
      # Updates YOUR browser only
    }
  end
end
```

```erb
<!-- create.turbo_stream.erb -->
<%= turbo_stream.update "flash", partial: "layouts/flash" %>
<%= turbo_stream.append "messages", @message %>
<%= turbo_stream.update "message_count", Message.count %>
```

**Use Mode 1 when:**
- Responding to form submissions
- Updating after user actions
- Showing validation errors
- Only the acting user needs to see updates

### Mode 2: WebSocket Broadcasting (Real-time)
When ANYONE does something, server broadcasts to EVERYONE:

```erb
<!-- View: Subscribe to updates -->
<%= turbo_stream_from "chat_room_123" %>
<div id="messages"></div>
```

```ruby
# Model: Broadcast to all subscribers
class Message < ApplicationRecord
  after_create_commit do
    broadcast_append_to "chat_room_123",
                       target: "messages",
                       partial: "messages/message",
                       locals: { message: self }
  end
end
```

**Use Mode 2 when:**
- Chat/messaging systems
- Live notifications
- Collaborative editing
- Real-time dashboards
- Activity feeds
- Any multi-user updates

### The Golden Rules of Turbo Stream Targeting

**Rule 1: You can target ANY element with an ID**
```html
<!-- All of these are valid targets -->
<div id="sidebar">...</div>
<span id="count">...</span>
<ul id="list">...</ul>
<literally-any-tag id="anything">...</literally-any-tag>
```

**Rule 2: You DON'T need special markup**
```erb
<!-- This is enough -->
<div id="notifications"></div>

<!-- Now you can do ALL of these -->
<%= turbo_stream.append "notifications", "<div>New!</div>" %>
<%= turbo_stream.update "notifications", "Empty" %>
<%= turbo_stream.remove "notifications" %>
```

**Rule 3: Different actions have different requirements**
- `append`/`prepend`: Target must be a container (div, ul, etc.)
- `replace`: Replacement must include the same ID
- `update`: Only changes inner content
- `remove`: Element just needs to exist
- `before`/`after`: Inserts sibling elements

### What You CAN'T Do
```erb
<!-- CAN'T target without IDs -->
<div class="messages"></div>  <!-- No ID = no targeting -->

<!-- CAN'T target multiple elements with one action -->
<%= turbo_stream.update ".status", "Active" %>  <!-- Needs single ID -->

<!-- CAN'T target non-existent elements (except remove) -->
<%= turbo_stream.append "nonexistent", "content" %>  <!-- Parent must exist -->
```

### Broadcasting Patterns

**Scoped Broadcasting (specific users):**
```ruby
# Only broadcast to specific account
broadcast_append_to [account, :messages], target: "messages"
```

**Conditional Broadcasting:**
```ruby
after_update_commit do
  broadcast_replace_to self if saved_change_to_status?
end
```

**Multiple Updates:**
```ruby
after_create_commit do
  broadcast_append_to "feed", target: "activities"
  broadcast_update_to "feed", target: "count", html: Activity.count
end
```

### The Implementation Magic

```ruby
# You write:
broadcast_append_to "room", target: "messages", html: "<div>Hi!</div>"

# Rails automatically:
# 1. Finds all WebSocket connections subscribed to "room"
# 2. Wraps HTML in Turbo Stream format
# 3. Sends through WebSocket to all subscribers
# 4. Turbo's JavaScript updates the DOM
# You never touch WebSockets directly!
```

### When to Use Which Mode?

**Mode 1 (HTTP Response):**
- Form validation errors ✓
- Success messages ✓
- Update after user action ✓
- Shopping cart updates ✓

**Mode 2 (WebSocket Broadcasting):**
- New chat message arrives ✓
- Someone else updates shared document ✓
- Real-time notification ✓
- Live visitor counter ✓

**Both modes use the SAME Turbo Stream format** - just different delivery methods!

### Turbo Streams makes WebSockets feel Easy:

#### The JavaScript WebSocket Horror Story (Why It Seemed Hard But Isn't)

In JavaScript land, you need ALL of this for basic chat:

```javascript
// 1. Choose and install libraries (2.5MB of dependencies)
npm install socket.io socket.io-client

// 2. Server setup
const io = require('socket.io')(server)
io.on('connection', (socket) => {
  socket.on('join-room', (roomId) => socket.join(roomId))
  socket.on('message', (data) => {
    io.to(data.roomId).emit('new-message', data)
  })
  socket.on('disconnect', () => { /* cleanup */ })
})

// 3. Client complexity
const socket = io()
socket.on('connect', () => { /* handle connection */ })
socket.on('reconnect', () => { /* handle reconnection */ })
socket.on('new-message', (data) => {
  // Manually build and insert HTML
  // Handle XSS vulnerabilities
  // Manage state synchronization
})

// 4. Deal with edge cases
// - Connection management
// - Message queuing
// - Heartbeat/ping-pong
// - Duplicate prevention
// - Error recovery
// ... 100s more lines
```

#### The Hotwire Turbo Streams Way

```ruby
# Model
after_create_commit -> { 
  broadcast_append_to chat, target: "messages" 
}
```

```erb
<!-- View -->
<%= turbo_stream_from @chat %>
<div id="messages"></div>
```

**That's it.** Rails handles everything else.

#### What WebSockets ACTUALLY Are

Just a persistent connection that stays open:

**Regular HTTP (like phone calls):**
```
Client: "Any messages?" → Server: "No" → *hangs up*
Client: "Any messages?" → Server: "No" → *hangs up*
Client: "Any messages?" → Server: "Yes!" → *hangs up*
```

**WebSocket (like keeping phone line open):**
```
Client: "Let's stay connected" → Server: "OK!"
*line stays open*
Server: "New message!" → Client: "Got it!"
Server: "Another one!" → Client: "Got it!"
```

#### Why JavaScript Made It Seem Hard

1. **They built everything from scratch** - connection pooling, auth, routing, errors
2. **Marketing hype** - Companies selling WebSocket services made it seem complex
3. **Complexity addiction** - JS ecosystem loves dependencies and abstractions

#### Examples of WebSockets Usage

- **Slack/Discord:** Real-time messages
- **Google Docs:** Collaborative editing
- **Stock tickers:** Live price updates
- **Notifications:** Instant delivery

#### The Rails Magic: Conceptual Compression

Rails hides the complexity behind beautiful APIs:

```ruby
# You write (thinking about your domain):
broadcast_append_to chat, target: "messages"

# Rails handles (all the plumbing):
# - Connection management
# - Authentication
# - Reconnection logic
# - Message routing
# - Error handling
# - Cleanup on disconnect
# - Security (CSRF, origin checking)
```

#### The Aha Moment

**WebSockets seemed hard because JavaScript developers had to build everything.**
**Rails makes it easy by handling the 95% use case perfectly.**

You learned in 10 minutes what takes weeks in JavaScript land. That's the power of Rails' philosophy: make the common cases trivial, and the complex cases possible.

## 3.5 Turbo Frames vs Turbo Streams: When to Use Which?

This is confusing because both can update parts of your page! Here's the key difference:

#### Turbo Frames = Navigation & Context
```erb
<turbo-frame id="modal">
  <!-- Everything inside stays together -->
  <%= link_to "Next Step", step2_path %>  <!-- Stays in frame -->
  <%= form_with model: @user %>  <!-- Submits in frame -->
</turbo-frame>
```

**Characteristics:**
- Creates a **boundary** (like an iframe)
- Links/forms inside **automatically** stay inside
- Good for: modals, tabs, wizards, multi-step forms
- **Lazy loading** built-in (`loading="lazy"`)
- One frame = one update

#### Turbo Streams = Surgical Updates
```erb
<%= turbo_stream.append "messages", @message %>
<%= turbo_stream.update "count", @messages.count %>
<%= turbo_stream.remove "notification_#{@notification.id}" %>
```

**Characteristics:**
- **Multiple** DOM updates from one response
- **Precise** targeting (append, prepend, before, after, replace, update, remove)
- Good for: real-time updates, form responses, notifications
- **No boundaries** - can update anywhere on page
- Can be delivered via HTTP response OR WebSocket

#### The Key Difference Example

**Scenario: Edit a comment in a thread**

**With Turbo Frame:**
```erb
<turbo-frame id="comment_123">
  <div class="comment">
    <%= @comment.body %>
    <%= link_to "Edit", edit_comment_path(@comment) %>
  </div>
</turbo-frame>
```
Click "Edit" → Frame replaced with form → Submit → Frame replaced with updated comment

✅ **Pros:** Everything contained, automatic handling
❌ **Cons:** Can only update this one frame

**With Turbo Streams:**
```erb
# After form submission
<%= turbo_stream.replace "comment_123", @comment %>
<%= turbo_stream.update "comment_count", @post.comments.count %>
<%= turbo_stream.prepend "activity_log", partial: "activity", 
    locals: { message: "Comment updated" } %>
```

✅ **Pros:** Can update multiple places at once
❌ **Cons:** Need to manually handle form submission, no automatic boundaries

#### Decision Framework

**Use Turbo Frames when:**
- You want a section to be **independent**
- Navigation should stay **within boundaries**
- You need **lazy loading**
- It's a **contained workflow** (modal, wizard)
- You want **automatic link/form handling**

**Use Turbo Streams when:**
- You need **multiple updates** from one action
- You want **real-time** updates via WebSockets
- You're doing **surgical DOM manipulation**
- Updates are **scattered** across the page
- You need to **append/prepend** to lists (not replace)

#### Can You Use Turbo Streams Instead of Frames?

YES, technically you could:
```erb
# Instead of Turbo Frame
<%= turbo_stream.replace "modal", partial: "edit_form" %>
```

But you'd lose:
- Automatic link/form interception within boundaries
- Lazy loading capability
- The semantic meaning of "this is a contained unit"
- Built-in navigation context

#### The Mental Model

**Turbo Frames** = "This section is a mini-app within my page"
**Turbo Streams** = "Here are specific instructions for DOM updates"

### Critical Distinction: Frames vs Streams (Don't Mix Them Up!)

#### The Handbook Warning Explained

> "Frames serve a specific purpose: to compartmentalize the content and navigation for a fragment of the document... If your application utilizes `<turbo-frame>` elements for the sake of a `<turbo-stream>` element, change the `<turbo-frame>` into another built-in element."

**Translation:** Don't use Frames as fancy divs. They fundamentally change how links and forms work!

#### The Key Insight: Frames TRAP Navigation

```erb
<!-- OUTSIDE a frame -->
<%= link_to "Profile", profile_path %>  <!-- Navigates whole page -->

<!-- INSIDE a frame -->
<turbo-frame id="sidebar">
  <%= link_to "Profile", profile_path %>  <!-- Only updates the frame! -->
</turbo-frame>
```

**Every link and form inside a frame is TRAPPED unless you explicitly break out with `data-turbo-frame="_top"`**

#### The Common Mistake

**❌ WRONG: Using Frames Just as Stream Targets**

```erb
<!-- DON'T DO THIS -->
<turbo-frame id="notification-area">
  <div id="notifications">
    <!-- Just using frame as a container for streams -->
  </div>
</turbo-frame>

<!-- Controller -->
turbo_stream.append "notifications", partial: "notification"
```

**Why it's wrong:**
- The frame adds NO value here
- You're not using it for navigation compartmentalization
- Any links inside notifications are now TRAPPED in the frame!
- You've introduced a bug, not a feature

**✅ RIGHT: Just Use a Regular Element**

```erb
<!-- DO THIS INSTEAD -->
<div id="notification-area">
  <div id="notifications">
    <!-- Streams can target ANY element with an ID -->
  </div>
</div>

<!-- Controller - exactly the same! -->
turbo_stream.append "notifications", partial: "notification"
```

#### Real Examples: Right vs Wrong

##### Scenario 1: Comment Section

**❌ WRONG - Frame just for updates:**
```erb
<turbo-frame id="comments-section">
  <div id="comments">
    <% @comments.each do |comment| %>
      <%= link_to comment.author, user_path(comment.author) %>
      <!-- BUG: This link is trapped! Can't navigate to user profile -->
    <% end %>
  </div>
</turbo-frame>
```

**✅ RIGHT - Frame for compartmentalized navigation:**
```erb
<turbo-frame id="comments-section">
  <div id="comments">
    <% @comments.each do |comment| %>
      <%= comment.body %>
      <%= link_to "Edit", edit_comment_path(comment) %>
      <!-- GOOD: Edit stays within frame - that's what we want -->
    <% end %>
  </div>
  <%= link_to "Load More", comments_path(page: 2) %>
  <!-- GOOD: Pagination stays within frame -->
</turbo-frame>
```

##### Scenario 2: Live Counter

**❌ WRONG - Unnecessary Frame:**
```erb
<turbo-frame id="user-count">
  <span id="count"><%= User.count %></span>
</turbo-frame>

<!-- Why add a frame? There's no navigation here! -->
```

**✅ RIGHT - Simple element:**
```erb
<span id="user-count"><%= User.count %></span>

<!-- Turbo Streams work perfectly without frames -->
<%= turbo_stream.update "user-count", User.count %>
```

##### Scenario 3: Modal (Perfect Frame Use)

**✅ RIGHT - Frame provides navigation boundary:**
```erb
<turbo-frame id="modal">
  <%= form_with model: @user do |f| %>
    <!-- Form submits WITHIN modal - perfect! -->
  <% end %>
  
  <%= link_to "Terms", terms_path %>
  <!-- Shows terms INSIDE modal - perfect! -->
  
  <%= link_to "Close", "#", data: { turbo_frame: "_top" } %>
  <!-- Explicitly breaks out when needed -->
</turbo-frame>
```

#### Decision Framework

Ask yourself: **"Do I want links/forms in this section to be trapped?"**

- **YES → Use a Turbo Frame** (modals, wizards, tabs, contained sections)
- **NO → Use a regular element with ID** (counters, notifications, real-time updates)

#### What Each Tool Is For

**Turbo Frames Are For:**
- Creating navigation boundaries
- Scoped interactions (modal, wizard, tabs)
- Lazy loading sections with `src`
- Independent page sections with their own navigation

**Turbo Frames Are NOT For:**
- Just being update targets for streams
- Simple real-time updates
- Broadcasting destinations
- Any place where you DON'T want trapped navigation

#### Quick Test

Which should use Frames?

1. Chat message list that receives new messages → **No Frame**
2. Login modal with form and "forgot password" link → **Use Frame**
3. Stock price updating every second → **No Frame**
4. Email inbox with pagination and filters → **Use Frame**
5. Notification count in header → **No Frame**
6. Multi-step wizard for user onboarding → **Use Frame**

#### The Core Realization

**Turbo Streams can target ANY element with an ID. You don't need Frames for Streams!**

```erb
<!-- This works perfectly -->
<div id="anything"><%= @content %></div>

<!-- Stream can target it -->
<%= turbo_stream.update "anything", "New content" %>
```

**Only use Frames when you specifically want the navigation-trapping behavior.**

## 4. Stimulus: The JavaScript Sprinkles for SSR

### The Problem Stimulus Solves

Even with Turbo handling navigation and updates, you still need JavaScript for:
- Dropdown menus
- Modal behaviors
- Keyboard shortcuts
- Drag and drop
- Form validation
- Copy to clipboard
- Tooltips
- Auto-save
- Character counters

The old Rails way was **jQuery soup**:

```javascript
// application.js - The jQuery mess
$(document).ready(function() {
  $('.dropdown-toggle').click(function() {
    $(this).next('.dropdown-menu').toggle()
  })
  
  $('.modal-trigger').click(function() {
    $('#modal').show()
  })
  
  // 500 more lines of spaghetti...
})
```

**Problems:** No organization, breaks with Turbo, global scope pollution, selector-based (fragile)

### Enter Stimulus: HTML-First JavaScript

Instead of JavaScript creating HTML (React), HTML declares what JavaScript to run:

```html
<!-- HTML declares the behavior -->
<div data-controller="dropdown">
  <button data-action="click->dropdown#toggle">Menu</button>
  
  <div data-dropdown-target="menu" class="hidden">
    <a href="/profile">Profile</a>
  </div>
</div>
```

```javascript
// dropdown_controller.js - Reusable, organized
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["menu"]
  
  toggle() {
    this.menuTarget.classList.toggle("hidden")
  }
}
```

### The Bridge Concept

#### Class as a Bridge to CSS
```html
<!-- HTML defines structure -->
<div class="alert danger large">Error message</div>
```

```css
/* CSS targets HTML through class names */
.alert { padding: 10px; }
.danger { background: red; }
.large { font-size: 20px; }
```

**The `class` attribute is the BRIDGE** - HTML and CSS connect through this convention.

#### Stimulus Uses the Same Pattern
```html
<!-- HTML defines structure AND behavior -->
<div data-controller="dropdown" class="menu">
  <button data-action="click->dropdown#toggle">Menu</button>
</div>
```

**The parallel is perfect:**
- `class="menu"` → CSS knows what to style
- `data-controller="dropdown"` → JavaScript knows what to control

### The Three Core Concepts

1. **Controllers** - JavaScript classes that bring behavior
2. **Actions** - Connect DOM events to controller methods
3. **Targets** - Mark important elements for controllers to access

### The Underlying Power: DOM Events

Every browser interaction fires an event. Stimulus can connect to ALL of them:

#### Mouse Events
```html
<div data-controller="mouse">
  <button data-action="click->mouse#handleClick">Click</button>
  <button data-action="dblclick->mouse#handleDouble">Double Click</button>
  <button data-action="mouseenter->mouse#showTooltip">Hover</button>
  <button data-action="mouseleave->mouse#hideTooltip">Leave</button>
  <button data-action="contextmenu->mouse#rightClick">Right-click</button>
  <div data-action="mousemove->mouse#trackMovement">Track mouse</div>
</div>
```

#### Keyboard Events
```html
<div data-controller="keyboard">
  <input data-action="keydown->keyboard#handleKey">
  <input data-action="keyup->keyboard#released">
  
  <!-- Global keyboard shortcuts -->
  <div data-action="keydown@window->keyboard#globalShortcut"></div>
</div>
```

```javascript
handleKey(event) {
  console.log({
    key: event.key,           // "a", "Enter", "ArrowUp"
    shiftKey: event.shiftKey, // Was shift held?
    ctrlKey: event.ctrlKey,   // Was ctrl held?
    metaKey: event.metaKey,   // Was cmd/win held?
  })
}
```

#### Form Events
```html
<form data-controller="form">
  <input data-action="input->form#validate">        <!-- As they type -->
  <input data-action="change->form#save">           <!-- When finished -->
  <input data-action="focus->form#highlight">       <!-- When selected -->
  <input data-action="blur->form#removeHighlight">  <!-- When deselected -->
  <form data-action="submit->form#handleSubmit">    <!-- Form submission -->
</form>
```

#### Window/Document Events
```html
<div data-controller="app">
  <div data-action="resize@window->app#handleResize"></div>
  <div data-action="scroll@window->app#handleScroll"></div>
  <div data-action="online@window->app#wentOnline"></div>
  <div data-action="offline@window->app#wentOffline"></div>
  <div data-action="popstate@window->app#browserBack"></div>
</div>
```

#### Touch Events (Mobile)
```html
<div data-controller="touch">
  <div data-action="touchstart->touch#start
                    touchmove->touch#move
                    touchend->touch#end">
    Swipe me
  </div>
</div>
```

#### Custom Events (The Secret Weapon!)
```javascript
// One controller dispatches
export default class extends Controller {
  save() {
    this.dispatch("saved", { 
      detail: { id: 123 } 
    })
  }
}
```

```html
<!-- Other controllers listen -->
<div data-controller="notifications"
     data-action="item:saved->notifications#show">
</div>
```

#### Event Modifiers: Superpowers

```html
<!-- Only trigger on specific keys -->
<input data-action="keydown.enter->form#submit">
<input data-action="keydown.esc->modal#close">
<input data-action="keydown.ctrl+s->editor#save">

<!-- Prevent default behavior -->
<a href="#" data-action="click->handler#doSomething:prevent">

<!-- Stop event propagation -->
<button data-action="click->handler#handle:stop">

<!-- Run only once -->
<div data-action="click->tutorial#shown:once">

<!-- Combine them! -->
<form data-action="submit->form#handle:prevent:stop">
```

### Real-World Patterns

#### Auto-Submit Search
```erb
<%= form_with url: search_path, data: { controller: "autosubmit" } do |f| %>
  <%= f.text_field :query, 
      data: { action: "input->autosubmit#submit" } %>
<% end %>
```

```javascript
// autosubmit_controller.js
export default class extends Controller {
  submit() {
    clearTimeout(this.timeout)
    this.timeout = setTimeout(() => {
      this.element.requestSubmit()
    }, 300) // Debounce
  }
}
```

#### Character Counter
```erb
<div data-controller="character-counter" 
     data-character-counter-max-value="280">
  <textarea data-character-counter-target="input"
            data-action="input->character-counter#update"></textarea>
  <span data-character-counter-target="output">280</span> remaining
</div>
```

```javascript
export default class extends Controller {
  static targets = ["input", "output"]
  static values = { max: Number }
  
  update() {
    const remaining = this.maxValue - this.inputTarget.value.length
    this.outputTarget.textContent = remaining
    this.outputTarget.classList.toggle("text-red-500", remaining < 0)
  }
}
```

#### Command Palette (Advanced)
```erb
<div data-controller="command-palette"
     data-action="keydown@window->command-palette#handleKeydown">
  
  <button data-action="click->command-palette#open">⌘K</button>
  
  <div data-command-palette-target="modal" class="hidden">
    <input data-command-palette-target="input"
           data-action="input->command-palette#filter">
    
    <% @commands.each do |command| %>
      <button data-command-palette-target="command"
              data-action="click->command-palette#execute"
              data-command="<%= command.action %>">
        <%= command.label %>
      </button>
    <% end %>
  </div>
</div>
```

```javascript
export default class extends Controller {
  static targets = ["modal", "input", "command"]
  
  handleKeydown(event) {
    if (event.metaKey && event.key === 'k') {
      event.preventDefault()
      this.open()
    }
  }
  
  open() {
    this.modalTarget.classList.remove("hidden")
    this.inputTarget.focus()
  }
  
  filter() {
    const query = this.inputTarget.value.toLowerCase()
    this.commandTargets.forEach(command => {
      const match = command.textContent.toLowerCase().includes(query)
      command.classList.toggle("hidden", !match)
    })
  }
  
  execute(event) {
    const action = event.currentTarget.dataset.command
    Turbo.visit(action) // Navigate with Turbo!
    this.close()
  }
}
```

### The Magic: Stimulus Works With Turbo

When Turbo updates the page:
1. Old Stimulus controller `disconnect()` runs
2. New HTML renders  
3. New Stimulus controller `connect()` runs
4. **Everything just works!**

```erb
<!-- This dropdown works even after Turbo Frame updates -->
<turbo-frame id="sidebar">
  <div data-controller="dropdown">
    <button data-action="click->dropdown#toggle">Menu</button>
    <div data-dropdown-target="menu">...</div>
  </div>
</turbo-frame>
```

### Stimulus vs React/Vue: The Philosophical Divide


#### State Management: The Fundamental Divide

**What is "State"?**

State is **what your application remembers**. For a slideshow:
- Which slide is currently showing
- Is it playing or paused
- What's the slide interval

For a real app like Campfire:
- Which room is selected
- Is the sidebar open or closed
- What message is being edited
- Who is currently typing
- How many unread notifications

##### Contemporary Frameworks: JavaScript Owns State

In React/Vue, state lives in JavaScript memory:

```javascript
// React Slideshow - State lives in JavaScript memory
function Slideshow() {
  // THIS is the state - in JavaScript variables
  const [currentIndex, setCurrentIndex] = useState(0)
  const [isPlaying, setIsPlaying] = useState(false)
  
  const slides = ['🐵', '🙈', '🙉', '🙊']
  
  return (
    <div>
      <button onClick={() => setCurrentIndex(currentIndex - 1)}>←</button>
      <button onClick={() => setCurrentIndex(currentIndex + 1)}>→</button>
      
      {/* DOM is RENDERED from JavaScript state */}
      {slides.map((slide, index) => (
        <div key={index} style={{ display: index === currentIndex ? 'block' : 'none' }}>
          {slide}
        </div>
      ))}
    </div>
  )
}
```

**Key point:** The DOM is "write-only" - JavaScript creates it but never reads from it.

##### Stimulus: DOM Owns State

With Stimulus, state lives in the DOM itself as attributes:

```html
<!-- State is IN the DOM as data attributes -->
<div data-controller="slideshow" 
     data-slideshow-index-value="0"
     data-slideshow-playing-value="false">
  <button data-action="slideshow#previous">←</button>
  <button data-action="slideshow#next">→</button>
  
  <!-- ALL slides exist in the DOM -->
  <div data-slideshow-target="slide" class="active">🐵</div>
  <div data-slideshow-target="slide" class="hidden">🙈</div>
  <div data-slideshow-target="slide" class="hidden">🙉</div>
  <div data-slideshow-target="slide" class="hidden">🙊</div>
</div>
```

```javascript
// slideshow_controller.js - Controller is STATELESS
export default class extends Controller {
  static targets = ["slide"]
  static values = { index: Number, playing: Boolean }
  
  next() {
    // Updates data-slideshow-index-value in DOM
    this.indexValue = (this.indexValue + 1) % this.slideTargets.length
  }
  
  // Automatically called when data-slideshow-index-value changes
  indexValueChanged() {
    this.showCurrentSlide()
  }
  
  showCurrentSlide() {
    this.slideTargets.forEach((slide, index) => {
      slide.classList.toggle("active", index === this.indexValue)
    })
  }
}
```

**Key point:** The DOM is "read-write" - JavaScript reads state from DOM and writes back to it.

#### What You Actually See in the Browser Inspector

##### Stimulus Approach - ALL Content in HTML
```html
<!-- Open inspector, you see ALL slides -->
<div data-controller="slideshow" data-slideshow-index-value="0">
  <button>←</button>
  <button>→</button>
  
  <div data-slideshow-target="slide" class="active">🐵</div>
  <div data-slideshow-target="slide" class="hidden">🙈</div>
  <div data-slideshow-target="slide" class="hidden">🙉</div>
  <div data-slideshow-target="slide" class="hidden">🙊</div>
</div>

<!-- After clicking next, only classes change -->
<div data-slideshow-target="slide" class="hidden">🐵</div>
<div data-slideshow-target="slide" class="active">🙈</div>
```

##### React Approach - Only Current Content in DOM
```html
<!-- Open inspector, you see ONLY current slide -->
<div>
  <button>←</button>
  <button>→</button>
  
  <div>🐵</div>  <!-- Only this exists -->
</div>

<!-- After clicking next, DOM is replaced -->
<div>
  <button>←</button>
  <button>→</button>
  
  <div>🙈</div>  <!-- Previous slide gone, new one created -->
</div>
```

##### Problem 1: Server Setting Initial State

**Stimulus (Simple):**
```erb
<!-- Server just sets an attribute -->
<div data-controller="slideshow" 
     data-slideshow-index-value="<%= @starting_slide %>">
```

**React (Complex):**
```javascript
// Need API endpoint
fetch('/api/current-slide')
  .then(res => res.json())
  .then(data => setCurrentIndex(data.slideNumber))

// Or prop drilling through component tree
<App initialSlide={3}>
  <Parent>
    <Child>
      <Slideshow startingSlide={props.startingSlide} />
    </Child>
  </Parent>
</App>
```

##### Problem 2: Other Parts Reading State

**Stimulus (Simple):**
```javascript
// Any JavaScript can read DOM directly
const currentSlide = document
  .querySelector('[data-controller="slideshow"]')
  .dataset.slideshowIndexValue
```

**React (Complex):**
```javascript
// Need Context API or Redux
const SlideshowContext = createContext()

function App() {
  const [currentSlide, setCurrentSlide] = useState(0)
  
  return (
    <SlideshowContext.Provider value={{ currentSlide }}>
      <Slideshow />
      <DisplayCurrentSlide /> {/* Needs to access state */}
    </SlideshowContext.Provider>
  )
}

function DisplayCurrentSlide() {
  const { currentSlide } = useContext(SlideshowContext)
  return <div>Slide: {currentSlide}</div>
}
```

##### Problem 3: Real-Time Updates

**Stimulus + Turbo Streams (Simple):**
```erb
<!-- Server broadcasts state change -->
<%= turbo_stream.set_attribute "slideshow",
    "data-slideshow-index-value", 3 %>
<!-- Controller automatically reacts! -->
```

**React (Complex):**
```javascript
// Set up WebSocket
const ws = new WebSocket('...')

// Listen for updates
ws.onmessage = (event) => {
  const data = JSON.parse(event.data)
  if (data.type === 'SLIDE_CHANGE') {
    setCurrentIndex(data.slideNumber)
  }
}

// Handle reconnection, race conditions, etc...
```

##### Problem 4: State Persistence

**Stimulus (Natural):**
```html
<!-- State survives because it's in the DOM -->
<div data-turbo-permanent 
     data-slideshow-index-value="2">
<!-- User returns to same slide -->
</div>
```

**React (Manual):**
```javascript
// Save state
useEffect(() => {
  localStorage.setItem('slide', currentIndex)
}, [currentIndex])

// Restore state
useEffect(() => {
  const saved = localStorage.getItem('slide')
  if (saved) setCurrentIndex(Number(saved))
}, [])
```

#### Complete Example: Full-Featured Slideshow

##### Stimulus Implementation
```html
<div data-controller="slideshow"
     data-slideshow-index-value="0"
     data-slideshow-playing-value="false"
     data-slideshow-speed-value="3000">
     
  <button data-action="slideshow#previous">←</button>
  <button data-action="slideshow#togglePlay">
    <span data-slideshow-target="playText">Play</span>
  </button>
  <button data-action="slideshow#next">→</button>
  
  <select data-action="change->slideshow#updateSpeed">
    <option value="1000">Fast</option>
    <option value="3000" selected>Normal</option>
    <option value="5000">Slow</option>
  </select>
  
  <div data-slideshow-target="slide" class="active">🐵</div>
  <div data-slideshow-target="slide">🙈</div>
  <div data-slideshow-target="slide">🙉</div>
  <div data-slideshow-target="slide">🙊</div>
</div>
```

```javascript
// slideshow_controller.js
export default class extends Controller {
  static targets = ["slide", "playText"]
  static values = { 
    index: Number,
    playing: Boolean,
    speed: Number
  }
  
  next() {
    this.indexValue = (this.indexValue + 1) % this.slideTargets.length
  }
  
  previous() {
    this.indexValue = this.indexValue - 1
    if (this.indexValue < 0) {
      this.indexValue = this.slideTargets.length - 1
    }
  }
  
  togglePlay() {
    this.playingValue = !this.playingValue
  }
  
  updateSpeed(event) {
    this.speedValue = Number(event.target.value)
  }
  
  // These fire automatically when values change!
  indexValueChanged() {
    this.slideTargets.forEach((slide, index) => {
      slide.classList.toggle("active", index === this.indexValue)
    })
  }
  
  playingValueChanged() {
    this.playTextTarget.textContent = this.playingValue ? "Pause" : "Play"
    
    if (this.playingValue) {
      this.timer = setInterval(() => this.next(), this.speedValue)
    } else {
      clearInterval(this.timer)
    }
  }
  
  speedValueChanged() {
    if (this.playingValue) {
      // Restart with new speed
      this.playingValue = false
      this.playingValue = true
    }
  }
  
  disconnect() {
    clearInterval(this.timer)
  }
}
```

##### Why This Philosophy Matters

**Write-Only DOM (React) Causes:**

1. **State synchronization complexity** - Need Redux/Context
2. **Server communication overhead** - JSON APIs instead of HTML
3. **Debugging difficulty** - State hidden in memory
4. **Integration pain** - Other libraries can't interact easily
5. **Hydration issues** - Server/client state mismatch

**Read-Write DOM (Stimulus) Enables**

1. **State visibility** - Just inspect the DOM
2. **Server-side state setting** - Render with attributes
3. **Universal state access** - Any JS can read DOM
4. **Natural persistence** - State survives in DOM
5. **Simple integration** - Works with any library

#### Why Rails Developers Don't Talk About State

**Rails doesn't have "state sharing" problems because there's nothing to share!**

Each request is stateless:
```ruby
class RoomsController < ApplicationController
  def show
    @room = Room.find(params[:id])  # Fresh from database
    @messages = @room.messages       # Fresh from database
    @sidebar_open = true             # Rendered into HTML
    # Request ends, everything is garbage collected
  end
end
```

The "state" lives in:
1. **The Database** - The real source of truth
2. **The URL** - `/rooms/123` tells you which room
3. **The Session** - User login, preferences
4. **The HTML** - What the user sees IS the state

#### The Component State Sharing Problem (React World)

Imagine this component tree:
```
App
├── Header
│   └── NotificationBell (needs: notification count)
├── Sidebar
│   └── RoomList
│       └── RoomItem (needs: selected room)
└── ChatPanel
    ├── MessageList (needs: messages, selected room)
    └── TypingIndicator (needs: who's typing)
```

**The Problem:** NotificationBell needs the count, but state lives in App. You must pass it through Header (which doesn't need it). This is "prop drilling."

**React's Solution:** Complex state management (Redux, Context, Zustand)
```javascript
// Redux store
const store = {
  notifications: { count: 5 },
  rooms: { selected: 'room-123' },
  typing: { users: ['Alice'] }
}

// Any component can access
const count = useSelector(state => state.notifications.count)
```

#### The Revolutionary Insight

**Contemporary frameworks:** "DOM is dumb, keep state in JavaScript"
- Created entire state management industry (Redux, MobX, Vuex)
- Complicated server/client coordination
- Made simple things complex

**Stimulus:** "DOM is already a state container, just use it"
- State visible in HTML attributes
- Server can set initial state
- Any JavaScript can read/write state
- Eliminated entire categories of complexity

**The Punchline:** React developers spend enormous effort managing state because they've chosen to keep it in JavaScript memory, separate from the DOM. Rails developers don't talk about state because each request fetches fresh data, renders HTML, and dies. The DOM holds any temporary UI state.

#### Translating React Patterns to Stimulus

When you see React components with state, ask:
1. What state exists? → Make them data-*-value attributes
2. What renders based on state? → Toggle classes/content
3. What updates state? → Create controller actions
4. What reacts to state changes? → Use valueChanged callbacks

The DOM becomes your Redux store, but simpler!

#### When You Need Stimulus

**Perfect for:**
- UI interactions (dropdowns, modals, tabs)
- Form enhancements (auto-submit, validation)
- Keyboard shortcuts
- Clipboard operations
- Animations and transitions
- Third-party library integration

**Not needed for:**
- Navigation (Turbo Drive handles it)
- Form submission (Turbo handles it)
- Real-time updates (Turbo Streams)
- Content loading (Turbo Frames)

#### The Trix Editor Principle

DHH's example: Basecamp built Trix (rich text editor) in JavaScript because **you can't send every character to the server** - that would create 270ms lag per keystroke.

**The principle:** Build complex JavaScript components **once** when truly needed (character-by-character interaction), then reuse them. Don't rebuild everything in JavaScript just because you need **one** interactive component.

**Examples that genuinely need JavaScript:**
- Rich text editors (character-level interaction)
- Real-time collaborative editing (Google Docs)
- Canvas/graphics apps (Figma)
- Games (continuous interaction)

**Everything else:** Information display and manipulation - perfect for Hotwire.

#### How Stimulus Completes the Hotwire Vision

1. **Server renders HTML** (Rails)
2. **Turbo Drive** makes navigation smooth
3. **Turbo Frames** enable partial updates
4. **Turbo Streams** add real-time capabilities
5. **Stimulus** adds interactivity

Together: **99% of your app needs ZERO custom JavaScript!**

### The Complete Picture

```erb
<!-- Everything working together -->
<div data-controller="email-app">
  
  <!-- Turbo Drive: Smooth navigation -->
  <%= link_to "Inbox", inbox_path %>
  
  <!-- Turbo Frames: Partial updates -->
  <turbo-frame id="email-list" src="/emails">
    Loading...
  </turbo-frame>
  
  <!-- Turbo Streams: Real-time -->
  <%= turbo_stream_from "notifications" %>
  <div id="notifications"></div>
  
  <!-- Stimulus: Interactivity -->
  <button data-action="click->email-app#compose">
    Compose
  </button>
</div>
```



### Working with External Resources

External resources = anything outside DOM (timers, HTTP requests, WebSockets, APIs)

#### Content Loader Pattern
```html
<div data-controller="content-loader"
     data-content-loader-url-value="/messages.html"
     data-content-loader-refresh-interval-value="5000">
</div>
```

```javascript
export default class extends Controller {
  static values = { url: String, refreshInterval: Number }
  
  connect() {
    this.load()
    if (this.hasRefreshIntervalValue) {
      this.startRefreshing()
    }
  }
  
  load() {
    fetch(this.urlValue)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
  
  startRefreshing() {
    this.timer = setInterval(() => {
      this.load()
    }, this.refreshIntervalValue)
  }
  
  disconnect() {
    // CRUCIAL: Clean up external resources!
    if (this.timer) clearInterval(this.timer)
  }
}
```

### When to Use Each Tool for Loading Content

#### Turbo Frames
```erb
<!-- Best for: Navigation, one-time loading -->
<turbo-frame id="messages" src="/messages" loading="lazy">
  <!-- Loads once, handles navigation within frame -->
</turbo-frame>
```

**Pros:** Declarative, automatic, handles navigation
**Cons:** Can't poll, no response processing

#### Turbo Streams  
```erb
<!-- Best for: Real-time updates -->
<%= turbo_stream_from "messages" %>
<div id="messages"></div>
```

**Pros:** Real-time, server-pushed, multiple updates
**Cons:** Requires WebSocket, overkill for polling

#### Stimulus
```html
<!-- Best for: Custom loading logic, polling -->
<div data-controller="poller"
     data-poller-url-value="/api/stats"
     data-poller-interval-value="1000">
</div>
```

**Pros:** Full control, can process response, handles cleanup
**Cons:** More code to write

#### The Complete Loading Strategy

```erb
<!-- Use the right tool for each job -->

<!-- Page navigation -->
<%= link_to "Inbox", inbox_path %>

<!-- Partial with navigation -->
<turbo-frame id="inbox" src="/messages">
  <%= link_to "Page 2", messages_path(page: 2) %>
</turbo-frame>

<!-- Real-time updates -->
<%= turbo_stream_from "notifications" %>

<!-- Custom polling/processing -->
<div data-controller="custom-loader"
     data-custom-loader-url-value="/special"
     data-custom-loader-transform-value="markdown">
</div>
```

### The Key Philosophy

**Turbo = Declarative** ("What I want")
- Frames: "Load this content here"
- Streams: "Update when server says"

**Stimulus = Imperative** ("How to do it")
- "Fetch every 5 seconds"
- "Process response as markdown"
- "Clean up when done"

Use Turbo for 90% of cases. Use Stimulus when you need control over the **how/when/why** of external resources.


### Where JavaScript Frameworks Actually Win

1. Complex Local State Synchronization

Google Docs' Real Challenge:
// Multiple users editing same document
// Need to handle:
- Operational Transformation (OT) algorithms
- Cursor positions for each user
- Local changes vs remote changes
- Conflict resolution
- Offline editing with sync

// This requires complex in-memory state
const documentState = {
  content: [...],
  localChanges: [...],
  remoteChanges: [...],
  cursors: { user1: pos1, user2: pos2 },
  conflicts: [...],
  undoStack: [...],
  redoStack: [...]
}

This genuinely needs sophisticated JavaScript because you're managing a complex data structure with algorithms running client-side.

2. Canvas/WebGL/Heavy Graphics

Figma, Photopea, Games:
// Direct pixel manipulation
ctx.drawImage(...)
ctx.fillRect(...)
// 60fps animations
requestAnimationFrame(render)

Stimulus can't help here - you need raw JavaScript performance.

3. Offline-First Applications

Apps that work without server:
- Notion (works offline)
- Linear (syncs when online)
- Obsidian (local-first)

These need client-side data stores (IndexedDB), sync engines, and conflict resolution.

What People THINK You Need JS Frameworks For (But Don't)

1. Rich Text Editing

Myth: "You need React for rich text editors"

Reality: Basecamp's Trix editor is vanilla JS + works perfectly with Hotwire
<%= form_with model: @post do |f| %>
  <%= f.rich_text_area :content %>  <!-- Full rich text editor! -->
<% end %>

2. Real-Time Collaboration

Myth: "Multi-user features need React"

Reality: Hotwire handles this beautifully
// Real-time collaborative todo list
class Todo < ApplicationRecord
  after_create_commit -> {
    broadcast_prepend_to "todos"
  }
  after_update_commit -> {
    broadcast_replace_to "todos"
  }
end

3. Drag and Drop

Myth: "Complex interactions need frameworks"

Reality: Stimulus + Sortable.js works great
import { Controller } from "@hotwired/stimulus"
import Sortable from "sortablejs"

export default class extends Controller {
  connect() {
    this.sortable = Sortable.create(this.element)
  }
}

4. Spreadsheets

Myth: "Excel-like apps need React"

Reality: You could build a decent spreadsheet with Hotwire
- Each cell is a Turbo Frame
- Formulas calculated server-side
- Real-time updates via Turbo Streams

The question is: Should you? Probably not for 10,000 cells.

The Real Dividing Line

Use Hotwire When:

- Server has the data (databases, files)
- Server does the computation (business logic)
- Collaboration is turn-based (comments, posts)
- UI updates are discrete (form submission, navigation)

Use JavaScript Frameworks When:

- Client has unique data (mouse position, local edits)
- Client does heavy computation (image processing)
- Collaboration is character-by-character (Google Docs)
- UI updates are continuous (animations, games)

## Why People Don't Realize Hotwire's Power

1. It's Too New

- Turbo: Released 2020
- Most developers learned React/Vue first
- Few large-scale Hotwire examples

2. JavaScript Momentum

- Entire industry invested in JS
- Boot camps teach React
- Job market demands React

3. Nobody's Tried

Hey.com proved you can build Gmail without JavaScript frameworks. Nobody's built Google Docs with Hotwire because:
- The companies that could (37signals) don't need it
- Startups default to React (that's what VCs expect)
- It's genuinely hard (but so is React version)

4. The 10% Case Gets 90% Attention

- Blog posts about "Building Google Docs Clone in React"
- No posts about "Building Basecamp with Hotwire"
- Complex examples get more clicks

Could You Build Google Docs with Hotwire?

Technically? Mostly yes:
<!-- The document -->
<div data-controller="document"
      data-action="input->document#sync">
  <div contenteditable="true"
        data-document-target="editor">
  </div>
</div>

<!-- Real-time cursor positions -->
<%= turbo_stream_from @document %>

<!-- Sync changes -->
<div id="cursors">
  <!-- Turbo Streams update cursor positions -->
</div>

// document_controller.js
export default class extends Controller {
  sync() {
    // Debounced sync to server
    clearTimeout(this.timeout)
    this.timeout = setTimeout(() => {
      this.pushChanges()
    }, 500)
  }

  pushChanges() {
    // Send changes via Turbo
    fetch('/documents/sync', {
      method: 'POST',
      body: this.editorTarget.innerHTML
    })
  }
}

But should you? Probably not. The complexity of:
- Operational transformation
- Conflict-free replicated data types (CRDTs)
- Character-by-character sync
- Offline support

...is better handled client-side.

The Bottom Line

Hotwire can handle 95% of web applications:
- Project management (Basecamp, Linear-style)
- Email clients (Hey.com proved it)
- Social networks (Twitter-like feeds)
- E-commerce (Shopify-style)
- SaaS dashboards
- Content management
- Forums/Communities

The 5% that genuinely need JS frameworks:
- Real-time collaborative editing (Google Docs)
- Design tools (Figma, Canva)
- Offline-first apps (Notion)
- Games
- Heavy data visualization (Complex D3.js)


## Case Study: Campfire by 37Signals

### 1. Why does "HTML over the wire" matter?

#### What complexity disappears when you don't need JSON serialization?

Look at Campfire's message creation in `app/controllers/messages_controller.rb:20-28`:

```ruby
# The ENTIRE message creation and real-time update logic:
def create
  set_room
  @message = @room.messages.create_with_attachment!(message_params)
  @message.broadcast_create  # Broadcasts HTML, not JSON!
  deliver_webhooks_to_bots
rescue ActiveRecord::RecordNotFound
  render action: :room_not_found
end
```

Compare this to what a React chat app would need:

```javascript
// React version would need:
// 1. API endpoint that returns JSON
app.post('/api/messages', async (req, res) => {
  const message = await Message.create(req.body)
  res.json({
    id: message.id,
    body: message.body,
    created_at: message.created_at,
    user: {
      id: message.user.id,
      name: message.user.name,
      avatar_url: message.user.avatar_url
    },
    attachments: message.attachments.map(a => ({...}))
  })
})

// 2. Client-side state management
const [messages, setMessages] = useState([])

// 3. Fetching and parsing
const response = await fetch('/api/messages', {...})
const data = await response.json()
setMessages([...messages, data])

// 4. React component to render
function Message({ message }) {
  return (
    <div className="message">
      <img src={message.user.avatar_url} />
      <div>{message.user.name}</div>
      <div>{message.body}</div>
    </div>
  )
}
```

**What disappears with HTML over the wire:**
- No JSON serialization layer
- No client-side state synchronization
- No duplicate rendering logic (server has template, client has React component)
- No API versioning issues
- No parsing/transformation logic

In Campfire, the server just sends this in `app/views/messages/create.turbo_stream.erb`:
```erb
<%= turbo_stream.append dom_id(@message.room, :messages), @message %>
```

That's it! The `@message` renders through the same `_message.html.erb` partial used everywhere.

#### How does this affect debugging?

**With Campfire/Hotwire:**
- Open browser inspector → Network tab → See actual HTML being sent
- What you see is literally what gets inserted into the DOM
- Server logs show exactly what was rendered
- One rendering path to debug (server-side ERB templates)

**With React/JSON:**
- Network tab shows JSON → need to mentally map to UI
- Client console shows state → may not match what's rendered
- Server logs show JSON → different from what user sees
- Two rendering paths to debug (server JSON + client React)

### 2. Why is DOM-as-state-container revolutionary?

#### What problems does Redux solve that don't exist with Stimulus?

Look at Campfire's `composer_controller.js`. It manages complex state (files being uploaded, message being typed, upload progress) but needs ZERO state management library:

```javascript
// No Redux needed! State lives in the DOM
export default class extends Controller {
  static targets = [ "clientid", "fields", "fileList", "text" ]
  static values = { roomId: Number }  // State is in data-room-id-value attribute!

  #files = []  // Temporary local state, not app state

  // When user types, the state is IN the textarea element
  // When files are selected, they're displayed in fileListTarget
  // No setState, no reducers, no actions!
}
```

Redux exists to solve these React problems:
1. **Prop drilling** - passing data through many components
2. **State synchronization** - keeping multiple components in sync
3. **Time-travel debugging** - seeing how state changed
4. **Server state hydration** - matching server/client state

**None of these problems exist with Stimulus because:**
1. **No prop drilling** - Any element can read DOM attributes directly
2. **No sync issues** - The DOM IS the state
3. **Time-travel built-in** - Browser DevTools show DOM changes
4. **No hydration** - Server renders HTML with state already in attributes

#### Why do React developers fear the DOM?

React developers avoid direct DOM manipulation because:
1. **Virtual DOM reconciliation** - Direct DOM changes break React's diffing
2. **State mismatch** - DOM changes don't update React state
3. **Performance myths** - Told "DOM is slow" (but it's the teardown/rebuild that's slow!)

Stimulus embraces the DOM because:
1. **No virtual DOM** - The real DOM is the source of truth
2. **State lives in DOM** - Changing DOM attributes IS updating state
3. **Surgical updates** - Only change what needs changing

### 3. What does "locality of behavior" mean?

#### Why is `data-action="click->controller#method"` better than separate event listeners?

Look at any Campfire view file. The behavior is declared RIGHT where it happens:

```erb
<!-- From app/views/messages/_actions.html.erb -->
<div class="message__actions" data-controller="soft-keyboard">
  <button data-action="click->reply#quote">Quote</button>
  <button data-action="click->boost#toggle">Boost</button>
</div>
```

**You can see EVERYTHING about this element in one place:**
- What it looks like (HTML/CSS)
- What controllers enhance it (`data-controller`)
- What happens when you interact (`data-action`)

**Compare to React with separate event handlers:**
```javascript
// Event handler is somewhere else in the file or another file
function MessageActions() {
  const handleQuote = () => { /* logic here */ }
  const handleBoost = () => { /* logic here */ }

  return (
    <div>
      <button onClick={handleQuote}>Quote</button>
      <button onClick={handleBoost}>Boost</button>
    </div>
  )
}
```

To understand the React version, you need to:
1. Find the component definition
2. Find the event handler function
3. Trace through any props/callbacks
4. Check what actions/reducers it dispatches

**Locality of behavior means:** Everything about an element's behavior is declared on the element itself.

### 4. Why do Turbo Frames have `src`?

#### How does this enable "micro-frontends in a monolith"?

Campfire's sidebar is a perfect example. Look at `app/views/users/sidebars/show.html.erb`:

```erb
<turbo-frame id="sidebar" src="<%= user_sidebar_path %>">
  <!-- This frame loads its content from a separate URL -->
</turbo-frame>
```

This creates a **micro-frontend** because:
1. **Independent URL** - Has its own endpoint
2. **Independent updates** - Can refresh without touching the rest of the page
3. **Independent caching** - Can have different cache strategies
4. **Lazy loadable** - Can load when needed with `loading="lazy"`
5. **Composable** - Can be embedded in any page

#### What architectural patterns does this unlock?

**1. Parallel Data Loading:**
```erb
<!-- Room page loads multiple independent sections in parallel -->
<div class="room">
  <%= turbo_frame_tag "messages", src: room_messages_path(@room) %>
  <%= turbo_frame_tag "members", src: room_members_path(@room), loading: "lazy" %>
  <%= turbo_frame_tag "files", src: room_files_path(@room), loading: "lazy" %>
</div>
```

Each frame loads independently, in parallel!

**2. Progressive Enhancement:**
```erb
<!-- Works without JavaScript, enhances with it -->
<turbo-frame id="search_results" src="/search">
  <a href="/search">View search results</a> <!-- Fallback -->
</turbo-frame>
```

**3. Team Independence:**
Different teams can own different frames:
- Chat team owns `/messages` endpoint
- Profile team owns `/sidebar` endpoint
- Search team owns `/search` endpoint

All compose into one page without coordination!

**4. Smart Caching:**
```ruby
class MessagesController < ApplicationController
  def index
    @messages = @room.messages
    fresh_when @messages  # Different cache strategy per frame!
  end
end
```

### The Revolutionary Realizations from Campfire

#### 1. Real-time Chat Without WebSocket Complexity

Campfire implements real-time messaging with just:

```ruby
# app/models/message/broadcasts.rb
module Message::Broadcasts
  def broadcast_create
    broadcast_append_to room, :messages, target: [ room, :messages ]
    ActionCable.server.broadcast("unread_rooms", { roomId: room.id })
  end
end
```

**That's the ENTIRE real-time implementation!** No Socket.io, no connection management, no client-side WebSocket code.

#### 2. File Uploads Without APIs

Look at `composer_controller.js` handling file uploads:

```javascript
const uploader = new FileUploader(file, this.element.action, clientMessageId, this.#uploadProgress.bind(this))
const resp = await uploader.upload()
Turbo.renderStreamMessage(resp)  // Server responds with HTML, not JSON!
```

The server responds with Turbo Stream HTML that updates the page. No JSON parsing, no state updates, no manual DOM manipulation.

#### 3. Complex UI State Without State Libraries

The typing indicator, online/offline status, notification badges - all handled with simple Stimulus controllers that read/write DOM attributes. No Redux, no MobX, no Zustand.

### Why This All Matters

**The Old World (2010-2020):**
- Build API backend
- Build React frontend
- Manage state with Redux
- Handle WebSockets manually
- Deploy two applications
- Maintain two codebases

**The Hotwire World (2020+):**
- Build one Rails app
- HTML does everything
- DOM is your state
- WebSockets are automatic
- Deploy one application
- Maintain one codebase

**Campfire proves you can build Slack-level features with 10x less complexity.**