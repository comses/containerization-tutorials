This tutorial shows how to create a Docker image of a NetLogo model and
how to run a container that will allow you to interact with its Graphic
User Interface (GUI). As a result, you will be able to open, modify and
run your containerized model as you would do with the original version.
You will also be able to create and run BehaviorSpace experiments in the
container, and results will be transferred to a folder of your
preference.

To start this tutorial, you will need to have
[Docker](https://www.docker.com/products/docker-desktop) installed in
your computer, as well as a .nlogo file with your working model. We will
use as an example the Wolf Sheep Predation model from the Model Library,
which we saved as WolfSheep.nlogo. Following our [suggested directory
structure](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1000424),
we created a project folder named `MyDocker` and it contains the folders
`data`, `docs`, `results`, and `src`. WolfSheep.nlogo is in the `src`
folder and the rest of the folders are empty.

Building the image
==================

------------------------------------------------------------------------

1. Determine what folder to use
-------------------------------

Docker looks for the files existing in the folder where you are when you
call the `build` command. Therefore, it is important to verify you are
able to navigate to the folder that contains your project using command
line. If that is the case, you can continue to step 2. Otherwise, we
suggest you to follow Step 1.1 to create a `MyDocker` folder that you
can later navigate to:

### Step 1.1 on Mac

1.  Use Finder to go to your home directory: go to Macintosh HD, then to
    Users and then to the folder with your user name (which should have
    a house icon).
2.  Create a folder named `MyDocker` in your home directory.
3.  Copy the contents of your project's folder to `MyDocker`. In our
    example, since our project consists of a single .nlogo file
    (WolfSheep.nlogo), only the `src` folder will contain any files, but
    we still need to have at least a `results` folder to store the
    results of our experiment.

### Step 1.1 on Windows

1.  Open File Explorer, go to My PC, then to the C: drive, then to
    Users, and finally your username.
2.  Create a new folder named `MyDocker`.
3.  Copy the contents of your project's folder to `MyDocker`. In our
    example, since our project consists of a single .nlogo file
    (WolfSheep.nlogo), only the `src` folder will contain any files, but
    we still need to have at least a `results` folder to store the
    results of our experiment

2. Include additional extensions
--------------------------------

If your model does not use extensions or uses the extensions that are
bundled with the NetLogo download (meaning that you did not have to
download anything to start using these extensions in your project), then
you can skip this and go to Step 3. Otherwise, it is necessary to follow
Step 2.1 to include a copy of the extensions you are using inside the
`src` folder of your project.

### Step 2.1 on Mac

1.  Use Finder to open the Applications folder, where you will find the
    folder that contains your NetLogo program (if you have multiple
    versions of NetLogo, be sure to open the right version).
2.  Go to the extensions folder and copy the folder corresponding to
    your extension. Note that you must copy the folder of the extension,
    not just the .jar file inside it. You might find useful the
    following description, taken from NetLogo's [Extensions
    Guide](https://ccl.northwestern.edu/netlogo/docs/extensions.html):

    > Each NetLogo extension consists of a folder with the same name as
    > the extension, entirely in lower case. This folder must contain a
    > JAR file with the same name as the folder. For example, the
    > `sound` extension is stored in a folder called `sound` with a file
    > inside called `sound.jar`.
    >
    > Some extensions depend on additional files. These files will be in
    > the extension’s folder along with the JAR file. The folder may
    > also contain other files such as documentation and example models.

3.  Paste the folder in the `src` folder of your project.

### Step 2.1 on Windows

1.  Use File Explorer to open the Program Files folder, where you will
    find the folder that contains your NetLogo program (if you have
    multiple versions of NetLogo, be sure to open the right version).
2.  Go to the app folder, then to extensions, and copy the folder
    corresponding to your extension. Note that you must copy the folder
    of the extension, not just the .jar file inside it. You might find
    useful the following description, taken from NetLogo's [Extensions
    Guide](https://ccl.northwestern.edu/netlogo/docs/extensions.html):

    > Each NetLogo extension consists of a folder with the same name as
    > the extension, entirely in lower case. This folder must contain a
    > JAR file with the same name as the folder. For example, the
    > `sound` extension is stored in a folder called `sound` with a file
    > inside called `sound.jar`.
    >
    > Some extensions depend on additional files. These files will be in
    > the extension’s folder along with the JAR file. The folder may
    > also contain other files such as documentation and example models.

3.  Paste the folder in the `src` folder of your project.

3. Create a Dockerfile
----------------------

The Dockerfile is the file that describes how to create the image of
your model and what steps to execute when you run a container based on
that image. To create one, open a plain text editor (examples are
TextEdit for Mac or Notepad++ for Windows) and copy the following code.

    FROM openjdk:8-jdk

    ARG MODEL_NAME
    ARG NETLOGO_VERSION
    ARG NETLOGO_NAME=NetLogo-$NETLOGO_VERSION
    ARG NETLOGO_URL=https://ccl.northwestern.edu/netlogo/$NETLOGO_VERSION/$NETLOGO_NAME-64.tgz

    ENV LC_ALL=C.UTF-8 \
        LANG=C.UTF-8 \
        DISPLAY=:14
        
    RUN mkdir /home/netlogoim \
     && wget $NETLOGO_URL \
     && tar xzf $NETLOGO_NAME-64.tgz -C /home/netlogoim --strip-components=1 \
     && rm $NETLOGO_NAME-64.tgz \
     && cp /home/netlogoim/netlogo-headless.sh /home/netlogoim/netlogo-headw.sh \
     && sed -i -e 's/org.nlogo.headless.Main/org.nlogo.app.App/g' /home/netlogoim/netlogo-headw.sh \
     && apt-get update && apt-get install -y libxrender1 libxtst6
        
    COPY . /home/

    RUN mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo

    CMD ["/home/netlogoim/netlogo-headw.sh", "/home/netlogoim/ModInImage.nlogo"]

If you are curious about the meaning of these lines, or how to get only
the table or the spreadsheet output, we included a short explanation
[below](#sections).

4. Save your Dockerfile
-----------------------

### Mac

1.  If using TextEdit, make sure it will be saved as plain text. To do
    this, you can click on Format and then on Make Plain Text, or just
    press "shift + command + T".
2.  Click on File, then Save... . In the dialog window that opens:
    -   Type `Dockerfile` as the name of the file.
    -   Choose the folder you are going to use (Step 1) as the location
        of the file. In our example, we would save the `Dockerfile` in
        `MyDocker`.
    -   Make sure that the format of the file (in the drop-down menu
        below) is plain text (or Unicode UTF-8).
3.  Open Finder and look for your `Dockerfile`. It should show up in the
    folder you chose to use in Step 1 (in our example, in `MyDocker`)
    and it should not include any extensions in its name (if it has a
    name like `Dockerfile.txt`, you should rename it to `Dockerfile`)

### Windows

1.  Save your plain text as `Dockerfile` (do not include any extensions
    in the file name, e.g. .txt) and make sure it is saved in the folder
    you are going to use (Step 1). In our example, we saved our
    `Dockerfile` in `MyDocker`.
2.  Use File Explorer to check that the file was saved with the right
    name (`Dockerfile`, no extensions) and in the right location, i.e.
    the folder you chose to use in Step 1 (`MyDocker` in our example).

5. Double-check your directory structure
----------------------------------------

The code in the `Dockerfile` assumes a specific organization within the
folder containing your Dockerfile. Either the `build` or the `run`
command will fail if it cannot find the right files due to an
organization or folder naming that are different than expected.
Therefore, it is necessary to ensure that the right contents are in the
right folders. Your project's folder (`MyDocker` in our example) can
have any name, but it must at least contain:

-   Your `Dockerfile`, and this is also a good time to check that its
    name does not have any extensions,
-   A `results` folder (in lower case), that in our case will be empty,
-   And a `src` folder (also in lower case), that contains your .nlogo
    file and, if using extensions that were not bundled in the NetLogo
    version that you are using, the folders of such extensions.

Your project's folder can contain other folders, like `data` if it uses
input files.

6. Build the Docker image
-------------------------

Now we are going to create the virtual entity that will download
NetLogo, contain your model, and be ready to execute any experiment that
you choose (from the list of BehaviorSpace experiment(s) that you saved
in your NetLogo file). To build the image, open a terminal window
(Terminal in Mac or Command Prompt in Windows) and navigate to the
folder that contains your model and the Dockerfile (if not sure how to
do so and followed Step 1.1, you can follow these [short
instructions](#navigate)). Once you moved to the directory that contains
your model and the Dockerfile, type the following `build` command,
replacing the `< >` spaces as directed:

    docker build --build-arg MODEL_NAME=<your file> --build-arg NETLOGO_VERSION=<your version> -t <image name> . 

-   Replace `<your file>` with the name of your NetLogo model file,
    including the .nlogo extension. In our example it would be
    `WolfSheep.nlogo`.
-   Replace `<your version>` with the version of NetLogo that you are
    using to run your model (in our case, `6.1.1`). If you are not sure
    what is the NetLogo version you are using, you can check the name of
    your NetLogo application and copy the number that goes after the
    word 'NetLogo'.
-   Replace `<image name>` with the name you want to give to your image.
    It can be any name, as long as all the letters you use are lower
    case, with no spaces.

In our example, this build command would look like this:
`docker build --build-arg MODEL_NAME=WolfSheep.nlogo --build-arg NETLOGO_VERSION=6.1.1 -t wolfsheepim .`.
Make sure you are *not* leaving any spaces between the `=` sign and the
file name or version number you inserted, and to include the `.` at the
end of the command.

When you press return, Docker will start showing the progress of the
processes you instructed it to perform with the Dockerfile. Keep in mind
that it has to download some programs from scratch, so building the
image could take some minutes. When it is done, you will see the line
`Successfully tagged <image name>:latest`. Now you will be able to run
reproducible BehaviorSpace experiments in your containerized model.

We included some known errors that can arise when building the image in
our short [Troubleshooting section](#trblshoot).

Running your container
======================

------------------------------------------------------------------------

1. Determine the folder where the experiment results should be stored
---------------------------------------------------------------------

When you run the container, you will be able to create and run
BehaviorSpace experiments. However, the results from such experiments
are only saved inside the container and will not be accessible once you
stop the container, unless you specify a folder in your computer's file
system where any obtained files (e.g. tables or spreadsheets) should be
transferred. It is not necessary to choose a location within your
project's folder, but doing so can help with the organization of your
results.

2. Run the container
--------------------

To be able to visualize and interact with NetLogo's GUI, it is necessary
to make the connection between your container and your computer. To do
this, we are going to use another image named
[docker-x11-bridge](https://github.com/JAremko/docker-x11-bridge). First
a container from the bridge image is started and then the container from
the image you just created is run.

Open a terminal window and type the first `run` command:

    docker run -d --name x11-bridge -e MODE="tcp" -e XPRA_HTML="yes" -e DISPLAY=:14 -p 10000:10000 jare/x11-bridge

This will pull the existing `docker-x11-bridge` image, which should take
a minute, and start a container with the name `x11-bridge`. Now you can
type the second `run` command:

### Mac

    docker run --name netlogo --volumes-from x11-bridge -v <path/to/your/results/folder>:/home/results <image name> 

Where:

-   `<path/to/your/results/folder>` is replaced by the *absolute* path
    to your results folder in your computer. If the folder does not
    exist, Docker will create it. In our case, we are going to create a
    folder named `TestResults` inside `MyDocker/results/`. Therefore,
    our path would be `~/MyDocker/results/TestResults`. Note that your
    path has to be absolute, or, in other words, start with `~/`.
-   `<image name>` is replaced by the name you gave to the image in the
    [`build` command](#buildc) (in our case, `wolfsheepim`).

In our example, the `run` command would look like this:
`docker run --name netlogo --volumes-from x11-bridge -v ~/MyDocker/results/TestResults:/home/results/ wolfsheepim`.

Now open a new tab or window of your browser of preference (e.g. Chrome,
Firefox) and type `http://localhost:10000/` in the search bar. You
should see your NetLogo model and be able to interact with it as usual
(although a little bit slower).

### Windows

    docker run --name netlogo --volumes-from x11-bridge -v <path/to/your/results/folder>:/home/results <image name> 

Where:

-   `<path/to/your/results/folder/>` is replaced by the *absolute* path
    to your results folder in your computer. If the folder does not
    exist, Docker will create it. In our case, we are going to create a
    folder named `TestResults` inside `MyDocker/results/`. Therefore,
    our path would be
    `c:/Users/<yourusername>/MyDocker/results/TestResults`. Note that
    your path has to be absolute, or, in other words, start with `c:/`.
    Also be sure to replace `<yourusername>`with your own user name.
-   `<image name>` is replaced by the name you gave to the image in the
    [`build` command](#buildc) (in our case, `wolfsheepim`).

In our example, the `run` command would look like this:
`docker run --name netlogo --volumes-from x11-bridge -v c:/Users/myusername/MyDocker/results/TestResults:/home/results/ wolfsheepim`.

Now open a new tab or window of your browser of preference (e.g. Chrome,
Firefox) and type `http://localhost:10000/` in the search bar. You
should see your NetLogo model and be able to interact with it as usual
(although a little bit slower).

Navigating to your folder of interest
=====================================

------------------------------------------------------------------------

If you are not familiar with the use of command line to move between
folders (directories) and followed Step 1.1, you can do the following:

Mac
---

1.  Open Terminal: press command + space bar, type "Terminal" and open
    it.
2.  Type `cd ~/MyDocker`

Now you can [continue building](#buildc) the Docker image. You can visit
websites like
[this](https://www.macworld.com/article/2042378/master-the-command-line-navigating-files-and-folders.html)
to continue becoming familiar with the command line.

Windows
-------

1.  Open [Command
    Line](https://www.computerhope.com/issues/chusedos.htm)
2.  Type `cd C:\Users\<your username>\MyDocker`, where `<your username>`
    is replaced by your own username.

Now you can [continue building](#buildc) the Docker image.

 Sections of this Dockerfile
============================

------------------------------------------------------------------------

`FROM`
------

Dockerfiles start with a `FROM`, followed by a base image. Since we are
not going to create everything from scratch, we need to establish what
image we are going to use to build our new image from. In this case, we
are going to use an image of OpenJDK. Therefore, our Dockerfile starts
with `FROM openjdk:8-jdk`.

`ARG` and `ENV`
---------------

ARGuments are pieces of information that may be given to Docker when the
`build` command is executed, but can also be set to some default values.
In our case, we are leaving the first two arguments (`MODEL_NAME` and
`NETLOGO_VERSION`) open for modifications in the [`build`
command](#buildc) because these are expected to be modified by each
user, while the second two (`NETLOGO_NAME` and `NETLOGO_VERSION`) were
assigned default values. You could also overwrite the second two
arguments in you `build` command by adding to it
`--build-arg <ARG>=<new value for this ARG>`, although there is no need
to do that.

While ARGuments are only stored at the time the image is built and then
discarded, ENVironment variables are kept in the image. In our case, we
are using `ENV` to set the coding to a standard format, to avoid any
incompatibilities.

`RUN` and `COPY`
----------------

In this section we are telling Docker the steps it has to follow to
build the image. In short, we are asking it to create some folders and
to download and unzip our specified NetLogo version. Afterwards, we are
asking it to copy our NetLogo model to a folder in the image. In other
words, this is the step in which our model is immortalized in the image
we built.

`ENTRYPOINT`
------------

The final step is to tell the image what to do when we use it to run a
container. We use `ENTRYPOINT` to tell it to run `netlogo-headless.sh`,
with the arguments `model`, `table`, and `spreadsheet` as specified.
When we use the [`run` command](#runc), we are specifying one extra
argument, `experiment`, to be the name of one of the available
BehaviorSpace experiments. This way, different experiments can be run in
the same model image.

If you are not interested in one type of output from your experiments
(notice this will modify the outputs of all the experiments that you run
using that image), you can delete the line corresponding to it. For
example, if you only wanted to get results in the table format and not
the spreadsheet, then your `ENTRYPOINT` section would look like the
following:

    ENTRYPOINT ["/home/netlogoim/netlogo-headless.sh", \
     "--model", "/home/netlogoim/ModInImage.nlogo", \
     "--table", "/home/netlogoim/results/table-output.csv"]

Troubleshooting Guide
=====================

------------------------------------------------------------------------

<details> <summary>Click here to see some known error messages and their
probable causes</summary>

### Your `build` command is not correct

When trying to build your image, you get an error message like the
following:

    "docker build" requires exactly 1 argument.

It means that your `build` command has some typing error. The most
likely ones are that it has spaces at any side of the `=` signs or that
it is lacking the last `.`.

### Your image name contains upper case letters.

    invalid argument "WolfSheepIm" for "-t, --tag" flag: invalid reference format: repository name must be lowercase

Use only lower case leters.

### Your Dockerfile is not in plain text format

When trying to build your image, you get an error message like the
following:

    Error response from daemon: Dockerfile parse error line 1: unknown instruction: {\RTF1\ANSI\ANSICPG1252\COCOARTF1671\COCOASUBRTF600

You will need to convert the format of your file to plain text. If you
are using TextEdit, open your Dockerfile, press shift + command + T and
save it. Use Finder to make sure that the name of the file does not
include any extensions.

### The Dockerfile was saved under a different name

When trying to build your image, you get an error message like the
following:

    unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat ~/MyDocker/Dockerfile: no such file or directory

Use Finder or File Explorer to check that the name of the file was
correctly spelled (`Dockerfile`), and it does not end with any
extensions. If your file's name looks like `Dockerfile.txt`, you have to
edit its name to erase the `.txt` termination. If your file's name ends
with a `.rtf`, go to the previous error to see how to correct it.

If you are not able to erase the extension from your `Dockerfile`'s
name, you can also add a `-f` flag to your `build` command. With this
flag you can tell Docker the name of your Dockerfile. In our example,
and assuming we have a file named `Dockerfile.txt`, our build command
would be
`docker build --build-arg MODEL_NAME=WolfSheep.nlogo --build-arg NETLOGO_VERSION=6.1.1 -f Dockerfile.txt -t wolfsheepim .`.
Note that it is necessary to have a Dockerfile in plain text format to
be able to build a Docker image.

### The name of your .nlogo file was mispelled

When trying to build your image, you get an error message like the
following:

    Step 9/10 : RUN rmdir /home/netlogoim/results  && mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo
     ---> Running in (12-digit number)
    mv: cannot stat '/home/src/WolfSheeppRnd.nlogo': No such file or directory
    The command '/bin/sh -c rmdir /home/netlogoim/results  && mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo' returned a non-zero code: 1

This means that the name of the model you provided in the `build`
command, i.e. the name after `--build-arg MODEL_NAME=`, is not identical
to the name of your .nlogo file. Check the spelling and that it includes
the `.nlogo` extension.

### The name of the BehaviorSpace experiment is wrong

When trying to run a container from your image, you get an error message
from NetLogo:

    Exception in thread "main" java.lang.IllegalArgumentException: Invalid run, specify experiment name or setup file

Verify that the name of the experiment you are asking the container to
run is correctly typed and that it exists in the .nlogo file that you
used to build the image. </details>