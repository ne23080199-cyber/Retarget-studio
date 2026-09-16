# L4D2 Retarget Studio

A native Windows application that converts a rigged FBX or compiled Garry's Mod character addon into a **Left 4 Dead 2** character replacement.

Version 2.21 adds multi-target FBX animation retargeting for 3ds Max Biped, Mixamo and ValveBiped skeletons. An animation can be baked independently onto any checked survivor or infected while preserving that target's original L4D2 bone lengths, bind axes and proportions. Root motion, loop mode and output FPS are selectable. Model compilation retains the 2.20 exact-grip and runtime index-buffer protections.

## Open the application

Run `dist/L4D2 Retarget Studio 2.21 Portable/L4D2 Retarget Studio.exe`. It is a desktop application, does not open a browser, and does not require a separate Python installation. Keep the adjacent `retarget_engine` folder with it; moving only the interface EXE will intentionally produce a clear missing-engine message.

## Animation Retarget

1. Open **Animation Retarget** and select or drop an animated FBX.
2. Enter the sequence name and output FPS. Root motion is enabled by default; enable loop only for a looping source.
3. Check one or more survivors or infected. Every checked target is processed against the exact original skeleton installed with L4D2.
4. Click **Build Animation Pack**. The result contains a compiled MDL animation library plus editable SMD/QC source for each target. The pack does not overwrite the game's complete animation libraries; include the generated library/sequence from the character addon that should use it.

FBX files readable by Valve's bundled SDK use its evaluated curves. Newer Mixamo FBX versions that the legacy SDK cannot expose automatically use the built-in curve reader instead.

On a first launch the model, texture, addon name, output and target fields are blank. The L4D2 installation is detected automatically from Steam, its configured library folders, the Windows Steam registry and common library locations. After that, the application remembers the selected files, folders, target, scale, rig/variant/preview options, Z adjustment and last successful output when it is closed, then restores the same workspace on the next launch. **Tools > Detect Left 4 Dead 2 Installation** can repeat detection manually.

## Workflow

1. Select or drop an FBX containing a rigged mesh, or a compiled Garry's Mod `.gma`/`.vpk` character addon. **Use the FBX / GMA / VPK filename automatically** is enabled by default and immediately updates the addon name; disable it only to enter a custom name.
2. For a compiled addon, select the detected model/style and use **Appearance** to choose its skin and bodygroups. Original VTF textures and a dedicated arms model are selected automatically when present.
3. Choose the survivor or infected character to replace.
4. Optionally select or drop any number of PNG, JPG, TGA, BMP or TIFF textures, or one or more folders containing textures. There is no selection-count limit.
5. Leave **Build matching first-person arms** enabled for a playable character.
6. Leave **Automatically match Valve pose and adapt reduced digits** enabled unless you specifically need the uncorrected bind pose. Original foot and toe weights are never flattened or rigidized.
7. Leave **Retarget strategy** on **Automatic — analyze and choose**. Use the two manual modes only when comparing a particular asset by hand.
8. Enable **Normalize source mesh floor/origin** only when the mesh is visibly above or below its armature/grid; it is disabled by default.
9. Optionally enable **Use custom Z offset** and enter a signed Source-unit value, or use the synchronized slider. Positive values raise the visible model and negative values lower it.
10. Leave **Show white checker floor in the model preview** enabled to judge foot contact against the original character floor.
11. Leave **Build installed character variants (Boomette / L4D1 / DLC)** enabled to include every installed alternate replacement that belongs to the selected target.
12. Click **Build VPK**, then use **Preview Model**, **Install Addon**, or open the output folder. Install Addon writes an enabled entry to `addonlist.txt`; if L4D2 is currently running, exit it completely and reopen it before testing the replacement.

The application uses the official tools installed with L4D2. It automatically reads embedded FBX images or original GMod VTF/VMT materials, generates SMD/QC source files, fits a new ragdoll collision model, compiles the body and arms with StudioMDL, and packs the result as a VPK. Compiled Source-addon mesh recovery uses the locally installed Crowbar application, which is detected from the Desktop or Downloads folder.

## Retargeting and proportions

