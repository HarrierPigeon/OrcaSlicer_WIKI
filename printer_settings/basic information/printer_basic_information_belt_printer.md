# Belt Printer

> [!IMPORTANT]
> NEW FEATURE: **Belt printer support**  
> Available in: [Nightly builds](https://github.com/OrcaSlicer/OrcaSlicer/releases/tag/nightly-builds) or Releases greater than **2.4.2**.

![belt_printer_settings_group](https://github.com/OrcaSlicer/OrcaSlicer_WIKI/blob/main/images/belt/belt_printer_settings_group.png?raw=true)

These settings configure conveyor belt printers and are found in **Printer settings → Basic information → Belt printer**. The group is shown when the settings mode is **Advanced** or higher. The settings below [Enable belt printing](#enable-belt-printing) remain hidden until that option is enabled.

The [Belt Printing](belt_printing) guide explains how these settings work together. The belt profiles included with OrcaSlicer are configured for a typical 45° machine, so most users only need to check the [Belt tilt](#belt-tilt).

- [Enable belt printing](#enable-belt-printing)
- [Infinite Y axis](#infinite-y-axis)
- [Belt tilt](#belt-tilt)
    - [Tilt axis](#tilt-axis)
    - [Tilt angle](#tilt-angle)
    - [Global](#global)
- [Pre-slice axis remap](#pre-slice-axis-remap)
- [Global mesh transforms](#global-mesh-transforms)
- [G-code back-transform](#g-code-back-transform)
- [First layer plane](#first-layer-plane)
    - [Plane](#plane)
    - [Belt plane offset](#belt-plane-offset)
    - [Plane band thickness](#plane-band-thickness)
- [Support floor Z offset](#support-floor-z-offset)
- [Z offset mode](#z-offset-mode)
- [Floor mode](#floor-mode)

## Enable belt printing

[Mode](option_mode): `Advanced`.  
[Variable](built_in_placeholders_variables): `belt_printer`.  
[Type](option_type#boolean): `Boolean`.  
[CLI Example](cli_mode#setting-overrides): `--belt-printer=1`.  
Enables belt printer mode. The model is rotated by the [belt tilt](#belt-tilt) before slicing, and the G-code is transformed for the tilted gantry and moving belt.

Enabling this option also changes how several features behave:

- The [brim](others_settings_brim) is printed on the belt, and additional brim options become available.
- Supports end on the belt surface.
- The [belt purge tower](printer_multimaterial_wipe_tower#belt-purge-tower) replaces the prime tower.
- Features that cannot work on a belt are disabled. See [Features that are not available](belt_printing#features-that-are-not-available).

When this option is disabled, belt-specific processing is inactive and the remaining settings on this page are hidden.

## Infinite Y axis

[Mode](option_mode): `Advanced`.  
[Variable](built_in_placeholders_variables): `belt_printer_infinite_y`.  
[Type](option_type#boolean): `Boolean`.  
[CLI Example](cli_mode#setting-overrides): `--belt-printer-infinite-y=1`.  
Removes the build volume limit along Y, allowing a part to extend beyond the plate displayed in Prepare.

The displayed plate size stays the same, but OrcaSlicer ignores the Y boundary when checking whether an object lies outside the printable area. The belt width (X) and the [printable height](printer_basic_information_printable_space#printable-height) are still enforced.

Disable this option to have OrcaSlicer flag parts that extend beyond the plate along Y.

## Belt tilt

[Mode](option_mode): `Advanced`.  
[Variables](built_in_placeholders_variables): `belt_slice_rotation`, `belt_slice_rotation_angle`, `belt_slice_rotation_global`.  
[Type](option_type): `belt_slice_rotation` (Choice: none, x, y, z), `belt_slice_rotation_angle` (Float), `belt_slice_rotation_global` (Boolean).  
[CLI Example](cli_mode#setting-overrides): `--belt-slice-rotation=none` (`belt_slice_rotation` shown; other variables above follow their own type).  
Defines the tilt of the gantry relative to the belt. The selected axis and angle control:

- the rotation applied to the model before slicing,
- the [machine-frame tilt](printer_basic_information_machine_frame_transforms#machine-frame-tilt) applied to the G-code,
- the belt surface used to generate supports and the brim,
- the [build plate tilt](printer_basic_information_advanced#build-plate-tilt) used for support generation,
- the size and position of the [belt purge tower](printer_multimaterial_wipe_tower#belt-purge-tower) and the order in which [Auto arrange](prepare_auto_arrange) packs parts.

### Tilt axis

The axis about which the model is rotated.

- **X:** The usual layout. The gantry is tilted about X and the belt travels along Y in Prepare.
- **Y:** The gantry is tilted about Y and the belt travels along X in Prepare.
- **Z:** Rotates the model in the plane of the bed. This is not a tilt: no machine-frame transform is applied and no brim is generated.
- **None:** No rotation. The model is sliced as on a flat bed.

### Tilt angle

The angle between the belt and the gantry, in degrees. Most belt printers use 45°.

A positive value rotates counter-clockwise when looking down the positive tilt axis. The sign determines the direction in which the layers lean, and the magnitude specifies the physical tilt angle.

> [!NOTE]
> A brim is only generated on a belt printer when the tilt axis is X or Y and the angle is between 1° and 85°.

### Global

Rotates every object about one common origin instead of about its own center.

When enabled, this option accounts for each object's position along the belt. All parts share the same set of tilted layers, and each part prints at its assigned position. Leave this option enabled for a belt printer.

## Pre-slice axis remap

[Mode](option_mode): `Developer`.  
[Variables](built_in_placeholders_variables): `preslice_remap_x`, `preslice_remap_y`, `preslice_remap_z`, `preslice_remap_global`.  
[Type](option_type): `preslice_remap_x` (Choice: pos_x, pos_y, pos_z, neg_x, neg_y, neg_z, rev_x, rev_y, rev_z), `preslice_remap_y` (Choice: pos_x, pos_y, pos_z, neg_x, neg_y, neg_z, rev_x, rev_y, rev_z), `preslice_remap_z` (Choice: pos_x, pos_y, pos_z, neg_x, neg_y, neg_z, rev_x, rev_y, rev_z), `preslice_remap_global` (Boolean).  
[CLI Example](cli_mode#setting-overrides): `--preslice-remap-x=pos_x` (`preslice_remap_x` shown; other variables above follow their own type).  
Reassigns the model axes before slicing to align the slicer's XY plane with a machine bed that lies in a different plane. Each of the **X**, **Y** and **Z** fields selects the model axis and direction to use for that slicer axis.

The available values have the same meanings as in the G-code axis remap. See [How axis remapping works](printer_basic_information_machine_frame_transforms#how-axis-remapping-works).

The **Global** checkbox makes the remap account for each object's position on the bed instead of applying it around the object's center. This checkbox is ignored when [Global mesh transforms](#global-mesh-transforms) is enabled, because that setting already accounts for object positions.

> [!WARNING]
> The included belt profiles leave this remap at its default (+X, +Y, +Z) and use the [belt tilt](#belt-tilt) together with the [G-code axis remap](printer_basic_information_machine_frame_transforms#g-code-axis-remap) instead. Change it only when creating a profile for a machine with different kinematics.

## Global mesh transforms

[Mode](option_mode): `Expert`.  
[Variable](built_in_placeholders_variables): `belt_preslice_global`.  
[Type](option_type#boolean): `Boolean`.  
[CLI Example](cli_mode#setting-overrides): `--belt-preslice-global=1`.  
Makes the pre-slice transforms (rotation and axis remap) account for each object's position on the bed, ensuring correct G-code coordinates for objects at different positions along the belt.

When this option is enabled, each instance of an object is sliced separately because copies at different positions along the belt have different layers. Leave this option enabled for a belt printer.

## G-code back-transform

[Mode](option_mode): `Expert`.  
[Variable](built_in_placeholders_variables): `gcode_back_transform`.  
[Type](option_type#boolean): `Boolean`.  
[CLI Example](cli_mode#setting-overrides): `--gcode-back-transform=1`.  
Undoes the pre-slice rotation and remap on every point written to the G-code, before the [G-code axis remap](printer_basic_information_machine_frame_transforms#g-code-axis-remap) and the [machine-frame tilt](printer_basic_information_machine_frame_transforms#machine-frame-tilt) are applied.

This option is required for the standard belt printing process. When disabled, the coordinates remain in the rotated coordinate system used for slicing.

## First layer plane

[Modes](option_mode):  
`Expert` [Variable](built_in_placeholders_variables): `first_layer_plane`.  
`Advanced` [Variables](built_in_placeholders_variables): `first_layer_plane_offset`, `first_layer_plane_thickness`.  
[Type](option_type): `first_layer_plane` (Choice: auto, xy, yz, xz, belt_affine), `first_layer_plane_offset` (Float), `first_layer_plane_thickness` (Float).  
[CLI Example](cli_mode#setting-overrides): `--first-layer-plane=auto` (`first_layer_plane` shown; other variables above follow their own type).  
On a belt printer, a single tilted layer contains extrusions at many heights above the belt, so layer number alone cannot identify the "first layer." These settings define a reference plane instead. First-layer settings apply to extrusions within one first layer height of that plane.

This affects first-layer speed, acceleration, jerk and temperature. Settings that count layers, such as [No cooling for the first](material_cooling#no-cooling-for-the-first), count bands above the plane instead.

### Plane

- **Auto:** Uses **Belt affine plane** when belt printing is enabled with a non-zero [belt tilt](#belt-tilt), and **XY (machine bed)** otherwise. This is the recommended choice for all included belt profiles.
- **XY (machine bed):** Applies first-layer settings to the first sliced layer, as on a flat-bed printer.
- **YZ:** A plane perpendicular to the slicer's X axis.
- **XZ:** A plane perpendicular to the slicer's Y axis.
- **Belt affine plane:** Uses the belt surface as the reference plane. When [Belt plane offset](#belt-plane-offset) is 0, extrusion heights are measured directly from the same belt surface used to generate supports and the brim, independently of the axis remap.

### Belt plane offset

Shifts the plane along its normal, in mm. A positive value moves the plane away from the belt surface, into the model.

Leave this value at 0 on a belt printer unless you need to shift the band. With a non-zero offset, the band is calculated from the belt tilt and G-code axis remap rather than measured directly from the belt surface.

### Plane band thickness

Sets the thickness of each band above the plane, in mm. Layer-count thresholds are multiplied by this value when the plane is active. The default value of `-1` uses the [first layer height](quality_settings_layer_height#first-layer-height).

## Support floor Z offset

[Mode](option_mode): `Advanced`.  
[Variable](built_in_placeholders_variables): `belt_support_floor_offset`.  
[Type](option_type#integer-float-percentage): `Float`.  
[CLI Example](cli_mode#setting-overrides): `--belt-support-floor-offset=1`.  
Shifts the belt surface used for support generation up or down, in mm. A negative value lowers this support floor, preserving more of the support geometry. A positive value raises it.

This is a diagnostic setting. Leave it at 0 unless supports are cut off above the belt or extend below it.

## Z offset mode

[Mode](option_mode): `Expert`.  
[Variable](built_in_placeholders_variables): `belt_support_z_offset_mode`.  
[Type](option_type#choice): `Choice`.  
[Options](option_type#choice): `none, unconditional, raft_only`.  
[CLI Example](cli_mode#setting-overrides): `--belt-support-z-offset-mode=none`.  
Selects how a global Z offset is applied to support layers: **None**, **Unconditional** or **Raft only**.

> [!NOTE]
> This setting is saved with the profile, but the current support generators do not use it. Changing it has no effect on the output.

## Floor mode

[Mode](option_mode): `Developer`.  
[Variable](built_in_placeholders_variables): `belt_support_floor_mode`.  
[Type](option_type#choice): `Choice`.  
[Options](option_type#choice): `none, generator_only`.  
[CLI Example](cli_mode#setting-overrides): `--belt-support-floor-mode=none`.  
Controls whether support generation uses the belt surface as its lower boundary.

![belt_printer_settings_developer](https://github.com/OrcaSlicer/OrcaSlicer_WIKI/blob/main/images/belt/belt_printer_settings_developer.png?raw=true)

- **Generator only:** Supports stop at the belt surface. This is the default.
- **None:** Disables the belt floor. Supports are no longer stopped at the belt surface.
