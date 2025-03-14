# Inconsistent Build Dependency Behavior in Solution File

## Issue Description

When using build dependencies in the slnx/sln format (slnx was released on March 13, 2025), there is an inconsistent and unexpected behavior in how outputs from dependent projects are handled. Specifically, executable dependencies have partial outputs copied to the main project's output directory, while library dependencies do not.

## Reproduction Steps

This repository contains a minimal repro with three projects:
- `SlnxSampleApp` (main executable)
- `BuildDependencyLibrary` (library project)
- `BuildDependencyExecutable` (executable project)

There are no project references between these projects, only build dependencies defined in the SLNX format.

1. Clone this repository
2. Run the following command:
   ```
   dotnet build -t:Rebuild
   ```
3. Check the output directory of `SlnxSampleApp`

## Actual Behavior

After building, the output directory of `SlnxSampleApp` contains:
- `SlnxSampleApp` output files (expected)
- **Some** `BuildDependencyExecutable` output files, specifically:
  - BuildDependencyExecutable.exe
  - BuildDependencyExecutable.deps.json
  - BuildDependencyExecutable.runtimeconfig.json
  
However:
1. The crucial `BuildDependencyExecutable.dll` is missing, rendering the executable non-functional
2. No output files from `BuildDependencyLibrary` are copied at all

## Expected Behavior

Since these are build dependencies (not project references), no output files from either dependent project should be copied to the main project's output directory.

## Additional Information

- Build dependencies in SLNX format are defined via `<BuildDependency/>` elements
- Both the legacy SLN format and the new SLNX format demonstrate identical behavior with this issue
- The difference between `BuildDependencyLibrary` and `BuildDependencyExecutable` is that the latter has `OutputType` set to `Exe`
- .NET 9.0 is used for all projects

This issue creates confusion as:
1. Build dependencies are only meant to establish build order, not create implicit dependencies
2. Even if copying was intended, the partial copying of executable outputs (missing DLLs) makes the copied executables non-functional

## System Information

- Visual Studio Version: Visual Studio Enterprise 2022 17.13.3
- .NET SDK Version: .NET 9.0.201
- OS: Windows 11 (26100.3476)
