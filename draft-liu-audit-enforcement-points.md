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

  HooksSecurity:
    title: "HooksSecurity"
    target: https://hooks.security/

  LiteLLMLogging:
    title: "LiteLLM StandardLoggingPayload Specification"
    target: https://docs.litellm.ai/docs/proxy/logging_spec

  DefenderEDR:
    title: "Overview of Endpoint Detection and Response"
    target: https://learn.microsoft.com/en-us/defender-endpoint/overview-endpoint-detection-response






--- abstract

Agent auditing relies on distributed enforcement points (EPs) to observe and intervene in agent activities. This document analyzes Agent Client, Gateway, and Host EPs by detailing their coverage and audit data. This helps to define the specific capabilities of different EPs and evaluate the overall evidence coverage.

--- middle

# Introduction {#intro}


Agents increasingly act on behalf of users across multiple services and administrative domains. Auditing their activities requires runtime information, primarily provided by enforcement points (EPs) that observe and, where supported, intervene in agent activities. These EPs are distributed across the interaction chain, each covering a different set of activities and providing corresponding audit records.

To support runtime monitoring and auditing, industry solutions record events at the client, gateway, and host layers: client hooks expose agent lifecycle events and tool calls {{ClaudeCodeHooks}}{{CursorHooks}}{{HooksSecurity}}; gateways inspect requests and responses and record access decisions and content filtering results{{LiteLLMLogging}}; host security tools monitor process execution, file access, and network connections{{DefenderEDR}}.

