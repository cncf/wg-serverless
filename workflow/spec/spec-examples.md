## Serverless Workflow Specification - Examples

## Table of Contents

- [Hello World](#Hello-World-Example)
- [Greeting](#Greeting-Example)
- [Event-based greeting](#Event-Based-Greeting-Example)
- [Solving Math Problems (ForEach)](#Solving-Math-Problems-Example)
- [Parallel Execution](#Parallel-Execution-Example)
- [Applicant Request Decision (Switch + SubFlow)](#Applicant-Request-Decision-Example)
- [Provision Orders (Error Handling)](#Provision-Orders-Example)
- [Monitor Job for completion (Polling)](#Monitor-Job-Example)
- [Send CloudEvent on Workflow Completion](#Send-CloudEvent-On-Workfow-Completion-Example)

### Hello World Example

#### Description

This example uses two relay states. The "Hello" state statically injects the following JSON into its data input:

```json
{
  "result": "Hello"
}
```

which then becomes the data input of the transition "World" state.
The "World" state merges its data input with it's injected JSON and uses a filter to set its data output to the 
value of the "result" property. Since it is an end state, it's data output becomes the workflow data output:

```
"Hello World!"
```

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
"name": "Hello World Workflow",
"description": "Static Hello World",
"startsAt": "Hello",
"states":[  
  {  
     "name":"Hello",
     "type":"RELAY",
     "inject": {
        "result": "Hello"
     },
     "transition": {
       "nextState": "World"
     }
  },
  {  
     "name":"World",
     "type":"RELAY",
     "inject": {
        "result": " World!"
     },
     "stateDataFilter": {
       "dataOutputPath": "$.result"
     },
     "end": {
       "type": "DEFAULT"
     }
  }
]
}
```
</td>
<td valign="top">

```yaml
name: Hello World Workflow
description: Static Hello World
startsAt: Hello
states:
- name: Hello
  type: RELAY
  inject:
    result: Hello
  transition:
    nextState: World
- name: World
  type: RELAY
  inject:
    result: " World!"
  stateDataFilter:
    dataOutputPath: "$.result"
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/helloworldexample.png" with="250px" height="400px" alt="Hello World Example"/>
</p>

### Greeting Example

#### Description

This example shows a single Operation state with one action that calls the "greeting" function. 
The workflow data input is assumed to be the name of the person to greet:

```json
{
  "greet": {
    "name": "John"
  }
}
```

The results of the action is assumed to be the full greeting for the provided persons name:

```json
{
  "payload": {
    "greeting": "Welcome to Serverless Workflow, John!"
  }
}
```

The states action data filter selects the greeting object from the function return to be placed into the state data.
Then the states state data filter selects only the greeting object to be returned as its data output, which 
becomes the workflow data output (as it is an end state):
```
   "Welcome to Serverless Workflow, John!" 
```

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
"name": "Greeting Workflow",
"description": "Greet Someone",
"startsAt": "Greet",
"functions": [
  {
     "name": "greetingFunction",
     "resource": "functionResourse"
  }
],
"states":[  
  {  
     "name":"Greet",
     "type":"OPERATION",
     "actionMode":"SEQUENTIAL",
     "actions":[  
        {  
           "functionRef": {
              "name": "greetingFunction",
              "parameters": {
                "name": "$.greet.name"
              }
           },
           "actionDataFilter": {
              "dataResultsPath": "$.payload.greeting"
           }
        }
     ],
     "stateDataFilter": {
        "dataOutputPath": "$.greeting"
     },
     "end": {
       "type": "DEFAULT"
     }
  }
]
}
```
</td>
<td valign="top">

```yaml
name: Greeting Workflow
description: Greet Someone
startsAt: Greet
functions:
- name: greetingFunction
  resource: functionResourse
states:
- name: Greet
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: greetingFunction
      parameters:
        name: "$.greet.name"
    actionDataFilter:
      dataResultsPath: "$.payload.greeting"
  stateDataFilter:
    dataOutputPath: "$.greeting"
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/greetingexample.png" with="400px" height="400px" alt="Greeting Example"/>
</p>

### Event Based Greeting Example

#### Description

This example shows a single Event state with one action that calls the "greeting" function. 
The event state consumes cloud events of type "greetingEventType". When an even with this type
is consumed, the Event state performs a single action that calls the defined "greeting" function.

For the sake of the example we assume that the cloud event we will consume has the format:

```json
{
    "specversion" : "1.0",
    "type" : "greetingEventType",
    "source" : "greetingEventSource",
    "data" : {
      "greet": {
          "name": "John"
        }
    }
}
```

The results of the action is assumed to be the full greeting for the provided persons name:

```json
{
  "payload": {
    "greeting": "Welcome to Serverless Workflow, John!"
  }
}
```
Note that in the workflow definition you can see two filters defined. The event data filter defined inside the consume element:

```json
{
  "eventDataFilter": {
    "dataOutputPath": "$.data.greet"
  } 
}
```

which is triggered when the greeting event is consumed. It extracts its "data.greet" of the event and 
merges it with the states data.

The second, a state data filter, which is defined on the event state itself:

```json
{
  "stateDataFilter": {
     "dataOutputPath": "$.payload.greeting"
  }
}
```

filters what is selected to be the state data output which then becomes the workflow data output (as it is an end state):

```
   "Welcome to Serverless Workflow, John!" 
```

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
"name": "Event Based Greeting Workflow",
"description": "Event Based Greeting",
"startsAt": "Greet",
"events": [
 {
  "name": "GreetingEvent",
  "type": "greetingEventType",
  "source": "greetingEventSource"
 }
],
"functions": [
  {
     "name": "greetingFunction",
     "resource": "functionResourse"
  }
],
"states":[  
  {  
     "name":"Greet",
     "type":"EVENT",
     "eventsActions": [{
         "eventRef": {
            "name": "GreetingEvent"
         },
         "eventDataFilter": {
            "inputPath": "$.data.greet"
         },
         "actions":[  
            {  
               "functionRef": {
                  "name": "greetingFunction",
                  "parameters": {
                    "name": "$.greet.name"
                  }
               }
            }
         ]
     }],
     "stateDataFilter": {
        "dataOutputPath": "$.payload.greeting"
     },
     "end": {
       "type": "DEFAULT"
     }
  }
]
}
```
</td>
<td valign="top">

```yaml
name: Event Based Greeting Workflow
description: Event Based Greeting
startsAt: Greet
events:
- name: GreetingEvent
  type: greetingEventType
  source: greetingEventSource
functions:
- name: greetingFunction
  resource: functionResourse
states:
- name: Greet
  type: EVENT
  eventsActions:
  - eventRef:
      name: GreetingEvent
    eventDataFilter:
      inputPath: "$.data.greet"
    actions:
    - functionRef:
        name: greetingFunction
        parameters:
          name: "$.greet.name"
  stateDataFilter:
    dataOutputPath: "$.payload.greeting"
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/eventbasedgreetingexample.png" with="400px" height="400px" alt="Event Based Greeting Example"/>
</p>

### Solving Math Problems Example

#### Description

In this example we show how to iterate over some data input using the ForEach state. 
The state will iterate over a collection of simple math expressions which are 
passed in as the workflow data input:

```json
    {
      "expressions": ["2+2", "4-1", "10x3", "20/2"]
    }
```
The ForEach state will execute a single defined operation state for each math expression. The operation
state contains an action which calls a serverless function which actually solves the expression
and returns its result.

Results of all mathe expressions are accumulated into the data output of the ForEach state which become the final
result of the workflow execution.

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
"name": "Solve Math Problems Workflow",
"description": "Solve math problems",
"startsAt": "Solve",
"functions": [
{
  "name": "solveMathExpressionFunction",
  "resource": "functionResourse"
}
],
"states":[  
{   
 "name":"Solve",
 "type":"FOREACH",
 "inputCollection": "$.expressions",
 "inputParameter": "$.singleexpression",
 "outputCollection": "$.results",
 "startsAt": "GetResults",
 "states": [
{  
    "name":"GetResults",
    "type":"OPERATION",
    "actionMode":"SEQUENTIAL",
    "actions":[  
       {  
          "functionRef": {
             "name": "solveMathExpressionFunction",
             "parameters": {
               "expression": "$.singleexpression"
             }
          }
       }
    ],
    "end": {
      "type": "DEFAULT"
    }
}
 ],
 "stateDataFilter": {
    "dataOutputPath": "$.results"
 },
 "end": {
   "type": "DEFAULT"
 }
}
]
}
```
</td>
<td valign="top">

```yaml
name: Solve Math Problems Workflow
description: Solve math problems
startsAt: Solve
functions:
- name: solveMathExpressionFunction
  resource: functionResourse
states:
- name: Solve
  type: FOREACH
  inputCollection: "$.expressions"
  inputParameter: "$.singleexpression"
  outputCollection: "$.results"
  startsAt: GetResults
  states:
  - name: GetResults
    type: OPERATION
    actionMode: SEQUENTIAL
    actions:
    - functionRef:
        name: solveMathExpressionFunction
        parameters:
          expression: "$.singleexpression"
    end:
      type: DEFAULT
  stateDataFilter:
    dataOutputPath: "$.results"
  end:
    type: DEFAULT

```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/loopingexample.png" with="400px" height="400px" alt="Looping Example"/>
</p>

### Parallel Execution Example

#### Description

This example uses a parallel state to execute two branches (simple wait states) at the same time. 
Note that the waitForCompletion flag is set to "false" so as soon as the "ShortDelay" delay state finishes,
the workflow complete execution. If waitForCompletion was set to true, the workflow would complete after both
of the branches are done.

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
"name": "Parallel Execution Workflow",
"description": "Executes two branches in parallel",
"startsAt": "ParallelExec",
"states":[  
  {  
     "name":"ParallelExec",
     "type":"PARALLEL",
     "branches": [
        {
          "name": "Branch1",
          "startsAt": "ShortDelay",
          "states": [
            {
                "name":"ShortDelay",
                 "type":"DELAY",
                 "timeDelay": "PT15S",
                 "end": {
                   "type": "DEFAULT"
                 }
            }
          ],
          "waitForCompletion": false
        },
        {
          "name": "Branch2",
          "startsAt": "LongDelay",
          "states": [
             {
                 "name":"LongDelay",
                  "type":"DELAY",
                  "timeDelay": "PT2M",
                  "end": {
                    "type": "DEFAULT"
                  }
             }      
          ],
          "waitForCompletion": false
        }
     ],
     "end": {
       "type": "DEFAULT"
     }
  }
]
}
```
</td>
<td valign="top">

```yaml
name: Parallel Execution Workflow
description: Executes two branches in parallel
startsAt: ParallelExec
states:
- name: ParallelExec
  type: PARALLEL
  branches:
  - name: Branch1
    startsAt: ShortDelay
    states:
    - name: ShortDelay
      type: DELAY
      timeDelay: PT15S
      end:
        type: DEFAULT
    waitForCompletion: false
  - name: Branch2
    startsAt: LongDelay
    states:
    - name: LongDelay
      type: DELAY
      timeDelay: PT2M
      end:
        type: DEFAULT
    waitForCompletion: false
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/parallelexample.png" with="400px" height="400px" alt="Parallel Example"/>
</p>

### Applicant Request Decision Example

#### Description

This example shows off the switch state and the subflow state. The workflow is started with application information data as input:

```json
    {
      "applicant": {
        "fname": "John",
        "lname": "Stockton",
        "age": 22,
        "email": "js@something.com"
      }
    }
```

We use the switch state with two choices to determine if the application should be made based on the applicants age. 
If the applicants age is over 18 we start the application (subflow state). Otherwise the workflow notifies the 
 applicant of the rejection. 

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
   "name": "Applicant Request Decision Workflow",
   "description": "Determine if applicant request is valid",
   "startsAt": "CheckApplication",
   "functions": [
     {
        "name": "sendRejectionEmailFunction",
        "resource": "functionResourse"
     }
   ],
   "states":[  
      {  
         "name":"CheckApplication",
         "type":"SWITCH",
         "choices": [
            {
              "path": "$.applicant.age",
              "value": "18",
              "operator": "GreaterThanEquals",
              "transition": {
                "nextState": "StartApplication"
              }
            },
            {
              "path": "$.applicant.age",
              "value": "18",
              "operator": "LessThan",
              "transition": {
                "nextState": "RejectApplication"
              }
            }
         ],
         "default": {
            "nextState": "RejectApplication"
         }
      },
      {
        "name": "StartApplication",
        "type": "SUBFLOW",
        "workflowId": "startApplicationWorkflowId",
        "end": {
          "type": "DEFAULT"
        }
      },
      {  
        "name":"RejectApplication",
        "type":"OPERATION",
        "actionMode":"SEQUENTIAL",
        "actions":[  
           {  
              "functionRef": {
                 "name": "sendRejectionEmailFunction",
                 "parameters": {
                   "applicant": "$.applicant"
                 }
              }
           }
        ],
        "end": {
          "type": "DEFAULT"
        }
    }
   ]
} 
```
</td>
<td valign="top">

```yaml
name: Applicant Request Decision Workflow
description: Determine if applicant request is valid
startsAt: CheckApplication
functions:
- name: sendRejectionEmailFunction
  resource: functionResourse
states:
- name: CheckApplication
  type: SWITCH
  choices:
  - path: "$.applicant.age"
    value: '18'
    operator: GreaterThanEquals
    transition:
      nextState: StartApplication
  - path: "$.applicant.age"
    value: '18'
    operator: LessThan
    transition:
      nextState: RejectApplication
  default:
    nextState: RejectApplication
- name: StartApplication
  type: SUBFLOW
  workflowId: startApplicationWorkflowId
  end:
    type: DEFAULT
- name: RejectApplication
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: sendRejectionEmailFunction
      parameters:
        applicant: "$.applicant"
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/switchstateexample.png" with="400px" height="400px" alt="Switch State Example"/>
</p>

### Provision Orders Example

#### Description

In this example we show off the states error handling capability. The workflow data input that's passed in contains 
missing order information that causes the function in the "ProvisionOrder" state to throw a runtime exception. With the "onError" expression we
can transition the workflow to different error handling states depending on the error thrown. Each type of error 
in this example is handled by simple delay states, each including an error data filter which sets the exception info as their 
data output. If no error is caught the workflow can transition to the "ApplyOrder" state.

Workflow data is assumed to me:
```json
    {
      "order": {
        "id": "",
        "item": "laptop",
        "quantity": "10"
      }
    }
```

The data output of the workflow contains the information of the exception caught during workflow execution.

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{  
"name": "Provision Orders",
"description": "Provision Orders and handle errors thrown",
"startsAt": "ProvisionOrder",
"functions": [
  {
     "name": "provisionOrderFunction",
     "resource": "functionResourse"
  }
],
"states":[  
  {  
    "name":"ProvisionOrder",
    "type":"OPERATION",
    "actionMode":"SEQUENTIAL",
    "actions":[  
       {  
          "functionRef": {
             "name": "provisionOrderFunction",
             "parameters": {
               "order": "$.order"
             }
          }
       }
    ],
    "onError": [
       {
         "expression": {
            "language": "spel",
            "body": "$.exception.name is 'MissingOrderIdException'"
         },
         "transition": {
           "nextState": "MissingId"
         },
         "errorDataFilter": {
          "dataOutputPath": "$.exception"
         }
       },
       {
         "expression": {
           "language": "spel",
           "body": "$.exception.name is 'MissingOrderItemException'"
         },
         "transition": {
           "nextState": "MissingItem"
         },
         "errorDataFilter": {
           "dataOutputPath": "$.exception"
          }
       },
       {
        "expression": {
          "language": "spel",
          "body": "$.exception.name is 'MissingOrderQuantityException'"
        },
        "transition": {
          "nextState": "MissingQuantity"
        },
        "errorDataFilter": {
          "dataOutputPath": "$.exception"
         }
       }
    ],
    "stateDataFilter": {
       "dataOutputPath": "$.exception"
    },
    "transition": {
       "nextState":"ApplyOrder"
    }
},
{
   "name": "MissingId",
   "type": "SUBFLOW",
   "workflowId": "handleMissingIdExceptionWorkflow",
   "end": {
     "type": "DEFAULT"
   }
},
{
   "name": "MissingItem",
   "type": "SUBFLOW",
   "workflowId": "handleMissingItemExceptionWorkflow",
   "end": {
     "type": "DEFAULT"
   }
},
{
   "name": "MissingQuantity",
   "type": "SUBFLOW",
   "workflowId": "handleMissingQuantityExceptionWorkflow",
   "end": {
     "type": "DEFAULT"
   }
},
{
   "name": "ApplyOrder",
   "type": "SUBFLOW",
   "workflowId": "applyOrderWorkflowId",
   "end": {
     "type": "DEFAULT"
   }
}
]
}
```
</td>
<td valign="top">

```yaml
name: Provision Orders
description: Provision Orders and handle errors thrown
startsAt: ProvisionOrder
functions:
- name: provisionOrderFunction
  resource: functionResourse
states:
- name: ProvisionOrder
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: provisionOrderFunction
      parameters:
        order: "$.order"
  onError:
  - expression:
      language: spel
      body: "$.exception.name is 'MissingOrderIdException'"
    transition:
      nextState: MissingId
    errorDataFilter:
      dataOutputPath: "$.exception"
  - expression:
      language: spel
      body: "$.exception.name is 'MissingOrderItemException'"
    transition:
      nextState: MissingItem
    errorDataFilter:
      dataOutputPath: "$.exception"
  - expression:
      language: spel
      body: "$.exception.name is 'MissingOrderQuantityException'"
    transition:
      nextState: MissingQuantity
    errorDataFilter:
      dataOutputPath: "$.exception"
  stateDataFilter:
    dataOutputPath: "$.exception"
  transition:
    nextState: ApplyOrder
- name: MissingId
  type: SUBFLOW
  workflowId: handleMissingIdExceptionWorkflow
  end:
    type: DEFAULT
- name: MissingItem
  type: SUBFLOW
  workflowId: handleMissingItemExceptionWorkflow
  end:
    type: DEFAULT
- name: MissingQuantity
  type: SUBFLOW
  workflowId: handleMissingQuantityExceptionWorkflow
  end:
    type: DEFAULT
- name: ApplyOrder
  type: SUBFLOW
  workflowId: applyOrderWorkflowId
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/handlerrorsexample.png" with="400px" height="400px" alt="Handle Errors Example"/>
</p>

### Monitor Job Example

#### Description

In this example we submit a job via an operation state action (serverless function call). It is assumed that it takes some time for 
the submitted job to complete and that it's completion can be checked via another separate serverless function call.
 
To check for completion we first wait 5 seconds and then get the results of the "CheckJob" serverless function. 
Depending on the results of this we either return the results or transition back to waiting and checking the job completion. 
This is done until the job submission returns "SUCCEEDED" or "FAILED" and the job submission results are reported before workflow
finishes execution.

In the case job submission raises a runtime error, we transition to a SubFlow state which handles the job submission issue.


#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{   
"name": "Job Monitoring",
"description": "Monitor finished execution of a submitted job",
"startsAt": "SubmitJob",
"functions": [
 {
   "name": "submitJob",
   "resource": "submitJobResource"
 },
 {
    "name": "checkJobStatus",
    "resource": "checkJobStatusResource"
 },
 {
    "name": "reportJobSuceeded",
    "resource": "reportJobSuceededResource"
 },
 {
    "name": "reportJobFailed",
    "resource": "reportJobFailedResource"
 }
],
"states":[  
  {  
    "name":"SubmitJob",
    "type":"OPERATION",
    "actionMode":"SEQUENTIAL",
    "actions":[  
    {  
       "functionRef": {
          "name": "submitJob",
          "parameters": {
            "name": "$.job.name"
          }
       },
       "actionDataFilter": {
          "dataResultsPath": "$.jobuid" 
       }
    }
    ],
    "onError": [
    {
     "expression": {
         "language": "spel",
         "body": "$.exception != null"
      },
      "errorDataFilter": {
        "dataOutputPath": "$.exception"
      },
      "transition": {
        "nextState": "SubmitError"
      }
    }
    ],
    "stateDataFilter": {
       "dataOutputPath": "$.jobuid"
    },
    "transition": {
       "nextState":"WaitForCompletion"
    }
},
{
   "name": "SubmitError",
   "type": "SUBFLOW",
   "workflowId": "handleJobSubmissionErrorWorkflow",
   "end": {
     "type": "DEFAULT"
   }
},
{
   "name": "WaitForCompletion",
   "type": "DELAY",
   "timeDelay": "PT5S",
   "transition": {
      "nextState":"GetJobStatus"
   }
},
{  
    "name":"GetJobStatus",
    "type":"OPERATION",
    "actionMode":"SEQUENTIAL",
    "actions":[  
    {  
      "functionRef": {
          "name": "checkJobStatus",
          "parameters": {
            "name": "$.jobuid"
          }
       },
       "actionDataFilter": {
        "dataResultsPath": "$.jobstatus"
       }
    }
    ],
    "stateDataFilter": {
       "dataOutputPath": "$.jobstatus"
    },
    "transition": {
       "nextState":"DetermineCompletion"
    }
},
{  
 "name":"DetermineCompletion",
 "type":"SWITCH",
 "choices": [
   {
     "path": "$.jobstatus",
     "value": "SUCCEEDED",
     "operator": "Equals",
     "transition": {
       "nextState": "JobSucceeded"
     }
   },
   {
     "path": "$.jobstatus",
     "value": "FAILED",
     "operator": "Equals",
     "transition": {
       "nextState": "JobFailed"
     }
   }
  ],
  "default": "WaitForCompletion"
},
{  
   "name":"JobSucceeded",
   "type":"OPERATION",
   "actionMode":"SEQUENTIAL",
   "actions":[  
   {  
      "functionRef": {
         "name": "reportJobSuceeded",
         "parameters": {
           "name": "$.jobuid"
         }
      }
   }
   ],
   "end": {
     "type": "DEFAULT"
   }
},
{  
  "name":"JobFailed",
  "type":"OPERATION",
  "actionMode":"SEQUENTIAL",
  "actions":[  
  {  
     "functionRef": {
        "name": "reportJobFailed",
        "parameters": {
          "name": "$.jobuid"
        }
     }
  }
  ],
  "end": {
    "type": "DEFAULT"
  }
}
]
}
```
</td>
<td valign="top">

```yaml
name: Job Monitoring
description: Monitor finished execution of a submitted job
startsAt: SubmitJob
functions:
- name: submitJob
  resource: submitJobResource
- name: checkJobStatus
  resource: checkJobStatusResource
- name: reportJobSuceeded
  resource: reportJobSuceededResource
- name: reportJobFailed
  resource: reportJobFailedResource
states:
- name: SubmitJob
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: submitJob
      parameters:
        name: "$.job.name"
    actionDataFilter:
      dataResultsPath: "$.jobuid"
  onError:
  - expression:
      language: spel
      body: "$.exception != null"
    errorDataFilter:
      dataOutputPath: "$.exception"
    transition:
      nextState: SubmitError
  stateDataFilter:
    dataOutputPath: "$.jobuid"
  transition:
    nextState: WaitForCompletion
- name: SubmitError
  type: SUBFLOW
  workflowId: handleJobSubmissionErrorWorkflow
  end:
    type: DEFAULT
- name: WaitForCompletion
  type: DELAY
  timeDelay: PT5S
  transition:
    nextState: GetJobStatus
- name: GetJobStatus
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: checkJobStatus
      parameters:
        name: "$.jobuid"
    actionDataFilter:
      dataResultsPath: "$.jobstatus"
  stateDataFilter:
    dataOutputPath: "$.jobstatus"
  transition:
    nextState: DetermineCompletion
- name: DetermineCompletion
  type: SWITCH
  choices:
  - path: "$.jobstatus"
    value: SUCCEEDED
    operator: Equals
    transition:
      nextState: JobSucceeded
  - path: "$.jobstatus"
    value: FAILED
    operator: Equals
    transition:
      nextState: JobFailed
  default: WaitForCompletion
- name: JobSucceeded
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: reportJobSuceeded
      parameters:
        name: "$.jobuid"
  end:
    type: DEFAULT
- name: JobFailed
  type: OPERATION
  actionMode: SEQUENTIAL
  actions:
  - functionRef:
      name: reportJobFailed
      parameters:
        name: "$.jobuid"
  end:
    type: DEFAULT
```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/jobmonitoringexample.png" with="400px" height="400px" alt="Job Monitoring Example"/>
</p>

### Send CloudEvent On Workfow Completion Example

#### Description

This example shows how we can produce a CloudEvent on completion of a workflow. Let's say we have the following
workflow data:

```json
{
  "orders": [{
    "id": "123",
    "item": "laptop",
    "quantity": "10"
  },
  {
      "id": "456",
      "item": "desktop",
      "quantity": "4"
    }]
}
```
Our workflow in this example uses a ForEach state to provision the orders in parallel. The "provisionOrder" function
used is assumed to have the following results:

```json
{
  "provisionedOrders": [
      {
        "id": "123",
        "outcome": "SUCCESS"
      }
  ]
}
```

After orders have been provisioned the ForEach states defines the end property which stops workflow execution.
It defines its end definition to be of type "EVENT" in which case a CloudEvent will be produced which can be consumed
by other orchestration workflows or other interested consumers. 
Note that we define the event to be produced in the workflows "events" property.
The data attached to the event contains the information on provisioned orders by this workflow. So the produced 
CloudEvent upon completion of the workflow could look like:

```json
{
  "specversion" : "1.0",
  "type" : "provisionCompleteType",  
  "datacontenttype" : "application/json",
  ...
  "data": {
    "provisionedOrders": [
        {
          "id": "123",
          "outcome": "SUCCESS"
        },
        {
          "id": "456",
          "outcome": "FAILURE"
        }
      ]
  }
}
```

#### Workflow Definition

<table>
<tr>
    <th>JSON</th>
    <th>YAML</th>
</tr>
<tr>
<td valign="top">

```json
{
"id": "sendcloudeventonprovision",
"name": "Send CloudEvent on provision completion",
"version": "1.0",
"startsAt": "ProvisionOrdersState",
"events": [
{
    "name": "provisioningCompleteEvent",
    "type": "provisionCompleteType"
}
],
"functions": [
{
    "name": "provisionOrderFunction",
    "resource": "functionResourse"
}
],
"states": [
{
    "name": "ProvisionOrdersState",
    "type": "FOREACH",
    "inputCollection": "$.orders",
    "inputParameter": "$.singleorder",
    "outputCollection": "$.results",
    "startsAt": "DoProvision",
    "states": [
    {
        "name": "DoProvision",
        "type": "OPERATION",
        "actions": [
        {
            "functionref": {
                "refname": "provisionOrderFunction",
                "parameters": {
                    "order": "$.order"
                }
            }
        }
        ],
        "end": {
            "type": "DEFAULT"
        }
    }
    ],
    "stateDataFilter": {
        "dataOutputPath": "$.provisionedOrders"
    },
    "end": {
        "type": "EVENT",
        "produceEvent": {
            "nameRef": "provisioningCompleteEvent",
            "data": "$.provisionedOrders"
        }
    }
}
]
}
```
</td>
<td valign="top">

```yaml
id: sendcloudeventonprovision
name: Send CloudEvent on provision completion
version: '1.0'
startsAt: ProvisionOrdersState
events:
- name: provisioningCompleteEvent
  type: provisionCompleteType
functions:
- name: provisionOrderFunction
  resource: functionResourse
states:
- name: ProvisionOrdersState
  type: FOREACH
  inputCollection: "$.orders"
  inputParameter: "$.singleorder"
  outputCollection: "$.results"
  startsAt: DoProvision
  states:
  - name: DoProvision
    type: OPERATION
    actions:
    - functionref:
        refname: provisionOrderFunction
        parameters:
          order: "$.order"
    end:
      type: DEFAULT
  stateDataFilter:
    dataOutputPath: "$.provisionedOrders"
  end:
    type: EVENT
    produceEvent:
      nameRef: provisioningCompleteEvent
      data: "$.provisionedOrders"

```
</td>
</tr>
</table>

#### Worfklow Diagram

<p align="center">
<img src="media/sendcloudeentonworkflowcompletion.png" with="400px" height="400px" alt="Send CloudEvent on Workflow Completion Example"/>
</p>
