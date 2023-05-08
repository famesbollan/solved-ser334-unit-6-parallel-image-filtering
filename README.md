Download Link: https://assignmentchef.com/product/solved-ser334-unit-6-parallel-image-filtering
<br>
<h1>Parallel Image Filtering</h1>
Summary: In this homework, you will be implementing a threaded program that loads an image le, applies a lter to it, and saves it back to disk. You will also learn how to read and write real-world binary le formats, speci cally BMP.
<h2>1 Background</h2>
Figure 1: Application of blur algorithm to Wonderbread’s portrait. (Image from Dr. Filley’s TMC110 slide deck.)

In this assignment you will write a program that applies a lter to an image. The image will be loaded from a local BMP le, speci ed by the user, and then transformed using a box blur image lter. The resulting le will be saved back to BMP le that can be viewed on the host system. It is highly <u>highly highly </u>suggested that you develop a non-multithreaded version of the lter algorithm and load/save routines before beginning to thread it.

This document is separated into four sections: Background, Requirements, Box Blur, Loading Binary Files, Include Files, and Submission. You have almost nished reading the Background section already. In Requirements, we will discuss what is expected of you in this homework. In Box Blur, we discuss the idea of the blur lter. In Loading Binary Files, we will discuss the BMP le format and how to interact with binary les. In Lastly, Submission discusses how your source code should be submitted on BlackBoard.
<h2>2 Requirements [40 points]</h2>
Your program needs to implement loading a binary BMP le, applying a box blur to its pixel data (described in the next section), and saving the modi ed image back into a BMP le. If it helps, you may assume that images will be no larger than 256 by 256 pixels (see the MAXIMUM_IMAGE_SIZE constant in the base le). Assume that all BMP les are 24-bit uncompressed les<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a> (i.e., compression method is BI_RGB), which have a 14-bit bmp header, a 40-bit dib header, and a variable length pixel array. This is shown in Figure 3. Regarding threading: start by hard-coding your program to use four threads to compute the result. Later, extend it to allow an arbitrary number (create and use a constant called THREAD_COUNT). Using global memory is ne.

As a base requirement, your program must compile and run under Xubuntu (or another variant of Ubuntu) 18.04. Sample output for this program is shown on the right side of Figure 1.
<ul>
 	<li>Speci c Requirements:</li>
</ul>
Reads input lename from a command-line argument. (Your program must support be using used as: ./LastNameBoxBlur in.bmp out.bmp) [1 points]

Reads output lename from a command-line argument. [1 points]

Create data structures for storing BMP les (must include DIB header struct, and 24-bit pixel struct). [6 points]

Supports loading 24-bit BMP les. (No libraries allowed.) [6 points]

Applies box blur algorithm to each pixel in data. [10 points]

Supports saving 24-bit BMP les. (No libraries allowed.) [6 points]

Box blur algorithm is computed in parallel with pthreads. Work must be evenly distributed over a number of threads de ned by a constant called THREAD_COUNT, which must support all values great than 2 and less than the image’s smallest dimension. [10 points]
<h2>3 Box Blur</h2>
The lter we will be applying in this assignment is called a box blur, takes a specic neighborhood of pixels and processes it (this is a so-called stencil operation). A given output pixel is computed as the average of itself and each of its neighbors. We will use a neighborhood size of 3×3. In the following example, we are looking at a pixel (the 100, 0, 0 one) at the bottom of an image. There are 6 valid pixels in the neighbor, so we add them and divide by 6 to produce the output pixel for that position in the output image.
<table width="352">
<tbody>
<tr>
<td width="62">(0, 0, 0)</td>
<td width="62">(0, 0, 0)</td>
<td width="76">(0, 0, 0)</td>
<td width="76">(0, 0, 0)</td>
<td width="76">(0, 0, 200)</td>
</tr>
<tr>
<td width="62">(0, 0, 0)</td>
<td width="62">(0, 0, 0)</td>
<td width="76">(0 ,0, 0)</td>
<td width="76">(0, 0, 0)</td>
<td width="76">(0, 0, 200)</td>
</tr>
<tr>
<td width="62">(0, 0, 0)</td>
<td width="62">(0, 0, 0)</td>
<td width="76">(100, 0, 0)</td>
<td width="76">(0, 0, 200)</td>
<td width="76">(0, 0, 200)</td>
</tr>
<tr>
<td width="62"></td>
<td width="62"></td>
<td width="76"></td>
<td width="76"></td>
<td width="76"></td>
</tr>
</tbody>
</table>
→

Figure 2: Computing blur result for pixel at (2, 2) in an image.
<h2>4 Loading Binary Files</h2>
<h3>4.1 The BMP File Format</h3>
For reference, use the BMP speci cation on Wikipedia: <a href="https://en.wikipedia.org/wiki/BMP_file_format">https://en.wikipedia.org/wiki/BMP_ le_format. </a>In Figure 3, a graphical overview of a BMP le’s layout is shown. The layout is literally the meaning of the bits/bytes in the le starting at 0 and going to the end of the le. The rst (green) region shows the BMP header information in 14-bits: 2 for the signature, 4 for the le size, 2 for reserved1, 2 for reserved2, and 4 for le o set. Details on each of these (such as their data format, and contents) can be found on the Wikipedia page. For example, the area labeled Signature should contain the characters BM to con rm that the le is in BMP format. The second (blue region) shows the DIP header information in 40 bytes: 4 for header size, 4 for width, 4 for height, and so on. The last region (yellow) forms a 2D array of pixels. It stores columns from left to right, but rows are inverted to be bottom to top. Note that each row of pixels in this section is padded to be a multiple of four bytes. For example, if a line contains two pixels (24-bits or 3-bytes each), then an addition 2-bytes of blank data will be appended.

