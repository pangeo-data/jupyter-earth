# JupyterHub

[JupyterHub](https://jupyterhub.readthedocs.io) is what organizations typically install (in a single server, cloud, or HPC) to provide Jupyter environments for a potentially very large group of users. Users can through a JupyterHub get started working in a Jupyter environment without any prior setup.

## Conceptual model of JupyterHub

![JupyterHub conceptual model](../images/2019-06-sdss-jupyterhub.png)

_Image credit: [Chris Holdgraf](https://github.com/choldgraf). Presented at the SDSS Conference. [Original slides](https://docs.google.com/presentation/d/1w-gvBn1c7Bos2ZPyKlBwMaO-caxgfpOwf6M00RtZ92c/edit#slide=id.g5a990f77bb_0_0)._

## JupyterHub distributions and deployments

A JupyterHub _distribution_ is an installation bundle including JupyterHub, and a JupyterHub _deployment_ can be thought of as a specific installation of JupyterHub. In this section we summarize two notable JupyterHub distributions with associated guides and some notable deployments.

### Single server

[The Littlest JupyterHub (TLJH)](https://tljh.jupyter.org) is a JupyterHub distribution to run on a single server, aimed at a smaller number of users (0-100). It can for example suitable for a class or a small research group. The documentation includes a section on [when to use TLJH](https://tljh.jupyter.org/en/latest/topic/whentouse.html#topic-whentouse) to determine if this is the right tool for you. The [installation instructions](https://tljh.jupyter.org/en/latest/install/index.html) are thorough and takes you from little experience and no server to having a having a server with a JupyterHub.

As each user server spawned (created) by a TLJH based JupyterHub will run on the same server, it is relatively easy for administrators work with shared storage and adjust the users' available libraries etc.

### Cloud

[Zero to JupyterHub on Kubernetes (Z2JH)](https://z2jh.jupyter.org/) is a JupyterHub distribution to run on a cloud ([Kubernetes cluster](https://kubernetes.io)) aimed at a larger group of users (100+) or a group of users with significant hardware needs. It can for example be suitable for an university or a research group performing work that a laptop would struggle with.

As each user server spawned (created) by a Z2JH based JupyterHub will be more isolated from each other (Docker containers), potentially on potentially different servers, it is relatively complicated to work with shared storage and adjust the users' available libraries etc in a Z2JH deployment.

### HPC

For installation of JupyterHub in _High-performance Computing (HPC)_ centers there are no official JupyterHub distribution, but there are useful components such as the [BatchSpawner, SlurmSpawner, and more](https://github.com/jupyterhub/batchspawner#readme) that are commonly used.

Here are some links to deployments on some of the major HPC facilities in the US that are accessed by geoscience researchers:
- NERSC ([login](https://jupyter.nersc.gov), [documentation](https://docs.nersc.gov/services/jupyter/))
- NCAR Cheyenne & Casper ([login](https://jupyterhub.ucar.edu/), [documentation](https://www2.cisl.ucar.edu/resources/jupyterhub-ncar))

If you would like add to this list, please feel free to [suggest a change in the source repository](https://github.com/pangeo-data/jupyter-earth).
