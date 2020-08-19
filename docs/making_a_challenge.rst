##################
Making a challenge
##################

.. note:: As you may see below, the OpenChallSpec is quite feature rich. Chances are that the CTF platform or deployment backend you may be using doesn't support all features, so be sure to research your platform or deployment backend to see what you can use.

***************
Getting Started
***************

A challenge is a folder containing a ``challenge.yml`` (or ``challenge.yaml`` if you prefer) configuration file. The ``challenge.yml`` YAML file is the heart of the challenge, defining all metadata like title, description and categories, as well as things like which files players get to download or what services they get access to. A minimal challenge configuration file looks like this:

::

    title: Example challenge
    description: This is an example minimal challenge to showcase the OpenChallSpec.
    authors: mateuszdrwal
    categories: misc

    flag_format: example{(.*)}
    flags: this_is_the_flag

    downloadable_files:
      - challenge.txt

    spec: <<spec_version>>

The first part of the example defines the metadata we are all used to: the title, description, author and category.

Then, the flag format and flag are defined. ``flag_format`` is a regex with exactly one capture group signifying the variable part of the flag. The variable part is defined right below in ``flags``, so the resulting flag of this challenge is ``example{this_is_the_flag}``.

Later, a ``challenge.txt`` is specified as a file that should be given to players through the ``downloadable_files`` option. The ``challenge.txt`` file has to exist in the same directory as the ``challenge.yml`` file.

Lastly, it is specified that the configuration follows the <<spec_version>> version of OCS. This will usually be the latest version when developing challenges.

********************
Using a build script
********************

Often, CTF challenges are built in some way by automated scripts. Sometimes this is compiling a binary, other times a script might be encoding data in an image for a steganography challenge. The OCS includes a standard way of specifying which file to run and how:

::

    build_script: build.sh

At this point, using a command line tool to assist with challenge creation can prove useful. The officially recommended tool is `challtools <TODO>`_, however you may use whichever tool you want. This tutorial will continue giving some examples using challtools.

To run the build script with challtools, run ``challtools build``. Challtools will then extract a valid flag from the challenge configuration and pass it as a command line argument to the build script when it's run. The build script should then embed the flag in the challenge, whether that's writing a `flag.txt` file or any other method.

Using a command line tool with a build script is the recommended way of building challenges as it guarantees the flag in the challenge is the same as the flag specified in the configuration. Of course, there is nothing forcing you to use this functionality. If you want, it is perfectly fine to not use a build script.

****************
Adding a service
****************

Many CTF challenges have a service, which is usually a website or a TCP server. In the OCS, these services are provided by `Docker <https://www.docker.com/>`_ containers. In the configuration, it looks thusly:

.. _docker-config:

::

    deployment:
      type: docker
      containers:
        web:
          image: ./container
          services:
            - type: website
              internal_port: 8080
              external_port: 80

First, ``type: docker`` is defined. The OCS supports adding more deployment types than just docker, but at the moment docker is the only option.

The ``containers`` list is a list of all containers for this challenge. This example (and most challenges) only specifies one container, here called ``web``.

The ``web`` container is built according to the ``image`` option. In this case, a directory ``container`` is specified, in which the container will be built when running ``challtools build``. The ``container`` directory is assumed to have a ``Dockerfile``. The ``image`` option can also have other values, like a docker image tag or a path to an image archive.

The ``services`` list defines which services this container exposes. In this case, the container exposes a web app which runs internally on port 8080, to the external port 80. By default, ``type`` can be ``website`` or ``tcp``, but :ref:`custom types can also be defined <custom_service_types_label>`. The ``external_port`` option is optional, and if omitted, a port will be assigned automatically.

You can use challtools to start the services according to the configuration by running ``challtools start``.

*********************
Adding a solve script
*********************

A solve script can be useful locally during development to check that the challenge is working properly, but it can also be used to check if a service is online by periodically running it against the service during the competition. To add a solve script to your challenge, add one of the below into your configuration:

::

    solution:
      type: docker
      image: ./solution

::

    solution:
      type: script
      script: solve.sh

The difference between these is that one is a docker container, and one is a raw script. Usually, CTF infrastructures will support running containers against live services, but not the raw scripts as they are harder to handle and may require unavailable dependencies. It is therefore recommended that you create a container solve script if the challenge has a service that you want to monitor solvability of during the competition, and using a raw script is perfectly acceptable if the challenge does not have a service. Again, this is just recommendation, and you are free to do whatever.

The ``docker`` configuration type takes an ``image`` option the same way as regular containers :ref:`above <docker-config>`. The ``script`` type takes a ``script`` option that is a solution script file.

Solve scripts will be run with the service as a command line argument. For example, for a TCP service a string like ``203.0.113.43:1337`` will be passed. For a website service, it will be the URL. Therefore, if you are using a container, make sure it runs the script using ``ENTRYPOINT`` in `exec form <https://docs.docker.com/engine/reference/builder/#entrypoint>`_ instead of ``CMD`` in the ``Dockerfile`` so that command line arguments get passed correctly.

********************
Other common options
********************

Below is a list of other commonly used configuration options. Some of these were also explained above, but have extra functionality that was not explained above.

authors
=======

The ``authors`` option can be a string for simplicity, but it can also array for when there are multiple authors.

::

    authors:
      - mateuszdrwal
      - loovjo

categories
==========

Similarly to authors_, the ``categories`` option can be a string for simplicity or an array if the challenge has multiple categories. The first category in the list will be the "main" category.

::

    categories:
      - web
      - forensics

tags
====

Tags are similar to categories, but can also include things that spoil the challenge. They are not shown to players, and are usually used for organizers own reference, but are also synonymous with tags on ctftime, so challenges can be easily added there with the right tags after a CTF. The ``tags`` option can be a single string, or a list of strings.

::

    tags: SQL injection

::

    tags:
      - SQL injection
      - local file inclusion

hints
=====

Challenge hints can be configured using the ``hints`` option. Below is an example with two hints, one free and one that costs 100 points.

::

    hints:
      - content: git gud # the hint cost defaults to 0
      - cost: 100
        content: this hint costs points

score
=====

If you are using static scoring, specify the challenge score here. A value of ``null`` usually means dynamic scoring. Defaults to ``null``, so if you are using dynamic scoring you don't have to specify this option.

::

    score: 500

predefined_services
===================

If you are deploying challenges manually or have some external unchanging service, you will want to define services using ``predefined_services``. These will show to users exactly the same as services defined in ``deployments``, but they are not managed automatically. Usually, services are either of type ``website``, in which case you need to specify ``url``, or of type ``tcp``, in which case you need to specify ``host`` and ``port``. If needed, :ref:`custom types can also be defined <custom_service_types_label>`.

::

    predefined_services:
      - type: website
        url: "https://example2.com"
      - type: tcp
        host: 203.0.113.43
        port: 1337

unlocked_by
===========

If a challenge should only be visible/available after a certain other challenge is solved, put the title of that challenge in ``unlocked_by``. This option also has more advanced features and several related options, explained in the :ref:`Advanced configuration options <unlocked_by_label>` section.