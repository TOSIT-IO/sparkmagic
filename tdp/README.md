# Kernels Spark Build

For building Kernels Spark, you need to recover the build container here :
https://github.com/TOSIT-IO/TDP/tree/main/build-env-python

## Start the container

Next, start the container with specific values regarding your environment but with the official build image above.

For example :

```sh
DCT_GIT="$HOME/git"
DCT_PIP="$HOME/.cache/pip"
DCT_BUILD="$HOME/build"
```

The git directory matchs to the local directory that contains the git repositories.
The build directory matchs to the local directory where the archive is created.

And the command :

```sh
docker run --rm=true -it --env "USER_UID=$(id -u)" --env "USER_GID=$(id -g)" --volume "$DCT_PIP:/home/builder/.cache/pip/" --volume "$DCT_GIT:/home/builder/git/" --volume "$DCT_BUILD:/home/builder/build" --workdir /home/builder/ tdp_builder_python
```

## Sourcing the NodeJS environment

```sh
source /usr/local/nvm/nvm.sh
source /usr/local/nvm/bash_completion
```

## Generate wheels

If you haven't cloned the repository before, do it, otherwise, it should be mounted with a volume. Verify that you're using the default branch of the repository.

In first, generate kernels wheels files 

```sh
python3.6 -m pip wheel --wheel-dir $HOME/build/dependencies/ $HOME/git/sparkmagic/sparkmagic/
```

## Configuration files

Then, create the requirement file to install packages to get kernels configuration files. The requirement file must contain :

```sh
echo "pip>=21.3.1
sparkmagic==0.21.0" > $HOME/build/requirements.txt
```

Finally, generate the configuration files by installing pip packages.

```sh
python3.6 -m pip install --requirement $HOME/build/requirements.txt --no-index --find-links file://$HOME/build/dependencies
```

## Copy files

Create the arborescence and copy the kernels configuration files

```sh
cd $HOME/build
mkdir -p $HOME/build/share/jupyter/kernels/
cp -R /home/builder/.local/lib/python3.6/site-packages/sparkmagic/kernels/pysparkkernel/ $HOME/build/share/jupyter/kernels/
cp -R /home/builder/.local/lib/python3.6/site-packages/sparkmagic/kernels/sparkkernel/ $HOME/build/share/jupyter/kernels/
cp -R /home/builder/.local/lib/python3.6/site-packages/sparkmagic/kernels/sparkrkernel/ $HOME/build/share/jupyter/kernels/
```

## Create archive

Create the archive with wheel files, kernels configuration files and the requirement file, then create the checksum

```sh
cd $HOME
tar cf sparkmagic-0.21.0-0.0.tar.gz build/
sha256sum sparkmagic-0.21.0-0.0.tar.gz | sed 's#.*/# #' > sparkmagic-0.21.0-0.0.tar.gz.sha256
```