This document considers three categories of EPs: Agent Client EPs, Gateway EPs, and Host EPs. For each category, it describes the audit information EPs can provide, providing a basis for identifying the records across participating domains and understanding the coverage of the available evidence.

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
+--------------------------+----------------------------------------+
| Name                     | Examples                               |
+--------------------------+----------------------------------------+
| Session Start            | Claude Code: `SessionStart`            |
|                          | Cursor: `sessionStart`                 |
|                          | GitHub Copilot: `sessionStart`         |
|                          | Hermes Agent: `on_session_start`       |
|                          | OpenClaw: `session_start`              |
|                          | Qwen Code: `SessionStart`              |
+--------------------------+----------------------------------------+
| Before Prompt Submit     | Claude Code: `UserPromptSubmit`        |
|                          | Cursor: `beforeSubmitPrompt`           |
|                          | Gemini CLI: `BeforeAgent`              |
|                          | GitHub Copilot: `userPromptSubmitted`  |
|                          | Qwen Code: `UserPromptSubmit`          |
+--------------------------+----------------------------------------+
| Prompt Expansion         | Claude Code: `UserPromptExpansion`     |
|                          | Qwen Code: `UserPromptExpansion`       |
+--------------------------+----------------------------------------+
| Before Model Request     | Gemini CLI: `BeforeModel`              |
|                          | Hermes Agent: `pre_api_request`        |
|                          | OpenClaw: `llm_input`                  |
+--------------------------+----------------------------------------+
| After Model Response     | Gemini CLI: `AfterModel`               |
|                          | Hermes Agent: `post_api_request`       |
|                          | OpenClaw: `llm_output`                 |
+--------------------------+----------------------------------------+
| Before Context Compaction| Claude Code: `PreCompact`              |
|                          | Cursor: `preCompact`                   |
|                          | Gemini CLI: `PreCompress`              |
|                          | GitHub Copilot: `preCompact`           |
|                          | OpenClaw: `before_compaction`          |
|                          | Qwen Code: `PreCompact`                |
+--------------------------+----------------------------------------+
| After Context Compaction | Claude Code: `PostCompact`             |
|                          | Gemini CLI: `PostCompress`             |
|                          | OpenClaw: `after_compaction`           |
|                          | Qwen Code: `PostCompact`               |
+--------------------------+----------------------------------------+
| Subagent Start           | Claude Code: `SubagentStart`           |
|                          | Cursor: `subagentStart`                |
|                          | GitHub Copilot: `subagentStart`        |
|                          | Hermes Agent: `subagent_start`         |
|                          | OpenClaw: `subagent_spawned`           |
|                          | Qwen Code: `SubagentStart`             |
+--------------------------+----------------------------------------+
| Subagent Completion      | Claude Code: `SubagentStop`            |
|                          | Cursor: `subagentStop`                 |
|                          | GitHub Copilot: `subagentStop`         |
|                          | Hermes Agent: `subagent_stop`          |
|                          | OpenClaw: `subagent_ended`             |
|                          | Qwen Code: `SubagentStop`              |
+--------------------------+----------------------------------------+
| Before Tool Use          | Claude Code: `PreToolUse`              |
|                          | Cursor: `preToolUse`                   |
|                          | Gemini CLI: `BeforeTool`               |
|                          | GitHub Copilot: `preToolUse`           |
|                          | Hermes Agent: `pre_tool_call`          |
|                          | OpenClaw: `before_tool_call`           |
|                          | Qwen Code: `PreToolUse`                |
+--------------------------+----------------------------------------+
| Permission Request       | Claude Code: `PermissionRequest`       |
|                          | Hermes Agent: `pre_approval_request`   |
|                          | Hermes Agent: `post_approval_response` |
|                          | Qwen Code: `PermissionRequest`         |
+--------------------------+----------------------------------------+
| Permission Denied        | Claude Code: `PermissionDenied`        |
|                          | Qwen Code: `PermissionDenied`          |
+--------------------------+----------------------------------------+
| After tool Use           | Claude Code: `PostToolUse`             |
|                          | Cursor: `postToolUse`                  |
|                          | Gemini CLI: `AfterTool`                |
|                          | GitHub Copilot: `postToolUse`          |
|                          | Hermes Agent: `post_tool_call`         |
|                          | OpenClaw: `after_tool_call`            |
|                          | Qwen Code: `PostToolUse`               |
+--------------------------+----------------------------------------+
| Tool Batch Completion    | Claude Code: `PostToolBatch`           |
|                          | Qwen Code: `PostToolBatch`             |
+--------------------------+----------------------------------------+
| Post Tool Use Failure    | Claude Code: `PostToolUseFailure`      |
|                          | Cursor: `postToolUseFailure`           |
|                          | GitHub Copilot: `postToolUseFailure`   |
|                          | Qwen Code: `PostToolUseFailure`        |
+--------------------------+----------------------------------------+
| Before MCP Execution     | Claude Code: `PreToolUse` (MCP tools)  |
|                          | Cursor: `beforeMCPExecution`           |
+--------------------------+----------------------------------------+
| After MCP Execution      | Claude Code: `PostToolUse` (MCP tools) |
|                          | Cursor: `afterMCPExecution`            |
+--------------------------+----------------------------------------+
| Before Command Execution | Claude Code: `PreToolUse` (Bash)       |
|                          | Cursor: `beforeShellExecution`         |
+--------------------------+----------------------------------------+
| After Command Execution  | Claude Code: `PostToolUse` (Bash)      |
|                          | Cursor: `afterShellExecution`          |
+--------------------------+----------------------------------------+
| Before File Access       | Claude Code: `PreToolUse` (Read)       |
|                          | Cursor: `beforeReadFile`               |
|                          | Cursor: `beforeTabFileRead`            |
+--------------------------+----------------------------------------+
| After File Modification  | Claude Code: `PostToolUse` (Write/Edit)|
|                          | Cursor: `afterFileEdit`                |
|                          | Cursor: `afterTabFileEdit`             |
+--------------------------+----------------------------------------+
| Agent Completion         | Claude Code: `Stop`                    |
|                          | Cursor: `stop`                         |
|                          | Gemini CLI: `AfterAgent`               |
|                          | GitHub Copilot: `agentStop`            |
|                          | Hermes Agent: `post_llm_call`          |
|                          | OpenClaw: `agent_end`                  |
+--------------------------+----------------------------------------+
| Session End              | Claude Code: `SessionEnd`              |
|                          | Cursor: `sessionEnd`                   |
|                          | GitHub Copilot: `sessionEnd`           |
|                          | Hermes Agent: `on_session_finalize`    |
|                          | OpenClaw: `session_end`                |
|                          | Qwen Code: `SessionEnd`                |
+--------------------------+----------------------------------------+
Note: Hooks for which consensus has not been reached are omitted.
~~~
*Table 1. Agent Client EP Hooks and Examples*

