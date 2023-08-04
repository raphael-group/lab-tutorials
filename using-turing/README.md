# Using turing

## Motivation 
Our lab's personal server on the cluster can only be accessed via another node
on the Princeton cs cluster, i.e. one of the cycle machines (at least to my
current understanding). Usually to access `turing` one has to ssh into 
one of the cycles machines and then from there ssh into `turing`. 

The instructions below show how to use control sockets/multiplexing (a concept that will not
be explained here, for more reading see [here](https://github.com/PrincetonUniversity/removing_tedium/blob/master/01_suppressing_duo/README.md#ii-multiplexing-approach-vpn-free)). to ssh into `turing` from your
local machine (like a laptop) with just:
* `ssh turing`

Setting up tunnels to `turing` for use with jupyter notebooks/lab becomes
significantly easier and will be the main showcase of this tutorial.

## Modifying your `~/.ssh/` folder

On your local machine (for example, your laptop) run the following command:
* `mkdir -p ~/.ssh/controlmasters ~/.ssh/sockets`

Moreover, modify your `~/.ssh/config` file to include the following lines
replacing <Your Princeton netid> with your netid, e.g. `aas4` and
`IdentityFile` to the path for your private key to the cycles machines.:
```
Host cycles.cs.princeton.edu cycles
  HostName cycles.cs.princeton.edu
  User <Your Princeton netid>
  IdentitiesOnly yes
  IdentityFile ~/.ssh/id_cycle # location of your ssh key for cycles
  ControlMaster auto
  ControlPersist yes
  ControlPath ~/.ssh/sockets/%p-%h-%r
  ServerAliveInterval 300

Host turing.cs.princeton.edu turing
  User <Your Princeton netid>
  IdentitiesOnly yes
  IdentityFile ~/.ssh/id_cycle # location of your ssh key for cycles
  HostName turing.cs.princeton.edu
  ProxyJump <Your Princeton netid>@cycles.cs.princeton.edu
  ControlMaster auto
  ControlPersist yes
  ControlPath ~/.ssh/sockets/%p-%h-%r

```

Assuming that you have generated ssh key pairs between:
* Your local machine and `cycles`
* `cycles` and `turing`

Running `ssh turing` now should only require one duo push and you will be on
`turing`. 

I have included an example file with the exact changes I have made to my local `~/.ssh/config` in
the file `example-config`.

## Using jupyter on `turing` to take advantage of the GPUs
The script `run-jupyter-template.sh` generates commands for how to ssh tunnel
into a jupyter notebook/lab and submits a slurm job to use a specific node on
the cs cluster for a maximum of one week before terminating.

The template script defaults to `turing`; the only thing you should modify is
which conda environment you want to use/module (use conda environemnets usually more flexible/easier).

I have generated a very simple conda environment called `turing-jupyter-tutorial` that has only downloaded 
pytorch using the gpu option (CUDA 11.7) with the following command (after creating the empty environment): 
* `mamba install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia`

In addition I installed `jupyter` and `ipykernel` to the environment:
* `mamba install jupyter ipykernel`

Usually it is sufficient to just conda/mamba install `ipykernel` to your environment for `jupyter` 
to auto detect the conda environment to use; I usually run the following command in the environment
to guarantee that my conda environment will be detected by jupyter:
* `python -m ipykernel install --user --name turing-jupyter-tutorial`

Putting this all together if we run `sh run-jupyter-template.sh` it will spit out something like:
* `ssh -N -L 12244:turing:12244 aas4@turing.cs.princeton.edu`
that you should copy and paste to your local machine terminal in a new window/tab (*nothing should happen*)
scrolling down to the bottom it should also prompt you to copy and paste something like the following "url"
in your local machines browser:
* `http://127.0.0.1:14108/lab?token=6e81f392f1cf1f30d9905f84de40f5a61b46751c10dfceb1`

where you should be able to open and run the notebook `check-pytorch-gpu.ipynb` (make sure to use the 
`turing jupyter-tutorial` kernel).
