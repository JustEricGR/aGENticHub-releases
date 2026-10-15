---
id: qgis-projects
title: Building QGIS projects and writing PyQGIS
triggers: [qgis, pyqgis, geopackage, gpkg, shapefile, crs, epsg, rivers, railways, elevation, basemap, layer, analysis, processing, spatial, work]
tools: [qgis, qgis-python]
---

Every rule here was paid for by a script that ran, exited 0, and produced
something wrong.

## Getting the data: fetch_osm, not a download script

If the task names a place rather than a file, `fetch_osm` is the whole data
step. It geocodes the place, queries Overpass, and writes a GeoPackage ready
for `add_vector_layer`:

    fetch_osm(area="Grenoble", kind="rivers", output="inputs/rivers.gpkg")

`kind` is a named set - rivers, waterways, water, railways, roads, buildings,
forests, parks, boundaries, peaks - not a raw tag filter. A filter written
from memory returns nothing rather than erroring, and an empty layer looks
like a place with no rivers in it.

Do not write a download script for this. Four projects in this repository
have a hand-written `osm_to_geojson.py` in their work folder, all slightly
different, all doing what that one call does.

Overpass is free and shared, with a per-IP slot limit. A burst of queries
exhausts it and every request then returns 504 for a minute or two - the
operation waits for a slot rather than failing, so a slow answer is normal
and a retry loop of your own is not needed.

## The project survives between steps

`new_project` is called ONCE, by whichever step gets there first. The project
then persists for the whole plan - a later step opens it already built.

So if your step is not the first one, call `describe_project` before anything
else and work with what is there. Calling `new_project` again throws away
every layer an earlier step added, and it reports what it discarded rather
than refusing, because starting over is a real thing to want.

Measured on a nine-step plan: step 1 created the project and added the river
layer, step 2 called `new_project`, and the layer was gone. Step 4 then
re-downloaded the same data. Every call succeeded.

## Never measure in EPSG:4326

The most expensive bug this project has shipped, and it raises no error.

`geometry().area()` returns square units **of the layer's CRS**. EPSG:4326 is
degrees, so it returns SQUARE DEGREES. A run reported the area of Romania as
85 billion km², and nothing failed - the number was simply meaningless, and
the digits looked plausible enough to publish.

Before any area or length:

- **reproject** to an equal-area CRS for areas, or an equidistant one for
  lengths, and say which in a comment
- or use `QgsDistanceArea` with `setEllipsoid()`, which does the geodesy for
  you and returns metres

Equal-area CRS by region: Europe `EPSG:3035`, world `EPSG:6933`. Both are in
`reference/crs.md` with the rest.

A number in square degrees is between 4 and 12 orders of magnitude out
depending on latitude. If an area looks impossible, this is why.

## Read layer names, never guess them

`layername=lines` was guessed and the layer was called something else. A
GeoPackage declares its layers in `gpkg_contents`:

```python
import sqlite3
names = [r[0] for r in sqlite3.connect(path).execute(
    "SELECT table_name FROM gpkg_contents")]
```

That is three lines and it cannot be wrong. A guessed layer name loads an
empty layer, which renders as an empty map rather than as an error.

## Order of operations

`QgsApplication` must be initialised before anything touches a layer, and
`exitQgis()` must come last. A script that called `exitQgis()` before writing
the project file produced a valid, empty `.qgz`.

    QgsApplication.setPrefixPath(...) -> initQgis() -> work -> write() -> exitQgis()

Write the project **before** teardown. Check the return value of
`QgsProject.instance().write(path)` - it returns False rather than raising.

## Layers must be added to the project, not just constructed

`QgsVectorLayer(...)` makes a layer object. It appears in nothing until
`QgsProject.instance().addMapLayer(layer)`. And check `layer.isValid()` first:
an invalid layer adds cleanly and renders as nothing.

## What will be checked before you finish

Not advice. These are evaluated against the script you write.

