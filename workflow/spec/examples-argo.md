# Examples - Argo Workflows

[Argo Workflows](https://github.com/argoproj/argo) is an open source container-native workflow engine for 
orchestrating parallel jobs on Kubernetes. 
The Argo markup is YAML based and workflows are implemented as a Kubernetes CRD (Custom Resource Definition).
Argo is also a [CNCF](https://www.cncf.io/) Incubating project. 

Argo has a number of [examples](https://github.com/argoproj/argo/tree/master/examples) which display 
different Argo templates.

The purpose of this document is to show side-by-side the Argo markup and the equivalent markup of the 
Serverless Workflow Specification. This can hopefully help compare and contrast the two markups and 
give a better understanding of both.

## Preface 

Argo YAML is defined inside a Kubernetes CRD (Custom Resource Definition). The resource definition contains a "spec"
parameter which contains the entrypoint of the workflow and the template parameter which defines one or more 
workflow definitions. When comparing the examples below please note that the Serverless Workflow specification YAML
pertains to the content of the "spec" parameter. Other parameters in the Kubernetes CRD are not considered and can
remain the same. Note that the Serverless Workflow YAML could also be embedded inside the CRD.

For the sake of comparing the two models, we use the YAML representation as well for the 
Serverless Workflow specification part.

## Table of Contents

- [Hello World with Parameters](#Hello-World-With-Parameters)
- [Multi Step Workflow](#Multi-Step-Workflow)
- [Directed Acyclic Graph](#Directed-Acyclic-Graph)
- [Scripts and Results](#Scripts-And-Results)
- [Loops](#Loops)
- [Conditionals](#Conditionals)
- [Retrying Failed Steps](#Retrying-Failed-Steps)
- [Recursion](#Recursion)
- [Exit Handlers](#Exit-Handlers)

### Hello World With Parameters

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#parameters)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-parameters-
spec:
  entrypoint: whalesay
  arguments:
    parameters:
    - name: message
      value: hello world

  templates:
  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```

</td>
<td valign="top">

```yaml
id: hello-world-parameters
name: Hello World with parameters
version: '1.0'
functions:
- name: whalesayimage
  resource: docker/whalesay
  type: container
  metadata:
    command: cowsay
states:
- name: whalesay
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: whalesayimage
      parameters:
        message: "$.message"
  end:
    kind: default
```

</td>
</tr>
</table>

### Multi Step Workflow

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#steps)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: steps-
spec:
  entrypoint: hello-hello-hello

  # This spec contains two templates: hello-hello-hello and whalesay
  templates:
  - name: hello-hello-hello
    # Instead of just running a container
    # This template has a sequence of steps
    steps:
    - - name: hello1            # hello1 is run before the following steps
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello1"
    - - name: hello2a           # double dash => run after previous step
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello2a"
      - name: hello2b           # single dash => run in parallel with previous step
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello2b"

  # This is the same template as from the previous example
  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```

</td>
<td valign="top">

```yaml
id: hello-hello-hello
name: Multi Step Hello
version: '1.0'
functions:
- name: whalesayimage
  resource: docker/whalesay
  type: container
  metadata:
    command: cowsay
states:
- name: hello1
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: whalesayimage
      parameters:
        message: hello1
  transition:
    nextState: parallelhello
- name: parallelhello
  type: parallel
  completionType: and
  branches:
  - name: hello2a-branch
    states:
    - name: hello2a
      type: operation
      start:
        kind: default
      actions:
      - functionRef:
          refName: whalesayimage
          parameters:
            message: hello2a
      end:
        kind: default
  - name: hello2b-branch
    states:
    - name: hello2b
      type: operation
      start:
        kind: default
      actions:
      - functionRef:
          refName: whalesayimage
          parameters:
            message: hello2b
      end:
        kind: default
  end:
    kind: default

```

</td>
</tr>
</table>

### Directed Acyclic Graph

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#dag)

*Note*: Even tho this example can be described (has a single 
starting task) using the specification, the spec does not currently support multiple
start events. Argo workflows that have multiple starting 
DAG tasks cannot be described using the specification at this time.

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-diamond-
spec:
  entrypoint: diamond
  templates:
  - name: echo
    inputs:
      parameters:
      - name: message
    container:
      image: alpine:3.7
      command: [echo, "{{inputs.parameters.message}}"]
  - name: diamond
    dag:
      tasks:
      - name: A
        template: echo
        arguments:
          parameters: [{name: message, value: A}]
      - name: B
        dependencies: [A]
        template: echo
        arguments:
          parameters: [{name: message, value: B}]
      - name: C
        dependencies: [A]
        template: echo
        arguments:
          parameters: [{name: message, value: C}]
      - name: D
        dependencies: [B, C]
        template: echo
        arguments:
          parameters: [{name: message, value: D}]
```

</td>
<td valign="top">

```yaml
id: dag-diamond-
name: DAG Diamond Example
version: '1.0'
functions:
- name: echo
  resource: alpine:3.7
  type: container
  metadata:
    command: '[echo, "{{inputs.parameters.message}}"]'
states:
- name: A
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: echo
      parameters:
        message: A
  transition:
    nextState: parallelecho
- name: parallelecho
  type: parallel
  completionType: and
  branches:
  - name: B-branch
    states:
    - name: B
      type: operation
      start:
        kind: default
      actions:
      - functionRef:
          refName: echo
          parameters:
            message: B
      end:
        kind: default
  - name: C-branch
    states:
    - name: C
      type: operation
      start:
        kind: default
      actions:
      - functionRef:
          refName: echo
          parameters:
            message: C
      end:
        kind: default
  transition:
    nextState: D
- name: D
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: echo
      parameters:
        message: D
  end:
    kind: default
```

</td>
</tr>
</table>

### Scripts And Results

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#scripts--results)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: scripts-bash-
spec:
  entrypoint: bash-script-example
  templates:
  - name: bash-script-example
    steps:
    - - name: generate
        template: gen-random-int-bash
    - - name: print
        template: print-message
        arguments:
          parameters:
          - name: message
            value: "{{steps.generate.outputs.result}}"  # The result of the here-script

  - name: gen-random-int-bash
    script:
      image: debian:9.4
      command: [bash]
      source: |                                         # Contents of the here-script
        cat /dev/urandom | od -N2 -An -i | awk -v f=1 -v r=100 '{printf "%i\n", f + r * $1 / 65536}'

  - name: gen-random-int-python
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        i = random.randint(1, 100)
        print(i)

  - name: gen-random-int-javascript
    script:
      image: node:9.1-alpine
      command: [node]
      source: |
        var rand = Math.floor(Math.random() * 100);
        console.log(rand);

  - name: print-message
    inputs:
      parameters:
      - name: message
    container:
      image: alpine:latest
      command: [sh, -c]
      args: ["echo result was: {{inputs.parameters.message}}"]
```

</td>
<td valign="top">

```yaml
id: scripts-bash-
name: Scripts and Results Example
version: '1.0'
functions:
- name: gen-random-int-bash
  resource: debian:9.4
  type: script
  metadata:
    command: bash
    source: |-
      cat /dev/urandom | od -N2 -An -i | awk -v f=1 -v r=100 '{printf "%i
      ", f + r * $1 / 65536}'
- name: gen-random-int-python
  resource: python:alpine3.6
  type: script
  metadata:
    command: python
    source: | 
      import random 
      i = random.randint(1, 100) 
      print(i)
- name: gen-random-int-javascript
  resource: node:9.1-alpine
  type: script
  metadata:
    command: node
    source: |
      var rand = Math.floor(Math.random() * 100); 
      console.log(rand);
- name: printmessagefunc
  resource: alpine:latest
  type: container
  metadata:
    command: sh, -c
    source: 'echo result was: {{inputs.parameters.message}}'
states:
- name: generate
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: gen-random-int-bash
    actionDataFilter:
      dataResultsPath: "$.results"
  transition:
    nextState: print-message
- name: print-message
  type: operation
  actions:
  - functionRef:
      refName: printmessagefunc
      parameters:
        message: "$.results"
  end:
    kind: default
```

</td>
</tr>
</table>

### Loops

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#loops)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: loops-
spec:
  entrypoint: loop-example
  templates:
  - name: loop-example
    steps:
    - - name: print-message
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "{{item}}"
        withItems:              # invoke whalesay once for each item in parallel
        - hello world           # item 1
        - goodbye world         # item 2

  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```

</td>
<td valign="top">

```yaml
id: loops-
name: Loop over data example
version: '1.0'
functions:
- name: whalesay
  resource: docker/whalesay:latest
  type: container
  metadata:
    command: cowsay
states:
- name: injectdata
  type: inject
  start:
    kind: default
  data:
    greetings:
    - hello world
    - goodbye world
  transition:
    nextState: printgreetings
- name: printgreetings
  type: foreach
  inputCollection: "$.greetings"
  inputParameter: "$.greeting"
  states:
  - name: foreach-print
    type: operation
    start:
      kind: default
    actions:
    - name: print-message
      functionRef:
        refName: whalesay
        parameters:
          message: "$.greeting"
    end:
      kind: default
  end:
    kind: default
```

</td>
</tr>
</table>

### Conditionals

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#conditionals)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: coinflip-
spec:
  entrypoint: coinflip
  templates:
  - name: coinflip
    steps:
    # flip a coin
    - - name: flip-coin
        template: flip-coin
    # evaluate the result in parallel
    - - name: heads
        template: heads                 # call heads template if "heads"
        when: "{{steps.flip-coin.outputs.result}} == heads"
      - name: tails
        template: tails                 # call tails template if "tails"
        when: "{{steps.flip-coin.outputs.result}} == tails"

  # Return heads or tails based on a random number
  - name: flip-coin
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        result = "heads" if random.randint(0,1) == 0 else "tails"
        print(result)

  - name: heads
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was heads\""]

  - name: tails
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was tails\""]
```

</td>
<td valign="top">

```yaml
id: coinflip-
name: Conditionals Example
version: '1.0'
functions:
- name: flip-coin-function
  resource: python:alpine3.6
  type: script
  metadata:
    command: python
    source: import random result = "heads" if random.randint(0,1) == 0 else "tails"
      print(result)
- name: echo
  resource: alpine:3.6
  type: container
  metadata:
    command: sh, -c
states:
- name: flip-coin
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: flip-coin-function
    actionDataFilter:
      dataResultsPath: "$.flip.result"
  transition:
    nextState: show-flip-results
- name: show-flip-results
  type: switch
  dataConditions:
  - path: "$.flip.result"
    value: heads
    operator: equals
    transition:
      nextState: show-results-heads
  - path: "$.flip.result"
    value: tails
    operator: equals
    transition:
      nextState: show-results-tails
- name: show-results-heads
  type: operation
  actions:
  - functionRef:
      refName: echo
    actionDataFilter:
      dataResultsPath: it was heads
  end:
    kind: default
- name: show-results-tails
  type: operation
  actions:
  - functionRef:
      refName: echo
    actionDataFilter:
      dataResultsPath: it was tails
  end:
    kind: default
```

</td>
</tr>
</table>


### Retrying Failed Steps

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#retrying-failed-or-errored-steps)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: retry-backoff-
spec:
  entrypoint: retry-backoff
  templates:
  - name: retry-backoff
    retryStrategy:
      limit: 10
      retryPolicy: "Always"
      backoff:
        duration: "1"      
        factor: 2
        maxDuration: "1m" 
    container:
      image: python:alpine3.6
      command: ["python", -c]
      # fail with a 66% probability
      args: ["import random; import sys; exit_code = random.choice([0, 1, 1]); sys.exit(exit_code)"]
```

</td>
<td valign="top">

```yaml
id: retry-backoff-
name: Retry Example
version: '1.0'
functions:
- name: fail-function
  resource: python:alpine3.6
  type: container
  metadata:
    command: python
states:
- name: retry-backoff
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: flip-coin-function
      parameters:
        args:
        - import random; import sys; exit_code = random.choice([0, 1, 1]); sys.exit(exit_code)
  retry:
  - expression:
      language: spel
      body: "$.exit_code == 1"
    maxAttempts: 10
    multiplier: PT2M
    interval: PT1M
  end:
    kind: default
```

</td>
</tr>
</table>


### Recursion

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#recursion)

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: coinflip-recursive-
spec:
  entrypoint: coinflip
  templates:
  - name: coinflip
    steps:
    # flip a coin
    - - name: flip-coin
        template: flip-coin
    # evaluate the result in parallel
    - - name: heads
        template: heads                 # call heads template if "heads"
        when: "{{steps.flip-coin.outputs.result}} == heads"
      - name: tails                     # keep flipping coins if "tails"
        template: coinflip
        when: "{{steps.flip-coin.outputs.result}} == tails"

  - name: flip-coin
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        result = "heads" if random.randint(0,1) == 0 else "tails"
        print(result)

  - name: heads
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was heads\""]
```

</td>
<td valign="top">

```yaml
id: coinflip-recursive-
name: Recursion Example
version: '1.0'
functions:
- name: heads-function
  resource: alpine:3.6
  type: container
  metadata:
    command: echo "it was heads"
- name: flip-coin-function
  resource: python:alpine3.6
  type: script
  metadata:
    command: python
    source: import random result = "heads" if random.randint(0,1) == 0 else "tail"  print(result)
states:
- name: flip-coin-state
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: flip-coin-function
    actionDataFilter:
      dataResultsPath: "$.steps.flip-coin.outputs.result"
  transition:
    nextState: flip-coin-check
- name: flip-coin-check
  type: switch
  dataConditions:
  - path: "$.steps.flip-coin.outputs.result"
    value: tails
    operator: equals
    transition:
      nextState: flip-coin-state
  - path: "$.steps.flip-coin.outputs.result"
    value: heads
    operator: equals
    transition:
      nextState: heads-state
- name: heads-state
  type: operation
  actions:
  - functionRef:
      refName: heads-function
      parameters:
        args: echo "it was heads"
  end:
    kind: default
```

</td>
</tr>
</table>


### Exit Handlers

[Argo Example](https://github.com/argoproj/argo/tree/master/examples#exit-handlers)

*Note*: With Serverless Workflow specification we can handle Argos "onExit" functionality
in couple of ways. One is the "onError" functionality to catch errors and transition to the 
"error" part of the workflow. Another is to send an event at the end of workflow execution 
which includes the workflow status. This event can then trigger execution other workflows
that can handle each status. For this example we use the "onError" definition.

<table>
<tr>
    <th>Argo</th>
    <th>Serverless Workflow</th>
</tr>
<tr>
<td valign="top">

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: exit-handlers-
spec:
  entrypoint: intentional-fail
  onExit: exit-handler           
  templates:
  # primary workflow template
  - name: intentional-fail
    container:
      image: alpine:latest
      command: [sh, -c]
      args: ["echo intentional failure; exit 1"]

  # Exit handler templates
  # After the completion of the entrypoint template, the status of the
  # workflow is made available in the global variable {{workflow.status}}.
  # {{workflow.status}} will be one of: Succeeded, Failed, Error
  - name: exit-handler
    steps:
    - - name: notify
        template: send-email
      - name: celebrate
        template: celebrate
        when: "{{workflow.status}} == Succeeded"
      - name: cry
        template: cry
        when: "{{workflow.status}} != Succeeded"
  - name: send-email
    container:
      image: alpine:latest
      command: [sh, -c]
      args: ["echo send e-mail: {{workflow.name}} {{workflow.status}}"]
  - name: celebrate
    container:
      image: alpine:latest
      command: [sh, -c]
      args: ["echo hooray!"]
  - name: cry
    container:
      image: alpine:latest
      command: [sh, -c]
      args: ["echo boohoo!"]
```

</td>
<td valign="top">

```yaml
id: exit-handlers-
name: Exit/Error Handling Example
version: '1.0'
functions:
- name: intentional-fail-function
  resource: alpine:latest
  type: container
  metadata:
    command: "[sh, -c]"
- name: send-email-function
  resource: alpine:latest
  type: script
  metadata:
    command: "[sh, -c]"
- name: celebrate-cry-function
  resource: alpine:latest
  type: script
  metadata:
    command: "[sh, -c]"
states:
- name: intentional-fail-state
  type: operation
  start:
    kind: default
  actions:
  - functionRef:
      refName: intentional-fail-function
      parameters:
        args: echo intentional failure; exit 1
  onError:
  - expression:
      language: spel
      body: "$.error != null"
    errorDataFilter:
      dataOutputPath: "$.exit-code"
  transition:
    nextState: send-email-state
- name: send-email-state
  type: operation
  actions:
  - functionRef:
      refName: send-email-function
      parameters:
        args: 'echo send e-mail: $.workflow.name $.exit-code'
  transition:
    nextState: emo-state
- name: emo-state
  type: switch
  dataConditions:
  - path: "$.exit-code"
    value: '1'
    operator: equals
    transition:
      nextState: celebrate-state
  - path: "$.exit-code"
    value: '1'
    operator: notequals
    transition:
      nextState: cry-state
- name: celebrate-state
  type: operation
  actions:
  - functionRef:
      refName: celebrate-cry-function
      parameters:
        args: echo hooray!
  end:
    kind: default
- name: cry-state
  type: operation
  actions:
  - functionRef:
      refName: celebrate-cry-function
      parameters:
        args: echo boohoo!
  end:
    kind: default
```

</td>
</tr>
</table>
