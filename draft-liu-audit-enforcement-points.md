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

informative:

...

--- abstract

This document identifies key audit enforcement points for agents, specifying the calls they cover and the audit records they provide. It also describes the hooks used to capture these records and how hook failures are handled.


--- middle

# Introduction {#intro}

Agents are increasingly acting on behalf of users across multiple services and administrative domains. Their activities may involve the original user, other agents, models, tools, and hosts. Information about these activities is distributed across participants, with no single party observing the entire interaction chain. AUDIT therefore aims to enable the collection and correlation of this information to provide visibility across the chain.




# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
