---
title: "Enforcement Points for Agents"
abbrev: "draft-liu-audit-enforcement-points"
category: info

docname: draft-liu-audit-enforcement-points-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Agent Use of Delegation and Interaction Traceability"
keyword:

venue:
  group: "Agent Use of Delegation and Interaction Traceability"
  type: "Working Group"
  mail: "audit@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/audit"
  github: "NiYuan224/draft-liu-audit-enforcement-points"
  latest: "https://NiYuan224.github.io/draft-liu-audit-enforcement-points/draft-liu-audit-enforcement-points.html"

author:
 -
  name: Chunchi Peter Liu
  organization: Huawei
  email: liuchunchi@huawei.com

 -
  name: Yuan Ni
  organization: Huawei
  email: niyuan1@huawei.com

normative:
 RFC2119:
 RFC8174:

informative:
informative:
  ClaudeCodeHooks:
    title: "Claude Code Hooks"
    target: https://code.claude.com/docs/en/hooks

  CursorHooks:
    title: "Cursor Hooks"
    target: https://cursor.com/docs/hooks

  CopilotHooks:
    title: "GitHub Copilot Hooks Reference"
    target: https://docs.github.com/en/copilot/reference/hooks-reference

  QwenCodeHooks:
    title: "Qwen Code Hooks"
    target: https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/hooks.md

  GeminiCLIHooks:
    title: "Gemini CLI Hooks Reference"
    target: https://geminicli.com/docs/hooks/reference/

  HermesHooks:
    title: "Hermes Agent Hooks"
    target: https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks/

  OpenClawHooks:
    title: "OpenClaw Plugin Hooks"
    target: https://docs.openclaw.ai/plugins/hooks

  LiteLLMHooks:
    title: "LiteLLM Proxy Call Hooks"
    target: https://docs.litellm.ai/docs/proxy/call_hooks

  PortkeyHooks:
    title: "Portkey Guardrail Checks"
    target: https://docs.portkey.ai/docs/product/guardrails/list-of-guardrail-checks

  TrueFoundryHooks:
    title: "Configure Guardrails on the TrueFoundry AI Gateway"
    target: https://www.truefoundry.com/blog/configure-guardrails-ai-gateway

  BifrostHooks:
    title: "Bifrost Runtime Extensions"
    target: https://bifrost.boelabs.com/docs/extensions



--- abstract

This document identifies key enforcement points (EPs) for agents, specifying the agent activities they cover and the audit records they provide. It describes how hooks exposed by agent implementations enable EPs to observe agent activities, intervene in these activities or their subsequent processing, and generate corresponding audit records.


--- middle

# Introduction {#intro}


智能体审计需要获取运行过程中的信息，而这些信息主要由不同类型的执行点（EP）通过观测和干预提供。
Agents increasingly act on behalf of users across multiple services and administrative domains. Auditing their activities requires runtime information, primarily provided by enforcement points (EPs) that observe and, where supported, intervene in agent activities. These EPs are distributed across the interaction chain, each covering a different set of activities and providing corresponding audit records.

案例

由于不同执行点覆盖的活动范围不同，需要明确它们能够提供哪些审计信息，以及各自的局限性。
This document considers three categories of EPs: Agent Client EPs, Gateway EPs, and Host EPs. For each category, it describes the audit information EPs can provide and their limitations, providing a basis for identifying the records needed across participating domains and understanding the coverage of the available evidence.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Enforcement Point (EP): A logical entity or network element that observe agent activities, enforces security controls, and generates corresponding audit records.

Audit Record: A record of the observable behavior of agents, including user intent, agent or tool interactions, invoked or declined actions, exchanged data, and resulting outcomes across trust boundaries.

Hook: A mechanism exposed by software or an operating system at a defined execution point to enable observation and, where supported, intervention.

# Agent Workflows and Enforcement Points

Figure 1 illustrates a typical agent workflow and the and the locations of the three EP categories.

~~~
+---------+      +-------+     +------+-(2) +--------+    +------+
|         |      |       |     |      |-----|        +--->| LLM  |
|         |      |       |     |      |<----|        +----|      |
|         | (1)  |       |     |      | (3) |        |    +------+
|         |------+       +----->      |     |        |
|         |      |       |     |      | (4) |        |    +------+
|   User  |      |Gateway|     | Agent|-----|Gateway +--->|Tools |
| /System |      |  EP   |     |(Agent|<----|   EP   +----|      |
|         |<-----+Inbound+-----|Client| (5) |Outbound|    +------+
|         | (10) |       |     |  EP) |     |        |
|         |      |       |     |      | (6) |        |    +------+
|         |      |       |     |      |-----|        +--->|Other |
|         |      |       |     |      |<----|        +----|Agents|
+---------+      +-------+     +--+-^-- (7) +--------+    +------+
                                  | |
                               (8)| |(9)
