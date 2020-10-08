---
description: The Color Table specification for OpenType.
title: COLR - Color Table
author: PeterCon
keywords: typography
ms.author: alib
ms.date: 9/23/2020
ms.topic: article


source-file: "https://www.microsoft.com/typography/otspec/colr.htm"
---

# COLR — Color Table

<style>
 ins{color:rgb(0,128,0); background-color:rgb(224,255,224); text-decoration:none;}
 del{color:rgb(176,0,0); background-color: rgb(255,224,224);}
 .ednote{background-color: cornsilk; font-style:italic; border-style:solid; border-width:1; border-color:orange; padding:4px;}
</style>

<p class="ednote"><b>Editor’s note:</b> The following is a draft for a revision of the COLR table spec for OT 1.9, introducing version 1 of the table. Changes shown reflect changes from version 1.8.3 of the OT spec.</p>

The COLR table adds support for multi-colored glyphs in a manner that is compatible with existing text engines and <ins>relatively </ins>easy to support with current OpenType font files.

The COLR table defines a list of base <del>glyphs —</del><ins>glyphs,</ins> which are <ins>typically </ins>regular glyphs, <del>typically</del><ins>often</ins> associated with a &#39;cmap&#39; entry. Each base glyph is associated <del>by the COLR table to a list of glyphs, each corresponding to layers that can be combined, creating a colored representation of</del><ins>with a set of glyphs composed together to create a colored presentation for</ins> the base glyph.<del> The layered glyphs are explicitly defined in bottom-up z-order and each of their advance widths must match those of the base glyph. If the font has vertical metrics, the associated layer glyphs must also have the same advance height and vertical Y origin as the base glyph.</del><ins> The COLR table works together with the [CPAL](cpal.md) table which holds the color palettes used by the color composition.</ins>

<del>The COLR table works together with the [CPAL](cpal.md) table which holds the color palettes used by the color layers.</del>

<ins>Two versions of the COLR table are defined.</ins>

<ins>Version 0 allows for a simple composition of colored elements: a linear sequence of glyphs that are stacked vertically (z-order) as layers. Each layer combines a glyph outline from the &#39;glyf&#39;, CFF or CFF2 table (referenced by glyph ID) with a solid color fill.</ins>

<ins>Version 1 supports much richer capabilities:</ins>

* <ins>The colored presentation for a base glyph can use a *directed, acyclic graph* of elements, with nodes in the graph corresponding to sub-compositions that are vertically layered.</ins>
* <ins>The individual elements can be glyph outlines, as in version 0. But they can also be compositions of elements, including a complete structure defined as the colored presentation for another base glyph.</ins>
* <ins>Fills are not limited to solid colors but can use different types of gradients.</ins>
* <ins>Several composition and blending modes are supported, providing options for how elements are graphically composed.</ins>

<ins>In addition, a COLR version 0 table can be used in variable fonts with glyph outlines being variable, but no other aspect of the color composition being variable. In version 1, several additional items can be variable:</ins>

* <ins>The design grid coordinates used to define gradients.</ins>
* <ins>The elements in transformation matrices.</ins>
* <ins>The relative placement of gradient color stops on a color line.</ins>
* <ins>The alpha values applied to individual colors.</ins>

<del>Fonts using COLR and CPAL tables must implement glyph ID 1 as the .null glyph. </del><ins>The COLR table has a dependency on the CPAL table. </ins>If the COLR table is present in a font but no CPAL table exists, then the COLR table <del>will not be supported for this font</del><ins>is ignored</ins>.

<ins>Processing of the COLR table is done on glyph sequences after text layout processing is completed and prior to rendering of glyphs. In the context of the COLR table, a *base glyph* is a glyph for which color presentation data is provided in this table. Typically, a base glyph is a glyph that may occur in a sequence that results from the text layout process. In some cases, a base glyph may be a virtual glyph used to define a re-usable color composition.</ins>

<ins>“Color glyph” will be used informally to refer to the graphic composition defined by the COLR data associated with a given base glyph. When a color glyph is used, it is a substitute for the base glyph: the base glyph is not presented. The same glyph ID may be used as an element in the color glyph definition, however.</ins>

<ins>The color values used in a color glyph definition are specified as entries in color palettes defined in the [CPAL](cpal.md) table. A font may define alternate palettes in its CPAL table; it is up to the application to determine which palette is used.</ins>

## <ins>Graphic Compositions</ins>

<ins>The graphic compositions in a color glyph definition use a set of 2D graphic concepts and constructs:</ins>

* <ins>Shapes (or *geometries*)</ins>
* <ins>Fills (or *shadings*)</ins>
* <ins>Layering—a *z-order*—of elements</ins>
* <ins>Composition modes—different ways that the content of a layer is combined with the content of layers above or below it</ins>
* <ins>Affine transformations</ins>

