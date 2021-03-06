==================================================
 Multiprocessing extension for Gevent_ (geventmp_)
==================================================

.. warning::
    HIC SUNT DRACONES!!!

    This code is extremely experimental (pre-pre-alpha). There is very little testing, lots of things are in flux,
    some platforms don't work at all.

    DO NOT use in production. This code may crash your server, bankrupt your company, burn your house down and be mean
    to your puppy. You've been warned.

Problem
=======

Due to internal implementation, `multiprocessing` (`MP`) is unsafe to use with Gevent_ even when `monkey-patched`__.
Namely, the use of OS semaphore primitives and inter-process IO in `MP` will cause the main
loop to stall/deadlock/block (specific issue depends on the version of CPython).

__ monkey_

Solution
========
Geventmp_ (`Gee-vent Em-Pee`, not `Gee-ven Tee-Em-Pee`) is an extension plugin for `monkey patch`__ subsystem
of Gevent_. As with the rest of the monkey patch subsystem the process is fairly clear:

__ monkey_

1. Identify all places where blocking occurs and where it may stall the loop.
2. If blocking occurs on a file descriptor (`FD`), try to convert the file descriptor from blocking to non-blocking
   (sockets/pipes/fifos, sometimes even files where, rarely, appropriate) and replace blocking IO functions with their
   gevent_ non-blocking equivalents.
3. If blocking occurs in a Python/OS primitive that does not support non-blocking access and thus cannot be geventized,
   wrap all blocking access to that primitive with native thread-pool-based wrappers and call it a day (while fully
   understanding that primitive access latency will increase and raw performance may suffer as a result).
4. If you are really brave and have lots of free time on your hands, completely replace a standard blocking Python
   non-`FD`-based primitive with implementation based on an `FD`-based OS primitive (e.g. POSIX semaphore =>
   Linux `eventfd-based semaphore for kernels > 2.6.30`__).
5. Due to launching of separate processes in `MP`, figure out how, when, and whether to `monkey patch`__ spawned/forked
   children and grandchildren.

__ eventfd_

__ monkey_

Installation
============
The package is hosted on PyPi_.

For stable version:

.. code-block:: bash

  pip install geventmp

For unstable version:

.. code-block:: bash

  pip install --pre geventmp


Once installed, `geventmp`_ will activate by default in the below stanza.

.. code-block:: python

   from gevent.monkey import patch_all
   patch_all()

If you would like `geventmp`_ to not activate by default, either do not install it or explicitly disable it:

.. code-block:: python

   from gevent.monkey import patch_all
   patch_all(geventmp=False)

That's it - there are no other flags, settings, properties or config values so far.

Supported Platforms
===================

.. note::
    All claims of support may not be real at all. You're welcome to experiment. See warnings on top.

* Linux, possibly Darwin.
* CPython 2.7, 3.5, 3.6, 3.7... maybe.

TODO
====
1. Implement CI/CD once we figure out how `Gevent Issue #1448 <https://github.com/gevent/gevent/issues/1448>`_ is going
   to go.
2. Test on CPython 2.7, 3.5, 3.6, 3.7, 3.8, PyPys etc.
3. Monkey patch Windows to the extent possible.
4. Test and fix Darwin if required.
5. Lots of applications use `Billiard <https://github.com/celery/billiard>`_ for multiprocessing instead of stock Python
   package. Consider monkey patching Billiard if detected.

Contact Us
==========

Post feedback and issues on the `Bug Tracker`_, `Gitter`_,
and `Twitter (@karelleninc)`_.

.. _Gevent: https://github.com/gevent/gevent/
.. _geventmp: https://github.com/karellen/geventmp
.. _bug tracker: https://github.com/karellen/geventmp/issues
.. _gitter: https://gitter.im/karellen/Lobby
.. _twitter (@karelleninc): https://twitter.com/karelleninc
.. _monkey: https://en.wikipedia.org/wiki/Monkey_patch
.. _eventfd: https://linux.die.net/man/2/eventfd
.. _pypi: https://pypi.org/project/geventmp/