+---------------------------------v-------------------------------+
|                           Host (Host EP)                        |
+-----------------------------------------------------------------+
~~~
*Figure 1. Agent Workflow and EPs*

1. (Optional) The User or System provides a request or instruction to the Agent.
2. The Agent provides context to the LLM.
3. The LLM returns output to the Agent to support the selection of subsequent actions.
4. The Agent invokes one or more tools.
5. The tools return results to the Agent. The Agent may provide these results as additional context to the LLM and repeat steps 2-5 as needed.
6. (Optional) The Agent delegates tasks to or exchanges requests with other agents.
7. (Optional) Other Agents return responses or task execution results to the Agent.
8. (Optional) The Agent performs operations on the host.
8. (Optional) The host provides corresponding results, status, or errors.
9. (Optional) The Agent returns a response to the User or System, or requests additional input.

EPs provide visibility into and control over the above agent activities. They may be located within the agent client, at communication gateways, or on the host where the agent runs. Accordingly, EPs are classified into three categories: Agent Client EPs, Gateway EPs, and Host EPs.

* Agent Client EPs: EPs integrated into the agent client, using hooks to observe and control agent activities, such as user interactions, model requests, tool calls, and host operations.

* Gateway EPs: EPs deployed in the inbound communication paths between users/systems and agents, and the outbound communication paths between agents and models, tools, and other agents. They inspect and control requests and responses passing through the gateway.

* Host EPs: EPs running on the hosts or endpoints where agents execute. They monitor and control agent-related processes, command execution, file access, and other system operations.

Each category provides audit records within its observation and intervention scope. Their coverage may overlap, but no single EP necessarily observes the entire interaction chain.


# Agent Client EP

Agent Client EPs use client-provided hooks to observe critical agent events, enforce controls where supported, and generate audit records.

## Summary of Agent Client EP Hooks

The following table lists selected hook points and representative implementations, ordered by the agent's workflow sequence.

