NDFileHDF5
==========
:authors: Ulrik Pedersen (Diamond Light Source), Arthur Glowacki
   (Argonne National Laboratory), Alan Greer (Observatory Sciences),
   Mark Rivers (University of Chicago)

.. contents:: Contents

Overview
--------

NDFileHDF5 inherits from NDPluginFile. This plugin uses the HDF5
libraries to store data. HDF5 file format is a self-describing binary
format supported by the `hdfgroup <http://www.hdfgroup.org/HDF5>`__.

The plugin supports all NDArray datatypes and any number of NDArray
dimensions (tested up to 3). It supports storing multiple NDArrays in a
single file (in stream or capture modes) where each NDArray gets
appended to an extra dimension.

NDArray attributes are stored in the HDF5 file. In case of multi-frame
files the attributes are stored in 1D datasets (arrays).

The NDFileHDF5 plugin is created with the NDFileHDF5Configure command,
either from C/C++ or from the EPICS IOC shell.

.. code-block:: c

   NDFileHDF5Configure (const char *portName, int queueSize, int blockingCallbacks, 
                        const char *NDArrayPort, int NDArrayAddr, size_t maxMemory, 
                        int priority, int stackSize)
     

For details on the meaning of the parameters to this function refer to
the detailed documentation on the NDFileHDF5Configure function in the
`NDFileHDF5.cpp
documentation <../areaDetectorDoxygenHTML/_n_d_file_h_d_f5_8cpp.html>`__
and in the documentation for the constructor for the `NDFileHDF5
class <../areaDetectorDoxygenHTML/class_n_d_file_h_d_f5.html>`__.

File Structure Layout
---------------------

The HDF5 files comprises a hierachial data structure, similar to a file
system structure with directories (groups) and files (datasets)
[`ref <http://www.hdfgroup.org/HDF5/Tutor/fileorg.html>`__]

XML Defined File Structure Layout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is possible to define the layout of the data structures in an XML
definition file. The XML file allow defining the location of HDF5
Groups, Datasets and Attributes based on detector data (NDArrays),
metadata (NDAttributes) and constants (for instance NeXus tags).

The XML definition contains the following 4 main elements: group,
dataset, attribute, and hardlink. The terms refer to the HDF5 elements
of the same names.

-  **attribute:** represent a HDF5 attribute (key, value metadata) and
   is assigned to groups and datasets
-  **dataset:** represent a HDF5 dataset (N-dimensional array). Can
   contain attributes and is assigned to a group
-  **group:** represent a HDF5 group. Groups can contain datasets,
   attributes and other groups (recursively)
-  **hardlink:** represent an HDF5 hard link. Hardlinks can point to
   other elements in the file, such as datasets. They are analogous to
   hard links in the Linux file system.
-  **global:** Rather than representing a HDF5 object; this element
   represents a global functionality definition

