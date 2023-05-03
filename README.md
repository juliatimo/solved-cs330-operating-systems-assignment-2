Download Link: https://assignmentchef.com/product/solved-cs330-operating-systems-assignment-2
<br>



<h1>Introduction</h1>

As part of this assignment, you will be implementing system calls in a teachingOS (<strong>gemOS</strong>). We will be using a minimal OS called gemOS to implement these system calls and provide system call APIs to the user space. The gemOS source can be found in the src directory. This source provides the OS source code and the user space process (i.e., init process) code (in src/user directory).

<h1>Step 0: Running GemOS code</h1>

Extract the provided zip to a folder, let say <strong>Assignment 2</strong>. So, all source files will be inside Assignment 2/gemOS/src folder. Now, start your container using below command:

$ docker start your_container_name

Copy <strong>Assignment 2 </strong>folder to the container using below command and enter password as cs330:

$ scp -P 2020 -r PATH_TO_ASSIGNMENT2_FOLDER <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="97f8e4e0e4d7fbf8f4f6fbfff8e4e3">[email protected]</a>:/home/osws

Now, get the container shell using below command (enter password as cs330) :

$ ssh -p 2020 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="49263a3e3a0925262a282521263a3d">[email protected]</a>

With ls command inside container shell, you will see <strong>Assignment 2 </strong>and <strong>gem5 </strong>folder in your home directory of the container.

Now, to generate kernel binary file, so that you can give this to gem5 running script file, go to src directory inside Assignment 2 folder and run make as following:

$ pwd

/home/osws

$ cd Assignment_2/gemOS/src

$ make

This will generate <strong>gemOS.kernel </strong>binary file in the src directory. Give this path to gem5 running script by going to gem5 directory as following

$ pwd

/home/osws

$ cd gem5

$ ./run.sh /home/osws/Assignment_2/gemOS/src/gemOS.kernel

Now open new terminal window or tab and get the container shell using ssh command mentioned above and run below command to get the shell of <strong>gemOS</strong>:

$ telnet localhost 3456

After getting <strong>gemOS </strong>shell type init command which will run your program logic written in the <strong>init.c </strong>file.

You can also copy your files or folder to the host machine from the container using the same scp command. For this get the container shell and use following command:

$ scp -r PATH_TO_THE_FOLDER <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="523a3d21260d3f33313a3b3c370d27213720123a3d2126">[email protected]</a>_ip:/host_machine_dir

# here host_ip can not be localhost

This way you can test your implementation and submit it in the format given in section 3.

<h1>1        Implement Basic Pipe Operations [40 Marks]</h1>

Pipe is a unidirectional data channel that can be used for inter-process communication. In this part, we will be implementing basic pipe related system calls in gemOS. Functions required to implement these system calls are given in src/pipe.c.

<strong>List of system calls/ functions to implement</strong>

<ul>

 <li>int pipe (int pipefd[2])</li>

 <li>int read (int fd, void *buf, int count)</li>

 <li>int write (int fd, const void *buf, int count)</li>

 <li>int close (int fd)</li>

 <li>int do pipe fork (struct exec context *child, struct file *filep)</li>

 <li>int is valid mem range (unsigned long buff, u32 count, int access bit)</li>

</ul>

<h1>Working</h1>

Let’s look at an example with 2 processes with some basic pipe operations to understand it’s working:

Say, we created a pipe of size 4096 Bytes (defined as a MAX PIPE SIZE in src/include/pipe.h). Currently only one process is alive and let’s call it process 1, it was the one who called for creating a pipe. Process 1 creates another process (process 2) using fork. Now, process 1 calls write 2000 Bytes. Then the first 2000 Bytes of the pipe will be filled with the data provided. Process 1 then calls read 1000 Bytes. Then the data will be read from the pipe and the given buffer will be filled with that data. Note that, now the first 1000 Bytes cannot be read by any process. So, if process 2 wants to read, it can only start reading from 1000 Byte mark.

