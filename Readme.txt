OOMMFTools
http://web.mit.edu/daigohji/projects/OOMMFTools/
https://github.com/deparkes/OOMMFTools

Mark Mascaro
doublemark@mit.edu

Duncan Parkes
ppxdep@nottingham.ac.uk

****************
* Introduction *
****************

OOMMFTools is a set of utilities designed to assist OOMMF postprocessing.
It includes the following subcomponents:

GUI:
OOMMFDecode - Converts OOMMF vector files into numpy arrays and/or MATLAB data files
OOMMFConvert- Converts OOMMF vector files into bitmaps and movies
ODTChomp    - Converts ODT files or subsets thereof into normally-delimited text files

Command line:
odt2dat.py - Converts ODT files or subsets thereof into normally-delimited text files
avf2bmp.py - Converts OOMMF vector files into bitmaps
bmp2avi.py - Convert bitmaps into movies
omf2mat.py -  Converts OOMMF vector files into numpy arrays and/or MATLAB data files

The Py2EXE-wrapped Windows version is designed to be portable and free of external dependencies.
Everything necessary is included, including a binary of FFmpeg for OOMMFConvert. The script
otherwise has the following dependencies:

- Python (2.6 preferred) [All]
- wxPython (Unicode) [All]
- numpy [OOMMFDecode]
- scipy [OOMMFDecode]
- FFmpeg [OOMMFConvert]
- OOMMF [OOMMFConvert]
- Tcl/Tk [OOMMFConvert]

Current features...
Replicate OOMMFTools, but with command line versions. Good for 
Planned features...

Current issues...
Limited use of imagemagick, could be better automated
Make it so that the different programs/components work in more or less the 
same way.
Requires editing of the individual files

Requirements:
- Python (2.7) [All]
- wxPython (Unicode) [GUI Programs]
- ...
- numpy [OOMMFDecode]
- scipy [OOMMFDecode]
- FFmpeg [OOMMFConvert, bmp2avi]
- OOMMF [OOMMFConvert, avf2bmp]
- Tcl/Tk [OOMMFConvert, avf2bmp]
Imagemagick (for making videos, converting and labelling images)

**********************
* Command Line Tools *
**********************

Installation/setup
1. Download OOMMFTools from github
2. Add OOMMFTools directory to system path
3. a) Set correct path to OOMMF in avf2bmp.py and oommf.bat
   b) Set correct path to tcl in avf2bmp.py and oommf.bat
   (These scripts need to call oommf in order to work, so need to know where
	oommf can be found and how to run it (with tcl).)

Usage:
The general way to operate is to run the scripts from the target directory.
Possible ways of doing this include:
	- run 'cmd' and navigate to the target directory
	- find the target directory in windows explorer, right clight and 
		"open command window here"
	- open the target directory, click in the address bar and type 'cmd'.
From this command line run the relevant command.

See below for details of each component.

***************
* OOMMFDecode *
***************


OOMMFDecode batch-processes vector files (omf, ovf, oef, ohf) into numpy arrays.
These can then be pickled, for python users, or saved into MATLAB data files,
for MATLAB users.


1. UTILIZATION


The GUI is very simple. Two checkboxes are offered - for making numpy data and for making MATLAB
data. Simply check the options for the types of data you want to save and drag one or more OMF files
onto the application. The data format (text, binary 4, binary 8) is automatically detected. When the
import is finished, you'll be prompted to save the data for each format you selected.

Dropping a batch of OMF files is primarily designed to automate aggregation of data from various
states of a single simulation. The program assumes all files you drop at a time are "similar" -
that is, they have the same grid size and other header data. Please be mindful of this constraint,
and if you wish to convert files from several disparate simulations do so in different drop
operations.

Some files (such as energy densities) contain only a single value, not a vector. In OOMMF, these
outputs are still vector files. The relevant quantity is stored in the X coordinate.


A. MATLAB Data


