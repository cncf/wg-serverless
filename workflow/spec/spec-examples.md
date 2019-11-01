## Serverless Workflow Specification - Examples

### Hello World Example
The following example illustrates a simple "Hello World" Serverless Workflow with three operation states
and an end state. The digram below show how information is passed through the workflow and filter mechanism 
used to access and reason over the workflow state.


```json
{  
   "starts-at": "HelloWorld",
   "states":[  
      {  
         "name":"HelloWorld",
         "type":"OPERATION",
         "action-mode":"Sequential",
         "actions":[  
            {  
               "function":"hello"
            }
         ],
         "next-state":"UpdateArg"
      },
      {  
         "name":"UpdateArg",
         "type":"OPERATION",
         "action-mode":"Sequential",
         "InputPath":"$.payload",
         "ResultPath":"$.ifttt.value1",
         "OutputPath":"$.ifttt",
         "actions":[  

         ],
         "next-state":"SaveResult"
      },
      {  
         "name":"SaveResult",
         "type":"OPERATION",
         "end":true,
         "action-mode":"Sequential",
         "actions":[  
            {  
               "function":"save_resut"
            }
         ],
         "next-state":"STATE_END"
      }
   ]
}
```

<p align="center">
<img src="media/helloworldexample.png" with="480px" height="270px" alt="Hello World Example"/>
</p>