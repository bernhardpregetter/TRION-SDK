Synchronization
===============

Multiple TRION (DEWE2, DEWE3) enclosures can be synchronized using various synchronization methods.
Supported methods are:

* TRION-SYNC-BUS
* PTP IEEE1588
* GPS
* IRIG A/B


TRION-SYNC-BUS is part of all enclosures. All other options need dedicated enclosures or TRION-TIMING boards in
the enclosures first slot.


TRION-SYNC-BUS
--------------

Using TRION-SYNC-BUS needs special setup for two different enclosure roles. There has to be one MASTER
instrument and one or more SLAVE instruments.
Setup of MASTER or SLAVE instrument roles is handled differently, for DEWE2 enclosures and DEWE3 enclosures.
The details are described within the chapter of the roles.
DEWE3 chassis need to be setup differently, depending on the Chassis-Controller-Version present.
This can bei either V1 or V2.
The easiest way to query this information is, by querying the "type" information of BoardId0 from API.

.. code:: c

    DeWeGetParamStruct_str("BoardId0/boardproperties/Key2Name/KEY/InternalId", "Type", , Buf, sizeof(Buf));

.. tabularcolumns:: |p{3cm}|p{3cm}|

.. table:: Enclosure Type via Chassis-Controller-Version
   :widths: 30 30

   +----------------------+----------------------------+
   | Buf content          | Enclosure-Type             |
   +======================+============================+
   | CONTROLLER-V1        | DEWE3 with Controller-V1   |
   +----------------------+----------------------------+
   | CONTROLLER-V2        | DEWE3 with Controller-V2   |
   +----------------------+----------------------------+
   | anything else        | DEWE2                      |
   +----------------------+----------------------------+

Cabling
~~~~~~~

Connect the SYNC cable to the sync-out plug at the master instrument to the sync-in plug of the first slave
instrument. For further slave instruments follow the pattern and connect the slave's sync-out plug to the next
slave's sync-in plug.


Master instrument
~~~~~~~~~~~~~~~~~

The board setup is the same when multiple TRION boards are used. The first board has to be set to Master mode,
all others to Slave:

Master board setup "BoardID0":


.. code:: c

    DeWeSetParamStruct_str("BoardID0/AcqProp", "OperationMode", "Master");
    DeWeSetParamStruct_str("BoardID0/AcqProp", "ExtTrigger", "False");
    DeWeSetParamStruct_str("BoardID0/AcqProp", "ExtClk", "False");


Slave board setup "BoardIDX" [for X from 1 to NrOfAvailableBoards]:

.. code:: c

    DeWeSetParamStruct_str("BoardIDX/AcqProp", "OperationMode", "Slave");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtTrigger", "PosEdge");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtClk", "False");


SYNC-OUT has to be configured:

DEWE2 enclosure
^^^^^^^^^^^^^^^^

On exactly one board

* Set the trigger line TRIG7, Source to low
* Set the trigger line TRIG7, Inverted to false

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT DEWE2
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Low          |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

On all other, but the one chosen for TRIG7 control.
This is important to prevent multiple boards trying to drive the TRIG7-line to different levels.

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT DEWE2
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Disable      |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+


These are the appropriate TRION-API commands:

.. code:: c

    DeWeSetParamStruct_str("BoardID0/Trig7", "Source", "Low");
    DeWeSetParamStruct_str("BoardID0/Trig7", "Inverted", "False");
    // Then apply the settings using:
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);


DEWE3 enclosure with Chassis-Controller-V1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

On the Chassis-Controller (BoardId0)

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT DEWE3 with Controller-V1
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | TRIG_IN0             | Source       | TRIG_OUT0    |
   +----------------------+--------------+--------------+
   | TRIG_IN0             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN1             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN1             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN2             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN2             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN3             | Source       | TRIG_OUT3    |
   +----------------------+--------------+--------------+
   | TRIG_IN3             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN4             | Source       | TRIG_OUT4    |
   +----------------------+--------------+--------------+
   | TRIG_IN4             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN5             | Source       | TRIG_OUT5    |
   +----------------------+--------------+--------------+
   | TRIG_IN5             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN6             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN6             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN7             | Source       | TRIG_OUT7    |
   +----------------------+--------------+--------------+
   | TRIG_IN7             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT0            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT0            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT1            | Source       | TRIG_IN1     |
   +----------------------+--------------+--------------+
   | TRIG_OUT1            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT2            | Source       | TRIG_IN2     |
   +----------------------+--------------+--------------+
   | TRIG_OUT2            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT3            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT3            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT4            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT4            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT5            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT5            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT6            | Source       | TRIG_IN6     |
   +----------------------+--------------+--------------+
   | TRIG_OUT6            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT7            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT7            | Inverted     | False        |
   +----------------------+--------------+--------------+