.. raw:: html

  <table class="table table-bordered">
    <tbody>
      <tr>
        <th>
          XML attribute</th>
        <th>
          Required</th>
        <th>
          Description</th>
        <th>
          Value</th>
      </tr>
      <tr>
        <td align="center" colspan="4,">
          XML <b>"attribute"</b> element attributes </td>
      </tr>
      <tr>
        <td>
          name</td>
        <td>
          yes</td>
        <td>
          Name (key) of the HDF attribute (key, value) pair</td>
        <td>
          string</td>
      </tr>
      <tr>
        <td>
          source</td>
        <td>
          yes</td>
        <td>
          Definition of where the attribute gets its value from</td>
        <td>
          Enum string: "constant", "ndattribute"</td>
      </tr>
      <tr>
        <td>
          when</td>
        <td>
          optional</td>
        <td>
          Event when the attribute data is updated</td>
        <td>
          Enum string: "OnFileOpen", "OnFileClose", "OnFileWrite"</td>
      </tr>
      <tr>
        <td>
          value</td>
        <td>
          Required only if source="constant"</td>
        <td>
          The constant value to give the attribute</td>
        <td>
          string (possibly containing an int or float)</td>
      </tr>
      <tr>
        <td>
          type</td>
        <td>
          optional - use if source="constant"</td>
        <td>
          The constant datatype</td>
        <td>
          Enum string: "int", "float", "string"</td>
      </tr>
      <tr>
        <td>
          ndattribute</td>
        <td>
          Required only if source="ndattribute"</td>
        <td>
          Name of the areaDetector NDAttribute which is the source of this HDF5 attribute's
          data value</td>
        <td>
          string containing the name of an NDAttribute</td>
      </tr>
      <tr>
        <td align="center" colspan="4,">
          XML <b>"group"</b> element attributes </td>
      </tr>
      <tr>
        <td>
          name</td>
        <td>
          yes</td>
        <td>
          The (relative) name of the HDF5 group</td>
        <td>
          string</td>
      </tr>
      <tr>
        <td>
          ndattr_default</td>
        <td>
          optional</td>
        <td>
          This attribute flags a group as being a 'default' container for NDAttributes which
          have not been defined to be stored elsewhere. If there is no group defined with
          ndaddr_default=true, and if the root group does not have auto_ndattr_default=false,
          then the 'default' container will be the root group.</td>
        <td>
          boolean (default=false)</td>
      </tr>
      <tr>
        <td>
          auto_ndattr_default</td>
        <td>
          optional</td>
        <td>
          If this attribute is present for the root group and is set to false then NDAttributes
          which have not been defined to be stored elsewhere will not be stored at all</td>
        <td>
          boolean (default=false)</td>
      </tr>
      <tr>
        <td align="center" colspan="4,">
          XML <b>"dataset"</b> element attributes </td>
      </tr>
      <tr>
        <td>
          name</td>
        <td>
          yes</td>
        <td>
          Name of the HDF5 dataset</td>
        <td>
          string</td>
      </tr>
      <tr>
        <td>
          source</td>
        <td>
          yes</td>
        <td>
          Definition of where the dataset gets its data values from</td>
        <td>
          string enum: "detector", "ndattribute", "constant"</td>
      </tr>
      <tr>
        <td>
          value</td>
        <td>
          Required only if source="constant"</td>
        <td>
          Constant value to write directly into the HDF5 dataset</td>
        <td>
          String - possibly containing int or float values. Arrays of int and float values
          can also be represented in a comma-separated string</td>
      </tr>
      <tr>
        <td>
          ndattribute</td>
        <td>
          Required only if source="ndattribute"</td>
        <td>
          The name of the areaDetector NDAttribute to use as a data source for this HDF5 dataset
        </td>
        <td>
          string containing the name of the NDAttribute</td>
      </tr>
      <tr>
        <td>
          det_default</td>
        <td>
          optional</td>
        <td>
          Flag to indicate that this HDF5 dataset is the default dataset for the detector
          to write NDArrays into. Only sensible to set true if source="detector"</td>
        <td>
          boolean (default=false)</td>
      </tr>
      <tr>
        <td align="center" colspan="4,">
          XML <b>"global"</b> element attributes </td>
      </tr>
      <tr>
        <td>
          name</td>
        <td>
          yes</td>
        <td>
          Name of the global functionality or parameter to set</td>
        <td>
          enum string: "detector_data_destination"
          <br />
          (currently only one supported parameter)</td>
      </tr>
      <tr>
        <td>
          ndattribute</td>
        <td>
          Required when name="detector_data_destination"</td>
        <td>
          Name of the NDAttribute which defines the name of the dataset where incoming NDArrays
          are to be stored</td>
        <td>
          string containing the name of an NDAttribute</td>
      </tr>
      <tr>
        <td align="center" colspan="4,">
          XML <b>"hardlink"</b> element attributes </td>
      </tr>
      <tr>
        <td>
          name</td>
        <td>
          yes</td>
        <td>
          Name of the link</td>
        <td>
          string: string containing the name of the hardlink being created </td>
      </tr>
      <tr>
        <td>
          target</td>
        <td>
          yes</td>
        <td>
          Name of the existing target object in the HDF5 file being linked to.</td>
        <td>
          string containing the name of the target object being linked to</td>
      </tr>
    </tbody>
  </table>

An example XML layout file is provided in
``ADExample/iocs/simDetectorIOC/iocBoot/iocSimDetector/hdf5_layout_demo.xml``.

An XML schema is provided in
``ADCore/iocBoot/hdf5_xml_layout_schema.xsd``. The schema defines the
syntax that is allowed in the user's XML definition. It can also be used
with the 'xmllint' command to validate a user's XML definition:

::

   xmllint --noout --schema ADCore/iocBoot/hdf5_xml_layout_schema.xsd /path/to/users/layout.xml

Default File Structure Layout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If no XML Layout Definition file is loaded, the plugin will revert to
using its default file structure layout. The default layout is
compatible with the plugin's original, hard-code layout. This layout is
actually defined by an XML layout string defined in the source code file
NDFileHDF5LayoutXML.cpp.

This default layout is compatible with the Nexus file format. This is
achieved by defining a specific hierachial structure of groups and
datasets and by tagging elements in the hierachy with certain
"NX_class=nnn" attributes. Although Nexus libraries are not used to
write the data to disk, this file structure allow Nexus-aware readers to
open and read the content of these HDF5 files. This has been tested with
the Nexus reader in the `GDA application <http://opengda.org/>`__.
Hardlinks in the HDF5 file can be used to make the same dataset appear
in more than one location. This can be useful for defining a layout that
is Nexus compatible, as well as conforming to some other desired layout.

NDArray attributes
------------------

The attributes from NDArrays are stored in the HDF5 files. The list of
attributes is loaded when a file is opened so XML attributes files
should not be reloaded while writing a file in stream mode.

If the dataset is defined in the XML layout file then the user-specified
name is used for the dataset. If the dataset is not defined in the XML
layout file then the dataset name will be the the NDArray attribute
name. The NDArray attribute datasets automatically have 4 HDF
"attributes") to indicate their source type and origin. These are:

