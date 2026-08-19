# Product Requirements Document (PRD)

**Product:** Program ↔ Geometry Sync (working title: *Live Program*)  
**Version:** V1  
**Status:** Draft for engineering  
**Date:** 19 August 2026  
**Audience:** Software engineers with **no architecture background**  
**Related research:** Internal market analysis of Snaptrude and the AEC early-design stack (see §18)

---

## 0. How to read this document if you are not an architect

This product is **not** Autodesk Revit, SketchUp, or a house renderer.

It is closer to:

> **A spreadsheet of rooms + a 3D canvas of colored boxes that always show the same numbers.**

If you have built:

- Google Sheets + a live chart  
- Figma + a properties panel  
- A CMS admin table + a preview  

…you already understand the pattern. The domain words are just names for rows, groups, and dimensions.

| Word they use | What it means in software |
|---|---|
| **Program** / **space program** / **accommodation schedule** | A table of required rooms (Excel). The *requirements list*, not the 3D file. |
| **Department** / **function** | A grouping column (e.g. “Classrooms”, “Bedrooms”). Like a folder or tag. |
| **Room / space** | One row. Has a name, count, and target area. |
| **Target area / programmed area** | The number the client asked for (e.g. 250 sq ft). |
| **Achieved / designed area** | The actual area of the 3D box after layout (`width × depth`). |
| **Variance** | `achieved − target`. Green if close, red if over/under. |
| **Massing** | Simple 3D blocks. Not walls, doors, or furniture. |
| **Schematic design (SD)** | Early design phase: “does this program fit?” — our V1 phase. |
| **BIM** | Building Information Model: 3D + data (Revit, IFC). We only *export* a thin BIM in V1. |
| **IFC** | Open file format (`.ifc`) that Revit/ArchiCAD can import. Not Autodesk-owned. |
| **Revit** | Autodesk’s closed BIM app. We do **not** edit `.rvt` files in V1. |
| **Net area** | Usable floor area inside a room. V1 uses this as “the number on the box.” |
| **Gross area / FAR** | Whole-building metrics. **Out of scope for V1.** |
| **Adjacency** | “Kitchen should be next to dining.” Optional hint in V1, not a solver. |
| **LOD** | Level of Detail of a BIM model. V1 is roughly “boxes with names” (LOD 100-ish). |

**What V1 does *not* do:** design a pretty house, generate construction drawings, clash-detect pipes, or replace Revit.

---

## 1. One-sentence product

**Paste (or type) an Excel space program → get a live 3D layout of rooms as boxes → change either the table or the 3D and both stay in sync → export JSON / Excel / IFC.**

Pitch line:

> *“Paste your program spreadsheet. Get a live 3D model that stays in sync. Stop rebuilding the same building three times.”*

---

## 2. Problem statement

### 2.1 The industry workflow today

Architecture firms typically keep the **same project in three disconnected tools**:

1. **Excel** — room list, areas, departments  
2. **SketchUp / Rhino** — fast 3D massing (geometry with little or no room data)  
3. **Revit** — BIM for later documentation (heavy, slow for early iteration)

When the client changes one room size, someone updates Excel, someone rebuilds SketchUp, someone reconciles Revit. That loop is called the **double data entry tax**.

Snaptrude’s published analysis of this fragmentation:  
https://www.snaptrude.com/blog/the-double-data-entry-tax-why-architecture-firms-maintain-two-models  

Related Snaptrude posts (same problem, different angles):

- Program spreadsheet as design intent: https://www.snaptrude.com/blog/program-spreadsheet-bim-design-intent  
- Excel → BIM layout: https://www.snaptrude.com/blog/excel-to-bim-ai-layout-workflow  
- Custom program sheets ↔ BIM: https://www.snaptrude.com/blog/how-to-custom-program-sheets-bim-sync-revit  
- Parallel SketchUp + Revit cost: https://www.snaptrude.com/blog/parallel-workflow-tax-sketchup-revit  

### 2.2 What practitioners say (not Snaptrude marketing)

**Archinect forum — Revit is for CDs, not early design**  
Thread: https://archinect.com/forum/thread/150377816/revit  

Quoted practitioner positions (paraphrased from that thread):

- Revit is a strong **construction document** tool; **early design** is faster in SketchUp / hand / other tools.  
- Firm policy example: tool-agnostic until “Design Approval,” then everything goes into Revit.  
- Revit “forces over-design”: you must specify too much too early, which kills iteration.  
- Revit-fluent juniors were **slower to design approval** than staff using SketchUp/hand.

**Revit Forum — SketchUp → Revit massing is a bad iterative workflow**  
https://www.revitforum.org/forum/revit-architecture-forum-rac/architecture-conceptual-massing-and-adaptive-components/23832-revit-cannot-import-sketchup-file  