These are the appropriate TRION-API commands:
(only shown for the first table entry, the rest applies accordingly)

.. code:: c

    DeWeSetParamStruct_str("BoardID0/TRIG_IN0", "Source", "TRIG_OUT0");
    DeWeSetParamStruct_str("BoardID0/TRIG_IN0", "Inverted", "False");
    // ... repeat for all other signals
    // ...
    // ...
    // Then apply the settings using:
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);


On exactly one board, other than the Chassis-Controller

* Set the trigger line TRIG7, Source to low
* Set the trigger line TRIG7, Inverted to false

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Signal               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Low          |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

On all other, but the one chosen for TRIG7 control.
This is important to prevent multiple boards trying to drive the TRIG7-line to different levels.

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Disable      |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

DEWE3 enclosure with Chassis-Controller-V2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In such case, the chassis-controller has to be utilized in the measurement with at least one activated channel,
and shall be operated as measurement master.

On the Chassis-Controller (BoardId0)

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT DEWE3 with Controller-V2
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | SYNC_OUT5            | Source       | Acq_Sync     |
   +----------------------+--------------+--------------+
   | SYNC_OUT5            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | SYNC_OUT7            | Source       | Low          |
   +----------------------+--------------+--------------+
   | SYNC_OUT7            | Inverted     | False        |
   +----------------------+--------------+--------------+

On all other boards

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-OUT
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Disable      |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+


Slave instrument
~~~~~~~~~~~~~~~~

On slave devices using TRION-SYNC-BUS has to be configured too.
All boards have to be configured to slave mode.


Slave board setup "BoardIDX" [for X from 0 to NrOfAvailableBoards]

.. code:: c

    DeWeSetParamStruct_str("BoardIDX/AcqProp", "OperationMode", "Slave");
    // Usually "PosEdge"
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtTrigger", "PosEdge");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtClk", "False");


SYNC-OUT has to be configured:

DEWE2 enclosure
^^^^^^^^^^^^^^^^

On exactly one board

* Set the trigger line TRIG7, Source to hig
* Set the trigger line TRIG7, Inverted to false

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN DEWE2
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | High         |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

On all other, but the one chosen for TRIG7 control.
This is important to prevent multiple boards trying to drive the TRIG7-line to different levels.

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN DEWE2
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Disable      |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

These are the appropriate TRION-API commands:

.. code:: c

    DeWeSetParamStruct_str("BoardID0/Trig7", "Source", "High");
    DeWeSetParamStruct_str("BoardID0/Trig7", "Inverted", "False");
    // Then apply the settings using:
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);

DEWE3 enclosure with Chassis-Controller-V1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

On the Chassis-Controller (BoardId0)

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN DEWE3 with Controller-V1
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | TRIG_IN0             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN0             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN1             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN1             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN2             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN2             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN3             | Source       | TRIG_OUT3    |
   +----------------------+--------------+--------------+
   | TRIG_IN3             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN4             | Source       | TRIG_OUT4    |
   +----------------------+--------------+--------------+
   | TRIG_IN4             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN5             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN5             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN6             | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_IN6             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_IN7             | Source       | TRIG_OUT7    |
   +----------------------+--------------+--------------+
   | TRIG_IN7             | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT0            | Source       | TRIG_IN0     |
   +----------------------+--------------+--------------+
   | TRIG_OUT0            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT1            | Source       | TRIG_IN1     |
   +----------------------+--------------+--------------+
   | TRIG_OUT1            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT2            | Source       | TRIG_IN2     |
   +----------------------+--------------+--------------+
   | TRIG_OUT2            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT3            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT3            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT4            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT4            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT5            | Source       | TRIG_IN5     |
   +----------------------+--------------+--------------+
   | TRIG_OUT5            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT6            | Source       | TRIG_IN6     |
   +----------------------+--------------+--------------+
   | TRIG_OUT6            | Inverted     | False        |
   +----------------------+--------------+--------------+
   | TRIG_OUT7            | Source       | Disable      |
   +----------------------+--------------+--------------+
   | TRIG_OUT7            | Inverted     | False        |
   +----------------------+--------------+--------------+

These are the appropriate TRION-API commands:
(only shown for the first table entry, the rest applies accordingly)

