# User registration

Workflow to register a user after receiving a registartion request. In this case
a registration request will be received and the workflow will forward this request
to the registration backend asynchronously.

The workflow will wait for a `user-registered` event for 5 minutes.
If the event is received, a `registration-success` event will be
sent to the `notifications` server which will send an email to the
user. On the other hand, if no event has been received for 5
minutes a retry will take place and will perform a second retry
after 10 minutes, then a last one after 15 minutes (10 minutes + 5
minutes delta).

Once the retries are exhausted the user will be notified with a `registration-error` message.

## Technical overview

- Basic events are defined. Different types for the same payload.
- An event is received from the default server.
- An event is emited to the default server. Such event is asynchronous and a callback event is expected on the same server.
- The username is used as correlationToken.
- A retry will take place if no event is received.
- An event is sent to a different server if the function call succeeds.
- An event is sent to a different server if the function call fails after all the configured retries.

## Diagram

// TBD
