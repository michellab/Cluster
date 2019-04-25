# Resources

To request a user account, email Antonia: [antonia.mey@ed.ac.uk](mailto:antonia.mey@ed.ac.uk).

## Access

Upload files to your section9/plato home folder:

<sub>*(from e.g. your workstation)*</sub>

```
scp -r files username@section9.chem.ed.ac.uk:~
```

```
scp -r files username@plato.chem.ed.ac.uk:~
```

Alternatively, [you can mount a cluster's file system to your local workstation](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh).

Login:

```
ssh username@section9.chem.ed.ac.uk
```

```
ssh username@section9.chem.ed.ac.uk
```



## Hardware

| Section9 nodes     | Units                                                        |
| ------------------ | ------------------------------------------------------------ |
| Nodes001-004       | 16 x AMD Opteron(tm) Processor 6134                          |
| Node005-007        | 32 x Intel(R) Xeon(R) CPU E5-2650 0 @ 2.00GHz                |
| Nodes005-007       | 4 x Tesla M2090                                              |
| Nodes010-013       | 4 x NVIDIA GeForce GTX 980 Ti                                |
| Nodes010-013       | 32 x Intel(R) Xeon(R) CPU E5-2640 v3 @ 2.60GHz               |
| Storage            | RAIDED NFS file system /home with a default quota of 100GB exported to all nodes |
| Additional Storage | /scratch on compute nodes, if needed.                        |

| Plato nodes | Units                                           |
| ----------- | ----------------------------------------------- |
| Nodes01-05  | 46 x Intel(R) Xeon(R) Silver 4116 CPU @ 2.10GHz |
| Nodes01-05  | 4 x NVIDIA GeForce GTX 1080 Ti                  |



# Software

All globally available modules (i.e. software) are installed in an environment module system. This means that environment variables are automatically set when a certain module is requested.

To load a module:

```
module load slurm
```

To list all available modules:

```
module avail
```

To list all loaded modules:

```
module list
```

To unload a module:

```
module unload slurm
```

Custom software will need to installed in the users /home directory.

# Queuing system

The queuing system is called slurm. An introduction on how to use it on the cluster can be found [here](https://github.com/michellab/Cluster/blob/master/getting_started.md). Job prioritisation (i.e. deciding which queued jobs are handled first) is done internally by slurm and depends on a set of [rules](https://slurm.schedmd.com/priority_multifactor.html#configexample). 

# Data handling: good practices

1. Frequently monitor your personal data usage on the cluster by:

   ```
   /users/username/ -sh
   ```

2.  

TD









