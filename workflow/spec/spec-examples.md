## Serverless Workflow Specification - Examples

### Hello World Example
The following example illustrates a simple "Hello World" Serverless Workflow with three operation states. 
The digram below show how information is passed through the workflow and filter mechanism 
used to access and reason over the workflow state.


```json
{  
   "startsAt": "HelloWorld",
   "states":[  
      {  
         "name":"HelloWorld",
         "type":"OPERATION",
         "actionMode":"Sequential",
         "actions":[  
            {  
               "function":{
                  "name": "helloWorldFunction",
                  "resource": "functionResourse"
               }
            }
         ],
         "transition": {
            "nextState":"UpdateArg"  
         }
      },
      {  
         "name":"UpdateArg",
         "type":"OPERATION",
         "actionMode":"Sequential",
         "filter": {
            "inputPath":"$.payload",
            "resultPath":"$.ifttt.value1",
            "outputPath":"$.ifttt"
         },
         "transition": {
            "nextState":"SaveResult"
         }
      },
      {  
         "name":"SaveResult",
         "type":"OPERATION",
         "end":true,
         "actionMode":"Sequential",
         "actions":[  
            {  
               "function":{
                  "name": "saveResults",
                  "resource": "functionResourse"
               }
            }
         ]
      }
   ]
}
```

<p align="center">
<img src="media/helloworldexample.png" with="480px" height="270px" alt="Hello World Example"/>
</p>