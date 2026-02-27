# Methodology: Project 7 — Salmon Stream Network Analysis on Vancouver Island

This document contains the detailed step-by-step methodology followed in Project 7.

---

## Task 1: Create Stream Segments
- Exported the `vancouver_island_dem` PostGIS raster from QGIS as a GeoTiff
- Clipped the DEM to the Vancouver Island boundary in ArcGIS Pro using the Clip Raster tool
- Ran the Flow Direction tool on the clipped DEM with edge cells forced to flow outward
- Ran the Flow Accumulation tool on the flow direction raster, outputting an integer raster
- Used the Con tool to assign a value of 1 to cells with flow accumulation ≥ 1,000, defining the stream network
- Generated stream links using the Stream Link tool (unique segment IDs)
- Assigned stream order using the Stream Order tool
- Converted the stream link raster to polyline features using Stream to Feature with simplified polylines toggled off

---

## Task 2: Create Routes and Apply Topology
- Created a feature dataset called `stream_routes_dams` in the project geodatabase, projected to NAD 1983 BC Environment Albers (EPSG:3005)
- Converted stream features to routes using the Create Routes tool with `arcid` as the route ID field, saved as `vancouver_island_stream_routes`
- Imported the `dams` layer from the UBC PostgreSQL server into the feature dataset
- Created a geodatabase topology with a rule that stream routes must not intersect dam polylines
- Validated the topology and exported the point errors as `stream_dam_junctions`
- Applied a -25 m negative buffer to the Vancouver Island boundary polygon to create a nibbled edge layer
- Converted the nibbled boundary to a polyline using Polygon to Line
- Updated the topology with a new rule that stream routes must not intersect the nibbled boundary
- Validated the topology again and exported point errors as `stream_ocean_junctions`

---

## Task 3: Record Stream Attributes with Linear Referencing
- Calculated percent rise slope from the clipped DEM
- Extracted slope values along stream locations only using Extract by Mask with the stream links raster
- Reclassified the slope raster: values < 20% assigned 1 (fish-bearing), all others assigned 0
- Converted the reclassified slope raster to line features using Stream to Feature
- Used Locate Features Along Routes to overlay gradient information with stream routes, producing a gradient event table
- Added a GRADIENT field and calculated it from `grid_code`
- Repeated the process for the stream order raster, producing a stream order event table with a SORDER field

---

## Task 4: Trace the Network
- Created a trace network (`stream_trace_network`) using the stream routes, dam junctions, and ocean junctions, with stream routes set to Complex Edge connectivity
- Enabled and validated network topology
- Ran the Trace tool starting from ocean junctions with dam junctions set as barriers, using Connected trace type
- Added a REACHED field to the stream routes attribute table and assigned 1 to selected (reachable) routes and 0 to all others
- Symbolized routes by the REACHED field to distinguish accessible from dam-impeded segments

---

## Task 5: Overlay and Query the Network
- Used Overlay Route Events to intersect the gradient and stream order event tables, producing `stream_route_overlay`
- Joined the REACHED field from stream routes to the overlay event table using Join Field
- Converted the overlay table to a feature layer using Make Route Event Layer
- Queried the event layer for segments that are ocean-reachable, 1st or 2nd order, and have gradient < 20% — exported as `accessible_salmon_habitat`
- Repeated the query for unreachable segments meeting the same habitat criteria — exported as `inaccessible_salmon_habitat`
- Clipped both habitat layers to the selected conservation unit for final mapping

---

*Back to [README](README_project7.md)*
