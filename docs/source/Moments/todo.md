# ToDo for Rotation Invariance Pattern Detection

- Cinema Levia paper examples for other use case examples.
- Trivial test example workflow available in ParaView GitLab

```
- compile vtk
- enable testing in cmake
- enable moments invariant module in cmake
- run tests for moments invariant module using built-in data: ctest moments
- point out python scripts that can be used as examples for further use

clone vtk
mk build
ccmake ./..
enable VTK_MODULE_ENABLE_VTK_MomentInvariants
enable VTK_BUILD_TESTING (might be enabled by the current default though)
make
ctest -R patternDetectionTestSimple

```
