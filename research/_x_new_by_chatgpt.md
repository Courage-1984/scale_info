Please research the following, and analyse and think deep:

Native Resolutions and descaling and Scaling and Resizing and Restoration of Native Resolutions

Also investigate:

I want to add that the following should also be investigated:

https://blog.kageru.moe/legacy/resolutions.html

https://guide.encode.moe/encoding/descaling.html

https://web.archive.org/web/20241203084912/https://ddl.kageru.moe/kbz12.pdf

https://silentaperture.gitlab.io/mdbook-guide/filtering/descaling.html

https://web.archive.org/web/20231123073420/https://anibin.blogspot.com/2014/01/blog-post_3155.html

ALSO: research DPID

FINALLY:
Create and al encompassing, comprehensive document and without letting anything out that contains info about all the above


# The Result

Native resolutions, descaling, scaling, resizing, restoration of native resolutions — a complete deep-dive

Nice brief. I went through the classic fansubbing/encoding canon (Kageru, encode.moe, AniBin, SilentAperture), the practical tools people use (getnative, descale, fvsfunc, Debilinear/Debicubic, MaskDetail), the academic / algorithmic side (DPID, perceptual / L0 / spectral downscaling work, invertible rescaling, modern SR like Real-ESRGAN) and the underlying sampling/filter theory (Nyquist, Lanczos, Mitchell–Netravali). Below is a single, all-in one document with explanations, diagnostics, workflows, pitfalls, scripts, references and recommendations. I cite the key sources next to the claims they support.

⸻

TL;DR — what you need to remember first
	•	“Native resolution” in anime/video means the resolution the show was produced / painted at (often <1080p) and later upscaled to 1080p for Blu-ray releases. Detecting that native resolution and reversing the studio upscale (“descaling”) can often restore crisper lineart and reduce ringing/haloing — but only if done carefully (mixed-source overlays, post-upscale sharpening, chroma subsampling and compression noise all break the assumptions).  ￼ ￼
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
	4.	Double-check visually: if descale+resample back looks identical in lineart and doesn’t add ringing, you’ve likely found the native res. If graphs are noisy or ambiguous, don’t force a descale.  ￼

Tools:
	•	getnative (Python) and forks (GitHub) for automated scanning.  ￼
	•	getfnative variants for fractional / odd heights (community scripts inspired by AniBin).  ￼ ￼

⸻

3) Descaling: algorithms, plugins and practical pipeline

Common plugins & scripts
	•	descale (Irrational-Encoding-Wizardry) — VapourSynth/AviSynth plugin to reverse common studio upscales (Debilinear, Debicubic, Lanczos, etc.). Use it via wrapper functions (e.g., fvf.Debilinear) and check recomposition with merged chroma.  ￼
	•	fvsfunc / kagefunc / fmtconv — helpers for VapourSynth; fmtconv can do resampling and inverse kernels. Kageru’s scripts and the encode.moe guide provide ready-to-use examples.  ￼ ￼

Practical (safe) pipeline — example (VapourSynth pseudo)

(This is the canonical “descend to native, then re-resize / mask overlays” flow.)

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
	•	DPID stands for Detail-Preserving Image Downscaling (SIGGRAPH Asia / related work). It’s an algorithm designed to produce downscaled images that retain visually important details (adaptive weights that favor pixels differing from neighborhood, edge-aware filtering). It’s intended for large downscales and preserves thin structures and stars/text better than simple bicubic. There are implementations (MATLAB, CUDA, community ports) and people reference DPID as a perceptually strong downscaler.  ￼ ￼

Practical notes:
	•	DPID is not a magic “descale” that will reproduce original masters — it’s a downscaler (purpose: produce good low-resolution images). Its parameters (lambda, etc.) affect how much detail is emphasized; users report lambda tuning matters and that perceptual benefits are most visible at large downscale ratios.  ￼ ￼

⸻

