IN PROGRESS

# Volumetric Ring Holographic Radiance Cascades (R-HRC)

Ring HRC is the origin of how [Holographic Radiance Cascades](https://github.com/entropylost/amitabha) began. However it has no formal paper from Adrian Margel the original inventor of the idea sited in the plane-based HRC paper. The following is a well-defined implementation of Ring-HRC that uses ring-based planar holographic radiance transfer combined with the merging scheme of [Vanilla HRC](https://github.com/Yaazarai/Volumetric-HRC) for tightly coupled merging.

## Ring Holography
The core idea is that holograms represent a [plane of radiance contributions over several different perspectives](https://www.youtube.com/watch?v=EmKQsSDlaa4). Rings and planes are synonymous in the sense that they're conceptually the same object from different viewing perspectives. A plane is a ring zoomed into an infinite distance and vice versa.
<img width="999" height="921" alt="sketch" src="https://github.com/user-attachments/assets/3bd3b6f2-8fc4-49f4-821d-559bcacc86b2" />

As such we get the concept of ring-based holography. Imagine that we divide the scene into a set of probes like in Vanilla RC where we halve the spatial resolution along each axis. Instead of storing a single probe we store a set of probes along a ring. Where the ring represents the point of our original RC probe and the size of each ring is defined as the `sqrt(2) * interval` allowing the rings to overlap maximally within the perimeter of their nearest CN+1 probe.
<img width="1916" height="716" alt="image" src="https://github.com/user-attachments/assets/271e477e-b455-40d8-bab5-cfb716172a04" />

The scaling factor of the number of probes/rays per ring is entirely arbitrary, but in the exercise of providing a well defined specification we start with 4 rays in C0, 4 rays + 4 probes in C1, 8 rays + 8 probes in C2 and so on. This means each cascade scales at a constant size:

<table align="center">
  <thead>
    <tr>
      <th align="center">Level</th>
      <th align="center">W-Rings</th>
      <th align="center">H-Rings</th>
      <th align="center">Rays</th>
      <th align="center">Probes</th>
      <th align="center">Total Rays</th>
    </tr>
  </thead>
  <tbody>
    <tr><td align="center">0</td><td align="center">1024</td><td align="center">1024</td><td align="center">4</td><td align="center">1</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">1</td><td align="center">512</td><td align="center">512</td><td align="center">4</td><td align="center">4</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">2</td><td align="center">256</td><td align="center">256</td><td align="center">8</td><td align="center">8</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">3</td><td align="center">128</td><td align="center">128</td><td align="center">16</td><td align="center">16</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">4</td><td align="center">64</td><td align="center">64</td><td align="center">32</td><td align="center">32</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">5</td><td align="center">32</td><td align="center">32</td><td align="center">64</td><td align="center">64</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">6</td><td align="center">16</td><td align="center">16</td><td align="center">128</td><td align="center">128</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">7</td><td align="center">8</td><td align="center">8</td><td align="center">256</td><td align="center">256</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">8</td><td align="center">4</td><td align="center">4</td><td align="center">512</td><td align="center">512</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">9</td><td align="center">2</td><td align="center">2</td><td align="center">1024</td><td align="center">1024</td><td align="center">4,194,304</td></tr>
    <tr><td align="center">10</td><td align="center">1</td><td align="center">1</td><td align="center">2048</td><td align="center">2048</td><td align="center">4,194,304</td></tr>
  </tbody>
</table>

In discrete space the number of points along a circle can be represented as the number of points on a box perimeter (4R) of the same radius. In this case we're using 1/2 that distribution where each ring as roughly 1 probe every 2 pixels. This allows for a bit of spatial interpolation when probes/rays don't line up perfectly, but not so much that we break occlusion.

## Hemispherical Rays/Cones
Rays are cast outward away from the ring distributed in a hemisphere merging with the nearest CN+1 ring.
<img width="1368" height="379" alt="image" src="https://github.com/user-attachments/assets/049f5a63-f742-426f-a1b8-3b3a04a603a0" />
You can imagine that when merging from the highest cascade (CN) to the lowest cascade (C0) the C0 probe will always contain a ray that lies approaches the tangent of the ring resulting in a hemispherical distribution in both directions to capture every perspective.
<img width="1316" height="1272" alt="image" src="https://github.com/user-attachments/assets/f1526734-823c-4d21-a904-26c5c61daee6" />

## Cone Merging
In Vanilla HRC merging rays is converted into merging cones by using extra one extra say per probe so that we can shared a parallel ray with an adjacent cone. This way we get perfect ray occupancy throughout the scene. Like so:
<img width="1637" height="1595" alt="image" src="https://github.com/user-attachments/assets/533ec318-c3be-4854-bd11-f644819331a1" />

Merging via rings works fundamentally the same way--again due to the parallelism between ring holography and planar holography. Each ray texel allocation in memory represents a cone and we cast a left ray and a right ray. Optionally we can pre-cast all of our rays and cache them in a texture to cut the total ray usage in half because each cone shares a ray. The benefit of rings however is that we're not locked to frustums, so we can deal with the full angular radiance field and avoid discontinuities between frustums.
