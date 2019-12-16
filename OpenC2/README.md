# OpenC2 Support

SimpleSOAR provides support for OpenC2 APIs using a HTTP Server that exposes the OpenC2 endpoint(s), and a Worker that provides the ability to capture OpenC2 HTTP requests, process them, and return a sync or async response (as per the spec).

The design for this support is to provide a generic interface for OpenC2 commands.

As OpenC2 matures and refines the specification, the underlying workers will generally remain the same, while the microservice HTTP server that exposes the OpenC2 Endpoints can be versioned and updated as needed.  In cases where the specification does not provide the needed flexibility or features, you can easily extend the HTTP server to expose any desired functionality.

## How it works

An HTTP server is provided that exposes OpenC2 endpoints.  This acts as a generic wrapper for the OpenC2 spec.  It does not provide any ability to action the requests, only to capture the requests and route them to a applicable worker.

Once a properly formatted request is sent into the http server, it is accepted and made available for a worker take the request/job and process it.

Each OpenC2 command/request creates a new workflow instance with a task.  Multiple configurations are possible depending on desired distribution, placement, and setup of the workers.

In the most basic setup, The OpenC2 Worker behaves the same as the Python Executor Worker.


## Advanced Configurations

1. All commands are fulfilled by a single task type such as type `openc2`
1. Each OpenC2 Action can be a Orchestrator task type (`scan`, `locate`, etc).
1. Each OpenC2 Action and Target can be paired together to be Orchestrator task types (`scan:artifact`, `locate:artifact`, etc).
1. Actuators can be paired into the task types to allow specific routing of tasks/jobs to location specific workers.  This is typically used in cases where workers would be installed on appliances, devices, and specific locations where a specific worker instance executes all commands/actions (such as a worker being installed on the same server as a firewall, and the worker executing a local command to update firewall configurations). 
1. Using the Message feature of the Orchestrator to distribute messages to multiple workflows that perform many different actions that are then aggregated and turned into a single response.
1. Multiple Orchestrators that report back to central Orchestrator.
1. Rate Limiting can be configured on the Worker level.
1. Workers can be running different versions of the executions depending on needs.
1. Multiple Targets can be handled by a single workflow.