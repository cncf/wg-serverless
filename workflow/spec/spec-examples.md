## Serverless Workflow Specification - Examples

## Table of Contents

- [Greeting](#Greeting-Example)


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
   "startsAt": "greetState",
   "states":[  
      {  
         "name":"greetState",
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