Download Link: https://assignmentchef.com/product/solved-cs2106-lab-5-part-1-and-2-simple-file-operations-and-mystery-and-usfat-file-system
<br>
PART 1

Exercise 1 is a simple task on unix file operations. The lab demo exercise 1 probably takes less than 5 lines of code J.

Please note that additional exercise(s) will be released only in week 11 as they requires additional topics covered in lecture 11. <strong>Note that there is no change to the lab demo schedule (i.e. demo only week 11 and 12 as per normal). </strong>There is no formal lab session on week 13, so you can use the lab as needed to finish the additional exercises.




<h1>Section 2. Exercise One [Lab Demo Exercise]</h1>

Take some time to familiarize yourself with the following Unix file related system calls:

<ul>

 <li><strong>read()</strong>: Reading data from an opened file descriptor.</li>

 <li><strong>lseek()</strong>: Move to a specified location in the file</li>

</ul>

You can read through the lecture slide and/or the relevant man pages on Unix for the above functions. Hint: You do not need any other file-related system calls to solve exercise 1 and 2.

Take a look at the sample execution session below. User input in <strong>bold </strong>font.




<table width="520">

 <tbody>

  <tr>

   <td width="267">Sample Run 1</td>

   <td width="253"> </td>

  </tr>

  <tr>

   <td width="267">File Name: <strong>10int.dat </strong>Size = 40 bytes123124</td>

   <td width="253">One of the provided input data file.File size is printedData in file are read and printed (a total of 10 integers in this case).</td>

  </tr>

 </tbody>

</table>

1 | P a g e




Prepared By SYJ                                                                                                          AY1819S1 CS2106

<table width="520">

 <tbody>

  <tr>

   <td width="267">125126127128129130131132</td>

   <td width="253"> </td>

  </tr>

 </tbody>

</table>




<table width="520">

 <tbody>

  <tr>

   <td width="267">Sample Run 2</td>

   <td width="253"> </td>

  </tr>

  <tr>

   <td width="267">File Name: <strong>wrong.dat </strong>Cannot Open</td>

   <td width="253">This is a non-existent file. Complain and exit.</td>

  </tr>

 </tbody>

</table>




The given skeleton file <strong>ex1.c</strong> has quite a large chunk of logic implemented. Your tasks are essentially:

<ol>

 <li>Check that the file can be opened.</li>

 <li>Find out the file size (hint: use <strong>lseek()</strong>).</li>

 <li>Read all data until end of the file.</li>

</ol>




<strong><u>A note on 32-bit vs 64-bit</u> </strong>

<strong> </strong>

The exercises in this lab are sensitive to the word size of the underlying execution environment. As some of you may have a 64-bit processor and OS, it is a little tricky to ensure the lab works across both 32-bit and 64-bit environments. For maximum compatibility, we have decided to stick to 32-bit for this lab.




To ensure the correct execution environment, the first part of the skeleton source will do a simple check on the integer size and warn you if your machine + compiler operates in 64-bit. It will terminate the program if 64-bit environment is detected.




The remedy is quite simple, you just need to compile your source code with an additional flag “<strong>-m32</strong>“, e.g.




<strong>gcc ex1.c –m32</strong>       //compiles as 32-bit application.




Please make sure you compile your code correctly.




<strong><u>For your own exploration:</u> </strong>




<ol>

 <li>If you open up the <strong>10int.dat</strong> in a normal editor, what do you see? Can you explain?</li>

</ol>




PART 2

<h1>Section 1. USFAT File System Overview</h1>

USFAT (pronounced as <strong>/ˈʌŋkl/ /suː/ /ɪz/ FAT</strong>) is a <strong>fictional </strong>file system invented just for CS2106 J! It draws inspirations from the basic FAT based file allocation scheme and the MS-DOS FAT16 file system. Your tasks in this lab is to understand and provide functionalities that interact with the underlying USFAT file system.

<h2>1.1 USFAT File System Layout</h2>

<strong> </strong>

Absolute Sector

# è                 <strong>0              1               2               3              …             …           128 </strong>

