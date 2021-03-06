Lastly, to fully test the app, it must be able to connect to a server. For
this use case, Tekton supports the Kubernetes sidecar pattern and allows
developers to run another app, such as a mock server, alongside each step of
the task, which you will use in this section to run the tests.

`tekton-examples/tasks/src/server` includes a Python server-side app that
listens at port 8080 and accepts connections from the client app. To run this
app as a sidecar in your task, edit `tekton-examples/tasks/src/tekton-katacoda/tasks/taskTemplate.yaml`
for the last time:

```yaml
specs:
  inputs:
  ...
  steps:
  - name: ...
    image: ...
    command:
    ...
    args:
    ...
    volumeMounts:
    ...
  # Ask the task to use the secret created earlier as a volume
  volumes:
  ...
  sidecars:
    # The name of the sidecar
  - name: example-server
    # The builder image to use
    image: python
    # The command to run with the builder image
    command:
    - /bin/bash
    - -c
    # The arguments to use with the command
    # Here you will clone the source code, install all the 
    # dependencies the app uses, and run the app
    args:
    - git clone https://github.com/michaelawyu/tekton-examples.git && cd ./tekton-examples/tasks/src/server && pip install -r requirements.txt && python main.py
```

With this setup, Tekton (Kubernetes) will run the step in the task and the
sidecar in the same pod, and they may communicate as if they are running side
by side on a local machine. The tests have been configured to use
`localhost:8080` as the address of the server.

Note that sidecars are started **before the task itself**; Tekton will stop
sidecars automatically after **all the steps in the task** have been completed.