The output .MAT file contains several variables.

First, a 3-element vector GridSize, which contains the X, Y, and Z span of each cell. This is
imported from the OMF file headers, specifically from the first file on the list of dropped
files (again, it is assumed all files in one batch are similar).

Second, a vector SimTime, which contains the simulation time associated with each OMF file,
if any. The input files are sorted based on SimTime if and only if times are available for all
files.

Third and fourth, the vectors Iteration and Stage, which contain the iteration and
stage number associated with each OMF file, if any. This is the total iteration number,
not the stage iteration number.

The final variable is a 5-D matrix OOMMFData, which can be understood as follows.

  OOMMFData(A,B,C,D,E):

A. The index of the OMF file, in simulation time order. If simtime data is not available,
   the files are in the same order as they were dropped on the program, which is generally
   the operating system sort order.

B. X coordinate in first-octant coordinates.

C. Y coordinate in first-octant coordinates.

D. Z coordinate in first-octant coordinates.

E. (1,2,3) are the x,y,z components of the vector.

Note that everything is indexed in the OOMMF (first-octant) coordinate system, but
row-column matrix notation is fourth-quadrant. Depending on what you're trying to do,
it may be necessary to transform the data accordingly. When in doubt, remember that
indices into matrices generated by this program match OOMMF's own numbers!


B. numpy Data


The output .PNP file can be unpickled into a tuple containing two items: first, a 5-D
matrix as documented above; second, a dictionary of header values extracted from the OMF
file header data. This latter data is useful for looking up the scale of each cell or the
simulation time of a particular file.


2. Known Bugs


No bugs are currently outstanding.

If the About dialog does not work under Ubuntu, your version of wxPython is outdated. You
will want to get a copy and build it - the prebuilt package for Ubuntu is often too old.


****************
* OOMMFConvert *
****************

OOMMFConvert is meant to ease converting OOMMF simulation results into bitmaps and movies,
especially for Windows users for whom the console is more unfamiliar or difficult. It uses the existing
avf2ppm capability of OOMMF, along with the open-source utility FFmpeg for movie conversion.


1. Utilization


The GUI is divided into five sections as follows.


A. Path to OOMMF - configures Tcl shell calls


The left dropbox contains the call necessary to involve tcl. This defaults to and should
almost always be left on the value "tclsh".  However, some Windows installations of
ActiveTcl/Tk use other commands, such as tclsh85  (which is provided as a dropdown option).
If necessary, enter a new value here.

The static text field to the right shows the path to the oommf.tcl file in your OOMMF
installation. This is the file that will be called to invoke avf2ppm. You can use the
"Load OOMMF" button to locate it, or simply drag the oommf.tcl file anywhere over the program
window. This path is recorded in the file oommf.path in the program directory,
and this configuration step is only necessary once.


B. Configuration File - shows mmDisp configuration file


This section shows which mmDisp configuration file is going to be used in avf2ppm. You can save
an mmDisp configuration file from an mmDisp view with the "Write config..." option in the File menu.
You can select a configuration file with the "Load Config" button, or by dragging a file with the
extension .conf, .config, or .cnf anywhere within the program window. The text shows the absolute
path to the currently selected config file. This value is _NOT_ saved between sessions, as it is
typically sim-dependent and unique for different groups of OMF files. To clarify, it is saved
between file drops, so you can easily use the same configuration file to convert multiple batches
of OMF files without closing the program.

For the magnetization, the maximum value of the vector field is fixed (Ms). For other kinds of
fields, such as demag, the maximum value of the vector field may fluctuate from file to file.
This could result in clipping unless you happened to use the largest-values o?f file to produce
your config file. If you're using a field where the maximum value fluctuates significant, check
"generate vector field maxima". If this is checked, the program will decode the files to be 
converted and find the maximum vector magnitude among all files. This can add significantly to 
the runtime, but makes it easy to generate uniformly-scaled pictures for these sorts of fields
without clipping.


