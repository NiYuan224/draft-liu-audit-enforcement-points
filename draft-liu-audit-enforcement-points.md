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
 RFC3198:

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

  AgentgatewayLogs:
    title: "Agentgateway Access Logs"
    target: https://agentgateway.dev/docs/standalone/latest/documentation/observability/access-logs/view/

  KongAILogs:
    title: "Kong AI Gateway Audit Log Reference"
    target: https://developer.konghq.com/ai-gateway/ai-audit-log-reference/

  EnvoyAccessLogs:
    title: "Envoy Gateway Proxy Access Logs"
    target: https://gateway.envoyproxy.io/latest/tasks/observability/proxy-accesslog/

  LiteLLMLogs:
    title: "LiteLLM Logging"
    target: https://docs.litellm.ai/docs/proxy/logging

  DefenderEDR:
    title: "Overview of Endpoint Detection and Response"
    target: https://learn.microsoft.com/en-us/defender-endpoint/overview-endpoint-detection-response

  AUDIT-CHARTER:
    title: "Agent Use of Delegation and Interaction Traceability (AUDIT) Working Group Charter"
    target: "https://datatracker.ietf.org/doc/bofreq-mirjak-audit/"

  LinuxLSM:
    title: "Linux Security Modules: General Security Hooks for Linux"
    target: "https://docs.kernel.org/security/lsm.html"

  WindowsKernelHooks:
    title: "Windows Kernel Callbacks and Filtering Mechanisms"
    target: "https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/"

  AppleEndpointSecurity:
    title: "Endpoint Security Event Types"
    target: "https://developer.apple.com/documentation/endpointsecurity/es_event_type_t"




--- abstract

Agent auditing relies on distributed enforcement points (EPs) to observe and intervene in agent activities. This document analyzes Agent, Gateway, and Host EPs by detailing their coverage and audit data. This provides a basis for understanding the capabilities of different EPs and evaluating overall audit evidence coverage.

--- middle

# Introduction {#intro}


Agents increasingly act on behalf of users across multiple services and administrative domains. Auditing their activities requires runtime information, primarily provided by enforcement points (EPs) that observe and, where supported, intervene in agent activities. These EPs are distributed across the interaction chain, each covering a different set of activities and providing corresponding audit records.

To support runtime monitoring and auditing, industry solutions record events at the agent, gateway, and host layers: agent hooks expose agent lifecycle events and tool calls {{ClaudeCodeHooks}}{{CursorHooks}}{{HooksSecurity}}; gateways inspect requests and responses and record access decisions and content filtering results{{LiteLLMLogs}}; Endpoint Detection and Response (EDR) systems monitor process execution, file access, and network connections{{DefenderEDR}}.

This document considers three categories of EPs: Agent EPs, Gateway EPs, and Host EPs. For each category, it describes the audit information EPs can provide, providing a basis for identifying the records across participating domains and understanding the coverage of the available evidence.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

The definitions in this document are based on the Policy Enforcement Point (PEP) definition in {{RFC3198}} and the audit record concepts described in the proposed AUDIT Working Group Charter {{AUDIT-CHARTER}}.


Enforcement Point (EP): A logical entity or network element that observes agent activities and generates corresponding audit records, and may enforce security controls where supported.

Audit Record: A record of the observable behavior of agents, including user intent, agent or tool interactions, invoked or declined actions, exchanged data, and resulting outcomes across trust boundaries.

Hook: A mechanism exposed by software or an operating system at a defined execution point to enable observation and, where supported, intervention.

# Agent Workflows and Enforcement Points

Figure 1 illustrates a typical agent workflow and the locations of the three EP categories. The interactions shown are illustrative and may occur in different sequences and multiple times depending on the agent workflow.

~~~
                           +-----------+
                           |           |
+------------------+ (1)   |           |        +------------------+
|                  +-------+           +-------->       LLM        |
|                  <-------+           +--------+                  |
|                  | (2)   |           |        +------------------+
|                  | (3)   |           |        +------------------+
|   Client Agent   +-------+   Gateway +------->|      Tools       |
|    (Agent EP)    <-------+     EP    +--------+                  |
|                  | (4)   |           |        +------------------+
|                  | (5)   |           |        +------------------+
|                  +-------+           +-------->   Server Agent   |
|                  |<------+           +--------+    (Agent EP)    |
+------------------+ (6)   |           |        +------------------+
|   Host(Host EP)  |       |           |        |  Host (Host EP)  |
+------------------+       |           |        +------------------+
                           |           |
                           |           |
                           +-----------+
~~~

*Figure 1. Agent Workflow and EPs*

1. The Agent provides context to the LLM.
2. The LLM returns output to the Agent to support the selection of subsequent actions.
3. The Agent invokes one or more tools.
4. The tools return results to the Agent.
5. The Agent delegates tasks to or exchanges requests with other agents.
6. Other Agents return responses or task execution results to the Agent.

EPs provide visibility into and control over the above agent activities. They may be located within the agent, at communication gateways, or on the host where the agent runs. Accordingly, EPs are classified into three categories: Agent EPs, Gateway EPs, and Host EPs.

