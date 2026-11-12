.. index:: fix pimd/uvt

fix pimd/uvt command
====================

Syntax
""""""

.. code-block:: LAMMPS

   fix ID group-ID pimd/uvt method nmpimd thermostat NHC temp T Tdamp tau_T mu Mu Udamp tau_U ne Ne0 dedn source [keyword value ...]

* ID, group-ID are documented in :doc:`fix <fix>` command
* ``method`` value = ``nmpimd``
* ``thermostat`` value = ``NHC``
* ``temp`` value = target temperature (temperature units)
* ``Tdamp`` value = nuclear thermostat damping parameter (time units)
* ``mu`` value = constant chemical potential
* ``Udamp`` value = positive electronic damping time (time units)
* ``ne`` value = initial electronic coordinate
* ``dedn`` value = global scalar or vector reference used as dE/dN

Optional keywords include ``integrator``, ``drag``, ``fmass``,
``fmmode``, ``sp``, ``lj``, ``ne_velocity``, ``removecom``,
``tchain``, ``tloop``, and ``tdof``. The ``ne_velocity`` keyword sets the
initial electron-number velocity (electron number per unit time); its
default is zero. Other optional keywords are described on the
:doc:`fix pimd <fix_pimd>` page.

Examples
""""""""

.. code-block:: LAMMPS

   variable dEdN equal 3.0*(f_cp[11]-1.2)
   fix cp all pimd/uvt method nmpimd thermostat NHC temp 0.8 Tdamp 0.2 &
          tchain 3 tloop 1 mu 1.5 Udamp 0.2 ne 1.8 dedn v_dEdN

Launch the input file with two bead partitions using:

.. code-block:: bash

   mpirun -np 2 lmp -partition 2x1 -in in.electron_pimd_uvt

Description
"""""""""""

``fix pimd/uvt`` performs constant-potential path-integral molecular
dynamics.  It extends the :doc:`fix pimd <fix_pimd>` Nose-Hoover
hierarchy with one global electronic coordinate shared by all beads.
The number of path-integral beads is set by the number of LAMMPS
partitions, so the job must be launched with one partition per bead.

The electronic mass is :math:`W_\mathrm{e}=g_\mathrm{c} k_B T\,\mathrm{Udamp}^2`,
where :math:`g_\mathrm{c}` is the centroid mode's nuclear thermostat DOF,
including any ``removecom`` correction or ``tdof`` override. This single
mass is shared by all beads and MPI ranks, even when internal modes have
different nuclear DOF.

The ``dedn`` source may be an equal-style variable (``v_name``), a
global compute (``c_ID``), or a global fix (``f_ID``).  If the source
provides a global vector, an entry can be selected with the usual
``[index]`` syntax.  ``dEdN`` is bead averaged before updating the
global electronic coordinate.

The global fix vector starts with the 10 shared nuclear energy and estimator
outputs of :doc:`fix pimd/nve <fix_pimd>`. The next six entries match the
physical outputs of :doc:`fix uvt <fix_uvt>`, in the same order:

#. entry 11: electron number ``Ne``
#. entry 12: combined instantaneous physical temperature ``T_ins``
#. entry 13: electron-number velocity ``Ne_dot``
#. entry 14: this bead's energy derivative ``dEdN``
#. entry 15: electronic kinetic energy
#. entry 16: electronic potential contribution ``-mu*Ne``

These six indices do not depend on ``tchain``. The ``4*tchain`` NHC outputs
follow, starting at entry 17, so the total vector length is ``16+4*tchain``.
The physical outputs are not normalized by atom count. Nuclear and NHC
outputs retain their inherited normalization.

The temperature combines nuclear kinetic energy converted from the internal
scaled-temperature representation with the shared electronic kinetic energy,
counted once. For P=1, it has the same definition as the combined temperature
reported by ``fix uvt``. The electronic kinetic energy and ``-mu*Ne`` are
also reported in the physical representation.

Entry 14 reports the individual bead derivative. Average these values
over beads to compare with the target chemical potential; this average
is used for propagation. For classical ``fix uvt``, entry 4 reports the
derivative of the single classical system.

The fix scalar reports cumulative nuclear plus electronic thermostat
work in the internal scaled ring-polymer representation. The classical UVT
scalar instead reports an extended-energy contribution, as described on its
documentation page.

The fix writes restart data for the nuclear thermostat state and the
electronic degree of freedom, so simulations may be continued with
:doc:`read_restart <read_restart>` followed by the same ``fix pimd/uvt``
command.

Compatibility
"""""""""""""

Update scripts using the old chain-dependent electronic indices. With
``tchain 3``, ``Ne`` moves from entry 23 to entry 11; nuclear entries
1 through 10 are unchanged. The target chemical potential and the three
thermostat-work entries are no longer vector outputs.

Restrictions
""""""""""""

This fix requires :doc:`run_style verlet <run_style>` and does not
support :doc:`fix dt/reset <fix_dt_reset>`.

This fix currently supports only 3d systems.

This fix requires an atom map, e.g. ``atom_modify map yes``, and consecutive atom IDs.
All bead partitions must use the same number of processors.

The bead count equals the number of LAMMPS partitions.

This fix is a complete time integrator.  No other time-integration fix
should be applied to the same atoms.

Pressure control is not supported by this fix.

This fix is part of the REPLICA package.  It is only enabled if
LAMMPS was built with that package.  See the :doc:`Build package
<Build_package>` page for more info.

Related commands
""""""""""""""""

:doc:`fix pimd <fix_pimd>`, :doc:`fix uvt <fix_uvt>`,
:doc:`read_restart <read_restart>`, :doc:`run_style <run_style>`
