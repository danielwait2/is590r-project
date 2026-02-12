# User Prompts

## 2026-02-11

I want to create a new project. The project is a **Couples & Family Quality Time App**, implemented as a **calendar-native integration** primarily with Google Calendar. The core problem it solves is the **Desire-to-Action Gap**: couples and families have mental lists of things they want to do together, but the experiences never happen due to the friction of planning, time blindness, and discovery effort. The primary customer is **dual-income parents with children under 10**, who are calendar-dependent and suffer from the highest desire-to-action gap. The solution is an activity facilitation system where users perform a one-time setup to input their interests, a family profile, and a "Want to Try" list. The app then automatically monitors their calendars for free time and uses local data sources (like Google Places API and Eventbrite API) to generate and insert specific, actionable suggestions directly into their Google Calendar. The key differentiator is that it bridges intention and action by connecting a couple's **stated desires** with their **available time** and **local opportunities**, turning "we should do that sometime" into "here's how to do it this Saturday." I dont know... how does this sound to you? How might this work?

---

## 2026-02-11 (continued)

Ok before we go further - I want you to be a devils advocate here. Why might this idea fail? What am I underestimating? What assumptions am I making that could be wrong? Be honest with me, dont hold back. Id rather hear the hard stuff now that after Ive built it.

---

## 2026-02-11 (continued)

Please give me a prompt that I can share with an online researcg agent such that we caan do market research on our project? Dont worry about any of the devils advocate points at this time.

---

## 2026-02-11 (continued)

I just did some market research on Perplexity. here are the results - please save this to ai/guides/quality-time-app-market-research.md and then give me your take on where our opportunity is: @ai/guides/external/marketResearch_perplexity.md

---

## 2026-02-11 (continued)

Ok. Ive seen enough. lets move forward with this. I think this is goind to be great with an engine and a seemless integration differentiator - no one is doing that well. Lets write up a PRD. Use market research we saved in ai/guides/ to inform the competitve landscape and positioning.

Some decisions: keep it simpke for an mvp.

Please create my PRD. Save it to aiDocs/prd.md

---

## 2026-02-11 (continued)

Great! Next, lets reconfigure the MVP. And lets not save the MVP infor in the prd doc. Lets create one next to it ccalled mvp.md. For out mvp, we just need be able to do a demo that it works. thinking a plugin or chrome extension that will read your google caendar and your things you want to do and add suggestions. How does this sound? Is this functional? Did I miss anything? This will be to show a demo of how this might wor.

---

## 2026-02-11 (continued)

Lets get our project structured set up properly. I need:

1. The two folder pattern. aiDocs/ (tracked shared project knowledge) and ai/ (gitignore, my local working space). ai/ should have guides/, roadmaps/, and notes/ subdirectories.
2. a solid .gitignore - make sure ai/ claud.md, .cursorrules, node_modules, .env, .testEnvVars are all ignored.
3. a context.md in aiDocs. that references our PRD and MVP lists our current tech stack, and has a "Current Focus" section. keep it concise - bullet points, not essays.
4. a CLAUDE.md file in the project root that points to aiDOcs/context.md and includes some behavioral guidelines - like asking my opioion before complex work, not over-engineering, and flagging uncertainty instead of guiesssing.

The PRD and MVP files we already created should stay in aiDocs/ where they are.

---

## 2026-02-11 (continued)

