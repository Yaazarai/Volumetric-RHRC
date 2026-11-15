# Volumetric-RHRC
Volumetric Ring Holographic Radiance Cascades

Ring HRC is the origin of how Holographic Radiance Cascades began. however it has no formal spec or implementation. The following is a well-defined implementation of Ring-HRC that uses ring-based planar holographic radiance transfer combined with the merging scheme of [Vanilla HRC](https://github.com/Yaazarai/Volumetric-HRC) for tightly coupled merging.

## Ring Holography
The core idea is that holograms represent a plane of radiance contributions over several different perspectives. Rings and planes are synonymous in the sense that they're conceptually the same object from different viewing perspectives. A plane is a ring zoomed into an infinite distance and vice versa.
<img width="999" height="921" alt="sketch" src="https://github.com/user-attachments/assets/3bd3b6f2-8fc4-49f4-821d-559bcacc86b2" />

As such we get the concept of ring-based holography. Imagine that we divide the scene into a set of probes like in normal RC where we halve the spatial resolution along each axis. Instead of storing a single probe we store a set of probes along a ring. Where the ring represents the point of our original RC probe.
<img width="1292" height="621" alt="image" src="https://github.com/user-attachments/assets/59902fa4-97c0-440c-92dc-06cd937c5b26" />

The scaling factor of the number of probes/rays per ring is entirely arbitrary, but in the exercise of providing a well defined specification we start with 4 rays in C0, 4 rays + 4 probes in C1, 8 rays + 8 probes in C2 and so on. This means each cascade scales at a constant size:

| Level | Width | Height | Rays | Probes | Total Rays |
| ----- | ----- | ------ | ---- | ------ | --------- |
| 0     | 1024  | 1024   | 4    | 1      | 4,194,304 |
| 1     | 512   | 512    | 4    | 4      | 4,194,304 |
| 2     | 256   | 256    | 8    | 8      | 4,194,304 |
| 3     | 128   | 128    | 16   | 16     | 4,194,304 |
| 4     | 64    | 64     | 32   | 32     | 4,194,304 |
| 5     | 32    | 32     | 64   | 64     | 4,194,304 |
| 6     | 16    | 16     | 128  | 128    | 4,194,304 |
| 7     | 8     | 8      | 256  | 256    | 4,194,304 |
| 8     | 4     | 4      | 512  | 512    | 4,194,304 |
| 9     | 2     | 2      | 1024 | 1024   | 4,194,304 |
| 10    | 1     | 1      | 2048 | 2048   | 4,194,304 |

In discrete space the number of points along a circle can be represented as the number of points on a box of the same radius. In this case we're using 1/2 that distribution where each ring as roughly 1 probe every 2 pixels. This allows for a bit of spatial interpolation when probes/rays don't line up perfectly, but not so much that we break occlusion.

## Hemispherical Rays/Cones
Rays are cast outward away from the ring distributed in a hemisphere merging with the nearest CN+1 ring.
<img width="1368" height="379" alt="image" src="https://github.com/user-attachments/assets/049f5a63-f742-426f-a1b8-3b3a04a603a0" />
You can imagine that when merging from the highest cascade (CN) to the lowest cascade (C0) the C0 probe will always contain a ray that lies approaches the tangent of the ring resulting in a hemispherical distribution in both directions to capture every perspective.
<img width="1316" height="1272" alt="image" src="https://github.com/user-attachments/assets/f1526734-823c-4d21-a904-26c5c61daee6" />
