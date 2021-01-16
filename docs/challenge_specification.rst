#######################
Challenge specification
#######################

This document contains the full detailed technical specification for creating a challenge compliant to the OpenChallSpec version <<spec_version>>. For tips on how to create a reader, see :doc:`writing_a_reader`.

***********************
Wording and terminology
***********************

This document uses key words defined in :rfc:`2119`.

The term "reader" in this document refers to any automated script that reads the challenge configuration file. It is used as an umbrella term for command line tools, CTF platforms, deployment backends or anything else that might interact with the OCS in its raw form.

**********
Versioning
**********

The OCS follows `semantic versioning <https://semver.org/>`_. Readers SHOULD support all versions of the OCS below the version they are written for but still at the same MAJOR version. Semantic versioning ensures that older OCS versions can be parsed without issue by readers designed for new versions, so in practice backwards compatibility should be automatic. Readers SHOULD refuse to parse versions of the OCS with a MINOR version higher than what they were designed for, at the same MAJOR version. Similarly, readers MAY also refuse to work with higher PATCH versions. This is because the Schema_ is not written in a forwards compatible manner, and new features will usually constitute schema violations when parsed with older OCS versions. 

*******************
Challenge structure
*******************

A challenge is a directory containing a ``challenge.yml`` or ``challenge.yaml`` configuration file. Readers MUST support both ``.yml`` and ``.yaml`` file extensions. The configuration file MUST be written in YAML. Anything outside the challenge directory is not considered part of the challenge. All challenge files MUST be located inside the challenge directory, as this can otherwise cause packageability issues.

******
Schema
******

The OCS includes a `JSON Schema <https://json-schema.org/>`_ for validation of the challenge configuration file structure, which can be found `here <https://github.com/CTF-Organizers/OpenChallSpec/blob/dev/challenge.schema.json>`_. Yes, the OCS uses JSON schema to validate a YAML file, but it works the same way as both JSON and YAML files parse to the same "dictionary" representation, and JSON Schema validators in any programming language usually take this dictionary representation as input. Readers MAY validate the configuration against the schema before attempting to interpret the configuration. A configuration file that doesn't comply with the schema is considered invalid. The schema includes default values for all non-required fields, which may be a convenient method of filling in defaults.

***************************
Implementation completeness
***************************

Readers are not required to implement functionality for all fields and sub-fields below. Functionality for any field MAY be implemented. If the challenge configuration contains fields the reader does not support the user SHOULD be notified of this, unless the incompatibility is obvious, like when a standalone deployment script that only deploys challenge containers does not do anything with the challenge description. The script MAY continue running ignoring these fields. No CTF platform (probably) supports all features in the OCS, so ignoring certain fields is a very typical thing to do.

********************
Configuration fields
********************

Below is a list of all valid fields in the challenge configuration. Unless otherwise specified, only the described keys are valid for any object. This means that any additional unexpected keys make the configuration invalid.

title
=====

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - String

The challenge title.

description
===========

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - String

The challenge description.

authors
=======

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - String, Array of strings

The challenge's authors. |strarr| The first item in the array MAY be considered the main author.

categories
==========

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - String, Array of strings

The challenge's categories. |strarr| The first item in the array MAY be considered the main category. If the reader does not support multiple categories, the first one MUST be used. There MUST be at least one category.

tags
====

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - String, Array of strings
    * - Default
      - []

The challenge's tags. |strarr| The first item in the array MAY be considered the main tag.

Tags are "private categories". They MUST NOT be shown to players. They record high level concepts that spoil the challenge, like "SQL injection".

hints
=====

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Array of objects
    * - Default
      - []

An array of the challenge's hints. If supported, these MUST be shown to players. Some hints cost points, in which case the action of opening a hint should subtract the price from the opening teams point total.

The objects are composed of these two sub-fields:

content
-------
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - String

  The hint text that is shown when opened.