.. code:: c

    DeWeSetParamStruct_str("BoardID0/TRIG_IN0", "Source", "Disable");
    DeWeSetParamStruct_str("BoardID0/TRIG_IN0", "Inverted", "False");
    // ... repeat for all other signals
    // ...
    // ...
    // Then apply the settings using:
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);


On exactly one board, other than the Chassis-Controller

* Set the trigger line TRIG7, Source to high
* Set the trigger line TRIG7, Inverted to false

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Signal               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | High         |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

On all other, but the one chosen for TRIG7 control.
This is important to prevent multiple boards trying to drive the TRIG7-line to different levels.

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Disable      |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

.. code:: c

    DeWeSetParamStruct_str("BoardID0/Trig7", "Source", "High");
    DeWeSetParamStruct_str("BoardID0/Trig7", "Inverted", "False");
    // Then apply the settings using:
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);

DEWE3 enclosure with Chassis-Controller-V2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In such case, the chassis-controller has to be utilized in the measurement with at least one activated channel,
and shall be operated as measurement slave.

On the Chassis-Controller (BoardId0)

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN DEWE3 with Controller-V2
   :widths: 30 30 30

   +----------------------+--------------+-------------------+
   | Target               | Property     | Value             |
   +======================+==============+===================+
   | SYNC_OUT0            | Source       | Disable           |
   +----------------------+--------------+-------------------+
   | SYNC_OUT0            | Inverted     | False             |
   +----------------------+--------------+-------------------+
   | SYNC_OUT5            | Source       | Disable           |
   +----------------------+--------------+-------------------+
   | SYNC_OUT5            | Inverted     | False             |
   +----------------------+--------------+-------------------+
   | SYNC_OUT7            | Source       | High              |
   +----------------------+--------------+-------------------+
   | SYNC_OUT7            | Inverted     | False             |
   +----------------------+--------------+-------------------+
   | TRIG5                | Source       | Acq_SyncStart     |
   +----------------------+--------------+-------------------+
   | TRIG5                | Inverted     | False             |
   +----------------------+--------------+-------------------+
   | TRIG6                | Source       | CLK_EXT_In_Detect |
   +----------------------+--------------+-------------------+
   | TRIG6                | Inverted     | False             |
   +----------------------+--------------+-------------------+

On all other boards

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Signal settings for SYNC-IN
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Target               | Property     | Value        |
   +======================+==============+==============+
   | Trig7                | Source       | Disable      |
   +----------------------+--------------+--------------+
   | Trig7                | Inverted     | False        |
   +----------------------+--------------+--------------+

Acquisition on the Master instrument
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Acquisition on the master instrument has to be started using:

For each (slave) board of the instrument start:

.. code:: c

    for (int BoardID = 1; BoardID < NrOfAvailableBoards; ++BoardID)
    {
        DeWeSetParam_i32(BoardID, CMD_START_ACQUISITION, 0);
    }
    // Then start acquisition on the master board
    DeWeSetParam_i32(0, CMD_START_ACQUISITION, 0);


Please keep in mind:

* Acquisition on slave instruments has to be started before starting acquisition on the master instrument.
* Acquisition on the slave boards has to be started before starting acquisition on the master board.


Acquisition on the Slave instrument
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Acquisition on the slave instruments has to be started using:

For each board of the instrument start:

.. code:: c

    for (int BoardID = 0; BoardID < NrOfAvailableBoards; ++BoardID)
    {
        DeWeSetParam_i32(BoardID, CMD_START_ACQUISITION, 0);
    }


Sync cabling check
~~~~~~~~~~~~~~~~~~

It is possible to check if the sync cables are plugged in correctly.

On each slave instrument use the following commands:

.. code:: c

    int state = 0;
    DeWeGetParam_i32(0, CMD_PXI_LINE_STATE, &state);
    if ((state & PXI_LINE_STATE_TRIG6) == 0)
    {
        // no TRION-SYNC-BUS plugged in on slave instrument
    }


PPS based synchronization methods
---------------------------------

PTP, GPS and IRIG are synchronization methods, where the actual synchronization between
the instruments or 3rd party instruments is achieved by the PPS signal.

After issuing a start-acquisition command the first subsequent PPS will be used as an
internal synchronization signal, and the actual acquisition will start one second after this.

This "one second later" is realized by setting the property "StartCounter" to a value equal
to the "SampleRate" of the TRION-board used as sync-board.

