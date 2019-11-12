# Gardening

Simple workflow to integrate a MQTT broker with an HTTP backend. The workflow will
listen to sensor updates from the MQTT broker (humidity, light, noise, ...) and the
events will be transformed and sent to the HTTP backend.

The backend will receive an process such events. It can be that after a manual interaction or upon certain level of the values received,
the backend will emit an action event that will be transformed and forwarded to the MQTT broker.

## Technical overview

- Multiple events are received and different actions are taken.
- Message filtering using defined CloudEvent headers.
- Integration between different protocols (MQTT / HTTP)
- This is a stateless workflow.

## Diagram

// TBD