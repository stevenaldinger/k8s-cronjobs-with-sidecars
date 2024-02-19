# Kubernetes CronJobs with Sidecars

This repo demonstrates multiple strategies for handling Kubernetes CronJobs that require sidecars. For a more detailed explanation, please refer to the [blog](https://medium.com/teamsnap-engineering/properly-running-kubernetes-jobs-with-sidecars-ddc04685d0dc) it's meant to complement.

## Repo Structure

```
.
├── README.md
└── manifests
    ├── 00.haproxy.configmap.yaml
    ├── cronjob.naive.failure.yaml
    ├── cronjob.naive.success.yaml
    ├── cronjob.pkill-nonroot.failure.yaml
    ├── cronjob.pkill-nonroot.success.yaml
    ├── cronjob.pkill.failure-bad-exit.yaml
    ├── cronjob.pkill.failure.yaml
    ├── cronjob.pkill.success.yaml
    ├── cronjob.shared-volume.failure.yaml
    ├── cronjob.shared-volume.success.yaml
    ├── cronjob.sidecar.failure.yaml
    └── cronjob.sidecar.success.yaml
```

Each `cronjob.*.yaml` in the [manifests](./manifests) directory declares a CronJob with a "primary" job container (that would represent your custom job code) alongside an [HAProxy](http://www.haproxy.org/) sidecar that serves as an arbitrary dependency of the "primary" job.

"Naive" strategy means that there's no attempt to shut down the sidecar after the primary container's job code runs.

"Pkill" strategy means that `pkill` is available inside the primary container and it will attempt to shut down the sidecar after it's job code runs.

"Shared volume" strategy means that a shell is available inside all containers as well as access to a common directory, and some scripting is used to watch the filesystem and let the sidecar shut itself down after the primary container's job code runs.

Any file ending in `.success.yaml` demonstrates the happy path where the job immediately succeeds.

Any file ending in `.failure.yaml` demonstrates the sad path where the job continuously fails, with retries configured.

## Note about Google's CloudSQL Proxy

Running database migration jobs in Google Cloud seems to be a common reason why people look for this solution.

A CloudSQL Proxy sidecar is commented out in the [manifests/cronjob.pkill.success.yaml](./manifests/cronjob.pkill.success.yaml) file and can be easily uncommented to prove the strategy works in your Kubernetes environment.

I've left it commented out because running the proxy without valid credentials causes it to crash with this error:

```
2022/11/02 19:46:33 the default Compute Engine service account is not configured with sufficient permissions to access the Cloud SQL API from this VM. Please create a new VM with Cloud SQL access (scope) enabled under "Identity and API access". Alternatively, create a new "service account key" and specify it using the -credential_file parameter
```
