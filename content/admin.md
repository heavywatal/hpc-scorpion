+++
title = "Admin"
menu = "main"
draft = true
+++

## Power

### Boot

1.  Start the head node.
1.  Start the compute nodes.

### Shutdown

1.  Notify the users.
1.  Run `sudo shutdown -h now` to stop the compute nodes.
1.  Run `sudo shutdown -h now` to stop the head node.


## Documentation

### Preparation

1.  Install [Hugo](https://gohugo.io/) on your local machine.

1.  Fork [heavywatal/hpc-scorpion](https://github.com/heavywatal/hpc-scorpion) to your account.

1.  Clone your fork repository to your local machine:

    ```sh
    REPO=https://github.com/{YOUR_NAME_HERE}/hpc-scorpion.git
    git clone -b master --single-branch --recurse-submodules $REPO
    cd hpc-scorpion/
    ```

1.  Set `upstream` repository:
    `git remote add upstream https://github.com/heavywatal/hpc-scorpion.git`


### Routine

1.  Fetch and merge any updates in `upstream` to your `origin`.

1.  Start a local hugo server to preview the output:
    `hugo -Dw server`<br>
    - View: http://localhost:1313/hpc-scorpion/
      (the port may vary)
    - Stop: <kbd>ctrl</kbd><kbd>c</kbd>

1.  Edit some markdown files in `content/`.
    The output HTML gets updated immediately by the hugo server.

1.  Make a new branch to commit the updates.

1.  Make a Pull Request to [heavywatal/hpc-scorpion](https://github.com/heavywatal/hpc-scorpion).


### Deploy document

```sh
make public && make deploy
```

1.  Generate public documents.

1.  Copy generated documents to `scorpion:/var/www/html/`.



## Add a new user

1.  Check a new entry on [Google Form](https://docs.google.com/forms/d/1Lb1Mu07HyLxTPJO2IPQPoYidoHg_VXgB46zeS2lxQJs/edit)

1.  Create an account:

    ```sh
    NEWUSER=______
    sudo adduser ${NEWUSER}
    ```

    Generate random password (e.g., copy partial sequence from ssh public key),
    and forget it.

1.  Update NIS:

    ```sh
    sudo make -C /var/yp
    sudo ypbind -c
    sudo ypcat passwd
    ```

1.  Configure SSH:

    ```sh
    sudo mkdir /home/${NEWUSER}/.ssh
    sudo vim /home/${NEWUSER}/.ssh/authorized_keys
    sudo chmod 700 /home/${NEWUSER}/.ssh
    sudo chmod 600 /home/${NEWUSER}/.ssh/authorized_keys
    sudo chown -R ${NEWUSER}:users /home/${NEWUSER}/.ssh
    ```

1.  Add the user (and his/her mentor) to
    [scorpion-tohoku](https://groups.google.com/forum/#!forum/scorpion-tohoku):
    "Manage" => "Direct add members".
    Message example:

    ```
    Your email address has been registered to scorpion-tohoku mailing list.
    Various notifications such as server maintenance and updates will be delivered.
    You can also post questions and requests here.
    ```

1.  Send an email to them:

    ```
    Dear ______,
    CC: Prof. ______,

    You have been successfully registered as a user of Scorpion system.
    Try logging in to the server with the following command:

    ssh scorpion
    or
    ssh ______@scorpion.biology.tohoku.ac.jp

    Best,
    Watal
    ```

### Remove a user

1.  `sudo userdel --remove ${THE_USER}`
1.  Update NIS
1.  Remove the user from [scorpion-tohoku](https://groups.google.com/forum/#!forum/scorpion-tohoku):
    "☑ Select => ⊖ Remove member"


## Initial settings

-   Modify `/etc/adduser.conf`:
    ```sh
    USERGROUPS=no
    DIR_MODE=0700
    ```
-   Clean `/etc/skel/`.
-   Create `/etc/profile.d/scorpion.sh` to set `PATH` for all the users.
-   Disable `motd`:
    ```sh
    cd /etc/update-motd.d
    sudo mkdir disabled
    sudo mv *-* disabled/
    ```


## Install softwares

### apt

```
sudo apt update
sudo apt install build-essential zsh emacs g++-8 clang-8
sudo apt install libblas-dev liblapack-dev libatlas-base-dev f2c
```

### Homebrew

https://docs.brew.sh/Homebrew-on-Linux

- If the software is available on Homebrew, use it.
- `brew` must be executed by a non-root user.
- Unlink `gcc` to remove it from `PATH`.
- Append `eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)` to `/etc/profile.d/scorpion.sh`.


### Python

Install python3 and some packages:
```sh
sudo apt install python3.8
brew install python
pip3 install --upgrade pip setuptools wheel
pip3 install pandas scikit-learn seaborn biopython
pip3 install flake8 requests psutil Pillow
```

Check updates:
```sh
pip3 list --outdated
```

### R

- https://cran.r-project.org/bin/linux/ubuntu/
- https://cran.r-project.org/doc/manuals/R-admin.html

Install R from the official repository:
```sh
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
echo "deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/" | sudo tee /etc/apt/sources.list.d/cran.list
sudo apt update
sudo apt install r-base r-base-dev
```

Install R with Homebrew: `brew install r`

Set `R_LIBS_USER` in `/etc/profile.d/scorpion.sh` in the all nodes:
```sh
export R_LIBS_USER='~/.R/library/%v'
```

Create `%v/site-library` and symlink it to the right place:
```sh
mkdir -p /home/linuxbrew/R/4.0/site-library
ln -s /home/linuxbrew/R/4.0 $(brew --prefix)/lib/R/
# make sure it is properly symlinked
ls -l $(brew --prefix)/opt/r/lib/R/
Rscript -e '.Library.site'
```

Put `Renviron.site` and `Rprofile.site` in `${R_HOME}/etc/` if necessary.

Install some `-dev` packages:
```sh
sudo apt install libssl-dev libv8-dev
```

Install packages to site library:
```r
pkgs = c(
  "Rcpp",
  "devtools",
  "tidyverse",
  "cowplot",
  "gridExtra",
  "igraph",
  "ape",
  "rstan",
  "doParallel",
  "BiocManager"
)
install.packages(pkgs, lib = .Library.site)

biocpkgs = c(
  "Biostrings",
  "GenomicRanges",
  "rtracklayer",
  "VariantAnnotation",
  "edgeR",
  "topGO"
)
BiocManager::install(biocpkgs, lib = .Library.site)
```

Check updates from time to time:
```r
BiocManager::valid(lib = .Library.site)
BiocManager::install(lib = .Library.site)
```

### Others

- Download an archive to `/home/local/src/`
- Install it with a prefix `/home/local`
- If the software does not provide any installation method,
  move the whole directory to `/home/local/Cellar/` with a version number,
  and create symlinks to `/home/local/bin`, `include`, `lib`, and so on.


## PBS

`/etc/pbs.conf` in head node:
```ini
PBS_EXEC=/opt/pbs
PBS_SERVER=scorpion
PBS_START_SERVER=1
PBS_START_SCHED=1
PBS_START_COMM=1
PBS_START_MOM=0
PBS_HOME=/var/spool/pbs
PBS_CORE_LIMIT=unlimited
PBS_SCP=/usr/bin/scp
```

`PBS_START_MOM=1` in compute nodes.

Some config files and logs are stored in `$PBS_HOME` (`/var/spool/pbs/`).


### Configuring the Server and Queues

```sh
man /opt/pbs/bin/qmgr

# interactively
qmgr

# with stdin
echo "print server" | qmgr
qmgr < input_file

# with command-line arguments
qmgr -c "print server"
qmgr -c "set server job_history_enable=True"
qmgr -c "set server job_history_duration=720:00:00"
```


## Install hardwares

-   https://www.hpc.co.jp/
