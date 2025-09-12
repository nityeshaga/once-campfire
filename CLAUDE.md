# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Campfire is a web-based team chat application built with Ruby on Rails. It's a single-tenant chat platform with real-time messaging, file attachments, push notifications, and bot integrations.

## Essential Commands

### Development
```bash
bin/setup          # Initial setup (installs dependencies, prepares database, removes old logs)
bin/rails server   # Start development server on localhost:3000
bin/dev            # Development mode with file watching
```

### Testing
```bash
bin/rails test               # Run unit and integration tests
bin/rails test:system       # Run system tests with headless Chrome
bin/rails test test/path/to/specific_test.rb  # Run a specific test file
bin/rails test test/path/to/test.rb:LINE_NUMBER  # Run specific test at line number
bin/ci                       # Run full CI suite locally
```

### Code Quality
```bash
bin/rubocop         # Ruby linting and style checks
bin/rubocop -a      # Auto-fix linting issues
bin/brakeman        # Security vulnerability scanning
bin/bundler-audit   # Check for vulnerable gem dependencies
```

### Database
```bash
bin/rails db:migrate         # Run database migrations
bin/rails db:rollback        # Rollback last migration
bin/rails db:seed            # Load seed data
bin/rails db:reset           # Drop, create, migrate, and seed database
```

## Architecture Overview

### Tech Stack
- **Rails 8.1** (edge) with Ruby 3.4.5
- **SQLite3** for database (including production)
- **Redis** for caching and job queuing
- **Hotwire** (Turbo + Stimulus) for frontend interactivity
- **ActionCable** for WebSocket real-time features
- **Resque** for background job processing

### Core Domain Models

The application centers around these key models:
- **User**: Authentication, profiles, with roles (admin/member/bot)
- **Room**: Chat rooms with three types - Open (public), Closed (invite-only), Direct (1:1)
- **Message**: Rich text messages with attachments, stored in `messages` table
- **Membership**: Links users to rooms with involvement tracking
- **Account**: Organization/tenant management (single-tenant deployment)

### Real-time Architecture

The app uses ActionCable for WebSocket connections with these key channels:
- `ApplicationChannel`: Base channel with room-based streaming
- Messages broadcast to `room:#{room_id}` channels
- Turbo Streams for DOM updates without full page reloads

### JavaScript Organization

Frontend code uses Stimulus controllers (40+ controllers in `app/javascript/controllers/`):
- Each controller handles specific UI behavior
- Controllers are auto-loaded via `application.js`
- Key controllers: `message_controller.js`, `room_controller.js`, `mentions_controller.js`

### Background Jobs

Uses Resque with resque-pool for job processing:
- Job classes in `app/jobs/`
- Critical jobs: message delivery, push notifications, file processing
- Redis required for job queue management

### File Storage

- Uses Active Storage for file attachments
- Local disk storage in `/rails/storage`
- Image processing via libvips
- Media processing with FFmpeg

## Testing Guidelines

### Test Structure
- Unit tests in `test/models/`
- Controller tests in `test/controllers/`
- System tests in `test/system/`
- Test fixtures in `test/fixtures/`

### Running Tests
- Tests run in parallel by default
- Use `PARALLEL_WORKERS=1` to run serially if needed
- System tests use headless Chrome via Selenium

### Test Helpers
Custom test helpers available:
- `SessionTestHelper`: Authentication helpers for tests
- `MentionsTestHelper`: Testing @mentions functionality
- `TurboTestHelper`: Turbo-specific assertions

## Development Workflow

1. **Feature Development**: Create feature branch from `main`
2. **Testing**: Write tests first or alongside implementation
3. **Linting**: Run `bin/rubocop` before committing
4. **Security**: Run `bin/brakeman` for security checks
5. **CI**: GitHub Actions runs tests, linting, and security scans automatically

## Key Conventions

### Ruby/Rails Conventions
- Follow Rails conventions for naming and file organization
- Use concerns for shared model/controller behavior
- Prefer Rails built-in features over external gems when possible

### Database
- All times stored in UTC
- Use Rails migrations for schema changes
- SQLite-specific features (FTS5) used for search

### Frontend
- Stimulus controllers for JavaScript behavior
- Turbo for page navigation and real-time updates
- CSS classes follow Rails conventions
- Import maps for JavaScript modules (no webpack/bundling)

### API/Bots
- Bot authentication via token in `app/controllers/bots/`
- RESTful JSON API endpoints
- Webhook support for integrations

## Deployment Notes

- Designed for single-machine Docker deployment
- All services (web, jobs, redis) run in single container
- SSL via Let's Encrypt (automatic with SSL_DOMAIN env var)
- Persistent volume required at `/rails/storage`