Review the following struct called BMP_Header which holds the bmp header information. (You should also consider creating structs to hold the dib header, and pixel data.) Notice that the entries in BMP_Header correspond to the pieces of data listed in the le format. In general, a chunk of 8-bits should be represented as a char, a chunk of 16-bits as a short, and 32-bits as a int. (Optionally: consider using unsigned types.)

struct BMP_Header { char signature [ 2 ] ; //ID f i e l d

Figure 3: BMP le format structure for 24-bit les without compression. Image modi ed from https://en.wikipedia.org/wiki/File:BMP leFormat.png.

int size ; // Size of the BMP f i l e short reserved1 ; // Application s p e c i f i c short reserved2 ; // Application s p e c i f i c

int offset_pixel_array ; // Offset where the pixel array can be found

};

Plan to review Example 1 on the Wikipedia page to get a feel for the contents of each of these regions. A recreated version of that image is provided as the attached test2.bmp le. A good exercise would be to view the le in a hex editor like HxD.
<h3>4.2 Loading the BMP header</h3>
Already provided for you is a base le that contains code to load the BMP le header (the rst 14 bits). Figure 4 shows the output of the base le. The that generates this output is shown below.

//sample code to read f i r s t 14 bytes of BMP f i l e format

FILE∗ f i l e = fopen (” test2 .bmp” , “rb “); struct BMP_Header header ;

Figure 4: Output of base le on test2.bmp.

//read bitmap f i l e header (14 bytes ) fread(&amp;header . signature , sizeof ( char )∗2 , 1 , f i l e ); fread(&amp;header . size , sizeof ( int ) , 1 , f i l e ); fread(&amp;header . reserved1 , sizeof ( short ) , 1 , f i l e ); fread(&amp;header . reserved2 , sizeof ( short ) , 1 , f i l e ); fread(&amp;header . offset_pixel_array , sizeof ( int ) , 1 , f i l e );

printf (” signature : %c%c
” , header . signature [0] , header . signature [ 1 ] ) ; printf (” size : %d
” , header . size ); printf (” reserved1 : %d
” , header . reserved1 ); printf (” reserved2 : %d
” , header . reserved2 );

printf (” offset_pixel_array : %d
” , header . offset_pixel_array ); fclose ( f i l e ); The key functions here are:
<ul>
 	<li></li>
</ul>
FILE ∗fopen ( const char ∗filename , const char ∗mode)

This creates a new le stream. fopen is used with the rb mode to indicate we are r eading a le in b inary mode. To read a le, use the mode wb instead of rb .
<ul>
 	<li></li>
</ul>
size_t fread ( void ∗ptr , size_t size , size_t nitems , FILE ∗stream )

For each call to fread, we give it rst a pointer to an element of struct containing the BMP header, then the number of bytes to read, the number of times to read (typically 1), and the le stream to use. Note that order of the calls to fread de ne the order in which we read data so the order must match the le layout. (There is also a function called fwrite which works in exactly the opposite manner. The rst won’t be a pointer though.)

One function not used here but which may be useful, is fseek:
<ul>
 	<li></li>
</ul>
int fseek ( FILE ∗ stream , long int offset , int origin )

The purpose of fseek is to move the reading head of the FILE object by some number of bytes. It can used to skip a number of bytes. The rst parameter to fseek is the le pointer, followed by a number of bytes to move, and an origin. For origin, SEEK_CUR is a relative repositioning while SEEK_SET is global repositioning.

The basic idea to support loading a BMP le will be to parse it by byte-byte (using fread) into a set of structs. Later, you can use fwrite to write out the contents of those structs to a le stream to save the ltered image.
<h2>5 Include Files</h2>
To complete this assignment, you may nd the following include les useful:
<ul>
 	<li>h: De nes standard IO functions.</li>
 	<li>h: De nes memory allocation functions.</li>
 	<li>h: De nes functionality for manipulating threads.</li>
</ul>
Useful functions:

∗ int pthread_attr_init(pthread_attr_t *attr) – Populates a pthreads attribute struct (pthread_attr_t) with default settings.

∗ int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg) – Creates a new thread with the id of thread, attributes speci ed by attr, and some data arg.

∗ int pthread_join(pthread_t thread, void **value_ptr) – Blocks until the speci ed pthread_t thread has terminated.

Useful types:

∗ pthread_attr_t – Contains attributes for a pthread thread. ∗ pthread_t – Contains a pthread thread id.
<ul>
 	<li>math.h: De nes additional functionality for manipulating strings. (Possibly useful for extra credit.)</li>
</ul>
Useful functions:

∗ double sqrt(double x) – Takes the square root of a number.

If you want to include any other les, please check with the instructor or TA before doing so.
<h3>5.1 Linking with External Libraries</h3>
By default, including a header le into your project only adds the code associated with that le to your project. For les like stdio.h or stdlib.h, which are self contained, that is enough. However, for other include les like pthread.h or math.h, we need to make sure that our object les (produced from our .c) is also linked with the library object le that supports that functionality. In GCC, we will do this with the -lm argument for math.h and -pthread argument for pthread.h.

If you are using the command-line to compile, or a make le, then these arguments should be appended to the end of your GCC command. For example:

gcc −c somefile . c −lm −pthread

NetBeans is a little more complicated Open your project in Netbeans. Right click the title of the project and select Properties. This will open up a new window called Project Properties. Select the Linker page from the left categories menu. At the bottom is a Additional Options line, which you can edit or open with the ellipsis button. There you can add any arguments that you need. In the sample screen shot, both the math and pthread libraries have been enabled.