.. code:: c

    //StartCounter has to be set equal to SampleRate (eg 2000)
    DeWeSetParamStruct_str("BoardID0/AcqProp", "StartCounter", "2000");

All PPS based modes share a couple of characteristics, which will show as a repeated pattern in
in the subsequent chapters:

An explicit command is used to force the TRION-board to synchronize itself to the external source.

.. code:: c

    DeWeSetParam_i32(0, CMD_TIMING_STATE, 0);

Before performing an acquisition start the application has to check, whether the instrument managed
to synchronize itself to the external time source.

.. code:: c

    int timing_state;
    DeWeSetParam_i32(0, CMD_TIMING_STATE, &timing_state);

Starting the acquisition needs to be delayed until timing_state has the value TIMINGSTATE_LOCKED.
If any value other than TIMINGSTATE_LOCKED or TIMINGSTATE_NOTRESYNCED is read for multiple times
the application should issue another sync-command.

The acquisition start will be on a full second.

PTP IEEE1588
~~~~~~~~~~~~

Currently TRION-systems can only operate as PTP slaves.

Cabling
^^^^^^^

Connect a PTP Master to the PTP-plug on your TRION-system.
This can either be on the enclosure, or if the enclosure does not provide a PTP plug on a
TRION-TIMING-family board in the first slot of the enclosure.

Slave instrument
^^^^^^^^^^^^^^^^

The TRION board used for the PTP slave role needs at least one synchronous channel activated.
From perspective of the instrument the PTP slave board is considered being a Master board.

PTP slave board setup (instrument master):

.. code:: c

    DeWeSetParamStruct_str("BoardID0/AcqProp/SyncSettings/SyncIn", "Mode", "PTP");
    //StartCounter has to be set equal to SampleRate (eg 2000)
    DeWeSetParamStruct_str("BoardID0/AcqProp", "StartCounter", "2000");
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);

Force synchronization of TRION-board-internal time to provided PTP time:

.. code:: c

    DeWeSetParam_i32(0, CMD_TIMING_STATE, 0);

Slave board setup "BoardIDX" [for X from 1 to NrOfAvailableBoards]:

.. code:: c

    DeWeSetParamStruct_str("BoardIDX/AcqProp", "OperationMode", "Slave");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtTrigger", "PosEdge");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtClk", "False");

Preparing for Acquisition start
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before starting the acquisition it is necessary to see, if the TRION-board
was already able to synchronize itself with the provided PTP time.

.. code:: c

    int timing_state;
    DeWeSetParam_i32(0, CMD_TIMING_STATE, &timing_state);

When timing_state has the value TIMINGSTATE_LOCKED the system is
synchronized to the PTP Master. This may take a couple of seconds.

Acquisition on the instrument
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Acquisition on the instrument has to be started using:

For each (slave) board of the instrument start:

.. code:: c

    for (int BoardID = 1; BoardID < NrOfAvailableBoards; ++BoardID)
    {
        DeWeSetParam_i32(BoardID, CMD_START_ACQUISITION, 0);
    }
    // Then start acquisition on the master board
    DeWeSetParam_i32(0, CMD_START_ACQUISITION, 0);


Please keep in mind:

* Acquisition on the slave boards has to be started before starting acquisition on the master board.
* When using any PPS based synchronization-method like PTP to synchronize multiple instruments their
  respective start may be off by 1 second to each other. The application has to take care to detect this
  and compensate for it.

Property overview
^^^^^^^^^^^^^^^^^
This shows the set of typical properties for PTP-sync-in mode.

.. tabularcolumns:: |p{2.5cm}|p{9cm}|

.. table:: PTP Property overview
   :widths: 10 80

   +-------------------------+------------------------------------------------------+
   | **Property Name**       | **Description**                                      |
   +=========================+======================================================+
   | CorrectionLimit         | Maximum absolute difference between TRION internal   |
   |                         | generated PPS and the PPS derived from the           |
   |                         | input signal before a deviation will be indicated    |
   |                         | with a result-value of TIMINGSTATE_LOCKEDOOR or      |
   |                         | TIMINGSTATE_RELOCKOOR when querying CMD_TIMING_STATE |
   +-------------------------+------------------------------------------------------+
   | Protocol                | Either ETH for Ethernet or UDP_V4 for IPV4 UDP       |
   +-------------------------+------------------------------------------------------+
   | DelayMechanism          | Either edge-to-edge or peer-to-peer                  |
   +-------------------------+------------------------------------------------------+
   | DefaultSettings         | This XML node is no settable property and is used    |
   |                         | API internally.                                      |
   |                         | This element shall be ignored by applications        |
   +-------------------------+------------------------------------------------------+

