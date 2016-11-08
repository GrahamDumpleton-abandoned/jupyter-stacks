# Jupyter Project Images for OpenShift

This repository provides versions of the Jupyter project images which have been fixed up to be able to work on OpenShift.

The Jupyter project provides Docker-formatted container images via their [GitHub project](https://github.com/jupyter/docker-stacks) and on [Docker Hub](https://hub.docker.com/u/jupyter/).

The images that the Jupyter project provides will not work with the default security profile of OpenShift. This is because the Jupyter project images, although they have attempted to set them up so they do not run as ``root``, will not run in a multi tenant PaaS environment where any container a user runs is forced to run with an assigned ``uid`` different to that specified by the image.

The issues preventing the Jupyter project images being able to be run in a default installation of OpenShift have been [reported](https://github.com/jupyter/docker-stacks/issues/188) via the GitHub project but at this point have not been addressed.

So that you can still use the Jupyter project images on OpenShift, this repository provides derived images which apply the necessary fixes to allow them to work. The images created from this repository can be found hosted on Docker Hub. The images are created using the automated build feature of Docker Hub and should be automatically rebuilt whenever the Jupyter project changes their base images.

* [getwarped/minimal-notebook](https://hub.docker.com/r/getwarped/minimal-notebook/)
* [getwarped/scipy-notebook](https://hub.docker.com/r/getwarped/scipy-notebook/)
* [getwarped/tensorflow-notebook](https://hub.docker.com/r/getwarped/tensorflow-notebook/)
* [getwarped/datascience-notebook](https://hub.docker.com/r/getwarped/datascience-notebook/)
* [getwarped/pyspark-notebook](https://hub.docker.com/r/getwarped/pyspark-notebook/)
* [getwarped/all-spark-notebook](https://hub.docker.com/r/getwarped/all-spark-notebook/)
* [getwarped/r-notebook](https://hub.docker.com/r/getwarped/r-notebook/)

## Using the Images in OpenShift

To deploy a specific notebook type to OpenShift, you can use the ``oc new-app`` command:

```
oc new-app getwarped/datascience-notebook --name experiments
```

You can also deploy them from the web console by selecting the _Deploy Image_ tab from _Add to project_, and then providing the full image name.

When started, the note server image will be empty. You will need to upload any notebooks and files through the Jupyter Notebook user interface. Alternatively, you can use ``oc rsync`` to copy up files to the running container. The directory any files should be copied into is ``/home/jovyan/work``.

You can manually install additional Python packages using ``pip`` from a terminal window created from the Jupyter notebook user interface, but you will need to ensure that you use the ``--user`` option to ``pip`` to install them locally to the user. This is necessary as it isn't practical to fix up permissions of the base Jupyter project images such that installation into the main Python installation will work.

Do be aware that if the container is restarted you will loose any work you have done. It is therefore recommended that you first mount a persistent volume against the deployed notebook server first. This should also be mounted at ``/home/jovyan/work``. This persistent volume will only be used for notebooks and files, you will need to re-install any Python packages after a restart. You may wish to therefore build a custom image which includes the additional Python packages, as explained below.

## Populating With Notebooks and Files

The versions of the Jupyter project images provided here have also been enabled for use as [Source-to-Image](https://github.com/openshift/source-to-image) (S2I) builders. This can be used to build in notebooks and files into an image which can then be deployed without needing to manually upload the notebooks and files. To create the image you can use the ``oc new-build`` command:

```
oc new-build getwarped/minimal-notebook~https://github.com/getwarped/powershift.git --name powershift-testing --context-dir=notebooks
```

The image can then be deployed using ``oc new-app`` by referencing the name of the image stream created.

```
oc new-app powershift-testing
```

If the source code repository the build is told to use contains a ``requirements.txt`` file for the Python ``pip`` command, any additional Python modules listed in the file will also be installed into the image.

If you do not need to retain the image and just want to deploy a notebook server using a specific set of files the one time, you can instead use ``oc new-app`` and do the build and deployment in one step.

```
oc new-app getwarped/minimal-notebook~https://github.com/getwarped/powershift.git --name powershift-testing --context-dir=notebooks
```

## Ad-hoc Instances vs Controlled Builds

The Jupyter project images come in various flavours and with the exception of the ``minimal-notebook`` are fat images that contain many commonly used Python packages. This is useful if you need an ad-hoc notebook server instances where you do not know what packages you might need, but can be a problem when you need additional packages, especially if you then need to share the work with others and know they can run it.

In the case where you need better control over exactly what packages are included, and what versions, you can use ``minimal-notebook`` as a S2I builder to build up your own custom image, with the packages to build in specified by the ``requirements.txt`` file for the Python ``pip`` command.

The ``minimal-notebook`` as provided by the Jupyter project is only available for Python 3 and not for Python 2. This could be an issue if not all packages you need are available for Python 3.

In this case, and especially where you want to set up a parallel compute cluster under OpenShift using ``ipyparallel``, you are better off using a set of alternate Jupyter notebook images provided by the ``getwarped`` project.

* [https://github.com/getwarped/jupyter-notebooks](https://github.com/getwarped/jupyter-notebooks)

These provide both Python 2 and Python 3 versions of a minimal Jupyter notebook image. These images are also S2I enabled and have additional features to customise builds and deployments by being based on the ``warpdrive`` images for Python.

That project also provides OpenShift templates which make it easier to build and deploy custom images for Jupyter notebook via the OpenShift web console, including the ability to easily start up a parallel compute cluster using ``ipyparallel``.