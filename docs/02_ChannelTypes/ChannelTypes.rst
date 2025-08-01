Channel Types
=============

TRION and TRION3 modules support multiple different channel types.
These channels further support different measurement modes.

The following sections describe the different channels and provide an
extensive list of the available modes.


As an SDK developer you do not have to create your own database
of boards and their channels and modes. A boards abilities
are reported by its *Properties.xml* document.
It can be requested during runtime for every device and is described
in more detail in the XML Reference chapter.



Analog Channels (AI)
--------------------

Technically the path for analog measurement data consist of three
distinct parts on TRION™-boards.

1. The analog input-path performing the signal-conditioning
2. The A/D conversion
3. Digital data post-processing


The TRION-API however encapsulates the exact details of this chain
in a way, so that the various differences in implementation depending
on the exact board-type are not visible above the interface. This allows
an application to choose a rather generic approach toward analog
channels in general, and frees the application developer from the need
to develop against a specific board-type. The property-set for analog
channels basically describes the whole chain from signal-conditioning
to postprocessing in an uniform way.

Scaling principles
~~~~~~~~~~~~~~~~~~

All information regarding the used gain and offset(s) are abstracted from
the user with the more abstract **Range** and **InputOffset** information.
The API will always setup the hardware in a way, so that the digital
values map in their full range to the selected **Range**, in the approbiate
unit.

The following example shows the scaling in 32, 24 and 16 bit

.. table:: Scaling Examples
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Selecte Range        | | 0xffffffff | | 0x7fffffff |
   |                      | | 0xffffff   | | 0x7fffff   |
   |                      | | 0xffff     | | 0x7fff     |
   +======================+==============+==============+
   | 10V (= -10V .. 10V)  | -10V         | 10V          |
   +----------------------+--------------+--------------+
   | -5 .. 10V            | -5V          | 10V          |
   +----------------------+--------------+--------------+
   | 0 .. 100000 Ohm      | 0V           | 100000 Ohm   |
   +----------------------+--------------+--------------+
   | 3 .. 10 mV/mA        | 3 mV/mA      | 10 mV/mA     |
   +----------------------+--------------+--------------+
   | -10 .. 5A            | -10A         | 5A           |
   +----------------------+--------------+--------------+
   | -10 .. 0V            | -10V         | 0V           |
   +----------------------+--------------+--------------+



Channel Properties
~~~~~~~~~~~~~~~~~~

The property set for analog channels is organized beneath
various measurement modes. Under each mode a selected set of
configurable parameters exists. Not all properties are available for
all TRION™-boards. But for each mode a minimum-set of obvious common
configuration items can be enumerated. This chapter will provide an
overview over all currently used properties, sorted by the currently
supported measurement modes, split into the parameters available on
all TRION analog channels, and those available on specific
boards only.

Each property has a list of potential allowed settings. The list has
a minimal size of one entry, if the property has a use within the
given mode.
For non-trivial measurement modes some of the properties have non-trivial
constraints. Those constraints are derivable from the
board-properties-xml-document.

The application does not strictly need to pre-validate properties
against those constraints.
The API will usually adjust set property-values to satisfy those
constraints, and will issue a WARNING-Level errorcode to indicate
this to the application. In such a case, it would be a viable strategy
to invoke the property-getter to retrieve the adjusted value for further
application-processing. However: As this approach might not be suitable
for all types of applications an exhaustive overview over those
property-constraints, and how to validate them on application level.



General Attributes
~~~~~~~~~~~~~~~~~~

Default Attribute
^^^^^^^^^^^^^^^^^

This indicates the index of the default-setting for the property.
The API will set all settings to their default-values, when the
mode is switched.

In the following code block *Default = "2"* selects
<ID2>10</ID2> as its default value.

.. code-block:: XML
    :caption: Default attribute

    <Range
        Default = "2">
        <ID0>100</ID0>
        <ID1>30</ID1>
        <ID2>10</ID2>
        <ID3>3</ID3>
        <ID4>1</ID4>
        <ID5>0.1</ID5>
    </Range>



