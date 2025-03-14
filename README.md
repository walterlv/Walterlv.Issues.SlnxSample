# SLNX Format Build Dependency Bug Report

## Issue Description

There is a critical inconsistency in how build dependencies are handled between the new SLNX format (released on March 13, 2025) and the traditional SLN format. When a project has build dependencies defined, the new SLNX format incorrectly copies executable dependency outputs to the main project's output directory.

## Reproduction Steps

This repository contains a minimal repro with three projects:
- `SlnxSampleApp` (main executable)
- `BuildDependencyLibrary` (library project)
- `BuildDependencyExecutable` (executable project)

There are no project references between these projects, only build dependencies defined in both SLN and SLNX formats.

1. Clone this repository
2. Run the following commands:
   ```
   dotnet build .\Walterlv.Issues.SlnxSample.sln -t:Rebuild
   ```
   Check the output directory of `SlnxSampleApp`

3. Then run:
   ```
   dotnet build .\Walterlv.Issues.SlnxSample.slnx -t:Rebuild
   ```
   Check the output directory of `SlnxSampleApp` again

## Actual Behavior

When building with the SLNX format, the output directory of `SlnxSampleApp` contains:
- `SlnxSampleApp` output files
- `BuildDependencyExecutable` output files (incorrectly copied)

Note that `BuildDependencyLibrary` outputs are not copied, suggesting the issue specifically affects executable build dependencies.

When building with the SLN format, the output directory of `SlnxSampleApp` contains only:
- `SlnxSampleApp` output files (expected behavior)

## Expected Behavior

The SLNX format should behave the same as the SLN format when handling build dependencies - build dependencies should only establish build order, not copy output files between projects.

## Additional Information

- Build dependency in SLN format is defined via `ProjectDependencies` section
- Build dependency in SLNX format is defined via `<BuildDependency/>` elements
- The only difference between `BuildDependencyLibrary` and `BuildDependencyExecutable` is that the latter has `OutputType` set to `Exe`
- .NET 9.0 is used for all projects

## System Information

- Visual Studio Version: Visual Studio Enterprise 2022 17.13.3
- .NET SDK Version: .NET 9.0
- OS: Windows 11 (26100.3476)
