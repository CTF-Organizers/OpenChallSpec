##################
Making a challenge
##################

.. note:: As you may see below, the OpenChallSpec is quite feature rich. Chances are that the CTF platform or deployment backend you may be using doesn't support all features, so be sure to research your platform or deployment backend to see what you can use.

***************
Getting Started
***************

A challenge is a folder containing a ``challenge.yml`` (or ``challenge.yaml`` if you prefer) configuration file. The ``challenge.yml`` YAML file is the heart of the challenge, defining all metadata like title, description, and categories, as well as things like which files players get to download or what services they get access to. A minimal challenge configuration file looks like this:

::

    title: Example challenge
    description: This is an example minimal challenge to showcase the OpenChallSpec.
    authors: mateuszdrwal
    categories: misc

    flag_format_prefix: example{
    flags: this_is_the_flag

    downloadable_files:
      - challenge.txt

    spec: <<spec_version>>

The first part of the example defines the metadata we are all used to: the title, description, author and category.

Then, the flag format and flag are defined. ``flag_format_prefix`` specifies the first part of the flag format. The variable part of the flag is defined right below in ``flags``, so the resulting flag for this challenge is ``example{this_is_the_flag}``.

Later, a ``challenge.txt`` is specified as a file that should be given to players through the ``downloadable_files`` option. The ``challenge.txt`` file has to exist in the same directory as the ``challenge.yml`` file.

Lastly, it is specified that the configuration follows the <<spec_version>> version of OCS. This will usually be the latest version when developing challenges.

****************
Adding a service
****************

Many CTF challenges have a service, which is usually a website or a TCP server. In the OCS, these services are provided by `Docker <https://www.docker.com/>`_ containers. In the configuration, it looks thusly:

.. _docker-config:

::

    service:
      image: ./container
      type: website
      internal_port: 8080
      external_port: 80

The docker image is built according to the ``image`` option. In this case, a directory ``container`` is specified, from which the image will be built. The ``container`` directory is assumed to have a ``Dockerfile``. The ``image`` option can also have other values, like a docker image tag or a path to an image archive.

``type`` can by default be one of ``website`` or ``tcp``, and represents what type of service it is that is exposed. ``internal_port`` then defines at which port this service listens inside the container, and the optional ``external_port`` defines what port should be exposed on the host machine.

At this point, using a command line tool to assist with challenge creation can prove useful. The officially recommended tool is `challtools <TODO>`_, however you may use whichever you like, or none at all. Challtools can build the docker image for you as defined in the config simply by running ``challtools build``. You can also start the service locally by running ``challtools start``.

*********************
Adding a solve script
*********************

.. note:: This is a useful but uncommonly supported feature on competition infrastructure. It is usually perfectly fine to not have a solve script.

A solve script can be useful locally during development to check that the challenge is working properly, but it can also be used to check if a service is online by periodically running it against the service during the competition. To add a solve script to your challenge, add the following into your configuration:

::

    solution_image: ./solution

This option behaves similarly to the ``image`` option :ref:`above <docker-config>`. In this case, a docker image will be built from the ``solution`` directory. To test if a challenge is solvable, this image will be ran with the challenge location as command line arguments. For example, a command line argument can be a string like ``203.0.113.43:1337``. Therefore, if you are using a container, make sure it runs the script using ``ENTRYPOINT`` in `exec form <https://docs.docker.com/engine/reference/builder/#entrypoint>`_ instead of ``CMD`` in the ``Dockerfile`` so that command line arguments get passed correctly. The container should output just the flag on STDOUT if the challenge was solved successfully.

If this seems complicated, check out the practical TODO example.

********************
Other common options
********************

Below is a list of other commonly used configuration options. Some of these were also explained above, but have extra functionality that was not explained above.

authors
=======

The ``authors`` option can be a string for simplicity, but it can also be an array for when there are multiple authors.

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

flag_format_prefix and flag_format_suffix
=========================================

``flag_format_prefix`` and ``flag_format_suffix`` together define the flag format for the challenge. ``flag_format_suffix`` defaults to ``}``, so it should rarely be needed (unless you are using a non-standard flag format, to which I say please don't). ``flag_format_prefix`` does not have a default so it needs to be specified in every challenge, for example ``exampleCTF{``. If the challenge does not include a flag format, flag_format_prefix should be set to ``null`` in which case both options will be ignored.

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

If you are deploying challenges manually or have some external unchanging service, you will want to define services using ``predefined_services``. These will show to users exactly the same as the service defined in ``service``, but they are not managed automatically. Usually, services are either of type ``website``, in which case you need to specify ``url``, or of type ``tcp``, in which case you need to specify ``host`` and ``port``. If needed, :ref:`custom types can also be defined <custom_service_types_label>`.

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