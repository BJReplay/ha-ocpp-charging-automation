# ha-ocpp-charging-automation
Collection of Home Assistant OCPP Charging Automation components for use with IAMMETER (or other) consumption monitors

## Quick start

These are not full instructions, just notes for those that are keen before I get a chance to write this up properly.  Help examples (Sensors / Inputs) are all in the SCREENSHOTS folder.

Create a Site Power sensor that is negative for power flowing to the grid, in watts.  This is the usual sign for sensors such as the [IAMMETER 3050T](https://www.iammeter.com/products/3phase-meter-3050t) set up to measure flow from the grid as a positive number, or Tesla Powerwall products correctly configured.  Note that Tesla Powerwall products need to be converted from kW to W as shown in the sample template.

Create a Time Throttled Site Power Sensor for Grid Power to trigger adjustment of the power sent to the charger - I've chosen 30 seconds as this works pretty well for me - but you could choose 60 seconds, or 15 seconds, but I wouldn't choose less than 15 seconds - the charger / car combination doesn't react that quickly.

Create a Charging Current Sensor that reports the current being delivered to the car - this helps damp the changes in current sent to the car to adjust for differences between the offered and accepted current.  If you have an IAMMETER current meter set up for measuring delivered charging current, use it in this template, otherwise just use the offered and accepted current as per the example screenshots.

Create a Voltage Source sensor that provides the Voltage to correctly adjust the spare grid power by the voltage to calculate the amperage to offer to the car to charge off excess solar.  This can be a cascading list of more reliable to less reliable voltage sources (e.g. IAMMETER down to smart plugs to a constant) to ensure that there is always a figure to use, even if one of the sources drops out during charging.

Create a Charger Serial input text helper that you save your OCPP charger ID in.  This enables the automations to work without significant modification to the code, as the charger ID is read from this variable.

Create an Charge Mode Input Select that allows you to choose the charge mode (Fast, Smart, Fixed, or Off), and set it to Smart after creation.

Create an Smart Charge Set Amps Input Number that you can use to manually set the charge amps in Fixed mode, and that the automations use to adjust charging amps.  Set the maximum for this to the maximum that your charger can supply.

Create a Turn Off Car Charger Timer with a timeout of 3:02 (Three Minutes Two Seconds).  This is used to turn off charging after allowing for the sun to come out again (if it was behind a cloud) after the solar output is insufficient to charge in the afternoon by setting charging amps to zero (rather than just a number below 6, which is typically a soft turn off).

Create the four automations.

If you need to alter any of the parameters of the Smart Charge Automation

``` yaml
  off_charging_amps: 0
  min_charging_amps: 4
  minimum_start_amps: 6
  never_exceed_amps: 32
```
then edit the automation in YAML mode, scroll to the bottom, and edit these variables nere the bottom.

If you have a period where you can charge for free (e.g. OVO EV plan, Globird Zero Hero between 11am and 2pm), you'll want to a) set up a charging schedule by editing the Smart Charge - Set Charge Amps Schedule to match your free period, and running it once - it sets a baseline schedule that will operate in the charger without any smart control - the default for when the car is plugged in.

In addition, you will probably want to automate setting the Charge Mode Input Select to Fast during your free periods - for example:

``` yaml
      - action: input_select.select_option
        metadata: {}
        data:
          option: Fast
        target:
          entity_id: input_select.charge_mode
        alias: Select Charge Mode Fast
```