<ins>The simplest color glyphs use just a few of the concepts above: shapes, solid color fills, and layering. This is the set of capabilities provided by version 0 of the COLR table.</ins>

<ins>In a version 0 color glyph, a sequence of layers is defined. Each layer has a shape and a solid color fill. The shapes are obtained from glyph outlines in the &#39;glyf&#39;, &#39;CFF &#39; or CFF2 table. Colors are obtained from the CPAL table. The filled shapes in the layers are composed using only alpha blending.</ins>

<ins>The following figure illustrates the version 0 capabilities: three shapes are in a layered stack: a blue square in the bottom layer, an opaque green circle in the next layer, and a red triangle with some transparency in the top layer.</ins>

<p class="ednote"><b>Editor’s note:</b> The following figure is added. (Figures cannot be formatted to reflect changes.)</p>

<figure><img src="images\colr_fig1.png" alt="Blue square, partially overlapped by an opaque green circle, both partially overlapped by a translucent red triangle."></figure>

<ins>These capabilities are sufficient to define a color glyph such as the following:</ins>

<p class="ednote"><b>Editor’s note:</b> The following figure is added. (Figures cannot be formatted to reflect changes.)</p>

<figure><img src="images\colr_fig2.png" alt="Color glyph for the woman teacher emoji using layered shapes with solid color fills."></figure>

<ins>The basic concepts also apply to color glyphs defined using the version 1 formats. As for version 0, all shapes are defined using glyph outlines, and all colors are obtained from the CPAL table. A sequence of layers is defined, and all shapes are arranged in layers, though there are some additional ways to incorporate layers.</ins>

<ins>Also, the version 1 concept of filling a shape is similar to that for version 0, but the fills have many more possibilities. Gradients can be used as well as solid colors. But content that fills a shape can also include *more complex compositions*. A different way to describe the relationship between a glyph outline and the way it is filled is that a fill is a graphic composition, and the glyph outline defines a bounds, or *clip region*, for the fill. This is still somewhat simplified.</ins>

<ins>More precisely, a version 1 color glyph definition is directed acyclic graph that specifies a set of nested 2D graphics operations. Glyph outlines define clip regions that apply to the nested operations that “fill” the outline. Affine transforms can be set at nodes within the graph, applying to the nested operations defined by the sub-graph. Also, composition modes can be specified at nodes within the graph determining how the composition produced by the nested operations of the sub-graph is blended into the destination surface.</ins>

<ins>All of the additional capabilities will be explained in greater detail, with examples, starting with gradients.</ins>

### <ins>Gradients</ins>


<ins>**&lt;forthcoming&gt;**</ins>

<a name="header"></a>

## Header

<del>The table starts with a fixed portion describing the overall setup for the color font records. All offsets, unless otherwise noted, will be from the beginning of the table</del><ins>The COLR table begins with a header. Two versions have been defined. Offsets in the header are from the start of the table.</ins>

<a id="colr_v0"></a>

<ins>*COLR version 0:*</ins>

<table>
<tbody>

<tr>
<th>Type</th>
<th>Name</th>
<th>Description</th>
</tr>

<tr id="version">
<td>uint16</td>
<td>version</td>
<td>Table version number<del> (starts at 0)</del><ins>—set to 0.</ins></td>
</tr>

<tr id="numBaseGlyphRecs">
<td>uint16</td>
<td>numBaseGlyphRecords</td>
<td>Number of Base Glyph Records.</td>
</tr>

<tr id="baseGlyphRecordsOffset">
<td>Offset32</td>
<td>baseGlyphRecordsOffset</td>
<td>Offset <del>(from beginning of COLR table) to Base Glyph records</del><ins>to baseGlyphRecords array</ins>.</td>
</tr>

<tr id="layerRecordsOffset">
<td>Offset32</td>
<td>layerRecordsOffset</td>
<td>Offset <del>(from beginning of COLR table) to Layer Records</del><ins>to layerRecords array</ins>.</td>
</tr>

<tr id="numLayerRecords">
<td>uint16</td>
<td>numLayerRecords</td>
<td>Number of Layer Records.</td>
</tr>

</tbody>
</table>

> <ins>*Note:* For fonts that use COLR version 0, some early Windows implementations of the COLR table require glyph ID 1 to be the .null glyph.</ins>

<a id="colr_v1"></a>