* Agent EPs: EPs integrated into the agent, using hooks to observe and control agent activities, such as user interactions, model requests, tool calls, and host operations.

* Gateway EPs: EPs deployed in the communication paths between agents and models, tools, and other agents. They inspect and control requests and responses passing through the gateway.

* Host EPs: EPs running on the hosts or endpoints where agents execute. They monitor and control agent-related processes, command execution, file access, and other system operations.

Each category provides audit records within its observation and intervention scope. Their coverage may overlap, but no single EP necessarily observes the entire interaction chain.


# Agent EP

Agent EPs use agent-provided hooks to observe critical agent events, enforce controls where supported, and generate audit records.

## Summary of Agent EP Hooks

The following table lists commonly adopted agent hooks and representative implementations.

The hooks cover critical events, including session lifecycle management, prompt handling, model requests and responses, subagent management, tool invocation and results, command execution, and file operations.
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
| After Tool Use           | Claude Code: `PostToolUse`             |
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
~~~
*Table 1. Agent EP Hooks and Examples*

## Critical Events and Corresponding Audit Record Schema

The events and audit record claims in this section are derived from commonly adopted agent hooks documented by Claude Code {{ClaudeCodeHooks}}, Cursor {{CursorHooks}}, GitHub Copilot {{CopilotHooks}}, Qwen Code {{QwenCodeHooks}}, Gemini CLI {{GeminiCLIHooks}}, Hermes Agent {{HermesHooks}}, and OpenClaw {{OpenClawHooks}}. For each event, this section describes the observable information, supported interventions, and corresponding REQUIRED or OPTIONAL audit record claims.

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
* `trigger`: REQUIRED. A string indicating whether compaction was triggered automatically ("auto") or manually ("manual").

7. **After Context Compaction:** Expose session information, the compaction trigger, and the generated summary.

* `trigger`: REQUIRED. A string indicating whether compaction was triggered automatically ("auto") or manually ("manual").
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
* `edits`: OPTIONAL. An array of objects describing the specific changes made, when available. Each object contains:
  * `old_string`: OPTIONAL. A string containing the original text segment before modification.
  * `new_string`: OPTIONAL. A string containing the replacement text segment after modification.

22. **Agent Completion:** Expose agent completion information, including the completion status, stop reason, and final assistant response. Allow requesting additional processing or continued execution before the agent terminates.

* `stop_reason`: OPTIONAL. A string specifying why the agent intends to stop (e.g., "end_turn", "task_completed", "max_tokens", "user_interrupt").
* `last_assistant_message`: OPTIONAL. A string or message object containing the final answer, summary, or response text generated by the agent prior to stopping.
* `stop_hook_active`: OPTIONAL. A boolean indicating whether this stop event was triggered during a re-entrant evaluation loop, used to prevent infinite hook recursion.

23. **Session End:** Expose session termination information, including the session identifier, termination reason, and session duration.
* `session_id`: REQUIRED. A string identifying the closed session.
* `reason`: OPTIONAL. A string indicating the precise cause of session termination (e.g., "user_exit", "clear_context", "timeout", "auth_logout", "fatal_error").
* `session_duration`: OPTIONAL. A numeric value representing the total active runtime of the session.
* `total_token_usage`: OPTIONAL. An object capturing cumulative token consumption across the session.

## Hook Failure Handling

Hook failures can affect security enforcement and audit coverage. Two common handling policies are:

* Fail-open: Allow the operation or subsequent processing to continue when a hook fails. This preserves availability but may bypass security checks.

* Fail-closed: Block the operation or subsequent processing when a hook fails. This prevents unchecked operations but may interrupt legitimate workflows.

Based on existing implementation practices listed in Table 1, the choice depends on the hook's purpose and the consequences of failure:

* Security-critical hooks SHOULD fail closed: Any hook involved in input defense, permission validation, pre-execution interception of sensitive operations, or subagent spawning that could lead to privilege escalation SHOULD block subsequent operations if it fails. Failure to perform required checks may lead to unauthorized operations or sensitive information exposure.
  * Typical events include Before Prompt Submit, Before Model Request, Subagent Start, Before Tool Use, Before MCP Execution, Before Command Execution, Before File Access, Permission Request, etc.

* Observation-only hooks SHOULD generally fail open when their failure does not affect required security checks. These include hooks used only for session lifecycle monitoring, invocation completion logging, context compaction monitoring, and command execution result collection. Failure to collect audit information should not normally interrupt legitimate operations.
  * Typical events include Session Start, After Model Response, Before and After Context Compaction, Subagent Completion, After Tool Use, After MCP Execution, After Command Execution, Tool Batch Completion, Post Tool Use Failure, etc.


# Gateway EP
Gateway EPs proxy requests and responses between agents and external models, tools, or other agents. They inspect traffic, apply access controls and content policies, and generate corresponding audit records.

## Critical Events and Corresponding Audit Record Schema

The following events and audit record claims are derived from access logging and audit practices in Agentgateway {{AgentgatewayLogs}}, Kong AI Gateway {{KongAILogs}}, Envoy Gateway {{EnvoyAccessLogs}}, and LiteLLM {{LiteLLMLogs}}.