Practitioners warn that importing SketchUp into Revit masses is initially fast and then a “trainwreck” through iterations.

**Revit space planning (why SD in Revit hurts)**  
https://bimheroes.com/architectural-space-planning/  

Teams put too much detail too early; layout changes become expensive; program is still unstable.

**Excel ↔ Revit is a known unsolved native gap**

- Autodesk: no first-class Excel link into Revit: https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-to-import-excel-sheet-into-Revit.html  
- Autodesk Community (space management from a database): https://forums.autodesk.com/t5/revit-architecture-forum/database-connections-and-space-management/td-p/11145439  
- People write **Dynamo scripts** for Excel ↔ rooms: https://parametrix.gitbooks.io/dynamo-revit-recipes/content/02_Roundtrip-to-Excel/2-1_roundtrip-to-excel.html  
- LinkedIn: need Excel to **create/delete** rooms, not only update properties: https://www.linkedin.com/pulse/connect-room-dataexcelrevit-revitexcel-different-juli-hariyanto  

**Independent Snaptrude reviews (what users praise vs gap)**

- ArchiGen (hands-on): program logic where a “room knows it is a room”; CD/docs still weak: https://archigenai.com/snaptrude-ai-bim-review-architects-2026.html  
- AECO.digital: almost no G2/Capterra reviews; treat as conceptual BIM, not Revit replacement: https://aeco.digital/snaptrude-review-2026/  
- CheckThat: G2 empty; community talks on forums not review sites: https://checkthat.ai/brands/snaptrude/reviews  

### 2.3 Why this is the V1 wedge

Competitors split the early-stage market:

| Product | What they optimize | Link |
|---|---|---|
| Autodesk Forma | Site / sun / wind / noise | https://www.autodesk.com/products/forma |
| TestFit | Yield, parking, unit mix (developer math) | https://www.testfit.io/ |
| SketchUp | Fast dumb geometry | https://www.sketchup.com/ |
| Revit | Construction documentation | https://www.autodesk.com/products/revit |
| Hypar | Programmable building functions | https://hypar.io/ |
| dRofus | Room program database ↔ Revit (enterprise, not a 3D massing editor) | https://www.drofus.com/ |

**Nobody owns “the Excel IS the model” as a simple, browser-native, bidirectional editor** at a small-team price. That is V1.

Snaptrude’s own product is built around four modes (Program, Design, BIM, Present) with Program as the live spreadsheet: https://www.snaptrude.com/  
CEO interview on four connected modes: https://aecmag.com/bim/snaptrude-on-ai/

We are **not** cloning all four modes. We clone **the Program ↔ geometry link only**.

---

## 3. Goals and non-goals

### 3.1 V1 goals (must be true)

1. A user can import a typical Excel program (or type it in-app) without being an architect.  
2. The app **creates** 3D room boxes from that table (does not require a Revit file).  
3. Changing **target area** in the table **resizes** the 3D box (deterministic math).  
4. Dragging/resizing a box in 3D **updates** achieved area in the table.  
5. A dashboard always shows **target vs achieved** by room and by department.  
6. User can export the same data back to Excel/CSV and a simple IFC (rooms as `IfcSpace`).  
7. Entire V1 can run **in the browser** with optional Python only for IFC authoring if we choose IfcOpenShell.

### 3.2 Explicit non-goals (do not build in V1)

| Out of scope | Why |
|---|---|
| Native `.rvt` read/write | Closed format; no good OSS editor. Industry uses IFC/Speckle instead. |
| AI that reads RFPs / PDFs | Unreliable; trust issues; commoditizing. Add in V2. |
| Real walls, doors, windows, furniture | That is a floor-plan CAD product. Boxes first. |
| Construction documents / sheets / dimensions | Snaptrude’s documented gap; Revit’s job. |
| Multiplayer / CRDT | Collaboration is a growth feature, not the wedge. |
| Site analysis, zoning, solar | Forma / TestFit territory. |
| Photorealistic rendering | Enscape / Lumion / Midjourney. |
| Clash detection, MEP, structure | Downstream BIM. |
| Offline desktop installer | Browser SPA is enough. |

### 3.3 V1 success (product)

A 5-minute demo:

1. Upload sample Excel.  
2. Boxes appear on a floor.  
3. Change one cell → box grows.  
4. Drag a box → table numbers update.  
5. Variance turns red if over budget of area.  
6. Download `.xlsx` and `.ifc`.

If that demo is not magic, V1 is not done.

---

## 4. Users and jobs-to-be-done

We are not selling to “architects” as a blob. V1 jobs:

