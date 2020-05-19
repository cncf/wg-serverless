# Extending

Serverless workflow model is aimed to be extensible. This allows you to add custom extensions
and still be compliant with the specification.

There are only a couple of rules for custom extensions:

- All custom extensions must be added to the "extensions" array of the workflow model definition.
- Extensions can define (optional) an "extensionschemaid" property. Workflow implementations should
then validate the extension JSON defined against the JSON schema specified.
- Each custom extension defined must have an unique "extensionid" property.
- Extension definition must be valid JSON format.

Below is an example custom extension for a workflow simulation scenario.
Note that all parameters of this extension are just made up for this example. You have
free reign to define your own.

Hope this gives implementors an idea on how to start adding their own custom extensions to the workflow model.

## Custom extension example

Let's say we are developing an extension which adds additional information
 to a serverless workflow that can be passed and processed by a simulation tool.

Our example extension can add "scenarios". Each scenario
can add "time" parameters for each of the workflow states and define a min and max value
within which the workflow state should be executed in. It should also add a "probability" parameter
which defines the probability that a state is triggered during the execution of the workflow.

So let's define a simple example workflow model and then add our custom extension into it:

```json
{  
   "name": "Simple Workflow",
   "functions": [
      {
         "name": "firstFunction",
         "type": "serverless"
      },
      {
         "name": "secondFunction",
         "type": "serverless"
      }
   ],
   "states":[  
      {  
         "name":"FirstOperation",
         "type":"operation",
         "start": {
            "kind": "default"
         },
         "actionMode":"Sequential",
         "actions":[  
            {  
               "name": "callFirstFunction",
               "functionRef": {
                  "refName": "firstFunction"
               }
            }
         ],
         "transition": {
            "nextState": "SecondOperation"
         }
      },
      {  
         "name":"SecondOperation",
         "type":"operation",
         "end": {
           "kind": "default"
         },
         "actionMode":"Sequential",
         "actions":[  
              {  
                 "name": "callSecondFunction",
                 "functionRef": {
                   "refName": "secondFunction"
                }
              }
         ]
      }
   ],
   "extensions": [
        .... our extension here ...
   ]
}
```

This workflow contains two Operation states with each calling a single function. In the "extensions" array we can add our custom extension, let's say like this:

```json
...
"extensions": [
    {
         "extensionid": "simextension",
         "scenario": {
            "name": "testSimScenario",
            "parameters": [
               {
                   "type": "exectime",
                   "forstate": "FirstOperation",
                   "min": "1s",
                   "max": "5s"
               },
               {
                  "type": "probability",
                  "forstate": "FirstOperation",
                  "value": "100%"
               },
               {
                  "type": "exectime",
                  "forstate": "SecondOperation",
                  "min": "4s",
                  "max": "15s"
                },
                {
                   "type": "probability",
                   "forstate": "SecondOperation",
                   "value": "100%"
                }
            ]
         }
    }
]
...
```

With an implementation of workflow simulation our scenario would for example start the workflow
and report if the execution and probability defined in the custom extension match the actual
values during workflow execution.