C. Images - configures bitmap output


This section controls the bitmap file output. If "Make Bitmaps" is unchecked, no images will be
created. It defaults on. "Image Magnify%" overrides the parameters in the mmDisp configuration
file to increase the output bitmap size. It employs a temporary copy of the configuration file,
and the original is not overwritten. The value is in percent of the OOMMF default size, usually
around 640x480.


D. Movies - configures movie output


This section controls the output of a movie file based on a collection of input OMF files.
A movie will be made from each batch of OMF files if "Make Movies" is checked, but it defaults off.

The leftmost configuration on the bottom row is the FPS of the output movie. You may also consider it
"simulation files per second." Since FFmpeg uses an awkward fixed-framerate form, frames will be
duplicated to fill in time when the FPS is reduced. These temporary files are cleaned up automatically.
This value can be between 1 and 25 FPS, and defaults to 25.

The middle combo box allows the choice of encoding from a number of codecs built into FFmpeg.
The default is HuffYUV, which I find gives the best-quality output files for the common red-white-blue
color scheme. MPEG4 has some difficulty with the black-on-blue arrows and gives a visibly worse encoding.
The HuffYUV decoder does not come standard on most systems (Windows) but is freely available, and
I highly suggest using this codec.

The rightmost control is the movie magnification, which functions similarly to the image magnification.
However, many codecs are unstable for large input image sizes (above around 140% over the OOMMF default
size) and may fail silently. This is a bug in the encoder and cannot be worked around. Therefore,
increasing the movie magnification is done at the user's risk. The default codec HuffYUV is quite robust,
and readily supports use of the movie magnification control.


E. Drop OOMMF Files Here! - a friendly reminder


Drag and drop configuration and vector field files here - or anywhere in the program window, but this
section is a friendly reminder. Multiple files can be dropped at a time. OMF files are converted
to bitmaps using the oommf command line utility avf2ppm and the supplied mmDisp configuration file.
If a movie is being made, the batch of simultaneously dropped OMF files is converted to a single movie
with the frames in filename order.


************
* ODTChomp *
************


ODTChomp takes in ODT data tables, simplifies the name scheme to the extent it it possible,
and outputs the desired columns into a text file with a given delimeter. The behavior is rather
distict from odtcols, which is better at fixed-width rather than fixed-delimitation formatting.


1. Utilization


Begin by loading an ODT file. Drag-and-drop also works. The leftmost panel shows the columns found
in the ODT file, using a simplified name scheme that uses the minimum number of uniquely identifying
descriptors. The result is generally very human-readable unless the simulation file is extremely
complex. The right panel shows which data fields will appear in the output file.

Double-clicking an entry in the left panel, or clicking an entry and then the "-->" button, will
mark it for export. Double-clicking an entry in the right panel, or clicking an entry and then
the "<--" button, will remove it from the export. The subsequent set of buttons can be used to add
or remove all fields from the output file. The "Move Up" and "Move Down" buttons affect
the item highlighted in the right panel, and are used to reorder the output columns.

The radio buttons between the two panels choose the delimiting symbol for the values in the output.
For example, if you want the comma-separated values commonly used by Excel, choose comma. If you
choose space as the delimited, spaces that appear in column names will be replaced by underscores in
the output.

Finally, the export button writes the selected columns in the selected order to a plaintext file.

2. Batch Mode

The checkbox just above the Export button enables Batch Mode. Batch Mode is designed to extract
the same data fields from a large group of ODT files using drag-and-drop. Batch Mode can only be
used once one file has already been loaded and export data has been selected. Even if Batch Mode
is checked, if no file has previously been loaded, the first file drop will give non-batch behavior.

Once a file has been loaded and data fields have been chosen, any files dragged and dropped onto ODTChomp
with Batch Mode enabled will have the specified fields extracted. The output will be placed in the same
folder as the dropped file, with the same filename and the ".txt" extension. Currently, dropping directories
is not supported and they will not be recursed.