ProgMin ProgMax Attribute
^^^^^^^^^^^^^^^^^^^^^^^^^

Some properties can be programmable in a given interval. If this
is the case for a given Property, it is indicated by presence of
the two attributes ProgMin and ProgMax.
Both attributes are always in the same unit as the underlying
property.

.. code-block:: XML
    :caption: ProgMin ProgMax attribute

    <Range
        ProgMax = "100"
        ProgMin = "-100">
    </Range>



Unit Attribute
^^^^^^^^^^^^^^

Generally indicates the Unit used with the given property. This
includes all fixed list-entries of the list, as well as the unit
for ProgMin and ProgMax if given.
In certain modes like Bridge for example, the attribute unit can
also work a distinction-predicate, if one property with all its
definition may exist multiple times. In bridge-mode this would be
for example the case for the property “Range”, which exists once
with unit = “mV/V” and once with the unit = “mV/mA”.

.. code-block:: XML
    :caption: Unit attribute

    <Range
        Unit = "V">
    </Range>

Channel Specific commands
~~~~~~~~~~~~~~~~~~~~~~~~~

Additionally to the properties listed, analog channels support a couple
of operations, that are addressed via string-commands, and executed
by the API or the hardware.
Availability of those commands can depend on the type of TRION™-board and
the mode used.

The available options are listed as **ChannelFeatures** on channel-level, and
on mode level.
The applicable set in a specific mode is the superset of those two table. This
is all entries from channel-level **plus** all entries from mode-level.
The order of entries is arbitrary, and has no underlying meaning.

.. table:: possible channel features
   :widths: 30 30

   +----------------------+----------------------------------------------------------------------+
   | Name (entry in xml)  | Description                                                          |
   +======================+======================================================================+
   | SupportTEDS          | TedsAccess commands can be executed on this channel.                 |
   +----------------------+----------------------------------------------------------------------+
   | AmplifierZero        | Amplifier Offset compensation                                        |
   |                      | This operation performs a brief measurment with input set to short   |
   |                      | and updates the internal e2prom information to compensate for any    |
   |                      | offset.                                                              |
   |                      | This command is triggered, when executing "Self Test"->"Auto Zero"   |
   |                      | in **DewetronExplorer**                                              |
   +----------------------+----------------------------------------------------------------------+
   | SensorUnbalance      | also refered to as **Sensor Offset** or **Bridge Balance**           |
   |                      | for details see: `Sensor Offset determination`_                      |
   |                      | This command averages the current input signal over a brief          |
   |                      | period of time (usually 100ms).                                      |
   |                      | At the end the average is reported back to the application.          |
   |                      | It is at the discretion of the application to apply this information |
   |                      | to the **InputOffset** to request compensation                       |
   +----------------------+----------------------------------------------------------------------+


Sensor Offset determination
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Command: "sensoroffset"

.. note:: the commands are case insensitive

Example Usage:
    | Execute command: DeweSetParamStruct_str("BoardId0/AI1", **"SensorOffset"**, "100 msec");
    | Query result: DeweGetParamStruct_str("BoardId0/AI1", **"SensorOffset"**, var, sizeof(var));

This operation starts a brief internal averaging of the current signal, over a given period (defualt 100ms).
If the currently measured average is clipped at max or min of range, the amplifier settings will escalate
to higher ranges up to 2 times, and try to get an unclipped average. After the averaging, API will restore
the original range setting.
This way offsets, that exceed the currently set range can be measured, and subsquentually be compensate for.

After execution the application needs to query the result.

.. note:: If the signal keeps clipping after 2 range escalations,
    the operation is considered to have failed. This is done to prevent
    accuracy loss due to an unfafourable rate between range and applied offset.
    For example a offset of 5V on a Range of 0.1 V would neet the amplifier
    to operate in a range higher than 5 V. So effectifly only the accuracy
    of the higher range is applicable, but the range of 0.1V may suggest otherwise.

