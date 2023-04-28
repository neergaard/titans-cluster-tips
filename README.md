# Remote development on `titans`

Author: Alexander Neergaard Zahid, aneol@dtu.dk

This document will show how to access and use the `titans` GPU cluster at DTU Compute.

## Preliminaries
Before you can access the SLURM GPU Cluster (titans), you need to be authorized.
Alex (aneol@dtu.dk) can help you with that.

## How to connect
If you are inside the DTU network, you can connect to the cluster using the following command:
```bash
> ssh <dtu_id>@titans
```
If the above does not work, or if you are outside the network, you need to connect through DTU Compute beforehand:
```bash
> ssh <dtu_id>@linuxterm1.compute.dtu.dk
> ssh titans
```
You will be greeted with a terminal prompt like this, which will indicate that you are on the `titans` cluster login node:
```bash
> ssh titans

_/\/\/\/\/\/\__/\/\/\/\__/\/\/\/\/\/\______/\/\______/\/\____/\/\____/\/\/\/\/\_
_____/\/\________/\/\________/\/\________/\/\/\/\____/\/\/\__/\/\__/\/\_________
_____/\/\________/\/\________/\/\______/\/\____/\/\__/\/\/\/\/\/\____/\/\/\/\___
_____/\/\________/\/\________/\/\______/\/\/\/\/\/\__/\/\__/\/\/\__________/\/\_
_____/\/\______/\/\/\/\______/\/\______/\/\____/\/\__/\/\____/\/\__/\/\/\/\/\___
________________________________________________________________________________

    https://titans.compute.dtu.dk


Last login: Fri Mar 10 09:54:53 2023 from 130.225.68.87
<dtu_id>@titans:~$
```

## Setting up your SSH config for easy access
The following will allow you to connect easily to `titans`.
1. Open (or create) the file `~/.ssh/config` in your favourite text editor.
2. Type in the following (insert relevant DTU ID and remember indentation)
    ```
    Host titans
        HostName titans
        User <dtu_id>
        ProxyJump compute

    Host compute
        HostName linuxterm1.compute.dtu.dk
        User <dtu_id>
    ```
3. Save the file

Now you should be able to run `ssh titans` from a terminal, which will setup the SSH connection automatically.
This sets up a connection to `titans` through the DTU Compute login node, by first connecting to `compute` and then to `titans`.

## Where to place data
As part of the Machine Learning for Neuroimaging Group (`macaroni`), you will have access to our group drive located at `/dtu-compute/macaroni` (`$GROUP_HOME`).
This is a folder where you can place data and share code with others in the group.
For convenience, we try to maintain a structured order:
```
aneol@titans:/dtu-compute/macaroni$ tree -L 1
.
├── data
└── projects
```
However, this is not strictly enforced, so if a project is using a type of data that is very specific to that project, the data itself can be located in the project folder instead of `./data`.

Note that `$GROUP_HOME` is a place to store **raw** data, and that any type of preprocessed data that you need to access often in your modeling scripts should be placed on the `/scratch/` space!

## Starting a batch job
In order to start a job on the cluster, you need to specify a batch script. 
Let's look at the contents of the batch script `my_script.sh`:
```bash
#!/bin/bash

#SBATCH --job-name=<job_name>
#SBATCH --time=<time>
#SBATCH -p {partition>
#SBATCH -w {reservation>
#SBATCH --cpus-per-task=<ncpus>
#SBATCH --gres=gpu:<gpus>
#SBATCH --mem=<memory>
#SBATCH --output=<path_to_logs>.out
#SBATCH --error=<path_to_logs>.err
##################################################

 # Change directory
cd <path_to_directory>

# Activate conda
source $GROUP_HOME/miniconda3/bin/activate

# Activate correct conda environment
conda activate <conda_env>

# Run command
python <my_python_script>
```
By running `sbatch my_script.sh` from the login node, the script will submit your job with the given parameters.
Here, no parameters or python commands are actually given, you need to modify to your use case!

## Setting up VSCode for remote development
It’s possible to develop and debug code directly on `titans` through the VS Code extension ‘Remote – SSH’.
This package is super useful, because you don’t need to have any code on your local machine at all.
To set it up, please perform the following steps.

### Configure 'Remote - SSH' extension:
1. Open VS Code and go to the ‘Extensions’ pane on the left hand side.
2. Search for ‘Remote – SSH’ and install it.
3. Now, you can press `Cmd+Shift+P` (`Ctrl+Shift+P`) and enter in the prompt: ‘Remote – SSH: Connect Current Window to Host…’.
4. Now, select the `titans` connection alias.
5. After a little while, enter your password and the connection should be done, and you can now open a directory by going to the File Explorer pane on the left and pressing ‘Open Folder’.
6. Optional: after opening the desired folder, I would advise you to save the current workspace as a workspace file; this way, you only have to open the workspace file in VS Code, and it will automatically setup the SSH connection and open the directory. To do this, go to ‘File’ -> ‘Save Workspace As…’, give a meaningful name and save it. I usually save my workspace files on Dropbox, so that I can access my workspaces from every machine I have Dropbox on.

### Set up helper SSH debug and connection aliases:
These commands are just super helpful when starting a debug session on a compute machine (not the login node).
Place them in your `~/.bashrc` or `~/.bash_profile` files on `titans` (if they don’t exist, create them using `touch ~/.bash_profile` or `touch ~/.bashrc`)
```
debug (){
    python -m debugpy –listen localhost:$1 –wait-for-client “${@:2}”
}
connect-node (){
    ssh -L $1:localhost:$2 $3
}
```

### Add a debug configuration in VSCode:
To start a debug session on a remote machine, we need to create a debug configuration that listens to a specific port (I usually use port 8883, but experiment and see whichever works for you)
1. In VS Code, open the Debug panel on the left side.
2. Press the gear icon, which will open a `launch.json` file.
3. Paste the following snippet into the `launch.json` in the `configurations` list:
    ```
    {
        “name”: “Python: Remote Attach”,
        “type”: “python”,
        “request”: “attach”,
        “port”: 8883,
        “justMyCode”: true,
        “host”: “localhost”,
        “pathMappings”: [
            {
                “localRoot”: “${workspaceFolder}”,
                “remoteRoot”: “.”
            }
        ]
    }

    ```

### Starting a debug session on a remote machine
Coming soon.
