## Serverless Workflow Specification - Examples

## Table of Contents

- [Greeting](#Greeting-Example)
- [Solving Math Problems (Looping)](#Solving-Math-Problems-Example)
- [Parallel Execution](#Parallel-Execution-Example)
- [Applicant Request Decision (Switch + SubFlow)](#Applicant-Request-Decision-Example)


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

The state filter merges the action results into its output, and then uses outputPath to only return the greeting as its data
output, which then becomes the data output of the workflow itself (as it is the end state).

```
   "Welcome to Serverless Workflow, John!" 
```

#### Workflow JSON

```json
{  
   "name": "Greeting Workflow",
   "description": "Greet Someone",
   "startsAt": "Greet",
   "states":[  
      {  
         "name":"Greet",
         "type":"OPERATION",
         "actionMode":"SEQUENTIAL",
         "actions":[  
            {  
               "function":{
                  "name": "greetingFunction",
                  "resource": "functionResourse",
                  "parameters": {
                    "name": "$.greet.name"
                  }
               }
            }
         ],
         "filter": {
            "resultPath": "$.out",
            "outputPath": "$.out.payload.greeting"
         },
         "end": true
      }
   ]
}
```
#### Worfklow Diagram

<p align="center">
<img src="media/greetingexample.png" with="400px" height="400px" alt="Greeting Example"/>
</p>

### Solving Math Problems Example

#### Description

In this example we show off looping in an Operation state. The state will loop over a collection of simple math expressions which are 
passed in as the workflow data input:

```json
    {
      "expressions": ["2+2", "4-1", "10x3", "20/2"]
    }
```

The operation state contains an action which calls the serverless function that solves the math expression and returns its answer.

The results of the action is assumed to be the answer, for example for the first expression:

```
"4"
```

The state filter is then used to only return the results of the solved math expressions which becomes the workflow data output:


```json
["4", "3", "30", "10"]
```

#### Workflow JSON

```json
{  
   "name": "Solve Math Problems Workflow",
   "description": "Solve math problems",
   "startsAt": "Solve",
   "states":[  
      {  
         "name":"Solve",
         "type":"OPERATION",
         "actionMode":"SEQUENTIAL",
         "actions":[  
            {  
               "function":{
                  "name": "solveMathExpressionFunction",
                  "resource": "functionResourse",
                  "parameters": {
                    "expression": "$."
                  }
               }
            }
         ],
         "loop": {
             "inputCollection": "$.expressions",
             "outputCollection": "$.answers"
         },
         "filter": {
            "outputPath": "$.answers"
         },
         "end": true
      }
   ]
}
```

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

#### Workflow JSON

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
                     "end": true
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
                      "end": true
                 }      
              ],
              "waitForCompletion": false
            }
         ],
         "end": true
      }
   ]
}
```

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

#### Workflow JSON

```json
{  
   "name": "Applicant Request Decision Workflow",
   "description": "Determine if applicant request is valid",
   "startsAt": "CheckApplication",
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
         "default": "RejectApplication"
      },
      {
        "name": "StartApplication",
        "type": "SUBFLOW",
        "workflowId": "startApplicationWorkflowId",
        "end": true
      },
      {  
        "name":"RejectApplication",
        "type":"OPERATION",
        "actionMode":"SEQUENTIAL",
        "actions":[  
           {  
              "function":{
                 "name": "sendRejectionEmailFunction",
                 "resource": "functionResourse",
                 "parameters": {
                   "applicant": "$.applicant"
                 }
              }
           }
        ],
        "end": true
    }
   ]
}
```

#### Worfklow Diagram

<p align="center">
<img src="media/switchstateexample.png" with="400px" height="400px" alt="Switch State Example"/>
</p>