<ins>*COLR version 1:*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint16</ins> | <ins>version</ins> | <ins>Table version number—set to 1.</ins> |
| <ins>uint16</ins> | <ins>numBaseGlyphRecords</ins> | <ins>May be 0 in a version 1 table.</ins> |
| <ins>Offset32</ins> | <ins>baseGlyphRecordsOffset</ins> | <ins>Offset to baseGlyphRecords array (may be NULL).</ins> |
| <ins>Offset32</ins> | <ins>layerRecordsOffset</ins> | <ins>Offset to layerRecords array (may be NULL).</ins> |
| <ins>uint16</ins> | <ins>numLayerRecords</ins> | <ins>May be 0 in a version 1 table.</ins> |
| <ins>Offset32</ins> | <ins>baseGlyphV1ListOffset</ins> | <ins>Offset to BaseGlyphV1List table.</ins> |
| <ins>Offset32</ins> | <ins>itemVariationStoreOffset</ins> | <ins>Offset to ItemVariationStore (may be NULL).</ins> |

<ins>The BaseGlyphV1List and its subtables are only used in COLR version 1. The ItemVariationStore is only used in variable fonts and in conjunction with a BaseGlyphV1List and its subtables. A font that uses only BaseGlyph and Layer records should use a version 0 table.</ins>

<ins>A font that includes a BaseGlyphV1List can also include BaseGlyph and Layer records for compatibility with applications that only support COLR version 0. For applications that support COLR version 1, if a given base glyph is supported in the BaseGlyphV1List as well as in a BaseGlyph record, the data in the BaseGlyphV1List should be used.</ins>

<ins>Color glyphs that can be implemented in COLR version 0 using BaseGlyph and Layer records can also be implemented using the version 1 BaseGlyphV1List and subtables. Thus, a font that uses a BaseGlyphV1List does not need to use the version 0 BaseGlyph and Layer records. However, a font may use the version 1 structures for some base glyphs and the version 0 structures for other base glyphs. Applications should search for a base glyph ID first in the BaseGlyphV1List, then if not found, search in the BaseGlyph records array, if present.</ins>

<a name="baseGlyphRecord"></a>

## Base Glyph <del>Record</del><ins>and Layer Records</ins>

<del>The header of the COLR table points to the base glyph record. This record is used to match the base glyph to the layered glyphs. Each base glyph record contains a base glyph index. This is usually the glyph index that is referenced in the &#39;cmap&#39; table. The number of layers is used to indicate how many color layers will be used for this base glyph. Each record then has an index to a glyph layer record. There will be numLayers of layer records for each base glyph. The firstLayerIndex refers to the lowest z-order, or bottom, glyph id for the colored glyph. The next layer record will represent the next highest glyph in the z-order, and this continues bottom-up until it reaches the numLayers glyph at the top of the z-order. The index is relative to the start of the Layer Records. The index does not have to be unique for each base glyph ID. If two base glyphs need to share the color glyphs and palette indices, this is acceptable. Likewise a Base Glyph Record could point partway into a z-order of another base glyph.</del>

<del>The base glyph records are sorted by glyph id. It is assumed that a binary search can be used to efficiently access the glyph IDs that have color values. Any use scenario that attempts to map glyphs to character codes must be aware of the mapping done by the COLR table.</del>

<ins>A BaseGlyph record is used to map a base glyph to a sequence of layer records that define the corresponding color glyph. The BaseGlyph record includes a base glyph index, an index into the layerRecords array, and the number of layers.</ins>

<ins>*BaseGlyph record:*</ins>

<table>
<tbody>

<tr>
<th>Type</th>
<th>Name</th>
<th>Description</th>
</tr>

<tr id="gID_bgr">
<td>uint16</td>
<td><del>gID</del><ins>glyphID</ins></td>
<td>Glyph ID of <del>reference glyph. This glyph is for reference only and is not rendered for color</del><ins>the base glyph</ins>.</td>
</tr>

<tr id="firstLayerIndex">
<td>uint16</td>
<td>firstLayerIndex</td>
<td>Index <del>(from beginning of the Layer Records) to the layer record. There will be numLayers consecutive entries for this base glyph</del><ins>(base 0) into the layerRecords array</ins>.</td>
</tr>

<tr id="numLayers">
<td>uint16</td>
<td>numLayers</td>
<td>Number of color layers associated with this glyph.</td>
</tr>

</tbody>
</table>

