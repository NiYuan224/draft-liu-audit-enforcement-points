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

# Enforcement Point Model

This section classifies audit enforcement points according to the agent activities they cover and describes the observation and intervention capabilities of hooks.

## Enforcement Point Categories

EPs are classified into three categories according to the activities they cover:

* Agent Interaction EP: Covers interactions between users or upstream systems and agents, including incoming requests and outgoing responses.

* Model and Tool Invocation EP: Covers interactions between agents and models or tools, including invocation requests and returned results.

* Host-level Operation EP: Covers host operations performed by agents or their tools, including process execution, file access, etc.

Figure 1 illustrates the three categories and their positions in the agent interaction chain.

~~~

      +----------------+       +----------------+       +----------------+
      |                |       |                |       |                |
      |                |       |                | (2)   | Large Language |
      |                | (1)   |                |------>| Model (LLM)    |
      |                |------>|                |<------|                |
      │   User/System  │       |                | (3)   +----------------+
      │                │       |    AI Agent    |
      │                │       |                | (4)   +----------------+
      │                │<------|                |------>|                |
      │                │  (8)  |                |<------|     Tools      |
      |                |       |                | (5)   |                |
      +----------------+       +----------------+       +----------------+
                                       │                       |
                                       │                       |
                                       +-------+--------^------+
                                            (6)│        │(7)
      +----------------------------------------v--------+----------------+
      |                               Host                               |
      +------------------------------------------------------------------+
~~~


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

* Observation: A hook provides information about a specific stage of an agent activity for audit recording, such as tool invocation arguments, execution results, error information, or execution status.

* Intervention: A hook allows EP to affect an agent activity or its subsequent processing, including by blocking or cancelling the activity, modifying its parameters or content, or changing information passed to subsequent processing.

Some hooks provide observation only, while others support both observation and intervention.

# User Interaction Enforcement Point

User Interaction Enforcement Points cover activities at the interaction boundary between users or upstream systems and agents, including session lifecycle management, input reception and processing, and response handling. This section describes the trigger stages of relevant hooks, the information they expose, and the intervention capabilities they provide to User Interaction EPs.

## Session Lifecycle Management

Session lifecycle management activities include the following stages:

* Session start: Hooks at this stage can expose session-related information, such as the session source. Some hooks also allow EPs to set session-specific environment variables or add initial context, such as security constraints applicable to the session.

* Session termination: Hooks at this stage provide information about the termination reason, session status, and elapsed time. EPs can use this information to audit session termination and investigate abnormal session behavior.

## Input Reception and Processing

Input reception and processing activities include the following stages:

* Input reception and submission: Hooks at message reception or before agent processing can expose the message source and content. They may allow EPs to inspect, modify, or discard the message, or take over input processing through a plugin.

* Command expansion: Hooks triggered during command expansion expose information about the expanded prompt. They may allow an EP to block the expansion or add context before the resulting input is processed by the agent.

## Response Handling

Response handling activities include the following stages:

* After Response generation: Hooks at this stage expose the generated response and may allow an EP to request further processing, including response regeneration.

* Before response sending: Hooks before a response is sent to a user or upstream system expose the outgoing message. They may allow an EP to modify the message or cancel the send operation.

* After response sending: Hooks after a send attempt can expose whether sending succeeded or failed, allowing an EP to record the send outcome.

# Model and Tool Invocation Enforcement Point

Model and Tool Invocation Enforcement Points cover interactions between agents and models or tools, including model invocation, tool invocation, and context management. 

## Model Invocation

Model invocation activities include the following stages:

* Model selection and prompt construction: Hooks before model invocation can provide information about the selected model, model provider, prompt, and associated context. They may allow an EP to select or restrict the model, modify the prompt or context, or restrict the tools available to the model for the current turn.

* Model request: Hooks at this stage can expose the input submitted to the model, including the prompt and associated context. Some hooks also allow the request to be modified or blocked, or a predefined response to be supplied instead of invoking the model.

* Model response: Hooks at this stage can expose the generated content and available token usage information. Some hooks allow an EP to modify or block the response before it is passed to subsequent processing.

* Invocation completion and failure: Hooks at this stage can expose invocation status, duration, usage information, and error details, providing observational information for investigating failed requests and abnormal resource consumption.

## Tool Invocation

Tool invocation activities include the following stages:

* Before tool invocation: Hooks at this stage can expose the information of selected tool and invocation arguments. They may allow an EP to restrict the available tools, modify invocation arguments, or block the invocation.

* Permission request: Hooks at this stage can expose the requested operation and associated permission information. Some hooks allow an EP to approve or deny the request.

* After tool invocation: Hooks at this stage can expose the execution result, status, errors, and available timing information. These hooks are primarily observational.

* Tool result processing: Hooks at this stage expose tool results before they are passed to the model or persisted in the conversation. Some hooks also allow an EP to modify, replace, or withhold the results.

## Context Management

Context management activities include the following stages:

* Before context compaction: Hooks at this stage can expose the context state before compaction. They provide information for tracing context changes.

* After context compaction: Hooks at this stage can expose the resulting context. When both pre-compaction and post-compaction information are available, an EP can compare them to investigate changes in the retained context.

# Host-level Enforcement Point

Host-level Enforcement Points cover operations performed by agents or their tools on the host, including command execution, file operations, and extension installation.

## Extension Installation

Extension installation activities include the following stages:

* Before extension installation: Hooks at this stage expose installation requests for tools, skills and plugins, allowing an EP to perform security checks and block installation based on the results.

## Command Execution

Command execution activities include the following stages:

* Before command execution: Hooks at this stage expose the command and execution directory. They allow an EP to allow or block execution, or request user confirmation.

* After command execution: Hooks at this stage expose the executed command, captured output, and execution duration. These records support auditing of command execution and analysis of command output.

## File Operations

File operation activities include the following stages:

* Before file content is provided to the agent: Hooks at this stage can expose the file path and content and may allow an EP to block the content from being provided to the agent.

* After file modification: Hooks at this stage can expose the modified file and available information about changes, such as the text before and after modification, supporting auditing of file modifications.



# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
