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

