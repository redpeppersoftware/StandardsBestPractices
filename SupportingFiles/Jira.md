# Jira Best Practices

This document outlines standard practices for interacting with the Jira project management software.

## Ticket Creation

When creating a ticket, the following fields/values are required before saving:

1) Priority
2) Name
3) Description
4) Steps to Reproduce
5) Expected Result
6) Actual Result

In addition it is __highly recommended to include a screenshot__ as an attachment to the ticket.

The following are recommended to include, but not required:

7) Environment (QA, Prod, Dev, etc)
8) Time of Encountering issue

## Ticket Template

The following is a description template for creating bugs with styling in Jira. Note that "Steps to Reproduce" is only relevant when reporting a bug, new features only require expected behavior and screenshots of UI mocks (if available).

```
*Description of Bug/Feature*
Short overview of the bug/feature

*Steps to Reproduce*
# Step 1 (EG: "Open support dialogue and enter text into report input")
# Step 2 (EG: "Press 'Send' button")
# Observed result (EG: "Observe that the modal does not close and the message is not sent")

*Expected Behavior*
Describe the expected result of the steps (EG: "Support dialogue should close and show confirmation that message was sent")
```

## Priority
If when creating a ticket as a developer you are unsure of priority, or if you require a time estimate be assigned to the ticket, please notify the Project Manager involved to assign these values.

## New Features
As a developer, unless you have specific ownership of defining new features on a project please ask the Project Manager assigned to the project to create and define new features. This is the preferred method for managing resource allocation in the company.
