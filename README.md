# AUTOCAD
// Set units
@settings(defaultLengthUnit = in, kclVersion = 1.0)

// Define parameters for the manual grain collector
sigmaAllow = 35000 // psi (6061-T6 aluminum)
width = 6.0 // Width of the collector frame
p = 400 // Force applied to grain intake - lbs
fos = 1.5 // Factor of safety
collectorFrontWidth = 8.0 // Front intake width
collectorChamberHeight = 12.0 // Chamber height
wheelDiameter = 10.0 // Diameter of side wheels
chainDriveLength = 18.0 // Length of chain drive connecting wheels to chamber
baggingSystemOffset = 6.0 // Distance from the chamber exit to the bagging system

// Mounting parameters
collectorMountLength = 8.0
wallMountLength = 3.5
collectorMountingHoleDiameter = .75

// Calculated parameters
moment = collectorFrontWidth * p // Assuming force applied at intake
thickness = sqrt(moment * fos * 6 / (sigmaAllow * width)) // Required thickness for durability
bendRadius = 0.5
extBendRadius = bendRadius + thickness
filletRadius = .5
collectorMountingHolePlacementOffset = collectorMountingHoleDiameter * 1.5

// Add checks to ensure structural integrity
assert(collectorMountLength, isGreaterThanOrEqual = collectorMountingHoleDiameter * 3, error = "Holes not possible. Adjust hole diameter or increase collectorMountLength")
assert(width, isGreaterThanOrEqual = collectorMountingHoleDiameter * 5.5, error = "Holes not possible. Adjust hole diameter or increase width")

// Create the body of the collector
collectorBody = startSketchOn(XZ) 
    |> startProfile(at = [0, 0]) 
    |> xLine(length = collectorMountLength - thickness, tag = $seg01) 
    |> yLine(length = collectorChamberHeight, tag = $seg02) 
    |> xLine(length = -collectorFrontWidth, tag = $seg03) 
    |> yLine(length = -wallMountLength, tag = $seg04) 
    |> xLine(length = thickness, tag = $seg05) 
    |> line(endAbsolute = [profileStartX(%), profileStartY(%)], tag = $seg06) 
    |> close() 
    |> extrude(%, length = width)

// Add mounting holes for stability
collectorMountingHoles = startSketchOn(collectorBody, face = seg03) 
    |> circle(center = [-(bendRadius + collectorMountingHolePlacementOffset), collectorMountingHolePlacementOffset], radius = collectorMountingHoleDiameter / 2)
    |> patternLinear2d(instances = 2, distance = -(extBendRadius + collectorMountingHolePlacementOffset) + collectorMountLength - collectorMountingHolePlacementOffset, axis = [-1, 0])
    |> extrude(%, length = -thickness - .01)

// Apply bends and fillets
fillet(collectorBody, radius = extBendRadius, tags = [getNextAdjacentEdge(seg03)])
fillet(collectorBody, radius = bendRadius, tags = [getNextAdjacentEdge(seg06)])

// Apply corner fillets
fillet(collectorBody, radius = filletRadius, tags = [seg02, getOppositeEdge(seg02), seg05, getOppositeEdge(seg05)])
