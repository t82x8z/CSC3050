java cCSC3050 Project 3: RISC-V Simulator with RVV
1 Background
RISC-V, an open standard instruction set architecture (ISA), has rapidly become a
pivotal force in academic research and industrial development due to its flexibility
and open-source nature. Unlike proprietary ISAs, RISC-V offers the freedom for
developers to customize and extend the architecture, making it an ideal platform
for innovation in research, education, and the design of specialized hardware. One
of its most impactful extensions is the RISC-V Vector Extension (RVV), which
introduces efficient vector processing capabilities—a cornerstone of modern high performance computing. This is especially critical for applications like machine
learning, cryptography, and scientific simulations, where parallel data processing is
essential for improving computational speed and efficiency.
In this project, you are tasked with extending the QTRVSim RISC-V simulator
to support vector operations by implementing some of the RVV instructions.
After reviewing the number of cycles, you will get a feeling of how this is faster
than conducting element-wise operations.
Start early, this project can be time-consuming if you are not familiar with
simulators.
2 QTRVSim
QTRVSim is a RISC-V CPU simulator for education, where you can try its online
version on this link. Just in case you want to try different instructions, you can refer
to this page: RISC-V Instruction Set Specifications. A helpful video about using
QTRVSim can be found on Youtube
After familiarizing yourself with the QtRVSim manual, you can begin planning how
to integrate RVV instructions into the existing implementation. The simulator’s
source code, written in C++ and including both the core simulation functions and
graphical user interfaces (GUIs), can be found in the repository at this link. To test
your modifications, QtRVSim offers two methods for simulating assembly code: GUI
or command-line prompts.
Note: For this project, you are not required to modify any of the GUI components.
Your primary goal is to ensure that the RVV instructions function correctly when
using command-line prompts. Another objective in this project is to save the number
of cycles; the smaller the number you get, the better the score you get.
1
2.1 How to run
We give the example of running QTRVSim on Ubuntu with the terminal. You can
follow these steps:
1. We assume you already have the necessary packages for compiling cpp. If
not, you can easily find tutorial for them on the internet.
2. Install QT6 (QT5 does not work in most cases) with sudo apt install qt6-
base-dev. You might need sudo apt update first, and make sure you are
installing QT6, not QT5.
3. Download QTRVSim from the given repository.
4. Make a new directory for building files (mkdir build; cd build)
5. cmake -DCMAKE BUILD TYPE=Release /path/to/qtrvsim
6. make -j X, where X is the number of threads you want to use
7. If everything goes correctly, you can use ./target/qtrvsim cli –asm XXXXX.S
to run your .S file.
8. Via ./target/qtrvsim cli –help, you can check all helpful arguments.
3 RVV Instructions
In this assignment, you are required to implement the following RVV instructions
(suppose max vector size is 32):
1. vsetvl rd, rs1, rs2: sets the length register vl to rs1 and rd, also sets the
register holding the type of vector to rs2 (8/16/32).
2. vadd.vv vd, vs2, vs1: adds two vectors vs2 and vs1, and stores the result
in vd
3. vadd.vx vd, vs2, rs1: adds rs1 to each element of vector vs2, and stores
the result in vd
4. vadd.vi vd, vs2, imm: adds the scalar value imm to each element of vector
vs2, and stores the result in vd
5. vmul.vv vd, vs2, vs1: conducts dot production on two vectors vs2 and vs1,
and stores the result in vd
6. vlw.v vd, (rs1): loads elements stored starting at rs1 into vector vd. The
length to load is dependent on the length stored at vl and the unit length
specified earlier.
7. vsw.v vs3, (rs1): stores vector elements of vs3 into memory starting at rs1.
The length to load is dependent on the length stored at vl and the unit length
specified earlier.
2
Figure 1: Matrix stored as vector
The whole point of this project is that, through the implementation, you will
understand why are vector operations is much faster than manipulate each ele ment individually. For example, writing 100 elements into memory will require 100
individual store instructions if in an element-wise manner. However, using vector
write, you only need to do one vector store instruction.
A detailed explanation of RVV instructions can be found at this manual. Reminder:
Do not forget to update vl when switching to operate on vectors with different
lengths.
4 Matrix Multiplication<代 写CSC3050、C++
代做程序编程语言br>After implementing and testing the aforementioned functionalities, you are required
to write a .S file that conduct matrix to matrix multiplication.
Ci,j =
X Ai,kBk,j
k
The actual matrix will be stored as a vector in memory, as shown in Figure 1. In
order to conduct vector multiplication, the size of the matrix n × m will be given.
We require you to generate two random matrices with sizes of 20 × 46 and
46 × 50 where elements can be of your own choice.
5 Tricks
There are several tricks you can apply to reduce cycle counts.
1. Reduction (required): This is similar to calculate the summation of a
vector, but more efficiently. The basic requirement is that you conduct this
summation on each element one-by-one, which leads to excessive cycles.
Another approach is to do binary split, i.e. repeatedly decompose the a vector
of size n into 2 vectors of size n//2, and then conduct vadd. There are also
other trick for conducting reduction, and you can explore any of them.
3
Possible reduction:
(a) scalar loop
(b) vector shift
(c) reduction instruction
(d) ...
2. Chaining (Extra credit): When conducting vector operations, it is not nec essary to wait for the entire instruction to complete. As shown in Figure 2, it
is possible to conduct VADD on the first element, right after obtaining the
first element of VMUL. A much better illustration can be found at Prof.Hsu’s
slides at this link.
Figure 2: chaining
6 Instruction on Implementation
The code involved in QTRVSim is quite complicated. Luckily, you only need to
focus on few script files.
1. src/machine/instruction.cpp: Edit this file to add new instructions. The
boxed fields are:
• instruction name
• instruction enum type (you can edit this by yourself; no need to follow
the example)
• input types (you can go through instruction.cpp to see what char is for
what type)
• machine code (hexadecimal)
• mask for effective bits for instruction (hexadecimal)
• customize flags (you can edit this by yourself; no need to follow the
example)
2. src/machine/core.cpp: Main pipeline of the simulator. You can find fetch,
decode, execute, writeback, memory in it, and edit these codes for your con venience.
4
3. src/machine/execute/alu.cpp: specify what to do for each alu operation.
You can create/edit these codes for your own convenience.
Other files might also interest you, but we will not go through all of them here.
Feel free to modify any codes as long as they work.
Notice: you need to use state.cycle count++; in core.cpp when needed.
Notice2: If you want to use v1,v2... as the vector register, you can modify
parse reg from string() in instruction.cpp.
Notice3: You might want to check dt.num rt, dt.num rd, dt.num rs for specific
register indexing.
Notice4: The largest vector register length is 32. Load instruction will have a
memory latency of 32. Besides, the cycles for multiplication is 4. (This means that,
to load a vector of length 10, the total cycles will be 1 + 1 + 32 + 10 + 1 + 1 = 46)
7 Grading Criteria
The maximum score you can get for this lab is 100 points. We will first exam ine the correctness of your outputs to test cases. Since hard-coding each opera tion is fairly easy in C++, we will check the execution information, such as the
number of cycles, and content in memories/registers. Using of ChatGPT to im prove writing/generate codes/provide ideas is allowed and highly-recommended
as ChatGPT has become one of the best productivity tools.
Conducting ”higher-level” reduction or finishing the task with less number of cycles
will be granted with extra credit.
You are also required to compose a report, where you should show the results
of your test case executions. Besides you also need to show the total number of
cycles and explain where those cycles come from. (few sentences, no need to be
super specific.)
The deadline of this project is 23:59, Tuesday, 2024/11/19. For each day after
the deadline, 10 points will be deducted from your final score up to 30 points, after
which you will get 0 points.
Besides, if anyone is interested in developing with QT, you are more than welcome
to implement GUI support for RVV instruction. If done properly, you will earn extra
credits, and might contribute to future contents of this class.
Feel free to ask questions if you find anything confusing.
5
8 Submission
You should make sure your code compiles and runs. Then, it should be compressed
into a .zip file and submitted to BlackBoard. Any necessary instructions to
compile and run your code should also be documented and included. Finally, you are
also required to include a report containing the results of your test case execution.
6

         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
