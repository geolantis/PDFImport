# Storage and Transformation File Plan

## Overview
This document outlines the plan for storing prepared images (with legends/frames removed) and transformation files that contain georeferencing data. This enables users to save their work, share it, and reload it later.

## Goals
1. **Persist Work** - Save prepared images and georeferencing data
2. **Enable Sharing** - Export files that can be shared with others
3. **Resume Sessions** - Reload previous work to continue editing
4. **Maintain Quality** - Preserve image quality and transformation accuracy
5. **Ensure Compatibility** - Use standard formats for interoperability

---

## File Formats

### 1. Prepared Image File
**Format**: PNG with transparency
- **Filename**: `[original-name]_prepared.png`
- **Contents**: PDF rendered to image with selected areas removed/kept
- **Metadata**: Embedded EXIF/PNG chunks with processing info

### 2. Transformation File
**Format**: JSON (human-readable, version-controlled)
- **Filename**: `[original-name]_transform.json`
- **Contents**: All data needed to recreate the georeferencing

### 3. Project Bundle (Optional)
**Format**: ZIP archive
- **Filename**: `[project-name]_bundle.zip`
- **Contents**: Original PDF, prepared image, transformation file, thumbnail

---

## Transformation File Structure

```json
{
  "version": "1.0",
  "metadata": {
    "created": "2024-01-07T10:30:00Z",
    "modified": "2024-01-07T11:45:00Z",
    "software": "PDF Map Overlay Tool v1.0",
    "author": "user@example.com"
  },
  
  "source": {
    "filename": "site-plan-2024.pdf",
    "fileHash": "sha256:abcd1234...",
    "dimensions": {
      "width": 1684,
      "height": 1190,
      "dpi": 300
    },
    "pageNumber": 1
  },
  
  "preparation": {
    "timestamp": "2024-01-07T10:35:00Z",
    "legendRemoval": {
      "enabled": true,
      "mode": "remove", // or "keep"
      "selections": [
        {
          "id": "rect_1",
          "x": 50,
          "y": 50,
          "width": 200,
          "height": 300,
          "unit": "pixels"
        }
      ]
    },
    "backgroundRemoval": {
      "enabled": true,
      "threshold": 240,
      "color": "#FFFFFF"
    }
  },
  
  "georeferencing": {
    "timestamp": "2024-01-07T11:00:00Z",
    "coordinateSystem": "WGS84", // or "EPSG:31256", etc.
    "referencePoints": [
      {
        "id": 1,
        "image": {
          "x": 0.2568,
          "y": 0.3134,
          "unit": "normalized" // 0-1
        },
        "world": {
          "x": 13.845103,
          "y": 46.608785,
          "unit": "degrees" // or meters for projected
        }
      },
      {
        "id": 2,
        "image": {
          "x": 0.8893,
          "y": 0.6721,
          "unit": "normalized"
        },
        "world": {
          "x": 13.846339,
          "y": 46.608444,
          "unit": "degrees"
        }
      }
    ],
    "transformation": {
      "type": "affine",
      "parameters": {
        "scale": 0.000001117536998974559,
        "rotation": 6.413752285936809, // degrees
        "translation": {
          "x": 13.844669213752924,
          "y": 46.608316796819025
        }
      },
      "bounds": {
        "topLeft": [13.844669213752924, 46.608316796819025],
        "topRight": [13.846539367299828, 46.60852702268899],
        "bottomRight": [13.846390811014047, 46.60984856824767],
        "bottomLeft": [13.844520657467143, 46.60963834237771]
      }
    }
  },
  
  "display": {
    "opacity": 0.7,
    "visible": true,
    "zIndex": 10
  }
}
```

---

## Implementation Plan

### Phase 1: Export Functionality

#### 1.1 Export Prepared Image
```javascript
function exportPreparedImage() {
    // Get the processed canvas
    const canvas = getCurrentProcessedCanvas();
    
    // Add metadata
    const metadata = {
        software: 'PDF Map Overlay Tool',
        timestamp: new Date().toISOString(),
        originalFile: originalFileName
    };
    
    // Convert to blob with metadata
    canvas.toBlob((blob) => {
        // Create download link
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `${getBaseFileName()}_prepared.png`;
        a.click();
        URL.revokeObjectURL(url);
    }, 'image/png');
}
```

