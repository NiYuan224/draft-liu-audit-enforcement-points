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
 - next generation
 - unicorn
 - AI-native
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

Agents increasingly act on behalf of users across multiple services and administrative domains. Their activities span users, other agents, models, tools, and hosts, but the audit information is distributed across participants, limiting visibility into the entire interaction chain.

Security products and components deployed along the interaction chain can observe agent activities, enforce security controls, and provide audit information. For example, agent gateways can inspect requests and responses and enforce access policies; model gateways can inspect model requests and responses; MCP proxies can inspect tool calls and results; and EDR systems and sandbox monitoring components can monitor host operations and restrict unauthorized operations. In this document, these products and components serve as enforcement points (EPs), each covering a different set of agent activities and providing corresponding audit records. Agent implementations may expose hooks at different stages to enable these capabilities.

This document classifies EPs into three categories based on the activities they cover: agent interaction, model and tool invocation, and host-level operation. For each category, it describes how hooks enable EPs to observe and, where supported, intervene in agent activities, as well as the corresponding audit records EP provide. This analysis is a prerequisite for identifying the audit records needed across participating domains and understanding the coverage and limitations of the available evidence.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Enforcement Point (EP): A logical entity that observes a defined set of agent activities and may intervene in these activities or their subsequent processing, including by blocking or modifying them, and generates corresponding audit records.

Hook: A mechanism exposed by an agent implementation at a defined stage of an agent activity, enabling an EP to access activity information and, where supported, intervene.

# Agent Workflows and Enforcement Points

## Agent Workflow

Figure 1 illustrates a typical agent workflow.

~~~
+----------------+       +----------------+ (2)   +----------------+
|                |       |                |------>| Large Language |
|                |       |                |<------| Model (LLM)    |
|                | (1)   |                | (3)   +----------------+
|                |------>|                |                         
|                |       |                | (4)   +----------------+
|  User/System   |       |      Agent     |------>|     Tools      |
|                |       |                |<------|                |
|                |<------|                | (5)   +----------------+
|                |  (10) |                |                         
|                |       |                | (6)   +----------------+
|                |       |                |------>|  Other Agents  |
|                |       |                |<------|                |
+----------------+       +---|--------^---+ (7)   +----------------+
                             |        |                             
                          (8)|        |(9)                          
+----------------------------v--------+----------------------------+
|                               Host                               |
+------------------------------------------------------------------+
~~~
*Figure 1. Agent Workflow*

1. (Optional) The User or System provides a request or instruction to the AI Agent.
2. The AI Agent provides context to the LLM.
3. The LLM returns output to the AI Agent to support the selection of subsequent actions.
4. The AI Agent invokes one or more Tools.
5. The invoked endpoints return results to the AI Agent. The Agent may provide these results as additional context to the LLM and repeat steps 2-5 until the task is completed.
6. (Optional) The AI Agent delegates tasks to or exchanges requests with other agents.
7. (Optional) Other agents return responses or task execution results to the AI Agent.
8. (Optional) The AI Agent perform operations on the Host.
8. (Optional) The Host provides corresponding results, status, or errors.
9. (Optional) The AI Agent returns a response to the User or System, or requests additional input.


## Enforcement Point Categories

Enforcement Points (EPs) provide visibility into and control over agent activities. They may be located within the agent, at communication gateways, or on the host where the agent runs. Accordingly, EPs are classified into three categories: Agent Client EPs, Gateway EPs, and EDR EPs.

* Agent Client EPs：EPs integrated into the agent client, using hooks to observe and control agent activities, such as user interactions, model requests, tool calls, and host operations.

* Gateway EPs: EPs deployed in the inbound communication paths between users/systems and agents, and the outbound communication paths between agents and models, tools, and other agents. They inspect and control requests and responses passing through the gateway.

* EDR EPs: EPs running on the hosts or endpoints where agents execute. They monitor and control agent-related processes, command execution, file access, and other system operations.

~~~
+----------------------------------------------------------------------+
|                               Agent                                  |
+------^--------------^-------------^-------------^-------------^------+
       |              |             |             |             |       
       |        +-----+-------------+-------------+-------------+------+
       |        |                    Agent-native EP                   |
       |        +-----+-------------+-------------+-------------+------+
       |              |             |             |             |       
+------+--------------+-------------+-------------+-----+ +-----+------+
|                        Gateway EP                     | |  EDR EP    |
+------+--------------+-------------+-------------+-----+ +-----+------+
       |              |             |             |             |       
