.. _ref-linux-kernel:

Linux Kernel
============

A common and unified Linux® Kernel source tree is provided and used by
the Linux microPlatform. The latest continuous release is available
at `github.com/foundriesio/linux`_.

The Linux Kernel recipe can be found in the :ref:`Meta-LMP layer
<ref-linux-layers-meta-lmp>`, under the
``meta-lmp-base/recipes-kernel/linux`` directory.

.. _ref-linux-fragments:

Linux microPlatform Kernel Configuration Fragments
--------------------------------------------------

Together with the unified Linux Kernel tree, the Linux microPlatform also
provides an additional repository for the kernel configuration fragments.
The latest continuous release for the kernel configuration fragments is
available at `github.com/foundriesio/lmp-kernel-cache`_.

You can find the list of supported BSP definitions and configuration fragments
used under the ``lmp-kernel-cache/bsp`` directory.

The fragments repository works similarly to the upstream ``yocto-kernel-cache``
repository, so the same development workflow and documentation applies.
See the `Yocto Project Linux Kernel Development Manual`_ for more information
on how to work and manage the kernel metadata and configuration fragments.

The Porting Guide includes the section :ref:`ref-pg-how-to-configure-linux` on
on how to add a custom Linux Kernel configuration which can be used to add:

* the complete machine configuration.

* fragments: a set of ``CONFIG_`` variables working to change
  a default machine configuration.

.. _github.com/foundriesio/linux: https://github.com/foundriesio/linux
.. _github.com/foundriesio/lmp-kernel-cache: https://github.com/foundriesio/lmp-kernel-cache
.. _Yocto Project Linux Kernel Development Manual: https://docs.yoctoproject.org/4.0.6/kernel-dev/advanced.html

Linux microPlatform with Real-Time Linux Kernel
-----------------------------------------------

The recipe ``meta-lmp/meta-lmp-base/recipes-kernel/linux/linux-lmp-rt_git.bb``
or ``meta-lmp/meta-lmp-base/recipes-kernel/linux/linux-lmp-fslc-imx-rt_git.bb``
can be used for real-time Linux.
This is based on the ``linux-lmp`` recipe,
but extended to include the ``PREEMPT_RT`` patch-set
(updated along with stable kernel updates).

The instructions to change the default Linux kernel to real-time are
described in the following sections.
After the changes,
build the Linux microPlatform image as usual.

Building Linux microPlatform with linux-lmp-rt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set  ``PREFERRED_PROVIDER_virtual/kernel``
to ``linux-lmp-rt``
in ``meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc``::

    $ cat meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc
    PREFERRED_PROVIDER_virtual/kernel:intel-corei7-64 = "linux-lmp-rt"

Building Linux microPlatform with linux-lmp-fslc-imx-rt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set ``PREFERRED_PROVIDER_virtual/kernel``
to ``linux-lmp-fslc-imx-rt``
in ``meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc``::

    $ cat meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc
    PREFERRED_PROVIDER_virtual/kernel:mx6ull-nxp-bsp = "linux-lmp-fslc-imx-rt"

Linux microPlatform with the Real-Time Xenomai4 Core
----------------------------------------------------

The recipe
``meta-lmp/meta-lmp-base/recipes-kernel/linux/linux-lmp-fslc-imx-xeno4_git.bb``
can be used to enable the Xenomai4 co-kernel on iMX boards.

	Like its predecessors in the Xenomai core series, `Xenomai4`_ with the
	`EVL core`_ brings real-time capabilities to Linux by embedding a
	companion core into the kernel, which specifically deals with tasks
	requiring ultra low and bounded response time to events.

	In this model, the general purpose kernel and the real-time core operate
	almost asynchronously, both serving their own set of tasks, always
	giving the latter precedence over the former.

.. _Xenomai4: https://evlproject.org/overview/
.. _EVL core: https://evlproject.org/core/

Building Linux microPlatform with linux-lmp-fslc-imx-xeno4
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set ``PREFERRED_PROVIDER_virtual/kernel`` to
``linux-lmp-fslc-imx-xeno4`` and ``MACHINE_FEATURES:append`` to ``xeno4``
in ``meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc``::

    $ cat meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc
    PREFERRED_PROVIDER_virtual/kernel:mx8mm-nxp-bsp = "linux-lmp-fslc-imx-xeno4"
    MACHINE_FEATURES:append = " xeno4"


Linux microPlatform with Linux upstream
---------------------------------------

The recipe ``meta-lmp/meta-lmp-base/recipes-kernel/linux/linux-lmp-dev.bb``
can be used to build the Linux microPlatform with the upstream kernel tree
instead of the LmP unified tree. ``linux-lmp-dev`` also uses the Linux
microPlatform Kernel Configuration Fragments repository for a compatible
configuration.

Building Linux microPlatform with linux-lmp-dev
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set the ``PREFERRED_PROVIDER_virtual/kernel`` to ``linux-lmp-dev`` in
``meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc``::

    $ cat meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc
    PREFERRED_PROVIDER_virtual/kernel = "linux-lmp-dev"

Now just build any of the supported Linux microPlatform images.

Specifying Linux Git Tree, Branch and Commit Revision
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following variables can be also set in
``meta-subscriber-overrides/conf/machine/include/lmp-factory-custom.inc``
in order to build ``linux-lmp-dev`` using a specific Linux tree, branch or
commit revision::

    KERNEL_REPO = "git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git" # Kernel git repository
    KERNEL_BRANCH = "master" # Git kernel branch (default: master)
    KERNEL_COMMIT = "94710cac0e" # Kernel commit revision (default: HEAD)
    KERNEL_META_REPO = "git://github.com/foundriesio/lmp-kernel-cache.git" # Kernel configuration fragments repository
    KERNEL_META_BRANCH = "master" # Git kernel meta branch (default: master)
    KERNEL_META_COMMIT = "1c67180cfe" # Kernel meta commit revision (default: HEAD)
    LINUX_VERSION = "4.19-rc" # Linux kernel base version (base package version)
