# OpenSees MPCO Tcl Patch for Windows

A minimal patch set that enables MPCO recording in Tcl-based OpenSees on Windows and fixes HDF5 1.12.3 runtime compatibility issues.

## Included Files

This repository contains the following modified files:

- `SRC/recorder/TclRecorderCommands.cpp`
- `SRC/actor/objectBroker/FEM_ObjectBrokerAllClasses.cpp`
- `SRC/recorder/CMakeLists.txt`
- `SRC/recorder/MPCORecorder.cpp`

## What This Patch Fixes

This patch addresses the following issues:

- `recorder mpco` not recognized in Tcl-based OpenSees
- HDF5 runtime version mismatch
- SWMR startup failure caused by incorrect `H5F_LIBVER_LATEST` handling

## Required Runtime Environment

This patch is intended for use with:

- Tcl-based OpenSees on Windows
- HDF5 1.12.3 runtime DLLs

Recommended HDF5 location:

- `C:\Program Files\HDF_Group\HDF5\1.12.3\bin`

Important DLLs:

- `hdf5.dll`
- `hdf5_hl.dll`

Make sure OpenSees loads the HDF5 1.12.3 DLLs instead of other versions such as those from Anaconda.

## Usage

Example recorder command:

```tcl
file mkdir D:/OPSmpco

recorder mpco "D:/OPSmpco/res.mpco" \
    -N "displacement" "rotation" \
    -E "force" "deformation" "section.force" "section.deformation" "section.fiber.stress" "section.fiber.strain"
