# Summary

This repository contains python wheels where no pre-built version is available.

The binaries can be downloaded under `relases`.

# Funding

The project was funded by the German Federal Ministry of Education and Research under grant number 01IS22S34 from September 2022 to February 2023. The authors are responsible for the content of this publication.

<img src="BMBF_gefoerdert_2017_en.jpg" width="300px"/>

# aarch64 

These are platforms like the Raspberry Pi 4 or NVIDIA Xavier NX.

All compilations are targeting Python 3.8. Install a

~~~shell
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
# Follow instructions on https://github.com/pyenv/pyenv
sudo apt-get install libbz2-dev libssl-dev libreadline-dev libsqlite3-dev libffi-dev
ldconfig
pyenv install 3.8.16
~~~

## Tensorflow

A cross-platform build with `tensorflow/tools/ci_build/ci_build.sh PI-PYTHON38 tensorflow/tools/ci_build/pi/build_raspberry_pi.sh AARCH64` does not work and fails currently,
hence the compilation must be executed on a Raspberry 4 directly (takes 10+ hours)

Before python dependencies can be installed run:

~~~shell
sudo apt-get install gcc g++ python3-dev libhdf5-dev
~~~

### Build TensorFlow directly on Raspberry Pi 4

__NOTE:__ In case the build configuration is changed it might be necessary to delete the bazel cache with `rm -rf ~/.cache/bazel/_bazel_ubuntu`.

~~~shell
git clone git@github.com:tensorflow/tensorflow.git
cd tensorflow
git checkout v2.8.4
wget https://github.com/bazelbuild/bazel/releases/download/4.2.1/bazel-4.2.1-linux-arm64
chmod 755 bazel-4.2.1-linux-arm64
sudo mv bazel-4.2.1-linux-arm64 /usr/local/bin/bazel
~~~

The Python version used for the build is Python 3.8 which is the default on Ubuntu 20.04.

Specify the `march` flag listed in `gcc -march=native -Q --help=target` while running the interactive `configure` script. It should be 
`-march=armv8-a+crc`

Instructions are taken partly from [qengineering](https://qengineering.eu/install-tensorflow-2.2.0-on-raspberry-64-os.html).

~~~shell
./configure
pyenv local 3.8.16
pip3 install numpy==1.23.5 wheel 
pip3 install keras_preprocessing --no-deps
sudo apt install python-is-python3
tmux
bazel --host_jvm_args=-Xmx6000m build --local_cpu_resources=2 --config=nogcp --config=nohdfs --config=nonccl --config=v2 --config=noaws --copt=-ftree-vectorize --copt=-funsafe-math-optimizations --copt=-ftree-loop-vectorize --copt=-fomit-frame-pointer --config=opt //tensorflow/tools/pip_package:build_pip_package
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
# copy the whl file from /tmp/tensorflow_pkg to aarch64 and zip it
~~~

## Tensorflow Addons

For Raspberry Pi 4 [TensorFlow Addons](https://github.com/tensorflow/addons) is needed for Rasa. There is no pre-built pip wheel.

Execute on the Raspberry Pi 4:

~~~shell
git clone https://github.com/tensorflow/addons.git
cd addons
git checkout v0.17.1
pyenv local 3.8.16
python3 -m venv venv
source venv/bin/activate
pip3 install ../python-wheels/aarch64/tensorflow-2.8.4-cp38-cp38-linux_aarch64.whl
python3 ./configure.py
bazel build build_pip_pkg
bazel-bin/build_pip_pkg artifacts
# relase the whl
~~~

## Tensorflow Text

For Raspberry Pi 4 [TensorFlow Text](git@github.com:tensorflow/text.git) is needed.

Execute on the Raspberry Pi 4:

~~~shell
git clone git@github.com:tensorflow/text.git
cd text
git checkout v2.8.2
pyenv local 3.8.16
python3 -m venv venv
source venv/bin/activate
pip3 install aarch64/tensorflow-2.8.2-cp38-cp38-linux_aarch64.whl
./oss_scripts/run_build.sh
# relase the whl
~~~

## confluent-kafka

The python module for confluent-kafka [librdkafka](https://github.com/confluentinc/librdkafka) is needed.

Compile the latest version of the [librdkafka package](https://docs.confluent.io/platform/current/installation/installing_cp/deb-ubuntu.html#get-the-software):

~~~shell
git clone https://github.com/confluentinc/librdkafka.git
cd librdkafka/
git checkout v1.9.2
sudo apt-get install gcc make libevent-pthreads-2.1-7 
./configure
make
sudo make install
~~~

Build the Python wheel:

~~~shell
pyenv local 3.8.16
python3 -m venv confluent-kafka
source confluent-kafka/bin/activate
pip install confluent-kafka
find ~/.cache/pip/wheels/ -name *_kafka*
# copy wheel from cache folder
cp ~/.cache/pip/wheels/<path>/confluent_kafka-1.9.2-cp38-cp38-linux_aarch64.whl aarch64/
rm -rf confluent-kafka
~~~

# Atom Based Processors

These are platforms like the Rock Pi X or MeLe Quieter2Q.

## Tensorflow

The x86_64 wheels for TensorFlow are causing:

> The TensorFlow library was compiled to use AVX instructions, but these aren't available on your machine.

Use a Docker image for building TensorFlow (a direct build was missing some dependencies) on a different Ubuntu based system.
The direct compilation on the Rock Pi X is slow and fails due to memory problems.

~~~shell
docker pull tensorflow/tensorflow:devel
docker run -it -w /tensorflow_src -v $PWD:/mnt -e HOST_PERMS="$(id -u):$(id -g)" tensorflow/tensorflow:devel bash
~~~

Inside the container execute:

~~~shell
git pull
git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
git fetch --all
git checkout v2.8.4
~~~

The Python version used for the build is Python 3.8, and the TensorFlow Docker image is only using version 3.6 on Ubuntu 18.04.

~~~shell
sudo apt-get install nano
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
# Follow instructions on https://github.com/pyenv/pyenv
source ~/.profile 
source ~/.bashrc
sudo apt-get install libbz2-dev libssl-dev libreadline-dev libsqlite3-dev libffi-dev
ldconfig
pyenv install 3.8.16
pyenv local 3.8.16
~~~

Specify the `march` flag listed in `gcc -march=native -Q --help=target` while running `configure`. It should be
* Rock Pi X: `-march=silvermont` 
* LeMe Quieter2Q: `-march=goldmont-plus`

~~~shell
./config
wget https://github.com/bazelbuild/bazel/releases/download/4.2.1/bazel-4.2.1-linux-x86_64
chmod 755 bazel-4.2.1-linux-x86_64
sudo mv bazel-4.2.1-linux-x86_64 /usr/local/bin/bazel
pip3 install numpy==1.23.5 wheel 
pip3 install keras_preprocessing --no-deps
bazel build --config=mkl --config=opt //tensorflow/tools/pip_package:build_pip_package
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /mnt
chown $HOST_PERMS /mnt/tensorflow-2.8.4-cp38-cp38-linux_x86_64.whl
# copy files from host computer
# relase the whl from directory where Docker was started
# quit the docker image
~~~


