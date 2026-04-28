# ArcPy-Grid-Code
# Run in ArcgisPro to created grids in a locational extent
import arcpy 
  
arcpy.env.workspace = r"X:\GISC\Community\ParkRidge\Project\20250326_JulieAreas\JulieConstructionAreas\Default.gdb" 
arcpy.env.overwriteOutput = True 
polygon_fc = "ConstructionaArea" 
 fishnet_out = "GridIndex_500ft" 
clipped_grid = "GridIndex_500ft_Clipped" 
cell_width = 500 
cell_height = 500 
desc = arcpy.Describe(polygon_fc) 
ext = desc.extent 
  
origin_coord = f"{ext.XMin} {ext.YMin}" 
y_axis_coord = f"{ext.XMin} {ext.YMin + 10}"   
corner_coord = f"{ext.XMax} {ext.YMax}" 
  
arcpy.management.CreateFishnet( 
    out_feature_class=fishnet_out, 
    origin_coord=origin_coord, 
    y_axis_coord=y_axis_coord, 
    cell_width=cell_width, 
    cell_height=cell_height, 
    number_rows="", 
    number_columns="", 
    corner_coord=corner_coord, 
	labels="NO_LABELS", 
	template=polygon_fc, 
    geometry_type="POLYGON" 
) 
  
arcpy.analysis.Clip(fishnet_out, polygon_fc, clipped_grid) 
  
# Add field to populate grid numbers 
field_list = [f.name for f in arcpy.ListFields(clipped_grid)] 
if "GridNumber" not in field_list: 
    arcpy.management.AddField(clipped_grid, "GridNumber", "LONG") 
	
  
# Populate sequential numbers for grids 
with arcpy.da.UpdateCursor(clipped_grid, ["GridNumber"]) as cursor: 
    i = 1 
	for row in cursor: 
    	row[0] = i 
        cursor.updateRow(row) 
        i += 1 
  
 

