# Jobs

A Job creates Pods that run until successful termination(for instance, exit with 0). Jobs are useful for things you only want to do once, such as database migrations or batch jobs.

## The Job Object

The Job object is responsible for creating and managing Pods defined in a template in the job specification. These Pods generally run until successful completion. The Job
object coordinates running a number of Pods in parallel.

If the Pod fails before a successful termination, the job controller will create a new Pod based on the Pod template in the job specification. Given that Pods have to be scheduled, there is a chance that your job will not execute if the scheduler does not find the required resources. Also, due to the nature of distributed systems, there is a small chance that duplicate Pods will be created for a specific task during certain failure scenarios.

## Job Patterns

Jobs are designed to manage batch-like workloads where work items are processed by one or more Pods. By default, each job runs a single Pod once until successful ter‐
mination. This job pattern is defined by two primary attributes of a job: the number of job completions and the number of Pods to run in parallel.

### One Shot

One-shot jobs provide a way to run a single Pod once until successful termination. This is done using a Pod template defined in the job configuration. Once a job is up and running,the Pod backing the job must be monitored for successful termination. A job can fail for any number of reasons.

There are multiple ways to create a one-shot job in Kubernetes. The easiest is to use the kubectl command-line tool:

```
$ kubectl run -i oneshot \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--restart=OnFailure \
--command /kuard \
-- --keygen-enable \
--keygen-exit-on-complete \
--keygen-num-to-gen 10
```
There are some things to note here:

- The `-i` option to kubectl indicates that this is an interactive command. kubectl will wait until the job is running and then show the log output from the first (and in this case only) Pod in the job.
- All of the options after `--` are command-line arguments to the container image. These instruct our test server (kuard) to generate ten 4,096-bit SSH keys and then exit.

Note that this job won’t show up in kubectl get jobs
unless you pass the `-a` flag. Without this flag, kubectl hides completed jobs. Delete the job before continuing:

```
$ kubectl delete pods oneshot
```
The other option for creating a one-shot job is using a configuration file, as shown in the [example](./job-oneshot.yaml)

You can view the results of the job by looking at the logs of the Pod that was created:
```
$ kubectl logs oneshot-4kfdt
```
Because jobs have a finite beginning and ending, users often create many of them. This makes picking unique labels more difficult and more critical. For this reason, the Job object will automatically pick a unique label and use it to identify the Pods it creates. In advanced
scenarios (such as swapping out a running job without killing the Pods it is managing), users can choose to turn off this automatic behavior and manually specify labels and selectors.

We suggest you use `restartPolicy: OnFailure`, so failed Pods are rerun in place.

### Parallelism

Our goal is to generate 100 keys by having 10 runs of
kuard, with each run generating 10 keys. But we don’t want to swamp our cluster, so we’ll limit ourselves to only five Pods at a time.

This translates to setting completions to 10 and parallelism to 5. The config is
shown in [Example](./job-parallel.yaml).

### Work Queues

A common use case for jobs is to process work from a work queue. 

#### Starting a work queue

We start by launching a centralized work queue service. kuard has a simple memory-based work queue system built in. We will start an instance of kuard to act as a coordinator for all the work.

### CronJobs

Sometimes you want to schedule a job to be run at a certain interval. To achieve this, you can declare a CronJob in Kubernetes, which is responsible for creating a new Job object at a particular interval.