~~~
+---------------------------+---------------------------------------------+
| Name                      | Examples                                    |
+---------------------------+---------------------------------------------+
| Session Start             | Claude Code: `SessionStart`                 |
|                           | Cursor: `sessionStart`                      |
|                           | GitHub Copilot: `sessionStart`              |
|                           | Hermes Agent: `on_session_start`            |
|                           | OpenClaw: `session_start`                   |
|                           | Qwen Code: `SessionStart`                   |
+---------------------------+---------------------------------------------+
| Before Prompt Submit      | Claude Code: `UserPromptSubmit`             |
|                           | Cursor: `beforeSubmitPrompt`                |
|                           | Gemini CLI: `BeforeAgent`                   |
|                           | GitHub Copilot: `userPromptSubmitted`       |
|                           | Qwen Code: `UserPromptSubmit`               |
+---------------------------+---------------------------------------------+
| Prompt Expansion          | Claude Code: `UserPromptExpansion`          |
|                           | Qwen Code: `UserPromptExpansion`            |
+---------------------------+---------------------------------------------+
| Before Model Request      | Gemini CLI: `BeforeModel`                   |
|                           | Hermes Agent: `pre_api_request`             |
|                           | OpenClaw: `llm_input`                       |
+---------------------------+---------------------------------------------+
| After Model Response      | Gemini CLI: `AfterModel`                    |
|                           | Hermes Agent: `post_api_request`            |
|                           | OpenClaw: `llm_output`                      |
+---------------------------+---------------------------------------------+
| Before Context Compaction | Claude Code: `PreCompact`                   |
|                           | Cursor: `preCompact`                        |
|                           | Gemini CLI: `PreCompress`                   |
|                           | GitHub Copilot: `preCompact`                |
|                           | OpenClaw: `before_compaction`               |
|                           | Qwen Code: `PreCompact`                     |
+---------------------------+---------------------------------------------+
| After Context Compaction  | Claude Code: `PostCompact`                  |
|                           | Gemini CLI: `PostCompress`                  |
|                           | OpenClaw: `after_compaction`                |
|                           | Qwen Code: `PostCompact`                    |
+---------------------------+---------------------------------------------+
| Subagent Start            | Claude Code: `SubagentStart`                |
|                           | Cursor: `subagentStart`                     |
|                           | GitHub Copilot: `subagentStart`             |
|                           | Hermes Agent: `subagent_start`              |
|                           | OpenClaw: `subagent_spawned`                |
|                           | Qwen Code: `SubagentStart`                  |
+---------------------------+---------------------------------------------+
| Subagent Completion       | Claude Code: `SubagentStop`                 |
|                           | Cursor: `subagentStop`                      |
|                           | GitHub Copilot: `subagentStop`              |
|                           | Hermes Agent: `subagent_stop`               |
|                           | OpenClaw: `subagent_ended`                  |
|                           | Qwen Code: `SubagentStop`                   |
+---------------------------+---------------------------------------------+
| Before Tool Use           | Claude Code: `PreToolUse`                   |
|                           | Cursor: `preToolUse`                        |
|                           | Gemini CLI: `BeforeTool`                    |
|                           | GitHub Copilot: `preToolUse`                |
|                           | Hermes Agent: `pre_tool_call`               |
|                           | OpenClaw: `before_tool_call`                |
|                           | Qwen Code: `PreToolUse`                     |
+---------------------------+---------------------------------------------+
| Permission Request        | Claude Code: `PermissionRequest`            |
|                           | Hermes Agent: `pre_approval_request`        |
|                           | Hermes Agent: `post_approval_response`      |
|                           | Qwen Code: `PermissionRequest`              |
+---------------------------+---------------------------------------------+
| Permission Denied         | Claude Code: `PermissionDenied`             |
|                           | Qwen Code: `PermissionDenied`               |
+---------------------------+---------------------------------------------+
| After tool Use            | Claude Code: `PostToolUse`                  |
|                           | Cursor: `postToolUse`                       |
|                           | Gemini CLI: `AfterTool`                     |
|                           | GitHub Copilot: `postToolUse`               |
|                           | Hermes Agent: `post_tool_call`              |
|                           | OpenClaw: `after_tool_call`                 |
|                           | Qwen Code: `PostToolUse`                    |
+---------------------------+---------------------------------------------+
| Tool Batch Completion     | Claude Code: `PostToolBatch`                |
|                           | Qwen Code: `PostToolBatch`                  |
+---------------------------+---------------------------------------------+
| Post Tool Use Failure     | Claude Code: `PostToolUseFailure`           |
|                           | Cursor: `postToolUseFailure`                |
|                           | GitHub Copilot: `postToolUseFailure`        |
|                           | Qwen Code: `PostToolUseFailure`             |
+---------------------------+---------------------------------------------+
| Before MCP Execution      | Claude Code: `PreToolUse` (MCP tools)       |
|                           | Cursor: `beforeMCPExecution`                |
+---------------------------+---------------------------------------------+
| After MCP Execution       | Claude Code: `PostToolUse` (MCP tools)      |
|                           | Cursor: `afterMCPExecution`                 |
+---------------------------+---------------------------------------------+
| Before Command Execution  | Claude Code: `PreToolUse` (Bash)            |
|                           | Cursor: `beforeShellExecution`              |
+---------------------------+---------------------------------------------+
| After Command Execution   | Claude Code: `PostToolUse` (Bash)           |
|                           | Cursor: `afterShellExecution`               |
+---------------------------+---------------------------------------------+
| Before File Access        | Claude Code: `PreToolUse` (Read)            |
|                           | Cursor: `beforeReadFile`                    |
|                           | Cursor: `beforeTabFileRead`                 |
+---------------------------+---------------------------------------------+
| After File Modification   | Claude Code: `PostToolUse` (Write/Edit)     |
|                           | Cursor: `afterFileEdit`                     |
|                           | Cursor: `afterTabFileEdit`                  |
+---------------------------+---------------------------------------------+
| Agent Completion          | Claude Code: `Stop`                         |
|                           | Cursor: `stop`                              |
|                           | Gemini CLI: `AfterAgent`                    |
|                           | GitHub Copilot: `agentStop`                 |
|                           | Hermes Agent: `post_llm_call`               |
|                           | OpenClaw: `agent_end`                       |
+---------------------------+---------------------------------------------+
| Session End               | Claude Code: `SessionEnd`                   |
|                           | Cursor: `sessionEnd`                        |
|                           | GitHub Copilot: `sessionEnd`                |
|                           | Hermes Agent: `on_session_finalize`         |
|                           | OpenClaw: `session_end`                     |
|                           | Qwen Code: `SessionEnd`                     |
+---------------------------+---------------------------------------------+
Note: Hooks for which consensus has not been reached are omitted.
~~~
*Table 1. Agent Client Hooks and Examples*