<table width="438">

 <tbody>

  <tr>

   <td width="56"><strong>FAT </strong></td>

   <td width="66"><strong>Data </strong></td>

   <td width="66"><strong>Data </strong></td>

   <td width="66"><strong>Data </strong></td>

   <td width="66"><strong>… </strong></td>

   <td width="66"><strong>… </strong></td>

   <td width="50"><strong>… </strong></td>

  </tr>

 </tbody>

</table>

Data Sector #             <strong>              0               1               2              …             …           127 </strong>

è

<strong> </strong>

Note that a “logical block” is the same size as a “sector” in USFAT and we use the two terms interchangeably.




There are a total of 129 sectors (with absolute index 0 to 128) and each sector is 256 bytes. So, a typical USFAT file system is 129 * 256 = 33,024 bytes in size. The FAT table <strong>occupies one sector</strong> and is located at sector 0. All remaining sectors (128) are used for data storage.




Each FAT entry is <strong>2 bytes </strong>in size, i.e. the FAT contains 256 / 2 = 128 entries. Note that the FAT index refers only to blocks in the file data region. For ease of reference (and coding), we will use <strong>data sector number </strong>to indicate sectors in the data region. For example, the status of data sector 2 can be found in FAT[2], but the actual storage on the “hard disk” is at sector 3. So, pay attention to whether you are using data sector number of the absolute sector number during coding to avoid “off-by-one” errors.

Each FAT entry can contain one of the following values:




<table width="440">

 <tbody>

  <tr>

   <td width="130"><strong>Values </strong></td>

   <td width="310"><strong>Meaning </strong></td>

  </tr>

  <tr>

   <td width="130"><strong>0xFFFA </strong></td>

   <td width="310">The sector is <strong>free</strong>.</td>

  </tr>

  <tr>

   <td width="130"><strong>0xFFF7 </strong></td>

   <td width="310">The sector is <strong>bad </strong>(i.e. not working, don’t store any content here).</td>

  </tr>

  <tr>

   <td width="130"><strong>0xFFFF </strong></td>

   <td width="310">The sector is the <strong>END</strong> of a linked sector chain.</td>

  </tr>

  <tr>

   <td width="130"><strong>0x0000 to</strong><strong> 0x007F </strong></td>

   <td width="310">The sector leads to the indicated sector as part of a linked sector chain.</td>

  </tr>

 </tbody>

</table>




Here’s a sample FAT printout:




From the FAT printout above, we can see that the data sector 0x0000 is free; the data sectors 0x004aà0x004bà0x004c (end) is <strong>part </strong>of a linked sector chain, etc.

<strong> </strong>

<strong> </strong>

<h2>1.2 Directory (Folder) and File under USFAT</h2>

<strong> </strong>

Under USFAT, directory and file both use the file data sectors to store information. For a directory, the data sector stores <strong>directory entries</strong>, which contains information about <strong>files under </strong>that directory. For a file, the data sector stores the <strong>actual file content</strong>.




For simplicity, the USFAT media provided in this lab has the following limitations:

<ul>

 <li>There is only one directory, the <strong>Root Directory</strong>. It is located at <strong>data sector 5</strong>.</li>

 <li>Directory uses <strong>only 1 </strong>sector for its directory entries, which place an upper limit on the number of files it can store.</li>

 <li>Your code only need to work with these limitations in place.</li>

 <li>[Note: These limits are imposed to simplify the exercises, the design of the USFAT is much more general / flexible.]</li>

</ul>




Each of the directory entry in a directory’s data sector occupies <strong>32 bytes</strong> and has the following layout:




<h2>      Offset     0       …     10      11      12      …     25      26      27      28      …     31</h2>

<table width="455">

 <tbody>

  <tr>

   <td width="113"><strong>Name </strong></td>

   <td width="41"><strong>Attr </strong></td>

   <td width="113"><strong>&lt;not used&gt; </strong></td>

   <td width="75"><strong>Start Sector </strong></td>

   <td width="113"><strong>File Size </strong></td>

  </tr>

 </tbody>

</table>

Usage




The name uses the old “8+3” format, where the file name is 8 characters long and the extension takes up 3 characters, e.g., a file with name “sample.cc” is stored as:




