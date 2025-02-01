---

## **Input Map Parsing**

### **ParsingData() Execution**
```go
// Phase 1: Room Initialization
GlobVar.AntsNumber = 10
GlobVar.Start = "start"
GlobVar.End = "end"

Rooms created (partial):
{
    "start": {Links: []},
    "0": {Links: []},
    "o": {Links: []},
    "n": {Links: []},
    ...
    "end": {Links: []}
}

// Phase 2: Link Processing
Processing links in order:
1. start-t → bidirection added
2. n-e → bidirection
3. a-m → bidirection
...
16. h-n → bidirection

Final Critical Links:
start connected to: t, h, 0
end connected to: m, e, k
```

---

## **Path Finding (Algorithms.FindValidPaths)**

### **First BFS Run**
**Queue Evolution:**
```go
Initial queue: [["start"]]
Expand "start" links [t, h, 0]

Path1: start-t
Path2: start-h
Path3: start-0
```

**Path Development:**
```go
1. Path start-t expands:
   - t connected to E
   - Path becomes start-t-E
   - E connected to a
   - Path start-t-E-a
   - a connected to m
   - Path start-t-E-a-m
   - m connected to end → VALID PATH

First valid path: [start t E a m end]
```

**Post-Processing:**
```go
Helpers.RemovePathsLinks():
   Remove links:
   start-t, t-E, E-a, a-m, m-end
   (and reverse links)
   
Updated Links:
start.Links = [h, 0]  # removed t
m.Links = [a]         # removed end
```

### **Second BFS Run**
**New Path Development:**
```go
Queue starts with ["start"]
Explore remaining links h and 0

Path start-h:
   h connected to A, n
   Path start-h-A
   A connected to c
   Path start-h-A-c
   c connected to k
   Path start-h-A-c-k
   k connected to end → VALID PATH

Second valid path: [start h A c k end]
```

**Post-Processing:**
```go
Remove links:
start-h, h-A, A-c, c-k, k-end

Updated Links:
start.Links = [0]  # only 0 remaining
k.Links = [c]      # removed end
```

### **Third BFS Run**
**Path Development:**
```go
Queue: ["start"]
Only link 0 remains:
Path start-0
0 connected to o
Path start-0-o
o connected to n
Path start-0-o-n
n connected to e, m
Path start-0-o-n-e
e connected to end → VALID PATH

Third valid path: [start 0 o n e end]
```

**Final ValidPaths:**
```go
GlobVar.ValidPaths = [
    [start t E a m end],    // Length 6
    [start h A c k end],    // Length 6
    [start 0 o n e end]     // Length 6
]
```

---

## **Ant Distribution (Algorithms.OrderAnts)**

### **Path Analysis**
```go
All paths equal length (6 rooms → 5 steps base)
Ants = 10
Optimal distribution: [4,3,3]

Turns calculation:
4 ants: 5 + 4 - 1 = 8 steps
3 ants: 5 + 3 - 1 = 7 steps
Total turns = 8
```

### **Ant Allocation Table**
| Ant# | Path       | Movement Timeline          |
|------|------------|-----------------------------|
| 1-4  | Path1      | Steps 1-8                   |
| 5-7  | Path2      | Steps 2-8                   |
| 8-10 | Path3      | Steps 3-8                   |

---

## **Movement Simulation (Utils.HandleExport)**

### **Step-by-Step Output Construction**
**Turn 1:**
```go
Path1 Ant1: start → t
result[0] = "L1-t"
```

**Turn 2:**
```go
Path1 Ant1: t → E
Path2 Ant5: start → h
result[1] = "L1-E L5-h"
```

**Turn 3:**
```go
Path1 Ant1: E → a
Path2 Ant5: h → A
Path3 Ant8: start → 0
result[2] = "L1-a L5-A L8-0"
```

**Turn 4:**
```go
Path1 Ant1: a → m
Path2 Ant5: A → c
Path3 Ant8: 0 → o
result[3] = "L1-m L5-c L8-o"
```

**Turn 5:**
```go
Path1 Ant1: m → end (exits)
Path2 Ant5: c → k
Path3 Ant8: o → n
result[4] = "L1-end L5-k L8-n"
```

**Turn 6:**
```go
Path2 Ant5: k → end (exits)
Path3 Ant8: n → e
result[5] = "L5-end L8-e"
```

**Turn 7:**
```go
Path3 Ant8: e → end (exits)
result[6] = "L8-end"
```

**Turns 8-9:**
```go
Remaining ants follow similar patterns:
result[7] = "L2-end L6-end L9-end"
result[8] = "L3-end L7-end L10-end"
```

---

## **Final Output**
```txt
10
##start
start 1 6
... (full input) ...

L1-t
L1-E L5-h
L1-a L5-A L8-0
L1-m L5-c L8-o
L1-end L5-k L8-n
L5-end L8-e
L8-end
L2-end L6-end L9-end
L3-end L7-end L10-end
L4-end
```

---

## **Critical Variable Snapshots**

### **After Path Removal (Post Path1)**
```go
GlobVar.Rooms["start"].Links = [h, 0]  // t removed
GlobVar.Rooms["m"].Links = [a]        // end removed
GlobVar.Rooms["end"].Links = [e,k]    // m removed
```

### **During Backtrack Check**
```go
When finding path [start h A c k end]:
Check against existing paths:
- No shared links except start-h
- h→A not present in previous paths
→ No backtrack detected
```

### **Optimal Path Validation**
```go
AllValidPaths = [
    [ [start t E a m end] ],
    [ [start h A c k end] ],
    [ [start 0 o n e end] ]
]
Path lengths: 6,6,6 → equal distribution optimal
```