## Critical Event and Corresponding Audit Record Claims

For each event in Table 1, this section describes observable information, supported interventions, and REQUIRED or OPTIONAL audit record claims.

1. **Session Start:** Expose session information, including the session identifier and source. Allow setting session-specific environment variables or injecting initial security context.
* `session_id`: REQUIRED. A string identifying the current session.
* `source`: OPTIONAL. A string indicating how the session started, such as "startup" or "resume".

2. **Prompt Submission:** Expose the submitted prompt and associated session information. Allow inspecting, modifying, or blocking the prompt before agent processing.
* `session_id`: REQUIRED. A string identifying the current session.
* `prompt`: REQUIRED. A string containing the submitted user prompt.

3. **Prompt Expansion:** Expose prompt and command information during prompt expansion. Allow adding context or blocking expansion or submission.
* `session_id`: REQUIRED. A string identifying the current session.
* `command_name`: REQUIRED. A string identifying the command being expanded.
* `command_args`: OPTIONAL. A string containing argument text after the command name.
* `prompt`: OPTIONAL. A string containing the original input or expanded prompt, depending on the implementation.

4. **Before Model Request:** Expose model request information, including the selected model, provider, and request messages. Allow modifying or blocking model requests before invocation.
* `session_id`: REQUIRED. A string identifying the current session.
* `model`: REQUIRED. A string identifying the model selected for the request.
* `provider`: OPTIONAL. A string identifying the model provider.
* `messages`: OPTIONAL. An array of message objects included in the model request.   

5. **After Model Response:** Expose generated content, response metadata, and token usage. Allow modifying, replacing, or blocking the response before subsequent processing.
* `session_id`: REQUIRED. A string identifying the current session.
* `model`: REQUIRED. A string identifying the model associated with the request.
* `response`: OPTIONAL. An object containing the model response information exposed by the hook.
* `finish_reason`: OPTIONAL. A string indicating why response generation ended.
* `usage`: OPTIONAL. An object containing token usage information.


6. **Before Context Compaction:** Expose session information and the compaction trigger. Allow blocking compaction.
* `session_id`: REQUIRED. A string identifying the current session.
* `trigger`: REQUIRED. A string indicating whether compaction was triggered automatically (`"auto"`) or manually (`"manual"`).

7. **After Context Compaction:** Expose session information, the compaction trigger, and the generated summary.

* `session_id`: REQUIRED. A string identifying the current session.
* `trigger`: REQUIRED. A string indicating whether compaction was triggered automatically (`"auto"`) or manually (`"manual"`).
* `compact_summary`: OPTIONAL. A string containing the conversation summary generated by compaction.

8.  **Subagent Start:** Expose session information and the subagent identifier and type. Allow injecting additional context or blocking subagent creation.

* `session_id`: REQUIRED. A string identifying the session associated with the event.
* `agent_id`: REQUIRED. A string identifying the subagent.
* `agent_type`: REQUIRED. A string indicating the subagent type.

9. **Subagent Completion:** Expose subagent information and its final response. Allow requesting continued execution.

* `agent_id`: REQUIRED. A string identifying the subagent.
* `agent_type`: REQUIRED. A string indicating the subagent type.
* `last_assistant_message`: OPTIONAL. A string containing the subagent's last message.


10. **Before Tool Use:** Expose the tool name, invocation arguments, and tool call identifier. Allow approving, denying, or modifying tool invocations before execution.

* `tool_name`: REQUIRED. A string identifying the tool to be invoked.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.
* `tool_use_id`: OPTIONAL. A string identifying the tool invocation.

11. **Permission Request:** Expose the tool name and invocation arguments associated with a permission request. Allow approving or denying the requested operation.

* `tool_name`: REQUIRED. A string identifying the tool requesting permission.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.


12. **Permission Denied:** Expose the tool name, invocation arguments, and reason for a denied operation.

* `tool_name`: REQUIRED. A string identifying the tool whose invocation was denied.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.
* `reason`: OPTIONAL. A string indicating why the invocation was denied.

13. **After Tool Use:** Expose the tool name, invocation arguments, and execution result. Allow modifying or withholding the result before subsequent model processing, where supported.

* `tool_name`: REQUIRED. A string identifying the invoked tool.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.
* `tool_response`: OPTIONAL. The tool execution result. Depending on the tool, this may be a string containing file content, an object containing command output and exit status, or an array of query results.

* **Tool Batch Completion:** Expose the tool calls in a completed batch, including their arguments and results.