cost
----
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - Number
      * - Default
        - ``0``

  The hint price.


flag_format_prefix
==================

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - String, null

The first part of the flag format that the challenge's flag(s) start(s) with. May also be ``null`` instead of a string signifying no flag format present for the challenge. In that case, the values of both flag_format_prefix_ and flag_format_suffix_ MUST be ignored for flag validation.

To validate a player submitted flag, a validator SHOULD first check if the flag starts with the flag_format_prefix_ and ends with the flag_format_suffix_. If so, the prefix and suffix is stripped from the flag and rest should be matched against the list of flags_. If it didn't, the flag's flag format is invalid.

flag_format_suffix
==================

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - String
    * - Default
      - ``}``

The last part of the flag format that the challenge's flag(s) start(s) with. Defaults to ``}`` for convenience. For more info, see flag_format_prefix_.

flags
=====

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - String, Array or objects

An array of the challenge's flags. If the type is string, this is simplified syntax. It SHOULD be interpreted as this array instead: ``[{"flag": "<initial string here>"}]``

Every element in the array is a separate flag, meaning that for a flag submission to be valid it must match at least one of the listed flags. If a reader doesn't support multiple flags, the first flag MUST be used.

The objects in the array are composed of these two sub-fields:

flag
----
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - String

  The flag contents, without the flag format as that is defined separately. 

type
----
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - String
      * - Default
        - ``text``

  MUST be either ``text`` or ``regex``.
  
  If the type is ``text``, the ``flag`` field is to be compared directly to the contents of the user submitted flag. If they are the same, the submission is considered correct.

  If the type is ``regex``, the ``flag`` field is considered to be a regex and the user submitted flag is to be matched against the regex. If it matches, the submission is considered correct. When writing challenges, the ``flag`` SHOULD start with ``^`` and end with ``$``, to prevent false positives for very short flags.

max_attempts
============

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Integer, null
    * - Default
      - null

A positive integer signifying how many times teams may attempt to submit a flag before they are stopped from submitting any more for the challenge. If ``null``, the teams have an unlimited number of tries.

Use of this option is heavily discouraged, as it often leads to a bad player experience. If you want to prevent brute-force attacks, try rate limiting instead.

score
=====

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Number, null
    * - Default
      - null

An integer signifying how many points a team receives in reward for solving the challenge, for static scoring. For dynamic scoring, set to ``null``. The dynamic scoring formula is handled by the ctf platform.

downloadable_files
==================

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - String, Array of strings
    * - Default
      - []

An array of files downloadable by players. |strarr| The string MUST be one of three things:

1. A relative path to a file in the challenge directory. Readers MUST check if the file exists, and if not, move on to the other two options.
2. A relative path to a directory in the challenge directory. All files in the directory should be included with the challenge. Readers MUST check if the directory exists, and if not, assume the string is the last option.
3. A URL to a file. When players attempt to download this file, they MUST be redirected to this URL. Therefore, it does not have to be a direct download and can be for example a google drive link.

custom_service_types
====================

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Array of objects
    * - Default
      - []

A list of custom service types. A service type is a concept that defines how services should be automatically shown to players. It is used in definitions of predefined_services_, service_ and deployment_. There are two built-in service types. they look like this:

::

    - type: website
      user_display: "{url}"
      hyperlink: true
    - type: tcp
      user_display: "nc {host} {port}"
      hyperlink: false

These built-ins MUST be treated as if they are always the first two items in the array. For example, if the custom_service_types array contains only a newly defined type ``foo``, the reader MUST treat the list of defined types as containing ``website``, ``tcp`` and ``foo``. Duplicate types MUST NOT be allowed. Therefore, ``website`` and ``tcp`` cannot be redefined.

Each object in the custom_service_types array has the following 3 fields:

type
----
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - String

  The name of the type that is being defined. Can be any string that is not an already defined type.