| Persona | Job | What they do in V1 |
|---|---|---|
| **Project architect / designer** | Keep program and layout aligned during SD | Import Excel, shuffle boxes, check variance |
| **Small firm principal** | Show client “your brief vs this layout” | Share screen; export Excel |
| **Junior / intern** | Stop copy-pasting areas from Excel into SketchUp | Use our table as the only list |
| **Us (engineering)** | Ship without AEC domain experts | Follow data model + glossary in this PRD |

**Not a V1 buyer:** BIM manager of a 200-person firm needing SSO, Revit family libraries, and CD sets.

---

## 5. User stories (V1)

1. As a user, I can create a blank project with units (sq ft or m²) and a rectangular floor plate size.  
2. As a user, I can import `.xlsx` / `.csv` and **map columns** (Room Name, Department, Count, Target Area, Floor).  
3. As a user, I can edit the program in a spreadsheet-like grid (add/delete rows, edit cells).  
4. As a user, I can see 3D (and top-down 2D) colored boxes, one per room instance.  
5. As a user, when I change Target Area, the box area updates using a fixed aspect-ratio rule (see §8.3).  
6. As a user, when I scale a box by dragging a handle, Target Area updates (or Achieved only — see decision in §8.3).  
7. As a user, I can select a room in 3D and see the same row highlighted in the table (and vice versa).  
8. As a user, I see department rollups and project totals with variance thresholds.  
9. As a user, I can undo/redo layout and table edits.  
10. As a user, I can save/load a project as JSON.  
11. As a user, I can export Excel and IFC.  
12. As a user, I can use a **sample project** (house + school snippet) without uploading a file.

---

## 6. Domain data model (source of truth)

**Single source of truth is JSON in memory** (Zustand/Redux). Excel and 3D are *views*.

### 6.1 Types (TypeScript sketch)

```ts
type Units = "sqft" | "m2";

interface Project {
  id: string;
  name: string;
  units: Units;
  floors: Floor[];
  rooms: Room[];
  settings: {
    gridSize: number;          // snap, in feet or meters
    defaultCeilingHeight: number;
    varianceWarnPct: number;   // e.g. 5
    varianceFailPct: number;   // e.g. 10
    /** When table targetArea changes, how to resize the box */
    resizePolicy: "keepAspect" | "keepWidth" | "keepDepth";
  };
}

interface Floor {
  id: string;
  name: string;       // "Level 1"
  elevation: number;  // 0, 10, 20...
  plateWidth: number;
  plateDepth: number;
}

interface Room {
  id: string;                 // uuid
  name: string;
  department: string;
  floorId: string;
  countOrigin?: string;       // if expanded from count>1, parent group id
  targetArea: number;         // programmed
  /** Geometry on the floor plate (plan) */
  x: number;
  y: number;
  width: number;
  depth: number;
  height: number;             // extrusion for 3D (ceiling)
  color?: string;             // derived from department if omitted
  notes?: string;
  adjacencyTags?: string[];   // V1: stored, not solved
}

function achievedArea(r: Room): number {
  return r.width * r.depth;   // plan area; ignore height
}
```

**Count expansion:** If Excel says Guest Room × 2, V1 creates **two Room records** with the same name + department, distinct ids, both with `targetArea` of one guest room.

### 6.2 Canonical Excel columns (import mapping)

Industry practice (dRofus minimum for Excel room import): **function/department, room name, programmed area, number of rooms**.  
Source: https://support.drofus.com/en/support/solutions/articles/16000078076-importing-room-space-program-from-excel  

dRofus also distinguishes **programmed vs designed** area (exactly our target vs achieved):  
https://drofus.atlassian.net/wiki/spaces/DV/pages/1318360545/Room-Properties  

**V1 required mapped fields:**

| Logical field | Example headers we auto-detect (case-insensitive) |
|---|---|
| `name` | Room Name, Space, Room, Name |
| `department` | Department, Function, Dept, Category |
| `targetArea` | Target Area, Programmed Area, Net Area, Area, NSF, sqft, m2 |
| `count` | Count, Qty, Number, Number of Rooms |
| `floor` | Floor, Level, Storey |

Unknown columns → stored in `notes` or ignored with a warning list.

**Sample CSV (include in repo as `fixtures/sample-house.csv`):**

```csv
Department,Room Name,Count,Target Area,Floor
Bedrooms,Master Bedroom,1,250,1
Bedrooms,Guest Bedroom,2,140,1
Kitchen,Kitchen,1,180,1
Living,Living Room,1,320,1
Bathrooms,Full Bath,1,80,1
Bathrooms,Powder Room,1,30,1
Utility,Laundry,1,40,1
Circulation,Hall,1,60,1
```