-  NDAttrName: The name of the NDArray attribute.
-  NDAttrDescrption: The description of the NDArray attribute.
-  NDAttrSourceType: The source type of the NDArray attribute:
   NDAttrSourceDriver, NDAttrSourceParam, NDAttrSourceEPICSPV, or
   NDAttrSourceFunct.
-  NDAttrSource: The source of the NDArray attribute data, i.e. the name
   of the EPICS PV, the drvInfo string for the parameter, or the name of
   the attribute function.

NDArray attributes will be stored as 1D datasets. If the location of an
NDArray attribute dataset is not defined in the XML layout file then the
dataset will appear in the group that has the property
ndattr_default="true". If there is no group with that property then the
dataset will appear in the root group. If the root group has the
property auto_ndattr_default="false" then datasets that are not
explicitly defined in the XML layout file will not appear in the HDF5
file at all.

There are 4 "virtual" attributes that are automatically created. These
are:

-  NDArrayUniqueId: The NDArray.uniqueId value.
-  NDArrayTimeStamp: The NDArray.timeStamp value.
-  NDArrayEpicsTSSec: The NDArray.epicsTS.secPastEpoch value.
-  NDArrayEpicsTSnSec: The NDArray.epicsTS.nsec value.

These properties are added to the property list and will be written to
the HDF5 file following the same rules as the actual NDArray
ndAttributes described above.

It is possible to validate the syntax of an NDArray attributes XML file.
For example, the command (starting from the ``iocBoot`` directory) to
validate the syntax of the
``iocBoot/iocSimDetector/simDetectorAttributes.xml`` file is:

::

   xmllint --noout --schema ./attributes.xsd iocSimDetector/simDetectorAttributes.xml

Default tree structure
----------------------

The group/dataset structure of the HDF5 files, generated by this plugin:

::

   entry                   <-- NX_class=NXentry
   |
   +--instrument           <-- NX_class=NXinstrument
      |
      +--detector          <-- NX_class=NXdetector
      |  |
      |  +--data           <-- NX_class=SDS, signal=1
      |  |
      |  +--NDAttributes
      |     |
      |     +--ColorMode
      |
      +--NDAttributes      <-- NX_class=NXCollection, ndattr_default="true"
      |  |
      |  +---              <-- Any number of NDAttributes from the NDArrays as individual 1D datasets
      |
      +--performance       <-- Performance of the file writing
         |
         +--timestamp      <-- A 2D dataset of different timing measurements taking during file writing
   +--data                 <-- NX_class=NDdata
      |  |
      |  +--data           <-- Hardlink to /entry/instrument/detector/data

HDF5 File Viewers
-----------------

Note that if the Single Writer Multiple Reader (SWMR) feature is
disabled then it is not possible to do a live "monitoring" of a file
which is being written by another process. The file writers mentioned in
this section can only be used to open, browse and read the HDF5 files
**after** the NDFileHDF5 plugin has completed writing and closed the
file. It is possible to view files while they are being written if the
following are all true:

-  SWMR support is enabled in the plugin.
-  The viewer client is built using HDF5 1.10 or later.
-  The file is being accessed using either a local, GPFS, or Lustre file
   system by both the client and the IOC.

`HDFView <http://www.hdfgroup.org/hdf-java-html/hdfview/index.html>`__
is a simple GUI tool for viewing and browsing HDF files. It has some
limited support for viewing images, plotting graphs and displaying data
tables.

The `HDF5
libraries <http://www.hdfgroup.org/HDF5/release/obtain5.html>`__ also
ships with a number of command-line tools for browsing and dumping data.

The screenshot below shows the hdfview application with a datafile open.
The datafile is generated by the plugin and a number of elements are
visible:

-  The NDArray NDAttributes appear as 1D datasets in the group
   "/entry/instrument/NDAttributes/"
-  The image data is in the dataset "/entry/instrument/detector/data".
   The metadata (in HDF known as "attributes") for that dataset indicate
   8bit unsigned char data, 10 frames of 60x40 pixels
-  Image and table view of the first frame data is open

.. figure:: HDFView-screenshot.png
    :align: center

Multiple Dimensions
-------------------

Both areaDetector and the HDF5 file format supports multidimensional
datasets. The dimensions from the NDArray are preserved when writing to
the HDF5 file. In multi-frame files (plugin in Stream or Capture mode)
an additional dimension is added to the dataset to hold the array of
frames.

In addition to the dimensions of the NDArray it is also possible to
specify up to 9 extra "virtual" dimensions to store datasets in the
file. This is to support applications where a sample is scanned in up to
nine dimensions. For two extra dimensions, say X and Y each scan point
contains a dataset comprising of multiple frames which can be stored.
The length of (i.e. number of points in) each of the virtual dimensions
have to be specified before the plugin opens the file for writing. This
feature is only supported in the Stream and Capture modes.

This feature allow for creating very large sets of scan data which
matches the dimensions of the performed scan in one datafile. Depending
on the application this can be a benefit in post processing.

