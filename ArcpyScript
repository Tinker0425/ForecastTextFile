"""
Name: Test_Fcst_Text                                                          
Created on: 01/16/23                                                             
Created By: Kayla Tinker
"""

import os
import math
import arcpy.mapping
import shutil
import arcpy
import datetime
import pytz

inFeature = "ARCTIC"
#U:/My Documents/ArcGIS/Default1.gdb/
outFeatureClass_Dissolve = "Text_Fcst_ARCTIC_Dissolve"
dissolveFields = ["POLY_TYPE"]
arcpy.Dissolve_management(inFeature, outFeatureClass_Dissolve, dissolveFields,"","SINGLE_PART","DISSOLVE_LINES")

#U:/My Documents/ArcGIS/Default1.gdb/
outFeatureClass_ICE = "Text_Fcst_ICE_Only"
definition_query = "POLY_TYPE = 'I'"

arcpy.MakeFeatureLayer_management(outFeatureClass_Dissolve, "Text_Fcst_ARCTIC_Dissolve_lyr")
arcpy.SelectLayerByAttribute_management("Text_Fcst_ARCTIC_Dissolve_lyr", "NEW_SELECTION", definition_query)
arcpy.CopyFeatures_management("Text_Fcst_ARCTIC_Dissolve_lyr", outFeatureClass_ICE)



###If zone is covered 100% print ... COMPLETELY_WITHIN or WITHIN
###If zone is covered otherwise print...WITHIN, no INTERSECT?
###The issue is the lat/lon used, so I need to adjust that

#All U:/My Documents/ArcGIS/Default1.gdb/
inFeature_Offshore = "AKOffshoreZones2016_Geography"
inFeature_Marine = "AKMarineZones2023_Geography"
outFeatureClass_Zones = "Zones2016_Geography"
outFeatureClass_Overlap = "Text_Fcst_Overlap"


### I think I need to merge offshore and marine zones, because the only way we separate them in the text is by ice covered, ice, or ice free
arcpy.Merge_management([inFeature_Marine, inFeature_Offshore], outFeatureClass_Zones)

arcpy.MakeFeatureLayer_management(outFeatureClass_Zones, "Text_Fcst_Zones2016_lyr")
arcpy.SelectLayerByLocation_management("Text_Fcst_Zones2016_lyr", "INTERSECT", outFeatureClass_ICE, "", "NEW_SELECTION")
arcpy.CopyFeatures_management("Text_Fcst_Zones2016_lyr", outFeatureClass_Overlap)


#Try for Ice covered
outFeatureClass_Covered = "Text_Fcst_Covered"

arcpy.MakeFeatureLayer_management(outFeatureClass_Overlap, "Text_Fcst_Overlap_lyr")
arcpy.SelectLayerByLocation_management("Text_Fcst_Overlap_lyr", "COMPLETELY_WITHIN", outFeatureClass_ICE)
arcpy.CopyFeatures_management("Text_Fcst_Overlap_lyr", outFeatureClass_Covered)




### Now I want to print those unique values to a text file
input_layer = outFeatureClass_Overlap
output_file = "U:/Zones_TextFile.txt"
field_values = []
ID = []
Name = []

beaufort_zones = ""
chukchi_zones = ""
cook_zones = ""
bering_zones = ""


for row in arcpy.da.SearchCursor(input_layer, ["ID", "Name"]):
    for field in row:
        field_values.append(field)
    zone_num = int(str(field_values[0][3:6]))

    if zone_num < 750 and zone_num > 700:
        cook_zones += (str(field_values[0]) + "-" + str(field_values[1]) + "-" + "\n")
    elif (zone_num < 816 and zone_num > 812) or (zone_num < 862 and zone_num > 858) or (zone_num < 511 and zone_num > 500):
        beaufort_zones += (str(field_values[0]) + "-" + str(field_values[1]) + "-" + "\n")
    elif (zone_num < 813 and zone_num > 806) or (zone_num < 859 and zone_num > 854) or (zone_num == 500):
        chukchi_zones += (str(field_values[0]) + "-" + str(field_values[1]) + "-" + "\n")
    else:
        bering_zones += (str(field_values[0]) + "-" + str(field_values[1]) + "-" + "\n")

    field_values = []

### Attempt to sort Beaufort Zones in order. This works 2/19/23
###
cook_order = sorted(cook_zones.split("\n"))
beaufort_order = sorted(beaufort_zones.split("\n"))
chukchi_order = sorted(chukchi_zones.split("\n"))
bering_order = sorted(bering_zones.split("\n"))