Now, the pipe has 2000 Bytes of content written in it and 1000 Bytes data has been read by one of the process (process 1). If some process wants to write data into the pipe, it cannot write more than 3096 Bytes (4096 – 2000 + 1000) and the order in which it can write is from 2000 Byte mark to 4096 byte mark then from 1st Byte mark to 1000 Byte mark (as the first 1000 Bytes of the pipe has been read by process 1, the space in that range can be reused).

Now, process 2 calls write 2496 Bytes. Then the data will be written from 2000 Byte mark to 4096 Byte mark and remaining 400 Bytes from 1st Byte mark to 400 Byte mark. Either of the processes can now read from 1000 Byte mark to 4096 Byte mark then from form 1st Byte mark to 400 Byte mark in that order only. So, if process 1 calls read 3296 Bytes then the pipe will be read till 200 Byte mark.

After these operations, the data that can be read by any process will be from 200 Byte mark to 400 Byte mark only (shown in figure 5).

<h1>Pipe structures</h1>

Pipe is represented as pipe info structure defined in src/pipe.c file which you must not change. Two structures pipe info per process, pipe info global are defined as members of this structure. The structure is given below: struct pipe_info {

struct pipe_info_per_process pipe_per_proc[MAX_PIPE_PROC]; struct pipe_info_global pipe_global;

}

pipe info global is used for maintaining information about the pipe which are global to everyone. pipe info per process is used for maintaining information which are specific to a process. These structures are defined in src/pipe.c file. You need to add members according to your need. Note that, members of both structures can only be single variables like int, float, char etc. It <strong>can not be an array or pointer</strong>. The prototype for both structures is given below:

struct pipe_info_global { char *pipe_buff;      // Do not modify this member.

// add members as per your need…

} struct pipe_info_per_process {

// add members as per your need…

}

<h2>1.1       pipe [10 Marks]</h2>

To implement pipe system call, you are required to provide implementation for the template function <strong>create pipe </strong>(in pipe.c) which takes the current context and fd array as argument. Since, pipe has two ends one for reading and other for writing, so you need to find two <strong>smallest </strong>available file descriptors, allocate file objects (<strong>struct file </strong>defined in src/include/file.h) for those descriptors using <strong>alloc file () </strong>function defined in file.c. Then use <strong>alloc pipe info () </strong>(defined in pipe.c) to allocate pipe info object (<strong>struct pipe info </strong>defined in src/pipe.c) where you also need to initialize per process and global info fields for the pipe.

Now, fill-in the fields of those allocated file objects. Note that, we have 2 file objects but both will point to same pipe info object such that first file object is used for reading and second for writing. You need to assign callbacks for reading, writing and closing the pipe on those file objects by accessing the fops field (<strong>struct fileops </strong>type defined in src/include/file.h) of those file objects. Functions <strong>pipe read</strong>, <strong>pipe write </strong>and <strong>pipe close </strong>are callback functions used for reading, writing and closing the pipe respectively which you will implement in upcoming parts.

As last step of pipe call, you need to fill passed fd array such that first index points to the read end and second to the write end of the pipe. The implementation of file objects and operations for <strong>stdin</strong>, <strong>stdout </strong>and <strong>stderr </strong>are already provided (in file.c) to help you with the understanding of the task.

<h2>1.2       read [5 Marks]</h2>

You need to implement the <strong>pipe </strong><strong>read </strong>function (defined in src/pipe.c). This function is to be assigned as the read handler in the file object while creating the pipe (see section 1.1). It takes three arguments with the following prototype: int pipe_read (struct file *filep, char *buff, u32 count)

Here <strong>filep </strong>is the file object, <strong>buff </strong>is the buffer supplied by user and <strong>count </strong>is number of bytes to be read. After successful read, it return number of bytes read. Before reading from the pipe, you need to validate the file object’s access right. If <strong>count </strong>is greater than the present data size in the pipe then just read that much available data and return number of bytes read.

<strong>Note: </strong>This pipe read call will be <strong>non blocking</strong>.

<h2>1.3       write [5 Marks]</h2>

You need to implement the <strong>pipe write </strong>function (defined in src/pipe.c). This function is to be assigned as the write handler in the file object while creating the pipe (see section 1.1). It takes three arguments with the following prototype: int pipe_write (struct file *filep, char *buff, u32 count)

