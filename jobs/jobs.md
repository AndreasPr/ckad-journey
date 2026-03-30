# Jobs in Kubernetes

## What is a Job?

A **Job** in Kubernetes is used to run a **finite task (batch job)** that:
- Executes **once or multiple times**
- Ensures the task **completes successfully**

Unlike Deployments:
- Jobs are **not long-running**
- They **terminate after completion**


## Use Cases

- Batch processing  
- Data migration  
- Backup tasks  
- One-time scripts  


# Basic Job Example

```
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  template:
    spec:
      containers:
      - name: math-add
        image: ubuntu
        command: ["expr", "3", "+", "2"]
      restartPolicy: Never
```

### Create the Job
```kubectl create -f job-definition.yaml```

### Check Jobs
```kubectl get jobs```

### View logs
```kubectl logs {name-of-pod}```

Note: Logs come from the Pod created by the Job

### Delete Job
```kubectl delete job {name-of-job}```

### Describe Job
```kubectl describe jpb {name-of-job}```

This deletes:
1. The Job
2. All Pods created by it



## Multiple Executions (Completions)
```
apiVersion: batch/v1
kind: Job
metadata:
    name: math-add-job
spec:
    completions: 3
    template:
        spec:
            containers:
            - name: math-add
              image: ubuntu
              command: ["expr", "3", "+", "2"]
            restartPolicy: Never
```

Meaning:
* Job must run successfully 3 times
* Kubernetes creates Pods until 3 succeed


## What happens if Pods fail?
Kubernetes will:

* Automatically retry failed Pods
* Continue until:
    * Required completions are reached
    * OR failure limit is exceeded

## Failure Control (Important)
backoffLimit: 4

- Default = 6 retries
- After limit exceeded -> Job marked Failed



### Parallelism
```
apiVersion: batch/v1
kind: Job
metadata:
    name: random-error-job
spec:
    completions: 3
    parallelism: 3
    template:
        spec:
            containers:
            - name: random-error
              image: mycloud/random-error
            restartPolicy: Never
```

Meaning:
* Run 3 Pods at the same time
* Complete all 3 executions in parallel

## Behavior
| Field       | Meaning                            |
| ----------- | ---------------------------------- |
| completions | Total successful runs required     |
| parallelism | How many Pods run at the same time |


## Scenarios
### Case 1:
```completions: 3```

```parallelism: 1```

* Runs one-by-one sequentially

### Case 2:
```completions: 3```

```parallelism: 3```

* Runs all 3 at the same time

### Case 3:
```completions: 6```

```parallelism: 2```

Runs:
* 2 Pods at a time
* Until 6 succeed

## Restart Policy
```restartPolicy: Never```

Options:
* Never -> create new Pod if failure
* OnFailure -> restart container inside same Pod

## Key Takeaways
* Job = run-to-completion workload
* Ensures task finishes successfully
* Retries failed Pods automatically
* Supports:
    * Multiple completions
    * Parallel execution
* Not for long-running services (use Deployment instead)

## Pro Tip

Use Jobs for:
* Scripts
* Data pipelines
* One-time tasks

Use Deployments for:
* APIs
* Web apps
* Long-running services



# CronJobs in Kubernetes


## What is a CronJob?

A **CronJob** is a Kubernetes resource used to **run Jobs on a schedule**, similar to the Linux `cron` utility.

It creates **Job objects at specified times**, and those Jobs then create Pods to execute tasks.


## Use Cases

- Scheduled backups  
- Report generation  
- Data cleanup  
- Periodic data processing  


# How It Works

Flow: CronJob -> Job -> Pod
- **CronJob** -> defines schedule  
- **Job** -> defines execution logic  
- **Pod** -> runs the container  


For example, we have the cron-job-definition.yaml:
```
apiVersion: batch/v1
kind: CronJob
metadata:
    name: reporting-cron-job
spec:
    schedule: "*/1 * * * *"
    jobTemplate:
        spec:
            completions: 3
            parallelism: 3
            template:
                spec:
                    containers:
                    - name: reporting-tool
                      image: reporting-tool
                    restartPolicy: Never
```

## Why 3 spec Sections?

Each spec belongs to a different resource layer:

1. CronJob spec
Defines schedule and job template
2. Job spec
Defines completions and parallelism
3. Pod spec
Defines containers and runtime behavior


## Cron Schedule Format
`* * * * *`


* 1st * is `Minute (0 - 59)`
* 2nd * is `Hour (0 - 23)`
* 3rd * is `Day of month (1 - 31)`
* 4th * is `Month (1 - 12)`
* 5th * is `Day of week (0 - 6)`


### Example
```schedule: "*/1 * * * *"```

* Runs every 1 minute



### Create CronJob
```kubectl create -f cron-job-definition.yaml```

### View CronJobs
```kubectl get cronjob```

### View Logs
```kubectl logs <pod-name>```







## Important CronJob Options
#### 1. `startingDeadlineSeconds`

Time limit to start a missed job

```startingDeadlineSeconds: 60```

#### 2. `concurrencyPolicy`

Controls how concurrent Jobs behave
```concurrencyPolicy: Forbid```

Options:
* `Allow` (default) -> run jobs in parallel
* `Forbid` -> skip if previous job still running
* `Replace` -> stop old job and start new one

#### 3. `successfulJobsHistoryLimit`

```successfulJobsHistoryLimit: 3```

Keeps last 3 successful jobs

#### 4. `failedJobsHistoryLimit`

```failedJobsHistoryLimit: 1```

Keeps last failed jobs

## Common Pitfalls
* Jobs may overlap if execution time > schedule
* Too many jobs -> resource exhaustion
* Logs are ephemeral unless stored externally

## Key Takeaways
* CronJob = scheduled Job execution
* Uses cron syntax (* * * * *)
* Creates Jobs -> which create Pods
* Supports:
    * Parallelism
    * Retry logic
    * History limits


## Real-World Example

**Nightly backup:**

`schedule: "0 2 * * *"`

Runs every day at 2 AM

## Pro Tip

Always configure:
* `concurrencyPolicy`
* History limits

Prevents resource leaks and overlapping executions