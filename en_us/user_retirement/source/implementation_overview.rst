
***********************
Implementation Overview
***********************

In the Open edX platform ecosystem, the user experience is enabled by several
services, such as LMS, Studio, ecommerce, credentials, discovery, and more. PII
about a user exists in many of these services, so each service containing PII
must be requested to remove/delete/unlink the data for that user in that
service.

A centralized process (the *driver* scripts) orchestrates all of these request.
The details of configuring these scripts are outlined in
:ref:`driver-setup`.

FIXME Each service must provide a REST API endpoint that will be used to
activate the removal of a user's PII from that service.  Here are some of those
endpoints:

The Retirement Workflow
***********************

The user retirement workflow is a configurable pipeline of building-block APIs
that are used to “forget” a user’s personally identifiable identification (PII)
as well as preventing that user from logging back in, and preventing re-use of
their username or email.  Depending on which thrid parties a given Open edX
site integrates with, the process may need to involve calling out to external
services or generating reports for later processing (also requiring subsequent
destruction of said reports).  Configurability and hackability was a design
goal from the beginning, so this tooling should be able to acommodate a wide
range of Open edX sites and custom use cases.

The workflow is designed to be linear and re-runnable, allowing recovery and
continuation in cases where a particular stage fails. The LMS is the source of
authority of where a user is in the retirement process via the
UserRetirementStatus model and associated APIs. UserRetirementStatus uses
RetirementState objects to track the progress through the workflow. 

.. digraph:: retirement_states_example
   :align: center

      //rankdir=LR;  // Rank Direction Left to Right
      ranksep = "0.3";

      node[fontname=Courier,fontsize=12,shape=box,group=main]
      PENDING -> RETIRING_ENROLLMENTS -> ENROLLMENTS_COMPLETE -> RETIRING_FORUMS -> FORUMS_COMPLETE -> "..." -> COMPLETE;

      node[group=bad];
      RETIRING_ENROLLMENTS -> ERRORED;
      RETIRING_FORUMS -> ERRORED;
      PENDING -> ABORTED;

      subgraph cluster_terminal_states {
          label = "Terminal States";
          labelloc = b  // put label at bottom
          {rank = same ERRORED COMPLETE ABORTED}
      }

The User Experience
*******************

From the learner's perspective, the vast majority of this process is obscured.
The Account page contains a new section called "Delete My Account" where the
learner may click the "Delete My Account" button and enter their password to
confirm their request.  Subsequently, all their browser sessions of logged off,
and they become locked out of their account.

An informational email is immediately sent to the learner as an indication of
their account deletion, after which they have a limited amount of time (defined
by the *cool-off period*) to contact the administrators and rescind their
request.

At this point, the learner's account has been deactivated, but *not* retired.
An entry in the UserRetirementStatus table is added, and their state set to
PENDING.
