# Run NEURON from JupyterLab inside a Docker container

This repository provides the Docker recipe to build an image providing
a JupyterLab environment with NEURON bindings.
It also contains a sample notebook, provided by
[@sharkovsky](https://github.com/sharkovsky) as a courtesy.

Jupyter Lab is started when the Docker container starts. The container
has the following specifities:
* content of `./notebooks` directory is mount in the container in
`/opt/src/notebooks` (the Jupyter Lab root directory).
* files created within the container in this directory will belong to the
  user on the host machine.
* SSH identity in the host is available in the container so that a
  Jupyter notebooks in the container are able to fetch private resources  available through SSH protocol.

The repository provides a Jupyter notebook that:
1. downloads public Allen Cell Types Database model
1. downloads a private model to highlight features described above
1. builds NEURON C files with `nrnivmodl`
1. runs a simulation

## Requirements

You must have:
* Docker installed and running.
* [Docker Compose](https://docs.docker.com/compose) utility installed.

## Installation

To retrieve the Docker image, you can either download it from Docker Hub or build it yourself.

### Pull from Docker Hub

You only need the Docker Compose configuration to get started:
```bash
mkdir jupyterlab-neuron
cd jupyterlab-neuron
wget https://raw.githubusercontent.com/tristan0x/jupyterlab-neuron/master/docker-compose.yml
docker-compose pull
```

Optionally you can also retrieve the demo Python notebook:
```
mkdir notebooks
cd notebooks
wget https://raw.githubusercontent.com/tristan0x/jupyterlab-neuron/master/notebooks/single-neuron-DetAMPAStim.ipynb
```

### Build manually

You need to clone this repository to retrieve the necessary to build the Docker
image:

```bash
git clone https://github.com/tristan0x/jupyterlab-neuron.git
cd jupyterlab-neuron
docker-compose build
```

## Basic Usage

To start a JupyterLab Docker container, you must first ensure that
an SSH agent is running. It will be used to provide the running container
access to your identity to grab private ressources.

```bash
$ echo $SSH_AUTH_SOCK
/run/user/188063/keyring/ssh
```

If the variable is not defined, you can start an SSH agent and register
your SSH identity with the following commands:

```bash
$ eval `ssh-agent`
$ ssh-add
Enter passphrase for /path/to/your/default/ssh/key:
Identity added: /path/to/your/default/ssh/key (/path/to/your/default/ssh/key)
$
```

You can register additional SSH keys to the agent with the `ssh-add /path/to/private/key` command.

Now you can use `docker-compose` to start the container:

```bash
export UID GID=`id -g`
docker-compose up jupyterlab
Creating jupyterlabneuron_jupyterlab_1 ... done
Attaching to jupyterlabneuron_jupyterlab_1
jupyterlab_1  | [I 06:52:11.383 LabApp] Writing notebook server cookie secret to /home/dummy/.local/share/jupyter/runtime/notebook_cookie_secret
jupyterlab_1  | [I 06:52:11.581 LabApp] JupyterLab extension loaded from /opt/conda/lib/python2.7/site-packages/jupyterlab
jupyterlab_1  | [I 06:52:11.581 LabApp] JupyterLab application directory is /opt/conda/share/jupyter/lab
jupyterlab_1  | [W 06:52:11.586 LabApp] JupyterLab server extension not enabled, manually loading...
jupyterlab_1  | [I 06:52:11.587 LabApp] JupyterLab extension loaded from /opt/conda/lib/python2.7/site-packages/jupyterlab
jupyterlab_1  | [I 06:52:11.587 LabApp] JupyterLab application directory is /opt/conda/share/jupyter/lab
jupyterlab_1  | [I 06:52:11.590 LabApp] Serving notebooks from local directory: /opt/src/notebooks
jupyterlab_1  | [I 06:52:11.591 LabApp] The Jupyter Notebook is running at:
jupyterlab_1  | [I 06:52:11.591 LabApp] http://localhost:8888/?token=e1fc2e039ca063ba873abe87e6d97f1f94086b9fbcc83ff5
jupyterlab_1  | [I 06:52:11.591 LabApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
jupyterlab_1  | [C 06:52:11.591 LabApp]
jupyterlab_1  |
jupyterlab_1  |     Copy/paste this URL into your browser when you connect for the first time,
jupyterlab_1  |     to login with a token:
jupyterlab_1  |         http://localhost:8888/?token=e1fc2e039ca063ba873abe87e6d97f1f94086b9fbcc83ff5
```

Then open your web browser at the provided HTTP address. In this case
http://localhost:8888/?token=e1fc2e039ca063ba873abe87e6d97f1f94086b9fbcc83ff5

## Behind the Hood

* The SSH agent UNIX socket is mounted in the container so that it is possible
to access private resources from inside the container, GIT repositories for
instances.
* A fake user with the user and group identifier is created inside the container
and used by JupyterLab instance to read and write files. It ensures that
data written to the `./notebooks` directory will belong to you.
* The container does not use the default Docker bridge network interface but
instead use the host net namespace. Rationale behind that is that the URL written
to standard output by JupyterLab when it starts will work in your browser.
* Did you know that `UID` and `GID` are standard Linux variables but are
not actually exported by your shell?

    ```bash
    $ echo $UID
    188063
    $  python2 -c "import os; print 'UID' in os.environ"
    False
    ```

## License

This project is licensed under the BSD License. See the [LICENSE](./LICENSE) file
for more details.