GPS
~~~

Cabling
^^^^^^^

Connect a GPS antenna to the GPS connector on your TRION-System..
This can either be on the enclosure, or if the enclosure does not provide a GPS connector on a
TRION-TIMING-family board in the first slot of the enclosure.

Slave instrument
^^^^^^^^^^^^^^^^

The TRION board used for the GPS sync-in role needs at least one synchronous channel activated.
From perspective of the instrument the GPS-synced board is considered being a Master board.

GPS board setup (instrument master):

.. code:: c

    DeWeSetParamStruct_str("BoardID0/AcqProp/SyncSettings/SyncIn", "Mode", "GPS");
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);

Force synchronization of TRION-board-internal time to provided GPS time:

.. code:: c

    DeWeSetParam_i32(0, CMD_TIMING_STATE, 0);

Slave board setup "BoardIDX" [for X from 1 to NrOfAvailableBoards]:

.. code:: c

    DeWeSetParamStruct_str("BoardIDX/AcqProp", "OperationMode", "Slave");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtTrigger", "PosEdge");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtClk", "False");

Preparing for Acquisition start
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before starting the acquisition it is necessary to see, if the TRION-board
was already able to synchronize itself with the provided GPS time.

.. code:: c

    int timing_state;
    DeWeSetParam_i32(0, CMD_TIMING_STATE, &timing_state);

When timing_state has the value TIMINGSTATE_LOCKED the system is
synchronized to the GPS time. This may take a couple of seconds.

Acquisition on the instrument
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Acquisition on the instrument has to be started using:

For each (slave) board of the instrument start:

.. code:: c

    for (int BoardID = 1; BoardID < NrOfAvailableBoards; ++BoardID)
    {
        DeWeSetParam_i32(BoardID, CMD_START_ACQUISITION, 0);
    }
    // Then start acquisition on the master board
    DeWeSetParam_i32(0, CMD_START_ACQUISITION, 0);


Please keep in mind:

* Acquisition on the slave boards has to be started before starting acquisition on the master board.
* When using any PPS based synchronization-method like GPS to synchronize multiple instruments their
  respective start may be off by 1 second to each other. The application has to take care to detect this
  and compensate for it.

Property overview
^^^^^^^^^^^^^^^^^
This shows the set of typical properties for GPS-sync-in mode.

.. tabularcolumns:: |p{2.5cm}|p{9cm}|

.. table:: GPS Property overview
   :widths: 10 80

   +-------------------------+------------------------------------------------------+
   | **Property Name**       | **Description**                                      |
   +=========================+======================================================+
   | CorrectionLimit         | Maximum absolute difference between TRION internal   |
   |                         | generated PPS and the PPS derived from the           |
   |                         | input signal before a deviation will be indicated    |
   |                         | with a result-value of TIMINGSTATE_LOCKEDOOR or      |
   |                         | TIMINGSTATE_RELOCKOOR when querying CMD_TIMING_STATE |
   +-------------------------+------------------------------------------------------+
   | CableLengthCompensation | This property can be used to compensate for signal   |
   |                         | propagation times on long cables between the GPS.    |
   |                         | antenna and the TRION system.                        |
   |                         | For reasonable short cables this value can be left   |
   |                         | at 0ms.                                              |
   +-------------------------+------------------------------------------------------+
   | PropCoeff               | This is an internally used parameter of the pi-      |
   |                         | controller and should be left alone by applications. |
   +-------------------------+------------------------------------------------------+
   | IntCoeff                | This is an internally used parameter of the pi-      |
   |                         | controller and should be left alone by applications. |
   +-------------------------+------------------------------------------------------+
   | ControlValue            | This is an internally used parameter of the pi-      |
   |                         | controller and should be left alone by applications. |
   +-------------------------+------------------------------------------------------+
   | DefaultSettings         | This XML node is no settable property and is used    |
   |                         | API internally.                                      |
   |                         | This element shall be ignored by applications        |
   +-------------------------+------------------------------------------------------+

IRIG
~~~~

Cabling
^^^^^^^

Connect an IRIG-Signal to the IRIG-connector (typically BNC) on the TRION-system.
This can either be on the enclosure, or if the enclosure does not provide an IRIG connector on a
TRION-TIMING-family board in the first slot of the enclosure.

Slave instrument
^^^^^^^^^^^^^^^^

The TRION board used for the IRIG sync-in role needs at least one synchronous channel activated.
From perspective of the instrument the IRIG-synced board is considered being a Master board.