```checks
- rule: script_requires
  parts: area,length
  needs: transform,QgsDistanceArea,3035,6933
  because: a measurement in EPSG:4326 is in square degrees
- rule: script_requires
  parts: QgsVectorLayer
  needs: addMapLayer
  because: a constructed layer that is never added renders as nothing
- rule: script_requires
  parts: QgsVectorLayer
  needs: isValid
  because: an invalid layer adds cleanly and draws nothing
- rule: script_requires
  parts: gpkg,GeoPackage
  needs: gpkg_contents,layername=
  because: a guessed layer name loads empty rather than failing
- rule: script_order
  before: write
  after: exitQgis
  because: tearing down before writing produces a valid empty project
```

## Learned in training (QGIS skill training, approved)

spatial analysis and map work in QGIS: PyQGIS processing, the QGIS interface (attribute table, selection, project settings), and answering GIS questions

## Directions learned in training

### Round 1
- Always retrieve the CRS of each dataset programmatically (e.g., using layer.crs().authid()) and report that EPSG code instead of assuming a value.
- After any UI action such as sorting the attribute table or applying a selection, programmatically read back the affected features (e.g., via layer.getFeatures() or a sorted list) and verify that the order or selection matches the intended criteria before stating that it is done.
- When exporting data, use QgsVectorFileWriter.writeAsVectorFormat with the appropriate driver (e.g., "GPKG"), check the returned error code for success, and confirm the output file exists on disk (e.g., with QFileInfo) before reporting the result.

### Round 2
- Always verify the current project CRS using QgsProject.instance().crs().authid() before performing any spatial operation, and explicitly reproject geometries using QgsCoordinateTransform if calculating area or distance, never rely on the layer's native CRS.
- After any sorting or selection in the attribute table, programmatically retrieve and validate the feature IDs or values using layer.selectedFeatureIds() or a sorted list of attributes to confirm the operation succeeded as intended.
- When exporting any vector layer, always use QgsVectorFileWriter.writeAsVectorFormat with explicit driver (e.g., 'GPKG'), check the return status for QgsVectorFileWriter.NoError, and confirm the output file exists using QgsFileUtils.createParentDirectory() and QFile.exists() before reporting success.

### Round 1
- Before any layer‑specific action, retrieve the layer by name with `QgsProject.instance().mapLayersByName('layer_name')`, confirm the list is non‑empty and that `layer.isValid()` returns True; if the layer is missing, create it via `QgsVectorLayer(path, 'layer_name', 'ogr')` and add it to the project with `QgsProject.instance().addMapLayer(layer)`.
- When opening an attribute table (e.g., via `iface.showAttributeTable(layer)`), immediately verify that the active table belongs to the intended layer by checking `iface.attributeTable().layer().name()` (or the appropriate method) matches the layer name before performing sorting or selections.
- After setting a renderer or any style on a layer, invoke `layer.triggerRepaint()` and confirm the change by asserting `layer.renderer().type()` equals the expected renderer type (e.g., `'graduatedSymbol'`) to ensure the style was applied correctly.

### Round 2
- After creating any derived layer (e.g., a buffer or a selection result), immediately set its name to the exact identifier required for downstream checks with layer.setName('expected_name') and verify with layer.name() == 'expected_name' before proceeding.
- When calculating geometry metrics (area, length, count), explicitly define the measurement units using QgsDistanceArea (e.g., setSourceCrs, setEllipsoid, setUnit) and convert the result to the task's requested unit, then output the value as a plain number rounded to the required precision (e.g., print(round(value, 3))) so the answer can be parsed correctly.
- Before writing the project file, ensure the project contains at least the expected number of layers and that each required layer name matches the pattern specified (case‑insensitive substring check) by iterating over QgsProject.instance().mapLayers().values() and asserting the condition; abort or raise an error if any required layer is missing before calling QgsProject.instance().write('path.qgz').