Example:
    configured InputOffset = 0mV/V
    configured Range = 10mV/V
    Offset of sensor = 12mV/V

    With such a setup, the signal would constantly clip at 10mV/V.
    When executing the command "SensorOffset", API will internally start an averaging process.
    As the signal clips, it will escalate the Range to the next higher range (TRION-board type specific),
    and will be able to measure 12mV/V.
    This will be stored as queryable result, the range will be reset to 10mV/V

This is also refered to as **Bridge Balance**, albeit only being the part of determining the current offset.
The operation itself does **not** apply any correction value.

Result document
^^^^^^^^^^^^^^^

For each channel the measured average will be found in the node "Offset".
This offset will need to be applied to the property **InputOffset** by the application.

.. code-block:: XML
    :caption: Sensor Offset result document

        <SensorOffset>
            <BoardId0 ID = "BoardId0" BrdName = "TRION-2402-MULTI-8-L0B" Slot = "1" SerialNumber = "12345678" Passed = "True">
                <Check Target = "AI0" Type = "Sensor Balance Test" Test = "BalanceCheck" Passed = "True">
                    <Averaging>100msec</Averaging>
                    <Date>03.07.2025</Date>
                    <Time>15:24:15</Time>
                    <BaseBoardTemp>25</BaseBoardTemp>
                    <ConPanelTemp>0</ConPanelTemp>
                    <ID0 Device = "SensorBalance" Range = "1000 mV/V" Passed = "True">
                        <Offset Unit = "mV/V" Passed = "True">0.000000</Offset>
                    </ID0>
                </Check>
                .......
                <Check Target = "AI7" Type = "Sensor Balance Test" Test = "BalanceCheck" Passed = "True">
                    <Averaging>100msec</Averaging>
                    <Date>03.07.2025</Date>
                    <Time>15:24:15</Time>
                    <BaseBoardTemp>25</BaseBoardTemp>
                    <ConPanelTemp>0</ConPanelTemp>
                    <ID0 Device = "SensorBalance" Range = "1000 mV/V" Passed = "True">
                        <Offset Unit = "mV/V" Passed = "True">0.000000</Offset>
                    </ID0>
                </Check>
            </BoardId0>
        </SensorOffset>



Voltage Mode
~~~~~~~~~~~~

On most TRION™-boards the modes “Voltage” and “Calibration” are
very similar. The Calibration mode usually is more restrictive on
the Range-property, but less restrictive on the Input-Types. The
Calibration Mode usually allows for signal routing to onboard
calibration-sources that have barely a use in normal measurement.
On the range-side it usually does not allow to use a free
programmable value.

.. code-block:: XML
    :caption: Voltage mode element

    <Mode Mode = "Voltage">
        <Range>..</Range>
        <InputOffset>..</InputOffset>
        <Excitation>..</Excitation>
        <LPFilter_Type>..</LPFilter_Type>
        <LPFilter_Order>..</LPFilter_Order>
        <LPFilter_Val>..</LPFilter_Val>
        <HPFilter_Type>..</HPFilter_Type>
        <HPFilter_Order>..</HPFilter_Order>
        <HPFilter_Val>..</HPFilter_Val>
        <InputType>..</InputType>
        <IIRFilter_Type>..</IIRFilter_Type>
        <IIRFilter_Order>..</IIRFilter_Order>
        <IIRFilter_Val>..</IIRFilter_Val>
        <HPIIRFilter_Type>..</HPIIRFilter_Type>
        <HPIIRFilter_Order>..</HPIIRFilter_Order>
        <HPIIRFilter_Val>..</HPIIRFilter_Val>
        <InputImpedance>..</InputImpedance>
        <ChannelFeatures>..</ChannelFeatures>
        <TEDSOptions>..</TEDSOptions>
    </Mode>


Range Attribute
^^^^^^^^^^^^^^^
Unit: V

Sets the input-range of the amplifier and post processing chain,
usually in V. In terms of Non-TRION™-signal conditioners this is
closely related to the used gain.


