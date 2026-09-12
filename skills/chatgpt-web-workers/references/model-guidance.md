# Allocation and prompting rationale

Official sources were opened and checked on **2026-09-11**. These are model-behavior guidelines; API parameters described in the sources are not controls that can be activated by typing them into a ChatGPT prompt. The skill does not change the host model or guarantee subscription savings.

## Astra: host judgment and coordination

The [Astra model guide](https://developers.openai.com/api/docs/guides/latest-model) describes stronger end-to-end work, long-task coherence, and instruction following. It recommends explicitly stating when/how much to delegate, calibrating testing to the change, and preventing unnecessary pauses caused by ambiguous skill guidance.

Application here: keep task framing, cross-part dependencies, difficult judgments, required local execution, conflict adjudication, and final acceptance with the host. Give each work unit one owner; host ownership of the outcome does not require a second research or validation pass. Use complete worker results by default, return specific defects to their owner, and add checks only for concrete issues or governing requirements. Explicitly require 1–6 web workers when this skill is invoked. This allocation is an implementation judgment based on those behaviors, not an official benchmark of this particular workflow.

## Sol: capable workers with lean instructions

The [GPT-5.6/Sol guide](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6) recommends leaner prompts, stating each instruction once, and retaining domain context, hard constraints, autonomy boundaries, and success criteria. Sol already tends to be concise; generic demands for extreme brevity can omit required information. Complete deliverables and necessary evidence should survive compression.

Application here: assign substantial, bounded deliverables, including their relevant self-checks. Supply the materials that the worker can actually access. Describe what done means; allow Sol to choose the reasoning steps. Use a task-specific output format and follow up with deltas or concrete defects, without an automatic extra review round. Do not copy Astra's entire host guidance into each worker or request private chain-of-thought. The source's API recommendations to compare reasoning settings do not override the user's fixed **极高** requirement for this skill.

## Ordinary Chat and usage

[ChatGPT's official updates](https://learn.chatgpt.com/docs/whats-new) identify the ordinary Chat thinking slider with GPT-5.6 Sol and distinguish it from Work/Codex behavior. The user's operating rule is to accept the visible **极高** label without probing model identity. Never conflate a Pro subscription with selecting Pro reasoning.

[Official pricing and usage guidance](https://learn.chatgpt.com/docs/pricing) explains that prompt/history/tool results and outputs consume tokens, and recommends limiting unnecessary context, source material, and output. Therefore the savings strategy is to transfer useful work, reduce host-side orchestration and repeated context, and retain quality. Opening a hidden browser alone does not demonstrate savings. API model prices cannot establish the percentage saved from a user's subscription allowance.

Revisit these sources when the user requests a model-strategy update, the UI changes materially, or measured behavior contradicts the cached guidance. Do not spend a new research pass on every ordinary invocation.
