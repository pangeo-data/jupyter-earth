## What is this?

This is the home of the Dockerfile that builds into the base environment used in
https://hub.jupytearth.org. The cloud infrastructure declaration and Helm chart
configuration for https://hub.jupytearth.org lives in
https://github.com/2i2c-org/pilot-hubs/.

### How to work with this Dockerfile?

Whenever a pull request is made modifying the image, a GitHub Workflow will try
verify if it can successfully rebuild.

Whenever a pull request is merged modifying the image, a GitHub Workflow will
build and push the image and emit a message on how to update
https://hub.jupytearth.org to start making use of the new image. In short, it is
to visit https://hub.jupytearth.org/services/configurator and enter the image
name:tag which is described in the executed GitHub Workflow.
