
### The Unix Philosophy
- 每一个程序只做好一件事。新的事就尽量用新的程序来解决。
- 预期每一个程序的输入，都是另一个程序的输入。
- rapid prototyping, incremental iteration
- 操作友好

Unix 怎么把这么多工具整合在一起的呢

 If you want to be able to connect any program’s output to any program’s input, that means that all programs must use the same input/output interface.
 In Unix, that interface is a file (or, more precisely, a file descriptor). A file is just an ordered sequence of bytes. 

 应用跟输入输出解耦，makes it easier to compose small tools into bigger systems.

 -  The input files to Unix commands are normally treated as immutable. This
means you can run the commands as often as you want, trying various
command-line options, without damaging the input files.
-  You can end the pipeline at any point, pipe the output into less, and look at it to
see if it has the expected form. This ability to inspect is great for debugging.
-  You can write the output of one pipeline stage to a file and use that file as input
to the next stage. This allows you to restart the later stage without rerunning the
entire pipeline.

## MapReduce and Distributed Filesystems
MapReduce 任务一般像unix，不会改变input。read and write on a Distributed File System.在Hadoop中叫HDFS.
HDFS is based on the shared-nothing principle.

### MapReduce Job Execution