- Measures the source and target skeleton from head to feet instead of trusting unreliable FBX unit metadata.
- Applies the FBX mesh-node scale and axis transform before fitting. This supports Blender exports where the mesh is stored at 1/100 of the bind skeleton scale and prevents tiny, violently stretched models.
- Aligns the imported bind pose to the actual up, forward and lateral axes of each individual L4D2 skeleton. This covers the different internal orientations used by survivors, special infected and common infected.
- Uses one uniform XYZ scale, preserving the imported model's proportions without stretching or flattening it.
- At 100%, matches the original height of the selected L4D2 character. The percentage control applies an optional uniform adjustment.
- Converts positions, normals, UV coordinates and the three strongest weights per vertex.
- Keeps conservative single-body reduction for very dense FBX inputs. Compiled GMA/VPK models instead retain every source triangle and are divided losslessly into 14,000-triangle compiler-safe render parts; this avoids Source's 16-bit index limit without welding hair, clothing, outline shells or separate body surfaces together.
- Uses Source Direct for compiled GMA/VPK input. At 100% it keeps the addon's original Source-unit scale, world XYZ origin, every authored body vertex and the imported arm/leg lengths. Upper arms, forearms, palms, fingers, thighs and calves use Valve-compatible animation axes; feet and toes retain their authored orientation so the compatibility pass cannot reintroduce diagonal shoes.
- Runs an Engine 2 preflight that classifies the imported rig, counts core bones and weighted digits, detects large left/right bind asymmetry and records warnings without silently changing the source.
- Builds and scores two independent candidates in Automatic mode. It compares median and 95th-percentile edge-length change, invalid vertices, invalid weights and degenerate geometry, then rejects unsafe results before StudioMDL or VPK packaging.
- Reads the exact installed MDL reference skeleton and weighted floor plane from its VVD. In source-shape mode, Valve's directions and animation axes are retained while hip, knee, ankle, toe, arm and finger segment lengths come from the FBX after one uniform character scale.
- Reorients each limb segment rigidly into the character's Valve bind pose without axial stretching. The imported body proportions, shoe dimensions and original skin-weight distribution remain intact instead of being flattened or lengthened to match the survivor/infected mesh.
- Places the fitted, foot-weighted support surface on the selected character's original ground plane. This is a translation rather than a foot scale, so it corrects floating or floor penetration without changing the shoe or paw's shape.
- Keeps target-limb fitting as a separate manual strategy for FBX input. Automatic rigid-toe stabilization and the diagonal foot-pitch transform have been removed; the foot and toe now move together as one rotation-free rigid surface while retaining their authored skin weights.
- Rebases the imported pelvis against the original character before applying the shared support-plane adjustment. The proportions animation therefore carries one consistent bind offset instead of leaving the rendered surface separated from its rotation pivots.
- Offers an optional full-mesh floor/origin normalization for badly offset FBX files. It is off by default because geometry such as tails, coats or accessories can extend below the actual feet.
- Converts a Mixamo T-pose into the selected character's original Valve bind directions. By default the compiled clavicle, upper-arm, forearm, hand, leg, foot and finger chains keep the FBX segment lengths, and the surface follows through rigid segment rotations rather than axial rescaling.
- Preserves Valve's proven leg, foot, shoulder, arm, hand and finger animation axes. Every generated axis is locked with `$definebone`, preventing StudioMDL from silently realigning limbs, lifting feet or turning finger curls into stretched claws.
- Adds a small, bounded support-plane compensation after pelvis rebasing, preventing broad feet and paws from clipping while the animated root remains compatible with the selected character.
- Keeps automatic ground placement in Source animation space instead of treating the FBX character-up axis as game-world Z. Standing, walking, running, charging and crouching therefore retain the original character's floor contact even when the imported model has unusual pelvis-to-foot proportions.
- Provides a separate signed vertical-position adjustment from -25.00 to +25.00 Source units. The numeric field and slider stay synchronized; the adjustment moves the visible mesh, third-person skeleton and fitted ragdoll together.
- Detects hands with fewer weighted fingers. A four-digit hand is spread across the Valve finger range so its outer digit receives the correct outer-finger animation instead of leaving a visual gap.
- If a target rig intentionally has no individual finger bones, or only a partial chain, missing fingertip weights attach rigidly to the corresponding hand. They never fall back to the pelvis and cannot be torn away by an animation the target skeleton does not support.
- Keeps every unused Valve finger chain next to its hand after converting a reduced-digit Mixamo rig. Unweighted Valve digits cannot retain stale T-pose offsets and contaminate the proportions animation.
- Places elbow, ulna and wrist helpers directly on the imported forearm and hand joints, keeps weapon helpers at Valve's proven hand-relative offsets, and rebuilds digit pivots from Valve's weapon-grip layout. Aim, firing and gesture animations therefore close the palm and fingers around the weapon instead of bending toward stale GMod pivots.
- Recreates the exact output directory immediately before every SMD write. Reused, synchronized or partially cleaned projects cannot fail halfway through a dense multi-part model because `source/mesh_part_XX.smd` disappeared.
- Limits only the internal filesystem identifier for unusually long source filenames, appending a stable hash to avoid collisions while retaining the complete user-facing addon title. VPK contents are copied to a short unique staging directory before invoking Valve's legacy packer, so long output folders and repeated Workshop names cannot exceed its path limit.
- Preserves Mixamo's original UV orientation when writing SMD files, keeping texture-atlas regions on the correct body parts.
- Maps standard ValveBiped skeletons and the custom skeleton variants used by special infected.
- Preserves required helper bones, original animation includes and expected base/ragdoll sequences.
- Copies each target model's original IK chains and pose-parameter ranges into the replacement. Valve's foot-contact rules can therefore place the feet on the floor during idle and movement animations instead of leaving differently proportioned meshes floating.
- Rebuilds the PHY around the imported body instead of reusing the old character's collision shape. It preserves the selected character's original rigid-body joints, total mass, inertia, damping, rotation damping, joint mass biases, axis limits, self-collision rules and animated-friction settings.
- Uses one stable convex proxy per original physics joint, fitted from the retargeted surface in bind space. This keeps the ragdoll proportional to broad, thin, short or unusually shaped imported models without exceeding Source's collision-piece limits.
- Builds arm-weighted geometry against the exact, unchanged first-person skeleton and animation model used by the selected playable character. Imported arm segments are fitted to Valve's original lengths in bind space, preventing twisted shoulders, wrists and fingers without changing the first-person animation rig.
- Applies the strict first-person crop to every playable target. Disconnected hand/arm mesh islands are kept intact while torso, head and leg islands are rejected; connected meshes use a conservative skin-weight boundary. Survivor viewmodels can no longer retain chest, back or clothing geometry through weak shoulder weights.
- Preserves the imported palm orientation relative to the forearm on infected viewmodels instead of copying the original claw model's additional wrist roll. Valve's unchanged hand and finger bones still drive every original first-person animation.
- Links Boomer, Hunter, Smoker and Tank replacement claws directly to their original `anims_v_claw_*` models. Infected arms receive no added local reference/proportions sequences, so the original numeric sequence order remains exact for Versus spawn, team-change, attack and idle viewmodel animation lookup. Charger, Jockey and Spitter expose their original locally stored sequences through an unshifted include. Survivor arms retain their existing proportions workflow.
- Honors each FBX material's explicit linked diffuse image before considering folder order. External high-resolution copies with the same linked filename override embedded images without swapping faces, clothing or accessory textures between material slots.
- Repairs stale FBX texture links copied from another computer by searching for the exact referenced filename beside the FBX and in its subfolders. The texture picker and drag-and-drop input accept unlimited files and recursively include all supported images in selected folders.
- Generates opaque materials by default so an image alpha channel cannot accidentally make the whole model invisible.
- Uses neutral imported-texture lighting by default. Original burn/gameplay material behavior is retained, while Phong, rim-light, environment and self-illumination effects that require Valve's original masks are disabled so custom textures do not turn white or develop a distant glint.
- Reopens the completed VPK index and verifies each compiled MDL, VVD and DX90 VTX body/arms family before reporting success. Every build writes a JSON report with the preflight, candidate scores, chosen strategy, scale, grounding, variants and verified package members.