ak_tz = pytz.timezone("America/Anchorage")
now = datetime.datetime.now(ak_tz)
five_days = now + datetime.timedelta(days=5)
five_time = five_days.strftime("%A %d %B %Y")
current_time = now.strftime("%I%M %p %Z %A %d %B %Y") 



### Attempting to add in ice edge points! 2/19/23
### POLY_TYPE = 'I' AND CT = '01' AND Shape_Area > 1000000000
### Remove the NAME portion if you do want the entire ice edge

outFeatureClass_ICE_EDGE = "Text_Fcst_ICE_EDGE_Only"
definition_query = "POLY_TYPE = 'I' AND CT = '01' AND Shape_Area > 1000000000 AND NAME <> 'Cook Inlet'"

arcpy.MakeFeatureLayer_management(inFeature, "Text_Fcst_ARCTIC_lyr")
arcpy.SelectLayerByAttribute_management("Text_Fcst_ARCTIC_lyr", "NEW_SELECTION", definition_query)
arcpy.CopyFeatures_management("Text_Fcst_ARCTIC_lyr", outFeatureClass_ICE_EDGE)

# Dissolve lines to create one polygon (main ice edge)
outFeaterClass_ICE_EDGE_Dissolve = "Text_Fcst_ICE_EDGE_MERGE_Only"
dissolveFields = ["POLY_TYPE"]
arcpy.Dissolve_management(outFeatureClass_ICE_EDGE, outFeaterClass_ICE_EDGE_Dissolve, dissolveFields,"","SINGLE_PART","DISSOLVE_LINES")

### Cook Inlet Ice Edge

outFeatureClass_ICE_EDGE_COOK = "Text_Fcst_ICE_EDGE_COOK_Only"
definition_query = "POLY_TYPE = 'I' AND CT = '01' AND Shape_Area > 1000000000 AND NAME = 'Cook Inlet'"

arcpy.MakeFeatureLayer_management(inFeature, "Text_Fcst_ARCTIC_lyr")
arcpy.SelectLayerByAttribute_management("Text_Fcst_ARCTIC_lyr", "NEW_SELECTION", definition_query)
arcpy.CopyFeatures_management("Text_Fcst_ARCTIC_lyr", outFeatureClass_ICE_EDGE_COOK)




### CREATE TEXT FILE

with open(output_file, "w") as f:
    f.write("Sea Ice Forecast for Western and Arctic Alaskan Coastal Waters\n")
    f.write(str(current_time) + os.linesep) 

    f.write("FORECAST VALID..." + five_time + os.linesep)
    f.write("ANALYSIS CONFIDENCE...Pick one: High; High to Moderate; Moderate to High; Moderate; Moderate to Low; Low" + os.linesep)
    f.write("The main ice edge..." + os.linesep)
    f.write("From land-based points in Alaska, the main ice edge extends from..." + os.linesep)
    f.write("SYNOPSIS...\r" + os.linesep)

    f.write("-BEAUFORT SEA-" + os.linesep)
    f.write("\n".join(beaufort_order)) 
    f.write(os.linesep)
    f.write("FORECAST FOR THE BEAUFORT SEA (DAYS 1 through 5)...Forecast confidence is High. No significant changes are expected during the forecast period" + os.linesep + os.linesep)

    f.write("-CHUKCHI SEA-" + os.linesep)
    #f.write(chukchi_zones + "\n")
    f.write("\n".join(chukchi_order)) 
    f.write(os.linesep)
    f.write("FORECAST FOR THE CHUKCHI SEA (DAYS 1 through 5)...Forecast confidence is High. No significant changes are expected during the forecast period" + os.linesep + os.linesep)

    f.write("-BERING SEA-" + os.linesep)
    f.write("\n".join(bering_order)) 
    f.write(os.linesep)
    f.write("FORECAST FOR THE BERING SEA (DAYS 1 through 5)...Forecast confidence is High. No significant changes are expected during the forecast period" + os.linesep + os.linesep)

    f.write("-COOK INLET-" + os.linesep)
    f.write("ANALYSIS CONFIDENCE...Pick one: High; High to Moderate; Moderate to High; Moderate; Moderate to Low; Low" + os.linesep)
    f.write("The main ice edge in Cook Inlet extends from..." + os.linesep)
    f.write("From land-based points, the main ice edge in Cook Inlet extends from..." + os.linesep)
    f.write("SYNOPSIS..." + os.linesep)
    f.write("\n".join(cook_order)) 
    f.write(os.linesep)
    f.write("FORECAST FOR COOK INLET (DAYS 1 through 5)...Forecast confidence is High. No significant changes are expected during the forecast period" + os.linesep)
    f.write("Cook Inlet sea ice changes significantly with tides.\r\r")

    f.write("&&")









