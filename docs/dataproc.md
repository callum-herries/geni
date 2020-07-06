# Geni on Dataproc

## Dataproc Setup

See the following guide to [setup dataproc on GCP](https://cloud.google.com/solutions/monte-carlo-methods-with-hadoop-spark) an to [create a dataproc cluster](https://cloud.google.com/dataproc/docs/guides/create-cluster#creating_a_cloud_dataproc_cluster). We will be using [Google Cloud SDK](https://cloud.google.com/sdk/install) with the `gcloud` CLI commands.

For this example, use the `preview` image version so that the cluster runs Spark 3. For instance, the following `gcloud` command creates a small dataproc cluster called `geni-cluster`:

```bash
gcloud dataproc clusters create geni-cluster \
    --region=asia-southeast1 \
    --master-machine-type n1-standard-1 \
    --master-boot-disk-size 30 \
    --num-workers 2 \
    --worker-machine-type n1-standard-1 \
    --worker-boot-disk-size 30 \
    --image-version=preview
```

Then access the primary node using:

```bash
gce geni-cluster-m ssh
```

## Running Geni on Yarn

Java should already be installed on the primary node. Install Leiningen using:

```bash
wget https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein && \
    sudo mv lein /usr/bin/ && \
    chmod a+x /usr/bin/lein && \
    lein
```

Then, create a templated Geni app and step into the app directory::

```bash
lein new geni app +dataproc && cd app
```

Create the uberjar and run it using `spark-submit`. This will spawn a Spark session running on YARN:

```bash
lein uberjar && \
    spark-submit --class app.core target/uberjar/app-0.0.1-SNAPSHOT-standalone.jar
```

By default, the templated main function:

1. prints the Spark configuration;
2. runs a Spark ML example;
3. starts an nREPL server on port 65204; and
4. steps into a [REPL(-y)](https://github.com/trptcolin/reply).

Verify that `spark.master` is set to `"yarn"`. To submit a standalone application, we can simply jump back to edit `core.clj`. Remove the `launch-repl` function to prevent stepping into the REPL. Note that it is necessary to run `lein uberjar` every time the app source code is changed.

## Cleaning Up

Once finished with the exercise, the easiest way to clean up is to simply delete the GCP project.

Alternatively, delete the cluster using:

```bash
gcloud dataproc clusters delete geni-cluster --region=asia-southeast1
```

There may be dangling storage buckets that have to be deleted separately.