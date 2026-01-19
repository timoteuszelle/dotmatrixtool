# Software Icon Export Feature

## Overview

This feature adds **grayscale export** capabilities to the Framework LED Matrix tool, enabling developers to create custom icons for LED matrix applications.

## Motivation

While the LED Matrix tool is excellent for testing hardware patterns, developers building LED matrix applications need:

1. **Grayscale intensities** (0-255) for smooth gradients and varying brightness
2. **Software-compatible formats** for use in IconManager and icon rendering systems
3. **Transposed output** (column-major) to match common LED matrix software conventions
4. **Animation frame export** for creating icon animation sequences

## Use Cases

### 1. Application Icon Creation
Create custom icons for system monitoring apps:
- Lock/unlock symbols
- App indicators (CPU, memory, network, disk)
- Status icons (battery, notifications)
- Custom branding

### 2. Animation Frame Design
Design frame-by-frame animations:
- Breathing effects
- Progress indicators
- Transitions
- Loading spinners

### 3. Software Development
Enable LED matrix app development:
- Rapid prototyping of icon designs
- Visual testing before code integration
- Icon library creation
- Community icon sharing

## Feature Specification

### Export Options

**1. Standard Export (Hardware)**
- Format: Binary (0/1)
- Layout: Row-major (34x9)
- Use: Direct hardware display

**2. Grayscale Export (Software)** ⭐ NEW
- Format: Grayscale (0-255 intensity)
- Layout: Configurable (row-major or column-major)
- Use: Software icon systems

**3. Transposed Export** ⭐ NEW
- Format: Grayscale (0-255)
- Layout: Column-major (9x34)
- Use: IconManager-compatible format

### Export Formats Comparison

```javascript
// Standard Export (existing - hardware)
// 34 rows x 9 columns, binary
[
  [0, 1, 0, 1, 0, 1, 0, 1, 0],  // Row 0
  [1, 0, 1, 0, 1, 0, 1, 0, 1],  // Row 1
  // ... 32 more rows
]

// Grayscale Export (new - software, row-major)
// 34 rows x 9 columns, 0-255 intensity
[
  [0, 128, 0, 255, 0, 128, 0, 64, 0],  // Row 0
  [255, 0, 192, 0, 128, 0, 64, 0, 32], // Row 1
  // ... 32 more rows
]

// Transposed Grayscale Export (new - software, column-major)
// 9 columns x 34 rows, 0-255 intensity
[
  [0, 255, 128, 64, ...],  // Column 0 (34 values)
  [128, 0, 255, 192, ...],  // Column 1 (34 values)
  // ... 7 more columns
]
```

## UI Design

### New Export Section

```
┌─────────────────────────────────────┐
│  Export Options                     │
├─────────────────────────────────────┤
│                                     │
│  Format:                            │
│  ○ Binary (0/1) - Hardware         │
│  ● Grayscale (0-255) - Software    │
│                                     │
│  Layout:                            │
│  ○ Row-major (34x9)                │
│  ● Column-major (9x34) - IconMgr   │
│                                     │
│  [Export Left]  [Export Right]     │
│                                     │
└─────────────────────────────────────┘
```

### Button Placement

Add buttons after existing "Clear" buttons:

```html
<div class="btn-group">
  <button id="clearLeftBtn">Clear Left</button>
  <button id="clearRightBtn">Clear Right</button>
</div>

<!-- NEW: Export section -->
<div class="export-section">
  <h3>Export for Software</h3>
  
  <div class="radio-group">
    <label><input type="radio" name="exportFormat" value="binary" checked> Binary (Hardware)</label>
    <label><input type="radio" name="exportFormat" value="grayscale"> Grayscale (Software)</label>
  </div>
  
  <div class="radio-group">
    <label><input type="radio" name="exportLayout" value="rowmajor" checked> Row-major (34x9)</label>
    <label><input type="radio" name="exportLayout" value="colmajor"> Column-major (9x34)</label>
  </div>
  
  <div class="btn-group">
    <button id="exportLeftBtn">Export Left</button>
    <button id="exportRightBtn">Export Right</button>
  </div>
</div>
```

## Implementation

### Core Functions

```javascript
/**
 * Export matrix in software-compatible format
 * @param {Array} matrix - The matrix data (34x9)
 * @param {string} side - 'left' or 'right'
 * @param {boolean} grayscale - true for 0-255, false for 0/1
 * @param {boolean} transpose - true for column-major, false for row-major
 */
function exportMatrixSoftware(matrix, side, grayscale = true, transpose = true) {
  const width = matrix[0].length;  // 9
  const height = matrix.length;     // 34
  
  let vals;
  
  if (transpose) {
    // Column-major: 9 columns x 34 rows
    vals = Array(width).fill(0).map(() => Array(height).fill(0));
    
    for (let col = 0; col < width; col++) {
      for (let row = 0; row < height; row++) {
        const isLit = !matrix[row][col];  // Inverted: 0 = off, 1 = on
        
        if (grayscale) {
          vals[col][row] = isLit ? 255 : 0;
        } else {
          vals[col][row] = isLit ? 1 : 0;
        }
      }
    }
  } else {
    // Row-major: 34 rows x 9 columns
    vals = Array(height).fill(0).map(() => Array(width).fill(0));
    
    for (let row = 0; row < height; row++) {
      for (let col = 0; col < width; col++) {
        const isLit = !matrix[row][col];
        
        if (grayscale) {
          vals[row][col] = isLit ? 255 : 0;
        } else {
          vals[row][col] = isLit ? 1 : 0;
        }
      }
    }
  }
  
  // Generate filename
  const formatStr = grayscale ? 'grayscale' : 'binary';
  const layoutStr = transpose ? 'transposed' : 'standard';
  const filename = `matrix_${side}_${formatStr}_${layoutStr}.json`;
  
  // Download JSON
  const blob = new Blob([JSON.stringify(vals, null, 2)], 
    { type: "application/json" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
  
  console.log(`Exported ${filename}: ${vals.length}x${vals[0].length}`);
}
```