Here <strong>filep </strong>is the file object, <strong>buff </strong>is the buffer supplied by user containing data and <strong>count </strong>is number of bytes to be written. After successful write, it return number of bytes written. Before writing to the pipe, you need to validate the file object’s access right. If <strong>count </strong>is greater than the available space in the pipe then just write that much data which fits in that space and return number of bytes written.

<strong>Note: </strong>This pipe write call will be <strong>non blocking</strong>.

<h2>1.4       close [5 Marks]</h2>

You need to implement the <strong>pipe close </strong>function (defined in src/pipe.c). This function is to be assigned as the close handler in the file object while creating the pipe (see section 1.1). It takes only one argument with the following prototype: int pipe_close (struct file *filep)

Here <strong>filep </strong>is the file object. In this function, you need to closes read or write end of the pipe for that process depending upon the file object’s mode (here you may need to update some per process or global info for the pipe). You need to free the pipe buffer and pipe info object using <strong>free pipe() </strong>function when the pipe is associated with only one process with both end closed.

<strong>file close </strong>function (defined in src/file.c) is called inside <strong>pipe close </strong>function to adjust reference count for that file object and delete the file whenever applicable (<strong>Do not remove this function</strong>).

After successful close, function <strong>pipe close </strong>returns 0. In case of any error, return <strong>-EOTHERS</strong>.

<h2>1.5       Fork Handler: do pipe fork [5 Marks]</h2>

You need to implement the <strong>do pipe fork </strong>function (defined in src/pipe.c). This function will be called when a process creates child using fork system call. It takes two arguments with the following prototype:

int do_pipe_fork (struct exec_context *child, struct file *filep)

Here <strong>child </strong>is the context of child (pointer to the PCB of the child) and <strong>filep </strong>is the file object. In this function, you may need to update some per process or global info for the pipe. Child process will inherit everything from the parent process <span style="text-decoration: line-through;">except the status of parent’s read/ write ends, i.e., both ends will be opened for the child even if parent has closed one of its ends (read/ write)</span>.

Since, pipe has 2 file objects (2 ends), hence this function will be called twice. You need to consider this case while implementing. You may also reach to the limit of processes a pipe can have which is defined as <strong>MAX PIPE PROC </strong>macro in src/include/pipe.h, in that case return error code <strong>-EOTHERS</strong>. Return 0 on success.

<h2>1.6       Memory range checker: is valid mem range [10 Marks]</h2>

You need to implement the <strong>is valid mem range </strong>function (defined in src/pipe.c). Here you need to check whether buffer (user provided) lies in the <strong>mm segment area </strong>or <strong>vm area </strong>with proper access permission. This function should be called inside <strong>pipe </strong><strong>read </strong>function before writing anything to the provided user buffer and inside <strong>pipe write </strong>function before reading anything from the provided user buffer. The function prototype is given below:

int is_valid_mem_range (unsigned long buff, u32 count, int access_bit)

Here <strong>buff </strong>is the starting address of the provided user buffer, <strong>count </strong>is the size of the provided buffer and <strong>access bit </strong>denotes bits to check in the <strong>access </strong><strong>flags </strong>field of <strong>mm segment area </strong>or <strong>vm area </strong>(defined in src/include/context.h). The <strong>access </strong><strong>flags </strong>field is interpreted as <strong>—XWR</strong>, where 1st bit from LSB will be used to denote read, 2nd for write and 3rd for execute permission.

So, if you want to check read permission on the provided buffer, the <strong>access bit </strong>will be <strong>1 </strong>and if you want to check write permission on the provided buffer, the <strong>access </strong><strong>bit </strong>will be <strong>2</strong>.

<h2>ERROR CODES</h2>

You should only use following error codes on errors. All these error codes should be negated before returning (Example: <strong>ENOMEM </strong>should be returned as <strong>-ENOMEM</strong>):

