# Python Executor Worker

The Python Executor Worker is a microservice that enables execution of python scripts.

Scripts are defined in the Worker's application.yml file.

One or more instances of Python Worker may be connected to the Orchestrator.

The Worker executes the python script without any restrictions or limits.

The Worker executes the python script under the OS user that the Worker is running under.

The Worker has configurations for:

1. Defining the specific location of Python
1. The folder where script executions should take place
1. The folder where scripts will be stored/found
1. The max execution time of a script
1. The list of scripts that the worker is allowed to execute.
1. The max number of parallel scripts being executed by the specific worker.
1. The max long poll duration with the Orchestrator

## How it Workers

1. Worker long polls the Orchestrator for tasks with type `script-python`
1. When `script-python` tasks are found, they are retrieved by the Worker.
1. The Worker gets the specific script to execute based on the `script` header in the workflow task.
1. The Worker generates a json file in a temporary folder with the variables from the Workflow instance.
1. The Worker starts the python script and passes the location of the json file as a argument, and a second argument of the expected json file location for variables that should be sent to the Orchestrator upon successful completion of the script.
1. The script runs and can execute whatever it wishes.
1. If the script wishes to return new or updated variables to the workflow instance in the Orchestrator, the script can write to a json file that was provided in the arguments of the script execution.
1. If an error occurs and the script does not return a exit code of 0, then the error is captured and returned to the Orchestrator as a failed job/failed script execution.  Depending on configurations, the Orchestrator may direct for the task to be attempted again, or it may create a Job Incident for a analyst to investigate.