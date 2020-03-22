
## I/O Traces & Reports


### FLASH 4.4

#### 1. [Sod 2D without collective I/O](./reports/sod_2d_nofbs.html)

* System: Stampede2 at TACC
* Setup command: `Sod -auto -2d +ug +nofbs +parallelio`
* Problem size: 512 x 512
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Remark: With +nofbs, the problem size is spcificied in flash.par and FLASH can not use collective I/O.

#### 2. [Sod 2D with collective I/O](./reports/sod_2d_ug.html)

* Setup command: `Sod -auto -2d +ug +parallelio -nxb=64 -nyb=64`
* Problem size: 512 x 512
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Remark: This setup makes the problem scales with the number of processes used.
        Problem size: nxb * iProcs, nyb * jProcs.
        This configuration enables collective I/O.

#### 2.1 [Stripe count set to 4](./reports/sod_2d_ug_stripe_count4.html)

Same configuration as above but with Lustre stripe count set to 4.

Example of one of produced checkpoint file.

```bash
./sod_hdf5_chk_0000
lmm_stripe_count:   4
lmm_stripe_size:    1048576
lmm_pattern:        1
lmm_layout_gen:     0
lmm_stripe_offset:  5
	obdidx		 objid		 objid		 group
	     5	     157834736	    0x9685df0	   0xac0000400
	     3	     158138579	    0x96d00d3	   0xb40000400
	    43	     158987015	    0x979f307	  0x1040000400
	    57	     161523633	    0x9a0a7b1	  0x1180000400
```

ROMIO Hints:

```bash
key = direct_read               value = false
key = direct_write              value = false
key = romio_lustre_co_ratio     value = 1
key = romio_lustre_coll_threshold value = 0
key = romio_lustre_ds_in_coll   value = enable
key = cb_buffer_size            value = 16777216
key = romio_cb_read             value = automatic
key = romio_cb_write            value = automatic
key = cb_nodes                  value = 8
key = romio_no_indep_rw         value = false
key = romio_cb_pfr              value = disable
key = romio_cb_fr_types         value = aar
key = romio_cb_fr_alignment     value = 1
key = romio_cb_ds_threshold     value = 0
key = romio_cb_alltoall         value = automatic
key = ind_rd_buffer_size        value = 4194304
key = ind_wr_buffer_size        value = 524288
key = romio_ds_read             value = automatic
key = romio_ds_write            value = automatic
key = cb_config_list            value = *:1
key = romio_filesystem_type     value = LUSTRE:
key = romio_aggregator_list     value = 0 8 16 24 32 40 48 56
key = striping_unit             value = 1048576
key = striping_factor           value = 4
key = romio_lustre_start_iodevice value = 0
```

#### 2.2. [Remove H5Fflush from the checkpointing routine](./reports/sod_2d_ug_stripe_count4_noh5flush.html)

Same configuration as 2.1 except we deleted the `H5Fflush` call in `io_h5write_unknowns.c`.

This file is used to write double precision multi-demensional variables (e.g., density, pressure) to HDF5 files.

So it is used mostly for writing checkpoint files. In contrast, plot files by default keep variables in single precision, which uses the routine from `io_h5write_unknowns_sp.c`.

The major difference between `io_h5write_unknowns` and `io_h5write_unknowns_sp` is that `io_h5write_unknowns` calls `H5Fflush` before closing the file. 
**FLASH calls `io_h5write_unknows` (which calls `H5Fflush`) n times to write n variables.**

`H5Fflush` enforces HDF5 library to flush the dirty metadata. So every time it is called, it writes to the same part of the file.
This explains why I/O on plot files have no conflicting behaviours but I/O on checkpoint files have WRITE-AFTER-WRITE patterns.


So without the `H5Fflush` call, there will be no conflicting I/O operations. This one line modification makes FLASH possible to run on UnifyFS which does not support conflicting writes.



#### 3. [Sod 2D (AMR) with collective I/O](./reports/sod_2d_amr.html)
* Setup command: `Sod -auto -2d +parallelio`
* Problem size: adaptive mesh refinement (nxb=8, nxy=8)
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Remark: By default (without -ug), FLASH uses adaptive mesh refinement.

