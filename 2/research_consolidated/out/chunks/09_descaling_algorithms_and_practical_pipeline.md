# Descaling Algorithms and Practical Pipeline

## 3) Descaling: algorithms, plugins and practical pipeline

Common plugins & scripts
	•	descale (Irrational-Encoding-Wizardry) — VapourSynth/AviSynth plugin to reverse common studio upscales (Debilinear, Debicubic, Lanczos, etc.). Use it via wrapper functions (e.g., fvf.Debilinear) and check recomposition with merged chroma.  ￼
	•	fvsfunc / kagefunc / fmtconv — helpers for VapourSynth; fmtconv can do resampling and inverse kernels. Kageru's scripts and the encode.moe guide provide ready-to-use examples.  ￼ ￼

Practical (safe) pipeline — example (VapourSynth pseudo)

(This is the canonical "descend to native, then re-resize / mask overlays" flow.)

# pseudocode (VapourSynth style) — adapt to your environment
src = core.lsmas.LWLibavSource("bd_stream.m2ts")
# 1) Use getnative.py offline to find best (height, kernel, b/c) for a chosen frame
# 2) Descale luma using discovered parameters
descale = fvf.Debilinear(src, 1280, 720)   # example; or Debicubic with b/c
# 3) Re-upscale the descale to 1080p for visual check
rescaled = descale.resize.Bilinear(src.width, src.height)
# 4) Merge the chroma from source (keep chroma untouched so 4:2:0/4:4:4 differences are handled)
merge_chroma = rescaled.std.Merge(src, [0,1])
# 5) Create mask for text/1080p overlays & merge conventional downscale in masked areas
noalias = core.fmtc.resample(src, 1280,720, kernel='blackmanminlobe', taps=5)
mask = maskDetail(src, 1280,720, kernel='bilinear')  # MaskDetail plugin
final = core.std.MaskedMerge(noalias, descale, core.std.Invert(mask))

Notes / why this form:
	•	merge_chroma preserves chroma details from original; descaling only affects luma where the upscaling fingerprint resides.  ￼ ￼
	•	Where there are native 1080p overlays (credits, typeset, text), descaling will ruin them — mask and fallback to conventional resizer for those areas.  ￼

Common gotchas
	•	Wrong kernel → severe jaggies/ringing (see Kageru examples). You must test multiple kernels and bicubic b/c values (b + 2c = 1 is a common family; Mitchell uses b = c = 1/3).  ￼ ￼
	•	Mixed resolution layers (backgrounds at different resolution than characters) require per-scene/per-layer masking — labor intensive; sometimes just use Spline36 or release the BD untouched.  ￼
	•	Blu-Ray lossy compression & transport noise means you will rarely reconstruct a frame-perfect master — small differences remain; still, descaling often improves perceived sharpness for anime.  ￼

⸻

4) Sampling, kernels and why kernel choice matters (short theory)
	•	Image down/up-sampling is fundamentally a sampling problem. To avoid aliasing you want a low-pass (anti-alias) filter before downsampling; conversely, kernels used for upscaling will produce characteristic ringing/halo signatures depending on their impulse response. The Nyquist-Shannon sampling theorem and anti-alias filtering explain why wrong inverse kernels create artifacts.  ￼
	•	Common kernels
	•	Bilinear — simple, low cost, noticeable softness.
	•	Bicubic (Mitchell–Netravali family) — widely used; parameters b,c control sharpness vs ringing. Mitchell default b=c=1/3 is a good compromise; many studios pick other b/c values.  ￼
	•	Spline36 / Spline16 — popular in fansubbing (Spline36 often default in fmtconv) — smooth, fewer ringing artifacts.  ￼
	•	Lanczos (a windowed sinc) — closer to ideal sinc reconstruction; tends to be sharper but can ring for high contrast edges.  ￼
	•	In short: the kernel defines the fingerprint — use getnative to identify it, then use the inverse kernel matching that fingerprint.  ￼

⸻

5) DPID — what it is and where it fits
	•	DPID stands for Detail-Preserving Image Downscaling (SIGGRAPH Asia / related work). It's an algorithm designed to produce downscaled images that retain visually important details (adaptive weights that favor pixels differing from neighborhood, edge-aware filtering). It's intended for large downscales and preserves thin structures and stars/text better than simple bicubic. There are implementations (MATLAB, CUDA, community ports) and people reference DPID as a perceptually strong downscaler.  ￼ ￼

Practical notes:
	•	DPID is not a magic "descale" that will reproduce original masters — it's a downscaler (purpose: produce good low-resolution images). Its parameters (lambda, etc.) affect how much detail is emphasized; users report lambda tuning matters and that perceptual benefits are most visible at large downscale ratios.  ￼ ￼
