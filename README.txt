
https://github.com/duncanjohnhowe/slice-select
Should have contain the latest au and pulse programs.

These files should be placed in the appropriate location in topspin.
********************
WARRANTY/disclaimer
********************
THIS PROGRAM IS DISTRIBUTED IN THE HOPE THAT IT WILL BE USEFUL, WITHOUT ANY WARRANTY.
IT IS PROVIDED 'AS IS' WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED OR IMPLIED,
INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTY OF FITNESS FOR A PARTICULAR PURPOSE.
THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU.
SHOULD THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING,
REPAIR OR CORRECTION.
IN NO EVENT WILL THE AUTHOR BE LIABLE TO YOU FOR DAMAGES, INCLUDING ANY GENERAL,
SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES ARISING OUT OF THE USE OR INABILITY
TO USE THE PROGRAM (INCLUDING BUT NOT LIMITED TO LOSS OF DATA OR DATA BEING RENDERED
INACCURATE OR LOSSES SUSTAINED BY YOU OR THIRD PARTIES OR A FAILURE OF THE PROGRAM
TO OPERATE WITH ANY OTHER PROGRAMS), EVEN IF THE AUTHOR HAS BEEN ADVISED OF THE
POSSIBILITY OF SUCH DAMAGES.

********************
Introduction
********************
This program will run a loop of 1D experiments; adding each one into a pseudo 2D dataset.
It was designed to build a quick 'map' of a tube, based on either 1H or 19F observe, to get the parameters for
the 'best' layer to perform a longer 1D acquisition.
It will examine the dataset it is run from to determine the nuclei. It uses the wavemaker software to calculate
the power level of the shaped pulse involved. Some default values and logic are included to try and ensure the pulse length
and power, as well as the gradient duty cycle are within limits. However I can't stress strongly enough; check before use!!!
The individual 1D experiments will have varying phase as you move up the tube; the 2D data can be processed in magnitude mode
to obscure this.
There is also another au program 'slice-proc-au' which will take apart the 2D into individual slices; phase correct them and put them
back into a processed directory; this gives sharper linewidths.

********************
S8.2 Setup (From paper published)
********************
S8.2.1 Initial setup
Install Bruker’s “Wavemaker” software if necessary. It can be downloaded from https://www.bruker.com/service/information-communication/nmr-pulse-program-lib/bruker-user-library/liquids/avance-3.html, (registration required).

Put the “slice-select” pulse program into the user pulse program directory :-(TOPSPIN_HOME_DIR/exp/stan/lists/pp/user/) or type “edpul slice-select” and paste the text in.
Put the au program “slice-set-au” into the TOPSPIN_HOME_DIR/exp/stan/prog/au/src/user directory, or use “edau” to create the program explicitly.

Prepare a test sample. A two-layer CDCl3/D2O sample was used; taking care to set the layer interface as near as possible to the mid-point of the probe’s transmit/receive coils, using the instrument’s depth gauge.

S8.2.2 Preparation of parameter sets
This protocol must be established only once on each instrument to render it capable of running the slice-selective experiment described here.

The basis of the setup is to prepare parameter sets for :-
-A 1D slice-selective experiment for the desired nuclei. (e.g. 1H)
-A pseudo 2D experiment into which to deposit the 1D results.

S8.2.3 Preparation of 1D slice-selective parameter set
Begin by running a 1D acquisition of the desired nuclei. Ensure the 90 degree pulse is properly calibrated and set in the experiment. Then type :-

iexpno
pulprog slice-select
ds 0

Then choose an appropriate irradiation bandwidth to excite a given region in your spectrum. There will be a limit as to what size region you can excite, due to the power levels necessary. (Consult the manufacturers documentation for appropriate radio-frequency and gradient power limits.)  On the instruments at the University of Cambridge; 5000Hz seems to work quite well for proton experiments.

Set cnst19 in the parameters to be this irradiation bandwidth e.g. :-
cnst19 5000

Set the GPZ1 value to an appropriate level, in our experiments we used 50%.
gpz1 50

S8.2.4 Calculation of the shaped pulse
Type :-

wvm –a

This command instructs the “Wavemaker” software to calculate the power of the shaped pulse and automatically fill in the parameters for the experiment.

Check that the pulse length calibrated is appropriate for the instrument. The ‘au’ program has hard-coded limits on the slice width in Hz; which should mitigate unusually powerful pulses from being generated. Nevertheless, it is advisable to check what radio power and gradient length a given bandwidth will require to avoid damaging the instrument.

Set the number of scans to one. Choose an offset for the slice, negative values show the bottom layer on our instrument. The Nyquist theory limits the value on our 500Mhz instrument, for a 5000Hz width pulse to be +/- 166666. We thus generally choose -50000 as our initial offset:

spoffs1 -50000

Run the experiment. You should see a spectrum from one layer of your sample; on our instruments, this layer will be from the bottom half of the sample if the spoffs1 value is negative.

Check the result, then run again with spoffs1 = +50000 to see a spectrum from the other layer.

S8.2.5 Writing the 1D parameter set
Within the experiment just run type :-

wpar slice-select-NUCLEI

Where NUCLEI is what is stored in the parameters as “NUC1” (e.g. 1H, 19F, etc…). The au program relies on this syntax being correct.

Next create and write the 2D parameter set into which the acquired 1D spectra will be deposited. Using the same experiment from which the slice-select-1H parameter set was created above, change the acquisition dimension to 2. Set TD for both acquisition AND processing in F1 to 64. Write the parameter set:
wpar slice-select-2d

S8.2.6 Running the ‘au’ program to create a z-map of the sample:
The au program will overwrite the current dataset. The recommendation is to run a 1D experiment of the nuclei of interest; setting the sweep width to be around the peaks of interest and setting the 90 pulse angle. Then :-
iexpno
slice-set-au
The ‘au’ program will use the sweep width and other parameters from the experiment you have just run as the basis for each slice experiment. Follow the prompts in setting up the experiment.
Avoid changing between datasets/experiments whilst the setup and acquisition is running; because parameters are set according to what is in the current dataset, errors may occur if the open dataset is changed.
As the program runs, the 1D experiments are accumulated into a 2D dataset. A logfile will be written into the dataset and the title file updated to reflect the actions performed during the individual acquisitions.
To abort the acquisition, type “kill” to stop the au program. Alternatively, exit Topspin entirely.

S8.2.7 Running a 1D experiment
The 2D result can be processed with the commands ‘xf2;xf2m’ This can then be used to choose a 1D slice to run. In Topspin, go into the multiple display mode and scroll through the increments. Choose the increment with the sharpest, most intense peaks; the ‘spoffs’ value for this increment can be found in the logfile/title.

Go to experiment 999, then use the command edc (or wra n) to copy to a new experiment. Set the spoffs, and change any other parameters such as NS and D1.

S8 Files

The following Bruker Topspin v3.2+ programs (as additional SI attachments)  are included to implement the experiments described in section S8:-
•	Au-programs/slice-set-au – Implements z-mapping of an NMR sample.
•	Au-programs/slice-proc-au – Processes a z-map to attempt automatical removal of phase distortions.
•	Pulse-program/slice-select – Spectrometer pulse program to acquire a ‘slice’ from an NMR tube; used to create a z-map.