### 6.3 What Excel is *not*

Excel is **not** a BIM file. It does not contain walls, materials, or coordinates until *we* write x/y/width/depth after layout.

---

## 7. Functional requirements (detailed)

### FR-1 Project bootstrap

- Create project: name, units, one default floor, default plate (e.g. 80 × 60 ft).  
- User can edit plate size; rooms that fall outside get a warning (not auto-deleted).

### FR-2 Import

- Accept `.xlsx`, `.xls`, `.csv`.  
- Library: **SheetJS Community** https://github.com/SheetJS/sheetjs (also `xlsx` on npm).  
- Column mapper UI: dropdown per detected header.  
- Validation: empty names, non-numeric area, count < 1, duplicate names allowed.  
- Preview table before commit.

### FR-3 Spreadsheet editor

Use an Excel-like grid:

- **Handsontable** (commercial license for production SaaS — check): https://github.com/handsontable/handsontable  
- **AG Grid Community**: https://github.com/ag-grid/ag-grid  

**V1 recommendation:** AG Grid Community to avoid license risk; Handsontable if we want closer Excel UX and can license.

Grid columns: Department, Name, Count (display-only after expand), Target Area, Achieved Area (read-only formula), Variance, Floor, Notes.

Inline add row / delete row.

### FR-4 Layout generation (Excel → 3D)

On first import (and “Re-pack layout” button):

Run **deterministic shelf / row packing** (see §8.2).

User can lock a room (`locked: true`) so re-pack skips it (nice-to-have; if time-boxed, skip locks).

### FR-5 3D / 2D canvas

- Library: **Three.js** https://github.com/mrdoob/three.js  
- React: **React Three Fiber** https://github.com/pmndrs/react-three-fiber  
- Helpers: **@react-three/drei** (OrbitControls, Gizmo, Html labels)

**Reference editors (study, do not copy code blindly):**

- open3dFloorplan (SvelteKit + Three.js 2D/3D floorplan): https://github.com/theLodgeBots/open3dFloorplan  
- Aedifex (React + R3F + Zustand architectural editor): https://github.com/TangSY/aedifex  
- Smarchitect (AI floor plans + editor; heavier): https://github.com/Salar24/Smarchitect  

**V1 interactions:**

- Orbit / pan / zoom  
- Top view camera preset (plan)  
- Click select  
- Drag move on XY (snap to grid)  
- Corner or edge handles to resize width/depth  
- Color by department (hash color)  
- Label: name + achieved area  

**Do not** implement CSG walls (Aedifex’s WallSystem) in V1.

### FR-6 Bidirectional sync rules

See §8. Entire product quality lives here.

### FR-7 Totals dashboard

- Project target sum, achieved sum, variance, %  
- Per-department subtotals  
- Color: within warn % → yellow; beyond fail % → red; else green  
- List of rooms over/under

### FR-8 Persistence

- Save as `.json` (download)  
- Load `.json`  
- `localStorage` autosave last project (quota-safe)

### FR-9 Export Excel

- SheetJS write workbook with Target + Achieved + Variance + x,y,width,depth  

### FR-10 Export IFC (V1-minimum)

Create IFC4 file with:

- `IfcProject` / `IfcBuilding` / `IfcBuildingStorey`  
- One `IfcSpace` per room with Name, LongName, and a rectangular footprint extruded or mapped as simple geometry  
- Property: programmed vs designed area if possible (`Qto_SpaceBaseQuantities` / `Pset_SpaceCommon`)

**Libraries:**

- **IfcOpenShell** (Python, most mature authoring): https://github.com/IfcOpenShell/IfcOpenShell  
  Site: https://ifcopenshell.org/  
  High-level API docs: create entities, spatial containment  
- **IfcCSV** (Excel ↔ IFC properties roundtrip, not full geometry): https://docs.ifcopenshell.org/ifccsv.html  
  PyPI: https://pypi.org/project/ifccsv/  
- **@ifc-lite/create** (JS create IFC from scratch): https://github.com/louistrue/ifc-lite  
  Docs: https://ltplus-ag.github.io/ifc-lite/  
- **web-ifc** (WASM read/write IFC): https://github.com/ThatOpen/engine_web-ifc  

**V1 recommendation:** Python microservice **or** serverless function using IfcOpenShell for export; keep editor 100% TS. If we must stay JS-only, spike `@ifc-lite/create` in week 1.

### FR-11 Optional: view existing IFC as ghost (stretch)

**Not required for V1 demo.** If time:

- **@thatopen/components** + **web-ifc**: https://github.com/ThatOpen/engine_components  
  Docs: https://docs.thatopen.com/Tutorials/Components/Core/IfcLoader  
  Getting started: https://docs.thatopen.com/components/getting-started  
