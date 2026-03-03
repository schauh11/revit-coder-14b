# Benchmark: revit-coder-14b vs Claude Opus 4.6

All 40 questions and all responses, unedited.

## Methodology

40 Revit C# code generation questions, evaluated zero-shot. No system prompt,
no few-shot examples, no domain context. Just the raw question to each model.

- **revit-coder-14b**: Run locally via Ollama (F16 weights, no quantization)
- **Claude Opus 4.6**: Called via Anthropic API, prompted only with "Answer with code only. No explanation."

Both models received identical questions. Every question and every response is included here in full.

## Scoring

Each response is scored automatically on three axes (no human grading, no reference answers):

| Component | Weight | What it measures |
|-----------|--------|-----------------|
| Signal Presence | 40% | Fraction of expected domain keywords found (e.g., `FilteredElementCollector`, `Transaction`) |
| Code Quality | 30% | Domain-specific structural checks (namespaces, class structure, API patterns) |
| Completeness | 30% | Response length, code block formatting, error-free output |

**Composite = 0.4 x signal + 0.3 x quality + 0.3 x completeness**

## Overall Results

| Model | Avg Score | Wins | Win Rate |
|-------|-----------|------|----------|
| **revit-coder-14b** | **0.799** | **25** | **62.5%** |
| Claude Opus 4.6 | 0.793 | 15 | 37.5% |

## By Difficulty

| Difficulty | Count | 14B Avg | Claude Avg | 14B Wins | Claude Wins |
|-----------|-------|---------|-----------|----------|------------|
| Easy | 9 | 0.800 | 0.796 | 5 | 4 |
| Medium | 19 | 0.839 | 0.801 | 14 | 5 |
| Hard | 12 | 0.736 | 0.779 | 6 | 6 |

---

## All 40 Questions & Responses

### RC01 | Easy | Winner: Claude Opus 4.6 (-0.124)

**Question:** Write C# code for a FilteredElementCollector that retrieves all wall instances in the active document and returns them as a List<Element>. Write code only. No explanation.

**Expected signals:** `FilteredElementCollector`, `BuiltInCategory`, `OST_Walls`, `WhereElementIsNotElementType`, `ToElements`, `List`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.500 | 0.667 |
| Quality | 0.300 | 0.417 |
| Completeness | 0.625 | 0.700 |
| **Composite** | **0.477** | **0.602** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
var collector = new FilteredElementCollector(doc);
var walls = collector.OfClass(typeof(Wall)).ToElements().Cast<Element>().ToList();
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
FilteredElementCollector collector = new FilteredElementCollector(doc);
List<Element> walls = collector
    .OfClass(typeof(Wall))
    .WhereElementIsNotElementType()
    .ToElements()
    .ToList();
```

</details>

---

### RC02 | Medium | Winner: revit-coder-14b (+0.165)

**Question:** Write C# code to read the area of every room in the model using BuiltInParameter.ROOM_AREA, convert from internal units (square feet) to square meters, and return a Dictionary<string, double> mapping room name to area in m². Write code only. No explanation.

**Expected signals:** `FilteredElementCollector`, `BuiltInParameter`, `ROOM_AREA`, `AsDouble`, `UnitUtils`, `ConvertFromInternalUnits`, `Dictionary`, `BuiltInCategory.OST_Rooms`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 1.000 | 1.000 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.922** | **0.757** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class RoomAreaExtractor
{
    public static Dictionary<string, double> GetRoomAreas(Document doc)
    {
        var roomAreas = new Dictionary<string, double>();
        var collector = new FilteredElementCollector(doc).OfCategory(BuiltInCategory.OST_Rooms);

        foreach (var room in collector)
        {
            var roomElement = room as Room;
            if (roomElement != null)
            {
                var areaParam = roomElement.get_Parameter(BuiltInParameter.ROOM_AREA);
                if (areaParam != null && !areaParam.IsReadOnly)
                {
                    var areaInFeet = areaParam.AsDouble();
                    var areaInMeters = UnitUtils.ConvertFromInternalUnits(areaInFeet, DisplayUnitType.DUT_SQUARE_METERS);
                    roomAreas.Add(roomElement.Name, areaInMeters);
                }
            }
        }

        return roomAreas;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Dictionary<string, double> GetRoomAreas(Document doc)
{
    var result = new Dictionary<string, double>();
    var rooms = new FilteredElementCollector(doc)
        .OfCategory(BuiltInCategory.OST_Rooms)
        .WhereElementIsNotElementType()
        .ToElements();

    foreach (var elem in rooms)
    {
        Room room = elem as Room;
        if (room == null || room.Area <= 0) continue;

        Parameter areaParam = room.get_Parameter(BuiltInParameter.ROOM_AREA);
        double sqFt = areaParam.AsDouble();
        double sqM = UnitUtils.ConvertFromInternalUnits(sqFt, UnitTypeId.SquareMeters);

        string name = room.get_Parameter(BuiltInParameter.ROOM_NAME).AsString() ?? "Unnamed";
        if (result.ContainsKey(name))
            name = name + " (" + room.Id.IntegerValue + ")";
        result[name] = sqM;
    }
    return result;
}
```

</details>

---

### RC03 | Medium | Winner: revit-coder-14b (+0.050)

**Question:** Write a complete IExternalCommand implementation that creates a new wall along a line from (0,0,0) to (20,0,0) on Level 1. Include the Transaction, level lookup, and error handling with TaskDialog. Write code only. No explanation.

**Expected signals:** `IExternalCommand`, `Execute`, `Transaction`, `Wall.Create`, `Line.CreateBound`, `XYZ`, `Level`, `FilteredElementCollector`, `t.Start`, `t.Commit`, `TaskDialog`, `Result.Succeeded`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.833 | 0.833 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.881** | **0.831** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using System;
using System.Linq;

public class CreateWallCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        try
        {
            UIApplication uiApp = commandData.Application;
            UIDocument uidoc = uiApp.ActiveUIDocument;
            Document doc = uidoc.Document;

            // Find Level 1
            Level level1 = null;
            using (var collector = new FilteredElementCollector(doc))
            {
                level1 = collector.OfClass(typeof(Level))
                                .Cast<Level>()
                                .FirstOrDefault(l => l.Name == "Level 1");
            }

            if (level1 == null)
            {
                TaskDialog.Show("Error", "Could not find Level 1.");
                return Result.Failed;
            }

            // Define wall line
            XYZ startPoint = new XYZ(0, 0, level1.Elevation);
            XYZ endPoint = new XYZ(20, 0, level1.Elevation);
            Line wallLine = Line.CreateBound(startPoint, endPoint);

            // Create wall
            using (Transaction trans = new Transaction(doc, "Create Wall"))
            {
                trans.Start();

                Wall wall = Wall.Create(doc, wallLine, level1.Id, true);

                trans.Commit();
            }

            return Result.Succeeded;
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
[Transaction(TransactionMode.Manual)]
public class CreateWallCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIDocument uidoc = commandData.Application.ActiveUIDocument;
        Document doc = uidoc.Document;

        try
        {
            Level level1 = new FilteredElementCollector(doc)
                .OfClass(typeof(Level))
                .Cast<Level>()
                .FirstOrDefault(l => l.Name == "Level 1");

            if (level1 == null)
            {
                TaskDialog.Show("Error", "Level 1 not found.");
                return Result.Failed;
            }

            Line line = Line.CreateBound(new XYZ(0, 0, 0), new XYZ(20, 0, 0));

            using (Transaction tx = new Transaction(doc, "Create Wall"))
            {
                tx.Start();
                Wall wall = Wall.Create(doc, line, level1.Id, false);
                tx.Commit();
            }

            TaskDialog.Show("Success", "Wall created successfully.");
            return Result.Succeeded;
        }
        catch (Exception ex)
        {
            message = ex.Message;
            TaskDialog.Show("Error", ex.Message);
            return Result.Failed;
        }
    }
}
```

</details>

---

### RC04 | Hard | Winner: Claude Opus 4.6 (-0.148)

**Question:** Validate this Revit API code and list every bug in code comments, then show the corrected version:
```csharp
public void RenameRooms(Document doc)
{
    var rooms = new FilteredElementCollector(doc)
        .OfCategory(BuiltInCategory.OST_Rooms)
        .ToElements();
    foreach (var room in rooms)
    {
        room.get_Parameter(BuiltInParameter.ROOM_NAME).Set("Updated");
    }
}
```
Write code only. No explanation.

**Expected signals:** `Transaction`, `null`, `WhereElementIsNotElementType`, `IsReadOnly`, `t.Start`, `t.Commit`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.333 | 0.667 |
| Quality | 0.833 | 0.833 |
| Completeness | 0.775 | 0.825 |
| **Composite** | **0.616** | **0.764** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
public void RenameRooms(Document doc)
{
    using (var trans = new Transaction(doc, "Rename Rooms"))
    {
        trans.Start();
        var rooms = new FilteredElementCollector(doc)
            .OfCategory(BuiltInCategory.OST_Rooms)
            .ToElements();
        foreach (var room in rooms)
        {
            var param = room.get_Parameter(BuiltInParameter.ROOM_NAME);
            if (param != null)
            {
                param.Set("Updated");
            }
        }
        trans.Commit();
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
// BUGS:
// 1. No Transaction wrapping the parameter Set() calls - modifying parameters requires a Transaction
// 2. No null check on the parameter returned by get_Parameter - it could be null
// 3. No check if the parameter is read-only
// 4. Should use WhereElementIsNotElementType() to get room instances only
// 5. Should filter out unplaced rooms (Area <= 0)

public void RenameRooms(Document doc)
{
    var rooms = new FilteredElementCollector(doc)
        .OfCategory(BuiltInCategory.OST_Rooms)
        .WhereElementIsNotElementType()
        .ToElements();

    using (Transaction tx = new Transaction(doc, "Rename Rooms"))
    {
        tx.Start();
        foreach (var room in rooms)
        {
            if ((room as Room)?.Area <= 0) continue;

            Parameter nameParam = room.get_Parameter(BuiltInParameter.ROOM_NAME);
            if (nameParam != null && !nameParam.IsReadOnly)
            {
                nameParam.Set("Updated");
            }
        }
        tx.Commit();
    }
}
```

</details>

---

### RC05 | Medium | Winner: revit-coder-14b (+0.200)

**Question:** Write C# code to collect all floor instances in the model, group them by floor type name, and return a Dictionary<string, int> with the count of each type. Write code only. No explanation.

