# CryEngine 1 CGF Importer/Exporter for Blender

A Blender addon for importing and exporting **CryEngine 1 / Far Cry (2004)** geometry and animation files.

Ported from the original [CryImporter for 3ds Max 8](https://www.takaro.net) by Takaro Pty. Ltd.

---

## Features

### Import
- `.cgf` and `.cga` geometry files
- `.caf` animation files onto armatures (auto-imports CGF if not in scene)
- `.cal` animation list files (multiple animations at once)
- Mesh with correct vertex positions, normals, and UV coordinates
- Multi-material support with correct face material assignment
- Texture auto-detection by game root path or relative path
- Skeleton / armature import from bone chunks
- Vertex weights (skinning) for character models
- Shape keys (morph targets / facial expressions)

### Export
- `.cgf` geometry files (mesh + materials + skeleton + weights)
- `.caf` animation files from Blender Actions
- `.cal` animation list (exports all Actions as CAF files)
- Correct material name format with shader: `name(ShaderName)/surface`
- Vertex colors (white by default — required by engine)
- Bone weights / physique data

### General
- Correct scale: Max inches ↔ Blender meters (`× 0.0254`)
- Supports Blender 4.0, 4.1, 4.2, 5.0+

---

## Installation

1. Download `io_import_cgf.zip` from [Releases](../../releases)
2. In Blender: **Edit → Preferences → Add-ons → Install...**
3. Select the downloaded `.zip` file
4. Enable **"CryEngine 1 CGF Importer/Exporter (Far Cry)"**

---

## Import Usage

### Geometry — CGF / CGA

**File → Import → CryEngine Geometry (.cgf, .cga)**

| Option | Description |
|---|---|
| Import UVs | Import texture coordinates |
| Import Normals | Use normals from file |
| Import Materials | Create Principled BSDF materials |
| Import Skeleton | Build armature from bone chunks |
| Import Vertex Weights | Assign bone weights |
| Game Root Path | Root folder of Far Cry (e.g. `C:\FarCry`) — used to find textures by relative path |

**Required file layout:**
```
FarCry\
├── Objects\
│   └── Characters\
│       └── model.cgf    ← select this
└── Textures\
    └── texture.dds      ← found automatically via Game Root Path
```

---

### Animation — CAF

**File → Import → CryEngine Animation (.caf)**

The addon **automatically finds and imports the CGF** from the same folder. No manual CGF import needed.

**Required file layout:**
```
any_folder\
├── model.cgf    ← found and imported automatically
└── model.caf    ← select this
```

---

### Animation List — CAL

**File → Import → CryEngine Animation List (.cal)**

Reads the CAL file, auto-imports the CGF, then imports all listed CAF files as separate Actions.

**Required file layout:**
```
any_folder\
├── model.cgf    ← found automatically
├── model.cal    ← select this
├── idle.caf
├── walk.caf
└── run.caf
```

After import, switch between animations in the **Action Editor**.

---

## Export Usage

### Geometry — CGF

**File → Export → CryEngine Geometry (.cgf)**

| Option | Description |
|---|---|
| Selected Only | Export only selected mesh objects |
| Export Materials | Write material chunks |
| Export Skeleton | Write bone chunks from visible armature |
| Export Vertex Weights | Write physique (bone weights) |

**Important for round-trip (import → modify → export):**
1. Import CGF with the new addon version — material shader names are stored as custom properties
2. Modify the mesh in Blender
3. Export — shader names are preserved automatically

**For new models:** add a custom property `cgf_full_name` to each material with the value in format `materialname(ShaderName)/surfaceName`, e.g. `mymat(Phong)/mat_default`. Without a shader name the engine will not render the mesh.

---

### Animation — CAF

**File → Export → CryEngine Animation (.caf)**

Exports the active Action on the selected armature as a CAF file.

Select the armature, set the active Action in the Action Editor, then export.

---

### Animation List — CAL

**File → Export → CryEngine Animation List (.cal)**

Exports **all Actions** that have bone animation curves as individual CAF files, then writes a CAL list file referencing them all.

---

## Supported Chunk Types

| Chunk | Description |
|---|---|
| `0x0000` Mesh | Geometry, UVs, normals, vertex colors, bone weights |
| `0x000B` Node | Scene hierarchy and world transform |
| `0x000C` Material | Colors, shader name, texture references |
| `0x0003` BoneAnim | Skeleton definition |
| `0x0005` BoneNameList | Bone names (v745 and other versions) |
| `0x000D` Controller | Animation keys (v826 typed and v827 pos+rotLog) |
| `0x000F` BoneMesh | Bone physics meshes |
| `0x0011` MeshMorphTarget | Shape keys / facial expressions |
| `0x0012` BoneInitialPos | Bone rest pose matrices |
| `0x000E` Timing | Animation timing / FPS |
| `0x0013` SourceInfo | Source file metadata |

---

## Coordinate System & Scale

CryEngine 1 / Far Cry was authored in **3ds Max with inches**.

- **Scale:** `1 Max inch = 0.0254 Blender meters` — applied automatically on import and export
- **Axes:** Max/CryEngine Z-up ↔ Blender Z-up (compatible, node transforms handle orientation)
- **Object scale:** always `(1, 1, 1)` — scale is baked into vertex coordinates

---

## Material Name Format

CryEngine embeds the shader name inside the material name string:

```
materialname(ShaderName)/surfaceName
```

Examples from Far Cry:
- `s_mut_abrr(TemplBumpSpec_GlossAlpha)/mat_default`
- `floor(Phong)/mat_concrete`

This full name is stored as `cgf_full_name` custom property on imported materials and restored on export. For new materials, set this property manually.

---

## Known Limitations

- BSpline controller types not supported (rare in Far Cry assets)
- VertAnim (vertex animation / CGA) not yet exported
- Physics constraints on bones not exported (zeros written)

---

## Credits

- Original **CryImporter for 3ds Max** by [Takaro Pty. Ltd.](https://www.takaro.net) — binary format parsing and animation logic ported directly from their MaxScript
- Binary format verified against original Far Cry game files
- Blender addon port and Python rewrite — this project

---

## License

This software is provided **as-is**, without any express or implied warranty.
Based on CryImporter by Takaro Pty. Ltd. — original license terms apply.

Free for non-commercial use. See `LICENSE` for details.