- Load IFC as **read-only** underlay; our boxes stay the editable layer.  
- IfcSpace coloring caveats: https://github.com/ThatOpen/engine_components/issues/713  

Do **not** try to round-trip edit the imported IFC in V1.

### FR-12 Accessibility & UX basics

- Keyboard: Delete selected room, Undo (Ctrl+Z)  
- Units displayed in labels  
- Empty states with sample file download  

---

## 8. Sync engine (the core)

### 8.1 Architecture

```
Excel grid  ──dispatch──►  ProjectStore  ◄──dispatch──  3D scene
                │                 │
                │                 ├── derived: achievedArea, totals
                │                 └── layoutEngine (only on import / re-pack)
                ▼
         React re-render both views
```

Use **Zustand** https://github.com/pmndrs/zustand + **zundo** https://github.com/charkour/zundo (Aedifex uses this pattern).

All mutations go through named actions:

- `importProgram(rows)`  
- `updateRoomFields(id, patch)`  
- `moveRoom(id, x, y)`  
- `resizeRoom(id, width, depth, source: "table" | "gizmo")`  
- `addRoom` / `deleteRoom`  
- `repackFloor(floorId)`  

### 8.2 Layout algorithm (deterministic)

**V1 algorithm: shelf packing (left-to-right, wrap to next row)**

1. Sort rooms on a floor by `targetArea` descending.  
2. For each room, set `width = sqrt(targetArea * aspect)` with default aspect `1.25` (slightly rectangular).  
   `depth = targetArea / width`.  
3. Place at cursor `(cx, cy)` if `cx + width <= plateWidth`; else wrap: `cy += rowHeight + gap`, `cx = margin`.  
4. `rowHeight = max(depth)` on that shelf.  
5. Gap = `gridSize`.  

**Reference implementations to read:**