#### 1.2 Export Transformation File
```javascript
function exportTransformation() {
    const transformData = {
        version: "1.0",
        metadata: {
            created: firstCreated || new Date().toISOString(),
            modified: new Date().toISOString(),
            software: "PDF Map Overlay Tool v1.0"
        },
        source: {
            filename: originalFileName,
            dimensions: {
                width: pdfWidth,
                height: pdfHeight
            }
        },
        preparation: {
            legendRemoval: {
                enabled: selectionState.selections.length > 0,
                mode: selectionState.mode,
                selections: selectionState.selections
            }
        },
        georeferencing: {
            coordinateSystem: document.getElementById('coordSystem').value,
            referencePoints: referencePoints,
            transformation: currentTransformation
        }
    };
    
    // Download JSON
    const json = JSON.stringify(transformData, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${getBaseFileName()}_transform.json`;
    a.click();
    URL.revokeObjectURL(url);
}
```

### Phase 2: Import Functionality

#### 2.1 Import Transformation File
```javascript
function importTransformation(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const data = JSON.parse(e.target.result);
            
            // Validate version
            if (!isCompatibleVersion(data.version)) {
                throw new Error('Incompatible transformation file version');
            }
            
            // Restore preparation state
            if (data.preparation.legendRemoval.enabled) {
                selectionState.selections = data.preparation.legendRemoval.selections;
                selectionState.mode = data.preparation.legendRemoval.mode;
            }
            
            // Restore georeferencing
            referencePoints = data.georeferencing.referencePoints;
            currentTransformation = data.georeferencing.transformation;
            
            // Apply to map
            applyStoredTransformation(data);
            
        } catch (error) {
            alert('Error loading transformation file: ' + error.message);
        }
    };
    reader.readAsText(file);
}
```

#### 2.2 Load Prepared Image
```javascript
function loadPreparedImage(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
            // Replace current image
            pdfImageUrl = e.target.result;
            document.querySelector('.pdf-image').src = pdfImageUrl;
            isProcessed = true;
            
            // Update UI
            updateUIForPreparedImage();
        };
        img.src = e.target.result;
    };
    reader.readAsDataURL(file);
}
```

### Phase 3: UI Integration

#### 3.1 Export Menu
```html
<div class="export-menu">
    <button class="export-btn" onclick="showExportOptions()">
        Export/Save
    </button>
    <div class="export-options" style="display: none;">
        <button onclick="exportPreparedImage()">
            📸 Export Prepared Image (PNG)
        </button>
        <button onclick="exportTransformation()">
            📄 Export Transformation File (JSON)
        </button>
        <button onclick="exportBundle()">
            📦 Export Complete Bundle (ZIP)
        </button>
    </div>
</div>
```

#### 3.2 Import UI
```html
<div class="import-section">
    <label for="importTransform" class="import-btn">
        📂 Load Transformation
    </label>
    <input type="file" 
           id="importTransform" 
           accept=".json"
           onchange="handleTransformImport(this.files[0])"
           style="display: none;">
</div>
```

### Phase 4: Advanced Features

#### 4.1 Auto-Save
```javascript
// Auto-save to localStorage
function autoSaveProgress() {
    const saveData = {
        timestamp: new Date().toISOString(),
        transformation: getCurrentTransformation(),
        preparation: getPreparationState()
    };
    
    localStorage.setItem('pdfOverlay_autosave', JSON.stringify(saveData));
}

// Restore on page load
function checkAutoSave() {
    const saved = localStorage.getItem('pdfOverlay_autosave');
    if (saved) {
        const data = JSON.parse(saved);
        if (confirm(`Restore previous session from ${formatDate(data.timestamp)}?`)) {
            restoreAutoSave(data);
        }
    }
}
```

#### 4.2 Project Management
```javascript
// Save/load complete projects
class ProjectManager {
    constructor() {
        this.projects = this.loadProjects();
    }
    
