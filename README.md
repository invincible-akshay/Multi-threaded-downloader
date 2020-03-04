# Multi-threaded-downloader
A multi-threaded downloader  command line utility


## INSTALLATION

Make sure python3.7 or above is installed and available on PATH environment variable. 

The following 2 libraries are required: 
1) click 2) requests
Install them using your favorite package manager, example: pip, conda, etc.


### A. UNIX/LINUX/MACOS
Step #1. Add the directory where the script file exists into path variable. Example - If script is in /Users/akshaydnehe/Documents/ then run the following command on your respective cli: 
export PATH=/Users/akshaydnehe/Documents/:$PATH

Step #2. Give the script executable permissions using the command:
chmod u+x downloader.py

[OPTIONAL] You can create a copy of the script to have an executable without the extension using the command -
cp downloader.py downloader


### B. WINDOWS
The steps as applicable for UNIX/LINUX/MACOS are applicable to Windows.
PATH variable can be updated using:
set PATH=%PATH%;C:\Users\akshaydnehe\Documents

You can also refer this stack overflow answer for more details:
https://stackoverflow.com/questions/11472843/set-up-python-on-windows-to-not-type-python-in-cmd




## II. USAGE
Depending on the way you performed the installation steps in previous sections, you can execute the script in 1 of following ways:

a) python downloader.py <URL> [--number_of_threads=nThreads] [--name=fileName]

b) ./downloader <URL> [--number_of_threads=nThreads] [--name=fileName]

Examples:
1. ./downloader https://file-examples.com/wp-content/uploads/2017/04/file_example_MP4_1920_18MG.mp4

2. python downloader.py https://file-examples.com/wp-content/uploads/2017/04/file_example_MP4_1920_18MG.mp4 --number_of_threads=3

3. python downloader.py https://file-examples.com/wp-content/uploads/2017/04/file_example_MP4_1920_18MG.mp4 --number_of_threads=8 --name="my_new_file.mp4"


There is one mandatory argument: 
URL: This is the fully qualified URL of the file to be downloaded.

There are 2 optional arguments:
a) number_of_threads=nThreads: nThreads is to be replaced with an integer value representing the number of desired threads to use for downloading. If not specified, default value is 2.
There is always a tradeoff to be considered when using multi-threading related to context-switching. If too many threads for smaller file sizes are used then the added overhead due to context-switching can cause longer execution time compared to sequential execution. 

Always remember, with more power comes greater responsibility :) 


b) name=fileName: fileName is to be replaced with a string representing the file name to give the downloaded file. Can be used if user wishes to specify a different name for the file. If not specified, the resource name (from URL) will be used as the filename.




## III. DESIGN
The script design is fairly simple. Here is a high-level overview of few notable points:

1. INPUT: Arguments are managed using click module which is very helpful to create interactive cli-based programs. Mandatory and optional arguments can be configured very easily using click instead of defining using argparse as was traditionally done;


2. OUTPUT: All console I/O is done via logging module, instead of simply using print() statements. This makes it easily extensible for future need of writing and maintaining logs in separate log files for persistence.

Also, SHA256 HASH is displayed at end of download, this can be used to validate the file integrity;


3. VISUAL PROGRESS FEEDBACK: Currently there is no way to signal the progress to the user other than on completion of some chunks download. In case of large chunks, the user may be left in the dark for longer durations. In ideal case, there should be some progress bar kind of response. Tkinter or tqdm can be used in the future to implement this functionality;


4. DOWNLOAD RESUME FUNCTIONALITY: If program fails or is terminated before completion, download restarts from new file, ignoring/re-writing the partial download from previous run(s). 

Resume download functionality is not currently implemented. This can be done by using the same chunk-wise download mechanism that is currently in place. Each thread will check if respective chunk has been downloaded or not and can request only the remaining range of bytes and write them at respective location. This will be very useful for use cases with very large files;


5. EXCEPTION HANDLING: Some exception handling has been done like checking if the URL allows partial downloads. In case partial downloads are not allowed then program is switched to single thread mode, irrespective of user input as number of threads. 

More scenarios can be checked in future, for example whether enough disk space is available or not for the file download, whether write access is available and so on;


6. THREAD_COUNT: As long as user inputs a positive integer for count of threads, it is acceptable. But additional checks can be added based on file size, to throttle the number of threads spawned because the additional cost of context switching can take away the benefits of multi-threading. 

There is a trade-off and some mechanism can be devised to check whether the number of threads are within a range of useful values, else should be defaulted to particular values;


7. CONCURRENCY: For multi-threaded download, the threads are spawned to download (roughly) equally-sized chunks. To achieve true concurrency, if multiple processors are available, multi-processing can be used instead of multi-threading for large files as each process will access and hence lock over specific section of the file. The entire file need not be locked by multiple threads;


8. UNHANDLED SCENARIOS: Scenarios like TIMEOUT are not handled currently. Threads could remain waiting on some resource and the response may take too long.
Currently only HTTP URLs are supported. Support can be added for FTP, SFTP, SSH;




## IV. TEST SCENARIOS
Tests of the following nature have been performed:
A. Download various file formats (videos, pdfs, docx);
B. Invalid/Incomplete URLs;
C. URLs which do not allow partial requests;
D. Time of execution on varying number of threads using the UNIX time utility;
E. Success status of all threads is checked before declaring successful download;
F. No connection, disrupted connection scenarios were tested;




## V. FAQ

What happens if your connection fails during a download?
A> Script waits for connection to be re-established and resumes if download was already in progress. If connection isn't present when executed a GET request then code exits with exception.


What is the impact your implementation might have on system resource usage?
A> Currently timeout hasn't been enforced so threads can continue executing without timeout.


A user will always find a way to shoot themselves in the foot. How do you make sure they don't hurt themselves?
A> Exception handling has been implemented for certain foreseeable scenarios. 


Can you make downloads faster? If so how?
A> Multi-threading has been used to download chunks concurrently. Since the task is more I/O bound, multi-threading is meaningful in this use case and context switching overhead is not too high unless thread count is not compatible with task size.


How can you ensure quality does not degrade?
A> Checksum is output at end of execution to validate the file integrity after download.


Are threads really threads?
A> Yes, Python threads are implemented using OS threads. But the GIL prevents multiple threads from running at once. However C Extensions in Python can be used to acquire/release GIL as per our need.

In our scenario, since the operation is more I/O bound than CPU-bound, multi-threading is actually beneficial as while a thread may be waiting to receive all data from server, another thread could be writing it's chunk of data to file.


And can it scale? 
A> Yes, multi-processing can be used, processes can run on different cores in a truly concurrent fashion. 

Map-Reduce can be used to distribute the download tasks to multiple nodes.
