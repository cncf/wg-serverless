## Serverless Workflow Specification - Use Cases

Use cases for the Serverless Workflow Specification highly depend on the reference implementations 
and the ecosystem available during workflow execution (available services functions/services/events, etc).

As mentioned in the [main specification document](spec.md) one of the main benefits of serverless workflows
is that they provide clear separation of business and orchestration logic for your serverless apps.
Developers can focus on solving business logic inside functions and write workflows 
that can orchestrate function invocations, provide data management for the invoked functions, as well as 
low-maintenance support for many cross-cutting concerns such as 
control-flow logic, error handling, retry definitions, parallel execution, etc.