******************************
* GNU General Public License *
******************************

                    GNU GENERAL PUBLIC LICENSE
                       Version 2, June 1991

 Copyright (C) 1989, 1991 Free Software Foundation, Inc.,
 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
 Everyone is permitted to copy and distribute verbatim copies
 of this license document, but changing it is not allowed.

                            Preamble

  The licenses for most software are designed to take away your
freedom to share and change it.  By contrast, the GNU General Public
License is intended to guarantee your freedom to share and change free
software--to make sure the software is free for all its users.  This
General Public License applies to most of the Free Software
Foundation's software and to any other program whose authors commit to
using it.  (Some other Free Software Foundation software is covered by
the GNU Lesser General Public License instead.)  You can apply it to
your programs, too.

  When we speak of free software, we are referring to freedom, not
price.  Our General Public Licenses are designed to make sure that you
have the freedom to distribute copies of free software (and charge for
this service if you wish), that you receive source code or can get it
if you want it, that you can change the software or use pieces of it
in new free programs; and that you know you can do these things.

  To protect your rights, we need to make restrictions that forbid
anyone to deny you these rights or to ask you to surrender the rights.
These restrictions translate to certain responsibilities for you if you
distribute copies of the software, or if you modify it.

  For example, if you distribute copies of such a program, whether
gratis or for a fee, you must give the recipients all the rights that
you have.  You must make sure that they, too, receive or can get the
source code.  And you must show them these terms so they know their
rights.

  We protect your rights with two steps: (1) copyright the software, and
(2) offer you this license which gives you legal permission to copy,
distribute and/or modify the software.

  Also, for each author's protection and ours, we want to make certain
that everyone understands that there is no warranty for this free
software.  If the software is modified by someone else and passed on, we
want its recipients to know that what they have is not the original, so
that any problems introduced by others will not reflect on the original
authors' reputations.

  Finally, any free program is threatened constantly by software
patents.  We wish to avoid the danger that redistributors of a free
program will individually obtain patent licenses, in effect making the
program proprietary.  To prevent this, we have made it clear that any
patent must be licensed for everyone's free use or not licensed at all.

  The precise terms and conditions for copying, distribution and
modification follow.

                    GNU GENERAL PUBLIC LICENSE
   TERMS AND CONDITIONS FOR COPYING, DISTRIBUTION AND MODIFICATION

  0. This License applies to any program or other work which contains
a notice placed by the copyright holder saying it may be distributed
under the terms of this General Public License.  The "Program", below,
refers to any such program or work, and a "work based on the Program"
means either the Program or any derivative work under copyright law:
that is to say, a work containing the Program or a portion of it,
either verbatim or with modifications and/or translated into another
language.  (Hereinafter, translation is included without limitation in
the term "modification".)  Each licensee is addressed as "you".

Activities other than copying, distribution and modification are not
covered by this License; they are outside its scope.  The act of
running the Program is not restricted, and the output from the Program
is covered only if its contents constitute a work based on the
Program (independent of having been made by running the Program).
Whether that is true depends on what the Program does.

  1. You may copy and distribute verbatim copies of the Program's
source code as you receive it, in any medium, provided that you
conspicuously and appropriately publish on each copy an appropriate
copyright notice and disclaimer of warranty; keep intact all the
notices that refer to this License and to the absence of any warranty;
and give any other recipients of the Program a copy of this License
along with the Program.

You may charge a fee for the physical act of transferring a copy, and
you may at your option offer warranty protection in exchange for a fee.

  2. You may modify your copy or copies of the Program or any portion