The figure below illustrate the use of the two extra "virtual"
dimensions in a 2D (X,Y) raster scan with N frames per point:

.. figure:: HDFmultiple-dimensions.png
    :align: center

Prior to starting a scan like this the user will need to configure the
number of virtual dimensions to use (from 0 up to 9); the number of
frames per point; and the length of each of the virtual dimensions (4 x
2 in the example figure). It is not possible to change the number or
size of dimensions while the file is open.

For 2D image (greyscale) formats the dimensions in the multiframe HDF5
file are organised as follows. Note that for backwards compatibility
(previously only 2 extra dimensions were available) when extra
dimensions are specified they are in reverse order
9th,8th,7th,6th,5th,4th,3rd,Yth,Xth and dimensions 2 and 1 are named Y
and X:

-  For a multiframe file with no use of "virtual" dimension the order
   is: {Nth frame, width, height}
-  For a multiframe file using 1 "virtual" dimension (X) the order is:
   {X, Nth frame, width, height}
-  For a multiframe file using 2 "virtual" dimensions (X,Y) the order
   is: {Y, X, Nth frame, width, height}
-  For a multiframe file using 3 "virtual" dimensions (X,Y,3rd) the
   order is: {3rd, Y, X, Nth frame, width, height}
-  For a multiframe file using 4 "virtual" dimensions (X,Y,3rd,4th) the
   order is: {4th, 3rd, Y, X, Nth frame, width, height}
-  For a multiframe file using 5 "virtual" dimensions (X,Y,3rd,4th,5th)
   the order is: {5th, 4th, 3rd, Y, X, Nth frame, width, height}

Chunking
--------

This plugin uses HDF5 chunking to store the raw image data. The chunk
size (the size of each I/O block) can be configured for the frame X and
Y dimensions as well as the N'th image (which essentially implies memory
caching before writing to disk). Configuring chunking correctly for a
given application is a complex matter where both the write performance
and the read performance for a given post processing application will
have to be evaluated. As a basic starting point, setting the nColChunks
and nRowChunks parameters to the X and Y frame size respectively, should
give a decent result. In fact if these parameters are configured by the
user to the special value 0, they will default to the dimensions of the
incoming frames. Further explanations and documentation of the HDF5
chunking feature is available in the HDF5 documentation:

-  HDF5 documentation advanced topics: `Chunking in
   HDF5 <http://www.hdfgroup.org/HDF5/doc/Advanced/Chunking/index.html>`__
-  HDF5 User guide: `14.3 Data
   Chunking <http://www.hdfgroup.org/training/HDFtraining/UsersGuide/Perform.fm2.html>`__
-  hdfgroup presentation: `HDF5 Advanced Topics - Chunking in
   HDF5 <http://www.hdfgroup.org/pubs/presentations/HDF5-EOSXIII-Advanced-Chunking.pdf>`__

Compression
-----------

The HDF5 library supports a number of compression algorithms. When using
HDF5 libraries to write and read files the compression is seemless: it
only need to be switched on when writing and HDF5 enabled applications
can read the files without any additional configuration. Only one
compression filter can be applied at the time.

The following compression filters are supported in the NDFileHDF5
plugin:

-  | Lossless `SZIP <http://www.hdfgroup.org/doc_resource/SZIP/>`__
     compression is using a separate library from the hdfgroup. NOTE: The
     szip library contains the following in its COPYING license agreement
     file:
   
   | `Revocable (in the event of breach by the user or if required by law),
     royalty-free,         nonexclusive sublicense to use SZIP compression
     software routines and underlying         patents for non-commercial,
     scientific use only is hereby granted by ICs, LLC, to         users of and
     in conjunction with HDF data storage and retrieval file format and
     software library products.`
     
   | This means that the szip compression should
     not be used by commercial users without first obtaining a license.

-  External libz -also lossless
-  `N-bit <http://www.hdfgroup.org/HDF5/doc/RM/RM_H5P.html#Property-SetNbit>`__
   compression is a bit-packing scheme to be used when a detector
   provide fewer databits than standard 8,16,32 bit words. Data width
   and offset in the word is user configurable

Single Writer Multiple Reader (SWMR)
------------------------------------

From version 1-10 of the HDF5 library, reader applications shall be able
to access the file whilst it is being written. The plugin has been
updated to support the additional SWMR feature when writing a file. The
plugin will know if SWMR mode is supported depending on the version of
the HDF5 library that the plugin has been compiled against, and SWMR
mode can be enabled or disabled by setting the appropriate parameter
(disabled by default). Once placed into SWMR mode the plugin accepts
parameters to control how often frames are flushed, how often
NDAttributes are flushed and the current SWMR status and number of
flushes that have taken place are reported for an acquisition. The SWMR
active status parameter can be used to signify that it is safe for
readers to open the file (the file has been placed into SWMR mode).

Storing Attributes with Dataset Dimensions
------------------------------------------

| The plugin has the option of storing all NDAttribute datasets as a
  single array of data values, or else these NDAttribute datasets can be
  stored with the same dimensionality as the main dataset.
