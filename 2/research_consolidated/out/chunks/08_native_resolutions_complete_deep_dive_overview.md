# Native Resolutions Complete Deep Dive Overview

## Native resolutions, descaling, scaling, resizing, restoration of native resolutions — a complete deep-dive

Nice brief. I went through the classic fansubbing/encoding canon (Kageru, encode.moe, AniBin, SilentAperture), the practical tools people use (getnative, descale, fvsfunc, Debilinear/Debicubic, MaskDetail), the academic / algorithmic side (DPID, perceptual / L0 / spectral downscaling work, invertible rescaling, modern SR like Real-ESRGAN) and the underlying sampling/filter theory (Nyquist, Lanczos, Mitchell–Netravali). Below is a single, all-in one document with explanations, diagnostics, workflows, pitfalls, scripts, references and recommendations. I cite the key sources next to the claims they support.

⸻

TL;DR — what you need to remember first
	•	"Native resolution" in anime/video means the resolution the show was produced / painted at (often <1080p) and later upscaled to 1080p for Blu-ray releases. Detecting that native resolution and reversing the studio upscale ("descaling") can often restore crisper lineart and reduce ringing/haloing — but only if done carefully (mixed-source overlays, post-upscale sharpening, chroma subsampling and compression noise all break the assumptions).  ￼ ￼
	•	Descaling is an inversion of the studio resize: you must estimate the original height (or fractional height), the kernel (bilinear, bicubic with b/c, Lanczos, spline), and use inverse resampling (Debilinear/Debicubic/descale) with masks and chroma handling. Do not blindly descale everything — test and inspect graphs produced by getnative.py.  ￼ ￼
	•	There are modern academic/algorithmic approaches for better downscaling that try to preserve perceptual details (DPID, perceptual downscaling, L0-regularized, spectral remapping). For restoring details after downscaling/upscaling, invertible rescaling networks and SR models (IRN, Real-ESRGAN, etc.) are state-of-the-art directions.  ￼ ￼ ￼

⸻

1) Terminology & core concepts
	•	Native (production) resolution — the resolution artists/animators targeted when making frames (often 720p/810p/864p/900p/other fractional heights for anime). Blu-rays are usually 1080p, so a lot of material is an upscale of the native resolution. Identifying native resolution is the first step.  ￼ ￼
	•	Upscale / resampling kernel — the filter (bilinear, bicubic with parameters b/c, Spline16/36, Lanczos) used to produce the higher resolution. Different kernels leave different fingerprints (ringing, halo, aliasing). The Mitchell–Netravali family is commonly used (bicubic variants: b and c parameters).  ￼ ￼
	•	Descale — apply an inverse filtering operation to revert an upscale back to the native resolution (Debilinear, Debicubic, the descale plugin). If the exact kernel and resolution are correct and the image was only upscaled (no destructive postprocessing), descaling is close to lossless for lineart. If wrong, descaling introduces ringing and jaggies.  ￼ ￼
	•	Chroma subsampling — Blu-ray is 4:2:0; when you descale the luma plane you should handle chroma separately (common pattern: descale luma, spline upscale chroma to target size, then merge) to avoid color bleeding.  ￼
	•	Mixed-resolution content / overlays — credits, text, grain or 1080p overlays on a 720p background cause local mixed resolutions. You must mask those areas and avoid descaling on them.  ￼ ￼

⸻

2) How people find the native resolution (practical detection)

Typical method (used by getnative.py, Anibin, fansub guides):
	1.	Choose a clean frame: bright, little FX, low grain, clear lineart.  ￼
	2.	For a range of candidate heights (e.g. 540→1080, fractional heights too) do: descale to candidate height with a test kernel → re-upscale using the same kernel back to source resolution → compute per-pixel relative error between the original and the re-upscaled frame. Plot the relative error vs candidate height. Sharp dips (spikes downward) indicate the correct native height (the inverse mapping matches best). getnative automates this and plots graphs.  ￼ ￼
	3.	Try several kernels (bilinear, bicubic b/c combos, spline, Lanczos). Kernels have parameters (bicubic b/c) and you should sweep those too if the result is ambiguous.  ￼
	4.	Double-check visually: if descale+resample back looks identical in lineart and doesn't add ringing, you've likely found the native res. If graphs are noisy or ambiguous, don't force a descale.  ￼

Tools:
	•	getnative (Python) and forks (GitHub) for automated scanning.  ￼
	•	getfnative variants for fractional / odd heights (community scripts inspired by AniBin).  ￼ ￼