- Floor-generator (constraint layouts + DXF): https://github.com/Ctechky/Floor-generator  
- HyparSpace (professional space planning functions, C#): https://github.com/hypar-io/HyparSpace  
- Hypar Elements (smallest useful BIM, rooms/spaces): https://github.com/hypar-io/elements  
  Docs: https://hypar-io.github.io/Elements/index.html  
- Hypar RoomKit (area allocations): https://github.com/hypar-io/RoomKit  

**Not V1:** MIP/Gurobi solvers, LLM layout (Co-Layout): https://github.com/xccElephant/co-layout  
Paper: https://arxiv.org/html/2511.12474v2  

**Not V1:** neural floorplan reconstruction (RoomFormer): https://github.com/ywyue/roomformer  

If rooms overflow the plate: still place them (overflow visually) + banner “Program does not fit this floor plate.”

### 8.3 Resize policies (product decision — lock this)

**Decision A (recommended for V1): table is programmed intent; 3D gizmo edits achieved geometry.**

| User action | Result |
|---|---|
| Edit **Target Area** in table | Resize box with `keepAspect`. Achieved becomes ≈ target. |
| Drag **move** in 3D | Update x,y only. Areas unchanged. |
| Drag **resize handle** in 3D | Update width/depth. **Achieved** changes. **Target stays** until user clicks “Match target to achieved” or we add a toggle. |

This matches dRofus mental model: programmed vs designed can differ; variance is the product.

**Alternative B:** Resize in 3D also writes Target Area (always equal). Simpler, less professional.

**V1 ships Decision A** plus a button per room: “Set target = achieved”.

When Target Area changes:

```
ratio = width / depth
depth = sqrt(targetArea / ratio)
width = targetArea / depth
```

Clamp min size (e.g. 3 ft).

### 8.4 Collision

V1: **soft**. Overlap is allowed but highlighted (red outline). Optional “nudge” later.

Do not spend V1 on physics.

### 8.5 Why not AI for sync

Architects need **auditability**. `250 → 300` must always produce the same box. AI is for V2: “generate program from PDF.”

---

## 9. Information architecture (screens)

### Screen A — Home

- New project / Open JSON / Load sample  
- One-paragraph explainer (from §0)

### Screen B — Workspace (main, 70% of engineering)

Split view:

| Left (~40%) | Right (~60%) |
|---|---|
| Spreadsheet + import/map | 3D canvas |
| Totals strip under grid | View tools: top/iso, grid, labels |

Top bar: project name, units, Export menu, Re-pack, Undo.

### Screen C — Import mapper (modal)

File drop → header preview → column mapping → Finish.

### Screen D — Export (modal)

Checkboxes: Excel, JSON, IFC. Download zip.

**No** login, teams, billing in V1 (optional later).

---

## 10. Technical architecture

### 10.1 Recommended stack

| Layer | Choice | Why |
|---|---|---|
| UI | React 18+ + TypeScript | Team familiarity; R3F |
| Bundler | Vite | Fast |
| State | Zustand + zundo | Simple; undo |
| Table | AG Grid Community | License-safe |
| Excel IO | SheetJS | Standard |
| 3D | Three.js + R3F + drei | Standard web 3D |
| Styling | Tailwind or existing CSS | Speed |
| IFC export | Python IfcOpenShell **or** ifc-lite | Quality vs JS-only |
| Tests | Vitest for sync/layout pure functions | Sync engine must be unit-tested |

### 10.2 Repo layout (proposed)

```
apps/web/                 # Vite React app
packages/schema/          # Zod types for Project/Room
packages/sync-engine/     # pure functions: resize, totals, pack
packages/excel/           # parse/map/serialize
packages/ifc-export/      # Python or TS adapter
fixtures/                 # sample xlsx/csv/json
docs/PRD-V1.md            # this file
```

### 10.3 Sync engine package API (contract)

Must be **framework-free** so tests don’t need Three.js:

```ts
packRooms(rooms, floor, options): Room[]
resizeFromTarget(room, targetArea, policy): Room
resizeFromBox(room, width, depth): Room
computeTotals(rooms): Totals
expandCounts(rows): Room[]
```

100% unit test coverage on this package is a **release gate**.

---

## 11. Interop later (do not block V1)

| System | Role | Link |
|---|---|---|
| Speckle | Object-based AEC data hub; Revit/Rhino/Excel connectors | https://github.com/specklesystems/speckle-server  
  Org: https://github.com/specklesystems  
  Docs: http://speckle.guide/  
  AEC Mag: https://aecmag.com/features/speckle-the-open-source-cloud-data-platform/ |
| Speckle Excel / Power BI | Spreadsheet analytics on model objects | https://github.com/specklesystems/speckle-powerbi |
| G.plus (Revit add-in) | Revit ↔ Excel for firms already in Revit | https://marketplace.autodesk.com/apps/718e1e87-7953-4656-8643-7b99d6da936d |
| Ideate BIMLink | Commercial Excel ↔ Revit | (vendor; competitor to our *browser* wedge) |
| Bonsai / BlenderBIM | GUI on IfcOpenShell | bundled with IfcOpenShell ecosystem |

V2 might **push rooms to Speckle** so a Revit user pulls them. V1 export IFC is enough.

---

## 12. Competitive / adjacent products (know them)

| Name | Relationship to us |
|---|---|
| Snaptrude | Full “design OS”; we take one feature. https://www.snaptrude.com/pricing |
| dRofus | Program database + Revit sync; weak as a massing toy. https://www.drofus.com/ |
| TestFit | Feasibility / yield. https://illustrarch.com/articles/design-softwares/74579-testfit-review.html |
| Forma | Environmental analysis. |
| Finch 3D | AI floor plans. |
| Rayon | Cloud design (watch). |
| Sweet Home 3D | OSS interior CAD — different job (furniture). https://www.sweethome3d.com/ |

Snaptrude pricing context (Free / $60 / $100 org): https://www.snaptrude.com/pricing  
Company funding (Series A $14M, total ~$21.8M): https://techcrunch.com/2023/11/09/snaptrude-funding-series-a/

---

## 13. Acceptance criteria (QA checklist)

**Import**

- [ ] Sample house CSV produces 8+ boxes (guest ×2).  
- [ ] Mapper can swap Area vs Name if headers are weird.  
- [ ] Bad numeric cell shows row error, does not crash.

**Sync**

- [ ] Change Master Bedroom 250 → 300: achieved within 0.1 of 300; aspect preserved.  
- [ ] Move box: x/y change; target unchanged.  
- [ ] Resize box: achieved updates; target unchanged; variance dashboard updates.  
- [ ] Delete row: mesh removed.  
- [ ] Add row: new box packed at end or at 0,0 with overlap warning.

**Totals**

- [ ] Department sums match `sum(target)` and `sum(width*depth)`.  
- [ ] Threshold colors match settings.

**Export**

- [ ] Excel re-imports without losing names/areas.  
- [ ] IFC opens in **BIM Vision** (free) or **Blender Bonsai** or **IfcOpenShell** viewer and shows spaces.  
  BIM Vision: https://bimvision.eu/  
  (Use any IFC viewer; do not require Revit in CI.)

**Performance**

- [ ] 200 rooms: grid + 3D remain interactive on a mid laptop.

**Tests**

- [ ] Vitest: packing does not NaN; resize math; count expansion.

---

## 14. Risks and mitigations

| Risk | Mitigation |
|---|---|
| We try to edit Revit | Ban `.rvt` from V1. IFC only. |
| Layout looks “stupid” | Ship re-pack + manual drag; don’t promise “architecture.” |
| Overlap hell | Visual warning; later collision. |
| IFC export ugly | Time-box; JSON+Excel still valuable. |
| License (Handsontable, AG Grid enterprise) | AG Grid Community. |
| Domain confusion on the team | This PRD glossary is required reading. |
| Scope creep (AI, walls, multiplayer) | Non-goals list is contractual. |

---

## 15. Phased delivery (engineering)

| Phase | Duration (1–2 engineers) | Deliverable |
|---|---|---|
| P0 | 3–5 days | Schema + Zustand + empty split UI |
| P1 | 1 week | Excel import + mapper + grid |
| P2 | 1 week | Shelf pack + Three.js boxes |
| P3 | 1 week | Bidirectional sync + totals + undo |
| P4 | 3–5 days | Export Excel/JSON + sample fixtures |
| P5 | 1 week | IFC export spike + polish + tests |

**Do not start IFC before P3 works.** The demo is the sync.

---

## 16. Open questions (decide in kickoff)

1. Default units: sq ft (US) vs m² (international)? **Proposal: toggle, default sq ft.**  
2. Decision A vs B on target vs achieved (§8.3). **Proposal: A.**  
3. IFC in Python vs JS-only. **Proposal: Python sidecar if we have backend; else ifc-lite spike.**  
4. Product name.  
5. Auth: none in V1?

---

## 17. Copy for in-app empty state (use verbatim if useful)

> This tool keeps a **room list** (like Excel) and a **3D sketch of boxes** as one project.  
> Change a number → the box changes. Drag a box → the measured area updates.  
> It does **not** replace Revit. When you are ready for construction drawings, export IFC and open it there.

---

## 18. Full reference appendix

### 18.1 Problem / market / Snaptrude

| Resource | URL |
|---|---|
| Snaptrude product | https://www.snaptrude.com/ |
| Snaptrude vs Revit | https://www.snaptrude.com/vs/revit |
| Snaptrude 2025 recap | https://www.snaptrude.com/blog/recap-of-2025 |
| Double data entry tax | https://www.snaptrude.com/blog/the-double-data-entry-tax-why-architecture-firms-maintain-two-models |
| Spreadsheet as design intent | https://www.snaptrude.com/blog/program-spreadsheet-bim-design-intent |
| Excel to BIM layout | https://www.snaptrude.com/blog/excel-to-bim-ai-layout-workflow |
| Program sheets BIM sync | https://www.snaptrude.com/blog/how-to-custom-program-sheets-bim-sync-revit |
| Parallel workflow tax | https://www.snaptrude.com/blog/parallel-workflow-tax-sketchup-revit |
| 90% problem (early design) | https://www.snaptrude.com/blog/the-90-problem-why-snaptrude-is-the-revit-alternative-architects-actually-need-in-early-stage-design |
| BIM massing speed | https://www.snaptrude.com/blog/bim-massing-speed |
| Architecture software 2026 | https://www.snaptrude.com/blog/best-architecture-software-2026 |
| Pricing | https://www.snaptrude.com/pricing |
| License help | https://help.snaptrude.com/en/articles/11581754-managing-licenses-billing |
| AEC Mag: Snaptrude on AI | https://aecmag.com/bim/snaptrude-on-ai/ |
| TechCrunch Series A | https://techcrunch.com/2023/11/09/snaptrude-funding-series-a/ |
| TechCrunch Seed | https://techcrunch.com/2023/01/04/snaptrude-seed-funding-accel-foundamental-vc-autodesk/ |
| ArchiGen review | https://archigenai.com/snaptrude-ai-bim-review-architects-2026.html |
| AECO.digital review | https://aeco.digital/snaptrude-review-2026/ |
| AECO conceptual BIM | https://aeco.digital/snaptrude-conceptual-design-bim/ |
| CheckThat reviews | https://checkthat.ai/brands/snaptrude/reviews |
| BestCRE Snaptrude | https://bestcre.com/snaptrude-review-cre-ai/ |

### 18.2 Practitioner forums

| Resource | URL |
|---|---|
| Archinect Revit thread | https://archinect.com/forum/thread/150377816/revit |
| Revit Forum SketchUp import | https://www.revitforum.org/forum/revit-architecture-forum-rac/architecture-conceptual-massing-and-adaptive-components/23832-revit-cannot-import-sketchup-file |
| Revit Forum SD templates | https://www.revitforum.org/forum/revit-architecture-forum-rac/architecture-and-general-revit-questions/16476-schematic-template-vs-general-purpose-template-workflow-long-winded-post-sorry |
| BIM Heroes space planning | https://bimheroes.com/architectural-space-planning/ |
| Autodesk Excel → Revit | https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-to-import-excel-sheet-into-Revit.html |
| Autodesk Community rooms/DB | https://forums.autodesk.com/t5/revit-architecture-forum/database-connections-and-space-management/td-p/11145439 |
| Dynamo Excel roundtrip | https://parametrix.gitbooks.io/dynamo-revit-recipes/content/02_Roundtrip-to-Excel/2-1_roundtrip-to-excel.html |
| LinkedIn Excel create rooms | https://www.linkedin.com/pulse/connect-room-dataexcelrevit-revitexcel-different-juli-hariyanto |
| SketchUp → Revit help | https://help.sketchup.com/cs/revit-interoperability/sketchup-to-revit |
| Rhino vs Revit (when to use which) | https://howtorhino.com/blog/software-for-architects/rhino-vs-revit/ |

### 18.3 Program / room data standards (learn the columns)

| Resource | URL |
|---|---|
| dRofus Excel import rooms | https://support.drofus.com/en/support/solutions/articles/16000078076-importing-room-space-program-from-excel |
| dRofus room properties (programmed vs designed) | https://drofus.atlassian.net/wiki/spaces/DV/pages/1318360545/Room-Properties |
| dRofus Revit trial steps | https://drofus.atlassian.net/wiki/spaces/DV/pages/2669248513/Step-by-Step-Instructions-Revit |

### 18.4 Open-source 3D / floorplan / layout (build from)

| Project | License (verify in repo) | URL |
|---|---|---|
| Three.js | MIT | https://github.com/mrdoob/three.js |
| React Three Fiber | MIT | https://github.com/pmndrs/react-three-fiber |
| Zustand | MIT | https://github.com/pmndrs/zustand |
| SheetJS | (check community vs pro) | https://github.com/SheetJS/sheetjs |
| AG Grid | MIT community | https://github.com/ag-grid/ag-grid |
| Handsontable | (commercial for SaaS) | https://github.com/handsontable/handsontable |
| open3dFloorplan | see repo | https://github.com/theLodgeBots/open3dFloorplan |
| Aedifex | see repo | https://github.com/TangSY/aedifex |
| Floor-generator | see repo | https://github.com/Ctechky/Floor-generator |
| Smarchitect | see repo | https://github.com/Salar24/Smarchitect |
| Hypar Elements | Apache-2.0 typically | https://github.com/hypar-io/elements |
| HyparSpace | MIT | https://github.com/hypar-io/HyparSpace |
| Hypar RoomKit | see repo | https://github.com/hypar-io/RoomKit |
| Co-Layout | research | https://github.com/xccElephant/co-layout |
| infinigen-room-layout-env | research constraints | https://github.com/Michael-YuQ/infinigen-room-layout-env |

### 18.5 IFC / BIM in the browser and Python

| Project | URL |
|---|---|
| IfcOpenShell | https://github.com/IfcOpenShell/IfcOpenShell — https://ifcopenshell.org/ |
| IfcCSV | https://docs.ifcopenshell.org/ifccsv.html |
| web-ifc | https://github.com/ThatOpen/engine_web-ifc |
| That Open components | https://github.com/ThatOpen/engine_components |
| That Open docs IfcLoader | https://docs.thatopen.com/Tutorials/Components/Core/IfcLoader |
| IFC-Lite | https://github.com/louistrue/ifc-lite — https://ltplus-ag.github.io/ifc-lite/ |
| Speckle server + viewer | https://github.com/specklesystems/speckle-server |
| Speckle Sharp (Revit connector, later) | https://github.com/specklesystems/speckle-sharp |

### 18.6 Market context (optional reading)

| Resource | URL |
|---|---|
| Early-stage feasibility tools 2026 | https://www.parametric.se/post/comparing-early-stage-feasibility-tools-2026-forma-giraffe-finch-testfit-hektar |
| TestFit review | https://bestcre.com/testfit-review-cre-ai/ |
| AEC software market (indicative) | https://www.mordorintelligence.com/industry-reports/aec-software-market |

---

## 19. Glossary recap for tickets

When a ticket says **“program,”** implement a **table of rooms**.  
When it says **“massing,”** implement **boxes**.  
When it says **“sync,”** implement **one JSON store, two views**.  
When it says **“Revit,”** implement **IFC export**, not Autodesk APIs.

---

## Document control

| Version | Date | Notes |
|---|---|---|
| 0.1 | 2026-08-19 | First PRD for engineering V1 |

**Owner:** Product / engineering  
**Reviewers:** Anyone implementing V1 must read §0, §3, §6, §8 before writing UI.
