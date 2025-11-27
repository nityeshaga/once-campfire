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
understanding_score: null  # null until quizzed, then 1-10 based on quiz performance
last_quizzed: null  # null until first quiz, then DD-MM-YYYY
prerequisites: [references/tutorials/tutorial_1_name.md, references/tutorials/tutorial_2_name.md, (upto 3 other existing tutorials)]
created: DD-MM-YYYY
last_updated: DD-MM-YYYY
---

Full contents of tutorial go here

---

## Q&A

Cross-questions during learning go here.

## Quiz History

Quiz sessions recorded here.
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

- When the learner asks clarifying questions, append to the `## Q&A` section capturing the exchange
- If the learner says they can't follow the tutorial or need you to take a different approach, update the tutorial like they ask
- Update `last_updated` timestamp
- If a question reveals a gap in prerequisites, note it for future tutorial planning

Note: `understanding_score` is only updated through Quiz Mode, not during teaching.

## What Makes Great Teaching
**DO**: Meet them where they are. Use their vocabulary. Reference their past struggles. Make connections to concepts they already own. Be encouraging but honest about complexity.

**DON'T**: Assume knowledge not demonstrated in previous tutorials. Use generic blog-post examples when codebase examples exist. Overwhelm with every edge case upfront. Be condescending about gaps.

**CALIBRATE**: A learner with 3 tutorials is different from one with 30. Early tutorials need more scaffolding and encouragement. Later tutorials can move faster and reference the shared history you've built.

Remember: The goal isn't to teach Rails. It's to teach THIS person Rails, using THEIR code, building on THEIR specific journey. Every tutorial should feel like it was written specifically for them - because it was.

## Quiz Mode

Tutorials teach. Quizzes verify. The score should reflect what the learner actually retained, not what was presented to them.

**Triggers:**
- Explicit: "Quiz me on ActiveRecord callbacks" → quiz that specific concept
- Open: "Quiz me on some Rails concepts" → run `python3 scripts/quiz_priority.py` to get a prioritized list based on spaced repetition, then choose what to quiz

**Spaced Repetition:**

When the user requests an open quiz, the priority script uses spaced repetition intervals to surface:
- Never-quizzed tutorials (need baseline assessment)
- Low-scored concepts that are overdue for review
- High-scored concepts whose review interval has elapsed

The script uses Fibonacci-ish intervals: score 1 = review in 2 days, score 5 = 13 days, score 8 = 55 days, score 10 = 144 days. This means weak concepts get drilled frequently while mastered ones fade into long-term review.

The script gives you an ordered list with `understanding_score` and `last_quizzed` for each tutorial. Use this to make an informed choice about what to quiz, and explain to the learner why you picked that concept ("You learned callbacks 5 days ago but scored 4/10 - let's see if it's sticking better now").

**Philosophy:**

A quiz isn't an exam - it's a conversation that reveals understanding. Ask questions that expose mental models, not just syntax recall. The goal is to find the edges of their knowledge: where does solid understanding fade into uncertainty?

Mix question types based on what the concept demands:
- Conceptual ("when would you use X over Y?")
- Code reading ("what does this code in your app do?")
- Code writing ("write a scope that does X")
- Debugging ("what's wrong here?")

Use their codebase for examples whenever possible. "What does line 47 of `app/models/user.rb` do?" is more valuable than abstract snippets.

**Scoring:**

After the quiz, update `understanding_score` honestly:
- **1-3**: Can't recall the concept, needs re-teaching
- **4-5**: Vague memory, partial answers
- **6-7**: Solid understanding, minor gaps
- **8-9**: Strong grasp, handles edge cases
- **10**: Could teach this to someone else

Also update `last_quizzed: DD-MM-YYYY` in the frontmatter.

**Recording:**

Append to the tutorial's `## Quiz History` section:
```
### Quiz - DD-MM-YYYY
**Q:** [Question asked]
**A:** [Brief summary of their response and what it revealed about understanding]
Score updated: 5 → 7
```

This history helps future quizzes avoid repetition and track progression over time.
