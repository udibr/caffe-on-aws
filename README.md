caffe-on-aws
============
[Caffee](http://caffe.berkeleyvision.org/) is the state of the art in deeplearning.


I've started with an AWS image "Ubuntu Server 14.04 LTS (HVM) - CUDA 6.5 - ami-2cbf3e44".
I have used a security policy which allowed me to connect to port 8888.

On the machine run
```bash
sudo rm -rf cuda_6.5.14_linux_64.run nvidia_installers/
sudo apt-get update
sudo apt-get install libopenblas-dev
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler
sudo apt-get install git
```

Next I installed anaconda
```bash
wget http://09c8d0b2229f813c1b93-c95ac804525aac4b6dba79b00b39d1d3.r79.cf1.rackcdn.com/Anaconda-2.1.0-Linux-x86_64.sh
bash Anaconda-2.1.0-Linux-x86_64.sh
conda update conda
conda update anaconda
conda update ipython
conda update scikit-learn
conda install mkl
sudo ln -s  ~/anaconda /opt/anaconda1anaconda2anaconda3
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
ipython profile create nbserver
```
You need to edit `/home/ubuntu/.ipython/profile_nbserver/ipython_notebook_config.py`
see [instructions](http://ipython.org/ipython-doc/1/interactive/public_server.html.)

Finally I was ready to install caffe
```bash
git clone https://github.com/BVLC/caffe.git
cd caffe/
cp Makefile.config.example Makefile.config
mv $HOME/anaconda/lib/libm.so $HOME/anaconda/lib/libm.so.tmp
mv /home/ubuntu/anaconda/lib/libm.so.6 /home/ubuntu/anaconda/lib/libm.so.6.tmp
```
You need to edit Makefile.config
```
27,28c27,28
<               -gencode arch=compute_50,code=sm_50 \
<               -gencode arch=compute_50,code=compute_50
---
>               #-gencode arch=compute_50,code=sm_50 \
>               #-gencode arch=compute_50,code=compute_50
34c34
< BLAS := open
---
> BLAS := atlas
48,49c48,49
< #PYTHON_INCLUDE := /usr/include/python2.7 \
< #             /usr/lib/python2.7/dist-packages/numpy/core/include
---
> PYTHON_INCLUDE := /usr/include/python2.7 \
>               /usr/lib/python2.7/dist-packages/numpy/core/include
51,53c51,53
< PYTHON_INCLUDE := $(HOME)/anaconda/include \
<                $(HOME)/anaconda/include/python2.7 \
<                $(HOME)/anaconda/lib/python2.7/site-packages/numpy/core/include
---
> # PYTHON_INCLUDE := $(HOME)/anaconda/include \
>               # $(HOME)/anaconda/include/python2.7 \
>               # $(HOME)/anaconda/lib/python2.7/site-packages/numpy/core/include
56,57c56,57
< #PYTHON_LIB := /usr/lib
< PYTHON_LIB := $(HOME)/anaconda/lib
---
> PYTHON_LIB := /usr/lib
> # PYTHON_LIB := $(HOME)/anaconda/lib
```
and make caffe
```bash
make all
make test
```
Add the following to end of `~/.bashrc` and re-login
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/ubuntu/anaconda/lib/
export PATH=/usr/local/cuda/bin:$PATH
```
now you can run caffe
```bash
make runtest
```
or run a full network, for example [LeNet](http://caffe.berkeleyvision.org/gathered/examples/mnist.html)
```
./data/mnist/get_mnist.sh
./examples/mnist/create_mnist.sh
./examples/mnist/train_lenet.sh
```
you can switch between GPU and CPU usage by editing `examples/mnist/lenet_solver.prototxt`


You can now run an ipython notebook server with the following command:
```bash
cd
ipython notebook --profile nbserver
```
and connect to your machine from a browser on port 8888. For example on OS X terminal
```
open https://ec2-54-197-210-161.compute-1.amazonaws.com:8888/notebooks/caffe/examples/hdf5_classification.ipynb
```