FROM arm64v8/ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive 

RUN apt-get update && apt-get install -y python3 python3-pip git
RUN apt-get install ffmpeg libsm6 libxext6 -y


RUN git clone https://github.com/onnx/onnx.git
RUN cd onnx
RUN git submodule update --init --recursive
# Optional: prefer lite proto
ENV CMAKE_ARGS=-DONNX_USE_LITE_PROTO=ON
RUN pip3 install -e .

RUN pip3 install opencv-python
RUN pip3 install argparse
RUN pip3 install numpy

COPY ./models models

