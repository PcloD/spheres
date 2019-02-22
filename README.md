Ray Tracing in One Weekend in CUDA -Optimized-==============================================This code is an optimization exercise of the excelent [_Ray Tracing in One Weekend in CUDA_](https://github.com/rogerallen/raytracinginoneweekendincuda)I've made several changes to make the code run faster on my hardware:- I switched from the default cuRand random generator to curandStatePhilox4\_32\_10 as it was taking too long to initialize- using _struct_ instead of _class_ for the scene objects- simplified the kernel logic in various places to reduce the register count- initialize the scene on the host and copy it to constant memory- using my own random number generatorThe code includes a Visual Studio solution, and was developped with CUDA 9.2.If you run the renderer from the command line you can optionally pass a number to indicate how many sample it should render per pixel. Default is 10. The argument parsing logic is rudimentary and the program may crash if you don't pass a positive value.On my hardware (GTX 1050), rendering an image of 1200x800 using 10 samples takes ~ .6sI wrote a [blog post](https://voxel-tracer.github.io/Your-First-Cuda-Project/) explaining how to Install Visual Studio 2017, CUDA 9.2 and get your first CUDA project up and running.## UPDATE 2/22/2019I did some more optimizations to the code, mostly trying to reduce the kernel's register usage to improve occupancy. As such, a lot of those optimizations made the code less readable and should only be considered last. Here is a list of all the changes I made:- use rsqrtf() to compute 1/sqrtf() when normalizing vectors- use sincosf() to compute sin() and cos() in a single call- changed scatter() to update passed ray and attenuation directly instead of computing them in separate variables- normalize ray.direction at creation time and remove all unnecessary normalizations later onThis set of changes helped reduce register usage from 56 registers to 46. Performance was slightly better.Other big change I made was changing the render logic to handle 1 sample per thread, and to use a 1-dimensional block (64x1) instead of a 2 dimensional 8x8 block. This helped reduce the register usage to 41 and improved the performance further. The main limitation in my approach is that I store the output of each sample separately and compute the pixel color on the host which means there is a limit on how many samples I can have before I run out of memory. This can be solved though by using a distributed reduction on the device.On my hardware (GTX 1050), here are a few median rendering times over 10 consecutive runs:1200x800x1 0.064s (was 0.063s before the update)1200x800x10 0.469s (was 0.588s before the update)1200x800x100 4.789s (was 5.846s before the update)I updated the renderer to accept 2 optional arguments from the command line: _num\_samples_ (defaults to 1) and _num\_runs_ (defaults to 1). If you pass _num\_runs_ > 1 the renderer will compute the median rendering time and print it at the end. 