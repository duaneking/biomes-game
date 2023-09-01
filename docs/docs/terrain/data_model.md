---
sidebar_position: 1
---

# Data Model

### Glossary

- `Shard`: A 32x32x32 [tensor](https://en.wikipedia.org/wiki/Tensor).
- `BlockId`: An integer value that corresponds to a given block type, e.g. `1: grass`, `2: dirt`. All block types are defined
  in [terrain.json](https://github.com/ill-inc/biomes-game/blob/main/src/shared/asset_defs/terrain.ts).
- `Seed`: The initial state of the terrain.
- `Voxel`: A single block.
- `Subvoxel`: A single block that's 1/8th the size of a full block. Each full block contains `8x8x8` subvoxels.

## Shards

The entire voxel 3D world is split up into shards.
The shard data is stored as a buffer, and need to be decompressed to be read.

Each shard contains the following data.

> _Note: all positions, will the exception of `Box`, are defined relative to the
> lowermost coordinate of the shard._
>
> _E.g. `[0, 0, 0]` corresponds to the lowestmost coordinate, `[31, 31, 31]` corresponds
> to the uppermost coordinate, and `[10, 16, 19]` lies somewhere between the lowest and highest most coordinates._

### Box

`v0`: lowermost coordinate of the shard.

`v1`: uppermost coordinate of the shard.

### ShardSeed

The Biomes terrain has some initial state we refer to as the `seed`. Each voxel in the world has some initial block type, which
is stored in `ShardSeed`s. `ShardSeed(x, y, z)` stores the `BlockId` of the initial block type at that location.

The seed shards are generated by scripts defined in [galois](https://github.com/ill-inc/biomes-game/tree/main/src/galois/py/notebooks).

### ShardDiff

When a voxel's block type is modified and becomes different then the seed shard, we store this diff(erence) in the diff
shards. Because most blocks will not be updated, these are sparse tensors - they store a maximum of `32x32x32` entries.
In other words, we _only_ store the updates.

Like the `ShardSeed`, these shards define a mapping for position to `BlockId`.

### ShardShape

The terrain is mostly occupied by full blocks, perfect cubes. However, all voxels can be transformed into a different shape,
for example a stair, a fence, window, table, etcetera, using shaping tools. The shape shards store information about the current shape of each voxel.

The shape data is encoded as an integer and contains two things:

1. `ShapeId`: e.g. stair. Shapes are defined in [shapes.json](https://github.com/ill-inc/biomes-game/blob/main/src/shared/asset_defs/gen/shapes.json).
2. `IsomorphismId`: e.g. stair flipped vertically and facing north.

### ShardPlacer

Each user has a `BiomesId` which is their unique identifier. The placer shards correspond voxels with the user,
`BiomesId`, that last modified it.

### ShardOccupancy

Placeables, like a boombox, TV, or flower, are not voxels however they still take up space. The occupancy tensor
records the space occupied by non-voxel things so that other things are not placed in the space they occupy.
Each position corresponds to the `BiomesId` of the entity that occupied the position, if there is one.