0          1          2          3          4          5          6          7          8          9         10

<table width="503">

 <tbody>

  <tr>

   <td width="46">ꟷ</td>

   <td width="46">ꟷ</td>

   <td width="46"><strong>S </strong></td>

   <td width="46"><strong>a </strong></td>

   <td width="46"><strong>m </strong></td>

   <td width="46"><strong>p </strong></td>

   <td width="46"><strong>l </strong></td>

   <td width="46"><strong>e </strong></td>

   <td width="46"><strong>c </strong></td>

   <td width="46"><strong>c </strong></td>

   <td width="46">ꟷ</td>

  </tr>

 </tbody>

</table>




Note that the filename is right aligned to the “.” while the extension is left aligned. The “.” itself is <strong>not stored</strong>. We use ‘ꟷ’ represents a space, i.e. ‘ ‘.




The attribute is a single byte (8 bits):




<strong>Bit         </strong>7             0

<table width="470">

 <tbody>

  <tr>

   <td width="43"> </td>

   <td width="43"><strong> </strong></td>

   <td width="43"><strong> </strong></td>

   <td width="80"><strong>Is directory? </strong></td>

   <td width="43"><strong> </strong></td>

   <td width="67"><strong>Is </strong><strong>System? </strong></td>

   <td width="69"><strong>Is </strong><strong>Hidden? </strong></td>

   <td width="81"><strong>Is </strong><strong>Readable? </strong></td>

  </tr>

 </tbody>

</table>

<h2>Usage</h2>




For our exercises, you can assume that <strong>all files have an attribute 0x01</strong>, i.e. readable, not hidden, not a system file and not a directory.




Since each directory entry is 32 bytes and the directory in USFAT can utilize only 1 sector for directory entries, this gives us 256 bytes / 32 bytes = 8 files under a directory <strong>at most</strong>.







<h2>1.3 USFAT “Media”, Library Calls and Utility Program</h2>




There are a number of “disk image” files provided for this lab, e.g. <strong><em>4files.img, empty.img</em></strong>, etc. Each of the file represents a complete USFAT file system. You can imagine they represent simulated storage media like a hard disk, etc.




A <strong>large number </strong>of library calls are provided for you to focus on “high level” file system functionalities. In the common/ directory, take a look at the <strong>USFAT.h</strong> header files which defines all important system parameters and the available library calls. Essentially, “low level” functionalities that deals with reading / write information from / to the media, e.g.  sector / FAT reading / writing, etc are available for use.




In addition, a “debug inspector” program, known as USFATI (USFAT Insepctor) is also available so that you can view the raw content on a USFAT media easily. Instructions to setup the inspector etc is given in Section 2.




<strong>          </strong>

<h1>Section 2. Exercises for USFAT</h1>




<h2>2.1 Directory structure of the skeleton code</h2>

There is one additional folder <strong><em>common/</em></strong> with the following files:




<table width="522">

 <tbody>

  <tr>

   <td width="152"><strong>Filename </strong></td>

   <td width="370"><strong>Purpose </strong></td>

  </tr>

  <tr>

   <td width="152">USFAT.h</td>

   <td width="370">USFAT header file with all key definitions and declarations.</td>

  </tr>

  <tr>

   <td width="152">USFAT_Util.c</td>

   <td width="370">Implementation of all USFAT library functions.</td>

  </tr>

  <tr>

   <td width="152">USFAT_Insepct.c</td>

   <td width="370">The debug inspector utility program. Compiles into the “USFATI” executable.</td>

  </tr>

  <tr>

   <td width="152">Various *.img</td>

   <td width="370">Backup copies of all USFAT disk images. <strong>In exercise 3, your program will modify the USFAT disk image</strong>, so if you ever need to “reset” the disk images, copy the backup over.</td>

  </tr>

  <tr>

   <td width="152">makefile</td>

   <td width="370">For compiling the USFATI debug inspector as mentioned above.reset.sh: A simple script file to copy the backup images to the exercise directories.</td>

  </tr>

 </tbody>

</table>