user_display
------------
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - String

  Defines how services with this type will be shown to players. Variables can be substituted in by typing the variable name immediately enclosed in curly brackets. Additional whitespace between the name and bracket MUST NOT be supported. For example, for the ``tcp`` service type, if the hostname of a service is ``192.0.2.69`` and the port is ``1337``, the ctf platform will show the string ``nc 192.0.2.69 1337`` in the challenge details.

  The variables in the substitution context depend on the environment. If the service is in predefined_services_, all needed variables MUST be provided in that same object. Otherwise, if the service is automatically deployed with a service_ or deployment_ configuration, it is the job of the deployment script to provide a context with the required variables. Deployment scripts MUST attempt to deploy services of a custom type they don't know of and format user_display by providing the ``host``, ``port`` and ``url`` variables in the substitution context.

hyperlink
---------
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - Boolean
      * - Default
        - false

  If the resulting user_display_ string after substitution is reachable by a web browser. If this is ``true``, ctf platforms MAY encase the string in an ``<a>`` tag.

predefined_services
===================

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Array of objects
    * - Default
      - []

A list of services for the challenge that are not automatically deployed. For example. if you will be manually deploying a service for a challenge, the hostname and port/URL to the challenge should be entered here.

Each predefined service object consists of the following one mandatory field, and any number of additional fields:

type
----
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - string

  The service type for this service. MUST be either ``website``, ``tcp``, or one defined in custom_service_types_. See custom_service_types_ for info on what a service type is.

All other fields are formatting context for formatting user_display_. Therefore, if the service type is ``website``, a ``url`` field must be passed. If the object is instead ``tcp``, a ``hostname`` and ``ip`` field must be passed.

service
=======

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Object, null
    * - Default
      - null

This field is a simplified syntax of the deployment_ field. It consists of 3 mandatory fields ``image``, ``type`` and ``internal_port``, and one optional field ``external_port``. When this field is present, assume that the deployment_ field has the following contents where ``<field name>`` is replaced by the contents of this service_ field:

::

    type: docker
    containers:
      default:
        image: <image>
        services:
          - type: <type>
            internal_port: <internal_port>
            external_port: <external_port>
    networks: {}
    volumes: {}

If this field is present, the deployment_ field MUST NOT be present.

deployment
==========

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Object, null
    * - Default
      - null

Defines in detail all services that are used by the challenge. At the top level, the object consists of the following fields:

type
----
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - string

  Currently, only the ``docker`` type is supported, so this MUST be the value. In the future more backends may be supported, like LXC or some jails.

networks
--------
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - Object
      * - Default
        - {}

  Defines networks between containers, for multiple containers. These behave the same way as regular docker networks. A container will be able to reach another container by its container name if they have a network in common.

  Each key in the networks_ object is a network name. Its value is an array of strings of container names in this network. For example, the following will put the ``foo`` and ``bar`` containers on the same network:

  ::

      networks:
        test-network:
          - foo
          - bar

volumes
-------
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - Object
      * - Default
        - {}

  Defines persistent volumes for one or multiple containers. These behave the same way as regular docker volumes. A volume can be mounted into a container at a mountpoint, and the data in it will persist between container recreations. If the volume is mounted in two containers at the same time, it behaves like a shared folder.

  Each key in the volumes_ object is a volume name. Its value is an array of objects representing a mountpoint. Each mountpoint object has exactly one key, being the container name, and its value is where to mount the volume inside the container. For example, the following will mount the same volume at ``/shared_volume`` in both the ``foo`` and ``bar`` containers:

  ::

      volumes:
        test-volume:
          - foo: /shared_volume
          - bar: /shared_volume

containers
----------
  .. list-table::
      :stub-columns: 1

      * - Required
        - true
      * - Type
        - Object

  This is the last field of the deployment_ object. Defines all docker containers for this challenge. Each key in the containers_ object is a container name. Its value is a container object. These objects contain the following fields:

