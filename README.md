# Volumetric-RHRC
Volumetric Ring Holographic Radiance Cascades

Ring HRC is the origin of how Holographic Radiance Cascades began. however it has no formal spec or implementation. The following is a well-defined implementation of Ring-HRC that uses ring-based planar holographic radiance transfer combined with the merging scheme of [Vanilla HRC](https://github.com/Yaazarai/Volumetric-HRC) for tightly coupled merging.

## Ring Holography
The core idea is that holograms represent a plane of radiance contributions over several different perspectives. Rings and planes are synonymous in the sense that they're conceptually the same object from different viewing perspectives. A ring is a plane zoomed out to an infinite distance and a plane is a ring zoomed into an infinite distance.
<img width="999" height="921" alt="sketch" src="https://github.com/user-attachments/assets/3bd3b6f2-8fc4-49f4-821d-559bcacc86b2" />

