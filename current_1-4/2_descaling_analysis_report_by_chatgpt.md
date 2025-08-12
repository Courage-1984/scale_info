
Executive summary (TL;DR):
- Descaling is a well-understood, mostly reversible operation for deterministic resizers (bilinear, bicubic, etc.). That theoretical basis is in the Kageru paper (kbz12) and the descale plugin's design.
- Two complementary detection approaches exist and are used in practice: (A) frequency / Fourier analysis (the “spectrogram” approach used by Anibin and tools like resdet) and (B) error-metric / trial-descale (try candidate native sizes, rescale back, compute difference) — both are recommended and often used together.
- Practical tooling is mature: resdet (FFT / spectral), getnative (error metric scripts, VapourSynth ecosystem), GetFnative / getfscaler (fractional/native detection & frac-descale helpers), and the descale plugin (VapourSynth/AviSynth) are the main player set. Use more than one tool and several frames to reduce false positives.
- Big caveats: overlays (credits/text), studio postprocessing (sharpening/grain added after upscale), mixed-resolution layers, heavy lossy compression, and neural upscalers (waifu2x/NN models) break reversibility or detection. Always validate visually and with an error metric.

What each resource contributes (short):
- kageru.moe (legacy article + script) — practical, hands-on recipes: debilinear/debicubic, masking for credits, splitting luma/chroma and producing 4:4:4 outputs; includes an importable VapourSynth script debilinearM and an explanation of masks and subsampling. Useful as a working example.
- kbz12.pdf (Kageru paper) — rigorous explanation: kernel math, why inversion is possible (A·x=b viewpoint), Fourier method for identifying missing high frequencies, and the error-measurement / trial approach (descale → re-upscale → compare). Good theory + short pseudocode for an error function.
- guide.encode.moe — up-to-date fansubbing guide that covers practical pitfalls and stepwise filtering pipelines (descale → filter → reupscale) and emphasizes testing/visual checks. Good “how you should do it in a release”.
- silentaperture / mdbook guide — another practical writeup that emphasizes descaling benefits and cautions (bad descaling gives halos, ringing). Useful second opinion on recommended tooling and masks.
- anibin (archived) — the classic FFT-based investigations on native / fractional, real examples of odd fractional sizes and studio quirks — extremely valuable as a reference dataset and for intuition.
- resdet — a C utility (lib + CLI) implementing spectral detection (FFT) to guess the source resolution from upscaled images/frames. Fast and robust on clean frames; tends to fail on neural resizers and heavy compression. Use it as the first, fast check.
- getnative — a VapourSynth/Python tool using the error measurement (trial descales and computes plane stats) to find candidate native sizes; recommended to run on several frames because success varies by scene brightness/detail.
- GetFnative / getfscaler — scripts to find fractional native resolutions (non-integer scaling factors) and helper wrappers for fractional descaling (JET family). Useful for those awkward non-integer studio cases and for the modern JET toolchain.
- descale plugin — the canonical VapourSynth/AviSynth plugin used to actually undo linear interpolation (Debilinear / Debicubic etc.). Use this for the core inverse operation; wrapper scripts will handle chroma planes and convenience.

Common failure modes (and how to spot them):
1. Credits / native 1080 overlays — result: ringing/artifacts around text. Fix: mask those areas and fall back to a conventional resizer for masked areas. (Kageru shows masked merges.)
2. Studio post-processing applied after upscale (sharpen / grain / noise injection) — result: cannot perfectly invert; descaling will produce halos or missing detail. Spot it: masked detail map shows residuals across the image.
3. Chroma subsampling mismatch (4:2:0 → 4:4:4 decisions) — If you only descale luma, chroma may be undersampled; the recommended approach is to descale luma and properly resample chroma, or produce 4:4:4 outputs to avoid color bleeding. Kageru covers plane splitting for this.
4. Neural upscalers — not reversible by these algorithms and will confuse spectral tools. resdet explicitly warns about NN upscalers.
5. Heavy compression / JPEG artifacts — spectral detectors can produce false spikes (multiples of 1/8) — resdet README documents mitigation (deblocking before analysis).

Recommended, repeatable workflow (practical — try this):
Below is the workflow I recommend. Do not rely on a single frame or single tool — use both spectral and error-metric methods and always visually inspect results.

0) Preparation:
- Work on a lossless extract of the video when possible (or highest quality few frame grabs). For video: extract a handful (3–10) representative frames that are bright and detailed and do not contain credits/overlays when possible. (Guides & tool READMEs recommend bright scenes for best detection.)

