# Campfire Codebase References

This directory contains the complete Campfire codebase for reference when providing DHH-style Rails guidance.

## Contents

- **app/** - All application code (models, controllers, views, JavaScript, jobs, channels, helpers)
- **config/** - Configuration files showing Rails setup patterns
- **db/** - Migrations showing database design evolution
- **lib/** - Custom library code and extensions
- **test/** - Complete test suite showing DHH's testing patterns

## What's Included

The full Campfire codebase with all code, tests, and configurations:
- **Models** with concerns, callbacks, and associations
- **Controllers** showing RESTful patterns and Turbo usage
- **Views** demonstrating server-rendered UI with Turbo Streams
- **Stimulus controllers** for JavaScript behavior
- **Background jobs** and ActionCable channels
- **Rails configuration** and initializers
- **Complete test suite** - system, model, controller, and integration tests
- **Test helpers** showing testing patterns
- **CSS/Stylesheets** for styling approaches

## What's Excluded

To keep the skill under size limits:
- **test/fixtures/files/** (51MB of sample images/videos like earth.png, alpha-centuri.mov)
- **app/assets/sounds/** (MP3 sound files)
- **app/assets/images/sounds/** (GIF/WebP sound reaction images)

These are test fixture files and assets not relevant for understanding Rails patterns.

## Usage

When reviewing code or making architectural decisions, search these directories for:
- **Specific patterns** (e.g., "how does Campfire handle file uploads?")
- **Concrete examples** to reference in feedback
- **Proven DHH-style implementations**
- **Testing approaches** - see how DHH structures tests
- **Configuration patterns** - Rails setup and initializers

The skill will automatically reference specific files and line numbers from this codebase when providing guidance.

## Testing Patterns Available

With the complete test suite included, you can now reference:
- **System tests** (`test/system/`) - Real user workflow testing with headless Chrome
- **Model tests** (`test/models/`) - Business logic and validation testing
- **Controller tests** (`test/controllers/`) - Request/response testing
- **Channel tests** (`test/channels/`) - ActionCable real-time testing
- **Test fixtures** (`test/fixtures/*.yml`) - Data structure examples
- **Test helpers** (`test/test_helpers/`) - Custom testing utilities