InputOffset Attribute
^^^^^^^^^^^^^^^^^^^^^
Unit: V

This property is often used synonymous to “Sensor-Offset”. It's
main use is to shift the virtual 0 V by a given value. Due to
various physical effects any non-ideal sensor usually has a bias.
With the property input-offset API can be setup to compensate for
this bias.


InputType Attribute
^^^^^^^^^^^^^^^^^^^
Unit: N/A

This property indicates the possible input-type-configurations.
For example: Single-Ended, Differential
.. note:: Some TRION™-boards only support one non-switchable input type.
    In this case the property still will be present, but only feature
    one entry.


Excitation Attribute
^^^^^^^^^^^^^^^^^^^^
Unit: either V, mA or both

This property allows to configure or disable the excitation
(e.g. for sensor-supply).





Current Mode
~~~~~~~~~~~~


Resistance Mode
~~~~~~~~~~~~~~~


Bridge Mode
~~~~~~~~~~~

Specific TRION-boards offer a native "Bridge" mode, usually featuring
support for full-, half- and quarter-bridge configurations with internal
bridge completion.

Bridge-measurement can either be driven by voltage or by current excitation.
As some properties are directly depending on this circumstance the bridge-mode-
subtree is more complex than the voltage-mode subtree, showing multiple
instances of some properties.

In bridge-mode the Excitation property should be the first one to be set,
as the validity of many other properties directly depends on this information.

.. code-block:: XML
    :caption: Bridge mode element

    <Mode Mode = "Bridge">
        <Range>..</Range>
        <Range>..</Range>
        <InputOffset>..</InputOffset>
        <InputOffset>..</InputOffset>
        <Excitation>..</Excitation>
        <Excitation>..</Excitation>
        <ShuntTarget>..</ShuntTarget>
        <ShuntTarget>..</ShuntTarget>
        <LPFilter_Type>..</LPFilter_Type>
        <LPFilter_Order>..</LPFilter_Order>
        <LPFilter_Val>..</LPFilter_Val>
        <HPFilter_Type>..</HPFilter_Type>
        <HPFilter_Order>..</HPFilter_Order>
        <HPFilter_Val>..</HPFilter_Val>
        <IIRFilter_Type>..</IIRFilter_Type>
        <IIRFilter_Order>..</IIRFilter_Order>
        <IIRFilter_Val>..</IIRFilter_Val>
        <HPIIRFilter_Type>..</HPIIRFilter_Type>
        <HPIIRFilter_Order>..</HPIIRFilter_Order>
        <HPIIRFilter_Val>..</HPIIRFilter_Val>
        <InputImpedance>..</InputImpedance>
        <InputType>..</InputType>
        <BridgeRes>..</BridgeRes>
        <BridgeRes>..</BridgeRes>
        <BridgeRes>..</BridgeRes>
        <ShuntType>..</ShuntType>
        <ShuntResistance>..</ShuntResistance>
        <ChannelFeatures>..</ChannelFeatures>
        <TEDSOptions>..</TEDSOptions>
    </Mode>

Excitation Attribute
^^^^^^^^^^^^^^^^^^^^
Unit: either V, mA

This property allows to configure the excitation.
As many other properties directly depend on the unit
of the excitation it is the first property that should
be set.

Range Attribute
^^^^^^^^^^^^^^^
Unit: either mV/V, mV/mA

.. warning::
    Due to the wide possible electrical range that can be covered
    by simply setting the Excitation to either a very low or very
    high value, an application either needs to follow the :ref:`more advanced
    constraint evaluation <range_calculation_bridge>`, or always
    requery the Range after changing a related attribute from the API,
    as it will perform automatic corrections to the range, if any
    constraint is violated.

InputOffset Attribute
^^^^^^^^^^^^^^^^^^^^^
Unit: either mV/V, mV/mA

This property is often used synonymous to “Sensor-Offset”. It's
main use is to shift the virtual 0 mV/V or 0mV/mA by a given value.
Due to various physical effects any non-ideal sensor usually has a bias.
With the property input-offset API can be setup to compensate for
this bias.

