
### FLASH 4.4

#### 1. [Sod 2D without collective I/O](./reports/sod_2d_nofbs.html)

* Setup command: `Sod -auto -2d +ug +parallelio -nxb=64 -nyb=64`
* Problem size: 512 x 512
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Remark: This setup makes the problem scales with the number of processes used.
        Problem size: nxb * iProcs, nyb * jProcs.
        This configuration uses collective I/O.

#### 2. [Sod 2D with collective I/O](./reports/sod_2d_ug.html)

* Setup command: `Sod -auto -2d +ug +nofbs +parallelio`
* Problem size: 512 x 512
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Remark: With +nofbs, the problem size is spcificied in flash.par and FLASH can not use collective I/O.

#### 3. [Sod 2D (AMR) with collective I/O](./reports/sod_2d_amr.html)
* Setup command: `Sod -auto -2d +parallelio`
* Problem size: adaptive mesh refinement (nxb=8, nxy=8)
* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
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
key = romio_aggregator_list     value = 0 1 2 3 4 5 6 7 
key = striping_unit             value = 1048576   
key = striping_factor           value = 1         
key = romio_lustre_start_iodevice value = 0
```

### ENZO 

#### 1. [CollapseTestNonCosmological](./reports/CollapseTestNonCosmological.html)

Problem Size: 64 x 64 x 64

* MPI: 64 MPI Processes - 8 nodes and 8 MPI ranks per node
* Configuration file:
* Remark: Each processor writes to its own checkpointing file (N-to-N pattern) using **serial HDF5**.
