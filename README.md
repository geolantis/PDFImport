# PDF Map Overlay Tool

A web-based tool for georeferencing PDF plans and overlaying them on interactive maps using 2-point reference system.

## Features

- 📄 **PDF Upload & Display** - Load PDF plans directly in the browser
- 🗺️ **Interactive Map** - Built on MapLibre GL with Austrian basemap and cadastral layers
- 📍 **2-Point Georeferencing** - Simple two-point reference system for accurate positioning
- 🎯 **Click-to-Select Coordinates** - Click on the map to automatically fill coordinates
- 🔍 **Transparent Background** - Remove white backgrounds from PDFs for better overlay visibility
- 🎚️ **Opacity Control** - Adjust overlay transparency in real-time
- 🌍 **Multiple Coordinate Systems** - Support for WGS84 and local coordinate systems

## How to Use

1. **Upload PDF**: Click "Upload PDF Plan" and select your PDF file
2. **Process PDF**: Click "Process & Remove Background" to make white areas transparent
3. **Start Georeferencing**: Click "Start 2-Point Referencing"
4. **Set Reference Points**:
   - Click first point on the PDF
   - Click corresponding location on the map (or enter coordinates manually)
   - Repeat for second point
5. **View Result**: The PDF will be overlaid on the map with proper scale and rotation
6. **Adjust**: Use the opacity slider to fine-tune visibility

## Technical Details

### Georeferencing Algorithm

The tool uses a 2-point affine transformation with Web Mercator projection for accurate georeferencing:

1. Converts WGS84 coordinates to Web Mercator (EPSG:3857) for metric calculations
2. Handles coordinate system differences (PDF Y-down vs Geographic Y-up)
3. Calculates rotation and scale in projected space
4. Transforms PDF corners to geographic coordinates
5. Displays as raster overlay on the map

### Technologies Used

- **MapLibre GL JS** - Interactive map rendering
- **PDF.js** - PDF rendering in browser
- **Canvas API** - Image processing and transparency
- **Web Mercator Projection** - Accurate distance calculations

## Browser Support

Works in modern browsers with support for:
- Canvas API
- ES6+ JavaScript
- WebGL (for MapLibre)

## Local Development

Simply open `pdf-map-overlay.html` in a web browser. No build process required.

For local testing with a web server:
```bash
python -m http.server 8000
# or
npx http-server
```

## Map Data Sources

- **Basemap**: Carinthian vector basemap (https://gis.ktn.gv.at/)
- **Cadastral Layer**: Austrian cadastral data with parcel boundaries

## License

This project is open source. Feel free to use and modify as needed.

## Future Enhancements

- [ ] Support for more coordinate systems (UTM, Gauss-Krüger)
- [ ] Multi-page PDF support
- [ ] Save/load georeferencing sessions
- [ ] Export georeferenced images
- [ ] Automatic legend/frame detection and removal
- [ ] Support for 3+ point georeferencing