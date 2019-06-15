PEP: TBD
Title: Moving tkinter, IDLE, and turtle to a bundling model
Version: $Revision$
Last-Modified: $Date$
Author: Nick Coghlan <ncoghlan@gmail.com>
Status: Draft
Type: Standards Track
Content-Type: text/x-rst
Created: 15-Jun-2019
Python-Version: 3.9
Post-History: TBD


[Initial conceptual draft. This will NOT be submitted as a PEP until after
it has been discussed with the current ``tkinter``, ``turtle``, and
``idlelib`` maintainers, and they have agreed to sign on as co-authors]


Abstract
========

This PEP proposes separating out ``tkinter``, ``idlelib``, and ``turtle`` as
regular, independently versioned PyPI packages, but continuing to bundle them
with the default Windows and macOS binary installers.

In this way, they will become readily separable optional components of the
installation in all cases, while still being available by default in policy
constrained environments where ready access to the full online Python Package
Index may not be available (e.g. in schools).


Motivation
==========

While some Python standard library modules are needed for the core Python
interpreter to be able to function at all (e.g. ``sys``, ``os``), others
are only needed for particular applications or use cases, and may not be
relevant when Python is ported to new platforms.

This has led to a number of proposals over the years to either slim down the
standard library, or else to version it independently from the reference
interpreter itself (TODO: reference the specific PEPs and articles).

In most cases, these efforts have struggled, as they've attempted to consider
the standard library as a whole, rather than focusing on one specific case,
and then attempting to address that case using techniques that could be
adapted to other specific instances of migration to a less monolithic
development process.

This PEP chooses to start with the specific cases of the ``tkinter``,
``idlelib``, and ``turtle`` modules, for the following reasons:

* ``tkinter`` introduces a dependency on the operating system's GUI windowing
  environment, so it is already regularly debundled on operating systems
  where the GUI environment is optional (e.g. Linux distro packages). This then
  further leads to IDLE and the ``turtle`` module being debundled, as these
  depend on ``tkinter``.
* new Python target environments may either have their own approach to handling
  GUIs (e.g. browsers, WASM), or no GUI support at all (e.g. microcontrollers).
  In either case, it doesn't make sense to provide Tcl/Tk bindings in these
  environments, so these components are already being omitted from Python
  interpreter targeting these environments
* as a bundled application, ``idlelib`` is already covered by a process
  exception in PEP 434 that allows it to receive new features and other
  enhancements in maintenance releases, making it more akin to a PyPI package
  than a regular standard library module
* if a Python distribution provides an alternative default Python editor
  (e.g. Mu), together with an alternative default GUI development framework
  (e.g. Qt), then it may not make sense to also provide IDLE and ``tkinter``
* however, we also routinely have end users express appreciation for the fact
  these tools are available by default in the Windows and macOS installs, so
  we don't want to simply remove them entirely, the way we're aiming to do for
  some other genuinely obsolete and outdated modules that are no longer being
  actively maintained

These characteristics make these libraries excellent candidates for conversion
to maintenance as regular Python packages published on PyPI


Proposal
========

This PEP proposes moving development of Python's Tcl/Tk bindings, the IDLE
Python editor, and the ``turtle`` module, into a new independently versioned
``tcl-tk-idle`` repository under the Python GitHub organisation. This repository
will be set up to publish three distinct projects to PyPI: ``tkinter``,
``idle``, and ``turtle``.

As we have routinely had end users express appreciation for having these
modules available as part of the default installation in various constrained
environments, the Python 3.9 Windows and macOS binary installers on python.org
will be updated to bundle the relevant wheel archives based on a
``requirements.txt`` file checked in to the CPython repository.

To prevent confusing behaviour on older version, all three projects will have
``Python-Requires: >= 3.9`` set in their package metadata.

Automated testing
-----------------

As part of the ``python`` org on GitHub, these projects would continue to have
access to the same CI services as they do today. However, rather than being
tested on every CPython commit, they would instead have their own CI matrix
that tested each of their own commits against their supported CPython branches,
other Python interpreter implementations, and the CPython development branch.

(See the open questions below regarding buildbot testing)

Versioning
----------

To clearly distinguish themselves from their standard library predecessors,
these projects would switch to calendar-based versioning.


Documentation
-------------

All three projects would migrate to primary documentation hosting on ReadTheDocs.

(However, see the open questions below)


Publishing to PyPI
------------------

The Python core-workflow maintainers will work with the PyPI administrators and
the PSF to ensure that publishing a new release of these libraries to PyPI is
as simple as tagging the git repo.


Open Questions
==============

One repository or multiple?
---------------------------

The PEP currently proposes a single tcl-tk-idle repository containing all 3
extracted packages. It may make more sense to instead have each package in its
own independently versioned repository so their release cycles and documentation
can be clearly independent.

One advantage of being the same repository is that IDLE and ``turtle`` could
more readily be used to test the compatibility of ``tkinter`` changes, but
there are ways to achieve that in the separate repository setup as well.


Handling existing documentation links
-------------------------------------

If the projects migrate to having their own documentation on ReadTheDocs, what
happens to any existing links to their pages in the main standard library
documentation?

Perhaps bundled libraries should also have a copy of their documentation hosted
as part of the main standard library documentation?


Buildbot testing?
-----------------

Pre-merge testing is only part of the testing performed on CPython commits:
there is also additional post-merge testing performed across the Buildbot fleet.
By moving out of the standard library, these components would potentially
miss out on that additional platform compatibility testing.

Perhaps the Buildbot master could be configured to monitor and test
addition repositories against the latest stable Python release across the full
set of supported platforms?


Backwards Compatibility
=======================

From a backwards compatibility perspective, there is one important distinction
between traditional standard library modules and bundled libraries that are
installed by default: the standard library is visible from virtual environments
by default, while other packages are only visible if the virtual environment
is explicitly configured to have access to the system site-packages directory.

This means that if an application is running in a virtual environment, then
it would be able to import these modules by default in Python 3.8 and earlier
(if they're installed), but would need to explicitly depend on them in Python
3.9 and later (or, alternatively, the virtual environment would need to be
reconfigured to have access to the system site-packages directly).

Either way, the change seems to be within the bounds of what would be acceptable
as an entry in the "Porting to Python 3.9" section of the What's New document.



References
==========

TODO...


Copyright
=========

This document has been placed in the public domain.



..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   coding: utf-8
   End:
