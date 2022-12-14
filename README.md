# Instructions for performing Cholla cosmological simulations on the OLCF systems

This markdown site documents how to perform Cholla cosmological simulations on OLCF systems, with Summit as an example.

## Step 0)

* Log onto Summit

`ssh [username]@[system].olcf.ornl.gov`

followed by

`[pin][RSA key]`

Common user name and pins cannot start with 0.

## Step 1) Clone code

* Code fork 

`https://github.com/brantr/cholla:cosmology`

The fork from Bruno's repo was deleted and then added back to include cosmology.

* Clone onto Summit

`/ccs/home/brantr/code/cholla`

Is a git clone of Bruno's repo.

## Step 2) Configure code

`source builds/setup.summit.gcc.sh`

or

`source builds/setup.frontier.gcc.sh`

builds from `gcc 9.1.0` currently.

## Step 3) Compilation

`builds/make type cosmology`

calls

`make type particles`
and
`make type hydro`

includes

`-DANALYSIS -DPHASE_DIAGRAM -DLYA_STATISTICS -DOUTPUT_SKEWERS`

### CHECK `AVERAGE_SLOW_CELLS`

### `N_OUPTUT_COMPLETE=1` sets restarts vs. truncated snapshot with limited info

### `make TYPE=cosmology` can add `-j 20`, takes 1-2 min.

### In `bin/` `cholla.cosmology.summit`

### Example `make ics`

* `git clone cosmo_analysis`
* `cd ~/code/cosmo_analysis/initial_conditions/ics_music/Music`
* `cp Makefile_summit makefile`
* `module load gsl hdf5`
* note `-I /usr/include/tirpc` for `tipsy` rpc include
* `make`
* `ics_music/music/` parameter files
* `level 2^8=256`
* `zstart = 100`
* `50 Mpc/h`
* `YHe=0 -> 0.29` CHECK
* `seeds for each level`
* `format enzo`
* `filename change ics_hydro_256_50Mpc.conf`
* `/gpfs/alpine/ast175/proj-shared/using_cholla/ics/ics_music/ics_256_60Mpc.enzo?`

* `outputs file to gps`
* run from Andes
* `-> outputs random numbers to working direcotry`
* log into Andes
* `ssh brant@andes.olcf.ornl.gov`
* `module load gsl hdf5`
* `fftw/3.3.10`
* `cp music /gpfs/alpine/proj-shared/ast175/using_cholla/ics/ics_music`

## Get an interactive node and create enzo ics

* `salloc -A AST175 -p gpu -N 1 -t 1:00:00`
* `./music_ands? ~ /paramfile`

## Convert enzo to cholla
* `~/code/cosmo_analysis/initial_conditions/generate_ics_from_enzo_file.py`
* pass size of box
* N of cells per side
* n_boxes -> subvolumes + mpi processes
* decompostion is specified in `proc_grid`
* `load pythno/3.7-anaconda3`
* `python3 generate ics_from_enzo_fil.py hydro particles`
* wrote `/gpfs/alpine/ast175/proj-shared/using_cholla/ics/ics_music/ics_256_50Mpc/ics_z100`
* Run Dir `/gpfs/alpine/ast175/proj-shared/using_cholla/simulations/256_50Mpc/param.txt`
* Example parameter file
* `tout` not used
* `outstep` not used
* specify outputs in scale factof file
* indir -> input files
* outdir -> output dir
* scale_output_file -> list of output scale factors
* list of a's
* 100>0.0099 -- --
* a = 0.2 -> z=4, ends here
* exactly lost entry?

## `UVB_rates_file` is in `/gpfs/alpine/ast175/proj-shared/using_cholla/simulatins/256_60Mpc/uvb_rates_v22.txt`
* SEE HEADER FOR COLUMNS
* eV/s and 1/2
* `lya_skewers_stride=8`
* skip 8 scale with /res
* `lya_Pk_d_log_k=0.1`
* `analysis_scale_outputs_file`
* `scale_factor_analysis_56.txt` lists scale factors

## Step X): Summit Submit Job

* run sims
* `/gpfs/alpine/ast175/proj-shared/using_cholla/simulations/256_50Mpc/submit_job_summit.lsf`
* `bsub submit.job_summit.lsf`
* `bjobs`
* computing transmitted flux is the slowest
* less `[job]_output.txt`
* CHECK Statistics
* mean gas is 13.8
* temp is ~ 231
* HI fraction is hand coded
* enzo using PRIMORDIAL+IONIZED
* YHe fixed 
* Dual energy parameters
* Delta scale 0.003 for this resolution
* Beginning Delta a set by expansion, dont want a timestep >> than delta a expansion
* `run_timing.log`, units are ms
* To do compute P(k) on the fly

## Step Y): Analysis
* go back to andes
* `salloc -A AST175 -p gpu -N 1 -t 1:00:00`
* compute matter Pk
* `code/cosmo_analysi/matter_distribution/`
* `module load python/3.7-anaconda3`
* `plot_matter_power_specturm.py`
* Lbox = 50000
* ncells = 256
* n_bins = 15

## Step Z): Choose snapshot type
* line 42 (part of compute matter pk)
* `load_snapshot_data_distributed`
* `subgrid + range of axes` for loading subvolume
* `subgird = [[0,256],[0,256],[0,256]]`
* any hdf5 fields to P(k)

## Step AA): Compute Flux P(k)
* `cosmo_analysis/lya_statistics/`
* `compute_transmitted_flux.py`
* `/gpfs/alpine/ast175/proj-shared/using-cholla/`
* `transmitted_flux_files`
* `ipython`
* `run compute_transmitted_flux.py` 
* `data_flux-> flux_new`
* `skewers flux` `skewers total`?
* `data_ps` -> ps data -> mean ps, k val, skewer ps
* TO DO > save flux pk on the fly

## Step AB): We did
* `compile ics generator`
* `run ics`
* `compile cholla`
* `rum cosmo sim`
* `compute matter pk`
* `compute flux pk`

## Step AC): OUTSTANDING ISSUES
* `running large grids of sims`
* `cholla ics generator`
* `perform mcmc analysis`

## Step AD): Cholla generate ics
* `src/fft/` -- create objects and call with different filters
* `Filter_inv_k2` -- input, output, ifdef?
* `hanning? -> filter`
* `pragma` `[=] device` `i2 k2 j2` `d=-1/k^2 multiply by grid`
* `1/k^2 filter`
* `Filter_rescale_by_power_spectrum (in out)`
* `from i,j,k -> compute kx,ky,kz`
* linear inter P(k) <- 'other approximation'
* sqrt(Pk)  to rescale random numbers
* calls `linear_interpolation`