<table width="522">

 <tbody>

  <tr>

   <td width="522"><strong>Preparation: </strong>1.     Go into the common/ folder and type “make” to produce the USFATI executable.2.     Enable the “reset.sh” script file by “chmod 700 reset.sh”3.     Execute the “reset.sh” script file “./reset.sh”, this copy a fresh set of disk images to the exercise directories. Use this step whenever you need to reset your disk images.</td>

  </tr>

 </tbody>

</table>




<strong>           </strong>

<h2>2.2 Exercise 2</h2>

<strong>Main task</strong>: Display the file content of a file under the root directory.




The main function is already written for you. The main function will repeatedly print the directory content of the root directory (i.e. similar to a “ls”), then prompt the user for a file to display (i.e. similar to a “cat” / “less” command). Your task is to implement the function “read_file( FAT_RUNTIME* rt, char filename[])” which returns:

<ul>

 <li><strong>0</strong> if the file with filename cannot be found under the root directory.</li>

 <li><strong>1</strong> if the operation is successful.</li>

</ul>




This function attempts to locate the directory entry for the file <em>filename</em>, then read <strong>all </strong>data sectors of this print and print them to the screen. <strong>Note: use the </strong><em>print_as_text() </em><strong>function when you need to print out the content of a file data sector. </strong>This ensure your output format is exactly the same as ours to facilitate checking.




Several key criteria:

<ul>

 <li>Entire content of the file should be shown (duh!). This requires you to follow the “<em>linked sector chain</em>” by traversing in the FAT……</li>

 <li>Note that the last sector may not be full! You need to print out <strong>only the valid content</strong>. (hint: use file size…..).</li>

 <li>You are allowed to define as many helper functions as you need.</li>

 <li>You can add / change the parameter(s) of the <em>read_file</em>() function if needed.</li>

 <li>The main function should not be changed except the function call to <em>read_file</em>() can be modified with new parameters if you change them.</li>

</ul>




Sample Output (using <strong><u>4files.img</u>, </strong> user input in <strong>bold</strong>, file content in <strong>red</strong>):

<table width="491">

 <tbody>

  <tr>

   <td width="491">  Filename     Attr     Start     Size ————————————–      fat.txt 01 &lt;file&gt; [0x0067]   1563  mystery.abc 01 &lt;file&gt; [0x003a]   1092    hello.c   01 &lt;file&gt; [0x007e]     74 rain.txt 01 &lt;file&gt; [0x0042]  12194Read File (“DONE” to quit) &gt; <strong>hello.c</strong><strong>#include &lt;stdio.h&gt; </strong><strong> </strong><strong>int main()                             </strong>Note that only 74<strong>{     </strong>bytes of “hello.c” <strong>        printf(“Hello World!
”); </strong>are valid out of 256 <strong>  </strong>bytes in the sector. <strong>        return 0; </strong><strong>} </strong> Filename     Attr     Start     Size</td>

  </tr>

  <tr>

   <td width="491">————————————–      fat.txt 01 &lt;file&gt; [0x0067]   1563  mystery.abc 01 &lt;file&gt; [0x003a]   1092    hello.c   01 &lt;file&gt; [0x007e]     74rain.txt 01 &lt;file&gt; [0x0042]  12194There is noRead File (“DONE” to quit) &gt; <strong>hi.txt</strong><strong>“hi.txt” not found!                    </strong>“hi.txt” in the rootFilename     Attr     Start     Size ————————————–      fat.txt 01 &lt;file&gt; [0x0067]   1563  mystery.abc 01 &lt;file&gt; [0x003a]   1092    hello.c   01 &lt;file&gt; [0x007e]     74 rain.txt 01 &lt;file&gt; [0x0042]  12194Read File (“DONE” to quit) &gt; <strong>mystery.abc</strong><strong>#include &lt;stdio.h&gt; </strong><strong>#include &lt;stdlib.h&gt; </strong><strong>#include &lt;string.h&gt; </strong>…… &lt;Some file content omitted to save space&gt; ……<strong>        } </strong><strong> </strong>The entire<strong>        fclose(fat_rt.media_f); </strong><strong><sub>          </sub></strong>“mystery.abc”  is <strong>        return 0; </strong>printed.<strong>} </strong> Filename     Attr     Start     Size ————————————–      fat.txt 01 &lt;file&gt; [0x0067]   1563  mystery.abc 01 &lt;file&gt; [0x003a]   1092    hello.c   01 &lt;file&gt; [0x007e]     74 rain.txt 01 &lt;file&gt; [0x0042]  12194Read File (“DONE” to quit) &gt; <strong>DONE</strong></td>

  </tr>

 </tbody>

