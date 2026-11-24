# Writing rules for safety and boundary text (ASD-STE100)

Read this before you write or change: the invariants in `SKILL.md`, `safety-boundaries.md`, the Workspace, Ownership and Rules sections of an assignment, and any WARNING, CAUTION or NOTE.

The rules below come from ASD-STE100 (Simplified Technical English, Issue 8). Rule numbers are omitted; the specification is the authority on exact text and on the approved dictionary. Where this workflow has no access to the dictionary, apply the principle: one word, one meaning; prefer the short common word.

## Words

- Use one term for one thing. Do not use two names for one thing. The glossary in `safety-boundaries.md` fixes the terms.
- Use a word only in its approved meaning. Use "can" for ability. Use "must" for a requirement. Do not use "may", "should", "shall", "might" or "could" in a rule.
- Use "if" for a condition. Use "when" for a time.
- Use "do not" for a prohibition. Do not use "never", "avoid", "refrain from".
- Use the article ("the", "a") when the noun needs one. Do not drop articles to make a sentence shorter.
- Do not use a cluster of more than three nouns. Write "the identity of the frozen candidate", not "frozen candidate content identity".
- Do not use slang, idioms, or metaphors ("fail open", "cheap", "burn tokens"). Say what happens.
- Keep technical names as they are (`PASEO`, `git`, `bun`, `tsconfig.json`, agent IDs, rule IDs). Put them in code font.

Common substitutions:

| Do not write | Write |
| --- | --- |
| may, might, could | can (ability) or must (requirement) |
| should, shall | must |
| ensure, verify (as a vague verb) | make sure that …, check that … |
| prior to | before |
| in order to | to |
| utilize | use |
| attempt | try |
| perform, carry out | do |
| commence, initiate | start |
| terminate | stop |
| sufficient | enough |
| it, this, that (without a clear noun) | the noun |

## Sentences

- One instruction per sentence.
- A procedural sentence has 20 words or fewer. A descriptive sentence has 25 words or fewer. Count them.
- A paragraph has six sentences or fewer. The first sentence gives the topic.
- Use the active voice. Name the actor: the coordinator, the implementer, the reviewer, the user.
- Use the imperative for an instruction: "Record the agent ID." Use the simple present for a fact: "The sandbox has no network."
- Do not use the -ing form of a verb, except in a technical name ("working tree" is allowed; "verifying the route" is not).
- Do not leave out words that the reader must guess (no ellipsis).
- Write a sequence as a numbered list. Write a set of conditions as a bulleted list. One item, one sentence.

## Safety instructions

Three levels. Use the level that matches the effect, not the one that sounds strongest.

| Level | Use for | Example effect |
| --- | --- | --- |
| **WARNING** | an action that cannot be undone, or that reaches outside the repository | merge into a shared branch, deploy, call a live system, send data to an external service, delete a worktree, expose a secret |
| **CAUTION** | an action that damages the run but can be repaired | a second writer on one candidate, a candidate changed after its freeze, a budget exceeded, a baseline regenerated with new entries |
| **NOTE** | information that helps, with no safety content | where a log is, how long a step takes |

Form of a WARNING or CAUTION:

1. Put it before the step it protects, not after.
2. Start with a clear and simple command, or with a condition followed by a command.
3. Give the reason in one sentence, if the reason is not obvious.
4. Make it stand out: the level in capitals, then a colon, then the text, on its own line.

```
WARNING: Do not merge into `integration`, `develop` or `main`. A merge cannot be undone.
CAUTION: Do not send a second assignment to a worker that has not finished. Two writers corrupt the candidate.
```

Do not put two dangers in one WARNING. Do not put a procedure step inside a NOTE.

## Checklist before you send boundary text

1. Every sentence has one instruction or one fact.
2. No sentence has more than 20 words (procedure) or 25 words (description).
3. No "may", "should", "shall", "might", "could".
4. Every pronoun has a clear noun in the same sentence.
5. Every prohibition starts with "Do not".
6. Every irreversible or external action has a WARNING before it.
7. Every term matches the glossary.
