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

## MLflow Projects

MLflow allows you to package code and its dependencies as a project that can be run in a reproducible fashion on other data. Each project includes its [code](mlflow_project/train.py) and a [MLproject file](mlflow_project/MLproject) 

### MLproject file
Stored in project folder and defines its dependencies (for example, Python environment) as well as what commands can be run into the project and what arguments they take.
```
name: tutorial

conda_env: conda.yaml

entry_points:
  main:
    parameters:
      batch_size: { type: int, default: 64 }
      epochs: { type: int, default: 14 }
      gamma: { type: float, default: 0.7 }
      lr: { type: float, default: 1.0 }
      seed: { type: int, default: 1 }

    command: "python train.py --batch-size {batch_size} --epochs {epochs} --gamma {gamma} --lr {lr} --save-model --seed {seed}"
```

[MLproject file](mlflow_project/MLproject) contain link to [conda.yaml](mlflow_project/conda.yaml) with conda environment for that project.

```
name: tutorial
channels:
  - defaults
dependencies:
  - pip
  - pip:
    - mlflow
    - torch
    - torchvision
```


### Modification in code
Example of [pytorch mnist train code](mlflow_project/train.py) taken from here https://github.com/pytorch/examples/tree/master/mnist. Thanks autor for working simple example. For integrate that example to mlflow, you need to write simple additional logging code:

`mlflow.log_param(key, value)` for log starting script parameters

In that case we log:

    # MLFLOW log starting params
    mlflow.log_param("batch_size", args.batch_size)
    mlflow.log_param("epochs", args.epochs)
    mlflow.log_param("gamma", args.gamma)
    mlflow.log_param("lr", args.lr)
    mlflow.log_param("seed", args.seed)
    # MLFLOW log starting params

`mlflow.log_metric(key, value)` for log learning and eval metrics

In our case:

    def test(model, device, test_loader):
        model.eval()
        test_loss = 0
        correct = 0
        with torch.no_grad():
            for data, target in test_loader:
                data, target = data.to(device), target.to(device)
                output = model(data)
                test_loss += F.nll_loss(output, target, reduction='sum').item()  # sum up batch loss
                pred = output.argmax(dim=1, keepdim=True)  # get the index of the max log-probability
                correct += pred.eq(target.view_as(pred)).sum().item()

        test_loss /= len(test_loader.dataset)
    
        # MLFLOW LOG METRIC
        mlflow.log_metric("test Average loss", test_loss)
        mlflow.log_metric("test Accuracy", 100. * correct / len(test_loader.dataset))
        # MLFLOW LOG METRIC


And last one change is adding log of artifacts. That can be models, images and other files.
`mlflow.log_artifacts(local_dir)`

    # MLFLOW LOG ARTIFACT
    if not os.path.exists("outputs"):
        os.makedirs("outputs")
    torch.save(model.state_dict(), "outputs/mnist_cnn.pt")
    mlflow.log_artifacts("outputs")
    # MLFLOW LOG ARTIFACT



## MLflow run project

You can easily run existing projects with the mlflow run command, which runs a project from either a local directory or a GitHub URI. 
Tips: for running from git repo, you need to keep `MLproject` file, `conda.yaml` in root dir of repo on `master` branch


    mlflow run mlflow_project -P epochs=5

    mlflow run https://github.com/warenick/mlflow_project.git -P epochs=10

By default mlflow run installs all dependencies using [conda](https://conda.io/). To run a project without using conda, you can provide the --no-conda option to mlflow run. In this case, you must ensure that the necessary dependencies are already installed in your Python environment.

[1]: ./imgs/mlruns_structure.png "mlruns folder structure"