</table>




To aid your checking, the original files “fat.txt”, “mystery.abc”, “hello.c” and “rain.txt” (as well as a few other text files) are included in the exercise folder.







<strong>           </strong>

<h2>2.2 Exercise 3</h2>

<strong>Main task</strong>: Import a normal file into the USFAT file system.




Similar to exercise 2, the main function is already coded for you. You only need to provide the implementation of the <em>import_file</em>()  function. This function returns:

o <strong>-1:</strong> if there is any error. Full error lists is given below. OR o <strong>Non-negative value:</strong> Actual number of bytes copied over.




To facilitate explanation, let us assume we make the following call using the given function prototype:

<em>import_file</em>( &amp;runtime, “example.txt”,  25 );

The function should perform the following checks:

<ul>

 <li>Ensure the normal file “txt” can be opened. You can assume the filename given by user follows the “8+3” filename restriction.</li>

 <li>Ensure the root directory of the USFAT media <strong>does not</strong> have another file with the same filename as “txt” as filename should be unique under a directory.</li>

 <li>Ensure the root directory is <strong>not full</strong>, i.e. has less than 8 files currently.</li>

 <li>If any of the above fails, the function returns “-1”.</li>

</ul>




<ul>

 <li>Once checks are all cleared, the function will now attempt to copy the “txt” into the data sectors.</li>

 <li>The function will <strong>try </strong>to use the first sector as specified (data sector 25) in this example. If the sector is free, copying can start there. Otherwise, you should check subsequent data sectors (e.g. 26, 27, 28 …..) and wraps around if needed. If there are no free data sector, the function terminates and return -1.</li>

 <li>Once copying starts, data sector chain needs to be constructed if you need more than one data sector. The logic for getting the next sector is the same: look for the free data sector in the subsequent indices and wrap around if needed. <strong>Remember to modify the FAT entries accordingly as you move along</strong>.</li>

 <li>Copy stops when i) the input file e.g. “txt” has been copied fully OR ii) there are no more free data sectors. <strong>Remember to “terminate” your sector chain by setting the END flag in the FAT.</strong></li>

 <li>The function will also add a new directory entry with the right information into the root directory.</li>

 <li><strong>Don’t forget to flush the FAT into the actual USFAT media.</strong></li>

 <li>Finally, the function returns the total number of bytes copied to the caller.</li>

</ul>




<ul>

 <li>Note that this exercise <strong>changes the disk image</strong> Use the “reset.sh” to restore the images if needed.</li>

</ul>




Hints and Tips:

<ul>

 <li>Browse the library calls. You have <strong>many </strong>helpful functions there to keep your pain in check….</li>

 <li>Define useful helper functions to reduce code spaghetti…</li>

 <li>The sample solution is about 120+lines (with newlines, debug code, comments included).</li>

 <li>You can use your c to check whether the files are imported properly.</li>

 <li>Use the utility program USFATI to monitor the changes of sectors.</li>

</ul>







<strong>Sample Output</strong> (using <strong><u>empty.img</u>, </strong> user input in <strong>bold</strong>, notable key info in <strong>red</strong>).




The <strong><u>empty.img</u></strong> is an empty USFAT media with <strong>only the root directory taking up data sector 5</strong> at the beginning.




