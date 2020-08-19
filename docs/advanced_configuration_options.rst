##############################
Advanced configuration options
##############################

Here are all other options not explained in :doc:`making_a_challenge`. Some options from :doc:`making_a_challenge` are also expanded upon here, in order to explain their full functionality.

*****
flags
*****

The ``flags`` option supports specifying multiple flags, and also regex flags. To do this, use the following syntax:

::

    flag_format: example{(.*)}
    flags:
    - flag: here_is_a_text_flag
      type: text
    - flag: here_is_another_text_flag # The type defaults to text, so it doesn't have to be specified
    - flag: ^here\s+is\s+some\s+flag\s+with\s+arbitrary\s+whitespace$
      type: regex

All of these flags will be valid when verifying against this challenge:

- ``example{here_is_a_text_flag}``
- ``example{here_is_another_text_flag}``
- ``example{here is some flag with arbitrary whitespace}``
- ``example{here    is  some flag      with  arbitrary whitespace}``

When making a regex flag, be sure to wrap the regex in ``^`` and ``$``, which matches the beginning and end of the flag respectively. Otherwise, a regex flag ``foo`` will match any flag containing the word ``foo``, including the flag ``example{you_probably_didnt_foo_intend_this_to_match}``.

************
max_attempts
************

If a team should only have a certain number of attempts at submitting the correct flag before they are blocked, specify this number in the ``max_attempts`` option. In most cases, this is not something you want to do, as this leads to teams creating alt accounts and testing flags there to not be locked out of their main one. If you are looking at a way to stop people from brute forcing flags, rate limiting is usually a better idea. This option should really only be used in closed CTFs where only admins can create team accounts which they distribute to teams. The default is ``null``, which signifies no limit.

::

    max_attempts: 10

******************
downloadable_files
******************

Like most other options, ``downloadable_files`` can be either a single string or a list of strings. In addition to specifying a path to a file, a directory or a URL can also be specified. In the case of a directory, all files within it will be included. For the URL, it will be forwarded to the user raw, meaning it does not have to lead to a direct download.

::

    downloadable_files: container/app.py

::

    downloadable_files:
      - container/app.py
      - downloads/ # all files in the downloads directory will be downloadable by players
      - "https://example.com/some_file" # players will be redirected to this link to download the file

.. _custom_service_types_label:

********************
custom_service_types
********************

By default, ``website`` and ``tcp`` service types are defined, which should be sufficient for the vast majority of cases. Sometimes it maybe useful to specify a custom type, like the type ``htjp`` for `this HTJP challenge <https://github.com/wat3vr/watevrCTF-2019/tree/master/challenges/web/HTJP>`_. Doing so would look like this:

::

    custom_service_types:
      - type: htjp
        user_display: "htjp://{host}:{port}"
        solve_script_display: "{host}:{port}"
        hyperlink: false # defaults to false

For the sake of example, lets say predefined_services looks like this:

::

    predefined_services:
      - type: htjp
        host: 203.0.113.43
        port: 1337

The ``user_display`` option specifies how this service is displayed to players. In this case, players would receive the string ``htjp://203.0.113.43:1337`` as the service. If the challenge uses ``deployment`` to deploy a ``htjp`` type service, the formatting options provided will be ``host`` containing the ip/DNS resolvable hostname, ``port`` containing the port, and ``url`` containing the host and port formatted as a proper http URL. You can use other formatting options, but in that case you must either write a custom deployment backend that provides those options or use predefined_services, where you can define arbitrary formatting options.

The ``solve_script_display`` option is similar to ``user_display``, but defines how the solve script is run against the service. in this case, the solve script would be run with the service command line argument like this: ``./solve_script 203.0.113.43:1337``.

The ``hyperlink`` option defines if a CTF platform should make the service clickable and openable in a browser.

The default ``website`` and ``tcp`` types cannot be overwritten. They are defined thusly:

::

    custom_service_types:
      - type: website
        user_display: "{url}"
        solve_script_display: "{url}"
        hyperlink: true
      - type: tcp
        user_display: "nc {host} {port}"
        solve_script_display: "{host}:{port}"
        hyperlink: false

Keep in mind that because they cannot be overwritten, the above snippet is not valid according to the OCS if it was inserted into a configuration file.

**********
deployment
**********

