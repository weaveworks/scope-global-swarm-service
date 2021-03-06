# Weave Scope as a Global Docker Swarm Service

Global Docker Swarm services have the advantage that they get automatically scheduled on new servers as you add them to a swarm.

The following hack allows you to schedule [Weave Scope](https://github.com/weaveworks/scope) on all the nodes in a swarm, without having to manually install the Scope probe on every node.

## 1. Get Weave Cloud Service Token

Go to [Weave Cloud](https://cloud.weave.works) and sign up for a Weave Cloud account.
Then you'll get a service token you can paste in below.

## 2. Start Weave Scope on your Swarm cluster

```
$ docker service create --name weave-scope --mode global \
    --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    lmarsden/scope-runner scope launch --service-token=<token>
```

Replacing `<token>` with your service token.

## 3. Deploy a sample application (optional)

Use the script [here](https://github.com/microservices-demo/microservices-demo/tree/master/deploy/swarmkit) to deploy a sample microservices app.
Then run `docker service inspect front-end` to find the port the app is running on, and load it in your browser.

You can run a [load test](https://github.com/microservices-demo/load-test) to apply to some load to the application and observe it "join up" in Weave Scope.

# How it works

Because Swarm services don't support `--privileged`, `--net=host` and `--pid=host` we had to workaround this by using a "jump" container: the service starts up a container which contains the docker client and has the docker socket bind-mounted so that it can run a non-background container privileged container in the usual way using the `scope launch` shell script.
That Dockerfile is [here](https://github.com/lukemarsden/weave-scope-as-global-swarm-service/blob/master/Dockerfile) and is at `lmarsden/scope-runner` on Docker Hub.

Now that we have this working, we would love to collaborate more with Docker to figure out how to enable this use-case natively in Swarm services without this hack. :-)

Relates issue: [#25303](https://github.com/docker/docker/issues/25303).

TODO: figure out exactly which capabilities and flags are required, and raise an issue on GitHub to capture the goal of getting Scope working as a Swarm service without this hack, linking to the above issue.

See also: [Notes document](https://docs.google.com/document/d/1Eie02teDKvxSrKIR9RHh340le-BgYExKzT6wQ2axil8/edit#)
