# 1. Problem

This problem uses the ResNet-50 CNN to do image classification.

## Requirements
* [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
* [MXNet 18.11-py3 NGC container](https://ngc.nvidia.com/registry/nvidia-mxnet)

# 2. Directions
## Steps to download and verify data
Download the data using the following command:

Please download the dataset manually following the instructions from the [ImageNet website](http://image-net.org/download). We use non-resized Imagenet dataset, packed into MXNet recordio database. It is not resized and not normalized. No preprocessing was performed on the raw ImageNet jpegs.

For further instructions, see https://github.com/NVIDIA/DeepLearningExamples/blob/master/MxNet/Classification/RN50v1.5/README.md#prepare-dataset .

## Steps to launch training

### NVIDIA DGX-1 (single node)
Launch configuration and system-specific hyperparameters for the NVIDIA DGX-1
single node submission are in the `config_DGX1.sh` script.

Steps required to launch single node training on NVIDIA DGX-1:

```
docker build --pull -t mlperf-nvidia:image_classification .
DATADIR=<path/to/data/dir> LOGDIR=<path/to/output/dir> DGXSYSTEM=DGX1 ./run.sub
```

### NVIDIA DGX-2 (single node)
Launch configuration and system-specific hyperparameters for the NVIDIA DGX-2
single node submission are in the `config_DGX2.sh` script.

Steps required to launch single node training on NVIDIA DGX-2:

```
docker build --pull -t  mlperf-nvidia:image_classification .
DATADIR=<path/to/data/dir> LOGDIR=<path/to/output/dir> DGXSYSTEM=DGX2 ./run.sub
```

#### jgw
```
docker build --build-arg PROXY=myproxy.com --pull -t  mlperf-nvidia:image_classification .
DATADIR=/raid/user-scratch/jgwohlbier/mlperf/data/image_classification LOGDIR=$(pwd)/log DGXSYSTEM=DGX2 ./run.sub
```
I have added echos of the docker commands that run.sub issues so that one
could do `docker run`, and `docker exec`.

Note that if changes are made to `run_and_time.sh` one has to rebuild the
container.

Getting the data ready. Assuming in imagenet data location. Did this on
bridges.
```
conda create -n opencv_env python=3 opencv mxnet
conda activate opencv_env
git clone git@github.com:apache/incubator-mxnet.git mxnet
# find im2rec.py in mxnet/tools
python im2rec.py --list --recursive val val
python im2rec.py --pass-through --num-thread 14 --recursive val.lst val
python im2rec.py --list --recursive train train
python im2rec.py --pass-through --num-thread 14 --recursive train.lst train
```



### NVIDIA DGX-1 (multi node)
Launch configuration and system-specific hyperparameters for the NVIDIA DGX-1
multi node submission are in the `config_DGX1_multi.sh` script.

Steps required to launch multi node training on NVIDIA DGX-1:

1. Build the docker container and push to a docker registry
```
docker build --pull -t <docker/registry>/mlperf-nvidia:image_classification .
docker push <docker/registry>/mlperf-nvidia:image_classification
```

2. Launch the training
```
source config_DGX1_multi.sh && CONT="<docker/registry>/mlperf-nvidia:image_classification" DATADIR=<path/to/data/dir> LOGDIR=<path/to/output/dir> DGXSYSTEM=DGX1_multi sbatch -N $DGXNNODES -t $WALLTIME --ntasks-per-node $DGXNGPU run.sub
```

### NVIDIA DGX-2 (multi node)
Launch configuration and system-specific hyperparameters for the NVIDIA DGX-2
multi node submission are in the `config_DGX2_multi.sh` script.

Steps required to launch multi node training on NVIDIA DGX-2:

1. Build the docker container and push to a docker registry
```
docker build --pull -t <docker/registry>/mlperf-nvidia:image_classification .
docker push <docker/registry>/mlperf-nvidia:image_classification
```

2. Launch the training
```
source config_DGX2_multi.sh && CONT="<docker/registry>/mlperf-nvidia:image_classification" DATADIR=<path/to/data/dir> LOGDIR=<path/to/output/dir> DGXSYSTEM=DGX2_multi sbatch -N $DGXNNODES -t $WALLTIME --ntasks-per-node $DGXNGPU run.sub
```
