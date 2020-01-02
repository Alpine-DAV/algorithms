# ToDo for Rotation Invariance Pattern Detection

- Add bubbles as novel use case; gradient search for scalar field example
- Take out reconstruction part -- too much detail.
- Look at Cinema Levia paper examples for other use case examples.
- Add the trivial test example:

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
