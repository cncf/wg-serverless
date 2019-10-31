## References

State machines/Workflows have a long history in software design and development. The goal of this section is to list other tools and languages that had been used to define orchestration between different actors (software and human). The main intention here is to make sure that we cover as many use cases as possible leveraging existing approaches to avoid pitfalls from the past. 

### BPMN 2.0 by OMG

[Business Process Modeling and Notation (BPMN)](https://www.omg.org/spec/BPMN/2.0/About-BPMN/) was designed by Object Management Group (OMG) in collaboration with companies such as: IBM, Red Hat, Oracle, SAP, TIBCO, Software AG, among others. The latest version of the specification provides a rich set of constructs to defines workflows in a technology agnostic way. One of the main advatages of the BPMN spec is that it visually defines how a workflow should look like and most importantely, it defines the execution semantics of such workflows. 

Workflows modeled using the BPMN specification can express a large number of use cases, covering the use cases listed in the [Serverless Workflow Specification - Use Cases](spec-usecases.md). 

BPMN provides the following constructs to define workflows capabiltiies:
- Tasks: Orchestrate interactions between systems and people
  - User Task 
  - Service Task
  - Business Rule Task 
- Events: Emitting and catching events that are relevant to a workflow instance
  - Condition Events
  - Message Event: Enable communication between different workflow instances
    - Throw
    - Catch  
  - Timer Events  
- Gateways: Enable fork/join behaviours based on certain condition
  - Exclusive
  - Parallel
  - Complex  
- Aggregation: Provide a mechanism to deal with complexity when workflows become to large to understand
  - Embedded Sub Process
  - Call Activity

There are currently several implementations of the BPMN specification, including tooling and runtime environments to execute these workflows. 

The BPMN Specificaiton defines XML Schemas for defining and validating workflow definitions.  

Here are a the BPMN diagrams covering two examples listed in the [Serverless Workflow Specification - Use Cases](spec-usecases.md) section: 

** Loan Approval Workflow **

![Loan Approval Example](media/references/loan-approval-workflow.png)

You can find the BPMN XML which can be executed in a number of Open Source and Propietary engines [here](media/references/loan-approval-workflow.bpmn)

** Travel Booking Workflow **

![Travel Booking Example](media/references/travel-booking-workflow.png)

You can find the BPMN XML which can be executed in a number of Open Source and Propietary engines [here](media/references/travel-booking-workflow.bpmn)



### Others

If you know other related tools, languages, projects that can help this specification to be better, more scoped and serve a wide range of use cases please feel free to get in touch or send a PR to this section. 