## Critical Events and Corresponding Audit Record Schema

For each event in Table 1, this section describes observable information, supported interventions, and REQUIRED or OPTIONAL audit record claims.

The following claims apply to all events:
* `session_id`: REQUIRED. A string identifying the current session.

In addition to the common claim above, each event defines its own audit record claims to capture event-specific information.

1. **Session Start:** Expose session information, including the session identifier and source. Allow setting session-specific environment variables or injecting initial security context.
* `source`: OPTIONAL. A string indicating how the session started, such as "startup" or "resume".

2. **Before Prompt Submit:** Expose the submitted prompt and associated session information. Allow inspecting, modifying, or blocking the prompt before agent processing.
* `prompt`: REQUIRED. A string containing the submitted user prompt.

3. **Prompt Expansion:** Expose prompt and command information during prompt expansion. Allow adding context or blocking expansion or submission.
* `command_name`: REQUIRED. A string identifying the command being expanded.
* `command_args`: OPTIONAL. A string containing argument text after the command name.
* `prompt`: OPTIONAL. A string containing the original input or expanded prompt, depending on the implementation.

4. **Before Model Request:** Expose model request information, including the selected model, provider, and request messages. Allow modifying or blocking model requests before invocation.
* `model`: REQUIRED. A string identifying the model selected for the request.
* `provider`: OPTIONAL. A string identifying the model provider.
* `messages`: OPTIONAL. An array of message objects included in the model request.

5. **After Model Response:** Expose generated content, response metadata, and token usage. Allow modifying, replacing, or blocking the response before subsequent processing.
* `model`: REQUIRED. A string identifying the model associated with the request.
* `response`: OPTIONAL. An object containing the model response information exposed by the hook.
* `finish_reason`: OPTIONAL. A string indicating why response generation ended.
* `usage`: OPTIONAL. An object containing token usage information.


6. **Before Context Compaction:** Expose session information and the compaction trigger. Allow blocking compaction.
* `trigger`: REQUIRED. A string indicating whether compaction was triggered automatically (`"auto"`) or manually (`"manual"`).

7. **After Context Compaction:** Expose session information, the compaction trigger, and the generated summary.

* `trigger`: REQUIRED. A string indicating whether compaction was triggered automatically (`"auto"`) or manually (`"manual"`).
* `compact_summary`: OPTIONAL. A string containing the conversation summary generated by compaction.

8.  **Subagent Start:** Expose session information and the subagent identifier and type. Allow injecting additional context or blocking subagent creation.

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

13. **After Tool Use:** Expose the tool name, invocation arguments, and execution result. Allow modifying or withholding the result before subsequent model processing.

* `tool_name`: REQUIRED. A string identifying the invoked tool.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.
* `tool_response`: OPTIONAL. The tool execution result. Depending on the tool, this may be a string containing file content, an object containing command output and exit status, or an array of query results.

14. **Tool Batch Completion:** Expose the tool calls in a completed batch, including their arguments and results.
* `tool_calls`: REQUIRED. An array of objects describing the tool calls in the batch. Each object contains:
  * `tool_name`: REQUIRED. A string identifying the invoked tool.
  * `tool_input`: REQUIRED. An object containing the tool's input parameters.
  * `tool_use_id`: REQUIRED. A string identifying the tool invocation.
  * `tool_response`: OPTIONAL. The tool execution result, represented as a string, array, or object depending on the implementation.

15. **Post Tool Use Failure:** Expose the tool name, invocation arguments, and error information for a failed tool invocation.
* `tool_name`: REQUIRED. A string identifying the tool whose invocation failed.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.
* `error`: REQUIRED. A string describing the tool invocation failure.
* `is_interrupt`: OPTIONAL. A boolean indicating whether the failure was caused by user interruption.