1. **Before Request Forwarding:** Expose request information, including the caller, destination, protocol, and operation. Allow permitting or denying request forwarding and modifying request content.

* `request_id`: REQUIRED. A string identifying the request.
* `caller`: OPTIONAL. A string identifying the entity initiating the request, such as an authenticated user or agent.
* `destination`: REQUIRED. A string identifying the destination of the request, such as a model, tool, agent, or service endpoint.
* `protocol`: OPTIONAL. A string identifying the protocol, such as "HTTP", "MCP", or "A2A".
* `operation`: OPTIONAL. A string identifying the requested operation, such as a model operation, MCP method, or A2A method.
* `action`: OPTIONAL. A string indicating the policy action, such as "allow", "deny", "mask", or "reject".
* `action_reason`: OPTIONAL. A string explaining the policy action.
* `request_body`: OPTIONAL. The request content.


2. **Before Response Forwarding:** Expose response information and execution outcomes. Allow permitting or denying response forwarding and modifying response content where supported by the gateway.

* `request_id`: REQUIRED. A string matching the identifier recorded for the corresponding request.
* `status`: OPTIONAL. A string or integer indicating the outcome of the request, such as an HTTP status code, an RPC error code, or an A2A task state.
* `action`: OPTIONAL. A string indicating the action taken by the gateway, such as "allow", "deny", "mask", or "reject".
* `action_reason`: OPTIONAL. A string explaining the policy action.
* `response_body`: OPTIONAL. The response content.
* `error`: OPTIONAL. A string describing an error encountered during request forwarding or response forwarding.
* `duration`: REQUIRED. A numeric value representing the elapsed time from receiving the corresponding request to receiving the complete response.
* `usage`: OPTIONAL. An object containing resource usage information, such as input and output token counts.


# Host EP
Host EPs use operating-system hooks to observe operations performed by agent-related processes, enforce controls where supported, and generate audit records.

## Summary of Host EP Hooks

The hooks in Table 2 cover major host-level activities, including process lifecycle management, file operations, network operations, access to other processes, and memory protection changes.
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
| Before Process Access| Linux LSM: ptrace_access_check           |
|                      | Windows: POB_PRE_OPERATION_CALLBACK      |
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

## Critical Events and Corresponding Audit Record Schema

The events and audit record claims in this section are derived from operating system hook interfaces and event data structures provided by Linux Security Modules (LSM) {{LinuxLSM}}, Windows kernel callbacks and filtering mechanisms {{WindowsKernelHooks}}, and Apple Endpoint Security {{AppleEndpointSecurity}}.

The following claims apply to all events:
* `process_id`: REQUIRED. An integer identifying the current process.

In addition to the common claim above, each event defines its own audit record claims to capture event-specific information.

1. **Process Creation:** Expose the identifiers of the newly created process and its parent.


* `parent_process_id`: REQUIRED. An integer identifying its parent process.

2. **Before Program Execution:** Expose the process identifier and the program to be executed. Allow permitting or denying program execution.

* `executable_path`: OPTIONAL. A string specifying the executable path.

3. **Before File Access:** Expose the requested file operation and the target file. Allow permitting or denying the operation.


* `operation`: REQUIRED. A string identifying the requested file access operation, such as "open" or "read".
* `file_path`: OPTIONAL. A string specifying the target file path.

4. **Before File Modification:** Expose the requested file modification, such as writing, deletion, or renaming, and the target file path. Allow permitting or denying the operation.


* `operation`: REQUIRED. A string identifying the requested operation, such as "write", "unlink", or "rename".
* `file_path`: OPTIONAL. A string specifying the target file path, or the source file path for a rename operation.
* `destination_path`: OPTIONAL. A string specifying the destination path for a rename operation.

* `operation`: REQUIRED. A string identifying the requested file access operation, such as "open" or "read".

* `operation`: OPTIONAL. A string identifying the completed file operation.
* `file_path`: OPTIONAL. A string specifying the path of the affected file, or the source path for a rename operation.
* `destination_path`: OPTIONAL. A string specifying the destination path for a rename operation.

6. **Before Network Operation:** Expose the local and remote network endpoints involved in the requested network operation. Allow permitting or denying the operation where supported by the operating system.

* `local_endpoint`: OPTIONAL. A string identifying the local network endpoint.
* `remote_endpoint`: OPTIONAL. A string identifying the remote network endpoint.

7. **Before Process Access:** Expose requests to access another process, including the identity of the target process. Allow permitting or denying access where supported by the operating system.

* `target_process_id`: REQUIRED. An integer identifying the target process, or the process owning the target thread.

8. **Before Memory Protection Change:** Expose the requested memory protection change, including the affected address range and protection attributes. Allow permitting or denying the change where supported by the operating system.

* `address`: OPTIONAL. An integer identifying the starting virtual memory address of the affected range.
* `size`: OPTIONAL. An integer specifying the size of the affected memory range.
* `protection`: OPTIONAL. An integer identifying the memory protection attributes to be applied to the affected range.


9. **Process Exit:** Expose the identifier of the exiting process. No additional event-specific audit record claims are defined.




# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