Before we start planning the implementation, I want to make sure we're working with real docs and not hallucinated APIs. Can you use context7 to pull the current docs for:
1. sharp (the node.js image processing library — we'll probably use this for text overlay)
2. openair(the node.js SDK — for the vision API and chat completions)
Save what you find to ai/guides/ so we can reference it during planning. I don't want us building a plan based on APIs that don't exist.

---
## 2026-02-11 (continued)

Please create a claude.md file in this project that reference @aiDocs/context.md  keep it simple otherwise.

---

## 2026-02-12

I need to create detailed implementation plans for each phase of the Family Quality Time App project. Read the main roadmap at ai/roadmaps/2026-02-11-high-level_mvp-project-roadmap.md and create separate detailed sub-plans for each phase.

Create the following files in ai/roadmaps/:

1. **2026-02-11-phase1-mvp-chrome-extension.md** - Phase 1: MVP Demo (4-5 days)
2. **2026-02-11-phase2-v1-core-web-app.md** - Phase 2: V1 Core (Weeks 1-4)
3. **2026-02-11-phase3-v1-polish-automation.md** - Phase 3: V1 Polish (Weeks 5-8)
4. **2026-02-11-phase4-v1-launch-beta.md** - Phase 4: V1 Launch (Weeks 9-12)
5. **2026-02-11-phase5-v2-advanced-features.md** - Phase 5: V2 (Months 4-6)
6. **2026-02-11-phase6-v3-ecosystem.md** - Phase 6: V3 (Months 7-12)

For each phase plan:

**Structure:**
- Header with phase name, timeline, status
- Overview (what this phase accomplishes)
- Success Criteria (clear, measurable)
- Prerequisites (what must be done before starting)
- Detailed Implementation Steps (break down each sub-phase from main roadmap)
- Technical Stack & Tools (list specific libraries/APIs with references to ai/guides/ docs)
- File Structure (what files/directories will be created)
- Testing Strategy (how to verify it works)
- Risks & Mitigation
- Deliverables (concrete outputs)
- References (link to main roadmap, architecture docs, library guides)

**Key Requirements:**
- Be DETAILED - each task should have clear steps
- Reference library docs from ai/guides/ (e.g., "See [Next.js docs](../guides/nextjs_context7.md) for setup")
- Include code structure examples (file organization, not full code)
- Add decision points and alternatives where applicable
- Keep the "no over-engineering" philosophy from main roadmap
- Each plan should be actionable on its own (a developer could execute it)

**Important:**
- Phase 1 is Chrome extension (client-side only, no backend)
- Phases 2-4 are the "12 weeks to V1 launch" from PRD
- Phase 5-6 are future enhancements
- Use the library documentation we just created in ai/guides/

Make each plan comprehensive but focused on what's needed for THAT phase only. Don't include features from future phases.

---

## 2026-02-12 (continued)

Create roadmap tracking documents for each phase to pair with the existing detailed plan documents. These roadmaps should be shorter checklist/tracking docs that reference their corresponding plan.

Create these files in ai/roadmaps/:

1. **2026-02-11-roadmap-phase1-mvp.md** (pairs with 2026-02-11-phase1-mvp-chrome-extension.md)
2. **2026-02-11-roadmap-phase2-v1-core.md** (pairs with 2026-02-11-phase2-v1-core-web-app.md)
3. **2026-02-11-roadmap-phase3-v1-polish.md** (pairs with 2026-02-11-phase3-v1-polish-automation.md)
4. **2026-02-11-roadmap-phase4-v1-launch.md** (pairs with 2026-02-11-phase4-v1-launch-beta.md)
5. **2026-02-11-roadmap-phase5-v2.md** (pairs with 2026-02-11-phase5-v2-advanced-features.md)
6. **2026-02-11-roadmap-phase6-v3.md** (pairs with 2026-02-11-phase6-v3-ecosystem.md)

**Structure for each roadmap:**

```markdown
# Phase X Roadmap: [Name]

**Status:** Not Started / In Progress / Complete
**Timeline:** [timeline]
**Detailed Plan:** [link to corresponding phase plan doc]

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

[2-3 sentence summary of what this phase accomplishes]

---

## Task Checklist

### [Sub-phase 1]
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

### [Sub-phase 2]
- [ ] Task 1
- [ ] Task 2

[etc - extract high-level tasks from the detailed plan]

---

## Success Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

---

## Key Deliverables

- [ ] Deliverable 1
- [ ] Deliverable 2
- [ ] Deliverable 3

---

## Notes & Decisions

[Empty section for tracking decisions during implementation]

---

## Completion Checklist

Before moving to ai/roadmaps/complete:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] Deliverables created
- [ ] Tests passing
- [ ] Documentation updated
- [ ] Changelog updated

---

**Created:** 2026-02-11
**Last Updated:** 2026-02-11
**Next Phase:** [Link to next phase roadmap]
```

**Requirements:**
- Keep roadmaps concise (1-2 pages max)
- Use checkboxes for all actionable items
- Include clear reference to detailed plan doc
- Include the over-engineering warning prominently
- Make it easy to track progress at a glance
- Include completion checklist

**After creating all roadmaps:**
Also update each detailed PLAN doc to add a reference back to its roadmap at the top.

Add something like:
```markdown
**Roadmap:** [2026-02-11-roadmap-phaseX-xxx.md](./2026-02-11-roadmap-phaseX-xxx.md) - Track progress
```

---

