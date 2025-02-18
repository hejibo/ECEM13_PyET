.. _ioHub:

11.20 - 12.00 Introducing the ioHub
========================================

****************************
Overview
****************************

* PsychoPy.ioHub is a Python package providing a cross-platform device
  event monitoring and storage framework. 
* ioHub is free to use and is GPL version 3 licensed.
* Support for the following operating Systems:
    * Windows XP SP3, 7, 8
    * Apple OS X 10.6+
    * Linux 2.6+
* Monitoring of events from computer _devices_ such as:
    * Keyboard
    * Mouse
    * Analog to Digital Converter
    * XInput compatible gamepads
    * Remote ioHub Server instances
    * **Eye Trackers, via the ioHub Common Eye Tracking Interface**

The ioHub is designed to solve several issues in experiments involving eyetracking and other high-throughput data collection.
    * Asynchronous from the stimulus presentation process so that all hardware polling or callback handling occurs at a high rate. Events are collected and time stamped if necessary regardless of what the experiment is doing (loading images or videos, waiting for the next retrace start, etc.
    * Not only asynchronous but running on a separate core (assuming you have more than one). That means intensive data collection doesn't result in sloppy timing in stimulus presentation
    * Can also save high-throughput data to disk using without impacting either stimulus presentation or data collection, and quickly query and access very large data sets.


PsychoPy.ioHub MultiProcess Design
---------------------------------------

.. image:: ./iohub_diagram.png
    :width: 480px
    :align: center
    :height: 325px
    :alt: ioHub MultiProcess Design
    

Useful PsychoPy.ioHub Links
---------------------------------------

Installation
~~~~~~~~~~~~~~~~~~~

* ioHub is installed as part of PsychoPy

Documentation
~~~~~~~~~~~~~~~~~~~

*Docs are yet to be merged with the core PsychoPy documentation*

* `PsychoPy Docs <http://www.psychopy.org/documentation.html>`_
* `ioHub Docs <http://www.isolver-solutions.com/iohubdocs/index.html>`_

Support
~~~~~~~~~~~~~~~~~~~

`PsychoPy User Group <https://groups.google.com/forum/#!forum/psychopy-users>`_

Want to Contribute?
~~~~~~~~~~~~~~~~~~~~~~

`PsychoPy Developer Group <https://groups.google.com/forum/#!forum/psychopy-dev>`_

High Level ioHub API Review
--------------------------------

Starting the ioHub Server Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three ways to create a PsychoPy experiment which uses the iohub process. All approaches ultimately give you access to an instance of the 
`ioHubConnection Class <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/iohub_process/getting_connected.html#the-iohubconnection-class>`_ 
for communication and control of the iohub server:

1. Directly create a `ioHubConnection Class <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/iohub_process/getting_connected.html#the-iohubconnection-class>`_ instance.
2. Use the `psychopy.iohub.launchHubServer() <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/iohub_process/launchHubServer.html#the-launchhubserver-function>`_ function.
3. Use the `psychopy.iohub.client.ioHubExperimentRuntime <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/iohub_process/ioHubExperimentRuntime.html#the-iohubexperimentruntime-class>`_, implementing the class's run() method for your experiment's starting python code.

Each approach has pros and cons depending on the devices being used, whether Coder or Buider is being used for experiment creation, etc.

See the `ioHub Getting Connected documentation section <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/iohub_process/getting_connected.html>`_ for  details on each approach.

.. image:: ./getting_connected_page.png
    :width: 800px
    :align: center
    :height: 500px
    :alt: The ioHub Getting Connected Online Help Page


An example python script is available at [workshop_materials_root]python_source/launchHubServer.py, which illustrates how to use the launchHubServer() method.
Several other examples included in the workshop materials illustrate how to use the different connection approaches.

.. _commonETinterface:

******************************************
The ioHub Common Eye Tracker Interface
****************************************** 

The API details for the ioHub EyeTracker device can be found `here <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/device_details/eyetracker.html#the-iohub-common-eye-tracker-interface>`_

The Common Eye Tracking Interface provides the same user level Python API for all supported eye tracking hardware, meaning:

* The same experiment script can be run with any supported eye tracker hardware.
* The eye event types supported by an eye tracker implementation are saved using the same format regardless of eye tracker hardware.

Currently Supported Eye Tracking Systems
------------------------------------------------------

* **LC Technologies** EyeGaze and EyeFollower models.
* **SensoMotoric Instruments** iViewX models.
* **SR Research** EyeLink models.
* **Tobii Technologies** Tobii models.

Visit the ioHub documentation for `Eye Tracking Hardware Implementations <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/device_details/eyetracker.html#eye-tracking-hardware-implementations>`_ for details on each implementation.

Areas of Functionality
---------------------------

1. **Initializing the Eye Tracker / Setting the Device State.**
2. **Performing Calibration / System Setup.**
3. **Starting and Stopping of Data Recording.**
4. Sending Messages or Codes to an Eye Tracker.
5. **Accessing Eye Tracker Data During Recording.**
6. **Accessing the Eye Tracker native time base.**
7. **Synchronizing the ioHub time base with the Eye Tracker time base.**

**Note:** Area's of Functionality in **bold** are considered core areas, and must be implemented for every eye tracker interface.

See the `Common Eye Tracker Interface API specification <http://www.isolver-solutions.com/iohubdocs/iohub/api_and_manual/device_details/eyetracker.html#the-iohub-common-eye-tracker-interface>`_ for API details.

.. _ioDataStore:

*************************************************************
ioDataStore - Saving Event Data
*************************************************************

For relatively small amounts of data you can fetch information from the ioHub while back to the stimulus presentation thread and use PsychoPy's standard data storage facilities. (You would still benefit from the fact that the events had been timestamped by ioHub on collection so it doesn't matter that you only retrieve the data from ioHub once per screen refresh). At other times you might be saving large streams of eye-movement data and you can use ioHub to save the data directly to disk using the HDF5 standardised data format.

