# Proposals

Use this folder to submit proposals for new work streams related to the
CNCF Serverless Working Group.

To submit a new proposal, open a PR adding a new document to this
"proposals" folder and add an agenda item to the Serverless WG's
[agenda](https://docs.google.com/document/d/1OVF68rpuPK5shIHILK9JOqlZBbfe91RNzQ7u_P7YCDE/edit?ts=5a1da559#)
document to raise aware of the PR.

If the work stream extends the scope of the Working Group beyond what the
TOC has previously agreed to, then the request might need to be taken to
the TOC after the Working Group has agreed to it.

## Proposed Work Streams

### Events

At this moment the group decided to move forward with `open-events`
leveraging `Cloudevents` and `Open-CADF-events` content as input. The
on-going work for the current spec is hosted at
[https://github.com/cloudevents/spec](https://github.com/cloudevents/spec).

### Orchestration / Workflows / Chaining

The way functions are orchestrated, workflows are built and chaining functions were deemed to be not in scope for CloudEvents, but maybe in scope for the Serverless working group.

There are issues such as the [correlation id](https://github.com/cloudevents/spec/pull/128), [event history/ chaining](https://github.com/cloudevents/spec/issues/204), [event nesting](https://github.com/cloudevents/spec/issues/72) and concerns regarding encryption/ signed content/ integrity that do need this to be defined. There are increasing number of question regarding who is allowed to modify a CloudEvent, how an event is forwarded and so on.

Having a field in CloudEvents that is to be used for say correlation, but no clear specification for who uses that field and what that field contains is not ideal.

### Function Signatures

There are multiple providers that have different ways to handle functions. Using multiple providers, switching providers and developing functions would be significantly better experiences if there was a common structure for function signatures.

#### Javascript/ NodeJS examples
- [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-handler.html)
```javascript
exports.myHandler = function(event, context) {
   ...
}
```
- [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node)
```javascript
// You must include a context, but other arguments are optional
module.exports = function(context) {
    // Additional inputs can be accessed by the arguments property
    if(arguments.length === 4) {
        context.log('This function has 4 inputs');
    }
};

// or you can include additional inputs in your arguments
module.exports = function(context, myTrigger, myInput, myOtherInput) {
    // function logic goes here :)
};
```

- [Google Cloud Functions](https://cloud.google.com/functions/docs/quickstart)
```javascript
/**
 * HTTP Cloud Function.
 *
 * @param {Object} req Cloud Function request context.
 * @param {Object} res Cloud Function response context.
 */
exports.helloGET = (req, res) => {
  res.send('Hello World!');
};
```

- [OpenFaaS](https://github.com/openfaas/faas)
```javascript
module.exports = (callback, context) => {
    callback(null, {"message": "You said: " + context})
}
```

- [OpenWhisk](https://github.com/apache/incubator-openwhisk/blob/master/docs/actions.md#creating-and-invoking-javascript-actions)
```javascript
function main(args) {
    return {payload: 'Hello world'};
}
```

#### Go examples

- [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/go-programming-model-handler-types.html)
```go
package main

import (
        "fmt"
        "context"
        "github.com/aws/aws-lambda-go/lambda"
)

type MyEvent struct {
        Name string `json:"name"`
}

func HandleRequest(ctx context.Context, name MyEvent) (string, error) {
        return fmt.Sprintf("Hello %s!", name.Name ), nil
}

func main() {
        lambda.Start(HandleRequest)
}
```

- [Google Cloud Functions](https://cloud.google.com/go/home)
```go
package main

import (
        "fmt"
        "log"

        // Imports the Google Cloud Storage client package.
        "cloud.google.com/go/storage"
        "golang.org/x/net/context"
)

func main() {
        ctx := context.Background()
        ...
}
```

- [Nuclio](https://github.com/nuclio/nuclio)
```go
package handler

import (
    "github.com/nuclio/nuclio-sdk-go"
)

func Handler(context *nuclio.Context, event nuclio.Event) (interface{}, error) {
    context.Logger.Info("Request received: %s", event.GetPath())

    return nuclio.Response{
        StatusCode:  200,
        ContentType: "application/text",
        Body: []byte("Response from handler"),
    }, nil
}
```

- [OpenFaaS](https://github.com/openfaas/faas/blob/master/sample-functions/BaseFunctions/golang/handler.go)
```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"
    "os"
)

func main() {
    input, err := ioutil.ReadAll(os.Stdin)
    if err != nil {
        log.Fatalf("Unable to read standard input: %s", err.Error())
    }
    fmt.Println(string(input))
}
```

- [OpenWhisk](https://github.com/apache/incubator-openwhisk/blob/master/docs/actions.md#creating-go-actions)
```go
package main

import "encoding/json"
import "fmt"
import "os"

func main() {
    //program receives one argument: the JSON object as a string
    arg := os.Args[1]

    // unmarshal the string to a JSON object
    var obj map[string]interface{}
    json.Unmarshal([]byte(arg), &obj)

    // can optionally log to stdout (or stderr)
    fmt.Println("hello Go action")

    name, ok := obj["name"].(string)
    if !ok { name = "Stranger" }

    // last line of stdout is the result JSON object as a string
    msg := map[string]string{"msg": ("Hello, " + name + "!")}
    res, _ := json.Marshal(msg)
    fmt.Println(string(res))
}
```

The same issues are present in other supported languages too.

### APIs for accessing CloudEvents

### CE Client SDK

See: https://github.com/cloudevents/spec/issues/205