    saveProject(name) {
        const project = {
            id: generateId(),
            name: name,
            created: new Date().toISOString(),
            thumbnail: generateThumbnail(),
            data: {
                source: currentSource,
                preparation: currentPreparation,
                transformation: currentTransformation
            }
        };
        
        this.projects.push(project);
        this.saveProjects();
        return project.id;
    }
    
    loadProject(id) {
        const project = this.projects.find(p => p.id === id);
        if (project) {
            restoreProject(project.data);
        }
    }
    
    deleteProject(id) {
        this.projects = this.projects.filter(p => p.id !== id);
        this.saveProjects();
    }
    
    saveProjects() {
        localStorage.setItem('pdfOverlay_projects', JSON.stringify(this.projects));
    }
    
    loadProjects() {
        const saved = localStorage.getItem('pdfOverlay_projects');
        return saved ? JSON.parse(saved) : [];
    }
}
```

### Phase 5: Export Formats

#### 5.1 GeoTIFF Export
```javascript
// Export as GeoTIFF with embedded geospatial metadata
async function exportGeoTIFF() {
    const geoTIFF = new GeoTIFF({
        width: processedImage.width,
        height: processedImage.height,
        bbox: transformationBounds,
        epsg: getEPSGCode(coordinateSystem),
        image: processedImageData
    });
    
    const blob = await geoTIFF.toBlob();
    downloadBlob(blob, `${baseFileName}_georef.tif`);
}
```

#### 5.2 KML/KMZ Export
```javascript
// Export as KML for Google Earth
function exportKML() {
    const kml = `<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://www.opengis.net/kml/2.2">
    <Document>
        <name>${originalFileName}</name>
        <GroundOverlay>
            <name>${baseFileName} Overlay</name>
            <Icon>
                <href>${baseFileName}_prepared.png</href>
            </Icon>
            <LatLonBox>
                <north>${bounds.north}</north>
                <south>${bounds.south}</south>
                <east>${bounds.east}</east>
                <west>${bounds.west}</west>
                <rotation>${rotation}</rotation>
            </LatLonBox>
        </GroundOverlay>
    </Document>
</kml>`;
    
    downloadText(kml, `${baseFileName}.kml`);
}
```

### Phase 6: Cloud Storage Integration (Future)

#### 6.1 Cloud Save
```javascript
async function saveToCloud() {
    const formData = new FormData();
    formData.append('image', preparedImageBlob);
    formData.append('transformation', transformationJSON);
    formData.append('metadata', JSON.stringify(metadata));
    
    const response = await fetch('/api/projects/save', {
        method: 'POST',
        body: formData,
        headers: {
            'Authorization': `Bearer ${authToken}`
        }
    });
    
    const result = await response.json();
    return result.projectId;
}
```

#### 6.2 Share Link Generation
```javascript
async function generateShareLink(projectId) {
    const response = await fetch(`/api/projects/${projectId}/share`, {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${authToken}`
        }
    });
    
    const { shareUrl } = await response.json();
    return shareUrl; // e.g., https://app.com/share/abc123
}
```

---

## Security Considerations

1. **File Validation**
   - Verify file types and sizes
   - Sanitize filenames
   - Check image dimensions
   - Validate JSON structure

2. **Privacy**
   - Strip sensitive EXIF data
   - Optional anonymization
   - Secure cloud storage
   - Encrypted sharing links

3. **Data Integrity**
   - File hashes for verification
   - Version compatibility checks
   - Transformation validation
   - Coordinate boundary checks

---

## Implementation Priority

1. **Phase 1** (Essential)
   - Export prepared image
   - Export transformation JSON
   - Basic import functionality

2. **Phase 2** (Important)
   - Auto-save to localStorage
   - Project management
   - Improved UI/UX

3. **Phase 3** (Nice to Have)
   - GeoTIFF export
   - KML/KMZ export
   - Cloud storage
   - Sharing features

---

## Testing Plan

1. **Unit Tests**
   - JSON serialization/deserialization
   - Coordinate transformation accuracy
   - File format validation

2. **Integration Tests**
   - Full export/import cycle
   - Cross-browser compatibility
   - Large file handling

3. **User Testing**
   - Workflow efficiency
   - Error handling
   - UI clarity