What can be Stored
----------------------------------------------

* All events from all monitored devices during an experiment runtime.
* Experiment meta-data (experiment name, code, description, version,...) 
* Session meta-data (session code, any user defined session level variables)
* Experiment DV and IV information (generally saved on a trial by trial basis, but up to you)

Notable Features
----------------------------------------------
* You control what devices save which event types to the data store.
* Data is saved in the standard `HDF5 file format <http://www.hdfgroup.org>`_
* `Pytables <http://www.pytables.org>`_ is the Python package used to provide HDF5 read / write access.
* Each device event is saved in a table like structure; different event types use different event tables.
* Events are retrieved as numpy ndarrays, and can therefore easily and directly be used by packages such as numpy, scipy, and matplotlib.
* Data Files can be opened and viewed using free HDF5 Viewing tools such as `HDFView` <http://www.hdfgroup.org/hdf-java-html/hdfview/>`_ and `ViTables <http://vitables.org/download/index.html>`_, both of which are open-source, free, and cross-platform.

*************************************
ioDataStore File Structure
*************************************

Hierchial File Structure and Meta-Data Tables
-----------------------------------------------
ioDataStore HDF5 File Viewed using the HDFView Application

.. image:: ./ioDataStore_HDF5_File_Structure.png
    :align: center
    :alt: ioDataStore HDF5 File Viewed using the HDFView Application

Example Event Table: Monocular Eye Sample Event
-------------------------------------------------

Example Event Table Viewed using the HDFView Application

.. image:: ./ioDataStore_MonoSample_Event.png
    :align: center
    :alt: Example Event Table Viewed using the HDFView Application   

***********************************************************
Reading Saved Data - the ExperimentDataAccessUtility Class
***********************************************************


* Contains ioHub device event reading functionality
* Simple event access API
* Access events using the same type constants and event attributes as are used during on-line event access. 
* Supports on-disk querying of event tables based on event attribute values and meta-data info.; *fast* retieval of only the events which meet the query constraints.
* When combined with use of the ExperimentVariableProvider class, events access can be filtered by:
    * Dependent and Independent Conditions
    * Session and Trial IDs
    * Other Variables Calculated at Runtime, e.g. Trial Start and End Times, Stimulus Onset and Offset Times, etc
    * Any Event Attribute Value

Examples of using the ioDataStore Access API
---------------------------------------------

Open ExperimentDataAccessUtility and explore ioDataStore file structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Source file:** python_source/datastore_examples/printing_datastore_file_structure.py

.. literalinclude:: python_source/iodatastore_examples/printing_datastore_file_structure.py
    :language: python
    

Accessing Experiment and Session Meta Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Source file:** python_source/datastore_examples/access_exp_metadata.py

.. literalinclude:: python_source/iodatastore_examples/access_exp_metadata.py
    :language: python
    
Read Any Saved Trial Condition Variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Source file:** python_source/datastore_examples/read_condition_data.py

.. literalinclude:: python_source/iodatastore_examples/read_condition_data.py
    :language: python
    

List Device Event Types Where the Event Count > 0
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Source file:** python_source/datastore_examples/access_events_with_data.py

.. literalinclude:: python_source/iodatastore_examples/access_events_with_data.py
    :language: python
    

Retrieving Specific Event Fields Grouped by Trial using a Trial Condition Query Selection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Source file:** python_source/datastore_examples/access_single_event_table.py

.. literalinclude:: python_source/iodatastore_examples/access_single_event_table.py
    :language: python
    
