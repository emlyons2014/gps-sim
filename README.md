# GPS-SIM

GPS-SIM is designed to use the HackRF to simulate GPS. This is a specific application the the GPS-SDR-SIM found at [osqzss/gps-sdr-sim](https://github.com/osqzss/gps-sdr-sim)


### Building with GCC

```
$ gcc gpssim.c -lm -O3 -o gps-sdr-sim
```

### Generating the GPS signal file
##### See satgen folder for creating csv from NMEA txt file

A user-defined trajectory can be specified in either a CSV file, which contains 
the Earth-centered Earth-fixed (ECEF) user positions, or an NMEA GGA stream.
The sampling rate of the user motion has to be 10Hz.
The user is also able to assign a static location directly through the command line.

The user specifies the GPS satellite constellation through a GPS broadcast 
ephemeris file. The daily GPS broadcast ephemeris file (brdc) is a merge of the
individual site navigation files into one. The archive for the daily file is:

[ftp://cddis.gsfc.nasa.gov/gnss/data/daily/](ftp://cddis.gsfc.nasa.gov/gnss/data/daily/)

Example file path for most recent data available on May 7, 2018 @ 16:43 EDT 
<ftp://cddis.gsfc.nasa.gov/gnss/data/daily/2018/brdc/brdc1270.18n.Z>

These files are then used to generate the simulated pseudorange and
Doppler for the GPS satellites in view. This simulated range data is 
then used to generate the digitized I/Q samples for the GPS signal.

hackrf_transfer supports I/Q pairs stored as signed bytes.
HackRF requires 2.6 MHz sample rate.

The simulation start time can be specified if the corresponding set of ephemerides
is available. Otherwise the first time of ephemeris in the RINEX navigation file
is selected.

The maximum simulation duration time is defined by USER_MOTION_SIZE to 
prevent the output file from getting too large.

The output file size can be reduced by using "-b 1" option to store 
four 1-bit I/Q samples into a single byte. 

```
Usage: gps-sdr-sim [options]
Options:
  -e <gps_nav>     RINEX navigation file for GPS ephemerides (required)
  -u <user_motion> User motion file (dynamic mode)
  -g <nmea_gga>    NMEA GGA stream (dynamic mode)
  -c <location>    ECEF X,Y,Z in meters (static mode) e.g. 3967283.15,1022538.18,4872414.48
  -l <location>    Lat,Lon,Hgt (static mode) e.g. 30.286502,120.032669,100
  -t <date,time>   Scenario start time YYYY/MM/DD,hh:mm:ss
  -T <date,time>   Overwrite TOC and TOE to scenario start time
  -d <duration>    Duration [sec] (dynamic mode max: 300 static mode max: 86400)
  -o <output>      I/Q sampling data file (default: gpssim.bin ; use - for stdout)
  -s <frequency>   Sampling frequency [Hz] (default: 2600000)
  -b <iq_bits>     I/Q data format [1/8/16] (default: 16)
  -i               Disable ionospheric delay for spacecraft scenario
  -v               Show details about simulated channels
```

The user motion can be specified in either dynamic or static mode, examples:

```
> gps-sdr-sim -e brdc3540.14n -u circle.csv
```

```
> gps-sdr-sim -e brdc3540.14n -g triumphv3.txt
```

```
> gps-sdr-sim -e brdc3540.14n -l 30.286502,120.032669,100
```

### Transmitting the samples

The TX port of a particular SDR platform is connected to the GPS receiver 
under test through a DC block and a fixed 50-60dB attenuator.

#### HackRF:

```
> hackrf_transfer -t gpssim.bin -f 1575420000 -s 2600000 -a 1 -x 0
```
```
Usage: hackrf_transfer [options]
Options:
  -t <filename>         # Generated binary file with GPS data (required)
  -f <frequency>        # GPS band center frequency (required)
  -s <sample_rate_hz>   # Sampling frequency [Hz]
  -a <set_amp>          # Set Amp 1 = Enable, 0 = Disable
  -x <gain_db>          # Gain start at 0 dB with increments of 1 dB up to 47 dB
Additional unused hackrf_transfer options:
  -r <filename>         # Receive data into file.
  -l <gain_db>          # Set lna gain, 0-40dB, 8dB steps
  -i <gain_db>          # Set vga(if) gain, 0-62dB, 2dB steps
  -n <num_sample>       # Number  of  samples  to  transfer (default  is unlimited).
  -b <baseband_filter_bw_hz> # Set baseband filter bandwidth in MHz.
        Possible values:
        1.75/2.5/3.5/5/5.5/6/7/8/9/10/12/14/15/20/24/28MHz
        default <sample_rate_hz>
```

Check for external clock (TCXO):
```
> hackrf_si5351c -n -0 -r 
	Output:
    [  0] -> 0x01	# detected 
    [  0] -> 0x51	# not detected
```


### License

Copyright &copy; 2015-2018 Takuji Ebinuma  
Distributed under the [MIT License](http://www.opensource.org/licenses/mit-license.php).
