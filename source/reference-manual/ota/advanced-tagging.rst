.. _ref-advanced-tagging:

Advanced Tagging
================

Some users incorporate non-trivial workflows that can require advanced tagging
techniques. These workflows can be handled in the :ref:`ref-factory-definition`.

Terminology
-----------

**Platform Build** - A build created by a change to the LmP (lmp-manifest.git
or meta-subscriber-overrides.git). This is the base OS image.

**Container Build** - A build created by a change to containers.git.

**Target** - This an entry in a factory's TUF targets.json file. It represents
what should be thought of as an immutable combination of the Platform build's
OSTree hash + the output of a Container build.

**Tag** - A Target has a "custom" section where with a list of Tags. The
tags can be used to say things like "this is a development build"
or this is a "production" build.

Scenario 1: A new platform build that re-uses containers
--------------------------------------------------------

A Factory is set up with the normal ``main`` branch::

  lmp:
    tagging:
      refs/heads/main:
        - tag: main
  containers:
    tagging:
      refs/heads/main:
        - tag: main

You'd like to introduce a new ``stable`` branch from the LmP but have it use
the latest containers from master. This can be done with::

  lmp:
    tagging:
      refs/heads/main:
        - tag: main
      refs/heads/stable:
        - tag: stable
          inherit: main
  containers:
    tagging:
      refs/heads/main:
        - tag: main
        - tag: stable

Consider this pseudo targets example::

  targets:
    build-1:
      ostree-hash: DEADBEEF
      compose-apps: foo:v1, bar:v1
      tags: stable
    build-2:
      ostree-hash: GOODBEEF
      compose-apps: foo:v2, bar:v2
      tags: main

If a change to the stable branch was pushed to the LmP, a new
target, build-3, would be added. The build logic would then look through
the targets list to find the most recent ``main`` target so that
it can copy those compose-apps. This would result in a new target::

  build-3:
    ostree-hash: NEWHASH
    compose-apps: foo:v2, bar:v2
    tags: stable

On the other hand, there might also be a new container build for ``main``.
In this case the build logic will produce two new targets::

  build-4:  # for stable it will be based on build-3
    ostree-hash: NEWHASH
    compose-apps: foo:v3, bar:v3
    tags: stable

  build-4:  # for master, it will be based on build-2
    ostree-hash: GOODBEEF
    compose-apps: foo:v3, bar:v3
    tags: main

Scenario 2: Multiple container builds using the same platform
-------------------------------------------------------------

This scenario is the reverse of the previous one. A factory might have a
platform build tagged with ``main``. However, there are two versions of
containers being worked on: ``main`` and ``foo``. This could be handled
with::

  lmp:
    tagging:
      refs/heads/main:
        - tag: main
        - tag: foo
  containers:
    tagging:
      refs/heads/main:
        - tag: main
      refs/heads/foo:
        - tag: foo
          inherit: main

Scenario 3: Multiple teams, different cadences
----------------------------------------------

Some organizations may have separate core platform and application teams. In
this scenario, it may be desirable to let each team move at their own decoupled
paces. Furthermore, the application team might have stages(branches) of
development they are working on. This could be handled with something like::

  lmp:
    tagging:
      refs/heads/main:
        - tag: main
  containers:
    tagging:
      refs/heads/main:
        - tag: main
      refs/heads/dev:
        - tag: dev
          inherit: main
      refs/heads/qa:
        - tag: qa
          inherit: main

This scenario is going to produce ``main`` tagged builds that have no
containers, but can be generically verified. Then each containers.git branch
will build Targets and grab the latest ``main`` tag to base its platform
on. **NOTE:** Changes to ``main`` don't cause new container builds. In
order to get a container's branch updated to the latest ``main`` a user
would need to push an empty commit to containers.git to trigger a new build::

 # from branch qa
 git commit --allow-empty -m 'Pull in latest platform changes from main'
