In CoMSES we believe that the containerization of models should become a
new standard. In this section we will explain what we mean by
containerization, what is Docker, and why you should use it. We also
provide a list of tutorials so you can dockerize your model written in
languages like NetLogo, R and Python.

Why should you use this (and keep reading on)?
----------------------------------------------

One of the main advantages of using containers is their portability,
which is an issue that we have to face frequently. For example, it is
still somewhat common to see presentations where the supporting slides
look distorted or their formatting is evidently wrong. This is a case of
poor portability: the same file can produce distinct results in
different computers, due to differences between the versions of the
software (be it PowerPoint, NetLogo or any other), the operating system
they are designed for, or even the versions of extra components it uses
(plug-ins, NetLogo extensions, R packages, etc.). In our example, the
same file produces different versions of the slides depending on the
computer that is used to open it, which is usually not a very serious
problem, but the same can apply to any computational model, potentially
undermining the reproducibility of its results.

Being able to produce a container of your model (or your slides) allows
you to run it in different machines in a reliable way, irrespectively of
the programs that these machines can have installed (as long as they
have the software that allows them to run the containers; more on that
[later](#dockersec)). This is useful if you are developing your model in
a team, or if you plan to run it on a high troughput computing service.
The container includes the right versions of the software and extensions
or packages that should be used, as well as your code, which allows it
to produce uniform results in any computer that runs it. On a more
general perspective, using containers is useful if you want your code to
be runnable even after 4 or 5 years and still generate reproducible
results, even when the software versions used now become obsolete, in
case future researchers would want to build on your work (which in the
spirit of good science should be one of our main aims).

As a final note, since each container includes the right version of the
software to run the code, using containers allows you to run models
requiring different versions of software without needing to change
anything in your own computer configuration. Also, you can use this to
test newer or older versions of software.

What exactly is a container?
----------------------------

The following is the definition given by
[Docker](https://www.docker.com/resources/what-container) (more on
Docker in the [next section](#dockersec)):

> A container is a standard unit of software that packages up code and
> all its dependencies so the application runs quickly and reliably from
> one computing environment to another.

In other words, a container can be understood as a bundle composed by
your code and the software environment it uses, plus any additional
extensions or packages it needs to run. The contents of a container are
isolated from the context of the computer that runs it. This means that
there is no communication between the code run in a container and the
data or programs present in the computer, save for the cases where the
user explicitly creates a connection between a folder of his machine and
one in the container. In those cases, the shared information is limited
to the files present in the designated folders.

Before the advent of containers, the issue of isolation used to be
solved through Virtual Machines (VM). As containers, VMs do not have
communication with the computer that runs it, but they require the
installation of an separate operating system as the foundation of their
environment. On the contrary, containers run on top of the operating
system, which makes them lightweight and easy to run.

A container runs from a previously built container image (or just
"image"), which is a collection of the necessary files and information
of the relationships between them, that is ready to be run in any
computer. One useful analogy is to think of an image as a recipe (or a
recipe that somehow includes the necessary ingredients), and the
container as the action of cooking the plate. Images can be easily
shared and also reused to build new images, using the existing one as a
base.

What is Docker?
---------------

As we were saying, containers are made to be able to run anywhere,
because they include all the necessary software, except for the platform
that is able to interpret the instructions embedded in the container
image. One of the most used platforms is
[Docker](https://www.docker.com). This is the preferred tool for many
users because it is a platform built on open source technologies. It has
the additional advantage of offering an extensive library of curated
images, created by software vendors, open-source projects and the
community. These images can be easily used as a base to create the
environment to run your code.

Docker Desktop, the Docker app for Windows and MacOS, can be downloaded
from this [link](https://www.docker.com/products/docker-desktop).
