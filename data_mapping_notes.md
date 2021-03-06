# Notes on tank-lab-to-nwb data mapping
## Open questions for lab meeting are in braces []

All positional information is contained in .mat files similar to the form: PoissonBlocksReboot4_cohort4_Bezos3_E65_T_20180202.mat

This matlab file contains a struct named "log" which contains several fields relevant for conversion...

### NWBFile
* log.session.start is an array of datetime information defining the start of the experiment
* log.animal.protocol may be worth including in session description for the NWBFile
* [do we have information on the experimenter name?]

### Subject
*Subject information is under log.animal
  * log.animal.name -> Subject name
  * log.animal.normWeight -> Subject weight [but was is the 'norm'?]
  * log.animal.importAge -> Subject age [what units?]

[Unable to find species information here. Assuming Mus musculus.]


### Position (processed, not acquisition)
* The field log.block is a struct of length n_epochs
* The sub-field log.block(j).trial is a struct of length n_trials for epoch j

* log.block(j).trial(k).time is a vector of timestamps for the positional recording; unfortunately, these do appear to be irregularly sampled and so must be specified in the NWBFile. Also, they always begin indexed at zero with respect to the *trial* start time (see "Intervals" below to learn how to access this)
* log.block(j).trial(k).position is a 3-d vector of length *less* than log.block(j).trial(k).time; the positional recording stops when the subject completes the maze. We will have to decide if we want to pad the position series with None or Nan to match the dimension of the timestamps
  
  
### Intervals
###### See Position for introduction to block and trial structure within the struct
* log.block(j).start contains an array of datetime information for the start of epoch j
* log.block(j).duration contains the number of seconds epoch j lasted for
* log.block(j).trial(k).start contains a float of the time difference (in seconds) between the start of the first epoch (not the session start time!) and the start of trial k [can they triple check this?]
* log.block(j).trial(k).duration contains the duration (in seconds) of trial k of epoch j

[I noticed that there is overlap between end and start times of consecutive epochs on the order of 0.1 seconds or so. Similarly, some but not all of the trials partially overlap on the order of a few microseconds or so. Is this a feature of their system?]

Since trials are concatenated in NWBFiles distinct from but in line with the actual epochs, the trial intervals will have to be pulled and assembled in order.