* `tool_calls`: REQUIRED. An array of objects describing the tool calls in the batch. Each object contains:
  * `tool_name`: REQUIRED. A string identifying the invoked tool.
  * `tool_input`: REQUIRED. An object containing the tool's input parameters.
  * `tool_use_id`: REQUIRED. A string identifying the tool invocation.
  * `tool_response`: OPTIONAL. The tool execution result, represented as a string, array, or object depending on the implementation.

* **Post Tool Use Failure:** Expose failed tool invocation information, including the tool name, invocation arguments, error details, and failure status.

* **Before MCP Execution:** Expose MCP invocation information, including the MCP server, tool name, and invocation arguments. Allow EPs to approve or deny MCP tool execution before invocation.

* **After MCP Execution:** Expose MCP execution information, including the MCP server, tool name, and execution results. Allow EPs to modify or replace MCP results before subsequent processing.

* **Before Command Execution:** Expose command execution information, including the command and working directory. Allow EPs to permit, deny, or request user confirmation before command execution.

* **After Command Execution:** Expose command execution results, including the executed command, captured output, exit status, and execution duration.

* **Before File Access:** Expose file access information, including the file path and requested file content. Allow EPs to permit or deny file access before the content is provided to the agent.

* **After File Modification:** Expose file modification information, including the file path and changes made, such as the original and modified text.

* **Agent Completion:** Expose agent completion information, including the completion status, stop reason, and final assistant response. Allow EPs to request additional processing or continued execution before the agent terminates.

* **Session End:** Expose session termination information, including the session identifier, termination reason, and session duration.

# Gateway EP

## Summary of Gateway EP Hooks

~~~
+-----------------------------+--------------------------------------------------+
| Name                        | Examples                                         |
+-----------------------------+--------------------------------------------------+
| Model Listing               | LiteLLM: `async_filter_listed_models`            |
+-----------------------------+--------------------------------------------------+
| Before Model Request        | LiteLLM: `async_pre_call_hook`                   |
|                             | Portkey: `beforeRequestHook`                     |
|                             | TrueFoundry: `LLM Input`                         |
|                             | Bifrost: `onCanonicalRequest`                    |
+-----------------------------+--------------------------------------------------+
| During Model Request        | LiteLLM: `async_moderation_hook`                 |
+-----------------------------+--------------------------------------------------+
| After Model Response        | LiteLLM: `async_post_call_success_hook`          |
|                             | LiteLLM: `async_post_call_streaming_hook`        |
|                             | LiteLLM: `async_post_call_streaming_iterator_    |
|                             |           hook`                                  |
|                             | Portkey: `afterRequestHook`                      |
|                             | TrueFoundry: `LLM Output`                        |
|                             | Bifrost: `onCanonicalResponse`                   |
|                             | Bifrost: `onStreamEvent`                         |
+-----------------------------+--------------------------------------------------+
| During Streaming Response   |                                                  |
+-----------------------------+--------------------------------------------------+
| Response Headers            | LiteLLM: `async_post_call_response_headers_hook` |
+-----------------------------+--------------------------------------------------+
| Request Failure             | LiteLLM: `async_post_call_failure_hook`          |
|                             | Bifrost: `onError`                               |
+-----------------------------+--------------------------------------------------+
| Before MCP Tool Invocation  | TrueFoundry: `MCP Tool Pre-Invoke`               |
+-----------------------------+--------------------------------------------------+
| After MCP Tool Invocation   | TrueFoundry: `MCP Tool Post-Invoke`              |
+-----------------------------+--------------------------------------------------+
~~~

## Observability and Interventability



# Hook Failure Handling

Hook failures can affect security enforcement and audit coverage. Two common handling policies are:

* Fail-open: Allow the operation or subsequent processing to continue when a hook fails. This preserves availability but may bypass security checks.

* Fail-closed: Block the operation or subsequent processing when a hook fails. This prevents unchecked operations but may interrupt legitimate workflows.

Based on existing implementation practices, such as OpenClaw, Cursor, and Claude Code, the choice depends on the hook's purpose and the consequences of failure:

* Security-critical hooks SHOULD fail closed: These include hooks enforcing input security checks, model request inspection, tool invocation, permission decisions, sensitive file access, and extension installation. Failure to perform required checks may lead to unauthorized operations or sensitive information exposure.

* Observation-only hooks SHOULD generally fail open: These include hooks used only for session lifecycle monitoring, invocation completion logging, context compaction monitoring, command execution result collection, and file modification auditing. Failure to collect audit information should not normally interrupt legitimate operations.

# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
