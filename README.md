# mlflow my template
My test template for mlflow framework for learning how it works

## links
* [Mlflow](https://www.mlflow.org/) origin site and also [documentation](https://www.mlflow.org/docs/latest/index.html) 
* Origin [mlflow repo](https://github.com/mlflow/mlflow) with some [examples](https://github.com/mlflow/mlflow/tree/master/examples) inside

## Quick start

Getted from origin mlflow [quickstart](https://www.mlflow.org/docs/latest/quickstart.html)

I collect mostly interest parts for me here

### Install mlflow and some of the most common MLflow extra dependencies


    pip3 install mlflow
    pip3 install mlflow[extras]

TODO: config mlflow


## Tracking API

The [MLflow Tracking API](https://www.mlflow.org/docs/latest/tracking.html) lets you log metrics and artifacts (files) from your data science code and see a history of your runs. You can try it out by writing a simple Python script as follows. You can also find this [code](./tracking_api.py) in my repo.

    import os
    from random import random, randint
    from mlflow import log_metric, log_param, log_artifacts

    if __name__ == "__main__":
        # Log a parameter (key-value pair)
        log_param("param1", randint(0, 100))

        # Log a metric; metrics can be updated throughout the run
        log_metric("foo", random())
        log_metric("foo", random() + 1)
        log_metric("foo", random() + 2)

        # Log an artifact (output file)
        log_artifacts("README.md")

After every running of this code `python3 tracking_api.py` mlflow create folder (local `./` by default) `./mlruns/[experements_id(default==0)]/[run_id]/` with all logged data of that run inside:

![mlruns_structure][1]


## Viewing the Tracking UI

For overviewing collected data, you need to start mlflow ui in folder with `mlruns/` and view it at http://localhost:5000. It look like tensorflow run with `log/` folder.

in terminal:

    mlflow ui







[1]: ./imgs/mlruns_structure.png "mlruns folder structure"