| As an example, consider a multi-dimensional dataset of images
  width=800 and height=600.

-  If two "virtual" dimensions are selected with N=3, X=4 and Y=5 then
   the main dataset will have the dimensions 5x4x3x600x800.
-  If the NDAttribute datasets are stored in the standard way then the
   dataset would be a single dimension array of 60 items.
-  If however, the NDAttribute dataset is stored with the same
   dimensionality as the main dataset then the NDAttribute dataset will
   have the dimensions of 5x4x3x1x1.

Selective Positional Placement of Individual Frames
---------------------------------------------------

The plugin can now store frames at specified positions within a dataset.
Each individual dimension index can be specified with NDAttributes that
are attached to the frame. The names of the NDAttributes that will be
used as the index values for each dimension are specified using
parameters (see parameter table below). The NDAttribute values
themselves must be integer type zero based index values; if values are
specified that are outside the maximum size of the dataset in any
dimension then the acquisition will fail. A named attribute must be
specified for each additional dimension if position placement mode is to
be used, if attributes are not specified or not named correctly then the
acquisition will fail.

The position placement mode of operation is enabled by setting the
PositionMode parameter to "On" and can be used with or without SWMR mode
enabled.

Writing Index Datasets
----------------------

The plugin will write out an index dataset for an extra dimension
multidimensional dataset if requested. This writing out of index values
can only take place when running in selective positional placement mode
and when storing attributes with dataset dimensions (see above). The
index parameters take the name of the NDAttribute that contains the
index values for the particular dimensions that you are interested in.

For example, extra X and Y dimensions specified along with positional
placement mode looking for NDAttributes called "x" and "y". If the index
dataset for the X dimension is set to "x" also then an additional
dataset will be written that contains only the index values for the X
dimension. The dataset will be a 1 dimensional dataset. Consider the
following index values

x=0, y=0

x=1, y=0

x=0, y=1

x=1, y=1

x=0, y=2

x=1, y=2

If the X index parameter is set to x then a 1D dataset will be produced
containing the values (0, 1). If the Y index parameter is set to y then
a 1D dataset will be produced containing the values (0, 1, 2).

Parameters and Records
----------------------