<table width="528">

 <tbody>

  <tr>

   <td width="528">  Filename     Attr     Start     Size ————————————–Import File (“DONE” to quit) &gt; <strong>hi.c</strong>” exist.Start sector (in Hex) &gt; 0x<strong>0</strong>                     <strong><sup>Hi.c</sup></strong><sup>” doesn’t </sup><strong>Import “hi.c” to [0x0000] Data Sector…FAILED! </strong>Filename     Attr     Start     Size ————————————– Import File (“DONE” to quit) &gt; <strong>hello.c</strong>Start sector (in Hex) &gt; 0x<strong>5</strong><strong>Import “hello.c” to [0x0005] Data Sector…Written 74 bytes.</strong>Filename     Attr     Start     SizeData sector 5 is occupied,————————————– so next available sectorhello.c   01 &lt;file&gt; [<strong>0x0006</strong>]     74(i.e. 6) is used. Import File (“DONE” to quit) &gt; <strong>hello.c </strong>Start sector (in Hex) &gt; 0x<strong>4A</strong><strong>Import “hello.c” to [0x004a] Data Sector…FAILED! </strong>Filename     Attr     Start     Size————————————–       <sup>There is already a </sup>” import failed.hello.c   01 &lt;file&gt; [0x0006]     74         <strong><sup>hello.c</sup></strong><sup>” </sup><sup>è</sup> Import File (“DONE” to quit) &gt; <strong>fat.txt</strong>Start sector (in Hex) &gt; 0x<strong>2f</strong><strong>Import “fat.txt” to [0x002f] Data Sector…Written 1563 bytes. </strong>Filename     Attr     Start     Size ————————————–    hello.c   01 &lt;file&gt; [0x0006]     74      fat.txt 01 &lt;file&gt; [0x002f]   1563 Import File (“DONE” to quit) &gt; <strong>mystery.abc</strong>Start sector (in Hex) &gt; 0x<strong>0</strong></td>

  </tr>

  <tr>

   <td width="528"><strong>Import “mystery.abc” to [0x0000] Data Sector…Written 1092 bytes. </strong>Filename     Attr     Start     SizeBoth ”————————————–         <strong><sup>fat.txt</sup></strong><sup>” and </sup>”hello.c   01 &lt;file&gt; [0x0006]     74     <strong><sup>mystery.abc</sup></strong><sup>” are </sup>fat.txt 01 &lt;file&gt; [0x002f]   <strong>1563</strong> <sup>imported fully. You can </sup> mystery.abc 01 &lt;file&gt; [0x0000]   <strong>1092</strong>   <sup>verify their file size. </sup> Import File (“DONE” to quit) &gt; <strong>alice.txt</strong>Start sector (in Hex) &gt; 0x<strong>4A</strong><strong>Import “alice.txt” to [0x004a] Data Sector…Written 30720 bytes. </strong>Filename     Attr     Start     Size————————————– <sub>The USFAT disk is </sub>   hello.c   01 &lt;file&gt; [0x0006]     74 almost full at this point      fat.txt 01 &lt;file&gt; [0x002f]   1563 and can only stores  mystery.abc 01 &lt;file&gt; [0x0000]   1092 29,184 bytes out of the    alice.txt 01 &lt;file&gt; [0x004a]  <strong>29184</strong> full 177,428 bytes for“<strong>alice.txt</strong>”Import File (“DONE” to quit) &gt; <strong>DONE</strong></td>

  </tr>

 </tbody>

</table>

<strong> </strong>

The FAT table (use USFATI to inspect) should looks like the following afterwards:

Several notable observations:

<ul>

 <li>“<strong>txt</strong>” is in sector 6, where the FAT entry is indicated with the END flag as it occupies only 1 sector.</li>

 <li>“<strong>abc</strong>” starts from sector 0, follow the linked sector list to understand the requirement better (use adjacent if possible, otherwise search forward for free sector).</li>

</ul>




<strong> </strong>

<strong>2.3 And beyond…. </strong>

<strong> </strong>

As exercise 3 is “a bit” challenging, I have decided to drop the bonus questions. L However, I hope you have the curiosity (and time) to explore further. Several things you can try (in increasing insanity order):

<ol>

 <li>Expand the directory to use multiple sectors. This removes the 8 files per directory limitation. Your code in ex2 can help.</li>

 <li>Implement subdirectory. (Rather straightforward actually).</li>

</ol>

With (2), extend ex2 and ex3 to support subdirectory, i.e. read file with full path “<strong>/dir1/dir2/example.txt</strong>”, import file for deeper directory