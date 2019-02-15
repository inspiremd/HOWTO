** NOT FINISHED YET -- DO NOT USE! **

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
# If desired, create and activage a miniconda environment here
#conda create --name yank
#conda activate yank
# Add omnia and conda-forge channels
conda config --add channels omnia --add channels conda-forge
# Install OpenMM for CUDA 9.1
conda install -c omnia/label/cuda91 --yes openmm
# Test installation interactively
qsub -I -A $PROJECT -l nodes=1,walltime=00:30:00 -q debug
module load python_anaconda
module load cudatoolkit
#conda activate yank
#export PATH=/ccs/proj/$PROJECT/mskcc/miniconda/bin:$PATH
#export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/ccs/proj/<project_id>/mskcc/miniconda/lib
#export PYTHONPATH=$PYTHONPATH:/ccs/proj/<project_id>/mskcc/miniconda/lib/python2.7/site-packages/
#PYTHONPATH=$PYTHONPATH:/sw/xk6/python_anaconda/2.3.0/sles11.3_gnu4.8.2/lib/python2.7/site-packages/
#export OPENMM_CUDA_COMPILER=/opt/nvidia/cudatoolkit7.5/7.5.18-1.0502.10743.2.1/bin/nvcc
aprun -n1 python -m simtk.testInstallation
exit
```

# Install YANK

```bash
conda install --yes yank
```