.. _bridge_res_input_type:

InputType Attribute
^^^^^^^^^^^^^^^^^^^
Unit: N/A

In bridge-mode this property indicates the possible input-path-configurations.

This usually covers the possible bridge-configurations
(full, half, quarter), the wiring configuration (3, 4 or 5-wire)
as well as internal routing types used to facilitate diagnostic
features without the need to change the mode (like applying a
virtual short to sense the amplifier offset, or measuring the line
voltage drop).

BridgeRes Attribute
^^^^^^^^^^^^^^^^^^^
Unit: N/A

This attribute allows to configure the nominal resistance value of the
used strain gauge. Which table is applicable is selected via the
:ref:`input type <bridge_res_input_type>`. On configurations with internal
completion this configures the used completion resistance.

ShuntType Attribute
^^^^^^^^^^^^^^^^^^^
Unit: N/A

This property is used together with the
:ref:`ShuntResistance <bridge_shunt_resistance>` property
to activate an internal shunt-resistor for a shunt-calibration.

.. _bridge_shunt_resistance:

ShuntResistance Attribute
^^^^^^^^^^^^^^^^^^^^^^^^^
Unit: Ohm

Selects the used shunt resistor for shunt-calibration.

.. note::
    Depending on the TRION board this may be realized via a
    :ref:`ShuntTarget <bridge_shunt_target>`, and therefore
    not a user selectable value.

.. _bridge_shunt_target:

ShuntTarget Attribute
^^^^^^^^^^^^^^^^^^^^^
Unit: mV/V

Some TRION-boards allow to set a specified target value for
shunt-calibration.
The API will then calculate a virtual shunt-resistance value
considering and compensating the linear resistance drop and apply
it's value when "ShuntType" is set to "Internal".

InputImpedance Attribute
^^^^^^^^^^^^^^^^^^^^^^^^
Unit: N/A

Some TRION-boards allow to set the input-impedance to a high-impedance path
if certain hardware-specific requirements are met.

Potentiometer Mode
~~~~~~~~~~~~~~~~~~

The "potentiometer"-mode technically is a half-bridge, where the
hardware is configured to scale to a percent full-scale (default 0..100%).

.. code-block:: XML
    :caption: Potentiometer mode element

    <Mode Mode = "Bridge">
        <Range>..</Range>
        <Excitation>..</Excitation>
        <ShuntTarget>..</ShuntTarget>
        <ShuntTarget>..</ShuntTarget>
        <LPFilter_Type>..</LPFilter_Type>
        <LPFilter_Order>..</LPFilter_Order>
        <LPFilter_Val>..</LPFilter_Val>
        <HPFilter_Type>..</HPFilter_Type>
        <HPFilter_Order>..</HPFilter_Order>
        <HPFilter_Val>..</HPFilter_Val>
        <IIRFilter_Type>..</IIRFilter_Type>
        <IIRFilter_Order>..</IIRFilter_Order>
        <IIRFilter_Val>..</IIRFilter_Val>
        <HPIIRFilter_Type>..</HPIIRFilter_Type>
        <HPIIRFilter_Order>..</HPIIRFilter_Order>
        <HPIIRFilter_Val>..</HPIIRFilter_Val>
        <InputImpedance>..</InputImpedance>
        <InputType>..</InputType>
        <ChannelFeatures>..</ChannelFeatures>
        <TEDSOptions>..</TEDSOptions>
    </Mode>

RTD-Temperature Mode
~~~~~~~~~~~~~~~~~~~~


IEPE Mode
~~~~~~~~~


ExcCurrentMonitor Mode
~~~~~~~~~~~~~~~~~~~~~~


ExcVoltMonitor Mode
~~~~~~~~~~~~~~~~~~~


Calibration Mode
~~~~~~~~~~~~~~~~


MSI Modes
~~~~~~~~~


CAN Mode
~~~~~~~~

.. _advanced_constraints:

Advanced Constraints
~~~~~~~~~~~~~~~~~~~~