16. **Before MCP Execution:** Expose the MCP server name, tool name, and invocation arguments. Allow approving or denying MCP tool execution before invocation.
* `mcp_server_name`: REQUIRED. A string identifying the MCP server in the client's configuration.
* `tool_name`: REQUIRED. A string identifying the MCP tool to be invoked.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.


17. **After MCP Execution:** Expose the MCP server name, tool name, invocation arguments, and execution result. Allow modifying or replacing the result before subsequent processing.
* `mcp_server_name`: REQUIRED. A string identifying the MCP server in the client's configuration.
* `tool_name`: REQUIRED. A string identifying the invoked MCP tool.
* `tool_input`: REQUIRED. An object containing the tool's input parameters.
* `tool_response`: OPTIONAL. The tool execution result, represented as a string, object, or array depending on the tool.

18. **Before Command Execution:** Expose the command and working directory. Allow permitting, denying, or requesting user confirmation before command execution.
* `command`: REQUIRED. A string containing the command to be executed.
* `cwd`: OPTIONAL. A string specifying the current working directory.

19. **After Command Execution:** Expose the executed command and captured output.

* `command`: REQUIRED. A string containing the executed command.
* `output`: OPTIONAL. A string containing the captured command output.

20. **Before File Access:** Expose file access information, including the file path and requested file content. Allow permitting or denying file access before the content is provided to the agent.

* `file_path`: REQUIRED. A string specifying the path of the file being accessed.
* `requested_content`: OPTIONAL. A string or object containing the content of the file requested by the agent.

21. **After File Modification:** Expose file modification information, including the file path and changes made, such as the original and modified text.

* `file_path`: REQUIRED. A string specifying the path of the modified file.
* `edits`: REQUIRED. An array of objects describing the specific changes made, where each object contains:
  * `old_string`: REQUIRED. A string containing the original text segment before modification.
  * `new_string`: REQUIRED. A string containing the replacement text segment after modification.

22. **Agent Completion:** Expose agent completion information, including the completion status, stop reason, and final assistant response. Allow requesting additional processing or continued execution before the agent terminates.

* `stop_reason`: REQUIRED. A string specifying why the agent intends to stop (e.g., "end_turn", "task_completed", "max_tokens", "user_interrupt").
* `last_assistant_message`: OPTIONAL. A string or message object containing the final answer, summary, or response text generated by the agent prior to stopping.
* `stop_hook_active`: OPTIONAL. A boolean indicating whether this stop event was triggered during a re-entrant evaluation loop, used to prevent infinite hook recursion.

23. **Session End:** Expose session termination information, including the session identifier, termination reason, and session duration.
* `session_id`: REQUIRED. A string identifying the closed session.
* `reason`: REQUIRED. A string indicating the precise cause of session termination (e.g., "user_exit", "clear_context", "timeout", "auth_logout", "fatal_error").
* `session_duration`: OPTIONAL. A numeric value representing the total active runtime of the session.
* `total_token_usage`: OPTIONAL. An object capturing cumulative token consumption across the session.

## Hook Failure Handling

Hook failures can affect security enforcement and audit coverage. Two common handling policies are:

* Fail-open: Allow the operation or subsequent processing to continue when a hook fails. This preserves availability but may bypass security checks.

* Fail-closed: Block the operation or subsequent processing when a hook fails. This prevents unchecked operations but may interrupt legitimate workflows.

Based on existing implementation practices listed in Table 1, the choice depends on the hook's purpose and the consequences of failure:

* Security-critical hooks SHOULD fail closed: Any hook involved in input defense, permission validation, pre-execution interception of sensitive operations, or subagent spawning that could lead to privilege escalation MUST block subsequent operations if it fails. Failure to perform required checks may lead to unauthorized operations or sensitive information exposure. Typical events include Before Prompt Submit, Before Model Request, Subagent Start, Before Tool Use, Before MCP Execution, Before Command Execution, Before File Access, Permission Request, etc.

