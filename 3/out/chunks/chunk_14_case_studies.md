# Case Studies and Real-World Applications

## Anime Content Analysis

### Studio-Specific Patterns

- **Kyoto Animation**: Historical use of 955.5p for features
- **CloverWorks**: Lanczos-878 upscaling with minimal taps
- **A-1 Pictures**: Post-production sharpening complicating detection
- **Various Studios**: Common bilinear/bicubic upscales from 720p-900p

### Common Native Resolutions

- 720p (1280×720): Traditional TV anime standard
- 810p (1440×810): Intermediate production resolution
- 846p (1504×846): Non Non Biyori Repeat example
- 864p (1536×864): Modern production standard
- 900p (1600×900): High-quality production target

### Fractional Resolutions

- 955.5p: Kyoto Animation feature films
- 878p: Various modern productions
- 873p: Some TV series
- Results from 1088p upscale followed by 1080p crop

## Success Case Studies

### Manaria Friends (878p Detection)

- Clear error graph with distinct spike at 878p
- Successful bicubic descaling with b=0, c=1/2
- Significant quality improvement over standard resizing
- Effective masking for credit sequences

### Nichijou (720p Bilinear)

- Classic example of bilinear upscale from 720p
- Clear frequency domain signatures
- Successful restoration with minimal artifacts
- Demonstrated superiority over Spline36 resizing

### Aikatsu Friends! (810p)

- Very clear error graph with distinct minimum
- Successful kernel identification and application
- Excellent example of clean source material
- Minimal post-processing artifacts

## Failure Case Studies

### Miru Tights (Unclear Results)

- Ambiguous error graph with multiple candidates
- No distinct spikes indicating native resolution
- Recommended against descaling due to uncertainty
- Example of when to avoid descaling

### Black Lagoon (Problematic Source)

- Unnatural graph swerves and multiple candidates
- Heavy post-processing complicating detection
- Impossible to determine reliable native resolution
- Case study in source quality importance

### Kizumonogatari (Letterboxing Issues)

- Distorted results due to letterboxing
- Required cropping before analysis
- Demonstrates importance of proper frame preparation
- Successful after letterboxing removal

## Professional Workflow Examples

### Beatrice-Raws Case Study

- Cleaned Lanczos upscaling artifacts
- Used frequency filtering and detail preservation
- Reduced audio bit depth to accommodate video bandwidth
- Comprehensive quality optimization approach

### Fansubbing Community Practices

- Systematic approach to source analysis
- Multiple tool validation for accuracy
- Conservative application when uncertainty exists
- Emphasis on visual verification over algorithmic results
