---
title: Dynamic Terrain Examples
image:
description: Examples of how to use the dynamic terrain extension.
keywords: dynamic terrain, terrain, dynamic
further-reading:
video-overview:
video-content:
---

## FreeCamera on the Ground

In this example, our player moves on the ground : a character walking, a car driving or anything like we can find in a FPS game.  
The used camera is a FreeCamera. It will keeps on the ground while moving onward, backward, to the right or to the left.

Let's create a simple data map with a sinus/cosinus function :

```javascript
// map creation
const mapSubX = 100;
const mapSubZ = 100;
const mapData = new Float32Array(mapSubX * mapSubZ * 3);
for (let l = 0; l < mapSubZ; l++) {
  for (let w = 0; w < mapSubX; w++) {
    mapData[3 * (l * mapSubX + w)] = (w - mapSubX * 0.5) * 2.0;
    mapData[3 * (l * mapSubX + w) + 1] = (w / (l + 1)) * Math.sin(l / 2) * Math.cos(w / 2) * 2.0;
    mapData[3 * (l * mapSubX + w) + 2] = (l - mapSubZ * 0.5) * 2.0;
  }
}
```

The hills have a different height depending on their positions on the map.

Let's create then a dynamic terrain from this map :

```javascript
// terrain creation
const terrainSub = 50;
const params = {
  mapData: mapData,
  mapSubX: mapSubX,
  mapSubZ: mapSubZ,
  terrainSub: terrainSub,
};
terrain = new BABYLON.DynamicTerrain("terrain", params, scene);
```

Now we can set the camera altitude on each frame at a fixed elevation above the terrain. For this, we use the method `getHeightFromMap(x, z)` what returns the height (y coordinate) of the terrain at the passed location _(x, z)_.

```javascript
const camElevation = 2.0;
const camAltitude = 0.0;
scene.registerBeforeRender(function () {
  camAltitude = terrain.getHeightFromMap(camera.position.x, camera.position.z) + camElevation;
  camera.position.y = camAltitude;
});
```

If we move the camera with the keyboard arrow keys, it will follow the terrain ground.

PG: <Playground id="#J6FMJ#6" title="Dynamic Terrain" description="Example Follow Terrain"/>

We could also make a bigger map and a bigger terrain :

```javascript
const mapSubX = 500;
const mapSubZ = 300;
const terrainSub = 100;
```

As the player keeps on the ground, he/she can't see very far in the distance, so we can choose to update the terrain only after 8 map quads crossed over either on the X axis, either on the Z axis :

```javascript
terrain.subToleranceX = 8;
terrain.subToleranceZ = 8;
```

The terrain is then updated less often, this lets more CPU to manage the player behavior and all what can happen or move in the close area around him.

Finally, we could also choose to extend the terrain visible size by setting some LOD in the distance.  
By example, `LODLimits = [4, 3, 2, 1, 1]` would mean that :

- the initial LOD factor (default 1, the one at the camera location) is incremented for all the terrain quads under the 4-th perimetric rows and columns, so it's 1 + 1 = 2 for them,
- it's incremented once again for all the terrain quads under the 3-th perimetric rows and columns, so it's 2 + 1 = 3 for them,
- it's incremented once again for the ones under the 2-nd rows and columns, so it's 4 for them
- it's incremented finally twice, because there are two `1` in the array, for the quads under the first rows and columns, so 3 + 2 x 1 = 5 for those last ones.

```javascript
terrain.LODLimits = [4, 3, 2, 1, 1];
```

Let's don't worry if this feature seems complex to understand and let's just remember that it extends the terrain visible size on its perimeter and reduces the rendered map details in the same time.

PG: <Playground id="#J6FMJ#7" title="Dynamic Terrain" description="Example Larger Map"/>