* Observation-only hooks SHOULD generally fail open: These include hooks used only for session lifecycle monitoring, invocation completion logging, context compaction monitoring, command execution result collection. Failure to collect audit information should not normally interrupt legitimate operations. Typical events includes Session Start, After Model Response, Before / After Context Compaction, Subagent Completion, After tool Use, After MCP Execution, After Command Execution, Tool Batch Completion, Post Tool Use Failure, etc.


# Gateway EP
Gateway EPs proxy requests and responses between clients and agents, and between agents and models, tools or other agents. They observe traffic passing through them, apply controls before forwarding, and generate audit records.

##  Critical Events and Corresponding Audit Record Schema

1. **Before Inbound Request Forwarding:** Expose the destination agent and incoming request. Allow permitting or denying forwarding and modifying request content.

* `request_id`: REQUIRED. A string identifying this inbound request to the destination agent.
* `caller_id`: OPTIONAL. A string identifying the authenticated caller of this inbound request.
* `agent`: REQUIRED. A string identifying the destination agent by its configured name or identifier.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this request to the destination agent.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this request before forwarding.
* `reason`: OPTIONAL. A string explaining why this request was blocked or modified.
* `request_body`: OPTIONAL. The incoming request content.

2. **Before Model Request Forwarding:** Expose the requested model and model input. Allow permitting or denying forwarding and modifying request content.

* `request_id`: REQUIRED. A string identifying this model request. Separate model requests within the same agent workflow use different identifiers.
* `caller_id`: OPTIONAL. A string identifying the authenticated caller of this model request.
* `model`: REQUIRED when identifiable from the request or gateway configuration. A string identifying the requested model or configured model alias.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this request to the model service.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this model request before forwarding.
* `reason`: OPTIONAL. A string explaining why this model request was blocked or modified.
* `request_body`: OPTIONAL. The model request content, including messages and supplied parameters.

3. **Before Model Response Forwarding:** Expose the model response. Allow permitting or denying forwarding and modifying the response content.

* `request_id`: REQUIRED. A string matching the identifier recorded for the corresponding model request.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this model response to the caller.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this model response before forwarding.
* `reason`: OPTIONAL. A string explaining why this model response was blocked or modified.
* `response_body`: OPTIONAL. The model response content.
* `error`: OPTIONAL. Error information reported for the model call.
* `usage`: OPTIONAL. An object containing token usage information.
* `duration`: OPTIONAL. A numeric value representing the elapsed time, in milliseconds, from forwarding the corresponding model request to receiving the complete model response.


4. **Before Tool Request Forwarding:** Expose the requested tool and supplied arguments. Allow permitting or denying forwarding and modifying tool arguments.

* `request_id`: REQUIRED. A string identifying this tool request. Separate tool requests within the same agent workflow use different identifiers.
* `caller_id`: OPTIONAL. A string identifying the authenticated caller of this tool request.
* `tool_name`: REQUIRED when identifiable from the request or gateway configuration. A string identifying the requested tool.
* `arguments`: OPTIONAL. An object containing the supplied tool arguments.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this request to the tool service.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this tool request before forwarding.
* `reason`: OPTIONAL. A string explaining why this tool request was blocked or modified.

5. **Before Tool Response Forwarding:** Expose the tool response, including reported errors. Allow permitting or denying forwarding and modifying or replacing supported response content.

* `request_id`: REQUIRED. A string matching the identifier recorded for the corresponding tool request.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this tool response to the caller.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this tool response before forwarding.
* `reason`: OPTIONAL. A string explaining why this tool response was blocked or modified.
* `result`: OPTIONAL. The returned tool result.
* `error`: OPTIONAL. Error information reported for the tool call.
* `duration`: OPTIONAL. A numeric value representing the elapsed time, in milliseconds, from forwarding the corresponding tool request to receiving the complete tool response.


6. **Before Outbound Agent Request Forwarding:** Expose the destination agent and outgoing request. Allow permitting or denying forwarding and modifying or redacting supported request content.