+------v-----+  +-----v-----+ +-----v-----+ +-----v-----+ +-----v------+
|Other Agents|  |User/System| |   LLM     | |   Tools   | |    Host    |
+------------+  +-----------+ +-----------+ +-----------+ +------------+
~~~
*Figure 2. Enforcement Points for Agents*



1. (Optional) The User or System provides a request or instruction to the AI Agent.
2. The AI Agent provides context to the LLM.
3. The LLM returns output to the AI Agent to support the selection of subsequent actions.
4. The AI Agent invokes one or more Tools.
5. The invoked endpoints return results to the AI Agent. The Agent may provide these results as additional context to the LLM and repeat steps 2-5 until the task is completed.
6. (Optional) The AI Agent or its Tools perform operations on the Host.
7. (Optional) The Host provides corresponding results, status, or errors.
8. (Optional) The AI Agent returns a response to the User or System, or requests additional input.

The three EP categories cover different parts of this interaction chain. The Agent Interaction EP covers the user/system-agent interactions in steps 1 and 8. The Model and Tool Invocation EP covers the model and tool interactions in steps 2-5. The Host-level Operation EP covers the host-level operations and their corresponding results in steps 6 and 7.

## Hook Capabilities

Hooks support two capabilities:

* Observability: A hook provides information about a specific stage of an agent activity for audit recording, such as tool invocation arguments, execution results, error information, or execution status.

* Interventability: A hook allows EP to affect an agent activity or its subsequent processing, including by blocking or cancelling the activity, modifying its parameters or content, or changing information passed to subsequent processing.

Some hooks provide observation only, while others support both observation and intervention.


# Agent Client

## Summary of Agent Client Hooks

This section summarizes Agent Client hooks along the agent execution workflow.

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
~~~

## Hook Capabilities
Hooks support two capabilities:

* Observability: A hook provides information about a specific stage of an agent activity for audit recording, such as tool invocation arguments, execution results, error information, or execution status.

* Interventability: A hook allows EP to affect an agent activity or its subsequent processing, including by blocking or cancelling the activity, modifying its parameters or content, or changing information passed to subsequent processing.

Some hooks provide observation only, while others support both observation and intervention.

## Observability and Interventability

The following sections summarize the observable information and intervention capabilities provided by representative Agent Client hooks. The specific observable information and supported interventions vary across implementations.

* **Session Start:** Expose session information, including the session identifier and source. Allow EPs to set session-specific environment variables or inject initial security context.

* **Prompt Submission:** Expose the submitted prompt and associated session information. Allow EPs to inspect, modify, or block the prompt before agent processing.

* **Prompt Expansion:** Expose the expanded prompt and associated command information, including the command name and arguments. Allow EPs to inject additional context or block the expanded prompt before model invocation.

* **Before Model Request:** Expose model request information, including the selected model, provider, and request messages. Allow EPs to modify or block model requests before invocation.

* **After Model Response:** Expose generated content, response metadata, and token usage. Allow EPs to modify, replace, or block the response before subsequent processing.

* **Before Context Compaction:** Expose compaction-related information, including the compaction trigger, context token count, and messages subject to compaction. Allow EPs to provide additional context or instructions for compaction, or block compaction.

* **After Context Compaction:** Expose compaction results, including the generated summary and available message or token counts.

* **Subagent Start:** Expose subagent information, including the agent identifier, agent type, and delegated task. Allow EPs to inject additional context or instructions into the subagent, or control its initialization.

* **Subagent Completion:** Expose subagent completion information, including its identifier, execution status, and returned summary. Allow EPs to modify the subagent response or request continued execution.

* **Before Tool Use:** Expose tool invocation information, including the tool name, invocation arguments, and tool call identifier. Allow EPs to approve, deny, or modify tool invocations before execution.

* **Permission Request:** Expose permission request information, including the requested tool, invocation arguments, and permission context. Allow EPs to approve or deny the requested operation.

* **Permission Denied:** Expose information about denied operations, including the tool name, invocation arguments, and denial reason.

* **After Tool Use:** Expose tool execution information, including the tool name, invocation arguments, execution results, and execution status. Allow EPs to modify or withhold results before subsequent model processing.

* **Tool Batch Completion:** Expose batch execution information, including the individual tool calls, their arguments, and execution results.

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
