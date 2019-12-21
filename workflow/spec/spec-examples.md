## Serverless Workflow Specification - Examples

## Table of Contents

- [Greeting](#Greeting-Example)
- [Looping](#Looping-Example)


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

### Looping Example

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
   "name": "Looping Workflow",
   "description": "Solve math expressions",
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