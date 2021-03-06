.. index:: Matrix elements
.. index:: psi4
.. index:: pyscf

``CheMPS2::Hamiltonian``
========================

There are several ways to feed matrix elements to CheMPS2. They are explained below in detail. Because CheMPS2 is a spin-adapted code, it can only handle spatial orbitals. It is hence not possible to use different orbitals for :math:`\uparrow` and :math:`\downarrow` electrons.

#. The ``CheMPS2::Hamiltonian`` object is able to read in FCIDUMP files.

#. The ``savehdf`` plugin to `psi4 <http://www.psicode.org/>`_ fills a ``CheMPS2::Hamiltonian`` object, and saves the matrix elements to disk in HDF5 format, taking into account their eightfold permutation symmetry.

#. Users can also generate matrix elements themselves, and feed them to the ``CheMPS2::Hamiltonian`` object.


psi4 ``fcidump`` plugin
-----------------------

The ``CheMPS2::Hamiltonian`` object is able to read in FCIDUMP files:

.. code-block:: c++

    CheMPS2::Hamiltonian::Hamiltonian( const string filename, const int psi4groupnumber )
    
The variable ``filename`` should contain the path to the FCIDUMP file, and the variable ``psi4groupnumber`` should be the group number of the abelian point group with real-valued character table as defined in `psi4 <http://www.psicode.org/>`_. The conversion table is provided here:

 +--------------+----+----+----+----+----+-----+-----+-----+
 | Group name   | c1 | ci | c2 | cs | d2 | c2v | c2h | d2h |
 +--------------+----+----+----+----+----+-----+-----+-----+
 | Group number | 0  | 1  | 2  | 3  | 4  | 5   | 6   | 7   |
 +--------------+----+----+----+----+----+-----+-----+-----+

FCIDUMP files can for example be generated by a plugin to `psi4 <http://www.psicode.org/>`_. The plugin has been tested on commit ``14c78eabdca86f8e094576890518d93d300d2500`` (February 27, 2015) on https://github.com/psi4/psi4public, and should work on later versions as well.

To make use of this feature, build `psi4 <http://www.psicode.org/>`_ with the plugin option, and then run:

.. code-block:: bash

    $ cd /mypsi4plugins
    $ psi4 --new-plugin fcidump
    $ cd fcidump

Now, replace the file ``fcidump.cc`` with ``/myfolder/chemps2/integrals/psi4plugins/fcidump.cc``. To compile the plugin, run:

.. code-block:: bash

    $ make

An example input file to generate a FCIDUMP file:

.. literalinclude:: N2.fcidump.in

This file (``N2.fcidump.in``) should be placed in the folder ``/mypsi4plugins``. The FCIDUMP file can then be generated with:

.. code-block:: bash

    $ cd /mypsi4plugins
    $ psi4 N2.fcidump.in N2.fcidump.out

Examples of output generated with this plugin can be found in ``/myfolder/chemps2/tests/matrixelements``.


psi4 ``savehdf`` plugin
-----------------------

The ``CheMPS2::Hamiltonian`` object is able to read in HDF5 files generated by a plugin to `psi4 <http://www.psicode.org/>`_. The plugin has been tested on commit ``14c78eabdca86f8e094576890518d93d300d2500`` (February 27, 2015) on https://github.com/psi4/psi4public, and should work on later versions as well. It stores all unique matrix elements (remember that there is eightfold permutation symmetry) in binary form with HDF5.

To make use of this feature, build `psi4 <http://www.psicode.org/>`_ with the plugin option, and then run:

.. code-block:: bash

    $ cd /mypsi4plugins
    $ psi4 --new-plugin savehdf
    $ cd savehdf

Now, replace the file ``savehdf.cc`` with ``/myfolder/chemps2/integrals/psi4plugins/savehdf.cc``. To compile the plugin, the Makefile should be adjusted. Change the line

.. code-block:: bash

    PSIPLUGIN = -L$(OBJDIR)/lib -lplugin

to

.. code-block:: bash

    PSIPLUGIN = -L$(OBJDIR)/lib -lplugin -lchemps2

Remember to add the library and include paths to the Makefile as well, if ``libchemps2`` is not installed in a standard location.
To compile the plugin, run:

.. code-block:: bash

    $ make

An example input file to use the ``savehdf`` plugin:

.. literalinclude:: N2.savehdf.in

This file (``N2.savehdf.in``) should be placed in the folder ``/mypsi4plugins``. The matrix elements can then be saved to disk in binary form with HDF5 by running:

.. code-block:: bash

    $ cd /mypsi4plugins
    $ psi4 N2.savehdf.in N2.savehdf.out

The plugin generates three files:

.. code-block:: bash

    /mypsi4plugins/CheMPS2_Ham_parent.h5
    /mypsi4plugins/CheMPS2_Ham_Tmat.h5
    /mypsi4plugins/CheMPS2_Ham_Vmat.h5

which allow to create an instance of the ``CheMPS2::Hamiltonian`` object:

.. code-block:: c++

    CheMPS2::Hamiltonian::Hamiltonian( const bool fileh5, const string main_file, const string file_tmat, const string file_vmat )

The variable ``fileh5`` should be ``true``, and the three strings should contain the paths to the three files listed above, in the same respective order.

.. _chemps2_psi4irrepconventions:

Custom matrix elements
----------------------

It is also possible to create a ``CheMPS2::Hamiltonian`` object with your own matrix elements. Please consult the ``CheMPS2::Hamiltonian`` class in the `doxygen html output <http://sebwouters.github.io/CheMPS2/doxygen/index.html>`_, and more specifically the functions:

.. code-block:: c++

    CheMPS2::Hamiltonian::Hamiltonian( const int Norbitals, const int nGroup, const int * OrbIrreps )
    void CheMPS2::Hamiltonian::setEconst( const double val )
    void CheMPS2::Hamiltonian::setTmat( const int index1, const int index2, const double val )
    void CheMPS2::Hamiltonian::setVmat( const int index1, const int index2, const int index3, const int index4, const double val )

The variable ``Norbitals`` should be the total number of orbitals in the active space. The variable ``nGroup`` should be the group number of the abelian point group with real-valued character table as defined in `psi4 <http://www.psicode.org/>`_. The array ``OrbIrreps`` of length ``Norbitals`` contains for each orbital its irreducible representation number as defined in `psi4 <http://www.psicode.org/>`_. The conversion table is provided here:

 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | Group / Irreps | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   |
 +================+=====+=====+=====+=====+=====+=====+=====+=====+
 | 0: c1          | A   |     |     |     |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 1: ci          | Ag  | Au  |     |     |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 2: c2          | A   | B   |     |     |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 3: cs          | A'  | A'' |     |     |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 4: d2          | A   | B1  | B2  | B3  |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 5: c2v         | A1  | A2  | B1  | B2  |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 6: c2h         | Ag  | Bg  | Au  | Bu  |     |     |     |     |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+
 | 7: d2h         | Ag  | B1g | B2g | B3g | Au  | B1u | B2u | B3u |
 +----------------+-----+-----+-----+-----+-----+-----+-----+-----+

Two important remarks:

#. Every matrix element which can be nonzero according to the point group symmetry should be assigned (also if its value is 0.0).
#. Physics notation is used for the two-electron integrals in CheMPS2: :math:`V_{ij;kl} = ( ik \mid jl )` or ``CheMPS2::Hamiltonian::setVmat( i, j, k, l, (ik|jl) )``.