## Supported targets

Third-person bodies are supported for Nick, Ellis, Coach, Rochelle, Bill, Zoey, Francis, Louis, Boomer, Boomette, Charger, Tank, Hunter, Jockey, Smoker, Spitter, Witch, Witch Bride, and male/female Common Infected.

Matching first-person arms are supported for all playable targets: the eight survivors and the playable special infected, including Boomette through the Boomer claw model. Witch, Witch Bride and Common Infected are non-playable and have no first-person arms model in L4D2, so the application correctly builds their body only.

When the installed-variant option is enabled, selecting Boomer also builds Boomette and Boomer L4D1; selecting Witch also builds Witch Bride; and Hunter L4D1, Smoker L4D1, Tank L4D1 and Tank DLC3 bodies and matching first-person claws are built automatically when those original files are installed. Boomette and Witch Bride are also selectable separately. Bill, Zoey, Francis and Louis are already available as normal L4D1 survivor targets. L4D2 contains no separate Witch L4D1 model, and Jockey did not exist in L4D1.

## Model preview

After a successful build, **Preview Model** opens a preview-only copy of the compiled body in the Valve Model Viewer installed with L4D2. The launcher validates the MDL/VVD/VTX family first, supplies the temporary project through `VPROJECT`/`VCONTENT`, mounts the L4D2 Authoring Tools context, and passes a game-relative model path so HLMV actually loads the character. By default the preview includes a white/gray checker plane at the original character's ground height, making foot penetration or floating visible before installation. The plane is not packed into the addon VPK. Close the viewer normally and click **Preview Model** again whenever you want to reopen it.

## Reference library

The **Reference Library** tab extracts the original body, arms, VVD, VTX and PHY files from the user's installed game packages, including the supported L4D1/DLC3 variants. These files are for local reference and should not be redistributed separately.

## Current limitations

- An FBX must use standard Mixamo bone names such as `mixamorig:Hips`, `Spine`, and `LeftArm`; GMod Source addons may use a standard ValveBiped playermodel rig.
- Automatic facial flexes, gore meshes and LOD generation are not included.
- A human Mixamo mesh can use special infected animations, but the application cannot invent limbs or geometry absent from the imported mesh.
- When no texture is selected, a visible checker texture is generated so the build can still finish and remain visible.

## Development

The retargeting engine is written in Python and the desktop interface uses Windows Forms. `build_exe.ps1` rebuilds the final self-contained EXE.