.. _deploy-image:

image
^^^^^
    .. list-table::
        :stub-columns: 1

        * - Required
          - true
        * - Type
          - String

    Defines the docker image for this container. This can be defined in one of three ways:

    1. A path to a directory containing a ``Dockerfile``. In this case, the image will be built from said dockerfile. Readers MUST check if the directory exists, and if not, move on to the other two options.
    2. A path to a file containing an exported docker image. This file is usually obtained using the ``docker save`` command and results in a tarball. In this case, the exported image will be imported and used. Readers MUST check if the file exists, and if not, assume the string is the last option.
    3. A docker image tag. This can be a from an image locally on the system, publically available on dockerhub, from a private container repository etc. In this case, the image will be pulled if required and used.

services
^^^^^^^^
    .. list-table::
        :stub-columns: 1

        * - Required
          - false
        * - Type
          - Array of objects
        * - Default
          - []

    Defines the services exposed by this challenge. Each service is an object in the array. The Object has the following three fields:

type
""""
      .. list-table::
          :stub-columns: 1

          * - Required
            - true
          * - Type
            - String

      The service type for this service. MUST be either ``website``, ``tcp``, or one defined in custom_service_types_. See custom_service_types_ for info on what a service type is.

internal_port
"""""""""""""
      .. list-table::
          :stub-columns: 1

          * - Required
            - true
          * - Type
            - Integer

      The port inside the container that is exposed. This is the port your service binds to when running in the container.

external_port
"""""""""""""
      .. list-table::
          :stub-columns: 1

          * - Required
            - false
          * - Type
            - String
          * - Default
            - See below

      The port on the host machine that the service is exposed on. If ommited, The deployment script will pick some available port. This SHOULD NOT be set unless the service requires being exposed on a specific port because this can cause issues with port collisions if the service is run on a host that also runs multiple other services.

extra_exposed_ports
^^^^^^^^^^^^^^^^^^^
    .. list-table::
        :stub-columns: 1

        * - Required
          - false
        * - Type
          - Array of objects
        * - Default
          - []

    Defines other ports that need to be exposed from within the container. These can be thought of as "hidden services_". They are formatted the same way as services_, however they do not have a ``type`` as they will never be shown to users or solve scripts, and ``external_port`` is mandatory because of this.

Here is an example of a fully utilized deployment configuration:

::

    deployment:
      type: docker
      containers:
        web:
          image: ./container
          services:
            - type: website
              internal_port: 80
              external_port: 80
          extra_exposed_ports:
            - internal_port: 1337
              external_port: 1337
        db:
          image: local_db_image:latest
      networks:
        network:
          - web
          - db
      volumes:
        volume:
          - web: /shared_volume
          - db: /shared_volume

While it is supported, it is highly RECOMMENDED that challenges are created without volumes_, networks_, or multiple containers_ and services_, as these features are not expected to be widely supported and are only required in very few situations. The service_ field SHOULD be used instead unless absolutely necessary.

If this field is present, the service_ field MUST NOT be present.

solution_image
==============

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - String, null
    * - Default
      - null

A solution script that can be run to validate the challenge is functioning and solvable. This is meant mostly to test challenges with services, and could be run periodically during a CTF to validate that a challenge has not gone offline or broke in other ways. The solution is housed in a docker container so it can be run anywhere. 

The string defines the docker image for this solution. This can be defined in the same ways as :ref:`the image in a service container<deploy-image>`.

The solution container usually needs to know on which host and port a service runs on. This information is passed as a string in a command line argument when running a docker container. The string MUST be formatted by separating the host and port of the service with a colon, like this: ``192.0.2.69:1337``

If a challenge has multiple services, they MUST be passed as separate command line arguments in the following order:
1. All predefined_services_, in the order they are defined
2. For all containers_ in the order they are defined: all services_, in the order they are defined

When creating the container, be sure to use `ENTRYPOINT in exec form <https://docs.docker.com/engine/reference/builder/#entrypoint>`_ as otherwise the command line arguments will not be passed to the entrypoint in the container. Using ``CMD`` instead will not work.

The solution container MUST be run with an environment variable ``FLAG``, containing the first ``text``-type flags_ entry enclosed in the flag format (a valid flag). If no such entry exists, the environment variable MUST be set to an empty string.

If the challenge is functioning as expected, the solution container MUST output nothing more than a valid flag and optionally a trailing newline. Scripts that run this solution container SHOULD strip the resulting flag from whitespace on both ends before validating, in order to prevent rouge whitespace from invalidating the flag. Any output that is not a valid flag should be treated as if the service is malfunctioning.

unlocked_by
===========

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - String, Array of strings
    * - Default
      - []

If a challenge should only be accessible to players after a certain other challenge is solved, this should be defined here. |strarr| Each entry in the array can be either the exact case sensitive challenge title of another challenge, or a different challenges challenge_id_. Referencing a challenge by its challenge_id_ has the added benefit of the link not breaking if the challenge is renamed.

Specifying multiple requirement challenges is NOT RECOMMENDED, as support in CTF platforms is uncommon. If you do specify multiple challenges, see all_unlocked_by_required_ for exact behaviour.

all_unlocked_by_required
========================

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Boolean
    * - Default
      - false

If unlocked_by_ contains multiple challenges, defines if one or all need to be solved for this challenge to unlock. If ``true``, all challenges in the array MUST be solved for this challenge to be accessible. If ``false``, any one of the challenges in the array MUST be solved for this challenge to be accessible.

release_delay
=============

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Number
    * - Default
      - 0

The amount of seconds after the CTF start when the challenge should be automatically released. If ``0``, the challenge is released when the CTF starts.

human_metadata
==============

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Object
    * - Default
      - {}

Contains metadata that is designed to be read by humans, and not parsed by scripts. This can be useful for some data that you want to display in user interfaces. 

This field is composed of the following two sub-fields:

challenge_version
-----------------
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - string, null
      * - Default
        - null

Defines the version of the challenge. SHOULD be shown on user interfaces for deployment backends so admins can easily see which version of the challenge is deployed, if they specified a version. The format of the string is undefined and can be decided by the challenge author.

event_name
----------
  .. list-table::
      :stub-columns: 1

      * - Required
        - false
      * - Type
        - string, null
      * - Default
        - null

Defines the name of the event the challenge is for, for example ``exampleCTF 2020``. For archival purposes.

challenge_id
============

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - string, null
    * - Default
      - null

A unique identifier for this challenge. MUST be unique not only among the pool of challenges for the CTF this challenge belongs to, but among all challenges. It is therefore RECOMMENDED that this is set to a generated UUID.

This id can be used in unlocked_by_ instead of the challenge title. The advantage of this is that the link will not break if the challenge is renamed. This can also be used by readers to recognize if it is reading a challenge it already knows about, even if the title has changed.

custom
======

.. list-table::
    :stub-columns: 1

    * - Required
      - false
    * - Type
      - Object
    * - Default
      - {}

An object with an undefined structure. Any custom data that is not supported by the OCS can be put here. This is useful if you have tooling that provides functionality not supported by the OCS itself, as you will be able to specify configuration values in this object in any format you like. For example, if you have implemented a feature in your CTF platform that plays an audio file when a player solves a specific challenge, you could specify which audio file to play in a custom ``solve_audio`` configuration field in this object.

spec
====

.. list-table::
    :stub-columns: 1

    * - Required
      - true
    * - Type
      - string

The version of the OCS the challenge is written in. The current version is ``<<spec_version>>``, so this MUST be the value if the challenge follows the version described in this document.

.. |strarr| replace:: If the type is string, this is simplified syntax. It SHOULD be interpreted as a single element array of this one string.