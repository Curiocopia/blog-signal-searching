# 📦 Signal Searching

This is a demonstration of using Discrete Fourier Transform (DFT) on weather data obtained from the [Open Meteo] API. Details are described in [Signal Searching].

## 🌟 Highlights
Kubernetes implementation includes:
- A Deployment that uses a pvc to retain both the source data, and the analysis results in sqlite3 db.
- A Deployment that uses the db service to access the analysis results as well as expose a web interface to users.
- A Job that obtains weather data from the Open Meteo API for a configurable set of cities around the world and populates the db.
- A Job that performs the DFT for the weather data and updates the db with analysis results.
- A Job that summarizes the analysis results per location into a sortable, global set of data for visual representation.

## ℹ️ Overview

Please refer to [Signal Searching] for the specific blog and the relevant repo(s). If you don't have time TL;DR:

Analysis of een a 365-day worth of weather data across the world provide interesting insights. Primarily, the influence of semi-diurnal (12-hour period) and diurnal (24-hour period) surface pressure oscillations are easily recognizable for different geographies. This simple demo has provided patterns matching results reported by a 1999 paper from [NSF National Center for Atmospheric Research].  

### ✍️ Authors

All repos shared in [CurioCopia] are shared under Creative Commons license for others to adopt and use it as they wish.

## 🚀 Usage

1. Generate your own Docker image for [weather-spectrum], place it in your favorite registry.
2. Clone this repo, adjust the essential parameters in [demo], run kustomize to generate the resources.
3. Access the UI via browser and enjoy.

![Alt text](assets/screenshot.png)

## ⬇️ Installation

Let's follow the typical Kustomize installation process.

Define a place to work:
```bash
TEST_HOME=$(mktemp -d)
```
### Establish the Base

```bash
BASE=$TEST_HOME/base
mkdir -p $BASE

CONTENT="https://raw.githubusercontent.com/Curiocopia/blog-signal-searching/refs/heads/main"

curl -s -o "$BASE/#1" "$CONTENT/base\
/{kustomization.yaml,weather-api-deployment.yaml,weather-db-service.yaml,weather-spectrum-config.env,locations.json,weather-api-service.yaml,weather-fetch-job.yaml,weather-spectrum-pvc.yaml,weather-analyze-job.yaml,weather-db-deployment.yaml,weather-reduce-job.yaml,weather-spectrum-run-window.env}"
```
Look at the directory:
```bash
tree $TEST_HOME
```
Expect something like:
```bash
/tmp/tmp.csvhZhFv4x
└── base

```
### The Base Customization

The base directory has a kustomization file:
```bash
more $BASE/kustomization.yaml
```
You can run kustomize on the base to emit customized resources to stdout and inspect:
```bash
kustomize build $BASE
```
Adjust the values of the kustomize output and split the resources into their own files and then execute them in the preferred order.

```bash
RESOURCES=$TEST_HOME/resources
mkdir -p $RESOURCES

kustomize build $BASE -o $RESOURCES
tree $TEST_HOME
/tmp/tmp.csvhZhFv4x

```
Follow the recipe below (adjust for your own exact filenames) for ConfigMap, pvc, `worker` StatefulSet, `worker` headless Service and `merger` Job:
```bash
cd $RESOURCES


```

Once the resources are running, follow the job status
```bash
kubectl get jobs -w 
```
```bash
$ kubectl get jobs -w
NAME        STATUS    COMPLETIONS   DURATION   AGE

```
Once the job is completed, access the UI
```

## Create Overlay

Create a `demo` overlay.
```bash
OVERLAYS=$TEST_HOME/overlays
mkdir -p $OVERLAYS/demo
```
## Demo Customization

```bash
curl -s -o "$OVERLAYS/demo/#1" "$CONTENT/overlays/demo\
/{kustomization.yaml,weather-analyze-job-patch.yaml,weather-spectrum-config-demo.env,weather-spectrum-run-window-demo.env}"
```
Adjust the parameters as you need. Set `namespace` for all resources and `weather-spectrum` in `kustomization.yaml`:
```yaml
namespace: demo

images:
- name: YOUR_REGISTRY/weather-spectrum
  newName: my-registry/weather-spectrum
  newTag: latest
```
Adjust `pi-sets-demo.env` values for ConfigMap creation to use in various reources.

Adjust the values for the `spec.parallelism` and `spec.completion`  in the `weather-analyze-job-patch.yaml` to be identical to `WORKER_COUNT` set in `weather-spectrum-config-demo.env`.


Delete any previous values in the resources and create the new resource files by applying the kustomization.
```bash
rm $RESOURCES/*
kustomize build $OVERLAYS/demo -o $RESOURCES
```
Inspect the values. If you are satisfied, follow the same base recipe for deployment and execution after you create the `demo` namespace.

## 💭 Feedback and Contributing

If you have any other suggestions for improvements or corrections, please drop a note in Discussions.

[Signal Searching]: https://curiocopia.com/blog/signal-searching
[Open Meteo]: https://open-meteo.com
[Curiocopia]: https://curiocopia.com
[NSF National Center for Atmospheric Research]: https://n2t.org/ark:/85065/d7pc33cf
[weather-spectrum]: https://github.com/Curiocopia/weather-spectrum
[demo]: overlays/demo/