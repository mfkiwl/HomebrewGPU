# HomebrewGPU

HomebrewGPU is a simple ray tracing GPU on FPGA which implements basic ray-primitive intersection, BVH traversal, shadowing, reflection and refraction.
This is a project I used to learn programming in Verilog and I think it should be educational to someone who is new to FPGA.

![](/doc/HomebrewGPU.gif "")
![](/doc/HomebrewGPU_2.gif "")
![](/doc/HomebrewGPU_3.gif "")

## FPGA Board
[NEXYS A7 XC7A100T](https://digilent.com/reference/programmable-logic/nexys-a7/start)

## EDA Tool
[Vivado 2021.2](https://www.xilinx.com/support/download.html)

## Number Format
The GPU uses 2 basic number formats for ALU.

1. Q18.14 fixed point
2. Q2.14 fixed point for normalized value

## Architecture
![](/doc/GPU_Architecture.png "")

### Thread Generator
Generate one thread per clock for each ray core. Basically, each thread presents one pixel in the frame buffer. The thread will go through ray core and output the final color of corresponding pixel.

### BVH Structure
BVH structure stores the BVH tree structure data. It accepts the node or leaf query from ray core and output the node or leaf data to ray core.

### Primitive Unit
Primitive Unit stores the raw data of all primitives of a scene. It accepts the query from ray core and output the related primitive data to ray core.

### [Ray Core](/doc/RayCore.md)
Ray core process one thread to output the final color of the pixel. It accepts the thread from thread generator or reflection/refraction ray. It is a 3 stages pipeline, surface, shadow and shade stage. Multiple threads can be processed in ray core at different stages. The design of ray core is scalable so you can use configuration to increase the number of ray cores to increase the throughput.

### Frame Buffer Writer
This module is reponsible for cache the output of ray cores and write the pixel data to the frame buffer. It uses 8 sets of 16-pixel wide cache to store the output of ray cores. Some threads with reflection/refraction take longer to get the final color so this module will wait util all threads in one cache set are finished then write the data to the frame buffer.
