Set up some paths. This can be in your `~/.bash_profile`:
```bash
unset PYTHONPATH

export USERNAME="`whoami`"
export EDITOR="emacs -nw"

# SUMMIT
alias summit="ssh summit.ccs.ornl.gov"
module add cuda/9.2.148
export PROJECT="bip178"
alias interactive="bsub -W 2:00 -nnodes 1 -P $PROJECT -Is /bin/bash"
export USER_ARCHIVE="/home/$USER"
export PROJECT_HOME="/ccs/proj/$PROJECT"
export MEMBER_WORK="/gpfs/alpine/scratch/$USER/$PROJECT/"
export PROJECT_WORK="/gpfs/alpine/proj-shared/$PROJECT"
export WORLD_WORK="/gpfs/alpine/world-shared/$PROJECT"
export PROJECT_ARCHIVE="/proj/$PROJECT"

# miniconda
export PATH=$MEMBER_WORK/miniconda/bin:$PATH
```
Now install miniconda for PPC:
```bash
source ~/.bash_profile
cd $MEMBER_WORK
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-ppc64le.sh
bash Miniconda3-latest-Linux-ppc64le.sh -b -p miniconda
export PATH=$MEMBER_WORK/miniconda/bin:$PATH
```

# Remove mpi4py and install special version for titan

```bash
# Make sure to remove glib, since it breaks `aprun`
conda remove --yes --force glib mpi mpich mpi4py

# Build and install special mpi4py for titan
cd $SOFTWARE
wget https://bitbucket.org/mpi4py/mpi4py/downloads/mpi4py-3.0.0.tar.gz -O mpi4py-3.0.0.tar.gz
tar zxf mpi4py-3.0.0.tar.gz
cd mpi4py-3.0.0

cat >> mpi.cfg <<EOF
# Summit spectrum MPI for gcc 8.1.1
# ---------------------------------
[summit]
mpi_dir              = $OLCF_SPECTRUM_MPI_ROOT
mpicc                = $OLCF_SPECTRUM_MPI_ROOT/bin/mpicc
mpicxx               = $OLCF_SPECTRUM_MPI_ROOT/bin/mpiCC
EOF

python setup.py build --mpi=summit
python setup.py install
```
