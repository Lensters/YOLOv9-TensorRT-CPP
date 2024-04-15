[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![LinkedIn][linkedin-shield]][linkedin-url]


<!-- PROJECT LOGO -->
<br />
  <h3 align="center">YoloV9 TensorRT CPP</h3>
  <p align="center">
    <b>
    A C++ Implementation of YoloV9 using TensorRT
    </b>
    <br/>
    Supports object detection.
</p>
<p align="center">
  <a href="https://github.com/cyrusbehr/YOLOv9-TensorRT-CPP">
    <img width="70%" src="assets/yolov9.gif" alt="logo">
  </a>
  <a href="https://github.com/cyrusbehr/YOLOv9-TensorRT-CPP">
    <img width="70%" src="assets/yolov9-seg.gif" alt="logo">
  </a>
  <a href="https://github.com/cyrusbehr/YOLOv9-TensorRT-CPP">
    <img width="70%" src="assets/yolov9-pose.gif" alt="logo">
  </a>
</p>


### Getting Started
This project demonstrates how to use the TensorRT C++ API to run GPU inference for YoloV9. 
It makes use of my other project [tensorrt-cpp-api](https://github.com/cyrusbehr/tensorrt-cpp-api) to run inference behind the scene, so make sure you are familiar with that project.

### Prerequisites
- Tested and working on Ubuntu 20.04 & 22.04
- Install CUDA, instructions [here](https://developer.nvidia.com/cuda-downloads).
  - Recommended >= 12.0 
- Install cudnn, instructions [here](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#download).
  - Recommended >= 8
- `sudo apt install build-essential`
- `sudo apt install python3-pip`
- `pip3 install cmake`
- Install OpenCV with cuda support. To compile OpenCV from source, run the `build_opencv.sh` script provided [here](https://github.com/cyrusbehr/tensorrt-cpp-api/blob/ec6a7529a792b2a9b1ab466f2d0e2da5df47543d/scripts/build_opencv.sh).
  - Recommended >= 4.8
- Download TensorRT 10 from [here](10x).
  - Required >= 10.0
- Extract, and then navigate to the `CMakeLists.txt` file and replace the `TODO` with the path to your TensorRT installation.


### Installation
- `git clone https://github.com/cyrusbehr/YOLOv9-TensorRT-CPP --recursive`
- **Note:** Be sure to use the `--recursive` flag as this repo makes use of git submodules. 

### Converting Model from PyTorch to ONNX
- Navigate to the [official YoloV9 repository](https://github.com/WongKinYiu/yolov9) and download your desired version of the model (ex. YOLOv9-C).
- Clone the [official YoloV9 repository](https://github.com/WongKinYiu/yolov9). 
- From within the YoloV9 repository, run the following:
- ```python3 export.py --weights <path to your pt file> --include onnx```
- After running this command, you should successfully have converted from PyTorch to ONNX.
- **Note**: If converting the model using a different script, be sure that `end2end` is disabled. This flag will add bbox decoding and nms directly to the model, whereas my implementation does these steps external to the model using good old C++. 
- Move the export onnx model into the `YOLOv9-TensorRT-CPP/modles/` directory. 

### Building the Project
- `mkdir build`
- `cd build`
- `cmake ..`
- `make -j`

### Running the Executables
- *Note*: the first time you run any of the scripts, it may take quite a long time (5 mins+) as TensorRT must generate an optimized TensorRT engine file from the onnx model. This is then saved to disk and loaded on subsequent runs.
- To run the benchmarking script, run: `./benchmark --model /path/to/your/onnx/model.onnx --input /path/to/your/benchmark/image.png`
- To run inference on an image and save the annotated image to disk run: `./detect_object_image --model /path/to/your/onnx/model.onnx --input /path/to/your/image.jpg`
  - You can use the images in the `images/` directory for testing
- To run inference using your webcam and display the results in real time, run: `./detect_object_video --model /path/to/your/onnx/model.onnx --input 0`
- For a full list of arguments, run any of the executables without providing any arguments.

### INT8 Inference
Enabling INT8 precision can further speed up inference at the cost of accuracy reduction due to reduced dynamic range.
For INT8 precision, calibration data must be supplied which is representative of real data the model will see.
It is advised to use 1K+ calibration images. To enable INT8 inference with the YoloV8 sanity check model, the following steps must be taken:
- Download and extract the COCO validation dataset, or procure data representative of your inference data: `wget http://images.cocodataset.org/zips/val2017.zip`
- Provide the additional command line arguments when running the executables: `--precision INT8 --calibration-data /path/to/your/calibration/data`
- If you get an "out of memory in function allocate" error, then you must reduce `Options.calibrationBatchSize` so that the entire batch can fit in your GPU memory.

### Benchmarking
- Before running benchmarks, ensure your GPU is unloaded. 
- Run the executable `benchmark` using the `/images/640_640.jpg` image. 
- If you'd like to benchmark each component (`preprocess`, `inference`, `postprocess`), recompile setting the `ENABLE_BENCHMARKS` flag to `ON`: `cmake -DENABLE_BENCHMARKS=ON ..`.
  - You can then rerun the executable

Benchmarks run on RTX 3050 Ti Laptop GPU, 11th Gen Intel(R) Core(TM) i9-11900H @ 2.50GHz using 640x640 BGR image in GPU memory and FP16 precision. 

| Model        | Total Time | Preprocess Time | Inference Time | Postprocess Time |
|--------------|------------|-----------------|----------------|------------------|
| yolov8n      | 3.753 ms   | 0.084 ms        | 2.625 ms       | 1.013 ms         |
| yolov8n-pose | 2.992 ms   | 0.084 ms        | 2.571 ms       | 0.315 ms         |
| yolov8n-seg  | 15.309 ms  | 0.110 ms        | 4.305 ms       | 10.792 ms        |

TODO: Need to improve postprocessing time. 

### How to debug
- If you have issues creating the TensorRT engine file from the onnx model, navigate to `libs/tensorrt-cpp-api/src/engine.cpp` and change the log level by changing the severity level to `kVERBOSE` and rebuild and rerun. This should give you more information on where exactly the build process is failing.

### Show your appreciation
If this project was helpful to you, I would appreicate if you could give it a star. That will encourage me to ensure it's up to date and solve issues quickly.

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[stars-shield]: https://img.shields.io/github/stars/cyrusbehr/YOLOv9-TensorRT-CPP.svg?style=flat-square
[stars-url]: https://github.com/cyrusbehr/YOLOv9-TensorRT-CPP/stargazers
[issues-shield]: https://img.shields.io/github/issues/cyrusbehr/YOLOv9-TensorRT-CPP.svg?style=flat-square
[issues-url]: https://github.com/cyrusbehr/YOLOv9-TensorRT-CPP/issues
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=flat-square&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/cyrus-behroozi/