6) Research / advanced downscaling & evaluation
	•	Standard research directions beyond simple kernels:
	•	Perceptual downscaling (Óztireli & Gross) — optimize an LR image so that when upscaled (with a chosen SR) it perceptually matches HR. It treats downscaling as an optimization problem using perceptual metrics.  ￼
	•	L0-regularized downscaling — preserves salient edges by using sparsity priors on gradient ratios. Good for preserving perceptual edges in LR output.  ￼
	•	Spectral remapping / other content-aware approaches — remap high-frequency content into the representable range of the LR spectrum to avoid losing perceptual cues.  ￼
	•	Evaluation problem — comparing downscalers is hard because LR ground-truth rarely exists for real images. The CVPR 2024 paper IDA-RD proposes measuring downscaling quality by treating downscaling as encoding and measuring distortion when upscaled by a learned generative SR — a process-based measure (rate-distortion inspired). This is an example of modern ways to evaluate downscalers more meaningfully.  ￼

⸻

7) Restoration of native resolutions (the “up” side)

There are two related but distinct goals:
	1.	Reverse an earlier upscale (descale): as covered earlier — try to restore the original lower-resolution content (usually the goal for a 720p release derived from BD 1080p).  ￼
	2.	Recover lost high-frequency detail (super-resolution / reconstruction): take an LR image and try to reconstruct plausible HR detail.

Modern SR & invertible rescaling
	•	Invertible Rescaling Networks (IRN / IRN extensions, IRRMs, etc.) train a bijection between HR and LR + latent, so that a specific downscaling operator stores enough latent info to reconstruct HR later. These models are designed so if the LR was produced by the paired downscaler, the HR can be (much) better recovered. This is promising but requires the LR to have been generated by the paired model (not typical for random BDs).  ￼ ￼
	•	Real-ESRGAN / ESRGAN / SwinIR / diffusion-based SR — practical models trained to upscale “real-world” degraded images. They are useful when you want to create a visually pleasing higher-res image from a low-res source, but results are generative (plausible details, not necessarily true-to-master). The dataset and degradation model used during training matter a lot.  ￼ ￼

Takeaway: If you want to recover the true original master pixel-for-pixel, only a true inverse (true original pre-upscale master or an invertible pipeline trained with the same downscaler) can do that. In practice, descaling (finding the native height and inverse kernel) recovers the original sampling grid; SR/invertible nets can then synthesize plausible high-frequency detail, but that detail may not equal the true master.  ￼ ￼

⸻

8) Evaluation metrics — how to judge success
	•	Visual inspection (lineart, ringing, halos, check masked overlays) — always first and decisive.  ￼
	•	PSNR / SSIM — traditional, but can be misleading for perceptual quality.  ￼
	•	Perceptual metrics (LPIPS, learned metrics) better for judged visual similarity. IDA-RD is an interesting process-based measure for downscaling quality (upscale then measure distortion in HR space using generative SR models). Use multiple metrics.  ￼

⸻

9) Decision tree — when to descale vs when to conventional resize or SR
	1.	Is the lineart clean and appears soft/blurred (not post-processed) and there are no 1080p overlays? → test descaling.  ￼
	2.	Does getnative.py show a clear, strong dip (low relative error) at some candidate height for a plausible kernel? → Descale is usually safe.  ￼
	3.	Is there heavy grain, post-upscale sharpening, or mixed resolutions (credits, 1080p overlays)? → prefer standard resizers (Spline36) or use heavy masking if you insist on descaling (costly).  ￼ ￼
	4.	Goal is archival / pixel-faithful restoration to the original master? → only possible if you have the original master or an invertible pipeline trained for that specific downscaling. Otherwise, descaling + SR may produce better-looking results, but not necessarily original pixels.  ￼

⸻

