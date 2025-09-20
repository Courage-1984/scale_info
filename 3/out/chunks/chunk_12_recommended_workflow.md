# Recommended, Repeatable Workflow

Below is the workflow recommended by experts. Do not rely on a single frame or single tool — use both spectral and error-metric methods and always visually inspect results.

## Phase 0: Preparation

- Work on a lossless extract of the video when possible (or highest quality few frame grabs)
- For video: extract a handful (3–10) representative frames that are bright and detailed and do not contain credits/overlays when possible
- Guides & tool READMEs recommend bright scenes for best detection

## Phase 1: Fast Spectral Check (resdet)

- Build/run resdet on a set of frames (it's fast, C tool)

```bash
# example: detect single frame (PNG) — resdet autodetects best guess
resdet frame.png
# for multiple frames, pass a y4m stream or a small y4m set (see README)
```

If resdet returns a clear integer native size (e.g. best guess: 1280x720) that's a strong signal. If it returns ambiguous results or unusual fractional numbers, move to the trial method.

## Phase 2: Trial / Error-Metric Sweep (getnative / kbz12 method)

- Use getnative (Vapoursynth helper) or the pseudocode in kbz12: for each candidate native size (e.g. heights 400p–1079p, or a focused range from resdet), do:
  1. inverse-rescale (descale) to candidate width×height
  2. rescale back to source resolution with the same kernel type used by the studio (bilinear/bicubic)
  3. compute absolute difference / plane stats between original and re-upscaled image
  4. the correct native size yields a local minimum in that error metric

This is exactly the method described in the kbz12 paper and implemented by getnative. Example pseudocode from kbz12 (conceptual):

```python
down = inverse_kernel(source, w, h)
up = upscale_bilinear(down, source.width, source.height)
error = mean_absolute_difference(source, up)  # PlaneStatsAverage
```

Run this on multiple frames and average or inspect the error graphs; consistent minima across frames are reliable.

## Phase 3: Fractional / Odd Sizes (GetFnative / getfscaler)

- If the trial shows minima at fractional widths/heights (or weird offsets, e.g., 955.5 vertical), use GetFnative / getfscaler which are designed to search fractional sampler offsets and fractional scales used by real studio workflows (and are part of the JET ecosystem). They will attempt to locate fractional subpixel offsets, crop offsets, or 1088→1080 crop scenarios.

## Phase 4: Descale with Masking (the Actual Inverse Operation)

- Use the descale plugin (or fmtconv with invks=True) to produce the inverse kernel result. For example in Vapoursynth (conceptual):

```py
# using fmtconv (kageru / example)
y = core.std.ShufflePlanes(src, 0, vs.GRAY)
y = core.fmtc.resample(y, 1280, 720, kernel='bilinear', invks=True)
# upsample chroma with spline, merge to YUV444, apply mask (MaskDetail/MaskedMerge)
```

If there are overlays (credits), produce a mask from the differences between source and Descale→Upscale and use MaskedMerge to keep the overlay from being inverse-scaled (kageru calls this MaskDetail approach). The kageru import script is a ready example.

## Phase 5: Validation (Automated + Human)

- **Automated**: Recompute the mean absolute difference (PlaneStatsAverage) between the original source and [descaled→rescaled]. The error should be near the minimum you saw during discovery. Kageru/kbz12 provide code and figures for this check.
- **Human**: Quickly scrub samples (especially text/credit heavy scenes and high-detail backgrounds). Look for ringing, halos, color shifts, or broken outlines — these are red flags.

## Phase 6: Produce Final Filtered Output

- If you plan to release in 720p: after descaling you may apply your chosen upscaler (re-upscale with a good kernel or leave to player), filter, encode
- If you want to store native resolution as archive: keep the descaled source (clean, smaller file) plus source and a short test report (frame comparisons and index of frames used)

## Example Commands and References

### resdet Example (from README)

```bash
magick frame.png -resize 150% resized.png
resdet resized.png
# output example: "best guess: 512x512"
```

(Resdet prefers y4m for video frames and has tips for deblocking JPEGs.)

### Descale Plugin Usage

descale.Debilinear(clip, width, height) and descale.Debicubic(clip, width, height, b, c) (see plugin wrapper docs). Use the plugin wrapper/py for convenience; the plugin expects certain planar formats (see README).

### getnative / getfnative

Read their READMEs and run them on a few frames; they are Vapoursynth-centric tools and expect a Python/Vapoursynth environment. They explicitly warn that bright/detailed frames give better results and that you should test multiple frames.

## Short Checklist

1. Pick 5–10 frames (avoid credits)
2. Run resdet on frames. If resdet yields a clear integer candidate, try that first
3. Run getnative (error sweep) across a window around the candidate(s). Average/compare frames
4. If fractional/odd: run GetFnative / getfscaler
5. Descale with descale / fmtconv + invks=True, split planes and handle chroma as recommended. Mask credits
6. Validate with PlaneStatsAverage / visual check. Use the kbz12 error measure to confirm
