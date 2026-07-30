# UX Copy

Vague copy creates support tickets and abandonment. Specific copy gets users through the task. Good UX writing is invisible — users understand immediately without noticing the words.

## Five rules

1. **Be specific**: "Enter email" not "Enter value"
2. **Be concise**: cut unnecessary words without sacrificing clarity
3. **Be active**: "Save changes" not "Changes will be saved"
4. **Be human**: "Oops, something went wrong" not "System error encountered"
5. **Tell users what to do**, not just what happened

## Button labels: never "OK"/"Submit"/"Yes"

Use verb + object, outcome-focused:

| Bad | Good | Why |
|---|---|---|
| OK | Save changes | Says what will happen |
| Submit | Create account | Outcome-focused |
| Yes | Delete message | Confirms the action |
| Cancel | Keep editing | Clarifies what "cancel" means |
| Click here | Download PDF | Describes the destination |

Destructive actions name the destruction: "Delete" not "Remove" (delete is permanent), and show the count: "Delete 5 items" not "Delete selected".

## Error messages: the formula

Answer three things — what happened, why, how to fix it. "Email address isn't valid. Please include an @ symbol." not "Invalid input". Never blame the user: "Please enter a date in MM/DD/YYYY format" not "You entered an invalid date".

| Situation | Template |
|---|---|
| Format error | "[Field] needs to be [format]. Example: [example]" |
| Missing required | "Please enter [what's missing]" |
| Permission denied | "You don't have access to [thing]. [What to do instead]" |
| Network error | "We couldn't reach [thing]. Check your connection and [action]." |
| Server error | "Something went wrong on our end. We're looking into it. [Alternative action]" |

## Empty states are onboarding moments

Acknowledge briefly, explain the value of filling it, give a clear action: "No projects yet. Create your first project to get started." — never just "No items".

## Loading states

Be specific about what's happening and how long: "Analyzing your data... this usually takes 30-60 seconds" not "Loading...". For long waits, show progress or set expectations explicitly.

Avoid whimsical AI-default loading lines ("Herding pixels...", "Teaching robots to dance...") — they're generic filler that says nothing about this product. Write copy specific to what's actually happening: "Compressing your video" beats "Just a moment...".

## Confirmation dialogs: use sparingly

Most confirmation dialogs are a design failure — consider undo instead (see motion-interaction.md). When you must confirm: state the specific action, explain consequences, use specific button labels ("Delete project" / "Keep project", never "Yes"/"No").

## Voice vs tone

Voice is the brand's consistent personality. Tone adapts to the moment:

| Moment | Tone |
|---|---|
| Success | Celebratory, brief: "Done! Your changes are live." |
| Error | Empathetic, helpful: "That didn't work. Here's what to try..." |
| Loading | Reassuring: "Saving your work..." |
| Destructive confirm | Serious, clear: "Delete this project? This can't be undone." |

Never use humor for errors — users are already frustrated.

## Terminology consistency

Pick one term and use it everywhere — variety creates confusion, not delight:

| Inconsistent | Consistent |
|---|---|
| Delete / Remove / Trash | Delete |
| Settings / Preferences / Options | Settings |
| Sign in / Log in / Enter | Sign in |
| Create / Add / New | Create |

## Accessibility & i18n

- Link text needs standalone meaning: "View pricing plans" not "Click here"
- Alt text describes information, not the image: "Revenue increased 40% in Q4" not "Chart"; `alt=""` for decorative images
- Icon buttons need `aria-label`
- Plan for expansion: German +30%, French +20%, Finnish +30-40%, Chinese -30% (fewer chars, same visual width)
- Keep numbers as separate tokens ("New messages: 3" not "You have 3 new messages") — word order varies by language
- Avoid abbreviations ("5 minutes ago" not "5 mins ago")

**Never**: jargon without explanation, blaming users, vague errors ("Something went wrong" with no detail), varying terminology for variety, redundant copy (if the heading explains it, the intro is dead weight).
