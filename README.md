# CGgraphV1.0

CGgraph is an ultra-fast graph processing system on a modern commodity machine with the CPU-GPU co-processor, it provides an efficient CPU-GPU cooperative processing scheme. First, CGgraph follows the principle of *load once and multiple times* to extract a subgraph and load it into GPU global memory. Then, CGgraph proposes on-demand task allocation to process graph algorithm by efficient CPU-GPU cooperative processing scheme. Furthermore, both the edge-level and vertex-level workload balance have been explicitly solved in it. 

---

## Getting Started

### 1. packages 
Before building CGgraph ensure that your system has:
- CUDA 11.7 +
- GCC 9.1.0 +
- CMake
- TBB
- CUB
- OpenMP
- gflag

For TBB, you can use the following command to install
`sudo apt-get install libtbb-dev`
For gflag, you can git clone from [gflag](https://github.com/gflags/gflags.git).

### 2. Compilation
``````
git clone https://github.com/PengBo410/CGgraphV1.git
cd CGgraphV1
mkdir build && cd build
cmake ..
make -j
``````

### 3. Input Arg
You can use 'CGgraph --help' get follow info:
``````
-algorithm (The Algorithm To Be Run: [0]:BFS, [1]:SSSP, [2]:WCC, [3]:PageRank)
-computeEngine (The Compute Engine: ([0]: Single-core, [1]Multi-cores, [2]:Single-GPU, [3]Co-CPU/GPU)
-gpuImple (The GPU Implement Type: [0]:CAT, [1]:COALESCE_CHUNK)
-gpuMemory (The GPU Memory Type: [0]:GPU_MEM, [1]:UVM) type: int32
-graphName (The Graph Name)
-orderMethod (The OrderMethod Can Be: [0]:Native, [1]:Random, [2]:CGgraph)
-root (The Scr For BFS/SSSP Or MaxIte For PageRank)
-runs (The Number Of Times The Algorithm Needs To Run)
-useDeviceId (The GPU ID To Be Used)
``````
Here are explanations for several important parameters:

- **algorithm**: Specifies the graph algorithm to be used. In the current version, we provide BFS, SSSP, WCC and PageRank.
- **computeEngine**: Selects the desired execution engine. Specifically, in the current version, we provide four engines: single-core CPU, multi-core CPU, GPU-only, and CPU-GPU collaborative.
- **gpuImple**: Specifies GPU-side optimizations.
- **gpuMemory**: Specifies the type of GPU memory to be used. In the current version, we provide GPU Memory(data located in device) and UVM (data located at host).
- **orderMethod**: Determines the reorder method for data.

### 4. Input Graph

For simplicity during runtime, we store the file path and name of each graph in a structure called GraphFile_type. This allows us to retrieve all information about a graph, including the number of vertices, edges, the path to the TXT file (SNAP format), the path to the CSR binary file, etc., by simply using the graph name. Therefore, users can predefine the information of the graphs they intend to run and store it in this file, located at *CGgraphV1/src/Basic/Graph/graphFileList.hpp* The following is an example:
```cpp
if (graphName == "GS")
{
    graphFile.vertices = 30809122;
    graphFile.edges = 581945983;

    if (OrderMethod_type == OrderMethod_type::NATIVE)
    {
        // you SNAP format file
        graphFile.graphFile = get_BaseGraphFile_path() + graphName + "/native_GS.txt";

        // you csr bin path 
        graphFile.csrOffsetFile =  get_BaseGraphFile_path() + graphName + "/native_csrOffset_u32.bin"; 
        graphFile.csrDestFile =  get_BaseGraphFile_path() + graphName + "/native_csrDest_u32.bin";
        graphFile.csrWeightFile =  get_BaseGraphFile_path() + graphName + "/native_csrWeight_u32.bin";

        return graphFile;
    }
    else if (OrderMethod_type == OrderMethod_type::CGgraphR)
    {
        // you SNAP format file
        graphFile.graphFile = get_BaseGraphFile_path() + graphName + "/CGgraphR_GS.txt";
        
        graphFile.old2newFile = get_BaseGraphFile_path() + graphName + "/CGgraphR_rank.txt";
        graphFile.addtitionFile = get_BaseGraphFile_path() + graphName + "/CGgraphR_addition.txt";
        
        // you csr bin path 
        graphFile.csrOffsetFile = get_BaseGraphFile_path() + graphName + "/CGgraphR_csrOffset_u32.bin";
        graphFile.csrDestFile = get_BaseGraphFile_path() + graphName + "/CGgraphR_csrDest_u32.bin";
        graphFile.csrWeightFile = get_BaseGraphFile_path() + graphName + "/CGgraphR_csrWeight_u32.bin";

        return graphFile;
    }
    else
    {
        Msg_info("Can not find graph [%s]", graphName.c_str());
        exit(1);
    }
}
```

### 5. Run Applications
If you wish to run the BFS algorithm on GS dataset using the CPU-GPU collaborative compute engine of CGgraph, you can use the following command:
``````
./CGgraph -graphName GS -algorithm 0 -root 0 -computeEngine 3 -gpuImple 1 -gpuMemory 0 -orderMethod 2  -useDeviceId 0 -runs 10
``````

### 6. Tips and Tricks
In our project, we've developed some quite interesting tools that I find particularly useful. Let me share a few of them:
- PrettyConsoleOutput 
- Lightweight log
- GPU-FriendlyBitmap