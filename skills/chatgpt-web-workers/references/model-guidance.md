# Allocation and prompting rationale

This is maintenance rationale, not an extra runtime checklist. The Astra article was rechecked on **2026-09-15**; Sol prompt sources were checked on **2026-09-14**, and earlier model/usage observations retain their **2026-09-11** check date. API documentation does not establish ordinary Chat settings or service limits.

## Skill design for Astra

The [Astra skill article](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) recommends precise short descriptions, task-based reference loading, and explicit completion boundaries rather than elaborate recipes. Applied here: the entrypoint owns workflow constraints and acceptance; transport owns communication mechanics; file instructions load only for transfer. Japan-trend-radar owns research standards and inherits orchestration. Keep short reminders for high-impact boundaries, with one detailed definition. Exact timeout and file routes are user requirements; UI failure details remain because they change operational decisions. Ordinary runs do not need this rationale or historical test results.

## Astra: host judgment and coordination

The [Astra model guide](https://developers.openai.com/api/docs/guides/latest-model) describes stronger end-to-end work, long-task coherence, and instruction following. It recommends explicitly stating when/how much to delegate, calibrating testing to the change, and preventing unnecessary pauses caused by ambiguous skill guidance.

The ownership and acceptance contract in SKILL.md is an implementation judgment based on this guidance, not an official benchmark of the workflow.

## Distribution policy

The 4–6 worker preference implements the user's request to distribute prompt burden; it is not an OpenAI concurrency recommendation. The user reports server-side failures with long single-chat prompts. Treat this as an operational concern, not proven causation or a documented character/token threshold. Preserve the six-chat total limit, coherent scopes, necessary shared context, and the current timeout stop defined in transport.md. Check coverage and completeness when evaluating a split; fewer input characters alone do not prove better results or fewer server errors.

The six-chat research audit showed that dividing topics/channels still left each initial prompt requesting discovery, verification, grading, selection, and self-checking for 30–75 candidates. Some correction prompts also bundled five or six defect groups. The user wants those operations separated while preserving the original search scale; smaller discovery quotas are not the intended optimization. The Astra article supports outcome boundaries and judgment rather than fixed recipes: the host chooses subsequent work against a durable final objective, while Sol returns a bounded stage. Independent continuation avoids a full-wave barrier; its performance and latency have not been live-benchmarked. Operational definitions belong only in SKILL.md and transport.md.

## Sol prompt sources

- [GPT-5.6/Sol model guidance](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6), “Prompting best practices”: lean instructions, relevant tools, explicit autonomy, task-specific response requirements, and attention to growing conversation context. Its advice to test prompt changes does not establish a safe prompt-length cap or require a fresh benchmark on every ordinary task.
- [Reasoning best practices](https://developers.openai.com/api/docs/guides/reasoning-best-practices), “How to prompt reasoning models effectively”: direct goals, clear input boundaries, selective examples, and no chain-of-thought prompting. This page includes older-model-specific material; use only these general prompting principles, not its historical model claims or API formatting switches.

The executable writing rules are in SKILL.md, not duplicated here. Apply them to Sol packets while keeping Astra's orchestration instructions at the host. API-only effort, verbosity, context, and Pro settings do not override the user's ordinary Chat / 极高 choice.

## Ordinary Chat and usage

[ChatGPT's official updates](https://learn.chatgpt.com/docs/whats-new) identify the ordinary Chat thinking slider with GPT-5.6 Sol and distinguish it from Work/Codex behavior. The user's operating rule is to accept the visible **极高** label without probing model identity. Never conflate a Pro subscription with selecting Pro reasoning.

[Official pricing and usage guidance](https://learn.chatgpt.com/docs/pricing) explains that prompt/history/tool results and outputs consume tokens, and recommends limiting unnecessary context, source material, and output. Therefore the savings strategy is to transfer useful work, reduce host-side orchestration and repeated context, and retain quality. Opening a hidden browser alone does not demonstrate savings. API model prices cannot establish the percentage saved from a user's subscription allowance.

Revisit these sources when the user requests a model-strategy update, the UI changes materially, or measured behavior contradicts the cached guidance. Do not spend a new research pass on every ordinary invocation.
