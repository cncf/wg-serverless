# Notify user registration

Simple notification workflow where a user is notified upon registration. It is assumed the registation is already completed.

[notify_user_registration.json](./notify_user_registration.json)

## Technical overview

- Basic events are defined.
- An event is received from the default server.
- An event is emited to the default server. Such event is asynchronous and no response is expected.
- A basic input transformation is needed to send the event.

## Diagram

// TBD