.. raw:: html

  <table class="table table-bordered">
    <tbody>
      <tr>
        <td align="center" colspan="7,">
          <b>Parameter Definitions and EPICS Record Definitions in NDFileHDF5.template</b>
        </td>
      </tr>
      <tr>
        <th>
          Parameter index variable</th>
        <th>
          asyn interface</th>
        <th>
          Access</th>
        <th>
          Description</th>
        <th>
          drvInfo string</th>
        <th>
          EPICS record name</th>
        <th>
          EPICS record type</th>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>HDF5 XML Layout Definition</b></td>
      </tr>
      <tr>
        <td>
          XMLFileName</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          XML filename, pointing to an XML HDF5 Layout Definition<br />
          This waveform also supports loading raw XML code directly; up to a maximum of 1MB
          long (NELM=1MB)</td>
        <td>
          HDF5_layoutFilename</td>
        <td>
          $(P)$(R)XMLFileName<br />
          $(P)$(R)XMLFileName_RBV</td>
        <td>
          waveform</td>
      </tr>
      <tr>
        <td>
          layoutValid</td>
        <td>
          asynInt32</td>
        <td>
          r/o</td>
        <td>
          Flag to report the validity (xml syntax only) of the loaded XML. Updated when the
          XMLFileName is updated with a new filename and when the XML file is read at HDF5
          file creation</td>
        <td>
          HDF5_layoutValid</td>
        <td>
          $(P)$(R)XMLValid_RBV</td>
        <td>
          bi</td>
      </tr>
      <tr>
        <td>
          layoutErrorMsg</td>
        <td>
          asynOctet</td>
        <td>
          r/o</td>
        <td>
          XML parser error message</td>
        <td>
          HDF5_layoutErrorMsg</td>
        <td>
          $(P)$(R)XMLErrorMsg_RBV</td>
        <td>
          waveform</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>HDF5 Chunk Configuration</b></td>
      </tr>
      <tr>
        <td>
          nRowChunks</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Configure HDF5 "chunking" to approriate size for the filesystem: sets number of
          rows to use per chunk</td>
        <td>
          HDF5_nRowChunks</td>
        <td>
          $(P)$(R)NumRowChunks<br />
          $(P)$(R)NumRowChunks_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          nColChunks</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Configure HDF5 "chunking" to approriate size for the filesystem: sets number of
          columns to use per chunk</td>
        <td>
          HDF5_nColChunks</td>
        <td>
          $(P)$(R)NumColChunks<br />
          $(P)$(R)NumColChunks_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          nFramesChunks</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Configure HDF5 "chunking" to approriate size for the filesystem: sets number of
          frames to use per chunk. For a 2D image, setting this parameter > 1 essentially
          implies using in-memory cache as HDF5 only writes full chunks to disk.</td>
        <td>
          HDF5_nFramesChunks</td>
        <td>
          $(P)$(R)NumFramesChunks<br />
          $(P)$(R)NumFramesChunks_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Disk Boundary Alignment</b></td>
      </tr>
      <tr>
        <td>
          chunkBoundaryAlign</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Set the disk boundary alignment in bytes. This parameter can be used to optimise
          file I/O performance on some file systems. For instance on the Lustre file system
          where the it is optimal to align data to the 'stripe size' (default 1MB).
          <br />
          This parameter applies to all datasets in the file.
          <br />
          Setting this parameter to 0 disables use of disk boundary alignment.
          <br />
          <i>Warning: setting this parameter to a larger size than the size of a single chunk
            will cause datafiles to grow larger than the actual contained data.</i> </td>
        <td>
          HDF5_chunkBoundaryAlign</td>
        <td>
          $(P)$(R)BoundaryAlign<br />
          $(P)$(R)BoundaryAlign_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          chunkBoundaryThreshold</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Set a minimum size (bytes) of chunk or dataset where boundary alignment is to be
          applied. This can be used to filter out small datasets like NDAttributes from the
          boundary alignment as it could blow up the file size.
          <br />
          Setting this parameter to 0 will disable the use of boundary alignment </td>
        <td>
          HDF5_chunkBoundaryThreshold</td>
        <td>
          $(P)$(R)BoundaryThreshold<br />
          $(P)$(R)BoundaryThreshold_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Metadata</b></td>
      </tr>
      <tr>
        <td>
          storeAttributes</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Enable or disable support for storing NDArray attributes in file</td>
        <td>
          HDF5_storeAttributes</td>
        <td>
          $(P)$(R)StoreAttr<br />
          $(P)$(R)StoreAttr_RBV</td>
        <td>
          bo<br />
          bi</td>
      </tr>
      <tr>
        <td>
          storePerformance</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Enable or disable support for storing file IO timing measurements in file</td>
        <td>
          HDF5_storePerformance</td>
        <td>
          $(P)$(R)StorePerform<br />
          $(P)$(R)StorePerform_RBV</td>
        <td>
          bo<br />
          bi</td>
      </tr>
      <tr>
        <td>
          dimAttDatasets</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Turn on or off NDAttribute dataset dimensions (1 = On, 0 = Off). When switched on
          NDAttribute datasets are forced to have the same dimensionality as the main dataset.
        </td>
        <td>
          HDF5_dimAttDatasets</td>
        <td>
          $(P)$(R)DimAttDatasets<br />
          $(P)$(R)DimAttDatasets_RBV</td>
        <td>
          bo<br />
          bi</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>SWMR</b></td>
      </tr>
      <tr>
        <td>
          SWMRSupported</td>
        <td>
          asynInt32</td>
        <td>
          r/o</td>
        <td>
          Does the HDF5 library version support SWMR mode of operation (1 = Supported, 0 =
          Not Supported).</td>
        <td>
          HDF5_SWMRSupported</td>
        <td>
          $(P)$(R)SWMRSupported_RBV</td>
        <td>
          bi</td>
      </tr>
      <tr>
        <td>
          SWMRMode</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Turn on or off SWMR mode for the next acquisition (1 = On, 0 = Off). Turning on
          will only work if SWMR mode is supported.</td>
        <td>
          HDF5_SWMRMode</td>
        <td>
          $(P)$(R)SWMRMode<br />
          $(P)$(R)SMWRMode_RBV</td>
        <td>
          bo<br />
          bi</td>
      </tr>
      <tr>
        <td>
          SWMRRunning</td>
        <td>
          asynInt32</td>
        <td>
          r/o</td>
        <td>
          This value is set to 1 once a file has been opened and placed into SWMR mode. It
          returns to 0 once the acquisition has completed.</td>
        <td>
          HDF5_SWMRRunning</td>
        <td>
          $(P)$(R)SWMRActive_RBV</td>
        <td>
          bi</td>
      </tr>
      <tr>
        <td>
          flushNthFrame</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Flush the image to disk every N'th frame.</td>
        <td>
          HDF5_flushNthFrame</td>
        <td>
          $(P)$(R)NumFramesFlush<br />
          $(P)$(R)NumFramesFlush_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          NDAttributeChunk</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          This value is used to determine when to flush NDAttribute datasets to disk, and
          the corresponding datasets chunk size. A value of zero will default where possible
          to the size of the dataset for a one dimensional dataset.</td>
        <td>
          HDF5_NDAttributeChunk</td>
        <td>
          $(P)$(R)NDAttributeChunk<br />
          $(P)$(R)NDAttributeChunk_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          SWMRCbCounter</td>
        <td>
          asynInt32</td>
        <td>
          r/o</td>
        <td>
          The number of flushes that have taken place for the current acquisition. In the
          case where flushing occurs for every frame and no flushing is set for NDAttribute
          datasets then this value will increment by one for each frame, and once the acquisition
          has completed it will jump by the number of flushes required for the NDAttribute
          datasets, one flush for each dataset.</td>
        <td>
          HDF5_SWMRCbCounter</td>
        <td>
          $(P)$(R)SWMRCbCounter_RBV</td>
        <td>
          longin</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Additional Virtual Dimensions</b></td>
      </tr>
      <tr>
        <td>
          nExtraDims</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Number of extra dimensions [0..9]</td>
        <td>
          HDF5_nExtraDims</td>
        <td>
          $(P)$(R)NumExtraDims<br />
          $(P)$(R)NumExtraDims</td>
        <td>
          mbbo<br />
          mbbi</td>
      </tr>
      <tr>
        <td>
          extraDimSizeN</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension N (no. of frames per point)</td>
        <td>
          HDF5_extraDimSizeN</td>
        <td>
          $(P)$(R)ExtraDimSizeN<br />
          $(P)$(R)ExtraDimSizeN_RBV</td>
        <td>
        </td>
      </tr>
      <tr>
        <td>
          extraDimSizeX</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension X</td>
        <td>
          HDF5_extraDimSizeX</td>
        <td>
          $(P)$(R)ExtraDimSizeX<br />
          $(P)$(R)ExtraDimSizeX_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSizeY</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension Y</td>
        <td>
          HDF5_extraDimSizeY</td>
        <td>
          $(P)$(R)ExtraDimSizeY<br />
          $(P)$(R)ExtraDimSizeY_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize3</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 3</td>
        <td>
          HDF5_extraDimSize3</td>
        <td>
          $(P)$(R)ExtraDimSize3<br />
          $(P)$(R)ExtraDimSize3_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize4</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 4</td>
        <td>
          HDF5_extraDimSize4</td>
        <td>
          $(P)$(R)ExtraDimSize4<br />
          $(P)$(R)ExtraDimSize4_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize5</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 5</td>
        <td>
          HDF5_extraDimSize5</td>
        <td>
          $(P)$(R)ExtraDimSize5<br />
          $(P)$(R)ExtraDimSize5_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize6</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 6</td>
        <td>
          HDF5_extraDimSize6</td>
        <td>
          $(P)$(R)ExtraDimSize6<br />
          $(P)$(R)ExtraDimSize6_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize7</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 7</td>
        <td>
          HDF5_extraDimSize7</td>
        <td>
          $(P)$(R)ExtraDimSize7<br />
          $(P)$(R)ExtraDimSize7_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize8</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 8</td>
        <td>
          HDF5_extraDimSize8</td>
        <td>
          $(P)$(R)ExtraDimSize8<br />
          $(P)$(R)ExtraDimSize8_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          extraDimSize9</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Size of extra dimension 9</td>
        <td>
          HDF5_extraDimSize9</td>
        <td>
          $(P)$(R)ExtraDimSize9<br />
          $(P)$(R)ExtraDimSize9_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Positional Placement</b></td>
      </tr>
      <tr>
        <td>
          posRunning</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Turn on/off positional placement mode</td>
        <td>
          HDF5_posRunning</td>
        <td>
          $(P)$(R)PositionMode<br />
          $(P)$(R)PositionMode_RBV</td>
        <td>
          bo<br />
          bi</td>
      </tr>
      <tr>
        <td>
          posName[0]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the N index</td>
        <td>
          HDF5_posNameDimN</td>
        <td>
          $(P)$(R)PosNameDimN<br />
          $(P)$(R)PosNameDimN_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[1]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the X index</td>
        <td>
          HDF5_posNameDimX</td>
        <td>
          $(P)$(R)PosNameDimX<br />
          $(P)$(R)PosNameDimX_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[2]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the Y index</td>
        <td>
          HDF5_posNameDimY</td>
        <td>
          $(P)$(R)PosNameDimY<br />
          $(P)$(R)PosNameDimY_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[3]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 3rd index</td>
        <td>
          HDF5_posNameDim3</td>
        <td>
          $(P)$(R)PosNameDim3<br />
          $(P)$(R)PosNameDim3_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[4]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 4th index</td>
        <td>
          HDF5_posNameDim4</td>
        <td>
          $(P)$(R)PosNameDim4<br />
          $(P)$(R)PosNameDim4_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[5]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 5th index</td>
        <td>
          HDF5_posNameDim5</td>
        <td>
          $(P)$(R)PosNameDim5<br />
          $(P)$(R)PosNameDim5_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[6]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 6th index</td>
        <td>
          HDF5_posNameDim6</td>
        <td>
          $(P)$(R)PosNameDim6<br />
          $(P)$(R)PosNameDim6_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[7]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 7th index</td>
        <td>
          HDF5_posNameDim7</td>
        <td>
          $(P)$(R)PosNameDim7<br />
          $(P)$(R)PosNameDim7_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[8]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 8th index</td>
        <td>
          HDF5_posNameDim8</td>
        <td>
          $(P)$(R)PosNameDim8<br />
          $(P)$(R)PosNameDim8_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posName[9]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute name for the 9th index</td>
        <td>
          HDF5_posNameDim9</td>
        <td>
          $(P)$(R)PosNameDim9<br />
          $(P)$(R)PosNameDim9_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Index Datasets</b></td>
      </tr>
      <tr>
        <td>
          posIndex[0]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the N dimension</td>
        <td>
          HDF5_posIndexDimN</td>
        <td>
          $(P)$(R)PosIndexDimN<br />
          $(P)$(R)PosIndexDimN_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[1]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the X dimension</td>
        <td>
          HDF5_posIndexDimX</td>
        <td>
          $(P)$(R)posIndexDimX<br />
          $(P)$(R)posIndexDimX_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[2]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the Y dimension</td>
        <td>
          HDF5_posIndexDimY</td>
        <td>
          $(P)$(R)posIndexDimY<br />
          $(P)$(R)posIndexDimY_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[3]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 3rd dimension</td>
        <td>
          HDF5_posIndexDim3</td>
        <td>
          $(P)$(R)posIndexDim3<br />
          $(P)$(R)posIndexDim3_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[4]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 4th dimension</td>
        <td>
          HDF5_posIndexDim4</td>
        <td>
          $(P)$(R)posIndexDim4<br />
          $(P)$(R)posIndexDim4_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[5]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 5th dimension</td>
        <td>
          HDF5_posIndexDim5</td>
        <td>
          $(P)$(R)posIndexDim5<br />
          $(P)$(R)posIndexDim5_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[6]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 6th dimension</td>
        <td>
          HDF5_posIndexDim6</td>
        <td>
          $(P)$(R)posIndexDim6<br />
          $(P)$(R)posIndexDim6_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[7]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 7th dimension</td>
        <td>
          HDF5_posIndexDim7</td>
        <td>
          $(P)$(R)posIndexDim7<br />
          $(P)$(R)posIndexDim7_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[8]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 8th dimension</td>
        <td>
          HDF5_posIndexDim8</td>
        <td>
          $(P)$(R)posIndexDim8<br />
          $(P)$(R)posIndexDim8_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td>
          posIndex[9]</td>
        <td>
          asynOctet</td>
        <td>
          r/w</td>
        <td>
          Specify the NDAttribute index for the 9th dimension</td>
        <td>
          HDF5_posIndexDim9</td>
        <td>
          $(P)$(R)posIndexDim9<br />
          $(P)$(R)posIndexDim9_RBV</td>
        <td>
          stringout<br />
          stringin</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Runtime Statistics</b></td>
      </tr>
      <tr>
        <td>
          totalRuntime</td>
        <td>
          asynFloat64</td>
        <td>
          r/o</td>
        <td>
          Total runtime in seconds from first frame to file closed</td>
        <td>
          HDF5_totalRuntime</td>
        <td>
          $(P)$(R)Runtime</td>
        <td>
          ai</td>
      </tr>
      <tr>
        <td>
          totalIoSpeed</td>
        <td>
          asynFloat64</td>
        <td>
          r/o</td>
        <td>
          Overall IO write speed in megabit per second from first frame to file closed</td>
        <td>
          HDF5_totalIoSpeed</td>
        <td>
          $(P)$(R)IOSpeed</td>
        <td>
          ai</td>
      </tr>
      <tr>
        <td align="center" colspan="7,">
          <b>Compression Filters</b></td>
      </tr>
      <tr>
        <td>
          compressionType</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          Select or switch off compression filter</td>
        <td>
          HDF5_compressionType</td>
        <td>
          $(P)$(R)Compression<br />
          $(P)$(R)Compression_RBV</td>
        <td>
          mbbo<br />
          mbbi</td>
      </tr>
      <tr>
        <td>
          nbitsPrecision</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          N-bit compression filter: number of data bits per pixel</td>
        <td>
          HDF5_nbitsPrecision</td>
        <td>
          $(P)$(R)NumDataBits<br />
          $(P)$(R)NumDataBits_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          nbitsOffset</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          N-bit compression filter: dataword bit-offset in pixel</td>
        <td>
          HDF5_nbitsOffset</td>
        <td>
          $(P)$(R)DataBitsOffset<br />
          $(P)$(R)DataBitsOffset_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          szipNumPixels</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          szip compression filter: number of pixels in filter [1..32]</td>
        <td>
          HDF5_szipNumPixels</td>
        <td>
          $(P)$(R)SZipNumPixels<br />
          $(P)$(R)SZipNumPixels_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
      <tr>
        <td>
          zCompressLevel</td>
        <td>
          asynInt32</td>
        <td>
          r/w</td>
        <td>
          zlib compression filter: compression level [1..9]</td>
        <td>
          HDF5_zCompressLevel</td>
        <td>
          $(P)$(R)ZLevel<br />
          $(P)$(R)ZLevel_RBV</td>
        <td>
          longout<br />
          longin</td>
      </tr>
    </tbody>
  </table>

Screenshots
-----------

.. figure:: NDFileHDF5.png
    :align: center
    
    NDFileHDF5.adl
