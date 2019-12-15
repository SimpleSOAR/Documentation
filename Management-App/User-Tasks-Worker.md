# User Tasks Worker

The User Tasks Worker is a special type of worker that enables a Service Task in the BPMN to act as a Manual Task / User Task / Human Task.

Typical use cases for User Tasks are:

1. Tickets
1. Manual actions / tasks
1. Backup for when a Automated script fails, and you want to task a analyst to manually perform the action
1. Creating a more restrictive contract for completion of a task

## How it Works

1. Worker Long Polls the Orchestrator for Tasks with type `user-task`
1. Orchestrator creates a task instance with type `user-task`
1. Worker receives/exclusive-locks the task
1. Worker converts the Orchestrator Job into a UserTaskEntity which is persisted into the User Task Database.
1. Someone/Something works on the User Task and at some point they use the `Submit Task` endpoint to complete the task.


For rendering the form in a browser, the flow is as follows:

1. Get Task
1. Using Task ID, get Task's Form, this will return the Form Schema that can be use to Render the form using the Form Renderer.
1. Render Form using the Form Schema
1. Submit Form and direct submission to Submit Form endpoint
1. If successful, the endpoint will return http status code of 200
1. If unsuccessful, the endpoint will return error details that can be passed back into the form renderer allowing the renderer to indicate which specific fields have errors.


## Form Renderer

The Form Renderer is based on the Form.io form render: [formio.js](https://github.com/formio/formio.js/).

formio.js provides several wrappers such as React and Angular.

For custom builds, see the [formio.js documentation](https://github.com/formio/formio.js/wiki/Form-Renderer) for customization capabilities and how to route submissions from the renderer to the `/submit` endpoint of the User Tasks Worker.

## Form Builder

To build Form Schemas you need to use the formio.js Form Builder

[Example form builder](https://formio.github.io/formio.js/app/builder)


### Customizing the Form Builder

You can customize the form builder, allowing modifications to look and feel, adding components or removing components you wish not to support.  See the [examples documentation](https://formio.github.io/formio.js/app/examples/custombuilder.html) of the form builder.

### Translation Support

Translation support is provided.

`Example goes here...`

### Saving Form Schemas

Once you have created a Form Schema, you need to save these schemas for use with User Tasks.

User Task Workers provide a dual role of Orchestrator Worker and provide HTTP endpoints for interacting with Forms.

#### Form Endpoints

##### Save Form

POST `localhost:8080/forms/`

```json
{
  "name": "Generic Approval Form",
  "description": "A Custom generic form that will be used for most approve/deny tasks",
  "formKey": "generic-approval-form"
}
```

Returns a JSON object of Form Entity

```json
{...}
```


##### Get Form

GET `localhost:8080/forms?formKey=generic-approval-form`
GET `localhost:8080/forms?formId=7fb30ba0-c610-4c35-87d8-17555a13c89e`

Returns a JSON object of Form Entity

```json
{...}
```


##### Save Form Schema

Save Form Schema.  Multiple Schemas can be saved for a single form.  Each Schema is versioned, ensuring you have full history of all changes.

POST `localhost:8080/forms/{formId}/schemas`

Body:

```json
{
  "schema": {
    ...form schema from Form Builder goes here...
  }
}
```

Returns a JSON object of Form Schema Entity

```json
{...}
```


##### Get Form Schemas

GET `localhost:8080/forms/{formId}/schemas`

Returns a list JSON object of Form Schemas Entity

```json
{...}
```


## User Task Endpoints

The following are the current target HTTP endpoints exposed by User Task Workers.

### Get All Tasks

Get all tasks in a Pageable format.  Tasks are filtered by current user's permissions related to user tasks.

GET `localhost:8080/tasks/{taskId}/`

Returns a JSON list of User Task Entity objects

```json
[...]

example goes here

```

### Get Task by ID

Get a special User Task based on Task ID.  Ability to view the task is based on the current user's permissions related to user tasks.

GET `localhost:8080/tasks/`

Returns a JSON object of the User Task Entity

```json
{...}

example goes here

```

### Claim Task

If the current user is a member of the Candidate Groups or Candidate Users, and is not currently assigned to the Task, and another user is not assigned to the task, then Claim/Assign the Task to the current User.  Otherwise return an error.

PATCH `localhost:8080/tasks/{taskID}/claim`

Returns a JSON object of the User Task Entity

```json
{...}
example goes here

```

### Un-Claim Task

If the current user is Assigned to the Task, UnClaim/UnAssign the Task from that user.  Otherwise returns an error.

PATCH `localhost:8080/tasks/{taskId}/unclaim`

Returns a JSON object of the User Task Entity

```json
{...}
example goes here

```

### Assignee Task

Used by administrators and higher level users, Assign a user task to user.  
The user performing the assign task request may assign to themselves.
If the task is already assigned, the task will be unassigned, and then assigned to the desired user.

PATCH `localhost:8080/tasks/{taskId}/assign`

Returns a User Task Entity as JSON

```json
{...}

example goes here

```

### Get Task Form

Get the JSON Schema of the Form assigned to this User Task based on the User Task's `formKey` property.

GET `localhost:8080/tasks/{taskId}/form`

Returns JSON Schema object of User Task Form

```json
{...}
example goes here

```

### Complete Task

A administrative endpoint for directly completing a task without having to submit through the Form Validation Service.  This endpoint skips validation against the Form Schema.  The variables supplied in this endpoint are directly injected into the workflow instance without any modification

POST `localhost:8080/tasks/{taskId}/complete`

Body:

```json
{
  "myKey": "myValue"
}
```

Returns a User Task Entity as JSON

```json
{...}
example goes here

```

### Submit Task

Most commonly used endpoint for completing a User Task.  This endpoint takes a Form Submission JSON object which is validated by the Form Validation Service, and if valid, will complete the Orchestrator Job, and update the User Task DB with the completion data/state.

The body has two fields: `data` (Required), and `metadata` (Optional).  The Form renderer will generally take care of this for you.  You only need to direct the submission endpoint of the Form renderer to point to the `/submit` endpoint.

POST `localhost:8080/tasks/{taskId}/submit`

Body:

```json
{
  "data": {
      "field1": "some value"
  },
  "metadata": {
      "browser": "safari-windows10"
  }
}
```

Returns a User Task Entity as JSON

```json
{...}
example goes here

```



# Form Submissions and Orchestrator Variables

When a successful Form Submission through the `/submit` endpoint occurs, the content of the form submission is saved as a variable in workflow instance in the Orchestrator.