```bash
key = direct_read               value = false     
key = direct_write              value = false     
key = romio_lustre_co_ratio     value = 1         
key = romio_lustre_coll_threshold value = 0         
key = romio_lustre_ds_in_coll   value = enable    
key = cb_buffer_size            value = 16777216  
key = romio_cb_read             value = automatic 
key = romio_cb_write            value = automatic 
key = cb_nodes                  value = 8         
key = romio_no_indep_rw         value = false     
key = romio_cb_pfr              value = disable   
key = romio_cb_fr_types         value = aar       
key = romio_cb_fr_alignment     value = 1         
key = romio_cb_ds_threshold     value = 0         
key = romio_cb_alltoall         value = automatic 
key = ind_rd_buffer_size        value = 4194304   
key = ind_wr_buffer_size        value = 524288    
key = romio_ds_read             value = automatic 
key = romio_ds_write            value = automatic 
key = cb_config_list            value = *:1       
key = romio_filesystem_type     value = LUSTRE:   
key = romio_aggregator_list     value = 0 8 16 24 32 40 48 56 
key = striping_unit             value = 1048576   
key = striping_factor           value = 1         
key = romio_lustre_start_iodevice value = 0         
```

### ENZO 

#### 1. [CollapseTestNonCosmological](./reports/CollapseTestNonCosmological.html)

Problem Size: 64 x 64 x 64

* System: Stampede2 at TACC
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Configuration file:
* Remark: Each processor writes to its own checkpointing file (N-to-N pattern) using **serial HDF5**.


### LAMMPS (7 Aug 2019)

#### 1. [benchmark/lj](./reports/lammps_lj_stripe_count4_ibrun.html)

Problem Size: 320 x 320 with 8,192,000 atoms

* System: Stampede2 at TACC
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 4
* Configuration file:
* Remark: LAMMPS is configured to use MPI-IO to write out checkpoints. ROMIO hints are the same as others.
          One interesting thing is that if running with **mpirun** (instead of ibrun), ROMIO can not detect the underlying
          filesystem as LUSTRE (still NFS). And other hints such as `cb_nodes` and `striping factor` seem to have no effect.
          No collective I/O is used. Check the [report](./reports/lammps_lj_stripe_count4_mpirun.html) using mpirun here.
	  
  * Update on this: The Intel MPI library uses ROMIO, but configures the file-system specific drivers a bit differently.   MPICH selects which file system drivers to support at **compile-time** with the –with-file-system configure flag.  These selected drivers are compiled directly into the MPICH library.  Intel-MPI builds its file-system drivers as loadable modules, and relies on two environment variables to enable and select the drivers.
  
    Example: mpiexec -env I_MPI_EXTRA_FILESYSTEM on -env I_MPI_EXTRA_FILESYSTEM_LIST lustre -n 2 ./test

### LAMMPS (3 Mar 2020)

#### 1.

* System: Quartz at LLNL
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Compiler & Libraries: intel/19.1.0, impi/2018.0, HDF5-1.12.20, ADIOS2
* Note: compiled with mpiio, hdf5, adios, h5md




### QMCPACK 3.8.0

#### 1. [example/H2O](./reports/qmcpack_h2o_stripe_count1.html)

* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Configuration file:
* Output files:
  Each section (computation, e.g. VMC, DMC) in input xml from top to bottom will be given a number,
  so the output files for this section will have filenames in the format of `TITLE.s00%d.xxx`, e.g., `H2O.s001.scalar.dat`
  * `TITLE.s###.scalar.dat`: block averages, e.g., LocalEnergy, LocalPotential, Kinetic, ElecElec
  * `TITLE.s###.dmc.dat`: averages per DMC step
  * `TITLE.s###.stat.h5`: block avegrages in HDF5
  * `TITLE.s###.random.h5`: array values for restart
  * `TITLE.s###.config.h5`: configuration for restart
* Remark: Parallel HDF5 is used. Each section has a fixed filename prefix, thus it overwrites the checkpoint files during one section's computation.
          Since we set stripe count to 1 and the ROMIO hints `romio_lustre_co_ratio` by default is also 1, we have only one aggregator doing all the I/O - even though we have 8 aggregators (one aggregator per node).
