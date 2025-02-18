.. _eyetrackingPsychoPy:

1:00 - 1:40pm: Incorporating Eye tracking into PsychoPy Experiments
==========================================================================

To add eyetracking into your study you will generally:

A. Configure ioHub for the eye tracker to be used ( configuration settings are hardware-dependent )
B. Run the eye tracker setup routine, which will hopefully result in the successful calibration of the ET hardware
C. Start event reporting for the ET device.
D. Monitor eye tracker events or status as needed
E. Inform the eye tracker to stop reporting events.
F. Close the connection to the ET device.

This can be done by writing Python script and using PsychoPy in the Coder mode,
or by adding custom python code segments to the PsychoPy Builder. Support for
graphically adding eye tracking support and data access from within Builder is 
expected to occur by end of this year.

*******************************
Using an Eye Tracker from Coder
*******************************

For this Section of the Workshop we will use the PsychoPy Coder.

1. Open the PsychoPy Coder IDE

    #. Start->Programs->PsychoPy2->PsychoPy

2. Ensure the IDE is in *Coder* Mode

    #. If title of IDE has *Coder* in it, you are in the Coder View.
    #. Otherwise, select menu View->Open Coder View.
    #. Close the Builder View.
    
3. Open the the getting_started.py demo script:
    - Select Menu File->Open
    - Python file is found in [Worshop Materials Root]\demos\coder\getting_started\getting_started.py
    
So now you should have the PsychoPy Coder IDE open and it should look soemthing like this:

.. image:: ./psychopy_coder.png
    :width: 422px
    :align: center
    :height: 450px
    :alt: PsychoPy Coder
    
Basic Eye Tracking Coder Example
------------------------------------------

.. literalinclude:: ..\..\demos\coder\getting_started\getting_started.py
    :language: python

*********************************
Using an Eye Tracker in Builder
*********************************

There isn't currently an Eyetracker Component in Builder (I'm sure there will be very soon!) but you can effectively create one yourself using a code component. Remember, these have 5 sections for `Beginning the Experiment`, `Beginning the Routine` (e.g. trial), `Each Frame` of the Routine, `End of the Routine` and `End of the Experiment`.

The way we've set up the demos is that they check first whether you've asked for an eye tracker to be used - in `Experiment Settings` we added an entry to the experiment info dialog box called 'Eye tracker'. In the code below, if that is set to be a string that represents a valid yaml config file then we'll have an eyetracker installed and if not we'll revert to using the mouse as before (handy while creating the experiment in your office!).


Stroop: 
-----------

We'll look at these steps for a new version of the Stroop task where we simply check whether fixation was maintained during the trial and flag trials where it was broken (at any point).

Begin the Experiment
~~~~~~~~~~~~~~~~~~~~~~~~~

Here we need to import and launch the ioHub server and set up some default values for the rest of the experiment (like how large a window we think is reasonable for fixation to be maintained)::

    maintain_fix_pix_boundary=66.0 # pixels
    eyetracker =False #will change if we get one!
    
    if expInfo['Eye Tracker']:
        from psychopy.iohub import EventConstants,ioHubConnection,load,Loader
        from psychopy.data import getDateStr
        
        # Load the specified iohub configuration file converting it to a python dict.
        io_config=load(file(expInfo['Eye Tracker'],'r'), Loader=Loader)

        # Add / Update the session code to be unique. Here we use the psychopy getDateStr() function for session code generation
        session_info=io_config.get('data_store').get('session_info')
        session_info.update(code="S_%s"%(getDateStr()))

        # Create an ioHubConnection instance, which starts the ioHubProcess, and informs it of the requested devices and their configurations.
        io=ioHubConnection(io_config)

        iokeyboard=io.devices.keyboard
        mouse=io.devices.mouse
        if io.getDevice('tracker'):
            eyetracker=io.getDevice('tracker')
            
            # Run the eye tracker setup routine.
            
            win.winHandle.minimize()
            eyetracker.runSetupProcedure()
            win.winHandle.activate()
            win.winHandle.maximize()
            
        display_gaze=False
        x,y=0,0
    
Notes:
    - We only import ioHub and set it up if it will be needed!
    - We perform the eye tracker setup procedure.
    - We should create initial values here for things that will be updated during the script (like the current x,y so that other parts of the script won't throw an error if they use them before the first time the true values are determined)
    
Begin the Routine
~~~~~~~~~~~~~~~~~~~~~~~~~

Simple code that runs if the eyetracker exists (remember, that started as False but was then assigned an eyetracker object if one was successfully created)::

    if eyetracker:
        heldFixation = True #unless otherwise
        io.clearEvents('all')
        eyetracker.setRecordingState(True)
Notes:
    - at the beginning of the trial we create a variable `heldFixation` and set it to be True. We'll check on each frame if it stays true but this is our default.
    - clearing events means we don't worry what happened before the trial started
    - we start collecting eye data

Each Frame
~~~~~~~~~~~~~~~~

Now we need to check whether gaze has strayed outside the valid fixation window. But we'll also check whether the user pressed 'g' and if so we'll toggle the `display_gaze` variable.::

    if eyetracker:
        # check for 'g' key press to toggle gaze cursor visibility
        iokeys=iokeyboard.getEvents(EventConstants.KEYBOARD_PRESS)
        for iok in iokeys:
            if iok.key==u'g':
                display_gaze=not display_gaze
        # get /eye tracker gaze/ position 
        gpos=eyetracker.getPosition()
        if type(gpos) in [list,tuple]:
            x,y=gpos
            d=np.sqrt(x**2+y**2)
            if d>maintain_fix_pix_boundary:
                heldFixation = False #unless otherwise

Notes:
    - On each frame we check if the 'g' key has been pressed. If so, we toggle the visibility of a gaze contingent dot.
    - We get the latest eye tracker gaze position from the iohub. If it is a tuple or list, we know we have valid eye position info.
    - If we have a valid eye position, we calculate the distance between the center of the screen and the eye position reported. 
    - If the eye to screan center distance is above threshold, we flag that fixation was not maintained for the trial.

End of Routine
~~~~~~~~~~~~~~~~~~~~

This is some simple code at the end of the trial that uses the standard data outputs of PsychoPy - a column will appear in the excel/csv file showing whether fixation was held on each trial. We also stop recording eye data at the end of each trial::

    if eyetracker:
        eyetracker.setRecordingState(False)
        #add eye-track data to data file
        trials.addData("heldFixation", heldFixation)
            
End Experiment
~~~~~~~~~~~~~~~~~~~~

At the end of the experiment we close the connection to the eye tracker. Since ioHub
runs in a separate process; it's good practice to shut that down just in case it 
fails to do so itself!::

    if eyetracker:
        eyetracker.setConnectionState(False)
        io.quit()

Gaze cursor
~~~~~~~~~~~~~~~~

How do we make use of that `display_gaze` variable to show where the gaze is currently located? Simple! We add a `Grating Component` (for example) that has:
    - a small size
    - an opacity of `$display_gaze`. That means it uses the display_gaze variable (which is True=1, False=0). Make sure you `set every frame`
    - a location of `$[x,y]`. Make sure you `set every frame`