### Event Handlers

```javascript
function initExportOptions() {
  $('#exportLeftBtn').click(function() {
    const grayscale = $('input[name="exportFormat"]:checked').val() === 'grayscale';
    const transpose = $('input[name="exportLayout"]:checked').val() === 'colmajor';
    exportMatrixSoftware(matrix_left, 'left', grayscale, transpose);
  });
  
  $('#exportRightBtn').click(function() {
    const grayscale = $('input[name="exportFormat"]:checked').val() === 'grayscale';
    const transpose = $('input[name="exportLayout"]:checked').val() === 'colmajor';
    exportMatrixSoftware(matrix_right, 'right', grayscale, transpose);
  });
}
```

## Integration with LED Matrix Monitoring Project

### Direct Usage

```python
from icon_manager import IconManager

# Load exported icon
mgr = IconManager()
icon = mgr.load_icon('custom_lock')  # Loads custom_lock/static.json

# Use in monitoring app
grid = np.zeros((34, 9), dtype=np.uint8)
icon_manager.overlay_icon(grid, icon, position='center')
```

### Workflow

1. **Design** - Draw icon in Framework tool
2. **Export** - Choose "Grayscale + Column-major"
3. **Save** - Place in `icons/rendered/your_icon/static.json`
4. **Load** - Use IconManager to load in app
5. **Render** - Overlay on LED matrix grid

## Benefits

### For Hardware Users
- **Better visuals** - Grayscale support enables intensity control
- **Gradients** - Smooth transitions between brightness levels
- **Professional look** - More sophisticated patterns

### For Software Developers
- **Rapid prototyping** - Visual icon design without coding
- **Icon library** - Build collections of reusable icons
- **Community sharing** - Exchange icon designs
- **Animation creation** - Design frame sequences

### For Framework Ecosystem
- **Developer enablement** - Lower barrier to LED app development
- **Community growth** - More LED matrix applications
- **Use case expansion** - Beyond hardware testing to software creation
- **Educational** - Shows full capabilities of LED panels

## Compatibility

### Backward Compatible
- ✅ Existing binary export unchanged
- ✅ Hardware workflow unaffected
- ✅ No breaking changes

### Forward Compatible
- ✅ Architecture supports future grayscale hardware rendering
- ✅ Import functionality can be extended
- ✅ Animation frame export (future)

## Testing

### Test Cases

1. **Binary export** - Verify 0/1 values, row-major format
2. **Grayscale export** - Verify 0-255 values, row-major format
3. **Transposed export** - Verify column-major layout (9x34)
4. **Combined export** - Grayscale + transposed
5. **Both panels** - Export left and right independently
6. **IconManager integration** - Load exported icons successfully

### Example Test

```javascript
// Draw simple pattern
matrix_left[0][0] = 0;  // Lit pixel
matrix_left[0][1] = 1;  // Dark pixel

// Export grayscale, transposed
exportMatrixSoftware(matrix_left, 'left', true, true);

// Expected output (9x34):
// [
//   [255, 0, 0, ...],  // Column 0: first pixel lit
//   [0, 0, 0, ...],     // Column 1: all dark
//   ...
// ]
```

## Documentation

### User Guide

Include in README.md:

```markdown
## Exporting Icons for Software

To create icons for LED matrix applications:

1. Draw your icon in the grid
2. Select "Grayscale (Software)" format
3. Select "Column-major (9x34)" layout
4. Click "Export Left" or "Export Right"
5. Save to your app's icon directory

The exported JSON can be loaded with IconManager in Python LED matrix apps.
```

### Developer Guide

Link to this document for technical details.

## Future Enhancements

### Phase 2: Advanced Features

1. **Intensity levels** - Allow drawing with varying grayscale (not just on/off)
2. **Import grayscale** - Load existing grayscale icons for editing
3. **Animation export** - Export multiple frames as animation sequence
4. **Preview modes** - Toggle between binary and grayscale view
5. **Dual-panel export** - Export combined 18x34 icon

### Phase 3: Community Features

1. **Icon library** - Browse and download community icons
2. **Share button** - Upload icons to shared repository
3. **Format converter** - Bidirectional conversion between formats
4. **Templates** - Common icon templates (lock, battery, wifi, etc.)

## Reference Implementation

See:
- `icon_editor.html` in led-matrix-monitoring repo
- `icon_manager.py` for IconManager integration
- `ICON_CREATION.md` for workflow documentation

## Credits

- Original concept: Framework LED Matrix tool team
- Grayscale export: Contributed by LED matrix monitoring project
- Use case: https://github.com/timoteuszelle/led-matrix

---

**Status**: Ready for implementation  
**Priority**: Medium  
**Complexity**: Low (simple JSON export)  
**Value**: High (enables developer ecosystem)