* I/O patterns explanation:
  * **RAR** on `H2O.HF.wfs.xml` and `simple-H2O.xml`: both are input files. All ranks simply read all bytes of them using POSIX calls independently at beginning of the execution.
  * **WAW** on `.qmc.xml`: this file written only by rank 0 using POSIX calls. They kept some intermediate information like energy value and other parameters. Each I/O on them simply overwrites the entire file.
  * **WAW** on `.random.h5` and `.config.h5`: By default, without HDF5 collective metadata flush, multiple processes may each write a portion of the dirty metadata to the file.
                                              So at each checkpoint step, the same rank may overwrite previous metadata and raw data (because it reuses the same checkpoint file during one section).
* Here's the [report](./reports/qmcpack_h2o_stripe_count4.html) for stripe count = 4, however, it seems that still rank 0 performs most of the I/O operations, no idea why.
  
  
### MACSio 

#### 1. [Ale3D](./reports/Ale3d/ale3d_8x8ranks.html)

* System: Quartz at LLNL
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Compiler & Libraries: intel/19.1.0, impi/2018.0, Silo-4.10.2-bsd, HDF5-1.8.20
* Command: `mpirun -np 64 ../macsio --interface silo --avg_num_parts 1 --part_size 100K --part_type unstructured --part_dim 3 --vars_per_part 50 --parallel_file_mode MIF 8`
* Output logs: [macsio-log.log](./reports/Ale3d/macsio-log.txt) [macsio-timings.log](./reports/Ale3d/macsio-timings.txt)
* I/O Patterns:
  * same rank: READ-AFTER-READ, WRITE-AFTER-READ
  * different rank: READ-AFTER-READ, READ-AFTER-WRITE, WRITE-AFTER-WRITE
* Explaination:
    This emulation(due to silo) produces 8 files per dump step (total of 10 dump steps).
    64 MPI ranks are divided into 8 sub-groups, where each group writes to one file.
    For example, rank 0~7 writes to file macsio\_silo\_00000\_000,silo, ..., macsio\_silo\_00000\_009.silo

    Each process in a sub-group is responsible of writting a portion of the file.
    The attribute **nlinks** is used to construct an unique name for the dataset for writing.
    Before writing to the next dataset, the process will read back **nlink**, increase it by one and then use it to name the dataset.

    The file is written by one process at a time (See offset-vs-rank and offset-vs-time plots). Open->write->close.
    The **nlink** attribute shows RAR, RAW and WAR on different ranks as the next process has to reopen the file.
    Note that there is no WAW because the attribute is only flushed once the file is closed.

    WAR on same rank is caused by `H5Gget_objinfo`. Rank 0 writes group information then later Rank 1 reads it.
    On close, Rank 1 writes this information again.



### VASP

#### [report](./reports/vasp_4x1ranks.html)

* System: Quartz at LLNL
* MPI: 4 MPI Processes - 4 nodes and 1 MPI rank per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Compiler & Libraries: intel/18.0.1, mvapich2-2.2
* Note: No MPI-I/O, No I/O libraries.


### ParaDis 2.5.1.1

* System: Quartz at LLNL
* MPI: 4 MPI Processes - 4 nodes and 1 MPI rank per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Compiler & Libraries: intel/19.1.0 impi/2018.0, hdf5 1.8.20
* Configuration: run 1000 steps, dump plot files, restart files, velocity files every 500 steps. [input file](./reports/ParaDis/Copper.ctrl)
* Note: This code is very old, it uses HDF5 1.6 API. Have to make some changes to compile it with HDF5 1.8.
       HDF5 is only used to write/read restart files. All other files are in ASCII format.

#### 1. [Without HDF5](./reports/ParaDis/paradis_copper_ascii-restart.html)
#### 1. [With HDF5](./reports/ParaDis/paradis_copper_hdf5-restart.html)



### NWchem 6.8.1

#### [report]()

* System: Quartz at LLNL
* MPI: 4 MPI Processes - 4 nodes and 1 MPI rank per node
* Filesystem: Lustre, stripe size: 1MB, stripe count: 1
* Compiler & Libraries: intel/19.1.0 impi/2018.0
* Note: NWchem does not use any I/O library.