In Voltage-measurement mode, the exact amplifier-setting only
depends on the range-property and the input-offset-attribute.
In the non-trivial measurement modes the amplifier-setting are affected
by more than those two logical parameters. A typical example would be
bridge-mode, where the amplifier settings are affected by logical range,
input-offset and excitation.

While it would be possible to limit each property in a way, so that all
possible combination would yield a legal amplifier setup, it would hurt
the versatility of the single properties.

This chapter will reveal the dependencies of the various parameters in
the different modes, as well as the formulas used to evaluate versus
the given constraints.


Almost all constraints affect the range-property.
Each range-property-node holds several attributes relevant for
constraints checking:

AmplRangeMax, AmplRangeMin, AmplRangeUnit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
These attributes indicate the legal maximum and minimum values for the
final amplifier-setup. The AmplRangeUnit is always in volt [V].

MaxInputOffset
^^^^^^^^^^^^^^
Maximum allowed input-offset. This is always given in %-of-range.
On most TRION™-boards this is +/-200%, unless already in the highest
possible range, where usually no further input-offset is allowed.

MaxOutputOffset
^^^^^^^^^^^^^^^
The output-offset is the virtual offset introduce by asymmetrical custom
ranges. For example a custom range of 0..10V would yield a output-offset
of -100%. The limit for the output-offset usually is +/-150%







Range calculation
~~~~~~~~~~~~~~~~~


As the TRION-API supports asymmetrical custom ranges, the range is split
into RangeMin and RangeMax. RangeMin is the lower value of a given
range-span, whereby RangeMax is the upper value.

.. tabularcolumns:: |p{3cm}|p{3cm}|p{3cm}|

.. table:: Range Examples
   :widths: 30 30 30

   +----------------------+--------------+--------------+
   | Range                | RangeMin     | RangeMax     |
   +======================+==============+==============+
   | 10V (= -10V .. 10V)  | -10V         | 10V          |
   +----------------------+--------------+--------------+
   | -5 .. 10V            | -5V          | 10V          |
   +----------------------+--------------+--------------+
   | 0 .. 10V             | 0V           | 10V          |
   +----------------------+--------------+--------------+
   | 3 .. 10V             | 3V           | 10V          |
   +----------------------+--------------+--------------+
   | -10 .. 5V            | -10V         | 5V           |
   +----------------------+--------------+--------------+
   | -10 .. 0V            | -10V         | 0V           |
   +----------------------+--------------+--------------+

This is the range (in [V]), the amplifier-path has to be set to, to satisfy
the promise, that the interval RangeMin..RangeMax is covered by the
raw-value-full-scale.


HWRangeMin, HWRangeMax, HWInputOffset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
As the properties Range (RangeMin..RangeMx) and InputOffset are always in
logical units (eg Ohms for resistance mode), a intermediate step of conversion
is necessary, to translate them to the underlying voltage-measurements.
The HWRangeMin/Max and InputOffset are used subsequently to calculate the
AmplifierRange. The main-purpose of those values is to keep the calculation
comprehensible.


Amplifier Range
^^^^^^^^^^^^^^^
The result of the calculated AmplifierRange must always satisfy following
condition:

    .. math:: AmplRangeMin[V] \leq AmplifierRange[V] \leq AmplRangeMax


Voltage Mode, Calibration Mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Depending on properties: Range, InputOffset

    .. math:: HWRangeMin[V] = RangeMin[V]
    .. math:: HWRangeMax[V] = RangeMax[V]
    .. math:: HWInputOffset[V] = InputOffset[V]
    .. math:: AmplifierRange[V] = max(abs(HWRangeMin+HWInputOffset), \\ abs(HWRangeMax+HWInputOffset))


Resistance Mode
^^^^^^^^^^^^^^^
Depending on properties: Range, InputOffset, Excitation

    .. math:: HWRangeMin[V] = RangeMin[\Omega] * Excitation[A]
    .. math:: HWRangeMax[V] = RangeMax[\Omega] * Excitation[A]
    .. math:: HWInputOffset[V] = InputOffset[\Omega] * Excitation[A]
    .. math:: AmplifierRange[V] = max(abs(HWRangeMin+HWInputOffset), \\ abs(HWRangeMax+HWInputOffset))

