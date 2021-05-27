# Scrum methodology model workflow


## Description

The workflow attempts to model the lifecycle of an application's tasks during the whole development process, in the context of the scrum/agile methodology and from the developer's point of view.

On our imagined scrum board, a task is composed of 3 items:
- an id
- an estimated number of story points 
- a status.

There are 7 possible consecutive states for a task, as it follows:
1. **TO_DO**: the initial state of a task, right when the application starts its lifecycle
2. **IN_PROGRESS**: the task is being processed by the developer
3. **PEER_REVIEW**: the task's associated "code" is being reviewed by the team
4. **PRODUCT_REVIEW**: the task's impact in the application is being reviewed by the product owner
5. **DONE**: the task is ready to have automated tests ran on it followed by it going into production
6. **TESTING**: the task has associated tests running in the cloud
7. **DEPLOYER**: the task is now present in the production environment

The established possible values for a task's story points must respect Fibonacci's sequence and can go up to 13.

## Data types

Therefore, we have defined a task using the following declarations:
```
colset TASK_ID = STRING;
colset TASK_STATUS = with TO_DO | IN_PROGRESS | PEER_REVIEW | PRODUCT_REVIEW | DONE | TESTING | DEPLOYED;
colset TASK_POINTS = INT;
colset TASK = product TASK_ID*TASK_POINTS_TASK_STATUS;
```

All tasks are stored in a 'database' in the form of a list. At the beginning of the application's lifecycle, all the tasks are available in the product backlog. 

```
colset LTASK = list TASK;
colset PRODUCT_BACKLOG = LTASK;
```

After all the tasks are entered in the *All available tasks* place of type **PRODUCT_BACKLOG** as an initial marking, all of them are to be estimated and a subset of them are to be chosen for the sprint backlog in the *Sprint tasks* place of type **SPRINT_BACKLOG**.

```
colset SPRINT_BACKLOG = LTASK;
```

For estimating a task, we have created a function that randomly assigns it a number of story points, according to the Fibonacci sequence.

```
    fun fibonacci(0) = 1
        | fibonacci(1) = 1
        | fibonacci(n) = fibonacci(n - 1) + fibonacci(n - 2)

    // The bounds for a story point are [fibonacci(1), fibonacci(6)] = [1, 13]
    fun assignStoryPoints(t: TASK) = (#1 t, fibonacci(discrete(1,6)), #3 t) 
```

After estimating all the tasks, the tasks are chosen depending on the maximum sprint story points defined in the initial marking of the *Max sprint story points* place of type **INT**.

```
    fun selectTasksFromProductBacklog(tasks: LTASK, points: INT) = 
        if tasks = [] then [] else
            if #2 (hd tasks) <= points then (hd tasks)::selectTasksFromProductBacklog(tl tasks, points - #2 (hd tasks))
            else selectTasksFromProductBacklog(tl tasks, points);
```

After the sprint starts, the developer will randomly pick a task which will go through the following states, according to the progress done:

```
    fun selectTaskFromSprintBacklog(lt: TASK) = List.nth(lt, (discrete(0, (length lt) - 1)));
```

<dl>
<dt><b>TO_DO</b></dt>
<dd>Initial state of the task.</dd>

<dt><b>IN_PROGRESS</b></dt>
<dd>A developer has picked this task and is working on it.</dd>

<dt><b>PEER_REVIEW</b></dt>
<dd>After the task is done, it is reviewed by the developer's peers. Upon a successful review by the peers, the task will go into the <b>PRODUCT_REVIEW</b> state, otherwise it will go to the previous state <b>IN_PROGRESS</b>.</dd>

<dt><b>PRODUCT_REVIEW</b></dt>
<dd>In this state the product owner will check the task. Similarly, the tasks either proceeds into the next state <b>DONE</b> or is rejected and goes back to <b>IN_PROGRESS</b>.</dd>

<dt><b>DONE</b></dt>
<dd>The task has passed all reviews and is ready to have automated tests ran on it.</dd>
</dl>

Only after all the tasks associated with this sprint get, one by one, to the done state, they can collectively be tested. The tasks which fail the tests will be moved back to the product backlog to be redeveloped in the following sprints. Otherwise, they will be stored in the *DEPLOY task* place of type **LTASK** where they will be queued for deployment. The sprint is considered to be finished once all the picked tasks get in the *All finished tasks* place of type **LTASK**. If there are remaining tasks in the product backlog, the process repeats until there are no more. Once all the available tasks have been processed and succesfully deployed, we can consider the product lifecycle to have reached an end, all of them will be found in the *Finished product lifecycle tasks* place of type **LTASK**.