10) Practical checklist & best practices (summary)
	•	Always pick a good frame for getnative (bright, minimal VFX, little grain).  ￼
	•	Run getnative.py across several kernels and inspect the error graph for sharp dips (trust your eyes if the graph is ambiguous).  ￼
	•	Use the descale plugin or Debilinear/Debicubic from fvsfunc / fmtconv for inverse kernels. Merge chroma carefully (don’t attempt to descale subsampled chroma). Mask 1080p content (credits/text) and other elements that are obviously native 1080p.  ￼ ￼
	•	If the scene has heavy postprocessing (grain added after upscaling, sharpening, overlays) — don’t descale globally; do per-scene masking or stick to conventional resampling.  ￼
	•	Evaluate with visual comparisons plus metrics (PSNR/SSIM + LPIPS/IDA-RD if you want research-level evaluation).  ￼

⸻

11) Representative references (links I used)

(These are the main practical and research sources I used while preparing this document.)
	•	Kageru — Native Resolutions and Scaling (classic fansubbing writeup; explains Debilinear/Debicubic, kernels, masking).  ￼
	•	encode.moe — Descaling (the modern fansubbing guide; getnative & descale examples and practical guidance).  ￼
	•	SilentAperture / Advanced Encoding Guide — Descaling and Rescaling (practical step-through, mask examples).  ￼
	•	getnative — GitHub repo (scripts to detect native resolution).  ￼
	•	descale plugin (Irrational-Encoding-Wizardry) — Github (VapourSynth/AviSynth plugin to undo upscales).  ￼
	•	AniBin blog (archived) — historical analyses of odd/fractional native sizes (e.g. 846p, 857p examples). Good background on how Japanese productions sometimes used fractional heights.  ￼
	•	DPID — Rapid, Detail-Preserving Image Downscaling (paper and community implementations). Good for perceptual downscaling.  ￼ ￼
	•	Mitchell–Netravali filters (b,c parameters) — Wikipedia / graphics references (why bicubic parameter choice matters).  ￼
	•	Lanczos / sinc resampling — Wikipedia (why Lanczos used in practice).  ￼
	•	Invertible Image Rescaling (IRN) — ECCV 2020 (how invertible rescaling can make downscale/upscale bijective for later reconstruction).  ￼
	•	IDA-RD (CVPR 2024) — Image Downscaling Assessment by Rate-Distortion (new measure for downscaling evaluation using generative SR).  ￼

⸻

12) Example workflows — quick decision cheat-sheet

A — Clean anime BD that looks upscaled from 720p and the lineart is soft
	1.	Pick frame → run getnative.py (bilinear + bicubic sweeps).  ￼
	2.	If clear dip: run descale (Debilinear/Debicubic) on luma; merge chroma; check rescaled back vs original. If good, produce 720p release and rely on player upscalers for quality.  ￼

B — BD with credits & frequent overlays / grain / sharpening
	•	Don’t descale globally. Use masks per scene heavily, or stick to conventional high-quality spline resize (Spline36).  ￼

C — You want to produce the best-looking 1080p from a real low-res master
	•	If you have access/control to downscaler/upsampler pair (IRN-type), train invertible rescaling; otherwise experiment with SR (Real-ESRGAN/SwinIR) but expect generative plausible detail (not true master).  ￼ ￼

⸻

13) Final thoughts & research notes (what’s active / open)
	•	Active research: better perceptual downscalers, invertible rescaling, learned downscale/upscale pairs and objective evaluation metrics (IDA-RD). These try to close the gap between aesthetically pleasing and information-preserving downscaling.  ￼ ￼ ￼
	•	Community practice: descaling remains a fansubbing/encoder craft — successful descales require human judgement, masking skill, kernel sweeps and caution. Tools like getnative, descale, MaskDetail and the scripts in Kageru/encode.moe are the community standard.  ￼ ￼ ￼

⸻

If you want, I can:
	•	produce a ready-to-run VapourSynth script template tailored to a particular BD (give me sample filename & whether it’s 4:2:0 or 4:4:4), with masking steps and getnative-style testing built in; or
	•	run a small experiment on 2–3 frames you provide (URLs or uploads): I can produce getnative graphs, suggested kernel/height, and a sample descale script for those frames (you’d run it locally).

Which do you want next?