.. _range_calculation_bridge:

Bridge Mode
^^^^^^^^^^^
Depending on properties: Range, InputOffset, Excitation

Note: Excitation and Range are related.


.. tabularcolumns:: |p{2.5cm}|p{2.5cm}|

.. table:: Bridge Range Examples
   :widths: 20 20

   +--------------------+---------------------+
   | Excitation Unit    | Range Unit          |
   +====================+=====================+
   | mA                 | mV/mA               |
   +--------------------+---------------------+
   | V                  | mV/mV               |
   +--------------------+---------------------+

The calculation is shown for mA-unit. Formulas also apply for V-excitations

    .. math:: HWRangeMin[V] = \frac{RangeMin[\frac{mV}{mA}] * Excitation[mA]}{1000}
    .. math:: HWRangeMax[V] = \frac{RangeMax[\frac{mV}{mA}] * Excitation[mA]}{1000}
    .. math:: HWInputOffset[V] = \frac{InputOffset[\frac{mV}{mA}] * Excitation[mA]}{1000}
    .. math:: AmplifierRange[V] = max(abs(HWRangeMin+HWInputOffset), \\ abs(HWRangeMax+HWInputOffset))


Potentiometer Mode
^^^^^^^^^^^^^^^^^^
Depending on properties: Range, InputOffset, Excitation

    .. math:: HWRangeMin[V] = \frac{RangeMin[\%] * Excitation[V]}{100}-\frac{Excitation[V]}{2}
    .. math:: HWRangeMax[V] = \frac{RangeMax[\%] * Excitation[V]}{100}-\frac{Excitation[V]}{2}
    .. math:: HWInputOffset = InputOffset[\%] * Excitation[V]
    .. math:: AmplifierRange[V] = max(abs(HWRangeMin+HWInputOffset), \\ abs(HWRangeMax+HWInputOffset))



RTD-Temperature Mode
^^^^^^^^^^^^^^^^^^^^

TBD


Current Mode, ExcCurrentMonitor Mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Depending on properties: Range, ShuntRes

    .. math:: HWRangeMin[V] = RangeMin[A] * ShuntRes[\Omega]
    .. math:: HWRangeMax[V] = RangeMax[A] * ShuntRes[\Omega]
    .. math:: HWInputOffset[V] = InputOffset[A] * ShuntRes[\Omega]
    .. math:: AmplifierRange[V] = max(abs(HWRangeMin+HWInputOffset), \\ abs(HWRangeMax+HWInputOffset))



Analog Out Channels
-------------------


MonitorOutput Mode
~~~~~~~~~~~~~~~~~~


MathOutput Mode
~~~~~~~~~~~~~~~


ConstOutput Mode
~~~~~~~~~~~~~~~~


FunctionGenerator Mode
~~~~~~~~~~~~~~~~~~~~~~


StreamOutput Mode
~~~~~~~~~~~~~~~~~




Counter Channels
----------------

Events Mode
~~~~~~~~~~~


Period Mode
~~~~~~~~~~~


PulseWidth Mode
~~~~~~~~~~~~~~~


TwoPulseEdgeSep Mode
~~~~~~~~~~~~~~~~~~~~


Subcounter Period Mode
~~~~~~~~~~~~~~~~~~~~~~


Subcounter TwoPulseEdgeSep Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Subcounter Frequency Mode
~~~~~~~~~~~~~~~~~~~~~~~~~




Digital Channels
----------------


DI Mode
~~~~~~~


DIO Mode
~~~~~~~~




CAN Channels
------------


HighSpeed Mode
~~~~~~~~~~~~~~


CANFD Channels
--------------


Currently not supported




RS485 Channels
--------------


Raw Mode
~~~~~~~~


NMEA Mode
~~~~~~~~~