* `request_id`: REQUIRED. A string identifying this request to another agent. Separate requests within the same agent workflow use different identifiers.
* `caller_id`: OPTIONAL. A string identifying the authenticated caller of this outgoing request, rather than an earlier caller merely referenced in the request.
* `agent`: REQUIRED when identifiable from the request or gateway configuration. A string identifying the destination agent by its configured name or identifier.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this request to the destination agent.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this outgoing agent request before forwarding.
* `reason`: OPTIONAL. A string explaining why this outgoing agent request was blocked or modified.
* `request_body`: OPTIONAL. The outgoing agent request content.

7. **Before Outbound Agent Response Forwarding:** Expose the called agent's response. Allow permitting or denying forwarding and modifying or redacting supported response content.

* `request_id`: REQUIRED. A string matching the identifier recorded for the corresponding outbound agent request.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this agent response to the caller.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this agent response before forwarding.
* `reason`: OPTIONAL. A string explaining why this agent response was blocked or modified.
* `response_body`: OPTIONAL. The returned agent response content.
* `error`: OPTIONAL. Error information reported by the destination agent or its service.
* `duration`: OPTIONAL. A numeric value representing the elapsed time, in milliseconds, from when the gateway received the corresponding outbound agent request until it received the complete agent response.

8. **Before Inbound Response Forwarding:** Expose the agent response to the inbound request. Allow permitting or denying forwarding and modifying response content.

* `request_id`: REQUIRED. A string matching the identifier recorded for the corresponding inbound request.
* `decision`: REQUIRED. A string indicating "allow" or "deny" for forwarding this agent response to the caller.
* `modified`: OPTIONAL. A boolean indicating whether the gateway modified this agent response before forwarding.
* `reason`: OPTIONAL. A string explaining why this agent response was blocked or modified.
* `response_body`: OPTIONAL. The agent response content.
* `error`: OPTIONAL. Error information reported by the destination agent or its service.
* `duration`: OPTIONAL. A numeric value representing the elapsed time, in milliseconds, from when the gateway received the corresponding inbound request until it received the complete agent response.


# Host EP
Host EPs use operating-system hooks to observe operations performed by agent-related processes, enforce controls where supported, and generate audit records.