IRIG board setup (instrument master):

.. code:: c

    //Set mode
    DeWeSetParamStruct_str("BoardID0/AcqProp/SyncSettings/SyncIn", "Mode", "IRIG");
    //Select the format
    DeWeSetParamStruct_str("BoardID0/AcqProp/SyncSettings/SyncIn", "IRIGFormat", "IRIG_B_DC");
    DeWeSetParam_i32(0, CMD_UPDATE_PARAM_ALL, 0);

Force synchronization of TRION-board-internal time to provided IRIG time:

.. code:: c

    DeWeSetParam_i32(0, CMD_TIMING_STATE, 0);

Slave board setup "BoardIDX" [for X from 1 to NrOfAvailableBoards]:

.. code:: c

    DeWeSetParamStruct_str("BoardIDX/AcqProp", "OperationMode", "Slave");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtTrigger", "PosEdge");
    DeWeSetParamStruct_str("BoardIDX/AcqProp", "ExtClk", "False");

Preparing for Acquisition start
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before starting the acquisition it is necessary to see, if the TRION-board
was already able to synchronize itself with the provided IRIG time.

.. code:: c

    int timing_state;
    DeWeSetParam_i32(0, CMD_TIMING_STATE, &timing_state);

When timing_state has the value TIMINGSTATE_LOCKED the system is
synchronized to the IRIG source. This may take a couple of seconds.

Acquisition on the instrument
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Acquisition on the instrument has to be started using:

For each (slave) board of the instrument start:

.. code:: c

    for (int BoardID = 1; BoardID < NrOfAvailableBoards; ++BoardID)
    {
        DeWeSetParam_i32(BoardID, CMD_START_ACQUISITION, 0);
    }
    // Then start acquisition on the master board
    DeWeSetParam_i32(0, CMD_START_ACQUISITION, 0);


Please keep in mind:

* Acquisition on the slave boards has to be started before starting acquisition on the master board.
* When using any PPS based synchronization-method like IRIG to synchronize multiple instruments their
  respective start may be off by 1 second to each other. The application has to take care to detect this
  and compensate for it.

Property overview
^^^^^^^^^^^^^^^^^
This shows the set of typical properties for IRIG-sync-in mode.

.. tabularcolumns:: |p{2.5cm}|p{9cm}|

.. table:: IRIG Property overview
   :widths: 10 80

   +-------------------------+------------------------------------------------------+
   | **Property Name**       | **Description**                                      |
   +=========================+======================================================+
   | IRIGFormat              | This is a composite of the IRIG code and the used    |
   |                         | modulation.                                          |
   |                         | eg IRIG_B_DC, IRIG_A_AC                              |
   +-------------------------+------------------------------------------------------+
   | CorrectionLimit         | Maximum absolute difference between TRION internal   |
   |                         | generated PPS and the PPS derived from the           |
   |                         | input signal before a deviation will be indicated    |
   |                         | with a result-value of TIMINGSTATE_LOCKEDOOR or      |
   |                         | TIMINGSTATE_RELOCKOOR when querying CMD_TIMING_STATE |
   +-------------------------+------------------------------------------------------+
   | CableLengthCompensation | This property can be used to compensate for signal   |
   |                         | propagation times on long cables between the GPS.    |
   |                         | antenna and the TRION system.                        |
   |                         | For reasonable short cables this value can be left   |
   |                         | at 0ms.                                              |
   +-------------------------+------------------------------------------------------+
   | PropCoeff               | This is an internally used parameter of the pi-      |
   |                         | controller and should be left alone by applications. |
   +-------------------------+------------------------------------------------------+
   | IntCoeff                | This is an internally used parameter of the pi-      |
   |                         | controller and should be left alone by applications. |
   +-------------------------+------------------------------------------------------+
   | ControlValue            | This is an internally used parameter of the pi-      |
   |                         | controller and should be left alone by applications. |
   +-------------------------+------------------------------------------------------+
   | DefaultSettings         | This XML node is no settable property and is used    |
   |                         | API internally.                                      |
   |                         | This element shall be ignored by applications        |
   +-------------------------+------------------------------------------------------+

Example
~~~~~~~~~~~~~

PTP, GPS and IRIG are similar to setup.

The following example covers the basic setup and acquisition start sequence.

.. literalinclude:: ../../trion/CXX/synchronization/synchronization_ptp.cpp
    :caption: PTP slave setup example
    :language: c++
    :linenos:
    :lines: 9-