of it, thus forming a work based on the Program, and copy and
distribute such modifications or work under the terms of Section 1
above, provided that you also meet all of these conditions:

    a) You must cause the modified files to carry prominent notices
    stating that you changed the files and the date of any change.

    b) You must cause any work that you distribute or publish, that in
    whole or in part contains or is derived from the Program or any
    part thereof, to be licensed as a whole at no charge to all third
    parties under the terms of this License.

    c) If the modified program normally reads commands interactively
    when run, you must cause it, when started running for such
    interactive use in the most ordinary way, to print or display an
    announcement including an appropriate copyright notice and a
    notice that there is no warranty (or else, saying that you provide
    a warranty) and that users may redistribute the program under
    these conditions, and telling the user how to view a copy of this
    License.  (Exception: if the Program itself is interactive but
    does not normally print such an announcement, your work based on
    the Program is not required to print an announcement.)

These requirements apply to the modified work as a whole.  If
identifiable sections of that work are not derived from the Program,
and can be reasonably considered independent and separate works in
themselves, then this License, and its terms, do not apply to those
sections when you distribute them as separate works.  But when you
distribute the same sections as part of a whole which is a work based
on the Program, the distribution of the whole must be on the terms of
this License, whose permissions for other licensees extend to the
entire whole, and thus to each and every part regardless of who wrote it.

Thus, it is not the intent of this section to claim rights or contest
your rights to work written entirely by you; rather, the intent is to
exercise the right to control the distribution of derivative or
collective works based on the Program.

In addition, mere aggregation of another work not based on the Program
with the Program (or with a work based on the Program) on a volume of
a storage or distribution medium does not bring the other work under
the scope of this License.

  3. You may copy and distribute the Program (or a work based on it,
under Section 2) in object code or executable form under the terms of
Sections 1 and 2 above provided that you also do one of the following:

    a) Accompany it with the complete corresponding machine-readable
    source code, which must be distributed under the terms of Sections
    1 and 2 above on a medium customarily used for software interchange; or,

    b) Accompany it with a written offer, valid for at least three
    years, to give any third party, for a charge no more than your
    cost of physically performing source distribution, a complete
    machine-readable copy of the corresponding source code, to be
    distributed under the terms of Sections 1 and 2 above on a medium
    customarily used for software interchange; or,

    c) Accompany it with the information you received as to the offer
    to distribute corresponding source code.  (This alternative is
    allowed only for noncommercial distribution and only if you
    received the program in object code or executable form with such
    an offer, in accord with Subsection b above.)

The source code for a work means the preferred form of the work for
making modifications to it.  For an executable work, complete source
code means all the source code for all modules it contains, plus any
associated interface definition files, plus the scripts used to
control compilation and installation of the executable.  However, as a
special exception, the source code distributed need not include
anything that is normally distributed (in either source or binary
form) with the major components (compiler, kernel, and so on) of the
operating system on which the executable runs, unless that component
itself accompanies the executable.

If distribution of executable or object code is made by offering
access to copy from a designated place, then offering equivalent
access to copy the source code from the same place counts as
distribution of the source code, even though third parties are not
compelled to copy the source along with the object code.

  4. You may not copy, modify, sublicense, or distribute the Program
except as expressly provided under this License.  Any attempt
otherwise to copy, modify, sublicense or distribute the Program is
void, and will automatically terminate your rights under this License.
However, parties who have received copies, or rights, from you under
this License will not have their licenses terminated so long as such
parties remain in full compliance.

  5. You are not required to accept this License, since you have not
signed it.  However, nothing else grants you permission to modify or
distribute the Program or its derivative works.  These actions are
prohibited by law if you do not accept this License.  Therefore, by
modifying or distributing the Program (or any work based on the
Program), you indicate your acceptance of this License to do so, and
all its terms and conditions for copying, distributing or modifying
the Program or works based on it.

  6. Each time you redistribute the Program (or any work based on the
Program), the recipient automatically receives a license from the
original licensor to copy, distribute or modify the Program subject to
these terms and conditions.  You may not impose any further
restrictions on the recipients' exercise of the rights granted herein.
You are not responsible for enforcing compliance by third parties to
this License.

  7. If, as a consequence of a court judgment or allegation of patent
