# Examples - Brigade

[Brigade](https://github.com/brigadecore/brigade) is an open-source Kubernetes-native tool for doing event-driven
scripting. Brigade allows you to
* Script simple and complex workflows using JavaScript
* Chain together containers, running them in parallel or serially.
* Fire scripts based on times, GitHub events, Docker pushes, or any other trigger.
* Create pipelines for Kubernetes
* and much more

Brigade has a number of [examples](https://github.com/brigadecore/brigade/tree/master/docs/content/examples) which display 
JavaScript code that can be used to utilize different features of the project.

The purpose of this document is to show side-by-side the Brigade JS and the equivalent markup of the 
Serverless Workflow Specification. This can hopefully help compare and contrast the two and 
give a better understanding of both.

You can find a lot more information on Brigade in their [website](https://brigade.sh/).

## Table of Contents

- [Greeting With Parameters](#Greeting-With-Parameters)
- [Greeting With Error Checking](#Greeting-With-Error-Checking)
- [Handling Multiple Events](#Handling-Multiple-Events)
- [Grouping Actions](#Grouping-Actions)
- [Event Data](#Event-Data)
- [Action Results](#Action-Results)
- [Emit Events](#Emit-Events)

### Greeting With Parameters

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/advanced-01.js)

<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const { events, Job } = require("brigadier");

events.on("exec", exec);

function exec(e, p) {
    let j1 = new Job("j1", "alpine:3.7", ["echo hello"]);
    let j2 = new Job("j2", "alpine:3.7", ["echo goodbye"]);

    j1.run()
    .then(() => {
        return j2.run()
    })
    .then(() => {
        console.log("done");
    });
};
```

</td>
<td valign="top">

```yaml
id: greeting
name: Greeting Workflow
version: '1.0'
events:
- name: execEvent
  type: exec
functions:
- name: greetingFunction
  resource: alpine:3.7
  type: echo
- name: consoleLogFunction
  type: console
states:
- name: GreetingState
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    actions:
    - name: sayHelloAction
      functionRef:
        refName: greetingFunction
        parameters:
          greeting: hello
    - name: sayGoodbyeAction
      functionRef:
        refName: greetingFunction
        parameters:
          greeting: hello
    - name: logDoneAction
      functionRef:
        refName: consoleLogFunction
        parameters:
          log: done
  end:
    kind: DEFAULT
```

</td>
</tr>
</table>

### Greeting With Error Checking

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/advanced-03.js)

<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const { events, Job } = require("brigadier");

events.on("exec", exec);

async function exec(e, p) {
    let j1 = new Job("j1", "alpine:3.7", ["echo hello"]);
    // This will fail
    let j2 = new Job("j2", "alpine:3.7", ["exit 1"]);

    try {
        await j1.run();
        await j2.run();
        console.log("done");
    } catch (e) {
        console.log(`Caught Exception ${e}`);
    } 
};
```

</td>
<td valign="top">

```yaml
id: greetingwitherrorcheck
name: Greeting Workflow With Error Check
version: '1.0'
events:
- name: execEvent
  type: exec
functions:
- name: greetingFunction
  resource: alpine:3.7
  type: echo
- name: consoleLogFunction
  type: console
states:
- name: GreetingState
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    actions:
    - name: sayHelloAction
      functionRef:
        refName: greetingFunction
        parameters:
          greeting: hello
    - name: sayGoodbyeAction
      functionRef:
        refName: greetingFunction
        parameters:
          greeting: hello
    - name: logDoneAction
      functionRef:
        refName: consoleLogFunction
        parameters:
          log: done
  onError:
  - expression:
      language: spel
      body: "$.exception != null"
    transition:
      nextState: HandleErrorState
  end:
    kind: DEFAULT
- name: HandleErrorState
  type: OPERATION
  actions:
  - name: logErrorAction
    functionRef:
      refName: consoleLogFunction
      parameters:
        log: Caught Exception $.exception
  end:
    kind: DEFAULT
```

</td>
</tr>
</table>

### Handling Multiple Events

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/brigade-03.js)

<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const { events } = require("brigadier")

events.on("exec", () => {
  console.log("==> handling an 'exec' event")
})

events.on("push", () => {
  console.log(" **** I'm a GitHub 'push' handler")
})
```

</td>
<td valign="top">

```yaml
id: multieventworkflow
name: Multiple Events Workflow
version: '1.0'
events:
- name: execEvent
  type: exec
- name: pushEvent
  type: push
functions:
- name: consoleLogFunction
  type: console
states:
- name: GreetingState
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    actions:
    - name: logExecEventAction
      functionRef:
        refName: consoleLogFunction
        parameters:
          log: "==> handling an 'exec' event"
  - eventRefs:
    - pushEvent
    actions:
    - name: logPushEventAction
      functionRef:
        refName: consoleLogFunction
        parameters:
          log: "**** I'm a GitHub 'push' handler"
  end:
    kind: DEFAULT
```

</td>
</tr>
</table>

### Grouping Actions

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/brigade-12.js)

* Note: Serverless Workflow specification does not currently support grouping of actions. This 
would be beneficial to have as then errors and retries could be done per group rather than all actions
defined as a whole. This is definitely a feature we can add to the specification that would be beneficial
and a great feature. For now until this is added, grouping needs to be handled by separate states,
where error checking could be performed in each.

<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const { events, Job, Group } = require("brigadier")

events.on("exec", () => {
  var hello = new Job("hello", "alpine:3.4", ["echo hello"])
  var goodbye = new Job("goodbye", "alpine:3.4", ["echo goodbye"])

  var helloAgain = new Job("hello-again", "alpine:3.4", ["echo hello again"])
  var goodbyeAgain = new Job("bye-again", "alpine:3.4", ["echo bye again"])


  var first = new Group()
  first.add(hello)
  first.add(goodbye)

  var second = new Group()
  second.add(helloAgain)
  second.add(goodbyeAgain)

  first.runAll().then( () => second.runAll() )
})
```

</td>
<td valign="top">

```yaml
id: groupActionsWorkflow
name: Group Actions Workflow
version: '1.0'
events:
- name: execEvent
  type: exec
functions:
- name: echoFunction
  resource: alpine:3.7
  type: echo
states:
- name: FirstGreetGroup
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    actions:
    - name: firstHelloAction
      functionRef:
        refName: echoFunction
        parameters:
          message: hello
    - name: firstGoodbyeAction
      functionRef:
        refName: echoFunction
        parameters:
          message: goodbye
  transition:
    nextState: SecondGreetGroup
- name: SecondGreetGroup
  type: OPERATION
  actions:
  - name: secondHelloAction
    functionRef:
      refName: echoFunction
      parameters:
        message: hello-again
  - name: secondGoodbyeAction
    functionRef:
      refName: echoFunction
      parameters:
        message: bye-again
  end:
    kind: DEFAULT
```

</td>
</tr>
</table>

### Event Data

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/brigade-13.js)

* Note: Events within Serverless Workflow specification require them to have the CloudEvents format. CE specification
defines a "data" context attribute which includes the event payload that we are using in this example.

<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const { events } = require("brigadier")

events.on("exec", (e, p) => {
  console.log(">>> event " + e.type + " caused by " + e.provider)
  console.log(">>> project " + p.name + " clones the repo at " + p.repo.cloneURL)
})
```

</td>
<td valign="top">

```yaml
id: eventDataWorkflow
name: Event Data Workflow
version: '1.0'
events:
- name: execEvent
  type: exec
functions:
- name: consoleFunction
  type: console
states:
- name: LogEventData
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    eventDataFilter:
      dataOutputPath: "$.event"
    actions:
    - name: eventInfoAction
      functionRef:
        refName: consoleFunction
        parameters:
          log: ">>> event $event.type caused by $.event.data.provider"
    - name: projectInfoAction
      functionRef:
        refName: consoleFunction
        parameters:
          log: ">>> project $event.data.project.name clones the repo at by $.event.data.repo.cloneURL"
  end:
    kind: DEFAULT

```

</td>
</tr>
</table>

### Action Results

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/brigade-15.js)

* Note: Serverless Workflow specification does not have built-in storage options or custom built-in functions, 
storing of data needs to be available as a function that can be invoked during workflow execution.

* Note: It is assumed that the "dest" variable is part of the event payload, rather than hard-coded, injected, or set.

<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const { events, Job, Group } = require("brigadier")

events.on("exec", (e, p) => {
  var dest = "/mnt/brigade/share/hello.txt"
  var one = new Job("one", "alpine:3.4", ["echo hello > " + dest])
  var two = new Job("two", "alpine:3.4", ["echo world >> " + dest])
  var three = new Job("three", "alpine:3.4", ["cat " + dest])

  one.storage.enabled = true
  two.storage.enabled = true
  three.storage.enabled = true

  Group.runEach([one, two, three])
})
```

</td>
<td valign="top">

```yaml
id: actionResultsWorkflow
name: Action Results Workflow
version: '1.0'
events:
- name: execEvent
  type: exec
functions:
- name: greetingFunction
  resource: alpine:3.7
  type: echo
- name: storeToFileFunction
  resource: alpine:3.7
  type: filestore
states:
- name: ExecActionsAndStoreResults
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    eventDataFilter:
      dataOutputPath: "$.event"
    actions:
    - name: helloAction
      actionDataFilter:
        dataResultsPath: "$.helloResult"
      functionRef:
        refName: greetingFunction
        parameters:
          message: hello
    - name: worldAction
      actionDataFilter:
        dataResultsPath: "$.worldResults"
      functionRef:
        refName: greetingAction
        parameters:
          message: world
    - name: storeToFileAction
      functionRef:
        refName: storeToFileFunction
        parameters:
          destination: "$.event.destination"
          value: "$.helloResult $.worldResults"
  end:
    kind: DEFAULT

```

</td>
</tr>
</table>

### Emit Events

[Brigade Example](https://github.com/brigadecore/brigade/blob/master/docs/content/examples/brigade-19.js)

* Note: Events can be emitted in Serverless Workflow Specification on state transitions. This also shows that yes.
you can have eventsActions definition without any actions :)
<table>
<tr>
    <th>Brigade</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```javascript
const {events} = require("brigadier")

events.on("exec", function(e, project) {
  const e2 = {
    type: "next",
    provider: "exec-handler",
    buildID: e.buildID,
    workerID: e.workerID,
    cause: {event: e}
  }
  events.fire(e2, project)
})

events.on("next", (e) => {
  console.log(`fired ${e.type} caused by ${e.cause.event.type}`)
})
```

</td>
<td valign="top">

```yaml
id: eventDataWorkflow
name: Event Data Workflow
version: '1.0'
events:
- name: execEvent
  type: exec
- name: nextEvent
  type: next
functions:
- name: consoleLogFunction
  type: console
states:
- name: ExecEventState
  type: EVENT
  start:
    kind: DEFAULT
  eventsActions:
  - eventRefs:
    - execEvent
    actions: []
    eventDataFilter:
      dataOutputPath: "$.execEvent"
  transition:
    nextState: NextEventState
    produceEvent:
      eventRef: nextEvent
      data:
        type: next
        provider: exec-handler
        buildID: "$.execEvent.data.buildID"
        workerID: "$.execEvent.data.workerID"
        cause:
          event: "$.execEvent"
- name: NextEventState
  type: EVENT
  eventsActions:
  - eventRefs:
    - nextEvent
    eventDataFilter:
      dataOutputPath: "$.nextEvent"
    actions:
    - name: consoleLogAction
      functionRef:
        refName: consoleLogFunction
        parameters:
          log: fired $.nextEvent.data.type caused by $.nextEvent.data.cause.event
  end:
    kind: DEFAULT
```

</td>
</tr>
</table>