**Expected signals:** `FilteredElementCollector`, `BuiltInCategory.OST_Floors`, `WhereElementIsNotElementType`, `GetTypeId`, `doc.GetElement`, `Name`, `Dictionary`, `GroupBy`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.625 | 0.500 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.757** | **0.557** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class FloorCollector
{
    public static Dictionary<string, int> CollectFloorTypes(Document doc)
    {
        var floorTypes = new Dictionary<string, int>();

        var floors = new FilteredElementCollector(doc)
            .OfClass(typeof(Floor))
            .ToElements();

        foreach (var floor in floors)
        {
            var floorType = floor.GetTypeId();
            if (floorType != ElementId.InvalidElementId)
            {
                var floorTypeName = doc.GetElement(floorType).Name;
                if (floorTypes.ContainsKey(floorTypeName))
                {
                    floorTypes[floorTypeName]++;
                }
                else
                {
                    floorTypes.Add(floorTypeName, 1);
                }
            }
        }

        return floorTypes;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Dictionary<string, int> GetFloorCountsByType(Document doc)
{
    var floors = new FilteredElementCollector(doc)
        .OfClass(typeof(Floor))
        .WhereElementIsNotElementType()
        .Cast<Floor>();

    var result = new Dictionary<string, int>();
    foreach (var floor in floors)
    {
        string typeName = floor.FloorType.Name;
        if (result.ContainsKey(typeName))
            result[typeName]++;
        else
            result[typeName] = 1;
    }
    return result;
}
```

</details>

---

### RC06 | Medium | Winner: revit-coder-14b (+0.050)

**Question:** Write C# code to place a family instance at a given XYZ point. Load the family from a .rfa file path if not already loaded, find the first available FamilySymbol, activate it, and place it on Level 1 inside a Transaction. Write code only. No explanation.

**Expected signals:** `FamilySymbol`, `Activate`, `NewFamilyInstance`, `doc.LoadFamily`, `Transaction`, `Level`, `FilteredElementCollector`, `t.Start`, `t.Commit`, `StructuralType`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.800 | 0.800 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.868** | **0.818** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using System;
using System.Linq;

public class PlaceFamilyInstanceCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        try
        {
            UIApplication uiApp = commandData.Application;
            UIDocument uiDoc = uiApp.ActiveUIDocument;
            Document doc = uiDoc.Document;

            if (doc.IsReadOnly)
            {
                message = "Document is read-only.";
                return Result.Failed;
            }

            // Get the family path from the user
            string familyPath = GetFamilyPathFromUser();

            if (string.IsNullOrEmpty(familyPath))
            {
                message = "Family path is required.";
                return Result.Failed;
            }

            // Load the family if not already loaded
            FamilySymbol familySymbol = GetFamilySymbol(doc, familyPath);

            if (familySymbol == null)
            {
                message = "Failed to load family symbol.";
                return Result.Failed;
            }

            // Get the point from the user
            XYZ point = GetPointFromUser();

            if (point == null)
            {
                message = "Point is required.";
                return Result.Failed;
            }

            // Get Level 1
            Level level1 = GetLevel1(doc);

            if (level1 == null)
            {
                message = "Level 1 not found.";
                return Result.Failed;
            }

            // Place the family instance
            using (Transaction transaction = new Transaction(doc, "Place Family Instance"))
            {
                transaction.Start();

                // Place the family instance
                FamilyInstance familyInstance = doc.Create.NewFamilyInstance(point, familySymbol, level1, StructuralType.NonStructural);

                transaction.Commit();
            }

            return Result.Succeeded;
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }
    }

    private string GetFamilyPathFromUser()
    {
        // Implement your logic to get the family path from the user
        return @"C:\path\to\your\family.rfa";
    }

    private XYZ GetPointFromUser()
    {
        // Implement your logic to get the point from the user
        return new XYZ(0, 0, 0);
    }

    private Level GetLevel1(Document doc)
    {
        // Get Level 1
        FilteredElementCollector collector = new FilteredElementCollector(doc);
        collector.OfClass(typeof(Level));
        Level level1 = collector.Cast<Level>().FirstOrDefault(l => l.Name == "Level 1");

        return level1;
    }

    private FamilySymbol GetFamilySymbol(Document doc, string familyPath)
    {
        // Check if the family is already loaded
        FamilySymbol familySymbol = GetLoadedFamilySymbol(doc, familyPath);

        if (familySymbol != null)
        {
            return familySymbol;
        }

        // Load the family
        Family family = LoadFamily(doc, familyPath);

        if (family == null)
        {
            return null;
        }

        // Get the first available family symbol
        familySymbol = GetFirstAvailableFamilySymbol(family);

        if (familySymbol == null)
        {
            return null;
        }

        // Activate the family symbol
        familySymbol.Activate();

        return familySymbol;
    }

    private FamilySymbol GetLoadedFamilySymbol(Document doc, string familyPath)
    {
        // Check if the family is already loaded
        FilteredElementCollector collector = new FilteredElementCollector(doc);
        collector.OfClass(typeof(Family));
        Family loadedFamily = collector.Cast<Family>().FirstOrDefault(f => f.GetFamilyName() == System.IO.Path.GetFileNameWithoutExtension(familyPath));

        if (loadedFamily != null)
        {
            // Get the first available family symbol
            FamilySymbol familySymbol = GetFirstAvailableFamilySymbol(loadedFamily);

            if (familySymbol != null)
            {
                return familySymbol;
            }
        }

        return null;
    }

    private Family LoadFamily(Document doc, string familyPath)
    {
        // Load the family
        Family family = doc.LoadFamily(familyPath, new FamilyLoadOptions());

        if (family == null)
        {
            return null;
        }

        return family;
    }

    private FamilySymbol GetFirstAvailableFamilySymbol(Family family)
    {
        // Get the first available family symbol
        FamilySymbol familySymbol = family.GetFamilySymbolIds().FirstOrDefault();

        if (familySymbol == null)
        {
            return null;
        }

        return familySymbol;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void PlaceFamilyInstance(Document doc, string rfaPath, XYZ point)
{
    Family family = null;
    FamilySymbol symbol = null;

    // Check if already loaded
    var families = new FilteredElementCollector(doc)
        .OfClass(typeof(Family))
        .Cast<Family>();

    string familyName = Path.GetFileNameWithoutExtension(rfaPath);
    family = families.FirstOrDefault(f => f.Name == familyName);

    using (Transaction tx = new Transaction(doc, "Place Family Instance"))
    {
        tx.Start();

        if (family == null)
        {
            if (!doc.LoadFamily(rfaPath, out family))
                throw new Exception("Failed to load family.");
        }

        ISet<ElementId> symbolIds = family.GetFamilySymbolIds();
        if (symbolIds.Count == 0)
            throw new Exception("No symbols found in family.");

        symbol = doc.GetElement(symbolIds.First()) as FamilySymbol;

        if (!symbol.IsActive)
            symbol.Activate();

        Level level1 = new FilteredElementCollector(doc)
            .OfClass(typeof(Level))
            .Cast<Level>()
            .FirstOrDefault(l => l.Name == "Level 1");

        if (level1 == null)
            throw new Exception("Level 1 not found.");

        doc.Create.NewFamilyInstance(point, symbol, level1, StructuralType.NonStructural);

        tx.Commit();
    }
}
```

</details>

---

### RC07 | Hard | Winner: revit-coder-14b (+0.125)

**Question:** Write C# code to extract the solid geometry from a wall element using the GeometryElement API. Get the geometry with ComputeReferences, iterate through GeometryObject instances, handle GeometryInstance by getting instance geometry, and return the first Solid with non-zero volume. Write code only. No explanation.

**Expected signals:** `GeometryElement`, `Options`, `ComputeReferences`, `GetGeometryObjectFromReference`, `GeometryInstance`, `GetInstanceGeometry`, `Solid`, `Volume`, `OfType`, `DetailLevel`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.700 | 0.800 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.802** | **0.677** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;

public static class GeometryExtractor
{
    public static Solid GetSolidGeometry(Wall wall)
    {
        Options options = new Options();
        options.ComputeReferences = true;
        options.DetailLevel = ViewDetailLevel.Undefined;
        options.IncludeNonVisibleObjects = true;
        options.SetDetailLevel(ViewDetailLevel.Undefined);

        GeometryElement geometryElement = wall.get_Geometry(options);
        if (geometryElement == null) return null;

        foreach (GeometryObject geometryObject in geometryElement)
        {
            if (geometryObject is Solid solid)
            {
                if (solid.Volume > 0) return solid;
            }
            else if (geometryObject is GeometryInstance geometryInstance)
            {
                GeometryElement instanceGeometry = geometryInstance.GetSymbolGeometry();
                if (instanceGeometry != null)
                {
                    foreach (GeometryObject instanceObject in instanceGeometry)
                    {
                        if (instanceObject is Solid instanceSolid)
                        {
                            if (instanceSolid.Volume > 0) return instanceSolid;
                        }
                    }
                }
            }
        }

        return null;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Solid GetWallSolid(Wall wall)
{
    Options options = new Options();
    options.ComputeReferences = true;
    options.DetailLevel = ViewDetailLevel.Fine;

    GeometryElement geomElem = wall.get_Geometry(options);

    foreach (GeometryObject geomObj in geomElem)
    {
        Solid solid = geomObj as Solid;
        if (solid != null && solid.Volume > 0)
            return solid;

        GeometryInstance geomInst = geomObj as GeometryInstance;
        if (geomInst != null)
        {
            GeometryElement instGeom = geomInst.GetInstanceGeometry();
            foreach (GeometryObject instObj in instGeom)
            {
                solid = instObj as Solid;
                if (solid != null && solid.Volume > 0)
                    return solid;
            }
        }
    }
    return null;
}
```

</details>

---

### RC08 | Medium | Winner: revit-coder-14b (+0.050)

**Question:** Write C# code to create a new ViewPlan for Level 1 using a ViewFamilyType, then create a new ViewSheet with a titleblock, and place the view on the sheet using Viewport.Create. All inside a Transaction. Write code only. No explanation.

**Expected signals:** `ViewPlan`, `ViewFamilyType`, `ViewSheet`, `Viewport`, `Create`, `Transaction`, `FilteredElementCollector`, `Level`, `TitleBlock`, `OfClass`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 1.000 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.948** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class ViewCreationCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        try
        {
            UIApplication uiApp = commandData.Application;
            UIDocument uidoc = uiApp.ActiveUIDocument;
            Document doc = uidoc.Document;

            using (Transaction trans = new Transaction(doc, "Create ViewPlan and ViewSheet"))
            {
                trans.Start();

                // Get Level 1
                FilteredElementCollector levelCollector = new FilteredElementCollector(doc)
                    .OfClass(typeof(Level))
                    .WhereElementIsNotElementType();

                Level level1 = levelCollector.FirstElement() as Level;

                // Get ViewFamilyType for Plan
                FilteredElementCollector viewFamilyTypeCollector = new FilteredElementCollector(doc)
                    .OfClass(typeof(ViewFamilyType))
                    .WhereElementIsElementType();

                ViewFamilyType planViewType = viewFamilyTypeCollector
                    .First(e => (e as ViewFamilyType).ViewFamily == ViewFamily.Plan) as ViewFamilyType;

                // Create ViewPlan
                ViewPlan viewPlan = ViewPlan.Create(doc, planViewType.Id, level1.Id);

                // Get ViewSheet and TitleBlock
                FilteredElementCollector sheetCollector = new FilteredElementCollector(doc)
                    .OfClass(typeof(ViewSheet))
                    .WhereElementIsNotElementType();

                ViewSheet viewSheet = sheetCollector.FirstElement() as ViewSheet;

                FilteredElementCollector titleBlockCollector = new FilteredElementCollector(doc)
                    .OfClass(typeof(FamilySymbol))
                    .WhereElementIsElementType();

                FamilySymbol titleBlock = titleBlockCollector
                    .First(e => (e as FamilySymbol).FamilyName.Contains("TitleBlock")) as FamilySymbol;

                // Place TitleBlock on ViewSheet
                XYZ titleBlockLocation = new XYZ(0.5, 0.5, 0);
                ElementId titleBlockId = doc.Create.NewFamilyInstance(titleBlockLocation, titleBlock, viewSheet, StructuralType.NonStructural);

                // Place ViewPlan on ViewSheet
                XYZ viewportLocation = new XYZ(0.5, 0.5, 0);
                Viewport viewport = Viewport.Create(doc, viewSheet.Id, viewPlan.Id, viewportLocation);

                trans.Commit();
            }

            return Result.Succeeded;
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateViewAndSheet(Document doc)
{
    using (Transaction tx = new Transaction(doc, "Create View and Sheet"))
    {
        tx.Start();

        // Find Level 1
        Level level = new FilteredElementCollector(doc)
            .OfClass(typeof(Level))
            .Cast<Level>()
            .FirstOrDefault(l => l.Name == "Level 1");

        // Find ViewFamilyType for FloorPlan
        ViewFamilyType vft = new FilteredElementCollector(doc)
            .OfClass(typeof(ViewFamilyType))
            .Cast<ViewFamilyType>()
            .FirstOrDefault(v => v.ViewFamily == ViewFamily.FloorPlan);

        ViewPlan viewPlan = ViewPlan.Create(doc, vft.Id, level.Id);
        viewPlan.Name = "New Floor Plan";

        // Find titleblock
        FamilySymbol titleblock = new FilteredElementCollector(doc)
            .OfClass(typeof(FamilySymbol))
            .OfCategory(BuiltInCategory.OST_TitleBlocks)
            .Cast<FamilySymbol>()
            .FirstOrDefault();

        if (!titleblock.IsActive)
            titleblock.Activate();

        ViewSheet sheet = ViewSheet.Create(doc, titleblock.Id);
        sheet.Name = "New Sheet";
        sheet.SheetNumber = "A-101";

        XYZ center = new XYZ(1.5, 1.0, 0);
        Viewport.Create(doc, sheet.Id, viewPlan.Id, center);

        tx.Commit();
    }
}
```

</details>

---

### RC09 | Easy | Winner: Claude Opus 4.6 (-0.032)

**Question:** Write C# code to get all doors in the model, then for each door read its width and height parameters using BuiltInParameter, and return a list of strings formatted as "DoorMark: W x H mm". Write code only. No explanation.

**Expected signals:** `FilteredElementCollector`, `BuiltInCategory.OST_Doors`, `WhereElementIsNotElementType`, `BuiltInParameter`, `DOOR_WIDTH`, `DOOR_HEIGHT`, `AsDouble`, `UnitUtils`, `List`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.444 | 1.000 |
| Quality | 1.000 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.725** | **0.757** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class DoorInfoExtractor
{
    public static List<string> GetDoorDimensions(Document doc)
    {
        var doorDimensions = new List<string>();

        using (var transactionGroup = new TransactionGroup(doc))
        {
            transactionGroup.Start("Get Door Dimensions");

            try
            {
                var doors = new FilteredElementCollector(doc)
                    .OfClass(typeof(FamilyInstance))
                    .OfCategory(BuiltInCategory.OST_Doors)
                    .ToElements();

                foreach (var door in doors)
                {
                    var doorMark = door.Parameter(BuiltInParameter.ALL_MODEL_MARK)?.AsString();
                    var width = door.Parameter(BuiltInParameter.INSTANCE_WIDTH_PARAM)?.AsValueString();
                    var height = door.Parameter(BuiltInParameter.INSTANCE_HEIGHT_PARAM)?.AsValueString();

                    if (!string.IsNullOrEmpty(doorMark) && !string.IsNullOrEmpty(width) && !string.IsNullOrEmpty(height))
                    {
                        doorDimensions.Add($"{doorMark}: {width} x {height} mm");
                    }
                }

                transactionGroup.Assimilate();
            }
            catch (Exception ex)
            {
                transactionGroup.RollBack();
                throw ex;
            }
        }

        return doorDimensions;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public List<string> GetDoorDimensions(Document doc)
{
    var result = new List<string>();

    var doors = new FilteredElementCollector(doc)
        .OfCategory(BuiltInCategory.OST_Doors)
        .WhereElementIsNotElementType()
        .ToElements();

    foreach (var door in doors)
    {
        string mark = door.get_Parameter(BuiltInParameter.ALL_MODEL_MARK)?.AsString() ?? "No Mark";

        double widthFt = door.get_Parameter(BuiltInParameter.DOOR_WIDTH)?.AsDouble() ?? 0;
        double heightFt = door.get_Parameter(BuiltInParameter.DOOR_HEIGHT)?.AsDouble() ?? 0;

        double widthMm = UnitUtils.ConvertFromInternalUnits(widthFt, UnitTypeId.Millimeters);
        double heightMm = UnitUtils.ConvertFromInternalUnits(heightFt, UnitTypeId.Millimeters);

        result.Add($"{mark}: {widthMm:F0} x {heightMm:F0} mm");
    }
    return result;
}
```

</details>

---

### RC10 | Hard | Winner: Claude Opus 4.6 (-0.299)

**Question:** Write C# code implementing an IUpdater that monitors when any wall's Location property changes, and automatically logs the wall's ElementId and new location curve endpoints to a text file. Include the registration in IExternalApplication.OnStartup. Write code only. No explanation.

**Expected signals:** `IUpdater`, `GetUpdaterId`, `Execute`, `GetAddedElementIds`, `GetModifiedElementIds`, `UpdaterRegistry`, `RegisterUpdater`, `AddTrigger`, `ChangeType`, `LocationCurve`, `IExternalApplication`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.000 | 0.909 |
| Quality | 0.750 | 0.583 |
| Completeness | 0.875 | 0.825 |
| **Composite** | **0.487** | **0.786** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System;
using System.IO;
using System.Linq;
using System.Text;
using System.Collections.Generic;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Analysis;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.MEPModel;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Electrical;
using

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public class WallLocationUpdater : IUpdater
{
    static AddInId _appId;
    static UpdaterId _updaterId;

    public WallLocationUpdater(AddInId id)
    {
        _appId = id;
        _updaterId = new UpdaterId(_appId, new Guid("D4B3C5A1-2E6F-4A8B-9C0D-1E2F3A4B5C6D"));
    }

    public void Execute(UpdaterData data)
    {
        Document doc = data.GetDocument();
        string logPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), "WallLocationLog.txt");

        foreach (ElementId id in data.GetModifiedElementIds())
        {
            Wall wall = doc.GetElement(id) as Wall;
            if (wall == null) continue;

            LocationCurve locCurve = wall.Location as LocationCurve;
            if (locCurve == null) continue;

            Curve curve = locCurve.Curve;
            XYZ start = curve.GetEndPoint(0);
            XYZ end = curve.GetEndPoint(1);

            string entry = $"{DateTime.Now}: Wall {id.IntegerValue} moved to ({start.X:F2},{start.Y:F2},{start.Z:F2}) - ({end.X:F2},{end.Y:F2},{end.Z:F2})";
            File.AppendAllText(logPath, entry + Environment.NewLine);
        }
    }

    public string GetAdditionalInformation() => "Monitors wall location changes";
    public ChangePriority GetChangePriority() => ChangePriority.FloorsRoofsStructuralWalls;
    public UpdaterId GetUpdaterId() => _updaterId;
    public string GetUpdaterName() => "WallLocationUpdater";
}

public class WallLocationApp : IExternalApplication
{
    public Result OnStartup(UIControlledApplication application)
    {
        WallLocationUpdater updater = new WallLocationUpdater(application.ActiveAddInId);
        UpdaterRegistry.RegisterUpdater(updater);

        ElementClassFilter wallFilter = new ElementClassFilter(typeof(Wall));
        UpdaterRegistry.AddTrigger(
            updater.GetUpdaterId(),
            wallFilter,
            Element.GetChangeTypeGeometry());

        return Result.Succeeded;
    }

    public Result OnShutdown(UIControlledApplication application)
    {
        WallLocationUpdater updater = new WallLocationUpdater(application.ActiveAddInId);
        UpdaterRegistry.UnregisterUpdater(updater.GetUpdaterId());
        return Result.Succeeded;
    }
}
```

</details>

---

### RC11 | Medium | Winner: revit-coder-14b (+0.100)

**Question:** Write C# code to collect all Revit links (RevitLinkInstance) in the current document, get each linked document, then count the total number of walls across all linked models. Return a Dictionary<string, int> mapping link name to wall count. Write code only. No explanation.

**Expected signals:** `RevitLinkInstance`, `FilteredElementCollector`, `GetLinkDocument`, `BuiltInCategory.OST_Walls`, `WhereElementIsNotElementType`, `Dictionary`, `Name`, `Count`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.750 | 0.875 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.807** | **0.708** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class LinkWallCounter
{
    public static Dictionary<string, int> CountWallsInLinks(Document doc)
    {
        var linkWallCounts = new Dictionary<string, int>();

        var linkInstances = new FilteredElementCollector(doc)
            .OfClass(typeof(RevitLinkInstance))
            .ToElements();

        foreach (var linkInstance in linkInstances)
        {
            var linkDoc = ((RevitLinkInstance)linkInstance).GetLinkDocument();
            if (linkDoc == null) continue;

            var wallCount = new FilteredElementCollector(linkDoc)
                .OfClass(typeof(Wall))
                .ToElementIds()
                .Count();

            linkWallCounts.Add(linkInstance.Name, wallCount);
        }

        return linkWallCounts;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Dictionary<string, int> GetLinkedWallCounts(Document doc)
{
    var result = new Dictionary<string, int>();

    var links = new FilteredElementCollector(doc)
        .OfClass(typeof(RevitLinkInstance))
        .Cast<RevitLinkInstance>();

    foreach (var link in links)
    {
        Document linkedDoc = link.GetLinkDocument();
        if (linkedDoc == null) continue;

        int wallCount = new FilteredElementCollector(linkedDoc)
            .OfClass(typeof(Wall))
            .WhereElementIsNotElementType()
            .GetElementCount();

        string linkName = link.Name;
        result[linkName] = wallCount;
    }
    return result;
}
```

</details>

---

### RC12 | Medium | Winner: revit-coder-14b (+0.050)

**Question:** Write C# code to move a selected element by a vector (10, 5, 0) feet using ElementTransformUtils.MoveElement. Get the currently selected element from UIDocument.Selection, validate it exists, and perform the move inside a Transaction. Write code only. No explanation.

**Expected signals:** `ElementTransformUtils`, `MoveElement`, `Selection`, `GetElementIds`, `XYZ`, `Transaction`, `t.Start`, `t.Commit`, `UIDocument`, `ElementId`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.800 | 0.800 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.853** | **0.802** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void MoveSelectedElementByVector()
{
    UIDocument uidoc = commandData.Application.ActiveUIDocument;
    Document doc = uidoc.Document;

    if (uidoc == null || doc == null)
    {
        return;
    }

    ElementId selectedElementId = uidoc.Selection.GetElementIds().FirstOrDefault();

    if (selectedElementId == null)
    {
        return;
    }

    Element selectedElement = doc.GetElement(selectedElementId);

    if (selectedElement == null)
    {
        return;
    }

    using (Transaction transaction = new Transaction(doc, "Move Element"))
    {
        transaction.Start();

        try
        {
            XYZ translationVector = new XYZ(10, 5, 0);
            ElementTransformUtils.MoveElement(doc, selectedElementId, translationVector);
            transaction.Commit();
        }
        catch
        {
            transaction.RollBack();
            throw;
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
[Transaction(TransactionMode.Manual)]
public class MoveElementCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIDocument uidoc = commandData.Application.ActiveUIDocument;
        Document doc = uidoc.Document;

        ICollection<ElementId> selectedIds = uidoc.Selection.GetElementIds();
        if (selectedIds.Count == 0)
        {
            TaskDialog.Show("Error", "No element selected.");
            return Result.Failed;
        }

        ElementId elemId = selectedIds.First();
        XYZ translation = new XYZ(10, 5, 0);

        using (Transaction tx = new Transaction(doc, "Move Element"))
        {
            tx.Start();
            ElementTransformUtils.MoveElement(doc, elemId, translation);
            tx.Commit();
        }

        return Result.Succeeded;
    }
}
```

</details>

---

### RC13 | Hard | Winner: Claude Opus 4.6 (-0.215)

**Question:** Write C# code to create a shared parameter from an external definition file, add it to the Walls category as an instance parameter under the Data group, and bind it to the project using BindingMap. Include creating the DefinitionFile and DefinitionGroup. Write code only. No explanation.

**Expected signals:** `DefinitionFile`, `ExternalDefinitionCreationOptions`, `DefinitionGroup`, `ExternalDefinition`, `CategorySet`, `InstanceBinding`, `BindingMap`, `Transaction`, `ParameterBindings`, `OpenSharedParameterFile`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.400 | 1.000 |
| Quality | 0.917 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.682** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.UI;

public class SharedParameterCreator
{
    public static void CreateSharedParameter(Document doc)
    {
        try
        {
            // Create a new DefinitionFile
            DefinitionFile definitionFile = DefinitionFile.Create("C:\\SharedParameters.txt");

            // Create a new DefinitionGroup
            DefinitionGroup definitionGroup = definitionFile.Groups.Create("Data");

            // Create a new Definition
            Definition definition = definitionGroup.Definitions.Create("MyParameter", StorageType.String);

            // Create a new ParameterBinding
            ParameterBinding parameterBinding = new ParameterBinding(new ElementId(BuiltInCategory.OST_Walls), definition);

            // Create a new BindingMap
            BindingMap bindingMap = new BindingMap();

            // Add the parameter binding to the BindingMap
            bindingMap.Insert(definition, parameterBinding);

            // Set the binding map to the document
            doc.ParameterBindings = bindingMap;
        }
        catch (Exception ex)
        {
            TaskDialog.Show("Error", ex.Message);
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateSharedParameter(Document doc, UIApplication uiApp)
{
    string sharedParamFile = Path.Combine(
        Environment.GetFolderPath(Environment.SpecialFolder.Desktop),
        "SharedParams.txt");

    if (!File.Exists(sharedParamFile))
        File.Create(sharedParamFile).Close();

    uiApp.Application.SharedParametersFilename = sharedParamFile;
    DefinitionFile defFile = uiApp.Application.OpenSharedParameterFile();

    DefinitionGroup group = defFile.Groups.get_Item("Custom Parameters")
        ?? defFile.Groups.Create("Custom Parameters");

    ExternalDefinitionCreationOptions options =
        new ExternalDefinitionCreationOptions("WallData", SpecTypeId.String.Text);
    options.Visible = true;

    Definition definition = group.Definitions.get_Item("WallData")
        ?? group.Definitions.Create(options);

    CategorySet catSet = uiApp.Application.Create.NewCategorySet();
    catSet.Insert(doc.Settings.Categories.get_Item(BuiltInCategory.OST_Walls));

    InstanceBinding binding = uiApp.Application.Create.NewInstanceBinding(catSet);

    using (Transaction tx = new Transaction(doc, "Add Shared Parameter"))
    {
        tx.Start();
        BindingMap bindingMap = doc.ParameterBindings;
        bindingMap.Insert(definition, binding, BuiltInParameterGroup.PG_DATA);
        tx.Commit();
    }
}
```

</details>

---

### RC14 | Medium | Winner: revit-coder-14b (+0.150)

**Question:** Write C# code to get all rooms in the model, extract each room's boundary segments using GetBoundarySegments with SpatialElementBoundaryOptions, and return a Dictionary<string, List<Curve>> mapping room name to its boundary curves. Write code only. No explanation.

**Expected signals:** `Room`, `GetBoundarySegments`, `SpatialElementBoundaryOptions`, `BoundarySegment`, `GetCurve`, `Curve`, `Dictionary`, `FilteredElementCollector`, `BuiltInCategory.OST_Rooms`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 1.000 | 1.000 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.922** | **0.772** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using System.Collections.Generic;

public class RoomBoundaryExtractor
{
    public static Dictionary<string, List<Curve>> GetRoomBoundaries(Document doc)
    {
        var roomBoundaries = new Dictionary<string, List<Curve>>();
        var options = new SpatialElementBoundaryOptions();
        options.StoreFreeBoundaryFaces = true;

        var rooms = new FilteredElementCollector(doc)
            .OfCategory(BuiltInCategory.OST_Rooms)
            .WhereElementIsNotElementType()
            .ToElements();

        foreach (var room in rooms)
        {
            var roomElement = room as Room;
            if (roomElement == null) continue;

            var boundarySegments = roomElement.GetBoundarySegments(options);
            var curves = new List<Curve>();

            foreach (var segment in boundarySegments)
            {
                foreach (var curve in segment.GetCurves())
                {
                    curves.Add(curve);
                }
            }

            roomBoundaries.Add(roomElement.Name, curves);
        }

        return roomBoundaries;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Dictionary<string, List<Curve>> GetRoomBoundaries(Document doc)
{
    var result = new Dictionary<string, List<Curve>>();

    var rooms = new FilteredElementCollector(doc)
        .OfCategory(BuiltInCategory.OST_Rooms)
        .WhereElementIsNotElementType()
        .Cast<Room>();

    SpatialElementBoundaryOptions options = new SpatialElementBoundaryOptions();
    options.SpatialElementBoundaryLocation = SpatialElementBoundaryLocation.Finish;

    foreach (Room room in rooms)
    {
        if (room.Area <= 0) continue;

        string name = room.get_Parameter(BuiltInParameter.ROOM_NAME).AsString() ?? "Unnamed";
        var curves = new List<Curve>();

        IList<IList<BoundarySegment>> boundaries = room.GetBoundarySegments(options);
        if (boundaries == null) continue;

        foreach (IList<BoundarySegment> loop in boundaries)
        {
            foreach (BoundarySegment seg in loop)
            {
                curves.Add(seg.GetCurve());
            }
        }

        if (result.ContainsKey(name))
            name = name + " (" + room.Id.IntegerValue + ")";
        result[name] = curves;
    }
    return result;
}
```

</details>

---

### RC15 | Medium | Winner: revit-coder-14b (+0.057)

**Question:** Write C# code to find all curtain walls in the model, access their CurtainGrid, iterate through panel and mullion ElementIds, and return a summary of panel count and mullion count per curtain wall. Write code only. No explanation.

**Expected signals:** `CurtainGrid`, `GetPanelIds`, `GetMullionIds`, `Wall`, `FilteredElementCollector`, `BuiltInCategory.OST_Walls`, `WallType`, `CurtainWallType`, `Dictionary`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.444 | 0.778 |
| Quality | 1.000 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.725** | **0.669** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;

public class CurtainWallAnalyzer
{
    public static Dictionary<ElementId, CurtainWallSummary> AnalyzeCurtainWalls(Document doc)
    {
        var curtainWalls = new FilteredElementCollector(doc)
            .OfClass(typeof(FamilyInstance))
            .OfCategory(BuiltInCategory.OST_CurtainWalls)
            .ToElementIds();

        var summary = new Dictionary<ElementId, CurtainWallSummary>();

        using (var transGroup = new TransactionGroup(doc))
        {
            transGroup.Start("Curtain Wall Analysis");

            foreach (var wallId in curtainWalls)
            {
                var wall = doc.GetElement(wallId) as FamilyInstance;
                if (wall == null) continue;

                var curtainGrid = wall.GetCurtainGrid();
                if (curtainGrid == null) continue;

                var panels = curtainGrid.GetPanels();
                var mullions = curtainGrid.GetMullions();

                var panelCount = panels.Count;
                var mullionCount = mullions.Count;

                summary.Add(wallId, new CurtainWallSummary(panelCount, mullionCount));
            }

            transGroup.Assimilate();
        }

        return summary;
    }

    public class CurtainWallSummary
    {
        public int PanelCount { get; }
        public int MullionCount { get; }

        public CurtainWallSummary(int panelCount, int mullionCount)
        {
            PanelCount = panelCount;
            MullionCount = mullionCount;
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Dictionary<string, (int panelCount, int mullionCount)> GetCurtainWallSummary(Document doc)
{
    var result = new Dictionary<string, (int, int)>();

    var curtainWalls = new FilteredElementCollector(doc)
        .OfClass(typeof(Wall))
        .WhereElementIsNotElementType()
        .Cast<Wall>()
        .Where(w => w.WallType.Kind == WallKind.Curtain);

    foreach (Wall cw in curtainWalls)
    {
        CurtainGrid grid = cw.CurtainGrid;
        if (grid == null) continue;

        int panelCount = grid.GetPanelIds().Count;
        int mullionCount = grid.GetMullionIds().Count;

        string name = cw.Name + " (" + cw.Id.IntegerValue + ")";
        result[name] = (panelCount, mullionCount);
    }
    return result;
}
```

</details>

---

### RC16 | Hard | Winner: revit-coder-14b (+0.020)

**Question:** Write C# code to find all duct instances in the model, get each duct's ConnectorManager, iterate through its Connectors, find connected elements via AllRefs, and build an adjacency list of connected duct ElementIds. Write code only. No explanation.

**Expected signals:** `ConnectorManager`, `Connector`, `ConnectorSet`, `AllRefs`, `Duct`, `FilteredElementCollector`, `BuiltInCategory.OST_DuctCurves`, `ElementId`, `Domain`, `MEPCurve`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.600 | 0.800 |
| Quality | 0.917 | 0.583 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.762** | **0.743** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Mechanical;
using System.Collections.Generic;

public class DuctConnectionFinder
{
    public static Dictionary<ElementId, List<ElementId>> FindDuctConnections(Document doc)
    {
        var ductConnections = new Dictionary<ElementId, List<ElementId>>();
        var ductCollector = new FilteredElementCollector(doc).OfClass(typeof(Duct));

        foreach (var duct in ductCollector)
        {
            var ductId = duct.Id;
            var connectorManager = duct as Duct;
            if (connectorManager == null) continue;

            var connectors = connectorManager.ConnectorManager.Connectors;
            foreach (var connector in connectors)
            {
                var allRefs = connector.AllRefs;
                foreach (var refConnector in allRefs)
                {
                    var connectedElementId = refConnector.Owner.Id;
                    if (!ductConnections.ContainsKey(ductId))
                    {
                        ductConnections[ductId] = new List<ElementId>();
                    }
                    ductConnections[ductId].Add(connectedElementId);
                }
            }
        }

        return ductConnections;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public Dictionary<ElementId, List<ElementId>> BuildDuctAdjacencyList(Document doc)
{
    var adjacency = new Dictionary<ElementId, List<ElementId>>();

    var ducts = new FilteredElementCollector(doc)
        .OfClass(typeof(Autodesk.Revit.DB.Mechanical.Duct))
        .WhereElementIsNotElementType()
        .ToElements();

    foreach (var elem in ducts)
    {
        MEPCurve duct = elem as MEPCurve;
        if (duct == null) continue;

        if (!adjacency.ContainsKey(duct.Id))
            adjacency[duct.Id] = new List<ElementId>();

        ConnectorSet connectors = duct.ConnectorManager.Connectors;
        foreach (Connector conn in connectors)
        {
            ConnectorSet allRefs = conn.AllRefs;
            if (allRefs == null) continue;

            foreach (Connector refConn in allRefs)
            {
                ElementId connectedId = refConn.Owner.Id;
                if (connectedId == duct.Id) continue;

                if (!adjacency[duct.Id].Contains(connectedId))
                    adjacency[duct.Id].Add(connectedId);
            }
        }
    }
    return adjacency;
}
```

</details>

---

### RC17 | Medium | Winner: Claude Opus 4.6 (-0.285)

**Question:** Write C# code to find a ViewSchedule by name, read its TableData including header and body sections, and return a List<Dictionary<string, string>> where each dictionary maps column header to cell value for that row. Write code only. No explanation.

**Expected signals:** `ViewSchedule`, `TableData`, `GetTableData`, `SectionData`, `GetSectionData`, `SectionType`, `GetCellText`, `NumberOfRows`, `NumberOfColumns`, `FilteredElementCollector`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.000 | 1.000 |
| Quality | 0.750 | 0.417 |
| Completeness | 0.875 | 0.825 |
| **Composite** | **0.487** | **0.772** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExternalData;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Piping;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Visual;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.Events;
using Autodesk.Revit.UI.ExternalData;
using Autodesk.Revit.UI.IFC;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.TaskDialogs;
using Autodesk.Revit.UI.UISupport;
using Autodesk.Revit.UI.ViewSupport;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Piping;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Visual;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.Events;
using Autodesk.Revit.UI.ExternalData;
using Autodesk.Revit.UI.IFC;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.TaskDialogs;
using Autodesk.Revit.UI.UISupport;
using Autodesk.Revit.UI.ViewSupport;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Piping;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Visual;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.Events;
using Autodesk.Revit.UI.ExternalData;
using Autodesk.Revit.UI.IFC;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.TaskDialogs;
using Autodesk.Revit.UI.UISupport;
using Autodesk.Revit.UI.ViewSupport;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Piping;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Visual;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.Events;
using Autodesk.Revit.UI.ExternalData;
using Autodesk.Revit.UI.IFC;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.TaskDialogs;
using Autodesk.Revit.UI.UISupport;
using Autodesk.Revit.UI.ViewSupport;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Piping;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Visual;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.Events;
using Autodesk.Revit.UI.ExternalData;
using Autodesk.Revit.UI.IFC;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.TaskDialogs;
using Autodesk.Revit.UI.UISupport;
using Autodesk.Revit.UI.ViewSupport;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Fabrication;
using Autodesk.Revit.DB.IFC;
using Autodesk.Revit.DB.Lighting;
using Autodesk.Revit.DB.Mechanical;
using Autodesk.Revit.DB.Piping;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.DB.Visual;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI.Events;
using Autodesk.Revit.UI.ExternalData;
using Autodesk.Revit.UI.IFC;
using Autodesk.Revit

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public List<Dictionary<string, string>> ReadScheduleData(Document doc, string scheduleName)
{
    var result = new List<Dictionary<string, string>>();

    ViewSchedule schedule = new FilteredElementCollector(doc)
        .OfClass(typeof(ViewSchedule))
        .Cast<ViewSchedule>()
        .FirstOrDefault(vs => vs.Name == scheduleName);

    if (schedule == null) return result;

    TableData tableData = schedule.GetTableData();
    TableSectionData headerSection = tableData.GetSectionData(SectionType.Header);
    TableSectionData bodySection = tableData.GetSectionData(SectionType.Body);

    // Get column headers from the first row of the body (or header section)
    int colCount = bodySection.NumberOfColumns;
    int rowCount = bodySection.NumberOfRows;

    // First row is typically the header in the body section
    var headers = new List<string>();
    for (int c = 0; c < colCount; c++)
    {
        headers.Add(bodySection.GetCellText(0, c));
    }

    // Data rows start at index 1
    for (int r = 1; r < rowCount; r++)
    {
        var row = new Dictionary<string, string>();
        for (int c = 0; c < colCount; c++)
        {
            row[headers[c]] = bodySection.GetCellText(r, c);
        }
        result.Add(row);
    }
    return result;
}
```

</details>

---

### RC18 | Easy | Winner: revit-coder-14b (+0.065)

**Question:** Write C# code to collect all wall instances that were created in a specific phase by filtering on the PHASE_CREATED built-in parameter. Get the phase from the active view. Write code only. No explanation.

**Expected signals:** `FilteredElementCollector`, `Phase`, `BuiltInParameter`, `PHASE_CREATED`, `View`, `WhereElementIsNotElementType`, `BuiltInCategory.OST_Walls`, `AsElementId`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.625 | 0.875 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.772** | **0.708** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class WallCollector
{
    public static ICollection<ElementId> CollectWallsInPhase(UIApplication uiApp)
    {
        Document doc = uiApp.ActiveUIDocument.Document;
        View activeView = doc.ActiveView;

        Phase phase = null;
        if (activeView != null)
        {
            phase = activeView.GetPhase();
        }

        if (phase == null)
        {
            throw new Exception("No phase found in the active view.");
        }

        FilteredElementCollector collector = new FilteredElementCollector(doc);
        collector.OfClass(typeof(Wall));

        ParameterFilterRuleFactory factory = new ParameterFilterRuleFactory();
        FilterRule rule = factory.CreateEqualsRule(BuiltInParameter.PHASE_CREATED, phase.Name);
        ElementParameterFilter filter = new ElementParameterFilter(rule);

        ICollection<ElementId> wallIds = collector.WherePasses(filter).ToElementIds();

        return wallIds;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public List<Wall> GetWallsByPhase(Document doc, View activeView)
{
    Phase phase = doc.GetElement(activeView.get_Parameter(BuiltInParameter.VIEW_PHASE).AsElementId()) as Phase;
    if (phase == null) return new List<Wall>();

    var walls = new FilteredElementCollector(doc, activeView.Id)
        .OfClass(typeof(Wall))
        .WhereElementIsNotElementType()
        .Cast<Wall>()
        .Where(w =>
        {
            Parameter phaseParam = w.get_Parameter(BuiltInParameter.PHASE_CREATED);
            return phaseParam != null && phaseParam.AsElementId() == phase.Id;
        })
        .ToList();

    return walls;
}
```

</details>

---

### RC19 | Easy | Winner: revit-coder-14b (+0.006)

**Question:** Write C# code to check if the document is workshared, get all worksets using FilteredWorksetCollector, and assign all walls to a workset named 'Structural' by setting the ELEM_PARTITION_PARAM inside a Transaction. Write code only. No explanation.

**Expected signals:** `FilteredWorksetCollector`, `WorksetKind`, `Workset`, `IsWorkshared`, `ELEM_PARTITION_PARAM`, `WorksetId`, `FilteredElementCollector`, `Transaction`, `BuiltInCategory.OST_Walls`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.667 | 0.778 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.814** | **0.809** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.UI;
using System;
using System.Collections.Generic;
using System.Linq;

public class WorksetAssignment
{
    public static void AssignWallsToWorkset(Document doc)
    {
        try
        {
            // Check if the document is workshared
            if (!doc.IsWorkshared)
            {
                TaskDialog.Show("Error", "The document is not workshared.");
                return;
            }

            // Get all worksets
            FilteredWorksetCollector worksetCollector = new FilteredWorksetCollector(doc);
            IList<Workset> worksets = worksetCollector.ToWorksets();

            // Find the 'Structural' workset
            Workset structuralWorkset = worksets.FirstOrDefault(ws => ws.Name == "Structural");

            if (structuralWorkset == null)
            {
                TaskDialog.Show("Error", "The 'Structural' workset does not exist.");
                return;
            }

            // Get all walls
            FilteredElementCollector wallCollector = new FilteredElementCollector(doc);
            wallCollector.OfClass(typeof(Wall));
            IList<Element> walls = wallCollector.ToElements();

            // Start a transaction
            using (Transaction transaction = new Transaction(doc, "Assign Walls to Structural Workset"))
            {
                transaction.Start();

                // Assign each wall to the 'Structural' workset
                foreach (Wall wall in walls)
                {
                    Parameter partitionParam = wall.get_Parameter(BuiltInParameter.ELEM_PARTITION_PARAM);
                    if (partitionParam != null)
                    {
                        partitionParam.Set(structuralWorkset.Id.IntegerValue);
                    }
                }

                // Commit the transaction
                transaction.Commit();
            }
        }
        catch (Exception ex)
        {
            TaskDialog.Show("Error", "An error occurred: " + ex.Message);
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void AssignWallsToWorkset(Document doc)
{
    if (!doc.IsWorkshared)
        throw new InvalidOperationException("Document is not workshared.");

    FilteredWorksetCollector worksetCollector = new FilteredWorksetCollector(doc)
        .OfKind(WorksetKind.UserWorkset);

    Workset structuralWorkset = worksetCollector
        .FirstOrDefault(ws => ws.Name == "Structural");

    if (structuralWorkset == null)
        throw new Exception("Workset 'Structural' not found.");

    var walls = new FilteredElementCollector(doc)
        .OfClass(typeof(Wall))
        .WhereElementIsNotElementType()
        .ToElements();

    using (Transaction tx = new Transaction(doc, "Assign Walls to Workset"))
    {
        tx.Start();
        foreach (var wall in walls)
        {
            Parameter worksetParam = wall.get_Parameter(BuiltInParameter.ELEM_PARTITION_PARAM);
            if (worksetParam != null && !worksetParam.IsReadOnly)
            {
                worksetParam.Set(structuralWorkset.Id.IntegerValue);
            }
        }
        tx.Commit();
    }
}
```

</details>

---

### RC20 | Medium | Winner: Claude Opus 4.6 (-0.019)

**Question:** Write C# code to create a linear dimension along a wall in the active view. Get the wall's LocationCurve, create a ReferenceArray from the wall's end references, and use doc.Create.NewDimension to place the dimension. Write code only. No explanation.

**Expected signals:** `Dimension`, `NewDimension`, `ReferenceArray`, `LocationCurve`, `Line`, `Reference`, `View`, `Transaction`, `Curve`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.889 | 1.000 |
| Quality | 0.917 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.878** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void CreateLinearDimensionAlongWall()
{
    UIDocument uidoc = this.ActiveUIDocument;
    Document doc = uidoc.Document;
    View view = doc.ActiveView;

    // Get the selected wall
    Reference wallRef = uidoc.Selection.PickObject(ObjectType.Edge, "Select a wall");
    Element wall = doc.GetElement(wallRef);

    // Get the wall's location curve
    LocationCurve locationCurve = wall.Location as LocationCurve;
    Curve wallCurve = locationCurve.Curve;

    // Create a reference array from the wall's end references
    ReferenceArray references = new ReferenceArray();
    references.Append(wallRef);

    // Create a dimension along the wall
    Dimension dimension = doc.Create.NewDimension(view, wallCurve, references);

    // Set the dimension type
    DimensionType dimensionType = new FilteredElementCollector(doc)
        .OfClass(typeof(DimensionType))
        .FirstElement() as DimensionType;
    dimension.ChangeTypeId(dimensionType.Id);
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateWallDimension(Document doc, Wall wall, View view)
{
    LocationCurve locCurve = wall.Location as LocationCurve;
    if (locCurve == null) return;

    Curve curve = locCurve.Curve;

    Options geomOptions = new Options();
    geomOptions.ComputeReferences = true;
    geomOptions.View = view;

    GeometryElement geomElem = wall.get_Geometry(geomOptions);
    ReferenceArray refArray = new ReferenceArray();

    // Get references from the wall's end faces
    foreach (GeometryObject geomObj in geomElem)
    {
        Solid solid = geomObj as Solid;
        if (solid == null || solid.Volume <= 0) continue;

        foreach (Face face in solid.Faces)
        {
            PlanarFace pf = face as PlanarFace;
            if (pf != null)
            {
                XYZ normal = pf.FaceNormal;
                XYZ wallDir = (curve.GetEndPoint(1) - curve.GetEndPoint(0)).Normalize();
                if (Math.Abs(normal.DotProduct(wallDir)) > 0.9)
                {
                    refArray.Append(face.Reference);
                }
            }
            if (refArray.Size >= 2) break;
        }
        if (refArray.Size >= 2) break;
    }

    if (refArray.Size < 2) return;

    using (Transaction tx = new Transaction(doc, "Create Dimension"))
    {
        tx.Start();
        XYZ offset = curve.GetEndPoint(0) + new XYZ(0, 3, 0);
        Line dimLine = Line.CreateBound(offset, offset + (curve.GetEndPoint(1) - curve.GetEndPoint(0)));
        doc.Create.NewDimension(view, dimLine, refArray);
        tx.Commit();
    }
}
```

</details>

---

### RC21 | Medium | Winner: Claude Opus 4.6 (-0.039)

**Question:** Write C# code to create a new Material in the document, set its color to red (255,0,0), assign a solid fill pattern to its surface foreground, and apply it as the structural material of an existing wall via parameter. Write code only. No explanation.

**Expected signals:** `Material`, `Create`, `Color`, `FillPatternElement`, `GetSolidFillPatternId`, `SurfaceForegroundPatternId`, `SurfaceForegroundPatternColor`, `Transaction`, `FilteredElementCollector`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.667 | 0.889 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.814** | **0.853** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Structure;

public void CreateAndApplyMaterial(Document doc, ElementId wallId)
{
    using (Transaction transaction = new Transaction(doc, "Create and Apply Material"))
    {
        transaction.Start();

        // Create a new material
        Material material = Material.Create(doc, "Red Material");

        // Set the material color to red
        Color redColor = new Color(255, 0, 0);
        material.SetColor(redColor);

        // Get the solid fill pattern
        FillPattern solidFillPattern = FillPattern.GetSolidFillPattern(doc);
        FillPatternElement fillPatternElement = FillPatternElement.Create(doc, solidFillPattern);

        // Set the fill pattern to the material's surface foreground
        material.SurfaceForegroundPatternId = fillPatternElement.Id;

        // Apply the material to the wall
        Parameter wallMaterialParam = doc.GetElement(wallId).get_Parameter(BuiltInParameter.MATERIAL_PARAM);
        wallMaterialParam.Set(material.Id);

        transaction.Commit();
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateAndApplyRedMaterial(Document doc, Wall wall)
{
    using (Transaction tx = new Transaction(doc, "Create Material"))
    {
        tx.Start();

        ElementId materialId = Material.Create(doc, "Red Material");
        Material mat = doc.GetElement(materialId) as Material;
        mat.Color = new Color(255, 0, 0);

        // Find solid fill pattern
        FillPatternElement solidFill = new FilteredElementCollector(doc)
            .OfClass(typeof(FillPatternElement))
            .Cast<FillPatternElement>()
            .FirstOrDefault(fp => fp.GetFillPattern().IsSolidFill);

        if (solidFill != null)
        {
            mat.SurfaceForegroundPatternId = solidFill.Id;
            mat.SurfaceForegroundPatternColor = new Color(255, 0, 0);
        }

        // Apply as structural material
        Parameter structMatParam = wall.get_Parameter(BuiltInParameter.STRUCTURAL_MATERIAL_PARAM);
        if (structMatParam != null && !structMatParam.IsReadOnly)
        {
            structMatParam.Set(materialId);
        }

        tx.Commit();
    }
}
```

</details>

---

### RC22 | Easy | Winner: revit-coder-14b (+0.073)

**Question:** Write C# code to copy a set of elements from their current position to a new location offset by (10, 0, 0) using ElementTransformUtils.CopyElements with a translation transform inside a Transaction. Write code only. No explanation.

**Expected signals:** `ElementTransformUtils`, `CopyElements`, `Transform`, `Translation`, `XYZ`, `Transaction`, `ElementId`, `ICollection`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 1.000 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.775 | 0.700 |
| **Composite** | **0.932** | **0.860** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void CopyElementsWithTransform(Document doc, ICollection<ElementId> elementIds)
{
    using (Transaction transaction = new Transaction(doc, "Copy Elements"))
    {
        transaction.Start();

        try
        {
            // Define the translation transform
            Transform translation = Transform.CreateTranslation(new XYZ(10, 0, 0));

            // Copy the elements
            ElementTransformUtils.CopyElements(doc, elementIds, translation);
        }
        catch (Exception ex)
        {
            transaction.RollBack();
            throw ex;
        }
        finally
        {
            transaction.Commit();
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CopyElementsWithOffset(Document doc, ICollection<ElementId> elementIds)
{
    using (Transaction tx = new Transaction(doc, "Copy Elements"))
    {
        tx.Start();

        XYZ translation = new XYZ(10, 0, 0);
        ElementTransformUtils.CopyElements(doc, elementIds, translation);

        tx.Commit();
    }
}
```

</details>

---

### RC23 | Hard | Winner: Claude Opus 4.6 (-0.190)

**Question:** Write C# code to create rebar in a concrete beam element using Rebar.CreateFromCurves. Define a rectangular stirrup shape with four line segments, specify the RebarBarType and RebarHookType, set RebarStyle to Stirrup, and place it inside a Transaction. Write code only. No explanation.

**Expected signals:** `Rebar`, `CreateFromCurves`, `RebarBarType`, `RebarHookType`, `RebarHookOrientation`, `RebarStyle`, `FilteredElementCollector`, `Line`, `Transaction`, `XYZ`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.500 | 1.000 |
| Quality | 0.917 | 0.833 |
| Completeness | 0.775 | 0.825 |
| **Composite** | **0.708** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Structure;
using System.Collections.Generic;

public class RebarCreator
{
    public static Rebar CreateStirrup(Document doc, ElementId hostId, ElementId barTypeId, ElementId hookTypeId, XYZ origin, XYZ xVec, XYZ yVec)
    {
        // Define the stirrup shape
        List<Curve> curves = new List<Curve>();
        curves.Add(Line.CreateBound(origin, origin.Add(xVec)));
        curves.Add(Line.CreateBound(origin.Add(xVec), origin.Add(xVec).Add(yVec)));
        curves.Add(Line.CreateBound(origin.Add(xVec).Add(yVec), origin.Add(yVec)));
        curves.Add(Line.CreateBound(origin.Add(yVec), origin));

        // Create the stirrup
        Rebar rebar = Rebar.CreateFromCurves(doc, RebarStyle.Stirrup, barTypeId, hookTypeId, hostId, curves, origin, xVec, yVec);

        return rebar;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateStirrup(Document doc, Element beam)
{
    using (Transaction tx = new Transaction(doc, "Create Rebar"))
    {
        tx.Start();

        RebarBarType barType = new FilteredElementCollector(doc)
            .OfClass(typeof(RebarBarType))
            .Cast<RebarBarType>()
            .FirstOrDefault();

        RebarHookType hookType = new FilteredElementCollector(doc)
            .OfClass(typeof(RebarHookType))
            .Cast<RebarHookType>()
            .FirstOrDefault();

        // Define stirrup corners (assuming beam cross-section)
        double width = 0.3; // ~300mm
        double height = 0.5; // ~500mm
        double cover = 0.025; // 25mm cover

        double w = width / 2 - cover;
        double h = height / 2 - cover;

        XYZ p0 = new XYZ(-w, 0, -h);
        XYZ p1 = new XYZ(w, 0, -h);
        XYZ p2 = new XYZ(w, 0, h);
        XYZ p3 = new XYZ(-w, 0, h);

        List<Curve> curves = new List<Curve>
        {
            Line.CreateBound(p0, p1),
            Line.CreateBound(p1, p2),
            Line.CreateBound(p2, p3),
            Line.CreateBound(p3, p0)
        };

        XYZ normal = XYZ.BasisY;

        Rebar rebar = Rebar.CreateFromCurves(
            doc,
            RebarStyle.StirrupTie,
            barType,
            hookType,
            hookType,
            beam,
            normal,
            curves,
            RebarHookOrientation.Left,
            RebarHookOrientation.Left,
            true,
            true);

        tx.Commit();
    }
}
```

</details>

---

### RC24 | Medium | Winner: revit-coder-14b (+0.010)

**Question:** Write C# code to place room tags on all untagged rooms in the active view using IndependentTag.Create. Get each room's location point, use TagMode.TM_ADDBY_CATEGORY, and set horizontal orientation. Write code only. No explanation.

**Expected signals:** `IndependentTag`, `Create`, `TagMode`, `TagOrientation`, `Reference`, `LocationPoint`, `Point`, `FilteredElementCollector`, `Transaction`, `View`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.900 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.907** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;

public class PlaceRoomTags : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIApplication uiApp = commandData.Application;
        UIDocument uidoc = uiApp.ActiveUIDocument;
        Document doc = uidoc.Document;

        if (doc.IsReadOnly)
        {
            message = "Document is read-only.";
            return Result.Failed;
        }

        try
        {
            using (TransactionGroup tg = new TransactionGroup(doc))
            {
                tg.Start("Place Room Tags");

                using (Transaction t = new Transaction(doc, "Place Room Tags"))
                {
                    t.Start();

                    FilteredElementCollector collector = new FilteredElementCollector(doc, uidoc.ActiveView.Id)
                        .OfClass(typeof(SpatialElement))
                        .OfCategory(BuiltInCategory.OST_Rooms);

                    List<ElementId> untaggedRoomIds = new List<ElementId>();

                    foreach (Room room in collector.Cast<Room>())
                    {
                        if (room.GetTagCount() == 0)
                        {
                            untaggedRoomIds.Add(room.Id);
                        }
                    }

                    if (untaggedRoomIds.Count > 0)
                    {
                        IndependentTagOptions options = new IndependentTagOptions();
                        options.TagMode = TagMode.TM_ADDBY_CATEGORY;
                        options.TagOrientation = TagOrientation.Horizontal;

                        foreach (ElementId roomId in untaggedRoomIds)
                        {
                            Room room = doc.GetElement(roomId) as Room;
                            XYZ locationPoint = room.Location.Point;

                            IndependentTag tag = IndependentTag.Create(doc, uidoc.ActiveView.Id, locationPoint, options);
                            tag.IsLeaderVisible = true;
                            tag.LeaderEnd = locationPoint.Add(new XYZ(0, 0, 1));
                        }
                    }

                    t.Commit();
                }

                tg.Assimilate();
            }
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }

        return Result.Succeeded;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void TagUntaggedRooms(Document doc, View activeView)
{
    var rooms = new FilteredElementCollector(doc, activeView.Id)
        .OfCategory(BuiltInCategory.OST_Rooms)
        .WhereElementIsNotElementType()
        .Cast<Room>()
        .Where(r => r.Area > 0);

    // Get already tagged room ids
    var existingTags = new FilteredElementCollector(doc, activeView.Id)
        .OfCategory(BuiltInCategory.OST_RoomTags)
        .WhereElementIsNotElementType()
        .Cast<RoomTag>()
        .Select(t => t.Room?.Id)
        .Where(id => id != null)
        .ToHashSet();

    using (Transaction tx = new Transaction(doc, "Tag Rooms"))
    {
        tx.Start();

        foreach (Room room in rooms)
        {
            if (existingTags.Contains(room.Id)) continue;

            LocationPoint locPt = room.Location as LocationPoint;
            if (locPt == null) continue;

            UV uv = new UV(locPt.Point.X, locPt.Point.Y);
            Reference roomRef = new Reference(room);

            IndependentTag.Create(
                doc,
                activeView.Id,
                roomRef,
                false,
                TagMode.TM_ADDBY_CATEGORY,
                TagOrientation.Horizontal,
                locPt.Point);
        }

        tx.Commit();
    }
}
```

</details>

---

### RC25 | Easy | Winner: Claude Opus 4.6 (-0.024)

**Question:** Write C# code to get the user's current selection from UIDocument, create a Group from the selected elements using doc.Create.NewGroup, and show the group name in a TaskDialog. Include a Transaction. Write code only. No explanation.

**Expected signals:** `Group`, `NewGroup`, `Selection`, `GetElementIds`, `ICollection`, `ElementId`, `Transaction`, `TaskDialog`, `UIDocument`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.778 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.859** | **0.882** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.UI;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI.Selection;
using System.Collections.Generic;

public class GroupCreator : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIDocument uidoc = commandData.Application.ActiveUIDocument;
        Document doc = uidoc.Document;

        try
        {
            using (Transaction transaction = new Transaction(doc, "Create Group"))
            {
                transaction.Start();

                Reference ref1 = uidoc.Selection.PickObject(ObjectType.Element, "Select first element");
                Reference ref2 = uidoc.Selection.PickObject(ObjectType.Element, "Select second element");

                ElementId id1 = ref1.ElementId;
                ElementId id2 = ref2.ElementId;

                List<ElementId> selectedIds = new List<ElementId> { id1, id2 };
                Group group = doc.Create.NewGroup(selectedIds);

                transaction.Commit();

                TaskDialog.Show("Group Name", group.Name);
            }
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }

        return Result.Succeeded;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
[Transaction(TransactionMode.Manual)]
public class GroupSelectedCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIDocument uidoc = commandData.Application.ActiveUIDocument;
        Document doc = uidoc.Document;

        ICollection<ElementId> selectedIds = uidoc.Selection.GetElementIds();
        if (selectedIds.Count == 0)
        {
            TaskDialog.Show("Error", "No elements selected.");
            return Result.Failed;
        }

        using (Transaction tx = new Transaction(doc, "Create Group"))
        {
            tx.Start();
            Group group = doc.Create.NewGroup(selectedIds);
            tx.Commit();

            TaskDialog.Show("Group Created", "Group name: " + group.GroupType.Name);
        }

        return Result.Succeeded;
    }
}
```

</details>

---

### RC26 | Easy | Winner: revit-coder-14b (+0.075)

**Question:** Write C# code to access the ProjectInfo object from the document and read the BuildingName, Address, and Number properties. Display them in a TaskDialog. Write code only. No explanation.

**Expected signals:** `ProjectInfo`, `doc.ProjectInformation`, `BuildingName`, `Address`, `Number`, `TaskDialog`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 1.000 | 1.000 |
| Quality | 0.917 | 0.667 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.907** | **0.833** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void GetProjectInfoData()
{
    Document doc = CommandData.Application.ActiveUIDocument.Document;
    ProjectInfo projectInfo = doc.ProjectInformation;

    if (projectInfo != null)
    {
        string buildingName = projectInfo.BuildingName;
        string address = projectInfo.Address;
        string number = projectInfo.Number;

        TaskDialog.Show("Project Information", $"Building Name: {buildingName}\nAddress: {address}\nNumber: {number}");
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
[Transaction(TransactionMode.Manual)]
public class ProjectInfoCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        Document doc = commandData.Application.ActiveUIDocument.Document;

        ProjectInfo info = doc.ProjectInformation;

        string buildingName = info.BuildingName ?? "N/A";
        string address = info.Address ?? "N/A";
        string number = info.Number ?? "N/A";

        TaskDialog.Show("Project Info",
            $"Building Name: {buildingName}\n" +
            $"Address: {address}\n" +
            $"Number: {number}");

        return Result.Succeeded;
    }
}
```

</details>

---

### RC27 | Easy | Winner: revit-coder-14b (+0.050)

**Question:** Write C# code to create a ModelLine in the active document. Create a SketchPlane from a Plane at the origin, then use doc.Create.NewModelCurve to draw a line from (0,0,0) to (10,0,0). Include a Transaction. Write code only. No explanation.

**Expected signals:** `SketchPlane`, `Plane`, `NewModelCurve`, `Line.CreateBound`, `XYZ`, `Transaction`, `doc.Create`, `ModelCurve`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 1.000 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.932** | **0.882** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void CreateModelLine()
{
    Document doc = Command.thisDoc;
    UIDocument uidoc = Command.thisUIDoc;

    try
    {
        using (Transaction trans = new Transaction(doc, "Create ModelLine"))
        {
            trans.Start();

            // Define plane at origin
            Plane plane = Plane.CreateByOriginAndBasis(XYZ.Zero, XYZ.BasisX, XYZ.BasisY);
            SketchPlane sketchPlane = SketchPlane.Create(doc, plane);

            // Define line
            Line line = Line.CreateBound(XYZ.Zero, new XYZ(10, 0, 0));

            // Create model line
            ModelCurve modelCurve = doc.Create.NewModelCurve(line, sketchPlane);

            trans.Commit();
        }
    }
    catch (Exception ex)
    {
        TaskDialog.Show("Error", ex.Message);
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateModelLine(Document doc)
{
    using (Transaction tx = new Transaction(doc, "Create Model Line"))
    {
        tx.Start();

        XYZ origin = XYZ.Zero;
        XYZ normal = XYZ.BasisZ;
        Plane plane = Plane.CreateByNormalAndOrigin(normal, origin);
        SketchPlane sketchPlane = SketchPlane.Create(doc, plane);

        XYZ start = new XYZ(0, 0, 0);
        XYZ end = new XYZ(10, 0, 0);
        Line line = Line.CreateBound(start, end);

        doc.Create.NewModelCurve(line, sketchPlane);

        tx.Commit();
    }
}
```

</details>

---

### RC28 | Easy | Winner: Claude Opus 4.6 (-0.050)

**Question:** Write C# code to duplicate an existing ViewPlan using View.Duplicate with ViewDuplicateOption.Duplicate, then rename the duplicated view by setting its ViewName property. Include a Transaction. Write code only. No explanation.

**Expected signals:** `Duplicate`, `ViewDuplicateOption`, `ViewPlan`, `ViewName`, `Transaction`, `FilteredElementCollector`, `ElementId`, `doc.GetElement`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.625 | 0.875 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.782** | **0.833** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void DuplicateAndRenameViewPlan(ViewPlan originalView)
{
    try
    {
        using (Transaction transaction = new Transaction(originalView.Document, "Duplicate and Rename ViewPlan"))
        {
            transaction.Start();

            // Duplicate the view
            ViewPlan duplicatedView = originalView.Duplicate(ViewDuplicateOption.Duplicate) as ViewPlan;

            // Rename the duplicated view
            duplicatedView.ViewName = "Renamed ViewPlan";

            transaction.Commit();
        }
    }
    catch (Exception ex)
    {
        // Handle any exceptions
        TaskDialog.Show("Error", ex.Message);
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void DuplicateView(Document doc)
{
    ViewPlan viewPlan = new FilteredElementCollector(doc)
        .OfClass(typeof(ViewPlan))
        .Cast<ViewPlan>()
        .FirstOrDefault(v => !v.IsTemplate && v.CanViewBeDuplicated(ViewDuplicateOption.Duplicate));

    if (viewPlan == null) return;

    using (Transaction tx = new Transaction(doc, "Duplicate View"))
    {
        tx.Start();

        ElementId newViewId = viewPlan.Duplicate(ViewDuplicateOption.Duplicate);
        View newView = doc.GetElement(newViewId) as View;
        newView.Name = viewPlan.Name + " - Copy";

        tx.Commit();
    }
}
```

</details>

---

### RC29 | Medium | Winner: Claude Opus 4.6 (-0.039)

**Question:** Write C# code implementing an IExternalEventHandler for executing Revit API calls from a modeless dialog. Show the handler class with Execute method, ExternalEvent.Create registration, and how to call Raise() from a button click. Include a Transaction inside the handler. Write code only. No explanation.

**Expected signals:** `IExternalEventHandler`, `Execute`, `ExternalEvent`, `Create`, `Raise`, `UIApplication`, `Transaction`, `GetName`, `modeless`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.778 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.859** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.UI;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI.Events;
using System.Windows.Forms;

namespace RevitExternalEventHandler
{
    public class ExternalEventHandler : IExternalEventHandler
    {
        public void Execute(UIApplication uiapp)
        {
            try
            {
                var doc = uiapp.ActiveUIDocument?.Document;
                if (doc == null) return;

                using (var trans = new Transaction(doc, "External Event Transaction"))
                {
                    if (trans.Start() != TransactionStatus.Started) return;

                    // Example: Create a wall
                    var wallType = new FilteredElementCollector(doc)
                        .OfClass(typeof(FamilySymbol))
                        .Where(e => e.Category?.Name == "Walls")
                        .First();

                    var line = Line.CreateBound(XYZ.Zero, XYZ.BasisX.Multiply(10));
                    var wall = Wall.Create(doc, wallType, line, doc.ActiveView, true, 10, 10);

                    trans.Commit();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error: {ex.Message}");
            }
        }

        public string GetName()
        {
            return "ExternalEventHandler";
        }
    }

    public class ExternalEventHandlerManager
    {
        public static ExternalEvent CreateEvent()
        {
            var handler = new ExternalEventHandler();
            return ExternalEvent.Create(handler);
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public class MyEventHandler : IExternalEventHandler
{
    private Action<UIApplication> _action;

    public void SetAction(Action<UIApplication> action)
    {
        _action = action;
    }

    public void Execute(UIApplication app)
    {
        if (_action == null) return;

        Document doc = app.ActiveUIDocument.Document;
        using (Transaction tx = new Transaction(doc, "External Event Action"))
        {
            tx.Start();
            _action.Invoke(app);
            tx.Commit();
        }
    }

    public string GetName() => "MyEventHandler";
}

// Registration and usage in a modeless WPF dialog
public partial class MyModelessDialog : Window
{
    private static MyEventHandler _handler;
    private static ExternalEvent _externalEvent;

    public MyModelessDialog()
    {
        InitializeComponent();
        _handler = new MyEventHandler();
        _externalEvent = ExternalEvent.Create(_handler);
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
        _handler.SetAction((uiApp) =>
        {
            Document doc = uiApp.ActiveUIDocument.Document;
            // Perform Revit API calls here
            TaskDialog.Show("Info", "Executed from modeless dialog.");
        });
        _externalEvent.Raise();
    }
}
```

</details>

---

### RC30 | Medium | Winner: revit-coder-14b (+0.165)

**Question:** Write C# code using ParameterFilterRuleFactory to create a FilterRule that finds walls where the Mark parameter equals a specific string value. Use it with a ParameterValueProvider and ElementParameterFilter in a FilteredElementCollector. Write code only. No explanation.

**Expected signals:** `ParameterFilterRuleFactory`, `FilterRule`, `ParameterValueProvider`, `ElementParameterFilter`, `FilteredElementCollector`, `BuiltInParameter`, `ALL_MODEL_MARK`, `BuiltInCategory.OST_Walls`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.750 | 0.750 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.823** | **0.657** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.DB.Structure;

public class WallFilter
{
    public static FilterRule CreateWallMarkFilter(string markValue)
    {
        // Create a parameter value provider for the "Mark" parameter
        ParameterValueProvider provider = new ParameterValueProvider(new ElementId(BuiltInParameter.ALL_MODEL_MARK));

        // Create a string filter for the specific mark value
        FilterStringRule rule = new FilterStringRule(provider, new FilterStringContains(), markValue, true);

        return rule;
    }

    public static FilteredElementCollector ApplyWallMarkFilter(Document doc, string markValue)
    {
        // Create the filter rule
        FilterRule filterRule = CreateWallMarkFilter(markValue);

        // Create the parameter value provider and filter
        ElementParameterFilter filter = new ElementParameterFilter(filterRule);

        // Apply the filter to the wall collection
        FilteredElementCollector collector = new FilteredElementCollector(doc)
            .OfClass(typeof(Wall))
            .WherePasses(filter);

        return collector;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public List<Wall> GetWallsByMark(Document doc, string markValue)
{
    ElementId markParamId = new ElementId(BuiltInParameter.ALL_MODEL_MARK);

    FilterRule rule = ParameterFilterRuleFactory.CreateEqualsRule(
        markParamId, markValue, false);

    ElementParameterFilter paramFilter = new ElementParameterFilter(rule);

    List<Wall> walls = new FilteredElementCollector(doc)
        .OfClass(typeof(Wall))
        .WhereElementIsNotElementType()
        .WherePasses(paramFilter)
        .Cast<Wall>()
        .ToList();

    return walls;
}
```

</details>

---

### RC31 | Medium | Winner: revit-coder-14b (+0.006)

**Question:** Write C# code to set a section box on a 3D view. Create a BoundingBoxXYZ with specific Min and Max points, find a View3D in the document, enable and assign the section box using View3D.SetSectionBox inside a Transaction. Write code only. No explanation.

**Expected signals:** `View3D`, `BoundingBoxXYZ`, `SetSectionBox`, `IsSectionBoxActive`, `Min`, `Max`, `XYZ`, `Transaction`, `FilteredElementCollector`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.889 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.888** | **0.882** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public void SetSectionBoxOn3DView(Document doc)
{
    // Create a BoundingBoxXYZ with specific Min and Max points
    BoundingBoxXYZ sectionBox = new BoundingBoxXYZ();
    sectionBox.Min = new XYZ(0, 0, 0);
    sectionBox.Max = new XYZ(100, 100, 100);

    // Find a View3D in the document
    FilteredElementCollector collector = new FilteredElementCollector(doc);
    collector.OfClass(typeof(View3D));
    View3D view3D = collector.FirstElement() as View3D;

    if (view3D != null)
    {
        // Start a transaction
        using (Transaction transaction = new Transaction(doc, "Set Section Box"))
        {
            transaction.Start();

            // Enable and assign the section box
            view3D.EnableSectionBox(true);
            view3D.SetSectionBox(sectionBox);

            // Commit the transaction
            transaction.Commit();
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void SetSectionBox(Document doc)
{
    View3D view3d = new FilteredElementCollector(doc)
        .OfClass(typeof(View3D))
        .Cast<View3D>()
        .FirstOrDefault(v => !v.IsTemplate);

    if (view3d == null) return;

    BoundingBoxXYZ bbox = new BoundingBoxXYZ();
    bbox.Min = new XYZ(-50, -50, -10);
    bbox.Max = new XYZ(50, 50, 30);

    using (Transaction tx = new Transaction(doc, "Set Section Box"))
    {
        tx.Start();
        view3d.IsSectionBoxActive = true;
        view3d.SetSectionBox(bbox);
        tx.Commit();
    }
}
```

</details>

---

### RC32 | Medium | Winner: revit-coder-14b (+0.065)

**Question:** Write C# code to create a new Level at a specified elevation using Level.Create, then create multiple Grid lines along the X axis using Grid.Create with Line geometry. Both operations inside a Transaction. Write code only. No explanation.

**Expected signals:** `Level.Create`, `Grid.Create`, `Line.CreateBound`, `XYZ`, `Transaction`, `Elevation`, `FilteredElementCollector`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.857 | 0.857 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.890** | **0.825** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

public class CreateLevelAndGrids : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIApplication uiApp = commandData.Application;
        UIDocument uiDoc = uiApp.ActiveUIDocument;
        Document doc = uiDoc.Document;

        try
        {
            using (Transaction trans = new Transaction(doc, "Create Level and Grids"))
            {
                trans.Start();

                // Create a new Level at a specified elevation
                Level newLevel = Level.Create(doc, 100.0);
                if (newLevel == null)
                {
                    message = "Failed to create Level.";
                    trans.RollBack();
                    return Result.Failed;
                }

                // Create multiple Grid lines along the X axis
                for (int i = 0; i < 5; i++)
                {
                    XYZ startPoint = new XYZ(i * 10, 0, newLevel.Elevation);
                    XYZ endPoint = new XYZ(i * 10, 10, newLevel.Elevation);
                    Line line = Line.CreateBound(startPoint, endPoint);

                    Grid grid = Grid.Create(doc, line);
                    if (grid == null)
                    {
                        message = "Failed to create Grid.";
                        trans.RollBack();
                        return Result.Failed;
                    }
                }

                trans.Commit();
            }
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }

        return Result.Succeeded;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateLevelAndGrids(Document doc)
{
    using (Transaction tx = new Transaction(doc, "Create Level and Grids"))
    {
        tx.Start();

        // Create a new level at 15 feet elevation
        Level newLevel = Level.Create(doc, 15.0);
        newLevel.Name = "Level 4";

        // Create grid lines along X axis
        double gridLength = 100.0;
        for (int i = 0; i < 5; i++)
        {
            double offset = i * 20.0;
            XYZ start = new XYZ(offset, -gridLength / 2, 0);
            XYZ end = new XYZ(offset, gridLength / 2, 0);
            Line line = Line.CreateBound(start, end);
            Grid grid = Grid.Create(doc, line);
            grid.Name = ((char)('A' + i)).ToString();
        }

        tx.Commit();
    }
}
```

</details>

---

### RC33 | Medium | Winner: revit-coder-14b (+0.025)

**Question:** Write C# code using Extensible Storage to create a Schema with SchemaBuilder, add a string field and a double field, then store data on a wall element using Entity and retrieve it back. Include GUID, schema name, and a Transaction. Write code only. No explanation.

**Expected signals:** `SchemaBuilder`, `Schema`, `Entity`, `FieldBuilder`, `SetEntity`, `GetEntity`, `Guid`, `Transaction`, `Element`, `Get`, `Set`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.818 | 0.818 |
| Quality | 0.917 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.850** | **0.825** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.ExtensibleStorage;
using Autodesk.Revit.UI;

public class ExtensibleStorageExample
{
    public static void CreateSchemaAndStoreData(Document doc, Wall wall)
    {
        // Define schema GUID and name
        Guid schemaGuid = new Guid("B7A9E62D-518B-422C-9141-1234567890AB");
        string schemaName = "WallDataSchema";

        // Create schema using SchemaBuilder
        SchemaBuilder builder = new SchemaBuilder(schemaGuid);
        builder.SetSchemaName(schemaName);
        builder.AddSimpleField("Description", typeof(string));
        builder.AddSimpleField("Measurement", typeof(double));
        Schema schema = builder.Finish();

        // Create entity and set field values
        Entity entity = new Entity(schema);
        entity.Set("Description", "Wall information");
        entity.Set("Measurement", 3.5);

        // Store entity on wall element
        wall.SetEntity(schema.GUID, entity);
    }

    public static void RetrieveData(Document doc, Wall wall)
    {
        // Retrieve entity from wall element
        Entity entity = wall.GetEntity(new Guid("B7A9E62D-518B-422C-9141-1234567890AB"));

        // Get field values
        string description = entity.Get<string>("Description");
        double measurement = entity.Get<double>("Measurement");

        // Output field values
        TaskDialog.Show("Wall Data", $"Description: {description}\nMeasurement: {measurement}");
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public class ExtensibleStorageExample
{
    private static readonly Guid SchemaGuid = new Guid("A1B2C3D4-E5F6-7890-ABCD-EF1234567890");

    public void StoreData(Document doc, Wall wall)
    {
        SchemaBuilder builder = new SchemaBuilder(SchemaGuid);
        builder.SetSchemaName("CustomData");
        builder.SetReadAccessLevel(AccessLevel.Public);
        builder.SetWriteAccessLevel(AccessLevel.Public);

        builder.AddSimpleField("Description", typeof(string));
        builder.AddSimpleField("Length", typeof(double));

        Schema schema = builder.Finish();

        using (Transaction tx = new Transaction(doc, "Store Data"))
        {
            tx.Start();

            Entity entity = new Entity(schema);
            entity.Set<string>("Description", "Exterior Wall");
            entity.Set<double>("Length", 25.5);

            wall.SetEntity(entity);

            tx.Commit();
        }
    }

    public (string description, double length) RetrieveData(Wall wall)
    {
        Schema schema = Schema.Lookup(SchemaGuid);
        if (schema == null) return (null, 0);

        Entity entity = wall.GetEntity(schema);
        if (!entity.IsValid()) return (null, 0);

        string description = entity.Get<string>("Description");
        double length = entity.Get<double>("Length");

        return (description, length);
    }
}
```

</details>

---

### RC34 | Medium | Winner: Claude Opus 4.6 (-0.030)

**Question:** Write C# code to create a ParameterFilterElement with filter rules, apply it to the active view using View.AddFilter, and set color overrides for matching elements using OverrideGraphicSettings with SetProjectionLineColor. Include a Transaction. Write code only. No explanation.

**Expected signals:** `ParameterFilterElement`, `FilterRule`, `ParameterFilterRuleFactory`, `OverrideGraphicSettings`, `SetProjectionLineColor`, `Color`, `AddFilter`, `SetFilterOverrides`, `View`, `Transaction`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.800 | 1.000 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.868** | **0.897** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB.Structure;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using System;
using System.Collections.Generic;
using System.Linq;

public class FilterAndColorOverride
{
    public static void CreateParameterFilterAndApply(Document doc, View view)
    {
        try
        {
            using (Transaction transaction = new Transaction(doc, "Create Filter and Apply"))
            {
                transaction.Start();

                // Create filter rules
                FilterRule rule1 = ParameterFilterRuleFactory.CreateEqualsRule(new ElementId(BuiltInParameter.ALL_MODEL_MARK), "FILTER1");
                FilterRule rule2 = ParameterFilterRuleFactory.CreateEqualsRule(new ElementId(BuiltInParameter.ALL_MODEL_FAMILY_NAME), "FILTER2");

                // Create filter
                FilterElement filter = FilterElement.Create(doc, "My Filter", new List<FilterRule> { rule1, rule2 });

                // Apply filter to view
                view.AddFilter(filter.Id);

                // Set color overrides
                OverrideGraphicSettings ogs = new OverrideGraphicSettings();
                ogs.SetProjectionLineColor(new Color(255, 0, 0)); // Red color
                view.SetElementOverrides(filter.Id, ogs);

                transaction.Commit();
            }
        }
        catch (Exception ex)
        {
            TaskDialog.Show("Error", ex.Message);
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void CreateAndApplyViewFilter(Document doc)
{
    View activeView = doc.ActiveView;

    using (Transaction tx = new Transaction(doc, "Create View Filter"))
    {
        tx.Start();

        IList<ElementId> categories = new List<ElementId>
        {
            new ElementId(BuiltInCategory.OST_Walls)
        };

        ElementId commentsId = new ElementId(BuiltInParameter.ALL_MODEL_INSTANCE_COMMENTS);
        FilterRule rule = ParameterFilterRuleFactory.CreateContainsRule(
            commentsId, "Temporary", false);

        ElementParameterFilter elemFilter = new ElementParameterFilter(rule);

        ParameterFilterElement filterElem = ParameterFilterElement.Create(
            doc, "Temporary Walls Filter", categories, elemFilter);

        activeView.AddFilter(filterElem.Id);
        activeView.SetFilterVisibility(filterElem.Id, true);

        OverrideGraphicSettings ogs = new OverrideGraphicSettings();
        ogs.SetProjectionLineColor(new Color(255, 0, 0));
        activeView.SetFilterOverrides(filterElem.Id, ogs);

        tx.Commit();
    }
}
```

</details>

---

### RC35 | Hard | Winner: revit-coder-14b (+0.165)

**Question:** Write C# code using ElementIntersectsElementFilter to find all elements that physically intersect a selected element. Get the selected element's solid geometry, create the intersection filter, and return a list of intersecting ElementIds with their category names. Write code only. No explanation.

**Expected signals:** `ElementIntersectsElementFilter`, `FilteredElementCollector`, `Solid`, `GeometryElement`, `Options`, `ExclusionFilter`, `ElementId`, `Category`, `Name`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.556 | 0.556 |
| Quality | 0.917 | 0.417 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.745** | **0.580** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;

public class ElementIntersectFinder : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        try
        {
            UIApplication uiApp = commandData.Application;
            UIDocument uiDoc = uiApp.ActiveUIDocument;
            Document doc = uiDoc.Document;

            Reference refSelected = uiDoc.Selection.PickObject(ObjectType.Element, "Select an element to find intersecting elements");
            Element selectedElement = doc.GetElement(refSelected);

            Solid selectedSolid = selectedElement.GetGeometryObjectFromReference(refSelected) as Solid;
            if (selectedSolid == null)
            {
                message = "Selected element does not have solid geometry.";
                return Result.Failed;
            }

            ElementIntersectsElementFilter intersectFilter = new ElementIntersectsElementFilter(selectedSolid);
            FilteredElementCollector collector = new FilteredElementCollector(doc);
            collector.WherePasses(intersectFilter);

            List<string> intersectingElements = new List<string>();
            foreach (Element intersectingElement in collector)
            {
                intersectingElements.Add($"{intersectingElement.Id} - {intersectingElement.Category.Name}");
            }

            TaskDialog.Show("Intersecting Elements", string.Join("\n", intersectingElements));
            return Result.Succeeded;
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public List<(ElementId Id, string CategoryName)> FindIntersectingElements(Document doc, Element selectedElement)
{
    var result = new List<(ElementId, string)>();

    ElementIntersectsElementFilter intersectFilter =
        new ElementIntersectsElementFilter(selectedElement);

    var intersecting = new FilteredElementCollector(doc)
        .WherePasses(intersectFilter)
        .ToElements();

    foreach (var elem in intersecting)
    {
        if (elem.Id == selectedElement.Id) continue;

        string catName = elem.Category?.Name ?? "No Category";
        result.Add((elem.Id, catName));
    }

    return result;
}
```

</details>

---

### RC36 | Hard | Winner: revit-coder-14b (+0.090)

**Question:** Write C# code to open a family document for editing using doc.EditFamily, use FamilyManager to add a new type parameter of type Length, set the parameter value, then close the family document and reload it into the project using LoadFamily. Write code only. No explanation.

**Expected signals:** `EditFamily`, `FamilyManager`, `AddParameter`, `FamilyDocument`, `LoadFamily`, `FamilyParameter`, `Set`, `Transaction`, `BuiltInParameterGroup`, `ParameterType`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.800 | 0.700 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.868** | **0.777** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Family;
using Autodesk.Revit.UI;

public void OpenAndEditFamily(Document doc, string familyPath)
{
    try
    {
        // Open family document for editing
        FamilySymbol familySymbol = null;
        using (Transaction transaction = new Transaction(doc, "Open Family"))
        {
            transaction.Start();
            familySymbol = doc.LoadFamily(familyPath, new FamilyLoadOptions());
            transaction.Commit();
        }

        // Get family document
        Document familyDoc = familySymbol.Family.Document;

        // Get family manager
        FamilyManager familyManager = familyDoc.FamilyManager;

        // Add a new type parameter of type Length
        familyManager.AddParameter("Length Parameter", BuiltInParameterGroup.PG_LENGTH, ParameterType.Length);

        // Set the parameter value
        familyManager.CurrentType.Set(familyManager.get_Parameter("Length Parameter"), UnitUtils.ConvertToInternalUnits(10, DisplayUnitType.DUT_FEET));

        // Close the family document
        familyDoc.Close(false);

        // Reload the family into the project
        using (Transaction transaction = new Transaction(doc, "Reload Family"))
        {
            transaction.Start();
            familySymbol = doc.LoadFamily(familyPath, new FamilyLoadOptions());
            transaction.Commit();
        }
    }
    catch (Exception ex)
    {
        // Handle any exceptions
        TaskDialog.Show("Error", ex.Message);
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void EditFamilyAndReload(Document doc, FamilyInstance instance)
{
    Family family = instance.Symbol.Family;
    Document familyDoc = doc.EditFamily(family);

    if (familyDoc == null) return;

    using (Transaction ftx = new Transaction(familyDoc, "Add Parameter"))
    {
        ftx.Start();

        FamilyManager fm = familyDoc.FamilyManager;

        FamilyParameter param = fm.AddParameter(
            "CustomLength",
            GroupTypeId.Dimensions,
            SpecTypeId.Length,
            true);

        if (fm.CurrentType == null)
            fm.NewType("Default");

        fm.Set(param, UnitUtils.ConvertToInternalUnits(500, UnitTypeId.Millimeters));

        ftx.Commit();
    }

    // Reload into project
    familyDoc.LoadFamily(doc, new FamilyLoadOptions());
    familyDoc.Close(false);
}

public class FamilyLoadOptions : IFamilyLoadOptions
{
    public bool OnFamilyFound(bool familyInUse, out bool overwriteParameterValues)
    {
        overwriteParameterValues = true;
        return true;
    }

    public bool OnSharedFamilyFound(Family sharedFamily, bool familyInUse,
        out FamilySource source, out bool overwriteParameterValues)
    {
        source = FamilySource.Family;
        overwriteParameterValues = true;
        return true;
    }
}
```

</details>

---

### RC37 | Hard | Winner: revit-coder-14b (+0.050)

**Question:** Write C# code using a TransactionGroup to wrap multiple sub-transactions. Perform two separate modifications (e.g., move a wall then change its type), and demonstrate rolling back the entire group if the second sub-transaction fails. Include TransactionGroup.Assimilate on success. Write code only. No explanation.

**Expected signals:** `TransactionGroup`, `Transaction`, `Start`, `Commit`, `RollBack`, `Assimilate`, `TransactionStatus`, `t.Start`, `t.Commit`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.667 | 0.667 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.825 | 0.825 |
| **Composite** | **0.814** | **0.764** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;

public class WallTypeChanger : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        UIApplication uiApp = commandData.Application;
        UIDocument uidoc = uiApp.ActiveUIDocument;
        Document doc = uidoc.Document;

        try
        {
            // Start a transaction group
            using (TransactionGroup tg = new TransactionGroup(doc))
            {
                tg.Start("Wall Type Change");

                // First sub-transaction: Move wall
                using (Transaction t1 = new Transaction(doc, "Move Wall"))
                {
                    t1.Start();

                    // Select wall
                    Reference ref1 = uidoc.Selection.PickObject(ObjectType.Element, "Select a wall to move");
                    ElementId wallId = ref1.ElementId;
                    Element wall = doc.GetElement(wallId);

                    // Move wall
                    LocationCurve location = wall.Location as LocationCurve;
                    if (location != null)
                    {
                        XYZ newPoint = location.Curve.GetEndPoint(0) + new XYZ(10, 0, 0);
                        location.Move(newPoint);
                    }

                    t1.Commit();
                }

                // Second sub-transaction: Change wall type
                using (Transaction t2 = new Transaction(doc, "Change Wall Type"))
                {
                    t2.Start();

                    // Select new wall type
                    Reference ref2 = uidoc.Selection.PickObject(ObjectType.Element, "Select a new wall type");
                    ElementId typeId = ref2.ElementId;
                    ElementType newType = doc.GetElement(typeId) as ElementType;

                    // Change wall type
                    Element wall = doc.GetElement(wallId);
                    wall.ChangeTypeId(newType.Id);

                    // Intentionally fail to demonstrate rollback
                    throw new Exception("Failed to change wall type");

                    t2.Commit();
                }

                // If we reach here, both sub-transactions succeeded
                tg.Assimilate();
            }
        }
        catch (Exception ex)
        {
            // If any sub-transaction fails, rollback the entire group
            message = ex.Message;
            return Result.Failed;
        }

        return Result.Succeeded;
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void TransactionGroupExample(Document doc, Wall wall, WallType newType)
{
    using (TransactionGroup txGroup = new TransactionGroup(doc, "Grouped Operations"))
    {
        txGroup.Start();

        bool success = true;

        // Sub-transaction 1: Move the wall
        using (Transaction tx1 = new Transaction(doc, "Move Wall"))
        {
            tx1.Start();
            try
            {
                ElementTransformUtils.MoveElement(doc, wall.Id, new XYZ(5, 0, 0));
                tx1.Commit();
            }
            catch
            {
                tx1.RollBack();
                success = false;
            }
        }

        // Sub-transaction 2: Change wall type
        if (success)
        {
            using (Transaction tx2 = new Transaction(doc, "Change Wall Type"))
            {
                tx2.Start();
                try
                {
                    wall.WallType = newType;
                    tx2.Commit();
                }
                catch
                {
                    tx2.RollBack();
                    success = false;
                }
            }
        }

        if (success)
        {
            txGroup.Assimilate();
        }
        else
        {
            txGroup.RollBack();
            TaskDialog.Show("Error", "Operation rolled back.");
        }
    }
}
```

</details>

---

### RC38 | Hard | Winner: Claude Opus 4.6 (-0.049)

**Question:** Write C# code to export the active 3D view to DWG format using doc.Export. Configure DWGExportOptions with specific settings for layers, colors, and file version. Specify the output folder path and filename. Write code only. No explanation.

**Expected signals:** `Export`, `DWGExportOptions`, `ExportColorMode`, `PropOverrides`, `LayerMapping`, `FileVersion`, `View3D`, `Transaction`, `ViewSet`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.556 | 0.778 |
| Quality | 0.917 | 0.833 |
| Completeness | 0.825 | 0.775 |
| **Composite** | **0.745** | **0.794** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Export;
using Autodesk.Revit.UI;

public class ExportToDwgCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        try
        {
            UIDocument uidoc = commandData.Application.ActiveUIDocument;
            Document doc = uidoc.Document;

            // Get the active 3D view
            View3D active3DView = null;
            foreach (View3D view in new FilteredElementCollector(doc).OfClass(typeof(View3D)).ToElementList())
            {
                if (view.IsWorksetVisible)
                {
                    active3DView = view;
                    break;
                }
            }

            if (active3DView == null)
            {
                message = "No active 3D view found.";
                return Result.Failed;
            }

            // Define output path and filename
            string outputFolder = @"C:\ExportOutput";
            string filename = "ExportedView.dwg";
            string fullPath = System.IO.Path.Combine(outputFolder, filename);

            // Create DWG export options
            DWGExportOptions options = new DWGExportOptions();
            options.ExportObjectData = true;
            options.ExportLayerInformation = true;
            options.LayerMapping = LayerMappingType.ByView;
            options.ColorMapping = ColorMappingType.ByView;
            options.FileVersion = DWGVersion.AutoDetect;
            options.ExportLinetypeInformation = true;
            options.ExportLinetypeAsByLayer = true;
            options.ExportUTF8Encoding = true;

            // Export the view
            bool exportSuccess = doc.Export(outputFolder, filename, active3DView, options);

            if (exportSuccess)
            {
                return Result.Succeeded;
            }
            else
            {
                message = "Export to DWG failed.";
                return Result.Failed;
            }
        }
        catch (Exception ex)
        {
            message = ex.Message;
            return Result.Failed;
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void ExportToDwg(Document doc)
{
    View3D view3d = new FilteredElementCollector(doc)
        .OfClass(typeof(View3D))
        .Cast<View3D>()
        .FirstOrDefault(v => !v.IsTemplate);

    if (view3d == null) return;

    DWGExportOptions options = new DWGExportOptions();
    options.FileVersion = ACADVersion.R2018;
    options.ExportOfSolids = SolidGeometry.ACIS;
    options.LayerMapping = "AIA";
    options.Colors = ExportColorMode.IndexColors;
    options.MergedViews = true;
    options.HideUnreferenceViewTags = true;

    string outputFolder = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
    string fileName = "Export_" + view3d.Name;

    ICollection<ElementId> viewIds = new List<ElementId> { view3d.Id };

    using (Transaction tx = new Transaction(doc, "Export DWG"))
    {
        tx.Start();
        doc.Export(outputFolder, fileName, viewIds, options);
        tx.Commit();
    }
}
```

</details>

---

### RC39 | Hard | Winner: revit-coder-14b (+0.050)

**Question:** Write C# code to find wall endpoints using LocationCurve, then use WallUtils.DisallowWallJoinAtEnd and WallUtils.AllowWallJoinAtEnd to control wall join behavior at each end. Iterate through all walls in the model and disallow joins at both ends inside a Transaction. Write code only. No explanation.

**Expected signals:** `WallUtils`, `DisallowWallJoinAtEnd`, `AllowWallJoinAtEnd`, `LocationCurve`, `Wall`, `FilteredElementCollector`, `Transaction`, `Curve`, `GetEndPoint`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.889 | 0.889 |
| Quality | 1.000 | 0.833 |
| Completeness | 0.775 | 0.775 |
| **Composite** | **0.888** | **0.838** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.UI;
using System.Collections.Generic;

public class WallJoinControl
{
    public static void Execute(Document doc)
    {
        using (Transaction transaction = new Transaction(doc, "Control Wall Joins"))
        {
            transaction.Start();

            FilteredElementCollector collector = new FilteredElementCollector(doc);
            collector.OfClass(typeof(Wall));

            foreach (Wall wall in collector)
            {
                LocationCurve locationCurve = wall.Location as LocationCurve;
                if (locationCurve != null)
                {
                    // Disallow joins at both ends
                    WallUtils.DisallowWallJoinAtEnd(wall, 0);
                    WallUtils.DisallowWallJoinAtEnd(wall, 1);
                }
            }

            transaction.Commit();
        }
    }
}
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public void DisableAllWallJoins(Document doc)
{
    var walls = new FilteredElementCollector(doc)
        .OfClass(typeof(Wall))
        .WhereElementIsNotElementType()
        .Cast<Wall>()
        .ToList();

    using (Transaction tx = new Transaction(doc, "Disable Wall Joins"))
    {
        tx.Start();

        foreach (Wall wall in walls)
        {
            LocationCurve locCurve = wall.Location as LocationCurve;
            if (locCurve == null) continue;

            WallUtils.DisallowWallJoinAtEnd(wall, 0);
            WallUtils.DisallowWallJoinAtEnd(wall, 1);
        }

        tx.Commit();
    }
}
```

</details>

---

### RC40 | Hard | Winner: Claude Opus 4.6 (-0.112)

**Question:** Write C# code to build a chained logical filter combining an ElementCategoryFilter for walls, an ElementParameterFilter on the Comments parameter, and a BoundingBoxIntersectsFilter with a specified Outline. Combine them using LogicalAndFilter and apply to a FilteredElementCollector. Write code only. No explanation.

**Expected signals:** `LogicalAndFilter`, `ElementCategoryFilter`, `ElementParameterFilter`, `BoundingBoxIntersectsFilter`, `Outline`, `FilteredElementCollector`, `BuiltInCategory.OST_Walls`, `ParameterFilterRuleFactory`, `XYZ`

| Component | revit-coder-14b | Claude Opus 4.6 |
|-----------|-----------------|-----------------|
| Signal | 0.889 | 1.000 |
| Quality | 0.417 | 0.417 |
| Completeness | 0.775 | 1.000 |
| **Composite** | **0.713** | **0.825** |

<details>
<summary>revit-coder-14b response</summary>

```csharp
var wallFilter = new ElementCategoryFilter(BuiltInCategory.OST_Walls);
var commentFilter = new ElementParameterFilter(new FilterStringContains(new ElementId(BuiltInParameter.ALL_MODEL_COMMENTS), "important"));
var outline = new Outline(new XYZ(0, 0, 0), new XYZ(10, 10, 10));
var boundingBoxFilter = new BoundingBoxIntersectsFilter(outline);
var combinedFilter = new LogicalAndFilter(wallFilter, commentFilter, boundingBoxFilter);
var collector = new FilteredElementCollector(doc).WherePasses(combinedFilter);
```

</details>

<details>
<summary>Claude Opus 4.6 response</summary>

```csharp
public List<Element> GetFilteredWalls(Document doc)
{
    // Filter 1: Category filter for walls
    ElementCategoryFilter categoryFilter =
        new ElementCategoryFilter(BuiltInCategory.OST_Walls);

    // Filter 2: Parameter filter on Comments
    ElementId commentsId = new ElementId(BuiltInParameter.ALL_MODEL_INSTANCE_COMMENTS);
    FilterRule rule = ParameterFilterRuleFactory.CreateContainsRule(
        commentsId, "Structural", false);
    ElementParameterFilter paramFilter = new ElementParameterFilter(rule);

    // Filter 3: Bounding box intersects filter
    XYZ min = new XYZ(-100, -100, -10);
    XYZ max = new XYZ(100, 100, 50);
    Outline outline = new Outline(min, max);
    BoundingBoxIntersectsFilter bboxFilter = new BoundingBoxIntersectsFilter(outline);

    // Chain with LogicalAndFilter
    LogicalAndFilter combinedFilter1 = new LogicalAndFilter(categoryFilter, paramFilter);
    LogicalAndFilter combinedFilter2 = new LogicalAndFilter(combinedFilter1, bboxFilter);

    List<Element> result = new FilteredElementCollector(doc)
        .WhereElementIsNotElementType()
        .WherePasses(combinedFilter2)
        .ToElements()
        .ToList();

    return result;
}
```

Confidence is high on these. The only one I'd sanity-check is rc36, specifically the `GroupTypeId.Dimensions` and `SpecTypeId.Length` parameter for `FamilyManager.AddParameter` since that signature changed between Revit 2022 and 2025 with the ForgeTypeId migration.

</details>

---