<ul>

 <li><strong>EINVAL </strong>– (Invalid Argument) It should be used in-case of invalid argument such as accessing closed read/ write ends.</li>

 <li><strong>EACCESS </strong>– (No Access) This should be used when you are trying to read from/ write to the pipe with wrong ends.</li>

 <li><strong>ENOMEM </strong>– (Not Enough Memory) It should be used if any memory allocation failed.</li>

 <li><strong>EBADMEM </strong>– (Bad Memory Range) If the provided buffer is not in suitable range or has permission issue.</li>

 <li><strong>EOTHERS </strong>– (Others) In case of any errors which is not specified above use this.</li>

</ul>

<h2>TESTING</h2>

In the GemOS terminal (accessed using the telnet command), you can type <strong>init </strong>to execute the user space process. The user space code is available in src/user/init.c. Three user space files are used to implement the user space logic. They are

<ul>

 <li><strong>c : </strong>Implements the first user space process which can invoke fork() to create more processes. Note that, there is no exec system call yet in the version provided to you. For changing the user space logic, you are required to modify only <strong>init.c</strong>.</li>

 <li><strong>h : </strong>Provides declarations of macros and functions. Note that you <strong>do not </strong>modify this file.</li>

 <li><strong>c : </strong>Implements system call wrappers and provide different user space libraries (e.g., printf). Note that you <strong>do not </strong>modify this file.</li>

</ul>

You need to write your test cases in <strong>init.c </strong>to validate your implementation. The sample test-cases (given in src/user/test cases part1) can be copied into <strong>init.c </strong>to make use of them. If your implementation is correct, the output of executing test cases should match the expected output provided in src/user/test cases part1. The user and kernel code are compiled into a single binary file, i.e., <strong>gemOS.kernel </strong>when built using make from the src directory.

<h1>2        Implement Persistent Pipe Operations [60 Marks]</h1>

In Persistent pipe data is persistent. Unlike basic pipe, in persistent pipe, processes can read already read data by some other process. So, each process maintain its own read and write pointers in persistent pipe. To reclaim the region read by all processes, flush system call is used. We will be implementing persistent pipe related system calls in gemOS. Functions required to implement these system calls are given in src/ppipe.c.

<strong>List of system calls/ handlers to implement</strong>

<ul>

 <li>int ppipe (int pipefd[2])</li>

 <li>int read (int fd, void *buf, int count)</li>

 <li>int write (int fd, const void *buf, int count)</li>

 <li>int close (int fd);</li>

 <li>int flush ppipe (int fd[2]);</li>

 <li>int do ppipe fork (struct exec context *child, struct file *filep );</li>

</ul>

<h1>Working</h1>

Let’s look at an example with 3 processes to understand the working of persistent pipe:

Say there is a process named process i. It creates a persistent pipe of size 4096 Bytes (defined as MAX PPIPE SIZE in src/include/ppipe.h) and creates 2 more processes j, k. So, now in the system there are 3 processes and 1 persistent pipe. Now, process i calls write 1000 Bytes data into it. Process j then calls read 500 Bytes. Now, unlike basic pipe rest all processes can read the first 1000 Bytes. Process k calls read 200 Bytes and reads 200 Bytes amount of data from 1st Byte mark (offset 0) which wouldn’t have been possible if it were basic pipe.

If any process calls write the write will be appended to already written data in the persistent pipe. If process k wants to call write, the amount of data it can write is only 3096 Bytes (4096 – 1000). If flush system call is called then the region of the persistent pipe which has been read by all the processes will be reclaimed and allowed to be written into it.

Process k calls write 1000 Bytes and those 1000 Bytes gets appended. Say process k has called flush, no region will be reclaimed as process i still hasn’t read anything. Now, process i closes it’s read end. Now, say some process (it could be reader or writer) called flush, then the first 200 Bytes of the persistent pipe will be reclaimed.

Process k calls read 3000 Bytes but it will read only 1800 Bytes because that’s the only part which is allowed, i.e., from 200 Byte mark to 2000 Byte mark. Process j calls write 2196 Bytes and it writes from 2000 Byte mark to 4096 Byte mark and then from 1st Byte mark to 100 Byte mark. Process k calls read 2196 Bytes so it reads from 2000 Byte mark to 4096 Byte mark and from 1st Byte mark to 100 Byte mark. Process j calls write 200 Bytes but it will be able to write only 100 Bytes as that’s the only amount of region available in the persistent pipe from 100 Byte mark to 200 Byte mark.

