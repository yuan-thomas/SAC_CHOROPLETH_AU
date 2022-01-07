# SAC_CHOROPLETH_AU
SAP Analytics Cloud Choropleth Map HANA content: Australia->States->Local Government Areas->Postcodes->Suburbs

## Shapefile source
Australia(SAC Default)
States(SAC Default)
Local Government Area(ABS)
Postcodes(ABS)
Suburbs(ABS)

## Prerequisites
- SAP HANA 2.0 SPS05+ or SAP BTP/HANA Cloud
- SAP Analytics Live HANA Connection Configured

## Download and Prepare ESRI Shapefiles from Australia Statistics Bureau (ABS)
### Download files
URL: https://www.abs.gov.au/websitedbs/D3310114.nsf/home/Digital+Boundaries
Or Google ABS Digital Boundary

Choose:
Non ABS Structures
|- Local Government Areas
|- Postal Areas
|- Suburbs and Localities

Use GDA94 version

### (Optional but recommended for performance) Reduce Shapefile resolution
#### 1. Goto https://mapshaper.org/
#### 2. Upload downloaded .zip file from previous section
#### 3. Choose Simplify and tick 'prevent shape removal'
#### 4. Export

## Deployment instruction (to XS Advanced or SAP BTP)
#### 1. Install SAC Geo Spatial Content (if haven't done)
Reference: https://blogs.sap.com/2021/03/04/using-choropleth-layers-with-hana-cloud-and-sap-analytics-cloud/
#### 2. Directly deploy project .mtar file or build in WebIDE/Business Application Studio
#### 3. Assign Access Roles of 3x Applications (FPA_SPATIAL_DATA, SAP_FPA_SPATIAL_CUSTOM_REGIONS and SAC_CHOROPLETH_AU), as well as SAC_CHOROPLETH_AU::SchemaFullAccess to a DB User
#### 4. Log in via SAP HANA Cockpit https://hana-cockpit.cfapps.ap10.hana.ondemand.com/
#### 5. Create Public Synonyms for metadata tables
```tsql
CREATE PUBLIC synonym SAC_CHOROPLETH_DATA for "FPA_SPATIAL_DATA"."FPA_SPATIAL_DATA.choropleth::CHOROPLETH";
CREATE PUBLIC synonym SAC_CHOROPLETH_HIER for "SAP_FPA_SPATIAL_CUSTOM_REGIONS"."sap.fpa.services.spatial.choropleth::CHOROPLETH_CUSTOM_HIERARCHY";
```
#### 6. Import 3x Shapefiles into Schema SAC_CHOROPLETH_AU (EPSG:4326)
#### 7. Run following SQL Script to transfer imported data to final tables and format (table names may vary depending on the version downloaded from ABS)
```tsql
TRUNCATE TABLE "SAC_CHOROPLETH_AU"."LGA_AUST_GDA94";
TRUNCATE TABLE "SAC_CHOROPLETH_AU"."POA_AUST_GDA94";
TRUNCATE TABLE "SAC_CHOROPLETH_AU"."SAL_AUST_GDA94";

INSERT INTO "SAC_CHOROPLETH_AU"."LGA_AUST_GDA94" (SELECT *, SHAPE.ST_TRANSFORM(3857) AS SHAPE_3857, NULL AS SHAPEPOINT FROM "SAC_CHOROPLETH_AU"."LGA_2021_AUST_GDA94");
INSERT INTO "SAC_CHOROPLETH_AU"."POA_AUST_GDA94" (SELECT *, SHAPE.ST_TRANSFORM(3857) AS SHAPE_3857, NULL AS SHAPEPOINT FROM "SAC_CHOROPLETH_AU"."POA_2021_AUST_GDA94");
INSERT INTO "SAC_CHOROPLETH_AU"."SAL_AUST_GDA94" (SELECT *, SHAPE.ST_TRANSFORM(3857) AS SHAPE_3857, NULL AS SHAPEPOINT FROM "SAC_CHOROPLETH_AU"."SAL_2021_AUST_GDA94");

UPDATE "SAC_CHOROPLETH_AU"."LGA_AUST_GDA94" SET SHAPEPOINT = SHAPE_3857.ST_CENTROID();
UPDATE "SAC_CHOROPLETH_AU"."POA_AUST_GDA94" SET SHAPEPOINT = SHAPE_3857.ST_CENTROID();
UPDATE "SAC_CHOROPLETH_AU"."SAL_AUST_GDA94" SET SHAPEPOINT = SHAPE_3857.ST_CENTROID();
```
#### 8. Run following SQL Script to register the Hierarchy in metadata table
```tsql
INSERT INTO "SAP_FPA_SPATIAL_CUSTOM_REGIONS"."sap.fpa.services.spatial::custom_hierarchy.CHOROPLETH_CUSTOM_HIERARCHY" (SELECT * FROM "SAC_CHOROPLETH_AU"."CHOROPLETH_CUSTOM_HIERARCHY")
```
#### 9. Complete. Geographic hierarchy 'Australia' is ready for use in SAC GeoMap
