# Examples - Argo Workflows

[Argo Workflows](https://github.com/argoproj/argo) is an open source container-native workflow engine for 
orchestrating parallel jobs on Kubernetes. 
The Argo markup is YAML based and workflows are implemented as a Kubernetes CRD (Custom Resource Definition).
Argo is also a [CNCF](https://www.cncf.io/) Incubating project. 

Argo has a number of [examples](https://github.com/argoproj/argo/tree/master/examples) which display 
different Argo template examples.

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
- [Directed Acyclic Graph (DAG)](#Directed Acyclic Graph)
- [Scripts and Results](#Scripts-And-Results)
- [Loops](#Loops)
- [Conditionals](#Conditionals)


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
  type: OPERATION
  start:
    kind: DEFAULT
  actions:
  - functionRef:
      refName: whalesayimage
      parameters:
        message: "$.message"
  end:
    kind: DEFAULT
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
  type: OPERATION
  start:
    kind: DEFAULT
  actions:
  - functionRef:
      refName: whalesayimage
      parameters:
        message: hello1
  transition:
    nextState: parallelhello
- name: parallelhello
  type: PARALLEL
  completionType: AND
  branches:
  - name: hello2a-branch
    states:
    - name: hello2a
      type: OPERATION
      start:
        kind: DEFAULT
      actions:
      - functionRef:
          refName: whalesayimage
          parameters:
            message: hello2a
      end:
        kind: DEFAULT
  - name: hello2b-branch
    states:
    - name: hello2b
      type: OPERATION
      start:
        kind: DEFAULT
      actions:
      - functionRef:
          refName: whalesayimage
          parameters:
            message: hello2b
      end:
        kind: DEFAULT
  end:
    kind: DEFAULT

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
  type: OPERATION
  start:
    kind: DEFAULT
  actions:
  - functionRef:
      refName: echo
      parameters:
        message: A
  transition:
    nextState: parallelecho
- name: parallelecho
  type: PARALLEL
  completionType: AND
  branches:
  - name: B-branch
    states:
    - name: B
      type: OPERATION
      start:
        kind: DEFAULT
      actions:
      - functionRef:
          refName: echo
          parameters:
            message: B
      end:
        kind: DEFAULT
  - name: C-branch
    states:
    - name: C
      type: OPERATION
      start:
        kind: DEFAULT
      actions:
      - functionRef:
          refName: echo
          parameters:
            message: C
      end:
        kind: DEFAULT
  transition:
    nextState: D
- name: D
  type: OPERATION
  start:
    kind: DEFAULT
  actions:
  - functionRef:
      refName: echo
      parameters:
        message: D
  end:
    kind: DEFAULT
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
  type: OPERATION
  start:
    kind: DEFAULT
  actions:
  - functionRef:
      refName: gen-random-int-bash
    actionDataFilter:
      dataResultsPath: "$.results"
  transition:
    nextState: print-message
- name: print-message
  type: OPERATION
  actions:
  - functionRef:
      refName: printmessagefunc
      parameters:
        message: "$.results"
  end:
    kind: DEFAULT
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
  type: INJECT
  start:
    kind: DEFAULT
  data:
    greetings:
    - hello world
    - goodbye world
  transition:
    nextState: printgreetings
- name: printgreetings
  type: FOREACH
  inputCollection: "$.greetings"
  inputParameter: "$.greeting"
  states:
  - name: foreach-print
    type: OPERATION
    start:
      kind: DEFAULT
    actions:
    - name: print-message
      functionRef:
        refName: whalesay
        parameters:
          message: "$.greeting"
    end:
      kind: DEFAULT
  end:
    kind: DEFAULT
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
  type: OPERATION
  start:
    kind: DEFAULT
  actions:
  - functionRef:
      refName: flip-coin-function
    actionDataFilter:
      dataResultsPath: "$.flip.result"
  transition:
    nextState: show-flip-results
- name: show-flip-results
  type: SWITCH
  conditions:
  - path: "$.flip.result"
    value: heads
    operator: Equals
    transition:
      nextState: show-results-heads
  - path: "$.flip.result"
    value: tails
    operator: Equals
    transition:
      nextState: show-results-tails
- name: show-results-heads
  type: OPERATION
  actions:
  - functionRef:
      refName: echo
    actionDataFilter:
      dataResultsPath: it was heads
  end:
    kind: DEFAULT
- name: show-results-tails
  type: OPERATION
  actions:
  - functionRef:
      refName: echo
    actionDataFilter:
      dataResultsPath: it was tails
  end:
    kind: DEFAULT
```

</td>
</tr>
</table>