<h1>Persistent Pipe structures</h1>

Persistent Pipe is represented as ppipe info structure defined in src/ppipe.c file which you must not change. Two structures ppipe info per process, ppipe info global are defined as members of this structure. The structure is given below:

struct ppipe_info {

struct ppipe_info_per_process ppipe_per_proc[MAX_PPIPE_PROC]; struct ppipe_info_global ppipe_global;

}

ppipe info global is used for maintaining information about the Persistent pipe which are global to everyone. ppipe info per process is used for maintaining information which are specific to a process. These structures are defined in src/ppipe.c file. You need to add members according to your need. Note that, members of both structures can only be single variables like int, float, char etc. It <strong>can not be an array or pointer</strong>. The prototype for both structures is given below:

struct ppipe_info_global { char *ppipe_buff; // Do not modify this member.

// add members as per your need…

} struct ppipe_info_per_process {

// add members as per your need…

}

<h2>2.1       ppipe [10 Marks]</h2>

To implement ppipe system call, you are required to provide implementation for the template function <strong>create persistent pipe </strong>(in ppipe.c) which takes the current context and fd array as argument. Since, persistent pipe has two ends one for reading and other for writing, so you need to find two <strong>smallest </strong>available file descriptors, allocate file objects for those descriptors using <strong>alloc file () </strong>function defined in file.c. Then use <strong>alloc ppipe info () </strong>(defined in ppipe.c) to allocate persistent pipe info object and fill-in the fields of those allocated file objects (<strong>struct file </strong>defined in src/include/file.h) and ppipe info object (<strong>struct ppipe info </strong>defined in src/ppipe.c). Note that, we have 2 file objects but both will point to same ppipe info object such that first file object is used for reading and second for writing. You need to assign callbacks for reading, writing and closing the pipe on those file objects by accessing the fops field (<strong>struct fileops </strong>type defined in src/include/file.h) of those file objects. Functions <strong>ppipe read</strong>, <strong>ppipe write </strong>and <strong>ppipe close </strong>are callback functions used for reading, writing and closing the persistent pipe respectively which you will implement in upcoming parts. As last step of persistent pipe call, you need to fill passed fd array such that first index points to the read end and second to the write end of the persistent pipe. The implementation of file objects and operations for <strong>stdin</strong>, <strong>stdout </strong>and <strong>stderr </strong>are already provided to help you with the understanding of the task.

<h2>2.2       read [10 Marks]</h2>

You need to implement the <strong>ppipe read </strong>function (defined in src/ppipe.c). This function is to be assigned as the read handler in the file object while creating the persistent pipe (see section 2.1). It takes three arguments with the following prototype:

int ppipe_read (struct file *filep, char *buff, u32 count)

Here <strong>filep </strong>is the file object, <strong>buff </strong>is the buffer supplied by user and <strong>count </strong>is number of bytes to be read. After successful read, it return number of bytes read from the persistent pipe. Before reading from the persistent pipe, you need to validate the file object’s access right. If <strong>count </strong>is greater than the present data size in the persistent pipe then just read that much available data and return number of bytes read.

<strong>Note: </strong>This persistent pipe read call will be <strong>non blocking</strong>.

<h2>2.3       write [10 Marks]</h2>

You need to implement the <strong>ppipe </strong><strong>write </strong>function (defined in src/ppipe.c). This function is to be assigned as the write handler in the file object while creating the pipe (see section 2.1). It takes three arguments with the following prototype:

int ppipe_write (struct file *filep, char *buff, u32 count)

Here <strong>filep </strong>is the file object, <strong>buff </strong>is the buffer supplied by user containing data and <strong>count </strong>is number of bytes to be written. After successful write, it return number of bytes written to the pipe. Before writing to the persistent pipe, you need to validate the file object’s access right. If <strong>count </strong>is greater than the available space in the persistent pipe then just write that much data which fits in that space and return number of bytes written.