<ins>The glyph ID must be less than the numGlyphs value in the [&#39;maxp&#39;](maxp.md) table.</ins>

<ins>The BaseGlyph records should sorted in increasing glyphID order. It is assumed that a binary search can be used to efficiently access the glyph IDs that have a color glyph definition.</ins>

<ins>The color glyph for a given base glyph is defined by the consecutive records in the layerRecords array for the specified number of layers, starting with the record indicated by firstLayerIndex. The first record in this sequence is the bottom layer in the z-order, and each subsequent layer is stack on top of the previous layer.</ins>

<ins>Note that the layer record sequences for two different base glyphs can overlap, with some layer records used in multiple color glyph definitions.</ins>

<a name="layerRecord"></a>

## <del>Layer Record</del>

<ins>The Layer record specifies the glyph used as the graphic element for a layer and the solid color fill.</ins>

<ins>*Layer record:*</ins>

<table>
<tbody>

<tr>
<th>Type</th>
<th>Name</th>
<th>Description</th>
</tr>

<tr id="gID_lr">
<td>uint16</td>
<td><del>gID</del><ins>glyphID</ins></td>
<td>Glyph ID of <del>layer glyph (must be in z-order from bottom to top)</del><ins>the glyph used for a given layer</ins>.</td>
</tr>

<tr id="paletteIndex">
<td>uint16</td>
<td>paletteIndex</td>
<td><del>Index value to use with a selected color palette. This value must be less than numPaletteEntries in the <a href="cpal.md">CPAL</a> table. A palette entry index value of 0xFFFF is a special case indicating that the text foreground color (defined by a higher-level client) should be used and shall not be treated as actual index into <a href="cpal.md">CPAL</a> ColorRecord array</del><ins>Index for a palette entry in the CPAL table</ins>.</td>
</tr>

</tbody>
</table>

<ins>The glyphID in a Layer record must be less than the numGlyphs value in the [&#39;maxp&#39;](maxp.md) table. That is, it must be a valid glyph with outline data in the [&#39;glyf&#39;](glyf.md), [&#39;CFF &#39;](cff.md) or [CFF2](cff2.md) table. The advance width of the referenced glyph must be the same as that of the base glyph.</ins>

<ins>The paletteIndex value must be less than the numPaletteEntries value in the [CPAL](cpal.md) table. A paletteIndex value of 0xFFFF is a special case, indicating that the text foreground color (as determined by the application) is to be used.</ins>

<del>The selection of color palette to be used for a given layer record is the responsibility of a higher-level client. With CPAL version 0, the palette selection needs to be made based on the information distributed with a font. CPAL version 1 offers user-selectable color palettes based on a textual descriptions of palette entries and palette labels.</del>

## <ins>BaseGlyphV1List and LayerV1List</ins>

<ins>The BaseGlyphV1List table is, conceptually, similar to the baseGlyphRecords array in COLR version 0, providing records that map a base glyph to a color glyph definition. The color glyph definition is significantly different, however, defined in a LayerV1List table rather than a sequence of layer records.</ins>

<ins>*BaseGlyphV1List table:*</ins>

| <ins>Type</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint32</ins> | <ins>numBaseGlyphV1Records</ins> |  |
| <ins>BaseGlyphV1Record</ins> | <ins>baseGlyphV1Records[numBaseGlyphV1Records]</ins> | |

<ins>*BaseGlyphV1Record:*</ins>

| <ins>Type</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint16</ins> | <ins>glyphID</ins> | <ins>Glyph ID of the base glyph.</ins> |
| <ins>Offset32</ins> | <ins>layerListOffset</ins> | <ins>Offset to LayerV1List table, from start of BaseGlyphsV1List table.</ins> |

<ins>*Note:* The glyph ID is not limited to the numGlyphs value in the &#39;maxp&#39; table.</ins>

<ins>The records in the baseGlyphV1Records array should sorted in increasing glyphID order.</ins>

<ins>A LayerV1List table defines the graphic composition for a color glyph as a sequence of *Paint* subtables.</ins>

<ins>*LayerV1List table:*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>numLayers</ins> |  |
| <ins>Offset32</ins> | <ins>paintOffset[numLayers]</ins> | <ins>Offsets to Paint tables, from the start of the LayerV1List table.</ins> |

<ins>Several formats for the Paint subtable are defined, each providing a different graphic capability. A format field is the first field for each format. Specifications for each format is provided below.</ins>

<ins>Each paint table will typically have a subtable graph to define a graphic composition. The composition must define a bounded region. If a paint table in the list defines an unbounded composition, it must be ignored. See above for more details.</ins>

<ins>The sequence of offsets to paint tables corresponds to a z-order layering of the graphic compositions defined by each paint table. The first paint table defines the element at the bottom of the z-order, and each subsequent paint table defines an element that is layered on top of the previous element.</ins>

### <ins>Bounding Box</ins>

<ins>The bounding box of the base glyph specified in the BaseGlyphV1Record is used at the bounding box for the color glyph defined in the corresponding LayerV1List.</ins>

<ins>Note that a &#39;glyf&#39; entry with two points at diagonal extrema is sufficient to define the bounding box.</ins>

> <ins>*Note:* Applications can use the bounding box to allocate a drawing surface without first needing to traverse the color glyph definition.</ins>

## <ins>Formats Used Within Paint Tables</ins>

<ins>Before providing specifications for the Paint table formats, various building-block elements used in paint tables will be described: variation records, colors and color lines, transforms, and composition modes.</ins>

### <ins>Variation Records</ins>

<ins>Several values contained within the Paint tables or their subtable formats are variable. These use various record formats that combine a basic data type with a variation delta-set index: [VarFWord](otvarcommonformats.md#varfword), [VarUFWord](otvarcommonformats.md#varufword), [VarF2Dot14](otvarcommonformats.md#varf2dot14), and [VarFixed](otvarcommonformats.md#varfixed). These are described in the chapter, [OpenType Font Variations Common Table Formats](otvarcommonformats.md).</ins>

### <ins>Colors and Color Lines</ins>

<ins>Colors are used in solid color fills for graphic elements, or as *stops* in a color line used to define a gradient. Colors are defined by reference to palette entries in the [CPAL](cpal.md) table. While CPAL entries include an alpha component, a *ColorIndex* record is defined here that includes a separate alpha specification that supports variation in a variable font.</ins>

<ins>*ColorIndex record:*</ins>

| <ins>Type</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint16</ins> | <ins>paletteIndex</ins> | <ins>Index for a CPAL palette entry.</ins> |
| <ins>VarF2Dot14</ins> | <ins>alpha</ins> | <ins>Variable alpha value.</ins> |

<ins>A paletteIndex value of 0xFFFF is a special case, indicating that the text foreground color (as determined by the application) is to be used.</ins>

<ins>The alpha.value is always set explicitly. Values for alpha outside the range [0., 1.] (inclusive) are reserved; values outside this range must be clipped. A value of zero means no opacity (fully transparent); 1.0 means opaque (no transparency). The alpha indicated in this record is multiplied with the alpha component of the CPAL entry (converted to float—divide by 255). Note that the resulting alpha value can be combined with and does not supersede alpha or opacity attributes set in higher-level contexts.</ins>

<ins>Gradients are defined using a color line, which is a specification of color values at proportional distances from the start to the end of the line.</ins>

<ins>*ColorStop record:*</ins>

| <ins>Type</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>VarF2Dot14</ins> | <ins>stopOffset</ins> | <ins>Proportional distance on a color line; variable.</ins> |
| <ins>ColorIndex</ins> | <ins>color</ins> | |

<ins>Values for stopOffset outside the range [0., 1.] (inclusive) are reserved; values outside this range must be clipped.</ins>

<ins>A color line is defined by array of color stops.</ins>

<ins>*ColorLine table:*</ins>

| <ins>Type</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>extend</ins> | <ins>An Extend enum value.</ins> |
| <ins>uint16</ins> | <ins>numStops</ins> | <ins>Number of ColorStop records.</ins> |
| <ins>ColorStop</ins> | <ins>colorStops[numStops]</ins> | |

<ins>The colorStops array should be in increasing stopOffset order.</ins>

<ins>A color line defines stops at proportional distances along the line, but in a gradient specification the start and end of the line are given positions in the glyph design grid. However, the color gradation can extend beyond those limits, depending on the graphic element that is being filled. Conceptually, the color line is extended infinitely in either direction beyond the [0, 1] range. The extend field is used to indicate how the color line is extended. The same behavior is used for extension in both directions.</ins>

<ins>The extend field uses the following enumeration:</ins>

<ins>*Extend enumeration:*</ins>

| <ins>Value</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>0</ins> | <ins>EXTEND_PAD</ins>     | <ins>Use nearest color stop.</ins> |
| <ins>1</ins> | <ins>EXTEND_REPEAT</ins>  | <ins>Repeat from farthest color stop.</ins> |
| <ins>2</ins> | <ins>EXTEND_REFLECT</ins> | <ins>Mirror color line from nearest end.</ins> |

<ins>EXTEND_PAD: All positions on the extended color line use the color of the closest color stop. By analogy, given a sequence “ABC”, it is extended to “…*AA* ABC *CC*…”.</ins>

<ins>EXTEND_REPEAT: The color line is repeated by extrapolating the design grid positions in the gradient definition in either direction. In either direction, the first color in the extended color line is that of the farthest color stop. By analogy, given a sequence “ABC”, it is extended to “…*ABC* ABC *ABC*…”.</ins>

<ins>EXTEND_REFLECT: The color line is repeated by extrapolating the design grid positions in the gradient definition in either direction. However, the ordering of colors along the extension in either direction is reversed. For each repetition of the color line, colors are reversed again. By analogy, given a sequence “ABC”, it is extended to “…*ABC CBA* ABC *CBA ABC*…”.</ins>

<ins>See above for graphical illustrations of these effects.</ins>

<ins>If a ColorLine in a font has an unrecognized extend value, applications should use EXTEND_PAD by default.</ins>

### <ins>Affine Transformation Matrix</ins>

<ins>A 2×3 affine transformation matrix is used to provide transformations of the design grid. The 2×3 supports scale, skew, reflection, rotation, and translation transformations. The matrix elements use VarFixed records, allowing the transform definition to be variable in a variable font.</ins>

<ins>Matrix operations are of the form *v&#x2032; = Mv*, where *v* and *v&#x2032;* are vectors for positions in the design grid. The starting position vector *v* is an extended 3×1 column matrix with the value 1 as a third matrix element: (x,y,1). The result vector *v&#x2032;* is a 2×1 column matrix (x&#x2032;,y&#x2032;).</ins>

<ins>*Affine2x3 record:*</ins>

| <ins>Type</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>VarFixed</ins> | <ins>xx</ins> | |
| <ins>VarFixed</ins> | <ins>xy</ins> | |
| <ins>VarFixed</ins> | <ins>yx</ins> | |
| <ins>VarFixed</ins> | <ins>yy</ins> | |
| <ins>VarFixed</ins> | <ins>dx</ins> | <ins>Translation in x direction.</ins> |
| <ins>VarFixed</ins> | <ins>dy</ins> | <ins>Translation in y direction.</ins> |

### <ins>Composition Modes</ins>

<ins>Composition modes are used to specify how two graphical compositions, one layered on top of the other, are composed together. Supported composition modes are taken from the W3C [Compositing and Blending Level 1][1] specification. In Paint tables, a composition mode is specified using the following enumeration.</ins>

<ins>*CompositeMode enumeration:*</ins>

| <ins>Value</ins> | <ins>Name</ins> | <ins>Description</ins> |
|-|-|-|
| | <ins>*Porter-Duff modes*</ins> | |
| <ins>0</ins> | <ins>COMPOSITE_CLEAR</ins> | <ins>See [Clear][2]</ins> |
| <ins>1</ins> | <ins>COMPOSITE_SRC</ins> | <ins>See [Copy][3]</ins> |
| <ins>2</ins> | <ins>COMPOSITE_DEST</ins> | <ins>See [Destination][4]</ins> |
| <ins>3</ins> | <ins>COMPOSITE_SRC_OVER</ins> | <ins>See [Source Over][5]</ins> |
| <ins>4</ins> | <ins>COMPOSITE_DEST_OVER</ins> | <ins>See [Destination Over][6]</ins> |
| <ins>5</ins> | <ins>COMPOSITE_SRC_IN</ins> | <ins>See [Source In][7]</ins> |
| <ins>6</ins> | <ins>COMPOSITE_DEST_IN</ins> | <ins>See [Destination In][8]</ins> |
| <ins>7</ins> | <ins>COMPOSITE_SRC_OUT</ins> | <ins>See [Source Out][9]</ins> |
| <ins>8</ins> | <ins>COMPOSITE_DEST_OUT</ins> | <ins>See [Destination Out][10]</ins> |
| <ins>9</ins> | <ins>COMPOSITE_SRC_ATOP</ins> | <ins>See [Source Atop][11]</ins> |
| <ins>10</ins> | <ins>COMPOSITE_DEST_ATOP</ins> | <ins>See [Destination Atop][12]</ins> |
| <ins>11</ins> | <ins>COMPOSITE_XOR</ins> | <ins>See [XOR][13]</ins> |
| | <ins>*Separable color blend modes:*</ins> | |
| <ins>12</ins> | <ins>COMPOSITE_SCREEN</ins> | <ins>See [screen blend mode][14]</ins> |
| <ins>13</ins> | <ins>COMPOSITE_OVERLAY</ins> | <ins>See [overlay blend mode][15]</ins> |
| <ins>14</ins> | <ins>COMPOSITE_DARKEN</ins> | <ins>See [darken blend mode][16]</ins> |
| <ins>15</ins> | <ins>COMPOSITE_LIGHTEN</ins> | <ins>See [lighten blend mode][17]</ins> |
| <ins>16</ins> | <ins>COMPOSITE_COLOR_DODGE</ins> | <ins>See [color-dodge blend mode][18]</ins> |
| <ins>17</ins> | <ins>COMPOSITE_COLOR_BURN</ins> | <ins>See [color-burn blend mode][19]</ins> |
| <ins>18</ins> | <ins>COMPOSITE_HARD_LIGHT</ins> | <ins>See [hard-light blend mode][20]</ins> |
| <ins>19</ins> | <ins>COMPOSITE_SOFT_LIGHT</ins> | <ins>See [soft-light blend mode][21]</ins> |
| <ins>20</ins> | <ins>COMPOSITE_DIFFERENCE</ins> | <ins>See [difference blend mode][22]</ins> |
| <ins>21</ins> | <ins>COMPOSITE_EXCLUSION</ins> | <ins>See [exclusion blend mode][23]</ins> |
| <ins>22</ins> | <ins>COMPOSITE_MULTIPLY</ins> | <ins>See [multiply blend mode][24]</ins> |
| | <ins>*Non-separable color blend modes:*</ins> | |
| <ins>23</ins> | <ins>COMPOSITE_HSL_HUE</ins> | <ins>See [hue blend mode][25]</ins> |
| <ins>24</ins> | <ins>COMPOSITE_HSL_SATURATION</ins> | <ins>See [saturation blend mode][26]</ins> |
| <ins>25</ins> | <ins>COMPOSITE_HSL_COLOR</ins> | <ins>See [color blend mode][27]</ins> |
| <ins>26</ins> | <ins>COMPOSITE_HSL_LUMINOSITY</ins> | <ins>See [luminosity blend mode][28]</ins> |

<ins>For details on the composition modes, see the W3C specification. See above for some graphical illustrations.</ins>

## <ins>Paint Tables</ins>

<ins>Seven Paint table formats (formats 1 to 7) are defined. Formats 1, 2, and 3 define fills. Format 4 uses a glyph outline to define a geometry. Format 5 allows an entire color glyph definition from the BaseGlyphV1List to be re-used as a component in another color glyph definition. Format 6 allows a composition, defined using a separate paint table, to be transformed. Format 7 allows compositing of two compositions, each defined using separate paint tables.</ins>

<ins>A color glyph definition using paint tables comprises a directed graph. This graph is expected to be *acyclic*. Paint format 5 creates potential for circularity by allowing the color glyph definition for a given glyph ID to reference its own glyph ID at some node in the graph. Applications should monitor the glyph ID in format 5 to see if has occurred at a higher node within the graph and, if so, ignore that sub-graph.</ins>

### <ins>Paint Format 1: Solid color fill</ins>

<ins>Format 1 is used to specify a solid color fill.</ins>

<ins>*PaintSolid table (format 1):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 1.</ins> |
| <ins>ColorIndex</ins> | <ins>color</ins> | <ins>Solid color fill.</ins> |

### <ins>Paint Format 2: Linear gradient fill</ins>

<ins>Format 2 is used to specify a linear gradient fill.</ins>

<ins>*PaintLinearGradient table (format 2):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 2.</ins> |
| <ins>Offset24</ins> | <ins>colorLineOffset</ins> | <ins>Offset to ColorLine, from start of PaintLinearGradient table.</ins> |
| <ins>VarFWord</ins> | <ins>x0</ins> | <ins>Start point x coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>y0</ins> | <ins>Start point y coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>x1</ins> | <ins>End point x coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>y1</ins> | <ins>End point y coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>x2</ins> | <ins>Rotation vector end point x coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>y2</ins> | <ins>Rotation vector end point y coordinate.</ins> |

<ins>The rotation vector uses the same start point as the gradient line vector. See above for more information.</ins>

### <ins>Paint Format 3: Radial/conic gradient fill</ins>

<ins>Format 3 is used to define a class of gradients that are a functional superset of a radial gradient: the color gradation is along a cylinder defined by two circles. In the general case, the circles can have different radii to create a conical cylinder. A radial gradient in the strict sense, with color gradation along rays from a single focal point, is formed by the starting circle having a radius of zero with center located inside the ending circle. See above for more information.</ins>

<ins>*PaintRadialGradient table (format 3):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 3.</ins> |
| <ins>Offset24</ins> | <ins>colorLineOffset</ins> | <ins>Offset to ColorLine, from start of PaintRadialGradient table.</ins> |
| <ins>VarFWord</ins> | <ins>x0</ins> | <ins>Start circle center x coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>y0</ins> | <ins>Start circle center y coordinate.</ins> |
| <ins>VarUFWord</ins> | <ins>radius0</ins> | <ins>Start circle radius.</ins> |
| <ins>VarFWord</ins> | <ins>x1</ins> | <ins>End circle center x coordinate.</ins> |
| <ins>VarFWord</ins> | <ins>y1</ins> | <ins>End circle center y coordinate.</ins> |
| <ins>VarUFWord</ins> | <ins>radius1</ins> | <ins>End circle radius.</ins> |

### <ins>Paint Format 4: Glyph outline</ins>

<ins>Format 4 is used to define a clip region using a glyph outline. The outline sets a clip region that constrains the content of a separate paint subtable. Conceptually, the paint subtable defines a (potentially complex) fill for the outline.</ins>

<ins>*PaintGlyph table (format 4):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 4.</ins> |
| <ins>Offset24</ins> | <ins>paintOffset</ins> | <ins>Offset to a Paint table, from start of PaintGlyph table.</ins> |
| <ins>uint16</ins> | <ins>glyphID</ins> | <ins>Glyph ID for the source outline.</ins> |

<ins>The glyphID value must be less than the numGlyphs value in the [&#39;maxp&#39;](maxp.md) table. That is, it must be a valid glyph with outline data in the [&#39;glyf&#39;](glyf.md), [&#39;CFF &#39;](cff.md) or [CFF2](cff2.md) table.</ins>

### <ins>Paint Format 5: COLR composition</ins>

<ins>Format 5 is used to allow a color glyph definition from the BaseGlyphV1List to be a re-usable component in multiple color glyph definitions.</ins>

<ins>*PaintColrGlyph table (format 5):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 5.</ins> |
| <ins>uint16</ins> | <ins>glyphID</ins> | <ins>Virtual glyph ID for a BaseGlyphV1List base glyph.</ins> |

<ins>The glyphID value must be a glyphID found in a BaseGlyphV1Record within the BaseGlyphV1List. It may be a *virtual* glyph ID, greater than or equal to the numGlyph value in the [&#39;maxp&#39;](maxp.md) table. The composition defined by the associated LayerV1List is used as a component within the current color glyph definition.</ins>

### <ins>Paint Format 6: Transformed composition</ins>

<ins>Format 6 is used to apply an affine 2×3 transform to a graphical composition defined by a separate paint table.</ins>

<ins>*PaintTransformed table (format 6):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 6.</ins> |
| <ins>Offset24</ins> | <ins>paintOffset</ins> | <ins>Offset to a Paint subtable, from start of PaintTransformed table.</ins> |
| <ins>Affine2x3</ins> | <ins>transform</ins> | <ins>An Affine2x3 record (inline).</ins> |

<ins>When the composition in the referenced paint table is composed into the destination (represented by the parent of this table), the source design grid origin is aligned to the destination design grid origin. The transform may translate the source such that a pre-transform position (0,0) is moved elsewhere. The post-transform origin, (0,0), is aligned to the destination origin.</ins>

### <ins>Paint Format 7: Composite</ins>

<ins>Format 7 is used to blend two layered compositions using different composition modes.</ins>

<ins>*PaintComposite table (format 7):*</ins>

| <ins>Type</ins> | <ins>Field name</ins> | <ins>Description</ins> |
|-|-|-|
| <ins>uint8</ins> | <ins>format</ins> | <ins>Set to 7.</ins> |
| <ins>Offset24</ins> | <ins>sourcePaintOffset</ins> | <ins>Offset to a source Paint table, from start of PaintComposite table.</ins> |
| <ins>uint8</ins> | <ins>compositeMode</ins> | <ins>A CompositeMode enumeration value.</ins> |
| <ins>Offset24</ins> | <ins>backdropPaintOffset</ins> | <ins>Offset to a backdrop Paint table, from start of PaintComposite table.</ins> |

<ins>The composition defined by the source paint table is layered on top of and blended into the destination composition defined by the backdrop paint table.</ins>

<ins>The compositionMode must be one of the values defined in the CompositeMode enumeration. If an unrecognized value is encountered, COMPOSITE_CLEAR should be used.</ins>

[1]: https://www.w3.org/TR/compositing-1/
[2]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_clear
[3]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_src
[4]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dst
[5]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcover
[6]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstover
[7]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcin
[8]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstin
[9]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcout
[10]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstout
[11]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcatop
[12]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstatop
[13]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_xor
[14]: https://www.w3.org/TR/compositing-1/#blendingscreen
[15]: https://www.w3.org/TR/compositing-1/#blendingoverlay
[16]: https://www.w3.org/TR/compositing-1/#blendingdarken
[17]: https://www.w3.org/TR/compositing-1/#blendinglighten
[18]: https://www.w3.org/TR/compositing-1/#blendingcolordodge
[19]: https://www.w3.org/TR/compositing-1/#blendingcolorburn
[20]: https://www.w3.org/TR/compositing-1/#blendinghardlight
[21]: https://www.w3.org/TR/compositing-1/#blendingsoftlight
[22]: https://www.w3.org/TR/compositing-1/#blendingdifference
[23]: https://www.w3.org/TR/compositing-1/#blendingexclusion
[24]: https://www.w3.org/TR/compositing-1/#blendingmultiply
[25]: https://www.w3.org/TR/compositing-1/#blendinghue
[26]: https://www.w3.org/TR/compositing-1/#blendingsaturation
[27]: https://www.w3.org/TR/compositing-1/#blendingcolor
[28]: https://www.w3.org/TR/compositing-1/#blendingluminosity
[29]: https://www.w3.org/TR/compositing-1/#blendingnormal