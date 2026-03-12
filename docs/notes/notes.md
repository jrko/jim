## Genearal Notes

The farther down the SDLC you go the harder it is to change the prior steps. This is true of any SDLC.

The more work done up front at the spec and research phase the less work needs to be done at the plan and code phase

## GOALS

should in theory support a small multi-person team working on a decent size monolith project, especially if developers focus on specific functional groupings

designed to track agile requirements changes and adjustments to implementation over time

scale to larger sized projects with multiple functional areas 

build upon learnings from v1 (what are the learnings from v1? manually running spec -> research -> plan feels cumbersome though it proves valuable can that be automated or streamlined? (v2 tries subagent merging research into plan); agile feedback critically important -- after an implementation is created during manual review we may find we need to adjust the implementation. Adjusting the implementation should adjust the spec that all needs to be done manually it's a pain. Bugfixes are like this too. Sometimes bugfix really results in what should have been a change to the plan to begin with. )
## NON GOALS

Not a true change management system. 

## Notes for future improvement

/plan mode is awesome can @architect use /plan mode and write the /plan mode output to plan.md? 

agile updates to spec->research->plan->implementation is critical to success

/plan mode with /model claude-opus-4.6 helps a lot with these updates need to specify which files to look at 
validate the thinking with Gemini pro on the side!

sometimes I feel that research should happen before spec sometimes the research is super important to define what to spec 

would be nice if we had synthesis of research (a research of all research most commonly requested info ex. FAQ's FOR AGENTS!!!)

any postmortem analysis after a feature group is shipped? could be nice for future learnings

## Upsides

We do have a historical record of how the system was built and what was taken into consideration
like a big context memory archive

Invest the time up front to clearly define what you want and how you want it built to significantly reduce implementation time while increasing overall quality

I've reference research from prior iterations in new iterations, especially as pm asks questions (ie. please see 00x research.md file has that info). That memory capture is super helpful. 



## Downsides

+ the farther down the sdlc you go the harder it is to make changes to the spec. This may be a feature and a bug. That is, the farther down the sdlc you go, the more solid the system gets. 

+ as coder I need to understand what's in the other iterations in some way to best manage the iterative development. 


## Differences with other frameworks

### GSD

The "locked decisions" state file will be discarded. The existing spec.md template already captures historical context perfectly in the Open Questions section by moving items from [ ] to [x] ~~Resolved~~ → Decision made. Keeping decisions self-contained in the spec avoids workflow bloat, respects the current architecture, and maintains the spec as the single source of truth.

### V1SDLC

v1sdlc was the first version of the agentic sdlc. jim is version 2 and provides significant upgrades over v1:

+ created jim: plugin for clearer namespacing and distribution
+ Kept core idea of grouped iterative development following spec -> research -> plan -> build phases
+ WORKFLOW combine reasearch and plan together into plan and have architect spawn researcher as subagent (let's see how that goes!)
+ More focus on AGILE SDC 
+ Switched to skills over commands per (claude code documentation about skills being available as commands)
+ Added (skills) skills responsible for (new artifacts)


## Concept

Regarding automated self-validation, the workflow will instead use a highly collaborative, human-in-the-loop validation model. In real software development, agents must act as conversational partners rather than overconfident, autonomous gatekeepers that risk barreling forward on incorrect plans. The SDLC is agile; artifacts are living documents refined through continuous feedback. If the PM agent detects a potential misalignment between a requested feature and the VISION.md, it will not block execution or throw an automated error. Instead, it will gently raise the observation in chat (e.g., "I don't see how this aligns with the vision...") to prompt a collaborative discussion. The primary goal is to ensure the final spec strictly matches expectations without dropping important details, requiring explicit human approval before any phase is finalized.


## Launch Party

Make a video with https://www.reddit.com/r/ClaudeAI/comments/1rr47ya/i_delayed_my_product_launch_for_months_because_i/ Remotion.dev ???

