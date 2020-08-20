.. OpenChallSpec documentation master file, created by
   sphinx-quickstart on Tue Aug 18 20:13:46 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

OpenChallSpec - The Open Challenge Specification
================================================

The OpenChallSpec (OCS) is a specification for jeopardy style Capture The Flag competition challenges. It defines a ``challenge.yml`` file which is used to describe the metadata and deployment of any CTF challenge. Usually, organizers either tediously enter metadata into their CTF platform and deploy challenges manually, or create their own challenge format that automates some or all of this work, but only covers their specific use case. The OCS aims to solve this by providing a challenge format that covers all use cases and works seamlessly with any CTF platform or tool.

The OCS has a few distinct design goals:

- Cover everyone's use cases. If a CTF platform or deployment tool has support for a certain feature, you should be able to configure this feature inside ``challenge.yml`` (within reason).
- Intercompatibility with every CTF platform and tool. A challenge written using the OCS for a CTF running CTFd should also work with any other CTF platform without issues.
- Challenge packageability. A folder containing a challenge should be able to be zipped, committed to a git repository, etc. and sent to another person while retaining all functionality, even if the challenge author and receiving person don't use the same tools or have the same environment.

If you have a use case the OCS doesn't cover, or a feature in your CTF platform not configurable from the OCS, don't hesitate to `submit an issue <TODO>`_ so that it can be added to the spec.

.. toctree::
   :maxdepth: 2
   :hidden:

   making_a_challenge
   advanced_configuration_options
   challenge_examples
   challenge_specification