### Round 3
- After invoking any processing algorithm (e.g., buffer, intersect, hillshade) via the processing framework, immediately inspect the returned dictionary for the 'OUTPUT' key, confirm it contains a non‑empty string path, then load the result with QgsVectorLayer/QgsRasterLayer and assert layer.isValid() before any downstream steps.
- When applying an attribute filter or selection, construct a QgsExpression and QgsFeatureRequest, retrieve matching feature IDs with layer.getFeatures(request), apply selection with layer.selectByIds(ids), and then verify layer.selectedFeatureCount() equals the expected number of features.
- Before writing any file (vector, raster, project, export image), ensure the target directory exists by calling QgsFileUtils.ensureFolderExists(QFileInfo(path).absolutePath()) and use an absolute path; after writing, confirm the file exists with QFileInfo(path).exists() and its size > 0.

### Round 4
- Always clear any existing selection on the target layer with layer.removeSelection() and confirm layer.selectedFeatureCount() == 0 before applying a new attribute‑based selection.
- Select features programmatically via layer.selectByExpression('"<field_name>" = <value>', QgsVectorLayer.SetSelection) (using double‑quoted field names), then immediately check that layer.selectedFeatureCount() equals the expected number of matches.
- When using the GUI Select by Expression dialog, after entering the expression trigger the dialog's 'Select Features' button (e.g., iface.selectByExpression(expression, layer) or iface.actionSelectByExpression().trigger()), wait for the dialog to close, and verify iface.activeLayer().selectedFeatureCount() matches the expected count before proceeding.

### Round 6
- Before confirming any dialog (e.g., Buffer, Select by Location, Export Map), verify that the intended layer is selected in the dialog's dropdown list, then click the dialog's primary action button (Run/OK/Select) and wait for the operation to complete before proceeding; afterward, programmatically or via the UI check the expected result (e.g., layer appears, selection count).
- To select features by attribute without code, open the attribute table (Layer ► Open Attribute Table or press F6), click the ‘Select features using an expression’ button (or press Ctrl+F), ensure the top of the dialog shows the correct layer, type the expression with double‑quoted field names (e.g., "floors" = 4), press the ‘Select’ button (or Enter), and then verify that the layer’s selected feature count matches the expected number.
- Export a map image by navigating the exact menu path Project ► Import/Export ► Export Map to Image…, set the extent to the current canvas (or define it), choose an absolute filename ending in .png, set the desired DPI/resolution, click the ‘Save’ button, and finally confirm that the PNG file exists on disk and has a non‑zero file size.

### Round 7
- Before saving any raster with NoData replacement, use the GDAL Translate algorithm via Processing Toolbox ► GDAL ► Translate, set 'Additional creation options' to '-a_nodata -9999' and ensure the output raster's minimum value is > 0 by checking the Layer Properties ► Statistics tab after loading, never rely on manual value substitution.
- To create a spatial selection based on proximity, always use Vector ► Geoprocessing Tools ► Buffer to create a buffer around the hazard zone, then use Vector ► Research Tools ► Select by Location with 'are within' and the buffer layer as the input, never use distance calculations without a precomputed buffer and verify the selection count via the attribute table's feature count display.
- Before exporting a map as PNG, confirm the Print Layout exists by going to Project ► New Print Layout, add a map item via the Add Item toolbar, set the extent to the desired layer, then use Layout ► Export as Image and explicitly choose PNG format, ensuring the output file is created and its size is greater than 1 KB before proceeding.

### Round 8
- After completing any QGIS dialog (e.g., Project Properties, Export, Processing), click the primary action button (OK/Apply/Run/Save), then wait until the dialog window disappears before proceeding; immediately verify the change by querying the related property (e.g., QgsProject.instance().crs().authid() for CRS changes or layer.isValid() for new layers).
- When adding a vector layer via the menu path Layer ► Add Layer ► Add Vector Layer…, use the file chooser to select the dataset, press Open, then confirm the layer appears in the Layers panel with the expected name and that layer.isValid() returns True before performing any subsequent operations.
- To export a map canvas as PNG, first create a Print Layout (Project ► New Print Layout…), set the map item extent and output size, then use Layout ► Export as Image…, enter the full path in the file field, press Save, and finally verify that the PNG file exists on disk and its file size is greater than zero bytes.