The ``deployment`` option has many buttons to push and knobs to twist. Support for these varies depending on what you use to deploy the challenge. A full configuration, using all features, looks like this:

::

    deployment:
      type: docker
      containers:
        web:
          image: container
          services:
            - type: website
              internal_port: 8080
              external_port: 80
          
          extra_exposed_ports:
            - internal_port: 1337
              external_port: 1337
        db:
          image: postgres:latest
      networks:
        test-network:
          - web
          - db
      volumes:
        test-volume:
          - web: /shared_volume
          - db: /shared_volume

Similarly to as explained in :doc:`making_a_challenge`, this deployment defines a container called ``web``, built from the ``container`` directory, which exposes a website service on external port 80.

Additionally, the ``web`` container exposes the port 1337 through ``extra_exposed_ports``. Ports exposed through this option are "secret" ports; they are not given to players. Here, as opposed to in ``services``, ``external_port`` is required.

After the ``web`` container is defined, another ``db`` container is defined. This challenge now consists of two separate docker containers. The ``db`` container is pulled from dockerhub as its image is specified as ``postgres:latest``. This container could expose its own services, but it doesn't.

Now that all containers are defined, ``networks`` are defined. One network named "test-network" is defined to which both ``web`` and ``db`` containers will be connected when they are deployed. This network behaves the same way as a `Docker bridge network <https://docs.docker.com/network/bridge/>`_. For example, the ``web`` container can use the hostname ``db`` to connect to the database and vice versa.

Lastly, shared volumes are defined. A volume called ``test-volume`` is created, which is mounted at ``/shared_volume`` in both ``web`` and ``db`` containers.

.. _unlocked_by_label:

***********
unlocked_by
***********

Similarly to other options, ``unlocked_by`` can be either a string or a list of strings. For behaviour relating to when a challenge should be unlocked if it has multiple requirements, see all_unlocked_by_required_. The recommended way to specify which challenge is a requirement for this challenge is by setting the string to its challenge_id_. The exact title of the challenge can also be specified, however this can cause errors if the required challenge is renamed.

::

    unlocked_by:
      - Example requirement challenge
      - a62c6318-9306-48c8-95ea-6a374461ac91

************************
all_unlocked_by_required
************************

This option is a boolean. If ``true``, all challenges in the `unlocked_by`_ list must be solved in order for this challenge to be accessible. If ``false``, only one challenge from the list needs to be solved. Defaults to ``false``.

::

    all_unlocked_by_required: true

*************
release_delay
*************

If the challenge should be automatically released/published after a certain time since the CTF started, specify the number of seconds in ``release_delay``. Defaults to 0.

::

    release_delay: 3600 # one hour

**************
human_metadata
**************

``human_metadata`` unsurprisingly contains metadata intended to be read and processed by humans. Filling this in isn't in any way required, but it's nice to have for people that might look at your challenge source in the future.

::

    human_metadata:
      challenge_version: "0.0.1"
      event_name: "exampleCTF 2020"

``challenge_version`` can be used to keep track of the challenge version for yourself. Some deployment backends or CTF platforms may show this to help with knowing what version of the challenge is currently deployed. Obviously, this is only useful if you yourself keep updating it.

``event_name`` is the name of the CTF this challenge is for. This is useful for archival purposes.

************
challenge_id
************

to uniquely identify a challenge in unlocked_by_ and perhaps across your infrastructure you can set ``challenge_id`` to a unique string. It is recommended to generate a UUID at creation time and use it, as it effectively guarantees a unique id for every challenge in existence. This is not a requirement though, and the id can be any string. It should however be something unique, even beyond the scope of your CTF. Defaults to ``null``.

::

    challenge_id: "3ce287f8-9c61-44c4-9113-79eb9a4d7d71"

******
custom
******

If you are writing your own infrastructure and have an obscure requirement the OCS doesn't support, you might find the ``custom`` option useful. ``custom`` is an object that you can format however you want, and there are no constraints on what you can put in it. If you for example want to play a different video when a team solves a challenge depending on which challenge they solve, a ``solve_video_url`` option would not be a good fit to include with the OCS as it's very obscure, but it can easily be configured by including it in ``custom``.

``custom`` may also be a good choice if there is a feature you are waiting for to be added into the OCS, but hasn't arrived yet and you need to use it now.
