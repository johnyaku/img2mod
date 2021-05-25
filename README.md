# img2mod

## Overview

This tool downloads a docker image and converts it to Singularity format (SIF).
SIF files are stored in an image cache specified by the $image_cache variable.
This variable is set by the modulefile for this tool, but can be overwritten to create a separate cache if desired.

The tool also creates a bash wrapper script for each image.
The name of the script is the same as the main command (tool) inside the the container, and the wrapper script simply passes all of its parameters on to the containerised command.
The net result is that the container becomes transparent to the user, and the user can use the wrapper script as if it were the containerised command (tool).

The path to the wrapper script is based on the command name and version (tag), as follows:
$image_cache/wrappers/$command/$tag/

The tool also creates a modulefile that prepends the above path to the $PATH variable.
By loading the module defined by this modulefile, the user can run the command (tool) without being aware of the underlying container.

## Usage

1. Create a configuration file

The make_config sript prompts the user for a "docker pull string".
This is the "docker pull ..." string that can be found in repositories such as DockerHub and Quay.io (Biocontainers).

The script then prompts the user for the containerised command.
Most of the time this is the same as the image name, and the user can simple press <Enter> to accept the default value.

Finally, the script prompts the user for a link to the reference manual for the command (tool).

A configuration file will be created in the current working directory, and the name of the file will be based on the command and version.

2. Create a bash wrapper and modulefile, and download the image

The make_mod script takes the configuration file from the previous step as its only parameter.
The script uses the values in the configuration file to create a wrapper script and modulefile, based on templates.
Both the wrapper and the modulefile are moved to the appropriate locations, as specified in the standard output.
Finally, the script submits a qsub job to download and convert the image to singularity format.
The user will receive email notifications when the job starts and finishes.

Note that the wrapper template binds the paths in the SINGULARITY_BIND environment variable.
Users who wish to bind custom paths or volumes are encourage to set this variable in their .bashrc file.
If this value is not set then /share will be bound by default.

## Special cases

### Fake home

Some tools look for certain configuration files in the user's home directory, such as .profile and .bashrc.
Singularity mounts $HOME by default, and this can cause odd behaviour with this type of tool. 
Accordingly, the default wrapper template includes the "--no-home" flag to avoid this kind of problem.

However, some tools also expect to *write* to $HOME and this expectation can be frustrated by the "--no-home" flag. 
Although removing the "--no-home" flag from the wrapper script may appear to alleviate this problem, this too can have unpredictable side effects. 
A better solution is to create a new directory at $image_cache/$command/$tag/fake_home and then bind this directory as follows: 
-B $image_cache/$command/$tag/fake_home:$HOME

This creates a writable space that will apear as $HOME within the container. 
If there are permissions collisions between multiple users of the containerised tool then it make be necessary for each user to create their own fake home (eg /share/ScratchGeneral/$USER/$command/fake_home) and to modify the wrapper script to point to this variable location.

### Unwritable paths

Some tools expect to be able to write to particular locations (log files, for example). 
Paths within the container are read-only, and so in order to accomodate this expectation it is necessary to create a directory to act as a writable space and then bind this to the appropriate path inside the directory.


