# read-CTS
### **_A virtual scoreboard by intercepting a [Colorado Time Systems](https://www.coloradotime.com/) scoreboard input_**
## [The CTS Scoreboard](https://www.google.com/patents/US4263736)
Colorado Time Systems developed the protocol to drive a scoreboard composed of multiple module displays having up to eight(8) seven-segment readouts for each.

The modules are configured to represent either Lane, Event/Heat, Lengths/Record, Team Scores, Place and Time information.

Lane modules are configured, by displaying left to right; Lane Number, Place, Minutes x 10, Minutes x 1, Seconds x 10, Seconds x 1, Seconds x .1 and Seconds x .01

The position of the readouts correlates similar to that of an eight character array[8].  So, data in array[0] - Lane Number; array[1] - Place; ... array[7] - Seconds x .01

Communications to the scoreboard is done serially at 9600 baud continuously having no delimiters, new line or carriage return; the scoreboard doesn't respond back at all to the timing console.  Refer to [ASCII Table](http://www.ascii-code.com/)

## The Protocol
The data for the readouts is preceded by a one byte of data representing the channel number of the module.

Each module will receive two sets of eight bytes of data with the first set representing the readout value to be displayed and the second set representing the format, i.e. decimal/colon activation.

**_For the purpose of this project the second set is not needed and is ignored._**

The channel number is determined from the byte of data having a value greater than 127 DEC which is immediately followed by up to eight bytes of data having a value less than 128 DEC for the readouts to display.
### Parsing Out the Channel Address
First, determine if the data represents readout values or format, i.e. decimal/colon activation.

The Least Significant Bit, LSB, from the channel number byte, determines this.  A FALSE or 0 LSB determines the following data set represents readout values.

Second, the channel number is determined by shifting right by one, masking the 5 LSB’s and then XOR it.
### Parsing Out the Value
Taking a closer look at the ASCII table, there are 8 sets of 16 values between 0 DEC to 127 DEC (00 HEX to 7F HEX).

Taking even a closer look, you can see how the HEX format comes into play.

The first readout in a Lane module, Lane Number, is assigned it’s value from the serial data containing 0F HEX to 06 HEX; Place – 1E HEX to 17 HEX; … Seconds x .01 – 7F HEX to 76 HEX.  Where the first four bits of the byte, 0-7, represents which readout and the second four bits of the byte, 0-F, represents the value the readout displays.

The second four bits are inverse so we have to XOR to display the correct value.
### Address Zero
This has been referred to a Channel 0 from others but it is the clock that runs consistently once the timing console is powered and initialized.

Each time there’s a start received at the timing console, it resets the data to blank including the lane data.

At the scoreboard this Channel displays the running time since the start for each lane module until a split/finish input is detected.
### Lane Data
The lane modules utilizes two different decimal values, greater than 127 DEC, to control what to display and when.

When the channel number address byte decimal is greater than 169 DEC and less than 190 DEC; Bit 7, having the Most Significant Bit (MSB) being 8, is FALSE or 0, the module will be:
- (Off) Blank
- (On) showing Lane Number and Split or
- (On) showing Lane Number, Place and Finish Time

When the channel number address byte decimal is greater than 191 DEC; Bit 7, is TRUE or 1, the module will be:
- (On) showing Lane Number and Running Time
