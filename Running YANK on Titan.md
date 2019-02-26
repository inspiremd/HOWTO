# Running YANK on Titan

This HOWTO describes how to get [YANK](http://getyank.org) up and running on [ORNL Titan](https://www.olcf.ornl.gov/olcf-resources/compute-systems/titan/).

## Set up your ~/.bash_profile to contain the following:
```bash
unset PYTHONPATH
export EDITOR="emacs -nw"

if [[ `hostname -f` == *"titan"* ]]; then
  echo "Configuring environment for TITAN"
  export PROJECT="chm126"

  alias titan="ssh titan.ccs.ornl.gov"

  module add cudatoolkit/9.1.85_3.10-1.0502.df1cc54.3.1
  
  # We need the gcc programming environment for building mpi4py
  module unload PrgEnv-pgi 
  module load PrgEnv-gnu
  module load cray-mpich

  # user-centric storage
  # https://www.olcf.ornl.gov/for-users/system-user-guides/titan/titan-user-guide/#user-centric-data-storage
  export USER_HOME=$HOME
  export USER_ARCHIVE=/home/$USER
  # project-centric storage
  # https://www.olcf.ornl.gov/for-users/system-user-guides/titan/titan-user-guide/#project-centric-data-storage
  export PROJECT_HOME=/ccs/proj/$PROJECT # not purged, backed up, not purged, 50G limit
  export MEMBER_WORK=$MEMBERWORK/$PROJECT # scratch space, purged
  export PROJECT_WORK=$PROJWORK/$PROJECT # scratch space, purged
  export WORLD_WORK=$WORLDWORK/$PROJECT # not writeable from nodes
  export PROJECT_ARCHIVE=/proj/$PROJECT # not writeable from nodes

  #export SOFTWARE="$PROJWORK/$PROJECT/yank/$USER"
  #export LD_LIBRARY_PATH="$MINICONDA3/lib:$LD_LIBRARY_PATH"
  #export SCRATCH="/lustre/atlas/scratch/$USERNAME/$PROJECT/"

  # OpenEye license
  export OE_LICENSE="$SOFTWARE/openeye/oe_license.txt"
elif [[ `hostname -f` == *"summit"* ]]; then
  echo "Configuring environment for SUMMIT"
  export PROJECT="bip178"

  alias summit="ssh summit.ccs.ornl.gov"

  module add cuda/9.2.148
  module add gcc/8.1.1 # forces mpicc/mpiCC to be reloaded

  alias interactive="bsub -W 2:00 -nnodes 1 -P $PROJECT -alloc_flags gpudefault -Is /bin/bash"

  export USER_ARCHIVE="/home/$USER" 
  export PROJECT_HOME="/ccs/proj/$PROJECT" 
  export MEMBER_WORK="/gpfs/alpine/scratch/$USER/$PROJECT/"
  export PROJECT_WORK="/gpfs/alpine/proj-shared/$PROJECT"
  export WORLD_WORK="/gpfs/alpine/world-shared/$PROJECT"
  export PROJECT_ARCHIVE="/proj/$PROJECT"
fi

# miniconda
export MINICONDA=$MEMBER_WORK/miniconda
export PATH=$MINICONDA/bin:$PATH
```

## Install Miniconda
```bash
source ~/.bash_profile
cd $MEMBER_WORK
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p miniconda
```

## Install OpenMM and omnia

```bash
# Add omnia and conda-forge channels
conda config --add channels omnia --add channels conda-forge
# Install OpenMM for CUDA 9.1
conda install -c omnia/label/cuda91 --yes openmm
# Test installation interactively
qsub -I -A $PROJECT -l nodes=1,walltime=00:30:00 -q debug
# Change to a valid directory for aprun
cd $MEMBER_WORK
# Test the installation on a compute node
aprun -n1 python -m simtk.testInstallation
```
This should produce output like
```
There are 4 Platforms available:

1 Reference - Successfully computed forces
2 CPU - Successfully computed forces
3 CUDA - Successfully computed forces
4 OpenCL - Successfully computed forces

Median difference in forces between platforms:

Reference vs. CPU: 1.98163e-05
Reference vs. CUDA: 2.15612e-05
CPU vs. CUDA: 1.55631e-05
Reference vs. OpenCL: 2.15471e-05
CPU vs. OpenCL: 1.55066e-05
CUDA vs. OpenCL: 1.29508e-07
Application 19786782 resources: utime ~45s, stime ~2s, Rss ~336972, inblocks ~25570, outblocks ~25843
```
Run the benchmark
```bash
# Run the benchmark on a compute node
cd $MINICONDA/share/openmm/examples
aprun -n1 python benchmark.py --test=pme --platform=CUDA --seconds=30 --heavy-hydrogens --precision=mixed
```
which should produce something like
```
Platform: CUDA
Precision: mixed

Test: pme (cutoff=0.9)
Step Size: 5 fs
Integrated 13002 steps in 30.0585 seconds
186.864 ns/day
Application 19786796 resources: utime ~51s, stime ~6s, Rss ~359792, inblocks ~28001, outblocks ~25968
```

# Install YANK

```bash
# Install yank and dependencies
conda install --yes yank
```

# TODO: Rebuild and reinstall mpi4py for cray
```bash
cd $MEMBER_WORK

# Remove mpi4py and install special version for titan
# Make sure to remove glib, since it breaks `aprun`
conda remove --yes --force glib mpi mpich mpi4py

# Build and install special mpi4py for titan
cd $SOFTWARE
wget https://bitbucket.org/mpi4py/mpi4py/downloads/mpi4py-3.0.0.tar.gz -O mpi4py-3.0.0.tar.gz
tar zxf mpi4py-3.0.0.tar.gz
cd mpi4py-3.0.0

cat >> mpi.cfg <<EOF
[cray]
mpi_dir              = /opt/cray/mpt/7.6.3/gni/mpich-gnu/4.9/
mpicc                = cc
mpicxx               = CC
extra_link_args      = -shared
include_dirs         = %(mpi_dir)s/include
libraries            = mpich
library_dirs         = %(mpi_dir)s/lib/shared:%(mpi_dir)s/lib
runtime_library_dirs = %(mpi_dir)s/lib/shared
EOF

python setup.py build --mpi=cray
python setup.py install
```
If the compilation step fails, make sure your `.bash_profile` correctly configured the `gcc` programming environment with:
```bash
module unload PrgEnv-pgi 
module load PrgEnv-gnu
module load cray-mpich
```
