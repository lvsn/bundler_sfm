Bundler User's Manual
---------------------
copyright 2008-2013 Noah Snavely (snavely@cs.cornell.edu)
  
based on the Photo Tourism work of Noah Snavely, Steven M. Seitz, 
  (University of Washington) and Richard Szeliski (Microsoft Research)

For more information, see the Bundler homepage at 
    http://www.cs.cornell.edu/~snavely/bundler/

--> Modification to use with ulavalSFM (desactivate with #define NORMALMODE) -- LERobot

Files :

* BundlerGeometry.cpp
* BundlerApp.h
* BundlerIO.cpp

The script _bundler.py_ has also been modified

--> Utilisateurs de colosse : BundlerSFM à utiliser, Makefile modifié

What is Bundler?
----------------

Bundler is a structure-from-motion system for unordered image
collections (for instance, images from the Internet). Bundler takes a
set of images, image features, and image matches as input, and
produces a 3D reconstruction of the camera and (sparse) scene geometry
as output. The system, described in [1] and [2], reconstructs the
scene incrementally, a few images at a time, using a modified version
of the Sparse Bundle Adjustment package of Lourakis and Argyros [3] as
the underlying optimization engine.

Currently, Bundler has been primarily compiled and tested under Linux
(though it may also compile in Windows under Cygwin, and a Visual
Studio solution file is also provided).

Conditions of use
-----------------

Bundler is distributed under the GNU General Public License.  For
information on commercial licensing, please contact the authors at the
contact address below.  If you use Bundler for a publication, please
cite the following paper:

  Noah Snavely, Steven M. Seitz, and Richard Szeliski. Photo Tourism:
  Exploring Photo Collections in 3D. SIGGRAPH Conf. Proc., 2006.

What's included
---------------

Included with the binary distribution is the Bundler executable
(bin/bundler), as well as a number of other utility scripts and
executables (in the bin/ directory). In addition, there are a number
of example image sets (and example results) under the examples/
directory. A version of the approximate nearest neighbors (ANN)
library of David M. Mount and Sunil Arya, customized for searching
verctors of unsigned bytes, is also included.

A utility program for converting bundle files (.out) to the input
required by Dr. Yasutaka Furukawa's PMVS multi-view stereo system
(http://www.di.ens.fr/pmvs/)
called Bundle2PMVS is also included.  Finally, this distribution
includes a program called RadialUndistort for generating undistorted
images (based on the undistortion parameters estimated by Bundler).


Before you begin
----------------

You'll first need to download the Bundler distribution from GitHub:

or visit the Bundler homepage at 

   http://phototour.cs.washington.edu/bundler 

and extract it into a directory (to be referred to as BASE_PATH).

You'll also need a feature detector components to get the system
working. Assuming you will be using SIFT features generated by David
Lowe's SIFT binary, you'll need to download that binary from

    http://www.cs.ubc.ca/~lowe/keypoints/

and copy it to BASE_PATH/bin (making sure it is called 'sift', or
'siftWin32.exe' under Windows).

The utils/bundler.py script requires that you have Python and the
Python Image Library (PIL) installed on your computer.

To make bundler, just type 'make' in the main bundler directory.
Note that if you plan to run Bundler on large problems, you may wish
to enable the use of the Ceres solver for bundle adjustment, which can
improve speed over the default SBA bundle adjuster.  To do so, edit
the file 'src/Makefile' and uncomment the line

    USE_CERES=true

Note that this assumes you have Ceres and its dependencies installed
on your system.  See the Ceres solver page at

   https://code.google.com/p/ceres-solver/

for more information.

Finally, once Bundler is compiled, copy the approximate nearest
neighbors (ANN) shared library at BASE_PATH/bin/libANN_char.so
(Linux/cygwin) or BASE_PATH/bin/ann_1.1_char.dll (Windows VS2005) to a
location in your LD_LIBRARY_PATH, or add BASE_PATH/bin to
LD_LIBRARY_PATH with a command like (in bash):

    LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/bundler/bin


Running bundler
---------------

The easiest way to start using Bundler is to either use the provided
RunBundler.sh bash script, or the included Python script (by Isaac
Lenton), utils/bundler.py.  Just execute either script in a directory
with a set of images in JPEG format, and it will automatically run all
the steps needed to run structure from motion on the images (assuming
everything goes well).  For RunBundler.sh, you may optionally provide
a configuration file as a command line argument---see RunBundler.sh
for a description of the configuration options available.  Notably,
you can choose to use Ceres when running Bundler.

To get help on using the Python script bundler.py:

  > bundler.py -h

To run bundler.py with verbose output on a single thread (this acts
similar to the older RunBundler.sh bash script):

  > bundler.py --verbose --no-parallel

bundler.py can also be imported into your own Python modules to
enable easy access to the bundler system.  Type 'help(bundler)'
at the python command prompt after importing bundler.py for more
information.

The Bundler exectutable is actually the last in a sequence of steps
that need to be run to reconstruct a scene.  bundler.py takes care
of all these steps for you, but it's useful to know what's going on.
The main initial steps are to generate features and pairwise feature
matches for the image set.  Any type of image features can be used,
but Bundler assumes the features are in the SIFT format, and so David
Lowe's SIFT detector, available at
http://www.cs.ubc.ca/~lowe/keypoints/, is probably the easiest to get
working with Bundler (bundler.py assumes that SIFT is used).  A
list of images containing estimating focal length information also
must be created.  The four steps to creating a reconstruction are
therefore:

1. Create a list of images using the function extract_focal_length.
   (this extracts focal length information, when available, from each
   image, and stores it in an image list).
2. Generate (SIFT) features for each image.
3. Match features between each pairs of images (this step can take a
   while).  The computed feature matches are stored in a file called
   'matches.init.txt'.
4. Run 'bundler' with a suitable options file.

Again, running the RunBundler.sh or bundler.py script is the easiest
way to perform these steps.  Steps 1-3 can also be invoked
individually from functions contained in the bundler.py script.

Bundler itself is typically invoked as follows:

 > bundler list.txt --options_file options.txt

The first argument is the list of images to be reconstructed.  Next,
an options file containing settings to be used for the current run is
given.  The RunBundler.sh and bundler.py scripts create an options
file that will work in many situations (and some of these options can
be controlled by the configuration file passed to RunBundler.sh).
Common options are described later in this document.  To generate only
the list of images, run:

 > bundler.py --extract-focal


Output format
-------------

Bundler produces files typically called 'bundle_*.out' (we'll call
these "bundle files").  With the default commands, Bundler outputs a
bundle file called 'bundle_<n>.out' containing the current state of
the scene after each set of images has been registered (n = the number
of currently registered cameras).  After all possible images have been
registered, Bundler outputs a final file named 'bundle.out'.  In
addition, a "ply" file containing the reconstructed cameras and points
is written after each round.  These ply files can be viewed with the
"scanalyze" mesh viewer, available at
http://graphics.stanford.edu/software/scanalyze/.

The bundle files contain the estimated scene and camera geometry have
the following format:

    # Bundle file v0.3
    <num_cameras> <num_points>   [two integers]
    <camera1>
    <camera2>
     ...
    <cameraN>
    <point1>
    <point2>
     ...
    <pointM>

Each camera entry <cameraI> contains the estimated camera intrinsics
and extrinsics, and has the form:

    <f> <k1> <k2>   [the focal length, followed by two radial distortion coeffs]
    <R>             [a 3x3 matrix representing the camera rotation]
    <t>             [a 3-vector describing the camera translation]

The cameras are specified in the order they appear in the list of
images.

Each point entry <pointI> has the form:

    <position>      [a 3-vector describing the 3D position of the point]
    <color>         [a 3-vector describing the RGB color of the point]
    <view list>     [a list of views the point is visible in]

The view list begins with the length of the list (i.e., the number of
cameras the point is visible in).  The list is then given as a list of
quadruplets <camera> <key> <x> <y>, where <camera> is a camera index,
<key> the index of the SIFT keypoint where the point was detected in
that camera, and <x> and <y> are the detected positions of that
keypoint.  Both indices are 0-based (e.g., if camera 0 appears in the
list, this corresponds to the first camera in the scene file and the
first image in "list.txt").  

We use a pinhole camera model; the parameters we estimate for each
camera are a focal length (f), two radial distortion parameters (k1
and k2), a rotation (R), and translation (t), as described in the file
specification above.  The formula for projecting a 3D point X into a
camera (R, t, f) is:

    P = R * X + t       (conversion from world to camera coordinates)
    p = -P / P.z        (perspective division)
    p' = f * r(p) * p   (conversion to pixel coordinates)

where P.z is the third coordinate of P.  In the last equation, r(p) is
a function that computes a scaling factor to undo the radial
distortion:

    r(p) = 1.0 + k1 * ||p||^2 + k2 * ||p||^4.

This gives a projection in pixels, where the origin of the image is
the center of the image, the positive x-axis points right, and the
positive y-axis points up (in addition, in the camera coordinate
system, the positive z-axis points backwards, so the camera is looking
down the negative z-axis, as in OpenGL).

Finally, the equations above imply that the camera viewing direction
is:

    R' * [0 0 -1]'  (i.e., the third row of R or third column of R')

(where ' indicates the transpose of a matrix or vector).

and the 3D position of a camera is 

    -R' * t .


Command-line options
--------------------

Bundler has a number of internal parameters, so there are a large
number of command-line options.  That said, we've found that a common
set of parameters works well for most image collections we've tried,
so it is probably safe to start with the recommended options (used by
the RunBundler.sh and bundler.py scripts).  One very useful option is
"--options_file <file>", which tells Bundler to read a list of options
from a file.  The default options file created by RunBundler.sh or
bundler.py includes the following options:

  --match_table matches.init.txt
     [specifies the file where the match files are stored]

  --output bundle.out
     [specifies the name of the final output reconstruction]

  --output_all bundle_
     [specifies that all intermediate reconstructions should be
      output to files with prefix "bundle_"]

  --output_dir bundle
     [the directory all output files should be written to, typically
      called "bundle"]

  --variable_focal_length
     [directs bundler to optimize for an independent focal length for
      each image]

  --use_focal_estimate
     [directs bundler to use the estimated focal lengths obtained from
      the Exif tags for each image]

  --constrain_focal
     [constrain the focal length of each camera to be close to the
      initial focal length estimate (from Exif tags).  This option
      adds penalty terms to the bundle adjustment objective function]

  --constrain_focal_weight 0.0001
     [weight on the penalty terms for the focal length constraints (a
      small weight is typically sufficient)]

  --estimate_distortion
     [directs bundler to estimate radial distortion parameters for
      each image]

  --run_bundle
     [run structure from motion (as opposed to other operations on
      existing reconstructions)]

There are a number of other useful options in addition to the default
ones listed above, including:

  --init_pair1 <image_idx1>
  --init_pair2 <image_idx2>
     [Specifies which images to use as the initial pair.  Very useful
      when the automatically chosen pair results in a bad
      reconstruction.]

  --options_file <options_file>
     [Read in a list of options from the specified file.]

  --sift_binary <sift>
     [The location of the SIFT binary on your installation, e.g.,
     '/usr/bin/sift' or '/cygdrive/c/usr/bin/siftWin32.exe'.]

  --add_images <add_list>
     [Given an existing reconstruction specified with the --bundle
      option, attempts to add the images listed in the file <add_list>
      to the reconstruction, writing the results to the file
      'bundle.added.out'.  The new list of images is written to
      'list.added.txt'.  Use the 'extract_focal.pl' script to generate
      the file <add_list> from a directory of JPEGs, but note that the
      correct path to these images must be included -- which may
      require editing the add list file.  Do not include the
      '--run_bundle' option when adding new images.  If the SIFT key
      files have not yet been generated for the new images, bundler
      will try to extract features, but this requires that the
      --sift_binary option be set.]

  --help
     [Print out the complete list of command-line options.]


Links
-----
Pierre Moulon has a cmake version of Bundler available here: 
    https://github.com/TheFrenchLeaf/Bundler.

Acknowledgements
----------------

This work was supported by Microsoft Research, the University of
Washington Animation Research Labs, an Achievement Rewards for College
Scientists (ARCS) fellowship, National Science Foundation grants
IIS-0413198 and DGE-0203031, and an endowment by Emer Dooley and Rob
Short.

Thanks to Manolis Lourakis and Antonis Argyros for their sparse bundle
adjustment package (http://www.ics.forth.gr/~lourakis/sba/), to David
Lowe for SIFT (http://www.cs.ubc.ca/~lowe/keypoints/), to David
M. Mount and Sunil Arya for their approximate nearest neighbors
library (http://www.cs.umd.edu/~mount/ANN/), and to Matthias Wandel
for his 'jhead' program.

Thanks as well to Kathleen Tuite and Sebastian Koch for testing this
distribution.


Contact information
-------------------

Questions?  Comments?  Bug reports?  Please see the FAQ at
http://www.cs.cornell.edu/~snavely/bundler/faq.html, or send email to
Noah Snavely at snavely@cs.cornell.edu.


[1] Noah Snavely, Steven M. Seitz, and Richard Szeliski.  Photo
    Tourism: Exploring Photo Collections in 3D.  SIGGRAPH Conf. Proc.,
    2006.

[2] Noah Snavely, Steven M. Seitz, Richard Szeliski.  Modeling the
    World from Internet Photo Collections. International Journal of
    Computer Vision, 2007.

[3] M.I.A. Lourakis and A.A. Argyros.  The Design and Implementation
    of a Generic Sparse Bundle Adjustment Software Package Based on
    the Levenberg-Marquardt Algorithm.  Tech.  Rep. 340, Inst. of
    Computer Science-FORTH, Heraklion, Crete, Greece. Available from
    http://www.ics.forth.gr/~lourakis/sba.
