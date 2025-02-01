# lem-in-readme

```markdown
# LEM-IN README: Comprehensive Simulation Scenarios

## Scenario 1: Simple Direct Path

### Input
```
4
##start
A 0 0
##end
B 1 1
A-B
```

### Execution Flow

1. **ParsingData()**
   ```go
   Line 1: "4" → GlobVar.AntsNumber = 4
   Line 2: "##start" → Next line is start room
   Line 3: "A 0 0" → GlobVar.Start = "A"
   Line 4: "##end" → Next line is end room
   Line 5: "B 1 1" → GlobVar.End = "B"
   Line 6: "A-B" → Add bidirectional link
   
   Final Rooms:
   GlobVar.Rooms = {
       "A": {Links: ["B"], IsChecked: false},
       "B": {Links: ["A"], IsChecked: false}
   }
   ```

2. **FindValidPaths()**
   - Initial BFS queue: `[["A"]]`
   - **First BFS Iteration:**
     ```go
     Process path ["A"]
     Last room A links to B (end room)
     → ValidPaths = [["A","B"]]
     → Remove A-B link from both rooms
     ```
   - Final ValidPaths: `[[A B]]`

3. **OrderAnts(0)**
   ```go
   Path lengths: [2]
   Ant distribution: [4]
   Turns calculation: 4 + 2 - 1 = 5 steps
   Ants list: [4]
   ```

4. **HandleExport()**
   ```go
   Ant movements:
   Step 1: L1-B
   Step 2: L2-B (L1 exits)
   Step 3: L3-B (L2 exits)
   Step 4: L4-B (L3 exits)
   Step 5: L4 exits
   ```

### Final Output
```
4
##start
A 0 0
##end
B 1 1
A-B

L1-B
L2-B
L3-B
L4-B
```

## Scenario 2: Multiple Paths with Backtracking

### Input
```
6
##start
A 0 0
##end
D 3 3
B 1 1
C 2 2
E 4 4
A-B
B-C
C-D  # Path 1: A-B-C-D (length 4)
A-E
E-D  # Path 2: A-E-D (length 3)
B-E  # Creates crossover
```

### Execution Flow

1. **Initial Parsing**
   ```go
   Rooms = {
       "A": {Links: ["B","E"]},
       "B": {Links: ["A","C","E"]},
       "C": {Links: ["B","D"]},
       "D": {Links: ["C","E"]},
       "E": {Links: ["A","D","B"]}
   }
   ```

2. **First FindValidPaths() Run**
   - BFS finds path `[A B C D]`
   - **CheckIfBackTrackingPath()**
     ```go
     No previous paths → returns false
     Remove links: B-C and C-B
     ```

3. **Second FindValidPaths() Run**
   - Reset rooms to original state
   - BFS finds path `[A E D]`
   - **CheckIfBackTrackingPath()**
     ```go
     Compare with first path:
     No reversed links → returns false
     Remove links: A-E and E-A
     ```

4. **Third FindValidPaths() Run**
   - BFS finds path `[A B E D]`
   - **CheckIfBackTrackingPath()**
     ```go
     Detects reversed B-E link from first path's E-B
     → Returns true, removes B-E and E-B links
     → Saves current paths to AllValidPaths
     ```

5. **Final ValidPaths Sets**
   ```go
   AllValidPaths = [
       [ [A B C D], [A E D] ],  // First valid set
       [ [A B E D] ]            // Second valid set
   ]
   ```

6. **OrderAnts() Analysis**
   - **For AllValidPaths[0] (paths 4 & 3 length):**
     ```go
     Distribute 6 ants:
     Path1 (length 4): 3 ants → 4 + 3 - 1 = 6 steps
     Path2 (length 3): 3 ants → 3 + 3 - 1 = 5 steps
     Total: 6 steps
     ```
   - **For AllValidPaths[1] (single path length 4):**
     ```go
     6 ants → 4 + 6 - 1 = 9 steps
     ```
   - **Optimal choice:** First path set with 6 steps

7. **HandleExport()**
   ```go
   Ant movements (partial):
   Step 1: L1-B, L4-E
   Step 2: L1-C, L4-D, L2-B, L5-E
   Step 3: L1-D, L4-exit, L2-C, L5-D, L3-B, L6-E
   Step 4: L2-D, L5-exit, L3-C, L6-D
   ```

### Final Output
```
6
##start
A 0 0
##end
D 3 3
B 1 1
C 2 2
E 4 4
A-B
B-C
C-D
A-E
E-D
B-E

L1-B L4-E
L1-C L4-D L2-B L5-E
L1-D L2-C L5-D L3-B L6-E
L2-D L3-C L6-D
L3-D L6-exit
```

## Key Execution Patterns

### BFS Internal States (Multiple Path Scenario)
| Iteration | Queue Contents               | Valid Path Found | Room Checks       |
|-----------|------------------------------|------------------|-------------------|
| 1         | [ [A] ]                      | None             | A.checked=true    |
| 2         | [ [A B], [A E] ]             | None             | B.checked=true    |
| 3         | [ [A B C], [A E D] ]         | A-E-D            | C.checked=true    |
| 4         | [ [A B C D] ]                | A-B-C-D          | D.checked=true    |

### OrderAnts() Variable Evolution
| Path Index | Path Length | Ants Added | Turns Calculation      |
|------------|-------------|------------|------------------------|
| 0          | 4           | 3          | 4 + 3 - 1 = 6          |
| 1          | 3           | 3          | 3 + 3 - 1 = 5 (max 6) |

### Backtracking Detection
```go
When finding path [A B E D]:
Check against previous path [A B C D]:
- Path[1] = B, previous paths have B->C
- E's BeforeInPath = B (from current path)
- B's BeforeInPath in previous path = A
- No conflict → continues
- Path[2] = E, previous paths have no E links
→ No backtracking detected
```

This documentation shows complete code path execution with all variable transitions, following the exact program logic from input parsing to final ant movement simulation.




