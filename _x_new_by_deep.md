## Comprehensive Analysis of Native Resolutions, Descaling, and Video Restoration

### **1. Fundamentals of Native Resolutions**  
**Native resolution** refers to the original pixel dimensions at which animated content (e.g., anime) was produced. Most anime is created at resolutions below 1080p (e.g., 720p, 810p, or 900p) and later upscaled to 1080p for Blu-ray releases. This upscaling process interpolates pixels to fit higher resolutions, often introducing artifacts like blurring or haloing. Restoring the native resolution through **descaling** reverses this upscaling, yielding sharper results and smaller file sizes .

---

### **2. Descaling: Theory and Techniques**  
#### **2.1 Inverse Kernel Rescaling**  
Descaling uses inverse kernels to revert studio upscaling. Common kernels include:  
- **Bilinear**: Simple but prone to blurring. Inverse: `Debilinear`.  
- **Bicubic**: Offers parameter control (`b`, `c`). Inverse: `Debicubic` .  
- **Lanczos**: Sharp but may introduce ringing.  

*Example VapourSynth code for descaling:*  
```python  
# Bilinear descaling to 720p  
descaled = core.descale.Debilinear(src, 1280, 720)  
```

#### **2.2 Why Descaling Outperforms Conventional Resizing**  
- **Preservation of Detail**: Descaling reconstructs original lineart, while spline36 resizing often exacerbates haloing .  
- **Efficiency**: Smaller native resolutions reduce encode bitrates by 30–50% without quality loss .  
- **Playback Benefits**: Players like MadVR can re-upscale with high-quality algorithms, enhancing viewer experience .  

---

### **3. Identifying Native Resolutions**  
#### **3.1 Tools and Workflows**  
- **getnative.py**: A Python script that analyzes frames to detect native resolutions. It descales/re-upscales frames and calculates "relative error" to identify resolution spikes (e.g., a dip at 720p indicates native 720p) .  
- **Ideal Frames for Analysis**: Bright, static scenes with minimal blur, grain, or VFX (e.g., clear lineart backgrounds). Dark or grainy frames yield unreliable results .  

*Interpreting getnative.py Output*:  
- **Clear Spike** (e.g., 810p): Safe to descale .  
- **Ambiguous Graph** (e.g., multiple peaks): Avoid descaling .  

#### **3.2 Anibin Database**  
A Japanese blog ([anibin](https://anibin.blogspot.com)) catalogs native resolutions for anime. Users search using original Japanese titles (e.g., "のんのんびより") to find values like 1504×846 .  

---

### **4. Practical Descaling Challenges**  
#### **4.1 Artifacts and Pitfalls**  
- **Ringing/Aliasing**: Caused by incorrect kernel/resolution mismatches (e.g., debilinearizing native 1080p) .  
- **Mixed Resolutions**:  
  - **Overlays**: Credits/text in 1080p over descaled backgrounds cause artifacting. Solution: Mask 1080p elements with `DescaleM` (spline36 downscaling) .  
  - **Per-Scene Changes**: Rare cases require frame-by-frame resolution adjustments .  
- **Compression Noise**: Blu-ray compression complicates perfect descaling .  

#### **4.2 Kernel Mismatch**  
Studios may use non-bilinear kernels (e.g., bicubic with `b=0, c=1`). Test kernels visually or via `getnative.py -k all` .  

---

### **5. Advanced Workflows**  
#### **5.1 Toolchain Integration**  
- **Frameservers**: VapourSynth (recommended) or AviSynth .  
- **Plugins**: `fmtconv` (resampling), `kagefunc` (masking), and `fvsfunc` (wrapper for `DescaleM`) .  
- **Preview Editors**: VSEdit or VS Multi-Viewer for filter tuning .  

#### **5.2 Filter Order Best Practices**  
1. Crop borders → Descale → Denoise → Deband → Grain → Output .  
2. **Chroma Handling**: Merge source chroma with descaled luma to avoid subsampling issues .  

---

### **6. DPID: Context and Irrelevance**  
**DPID (Decentralized Persistent Identifiers)** are cryptographic fingerprints for research data (e.g., scientific artifacts). They ensure uniqueness, immutability, and machine-actionable resolution via protocols like SideTree. While valuable for FAIR data principles, DPIDs are unrelated to video descaling and serve entirely different domains .  

---

### **7. Key Takeaways and Workflow**  
| **Step** | **Action** | **Tools** |  
|----------|------------|----------|  
| 1. Source Analysis | Check for blur/ringing; select test frames | VSEdit, `getnative.py` |  
| 2. Detect Native Res | Run `getnative.py` on ideal frames | Python, reference anibin |  
| 3. Descaling | Apply inverse kernel; mask overlays | `fmtconv`, `fvsfunc.DescaleM` |  
| 4. Validation | Compare source vs. rescaled output | VS Multi-Viewer |  

**Golden Rules**:  
- Never descale without confirming native resolution .  
- Prioritize visual inspection over algorithmic guesses .  
- Mixed-resolution sources often preclude descaling .  

For further implementation details, see:  
- [VapourSynth Descaling Guide](https://guide.encode.moe/encoding/descaling.html)   
- [Native Resolution Theory](https://blog.kageru.moe/legacy/resolutions.html)   
- [Advanced Artifact Handling](https://silentaperture.gitlab.io/mdbook-guide/filtering/descaling.html)