## Summary of Host EP Hooks
~~~
+----------------------+------------------------------------------+
| Name                 | Examples                                 |
+----------------------+------------------------------------------+
| Process Creation     | Linux: sched_process_fork                |
|                      | Windows:                                 |
|                      |   PCREATE_PROCESS_NOTIFY_ROUTINE_EX      |
|                      | macOS: ES_EVENT_TYPE_NOTIFY_FORK         |
+----------------------+------------------------------------------+
| Before Program       | Linux LSM: bprm_check_security           |
| Execution            | Windows:                                 |
|                      |   PCREATE_PROCESS_NOTIFY_ROUTINE_EX      |
|                      |   (CreateInfo != NULL)                   |
|                      | macOS: ES_EVENT_TYPE_AUTH_EXEC           |
+----------------------+------------------------------------------+
| Before File Access   | Linux LSM: file_open                     |
|                      | Linux LSM: file_permission (read)        |
|                      | Windows: PFLT_PRE_OPERATION_CALLBACK     |
|                      |   (IRP_MJ_CREATE)                        |
|                      | Windows: PFLT_PRE_OPERATION_CALLBACK     |
|                      |   (IRP_MJ_READ)                          |
|                      | macOS: ES_EVENT_TYPE_AUTH_OPEN           |
+----------------------+------------------------------------------+
| Before File          | Linux LSM: file_permission (write)       |
| Modification         | Linux LSM: inode_unlink                  |
|                      | Linux LSM: inode_rename                  |
|                      | Windows: PFLT_PRE_OPERATION_CALLBACK     |
|                      |   (IRP_MJ_WRITE)                         |
|                      | Windows: PFLT_PRE_OPERATION_CALLBACK     |
|                      |   (IRP_MJ_SET_INFORMATION)               |
|                      | macOS: ES_EVENT_TYPE_AUTH_UNLINK         |
|                      | macOS: ES_EVENT_TYPE_AUTH_RENAME         |
+----------------------+------------------------------------------+
| After File Operation | Windows: PFLT_POST_OPERATION_CALLBACK    |
|                      |   (IRP_MJ_CREATE)                        |
|                      | Windows: PFLT_POST_OPERATION_CALLBACK    |
|                      |   (IRP_MJ_READ)                          |
|                      | Windows: PFLT_POST_OPERATION_CALLBACK    |
|                      |   (IRP_MJ_WRITE)                         |
|                      | Windows: PFLT_POST_OPERATION_CALLBACK    |
|                      |   (IRP_MJ_SET_INFORMATION)               |
|                      | macOS: ES_EVENT_TYPE_NOTIFY_UNLINK       |
|                      | macOS: ES_EVENT_TYPE_NOTIFY_RENAME       |
+----------------------+------------------------------------------+
| Before Network       | Linux LSM: socket_connect                |
| Operation            | Linux LSM: socket_bind                   |
|                      | Linux LSM: socket_listen                 |
|                      | Windows WFP: classifyFn                  |
|                      |   FWPM_LAYER_ALE_AUTH_CONNECT_V4/V6      |
|                      | Windows WFP: classifyFn                  |
|                      |   FWPM_LAYER_ALE_RESOURCE_ASSIGNMENT_V4  |
|                      |   /V6                                    |
|                      | Windows WFP: classifyFn                  |
|                      |   FWPM_LAYER_ALE_AUTH_LISTEN_V4/V6       |
+----------------------+------------------------------------------+
| Before Access to     | Linux LSM: ptrace_access_check           |
| Another Process      | Windows: POB_PRE_OPERATION_CALLBACK      |
|                      |   (process/thread handles)               |
|                      | macOS: ES_EVENT_TYPE_AUTH_GET_TASK       |
+----------------------+------------------------------------------+
| Before Memory        | Linux LSM: file_mprotect                 |
| Protection Change    | macOS: ES_EVENT_TYPE_AUTH_MPROTECT       |
+----------------------+------------------------------------------+
| Process Exit         | Linux: sched_process_exit                |
|                      | Windows:                                 |
|                      |   PCREATE_PROCESS_NOTIFY_ROUTINE_EX      |
|                      |   (CreateInfo == NULL)                   |
|                      | macOS: ES_EVENT_TYPE_NOTIFY_EXIT         |
+----------------------+------------------------------------------+
~~~
*Table 2.  HOST EP Hooks and Examples*

## Critical Event and Corresponding Audit Record Schema

The following claims apply to all events:
* `process_id`: REQUIRED. An integer identifying the current process.

In addition to the common claim above, each event defines its own audit record claims to capture event-specific information.

1. **Process Creation:** Expose the identifiers of the newly created process and its parent. Allow denying process creation.


* `parent_process_id`: REQUIRED. An integer identifying its parent process.

2. **Before Program Execution:** Expose the process identifier and the program to be executed. Allow permitting or denying program execution.

* `executable_path`: OPTIONAL. A string specifying the executable path.

3. **Before File Access:** Expose the requested file operation and the target file. Allow permitting or denying the operation.


* `operation`: REQUIRED. A string with the value "open" or "read".
* `file_path`: OPTIONAL. A string specifying the target file path.

4. **Before File Modification:** Expose the requested file modification, such as writing, deletion, or renaming, and the target file path. Allow permitting or denying the operation.


* `operation`: REQUIRED. A string identifying the requested operation, such as "write", "unlink", or "rename".
* `file_path`: OPTIONAL. A string specifying the target file path, or the source file path for a rename operation.
* `destination_path`: OPTIONAL. A string specifying the destination path for a rename operation.

5. **Before Access to Another Process:** Expose the identity of the process being accessed. Allow permitting or denying access.

* `target_process_id`: REQUIRED. An integer identifying the target process, or the process owning the target thread.

6. **Process Exit:** Expose the identifier of the exiting process. No additional event-specific audit record claims are defined.







# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
