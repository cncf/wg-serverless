# Proposals

Use this folder to submit proposals for new work streams related to the
CNCF Serverless Working Group.

To submit a new proposal, open a PR adding a new document to this
"proposals" folder and add an agenda item to the Serverless WG's
[agenda](https://docs.google.com/document/d/1OVF68rpuPK5shIHILK9JOqlZBbfe91RNzQ7u_P7YCDE/edit?ts=5a1da559#)
document to raise aware of the PR.

If the work stream extends the scope of the Working Group beyond what the
TOC has previously agreed to, then the request might need to be taken to
the TOC after the Working Group has agreed to it.

## Proposed Work Streams

### Events

At this moment the group decided to move forward with `open-events`
leveraging `Cloudevents` and `Open-CADF-events` content as input. The
on-going work for the current spec is hosted at
[https://github.com/cloudevents/spec](https://github.com/cloudevents/spec).

### Orchestration / Workflows / Chaining

The way functions are orchestrated, workflows are built and chaining functions were deemed to be not in scope for CloudEvents, but maybe in scope for the Serverless working group.

There are issues such as the [correlation id](https://github.com/cloudevents/spec/pull/128), [event history/ chaining](https://github.com/cloudevents/spec/issues/204), [event nesting](https://github.com/cloudevents/spec/issues/72) and concerns regarding encryption/ signed content/ integrity that do need this to be defined. There are increasing number of question regarding who is allowed to modify a CloudEvent, how an event is forwarded and so on.

Having a field in CloudEvents that is to be used for say correlation, but no clear specification for who uses that field and what that field contains is not ideal.

### Function Signatures

There are multiple providers that have different ways to handle functions. Using multiple providers, switching providers and developing functions would be significantly better experiences if there was a common structure for function signatures.

Some examples can be seen in [function-signatures-examples.md](function-signatures-examples.md) but the same issues are present in other supported languages too.

### APIs for accessing CloudEvents

### CE Client SDK

See: https://github.com/cloudevents/spec/issues/205

