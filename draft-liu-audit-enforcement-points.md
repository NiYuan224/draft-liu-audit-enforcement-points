---
title: "Audit Enforcement Points for Agents"
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
    fullname: "NiYuan224"
    organization: Your Organization Here
    email: "niyuan1@huawei.com"

normative:
RFC2119:
RFC8174:

informative:



--- abstract

This document identifies key audit enforcement points for agents, specifying the activities they cover and the audit records they provide. It also describes the hooks used to capture these records and how hook failures are handled.


--- middle

# Introduction {#intro}

Agents are increasingly acting on behalf of users across multiple services and administrative domains. Their activities may involve the user, other agents, models, tools, and hosts. Information about these activities is distributed across participants, with no single party having complete visibility into the entire interaction chain. AUDIT therefore aims to enable the collection and correlation of audit information across participants to provide visibility across the chain.

Security products and components deployed along the interaction chain can provide audit information by inspecting requests, enforcing access decisions, and monitoring execution. For example, agent gateways can observe requests from users, applications, or other agents and the corresponding responses; model gateways can inspect model requests, context, and responses; MCP proxies can inspect tool calls and results; and EDR systems and sandbox monitoring components can capture host operations such as process creation, file access, and network connections. These products and components serve as audit enforcement points, each covering a different set of agent activities.

To use this information for auditing, it is necessary to understand which activities each enforcement point covers and what audit records it can provide. This document classifies audit enforcement points into three categories based on the activities they cover: agent interaction, model and tool invocation, and host-level operation. For each category, it describes the audit records available through hooks, the intervention capabilities of the corresponding hooks, and how hook failures are handled. This analysis is a prerequisite for identifying the audit records needed across participating domains and understanding the coverage and limitations of the available evidence.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Audit Enforcement Point: A logical entity that observes a defined set of agent activities and provides audit records for them. It may also intervene in these activities or their subsequent processing, including by blocking or modifying them. For example, an audit enforcement point at the interaction boundary between agents and tools may observe the tool invocation activity and provide corresponding audit records.

Hook: A mechanism associated with a specific stage of an agent activity that enables observation of information at that stage and may enable intervention in the activity or its subsequent processing. For example, for a tool invocation activity, a hook may be triggered before the invocation to record its arguments or block it, or after the invocation to record its result.

# Audit Enforcement Point Model

This section classifies audit enforcement points according to the agent activities they cover and categorizes hooks by trigger stage and capability.

## Audit Enforcement Point Categories

Audit enforcement points are classified into three categories according to the activities they cover:

* Agent Interaction Enforcement Point: Cover interactions between users or upstream systems and agents, including incoming requests and outgoing responses.

* Model and Tool Invocation Enforcement Point: Cover interactions between agents and models or tools, including invocation requests and returned results.

* Host-level Operation Enforcement Point: Cover activities performed by agents or their tools on the host, including process creation, file access, and network connections, etc., and responses.

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

Figure 1 illustrates the interactions among the User or System, AI Agent, LLM, Tools, and Host.

1. (Optional) The User or System provides an request or instruction to the AI Agent.
2. The AI Agent provides context to the LLM.
3. The LLM returns output to the AI Agent to support the selection of subsequent actions.
4. The AI Agent invokes one or more Tools.
5. The invoked endpoints return results to the AI Agent. The Agent may provide these results as additional context to the LLM and repeat steps 2–5 until the task is completed.
6. (Optional) The AI Agent or its Tools perform operations on the Host.
7. (Optional) The Host provides corresponding results, status, or errors where available.
8. (Optional) The AI Agent returns a response to the User or System, or requests additional input.

The three categories of audit enforcement points cover different parts of this interaction chain. The Agent Interaction Enforcement Point covers the user-agent interactions in steps 1 and 8. The Model and Tool Invocation Enforcement Point covers the model and tool interactions in steps 2–5. The Host-level Operation Enforcement Point covers the host-level operations and their corresponding results in steps 6 and 7.

## Hook Classification

Hooks are classified along two independent dimensions: trigger stage and capabilities. Trigger stage describes when a hook is triggered during an agent activity, while capabilities describe whether a hook provides information only or can also intervene in the activity or its subsequent processing.

### Trigger Stage

According to their trigger stage, hooks are classified as pre-operation hooks and post-operation hooks.

Pre-operation hooks are triggered before the associated activity is performed. Post-operation hooks are triggered after the associated activity has been performed. Typical examples include hooks triggered before or after session establishment, tool invocation, and context compaction.

### Hook Capabilities

Hooks provide the following capabilities:

* Observation: A hook provides information about an agent activity for audit recording, such as tool invocation arguments, execution results, error information, or execution status.

* Intervention: A hook allows an agent activity or its subsequent processing to be affected, including by blocking or cancelling the activity, modifying its parameters or content, or changing information passed to subsequent processing.

Some hooks provide observation only, while others support both observation and intervention.


# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
