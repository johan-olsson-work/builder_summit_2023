# builder_summit_2023
Try to deploy Azure ONNX inference server on our camera

Azure provide a runtime image that can be used to deploy models (running on CPU).
Would be interesting to try it out to see how do it perform in comparison to our examples.

Investigate what we can do with ONNX


What is Azure Machine Learning?

Azure Machine Learning is a cloud service for accelerating and managing the machine learning project lifecycle. Machine learning professionals, data scientists, and engineers can use it in their day-to-day workflows: Train and deploy models, and manage MLOps.
You can create a model in Azure Machine Learning or use a model built from an open-source platform, such as Pytorch, TensorFlow, or scikit-learn. MLOps tools help you monitor, retrain, and redeploy models.





Azureml ONNX Runtime 1.6 Inference CPU Image:
[Azureml ONNX Runtime 1.6 Inference CPU Image](https://hub.docker.com/_/microsoft-azureml-onnxruntime-1-6-ubuntu18-04-py37-cpu-inference) 


[Azure ML examples](https://github.com/Azure/azureml-examples ) 



How to use inference prebuilt docker images?


Check [examples in the Azure machine learning GitHub repository](https://github.com/Azure/azureml-examples/tree/main/cli/endpoints/online/custom-container) 

This directory contains examples on how to use custom containers to deploy endpoints to Azure. In each example, a Dockerfile defines an image that may be either an extension of an Azure-originated image such as the AzureML Minimal Inference image or a third party BYOC such as Triton.




Tryring to run inital image
root@axis-b8a44f0d4135:~# docker run mcr.microsoft.com/azureml/minimal-ubuntu20.04-py38-cpu-inference:latest
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
exec /usr/bin/runsvdir: exec format error

Created or own image:
```
FROM arm64v8/ubuntu:20.04

RUN apt-get update && apt-get install -y python3 python3-pip

RUN pip3 install onnxruntime

```


Also downloaded:
```
# --------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.
# --------------------------------------------------------------
# Dockerfile to run ONNXRuntime with source build for CPU

FROM arm64v8/centos:7
MAINTAINER Changming Sun "chasun@microsoft.com"
ADD . /code


RUN /code/dockerfiles/scripts/install_centos_arm64.sh && cd /code && CC=/opt/rh/devtoolset-10/root/usr/bin/gcc CXX=/opt/rh/devtoolset-10/root/usr/bin/g++ ./build.sh --skip_submodule_sync --config Release --build_wheel --update --build --parallel --cmake_extra_defines ONNXRUNTIME_VERSION=$(cat ./VERSION_NUMBER) 

FROM arm64v8/centos:7
COPY --from=0 /code/build/Linux/Release/dist /root
COPY --from=0 /code/dockerfiles/LICENSE-IMAGE.txt /code/LICENSE-IMAGE.txt
RUN yum install -y python3-wheel python3-pip && python3 -m pip install --upgrade pip && python3 -m pip install /root/*.whl && rm -rf /root/*.whl
```


# Got it up and running:
root@axis-b8a44f0d4135:~# docker run onnx_runtime-aarch64
root@axis-b8a44f0d4135:~# docker run -it onnx_runtime-aarch64 bash
root@27e021567550:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@27e021567550:/# [11:24] Yuwei Zhao
bash: [11:24]: command not found
root@27e021567550:/# print("ONNX Runtime version: ", onnxruntime.__version__)
bash: syntax error near unexpected token `"ONNX Runtime version: ",'
root@27e021567550:/# 
root@27e021567550:/# python
bash: python: command not found
root@27e021567550:/# python3
Python 3.8.10 (default, Nov 14 2022, 12:59:47) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> [11:24] Yuwei Zhao
  File "<stdin>", line 1
    [11:24] Yuwei Zhao
       ^
SyntaxError: invalid syntax
>>> print("ONNX Runtime version: ", onnxruntime.__version__)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'onnxruntime' is not defined
>>> 
>>> import onnxruntime
>>> print("ONNX Runtime version: ", onnxruntime.__version__)
ONNX Runtime version:  1.13.1
>>> 



ERROR:
```
root@3d6a7f4733e8:/models/vision/body_analysis/ultraface# python3 demo.py 
Traceback (most recent call last):
  File "demo.py", line 3, in <module>
    import cv2
  File "/usr/local/lib/python3.8/dist-packages/cv2/__init__.py", line 181, in <module>
    bootstrap()
  File "/usr/local/lib/python3.8/dist-packages/cv2/__init__.py", line 153, in bootstrap
    native_module = importlib.import_module("cv2")
  File "/usr/lib/python3.8/importlib/__init__.py", line 127, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
ImportError: libGL.so.1: cannot open shared object file: No such file or directory
root@3d6a7f4733e8:/models/vision/body_analysis/ultraface# 

```



Solution?
```
RUN apt-get install ffmpeg libsm6 libxext6 -y

```




