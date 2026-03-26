# MPCO Patch README

## Scope

This note documents the local changes made to enable `recorder mpco` in the Tcl-based OpenSees build and to make it work with HDF5 1.12.3 on Windows.

This patch was prepared for the Tcl build chain only.

## Problem Summary

The original behavior showed two separate issues:

1. `recorder mpco` was not registered in the Tcl executable unless `_HDF5` was defined at compile time, so OpenSees reported:

```text
commands.cpp - : mpco NOT FOUND
WARNING No recorder type exists for recorder of type:mpco
```

2. After enabling the recorder, the runtime-loaded HDF5 path still had version/configuration problems:

- The runtime HDF5 version check in `MPCORecorder.cpp` was hard-coded to `1.12.2`
- The runtime-loaded HDF5 enum value for `H5F_LIBVER_LATEST` was incorrectly defined as `1`
- That caused SWMR startup to fail with a superblock version error

## Modified Files

The following source files were modified:

1. `SRC/recorder/TclRecorderCommands.cpp`
2. `SRC/actor/objectBroker/FEM_ObjectBrokerAllClasses.cpp`
3. `SRC/recorder/CMakeLists.txt`
4. `SRC/recorder/MPCORecorder.cpp`

## What Changed

### 1. `SRC/recorder/TclRecorderCommands.cpp`

Changed the Tcl recorder command registration so that `mpco` is available even when `_HDF5` is not defined.

Reason:

- `MPCORecorder.cpp` already supports runtime loading of `hdf5.dll`
- Therefore `mpco` should not be hidden behind `_HDF5` for the Tcl build

`vtkhdf` was left under `_HDF5`.

### 2. `SRC/actor/objectBroker/FEM_ObjectBrokerAllClasses.cpp`

Moved `MPCORecorder` include/creation outside `_HDF5` so the recorder object can actually be instantiated when requested.

`VTKHDF_Recorder` was left under `_HDF5`.

### 3. `SRC/recorder/CMakeLists.txt`

Made `MPCORecorder.cpp` and `MPCORecorder.h` part of the recorder target by default.

`VTKHDF_Recorder` remains conditional on HDF5 support.

### 4. `SRC/recorder/MPCORecorder.cpp`

Two runtime-HDF5 fixes were applied:

- HDF5 version check changed from `1.12.2` to `1.12.3`
- `H5F_LIBVER_LATEST` changed from `1` to `3`

Reason:

- For HDF5 1.12.3, `H5F_LIBVER_LATEST` maps to `H5F_LIBVER_V112`
- In the official HDF5 1.12.3 headers, `H5F_LIBVER_V112 = 3`
- The previous incorrect value caused SWMR file initialization failure

## Required Runtime HDF5 Version

This patch expects runtime loading of HDF5 1.12.3.

Installed location used during testing:

```text
C:\Program Files\HDF_Group\HDF5\1.12.3
```

Important DLLs:

```text
C:\Program Files\HDF_Group\HDF5\1.12.3\bin\hdf5.dll
C:\Program Files\HDF_Group\HDF5\1.12.3\bin\hdf5_hl.dll
```

## Important Windows Note

If another `hdf5.dll` appears earlier in `PATH` than the HDF5 1.12.3 directory, OpenSees may still load the wrong DLL.

Typical example:

- Anaconda may provide `C:\Users\<user>\anaconda3\Library\bin\hdf5.dll`

To avoid loading the wrong DLL, use one of these approaches:

1. Put `C:\Program Files\HDF_Group\HDF5\1.12.3\bin` before other HDF5 locations in `PATH`
2. Copy `hdf5.dll` and `hdf5_hl.dll` into the same directory as `OpenSees.exe`

## Rebuild Requirement

After copying these source changes, rebuild the Tcl executable.

At minimum, rebuild the components affected by the patch:

1. `recorder`
2. `actor`
3. final Tcl `OpenSees.exe`

## Example Recorder Command

To place large MPCO files in `D:/OPSmpco`, use:

```tcl
file mkdir D:/OPSmpco

recorder mpco "D:/OPSmpco/res.mpco" \
    -N "displacement" "rotation" \
    -E "force" "deformation" "section.force" "section.deformation" "section.fiber.stress" "section.fiber.strain"
```

Use forward slashes in Tcl paths.

## Expected Result After Patch

After rebuilding and loading the correct HDF5 1.12.3 DLL, the previous errors should disappear:

- `mpco NOT FOUND`
- HDF5 version mismatch against `1.12.2`
- SWMR startup error caused by wrong `H5F_LIBVER_LATEST`

The recorder should then create `.mpco` output normally.
