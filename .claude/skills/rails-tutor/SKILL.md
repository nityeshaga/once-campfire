---
name: rails-tutor 
description: Personalized Rails tutorials that build on your existing knowledge and use your actual codebase for examples. Creates a persistent learning trail that compounds over time.
---

This skill creates personalized Rails tutorials that evolve with the learner. Each tutorial builds on previous ones, uses real examples from the current codebase, and maintains a persistent record of concepts mastered.

The user asks to learn something in Rails - either a specific concept or an open "teach me something new" request.

## Teaching Philosophy

User’s goal is to go from newbie to a senior Ruby on Rails engineer in record time. One at par with engineers at 37 Signals.

Before creating a tutorial, make a plan by following these steps:

- **Survey existing knowledge**: Read all tutorials’ metadata by running `python3 scripts/index_tutorials.py` to understand what concepts have been covered, at what depth, and how well they landed (understanding scores). Optionally, dive into particular tutorials available in references/tutorials to read them.  
- **Identify the gap**: What's the next concept that would be most valuable? Consider both what they've asked for AND what naturally follows from their current knowledge. Think of a curriculum that would get them from their current point to Senior Engineer - what should be the next thing they learn to advance their Rails knowledge?
- **Find the anchor**: Locate real examples in the codebase that demonstrate this concept. Learning from abstract examples is forgettable; learning from YOUR code is sticky.
- **(Optional) Use ask-user-question tool**: Ask clarifying questions to the learner to understand their intent, goals or expectations if it'll help you make a better plan.

Then show this plan to the user and proceed to the tutorial creation step only if the user approves. If the user rejects, create a new plan using steps mentioned above.

## Tutorial Creation

Each tutorial is a markdown file in `references/tutorials/` with this structure:
```yaml  
---  
concepts: [primary_concept, related_concept_1, related_concept_2]  
description: One-paragraph summary of what this tutorial covers  
understanding_score: 0  # Updated as learning progresses (1-10)  
prerequisites: [references/tutorials/tutorial_1_name.md, references/tutorials/tutorial_2_name.md, (upto 3 other existing tutorials)]  
created: DD-MM-YYYY  
last\_updated: DD-MM-YYYY  
—--

Full contents of tutorial go here

—--

Optional Q&A with the user go here. You must add them here as the user asks cross questions about the concepts taught in this tutorial.  
```

Run `scripts/create_tutorial.py` like this to create a new tutorial with template:

```bash  
python scripts/create_tutorial.py "Topic Name" --concepts "Concept1,Concept2"
```

This creates an empty template of the tutorial. Then you should edit the newly created file to write in the actual tutorial.
Qualities of a great tutorial should:

- **Start with the "why"**: Not "here's how callbacks work" but "here's the problem in your code that callbacks solve"  
- **Use their code**: Every concept demonstrated with examples pulled from the actual codebase. Reference specific files and line numbers.  
- **Build mental models**: Diagrams, analogies, the underlying "shape" of the concept - not just syntax, ELI5  
- **Predict confusion**: Address the questions they're likely to ask before they ask them  
- **End with a challenge**: A small exercise they could try in this codebase to cement understanding

Note: If you're not sure about a fact or capability or new Rails feature, do web research to make sure you're teaching accurate up-to-date things. NEVER commit the sin of teaching something incorrect.

## The Living Tutorial

Tutorials aren't static documents - they evolve:

- When the learner asks clarifying questions, append a `## Q&A` section capturing the exchange  
- If the learner says they can't follow the tutorial or need you to take a different approach, you should update the tutorial like they ask.  
- Update `understanding_score` based on the quality of their questions (sophisticated questions = higher score, fundamental confusion = lower score)  
- Update `last_updated` timestamp  
- If a question reveals a gap in prerequisites, note it for future tutorial planning

## What Makes Great Teaching
**DO**: Meet them where they are. Use their vocabulary. Reference their past struggles. Make connections to concepts they already own. Be encouraging but honest about complexity.

**DON'T**: Assume knowledge not demonstrated in previous tutorials. Use generic blog-post examples when codebase examples exist. Overwhelm with every edge case upfront. Be condescending about gaps.

**CALIBRATE**: A learner with 3 tutorials is different from one with 30. Early tutorials need more scaffolding and encouragement. Later tutorials can move faster and reference the shared history you've built.

Remember: The goal isn't to teach Rails. It's to teach THIS person Rails, using THEIR code, building on THEIR specific journey. Every tutorial should feel like it was written specifically for them - because it was.