<strong>Note: </strong>This persistent pipe write call will be <strong>non blocking</strong>.

<h2>2.4       close [10 Marks]</h2>

You need to implement the <strong>ppipe close </strong>function (defined in src/ppipe.c). This function is to be assigned as the close handler in the file object while creating the persistent pipe (see section 2.1). It takes only one argument with the following prototype:

int ppipe_close (struct file *filep)

Here <strong>filep </strong>is the file object. In this function, you need to closes read or write end of the persistent pipe for that process depending upon the file object’s mode (here you may need to update some per process or global info for the persistent pipe). You need to free the persistent pipe buffer and ppipe info object using <strong>free ppipe() </strong>function when the persistent pipe is associated with only one process with both end closed.

<strong>file close </strong>function (defined in src/file.c) is called inside <strong>ppipe close </strong>function to adjust reference count for that file object and delete the file whenever applicable (<strong>Do not remove this function</strong>).

After successful close, function <strong>ppipe close </strong>returns 0. In case of any error, return <strong>-EOTHERS</strong>.

<h2>2.5       flush ppipe [10 Marks]</h2>

You need to implement the <strong>do flush ppipe </strong>function (defined in src/ppipe.c). This function will be used to reclaim the space for reusing which has been read by all the processes. This function will be invoked when system call flush ppipe is called. The prototype is given below: int do_flush_ppipe (struct file *filep)

It takes one argument <strong>filep </strong>which is the file object. It is supposed to return the number of Bytes that have been reclaimed which will be the amount of data read by all the processes (as shown in figure 8 of section 2).

<h2>2.6       Fork Handler: do ppipe fork [10 Marks]</h2>

You need to implement the <strong>do ppipe </strong><strong>fork </strong>function (defined in src/ppipe.c). This function will be called when a process creates child using fork system call. It takes two arguments with the following prototype:

int do_ppipe_fork (struct exec_context *child, struct file *filep)

Here <strong>child </strong>is the context of child (pointer to the PCB of the child) and <strong>filep </strong>is the file object. In this function, you may need to update some per process or global info for the persistent pipe. Child process will inherit everything from the parent process <span style="text-decoration: line-through;">except the status of parent’s read/ write ends, i.e., both ends will be opened for the child even if parent has closed one of its ends (read/ write)</span>.

Since, persistent pipe has 2 file objects (2 ends), hence this function will be called twice. You need to consider this case while implementing. You may also reach to the limit of processes a persistent pipe can have which is defined as <strong>MAX </strong><strong>PPIPE PROC </strong>macro in src/include/ppipe.h, in that case return error code <strong>-EOTHERS</strong>. Return 0 on success.

<h2>ERROR CODES</h2>

You should only use following error codes on errors. All these error codes should be negated before returning (Example: <strong>ENOMEM </strong>should be returned as <strong>-ENOMEM</strong>):

<ul>

 <li><strong>EINVAL </strong>– (Invalid Argument) It should be used in-case of invalid argument such as accessing closed read/ write ends.</li>

 <li><strong>EACCESS </strong>– (No Access) This should be used when you are trying to read from/ write to the pipe with wrong ends.</li>

 <li><strong>ENOMEM </strong>– (Not Enough Memory) It should be used if any memory allocation failed.</li>

 <li><strong>EOTHERS </strong>– (Others) In case of any errors which is not specified above use this.</li>

</ul>

<h2>TESTING</h2>

The test procedure is similar to what mentioned in Part 1. Some sample test cases and their expected outputs are given in the src/user/test cases part2 folder for this part. You can write your own test cases in <strong>init.c</strong>.

<h1>3        Submission</h1>

<ul>

 <li>Make sure that your implementation does not print any unnecessary info like some debug messages, error messages, etc.</li>

 <li>You have to submit zip file named <strong>your roll number.zip</strong>, Eg: <strong>zip </strong>containing only pipe.c and ppipe.c in specified folder format:</li>

</ul>

– YourRollNo/src/pipe.c, YourRollNo/src/ppipe.c