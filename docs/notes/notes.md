## Notes for improvement

/plan mode is awesome can @architect use /plan mode and write the /plan mode output to plan.md? 

agile updates to spec->research->plan->implementation is critical to success

/plan mode with /model claude-opus-4.6 helps a lot with these updates need to specify which files to look at 
validate the thinking with Gemini pro on the side!

## Upsides

We do have a historical record of how the system was built and what was taken into consideration
like a big context memory archive

Invest the time up front to clearly define what you want and how you want it built to significantly reduce implementation time while increasing overall quality

## Downsides

+ the farther down the sdlc you go the harder it is to make changes to the spec. This may be a feature and a bug. That is, the farther down the sdlc you go, the more solid the system gets. 



## Differences with other frameworks

### GSD

The "locked decisions" state file will be discarded. The existing spec.md template already captures historical context perfectly in the Open Questions section by moving items from [ ] to [x] ~~Resolved~~ → Decision made. Keeping decisions self-contained in the spec avoids workflow bloat, respects the current architecture, and maintains the spec as the single source of truth.

## Concept

Regarding automated self-validation, the workflow will instead use a highly collaborative, human-in-the-loop validation model. In real software development, agents must act as conversational partners rather than overconfident, autonomous gatekeepers that risk barreling forward on incorrect plans. The SDLC is agile; artifacts are living documents refined through continuous feedback. If the PM agent detects a potential misalignment between a requested feature and the VISION.md, it will not block execution or throw an automated error. Instead, it will gently raise the observation in chat (e.g., "I don't see how this aligns with the vision...") to prompt a collaborative discussion. The primary goal is to ensure the final spec strictly matches expectations without dropping important details, requiring explicit human approval before any phase is finalized.