infringement or for any other reason (not limited to patent issues),
conditions are imposed on you (whether by court order, agreement or
otherwise) that contradict the conditions of this License, they do not
excuse you from the conditions of this License.  If you cannot
distribute so as to satisfy simultaneously your obligations under this
License and any other pertinent obligations, then as a consequence you
may not distribute the Program at all.  For example, if a patent
license would not permit royalty-free redistribution of the Program by
all those who receive copies directly or indirectly through you, then
the only way you could satisfy both it and this License would be to
refrain entirely from distribution of the Program.

If any portion of this section is held invalid or unenforceable under
any particular circumstance, the balance of the section is intended to
apply and the section as a whole is intended to apply in other
circumstances.

It is not the purpose of this section to induce you to infringe any
patents or other property right claims or to contest validity of any
such claims; this section has the sole purpose of protecting the
integrity of the free software distribution system, which is
implemented by public license practices.  Many people have made
generous contributions to the wide range of software distributed
through that system in reliance on consistent application of that
system; it is up to the author/donor to decide if he or she is willing
to distribute software through any other system and a licensee cannot
impose that choice.

This section is intended to make thoroughly clear what is believed to
be a consequence of the rest of this License.

  8. If the distribution and/or use of the Program is restricted in
certain countries either by patents or by copyrighted interfaces, the
original copyright holder who places the Program under this License
may add an explicit geographical distribution limitation excluding
those countries, so that distribution is permitted only in or among
countries not thus excluded.  In such case, this License incorporates
the limitation as if written in the body of this License.

  9. The Free Software Foundation may publish revised and/or new versions
of the General Public License from time to time.  Such new versions will
be similar in spirit to the present version, but may differ in detail to
address new problems or concerns.

Each version is given a distinguishing version number.  If the Program
specifies a version number of this License which applies to it and "any
later version", you have the option of following the terms and conditions
either of that version or of any later version published by the Free
Software Foundation.  If the Program does not specify a version number of
this License, you may choose any version ever published by the Free Software
Foundation.

  10. If you wish to incorporate parts of the Program into other free
programs whose distribution conditions are different, write to the author
to ask for permission.  For software which is copyrighted by the Free
Software Foundation, write to the Free Software Foundation; we sometimes
make exceptions for this.  Our decision will be guided by the two goals
of preserving the free status of all derivatives of our free software and
of promoting the sharing and reuse of software generally.

                            NO WARRANTY

  11. BECAUSE THE PROGRAM IS LICENSED FREE OF CHARGE, THERE IS NO WARRANTY
FOR THE PROGRAM, TO THE EXTENT PERMITTED BY APPLICABLE LAW.  EXCEPT WHEN
OTHERWISE STATED IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES
PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED
OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.  THE ENTIRE RISK AS
TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU.  SHOULD THE
PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING,
REPAIR OR CORRECTION.

  12. IN NO EVENT UNLESS REQUIRED BY APPLICABLE LAW OR AGREED TO IN WRITING
WILL ANY COPYRIGHT HOLDER, OR ANY OTHER PARTY WHO MAY MODIFY AND/OR
REDISTRIBUTE THE PROGRAM AS PERMITTED ABOVE, BE LIABLE TO YOU FOR DAMAGES,
INCLUDING ANY GENERAL, SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES ARISING
OUT OF THE USE OR INABILITY TO USE THE PROGRAM (INCLUDING BUT NOT LIMITED
TO LOSS OF DATA OR DATA BEING RENDERED INACCURATE OR LOSSES SUSTAINED BY
YOU OR THIRD PARTIES OR A FAILURE OF THE PROGRAM TO OPERATE WITH ANY OTHER
PROGRAMS), EVEN IF SUCH HOLDER OR OTHER PARTY HAS BEEN ADVISED OF THE
POSSIBILITY OF SUCH DAMAGES.