### Round 9
- When using the GUI 'Select by Location' tool (Vector ► Research Tools ► Select by Location…), first set the 'Select features from' dropdown to the target layer, then set the 'Where the features (source) are located' list to the source layer, choose the exact spatial predicate (e.g., "within"), click the 'Run' button, and finally verify the selection by checking that the target layer’s status bar shows the expected number of selected features.
- To export a map image via the interface, open Project ► Import/Export ► Export Map to Image…, enter an absolute file path ending in .png, ensure the 'Extent' is set to 'Current canvas extent' (or manually draw the extent), set the desired resolution (e.g., 300 dpi), click the 'Save' button, and after the dialog closes confirm the file exists on disk with a file‑info check.
- When saving a processed vector layer through the UI, right‑click the layer, choose Export ► Save Features As…, select format "GeoPackage", provide an absolute output path (e.g., output/at_risk_arden_500.gpkg), check the 'Add saved file to map' option, click 'OK', then programmatically or via the layer list ensure the new layer is present, is valid, and its feature count matches the expected number.

## Common errors, and what fixes them
- IndexError: list index out of range","new_files":[],"script":"pyqgis_1.py","hint":"Every script starts with an EMPTY project - no layer is loaded, whatever was added with other tools. Load what you ne
  Fix: above_500 = QgsRasterLayer('output/above_500.tif', 'above_500') | 'INPUT': 'output/above_500.tif',
- _core.QgsProcessingException: Unable to execute algorithm
  Fix: from qgis.core import QgsRasterLayer, QgsProject | result = processing.run('gdal:rastercalculator', params)
- ImportError: cannot import name 'processing' from 'qgis.core' (C:\\PROGRA~1\\QGIS40~1.2\\apps\\qgis\\python\\qgis\\core\\__init__.py)","new_files":[],"script":"pyqgis_1.py
  Fix: from qgis.core import QgsRasterLayer, QgsProject | import processing | params = { | 'INPUT_A': elevation_layer, | 'BAND_A': 1, | 'FORMULA': '(A > 500) * 1',
- ValueError: that element is gone (its window changed) - list them again with gui_elements
  Fix: {"element": 1}
- ImportError: PyQt5 classes cannot be imported in a QGIS build based on Qt6.
  Fix: from qgis.PyQt.QtGui import QColor | print('Raster layer styled with continuous color ramp.')"
- _core.QgsProcessingException: Could not create layer output/flood_zone_buffer_500.gpkg: Creation of data source failed (OGR error: A file system object called 'output/flood_zone_buffer_500.gpkg' alrea
  Fix: path = 'output/at_risk_arden_500.gpkg' | if os.path.exists(path): | print('File exists:', path) | print('File does not exist:', path)
- PermissionError: [WinError 32] The process cannot access the file because it is being used by another process: 'output/flood_zone_buffer_500.gpkg'","new_files":[],"script":"pyqgis_18.py
  Fix: path = 'output/at_risk_arden_500.gpkg' | if os.path.exists(path): | print('File exists:', path) | else: | print('File does not exist:', path)
- _core.QgsProcessingException: Error: Algorithm native:summarizeattributes not found","new_files":[],"script":"pyqgis_1.py
  Fix: processing.run('native:aggregate', {'INPUT':'data/cities.gpkg', 'GROUP_BY':'region', 'AGGREGATES':[{'aggregate': 'sum', 'input': 'population', 'name': 'pop_total', 'type': 4}], 'OUTPUT':'output/regions_population.gpkg'})

<!-- reference -->

# Why these rules

Everything here is for whoever maintains this skill. It is kept in the file
and never sent to an agent.

## Where the numbers come from

`evals/harness/run_ab.py` measures PyQGIS code generation against these exact
failures, and `evals/harness/qgis_server.py` wraps the same composition as
typed operations. The server's docstring names the three bugs this skill
covers - a guessed layer name, area in square degrees, and teardown ordered
before the work - because all three were shipped before they were written
down.

## The one that is not a knowledge gap

A model asked whether area in EPSG:4326 is in square degrees will say yes. It
knows. It writes `geometry().area()` anyway, because the knowledge is not
attached to the moment of writing that line. That is the argument for a check
rather than a paragraph: the paragraph was already true and already ignored.