1) Fast spectral check (resdet):
- Build/run resdet on a set of frames (it's fast, C tool). Example (taken from README):

```bash
# example: detect single frame (PNG) — resdet autodetects best guess
resdet frame.png
# for multiple frames, pass a y4m stream or a small y4m set (see README)
```

If resdet returns a clear integer native size (e.g. best guess: 1280x720) that’s a strong signal. If it returns ambiguous results or unusual fractional numbers, move to the trial method.

2) Trial / error-metric sweep (getnative / kbz12 method):
- Use getnative (VapourSynth helper) or the pseudocode in kbz12: for each candidate native size (e.g. heights 400p–1079p, or a focused range from resdet), do:
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

3) Fractional / odd sizes (GetFnative / getfscaler):
- If the trial shows minima at fractional widths/heights (or weird offsets, e.g., 955.5 vertical), use GetFnative / getfscaler which are designed to search fractional sampler offsets and fractional scales used by real studio workflows (and are part of the JET ecosystem). They will attempt to locate fractional subpixel offsets, crop offsets, or 1088→1080 crop scenarios.

4) Descale with masking (the actual inverse operation):
- Use the descale plugin (or fmtconv with invks=True) to produce the inverse kernel result. For example in VapourSynth (conceptual):

```py
# using fmtconv (kageru / example)
y = core.std.ShufflePlanes(src, 0, vs.GRAY)
y = core.fmtc.resample(y, 1280, 720, kernel='bilinear', invks=True)
# upsample chroma with spline, merge to YUV444, apply mask (MaskDetail/MaskedMerge)
```

If there are overlays (credits), produce a mask from the differences between source and Descale→Upscale and use MaskedMerge to keep the overlay from being inverse-scaled (kageru calls this MaskDetail approach). The kageru import script is a ready example.

5) Validation (automated + human):
- Automated: Recompute the mean absolute difference (PlaneStatsAverage) between the original source and [descaled→rescaled]. The error should be near the minimum you saw during discovery. Kageru/kbz12 provide code and figures for this check.
- Human: Quickly scrub samples (especially text/credit heavy scenes and high-detail backgrounds). Look for ringing, halos, color shifts, or broken outlines — these are red flags.

6) Produce final filtered output:
- If you plan to release in 720p: after descaling you may apply your chosen upscaler (re-upscale with a good kernel or leave to player), filter, encode. If you want to store native resolution as archive: keep the descaled source (clean, smaller file) plus source and a short test report (frame comparisons and index of frames used).

Example quick commands & references:
- resdet example (from README):

```bash
magick frame.png -resize 150% resized.png
resdet resized.png
# output example: "best guess: 512x512"
```

(Resdet prefers y4m for video frames and has tips for deblocking JPEGs.)

- Descale plugin usage (wraps plugin; supported calls):

descale.Debilinear(clip, width, height) and descale.Debicubic(clip, width, height, b, c) (see plugin wrapper docs). Use the plugin wrapper/py for convenience; the plugin expects certain planar formats (see README).

- getnative / getfnative — read their READMEs and run them on a few frames; they are VapourSynth-centric tools and expect a Python/VapourSynth environment. They explicitly warn that bright/detailed frames give better results and that you should test multiple frames.

Short checklist you can copy/paste:
1. Pick 5–10 frames (avoid credits).
2. Run resdet on frames. If resdet yields a clear integer candidate, try that first.
3. Run getnative (error sweep) across a window around the candidate(s). Average/compare frames.
4. If fractional/odd: run GetFnative / getfscaler.
5. Descale with descale / fmtconv + invks=True, split planes and handle chroma as recommended. Mask credits.
6. Validate with PlaneStatsAverage / visual check. Use the kbz12 error measure to confirm.

Where the research adds value vs. just “using a plugin”:
- The paper (kbz12) gives the rigorous justification for why the linear systems can be inverted and why the trial/error approach works — this helps tune parameters and understand edge cases.
- The kageru article + scripts are a practical cookbook that handles real-world messy sources (masks, chroma handling). Keep those scripts handy; they’re a pragmatic baseline.
- resdet gives a fast spectral inspection that frequently produces a reliable integer result in one command.

Final recommendations / next steps I can do for you:
Tell me which of the following you want me to do next (I can do one of these now in this session):
1. Run a diagnostic plan on a sample frame(s) you upload (I’ll run resdet-style spectral analysis and a trial descaling sweep and return plots & recommended target sizes). — you’d need to upload frames or point to samples.
2. Produce a ready-to-run VapourSynth script that ties together best practices here (select frames, descale, mask, convert to 4:4:4, compute PlaneStatsAverage) tuned to a target height (e.g., 720p). I’ll include command lines and exact package dependencies.
3. Walk through how to install and run each tool locally (resdet, getnative, GetFnative, descale) with exact commands for your OS (Linux / Windows).

Pick one (1), and I’ll produce the full, runnable artifact (script, commands, or test) — and I’ll cite the exact lines in the repos/readmes I used.

If you want, I can start by analyzing a couple of frames from your source (upload 3–5 PNG/Y4M frames, avoiding full credits) and produce the detection graphs + a suggested descaling target. Which would you like?
