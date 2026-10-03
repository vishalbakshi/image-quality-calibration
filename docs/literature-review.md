# How people judge distortion severity: a short literature review

Background for the glare calibration tool. The question: when people look at a set of images with increasing distortion, how well can they order them and say how far apart they are? And which method should the tool use to collect that judgment?

## Summary

1. "Perceptually linear" can mean four different things, and the methods that measure them don't always agree. The tool has to pick one on purpose.
2. Hand-tuned "linear" levels in KADID-10k needed nonlinear parameter spacing. Brighten, the KADID distortion closest to glare, used levels 0.1, 0.2, 0.4, 0.7, 1.1, which is exactly `0.1 + 0.05·k(k+1)`, a quadratic.
3. Typing free numbers anchored to one image is called **magnitude estimation** (Stevens). It is a standard method, and it can show power-law curves like "quadratic". It also has known biases, and the tool should design around them.
4. The most reliable methods ask for **comparisons**, not numbers: pairwise "which is worse?" or "which pair differs more?" These produce scales in just-noticeable-difference (JND) units.
5. **Absolute labels** for a single image are the weak link, especially at mild severities. That matches what the blog post found.

## 1. Four meanings of "perceptually linear"

| Method | What the rater does | Scale you get | Used in |
|---|---|---|---|
| Category rating | Picks a label (imperceptible … very annoying) | Ordinal, often treated as interval | ITU-R BT.500 DSIS, KADID-10k, TID2013 |
| Magnitude estimation | Gives a number relative to a reference ("2× as bad") | Ratio | Stevens' psychophysics; your proposal |
| Difference scaling | Says which of two *gaps* is bigger, or places images so that distance = difference | Interval | MLDS; CSIQ's displacement method |
| Discrimination / pairwise | Says which of two images is worse | JND units (Thurstone) | ISO 20462, Mantiuk et al., JPEG AIC-3 |

These scales are related but not identical:

- Stevens & Galanter (1957) found that category scales are a **nonlinear** (compressive) function of magnitude-estimation scales across a dozen perceptual continua. Equal steps in "how many times worse" are not equal steps in category labels.
- Shooner & Mullen (2022) compared difference scaling (MLDS) with discrimination thresholds for contrast. The two gave curves of similar shape, but MLDS was noisier.

The 5-point scale in the blog post (imperceptible → very annoying) is the ITU-R BT.500 Double Stimulus Impairment Scale. If the end goal is annotators applying those 5 labels consistently, then category-scale linearity is the target. The other methods are ways to get there.

## 2. What KADID-10k actually achieved

KADID-10k (Lin, Hosu, Saupe 2019) set its parameters by hand so that quality would vary "roughly linearly" across 5 levels. The DistortBench appendix reports the parameters and the crowd scores:

- **Gaussian blur σ:** 0.1, 0.5, 1, 2, 5. Roughly geometric spacing.
- **Brighten:** 0.1, 0.2, 0.4, 0.7, 1.1. The gaps are 0.1, 0.2, 0.3, 0.4, so the spacing is quadratic.
- **Mean crowd score by level** (all types together): 4.08, 3.52, 3.06, 2.50, 2.01. The steps are fairly even (about 0.5 each), but they cover only the middle half of the 1–5 range rather than "1 to 5".
- 22 of 25 distortion types had scores that never increased as the level went up.

So the hand calibration produced roughly even spacing, but squeezed into the middle of the scale, and only by using nonlinear parameter spacing.

## 3. Magnitude estimation (the free-number design)

**Method.** The rater sees a reference stimulus with a fixed number (the "modulus") and gives other stimuli numbers in proportion. Stevens' power law, `ψ = k·φ^β`, comes from this method. For brightness, β ≈ 0.33 (5° target, dark-adapted), which is strongly compressive. How annoying glare looks on a photo is not the same as brightness, so the exponent for glare is unknown. That is why it needs measuring.

**Known problems:**

- **People think in differences, not ratios.** Mertens, Mertens & Lerche (2021) showed that values below the modulus are squeezed into 0–modulus, while values above it can run to infinity. This asymmetry bends the fitted exponent: brightness came out at 0.44 with the standard method and 0.55 with their fix. The fix is a **unidirectional** method: all stimuli on one side of the reference, with responses above 10. Anchoring the mildest image already puts every other image on one side.
- **Range bias.** Poulton showed that the range of stimuli shown changes the numbers people give. This has been measured for glare specifically: Fotios & Kent (2020) report that the luminance needed for a given discomfort rating rose as the top of the stimulus range rose. In an iterative tool the range changes every round, so the ratings will drift with it unless the endpoints are fixed.
- **Labels mean different things to different people.** When asked to put the de Boer glare scale's labels in order, only 7 of 26 naive participants and 1 of 14 experts matched de Boer's order (Fotios & Kent). Free numbers avoid this problem, which is a point in favor of your design.
- **People use numbers differently.** Some raters compress their numbers and some stretch them, and they favor round numbers (10, 20, 50, 100). Average across raters in log space (geometric mean), not with an arithmetic mean.

## 4. Difference scaling (MLDS)

**Method.** In each trial the rater sees three or four images and answers: "Is the difference between A and B bigger than the difference between B and C?" A maximum-likelihood fit turns many such answers into an interval scale (Maloney & Yang 2003; the R package `MLDS` by Knoblauch & Maloney).

**Used on image quality.** Charrier, Maloney, Cherifi & Knoblauch (2007) used MLDS on 9 images compressed at 10 levels each. The method asks directly whether the gap from 1 to 2 equals the gap from 3 to 4, which is the question the tool cares about.

**Cost and setup.** Shooner & Mullen used 280–1,040 trials per fit, with stimuli about 4 discrimination thresholds apart so that judgments were neither trivial nor impossible. With 8 severity levels there are C(8,3) = 56 triads, so two passes take roughly 5–10 minutes. Aguilar, Wichmann & Maertens (2017) showed that MLDS sensitivity estimates agree with forced-choice estimates.

**Why it fits this project.** It needs no numbers, does not depend on the starting levels, and gives the whole curve in one session rather than through repeated rounds.

## 5. Pairwise comparison and JND scales

- **Pairwise is the most accurate.** Mantiuk, Tomaszewska & Mantiuk (2012) compared single-stimulus rating, double-stimulus rating, forced-choice pairwise, and similarity judgments. Forced-choice pairwise had the smallest measurement variance.
- **Tools.** Perez-Ortiz & Mantiuk (2017) released `pwcmp` for Thurstone scaling with confidence intervals and outlier checks. `ASAP` picks the most informative next pair, which cuts the number of comparisons needed. A later paper merges rating and pairwise data onto one scale.
- **ISO 20462 is the closest existing standard to this tool.** A "quality ruler" is an ordered series of reference images of one scene that differ in one attribute, spaced a known number of JNDs apart. The standard defines 1 JND as the difference that gives a 75:25 split in paired comparison, and its Standard Quality Scale uses 1 unit = 1 JND (Keelan & Urabe 2003). A calibrated glare ladder is a quality ruler for glare.
- **Near-threshold levels need help.** JPEG AIC-3 uses "boosted" triplet comparisons for distortions that are barely visible: 2× zoom, the reference-to-distorted difference amplified 2×, and 10 Hz flicker between reference and distorted. Scores are then fit with Thurstone Case V into JND units. This matters for the "imperceptible / perceptible but not annoying" end of the scale.

## 6. Absolute severity labels are the weak link

- **DistortBench** (Goyal, Eppa, Kumar 2026): three imaging experts named the distortion type correctly 83.6% of the time, but the severity level only 69.9%. Accuracy by level, from mild (L1) to strong (L5): 61.9%, 53.8%, 66.7%, 63.8%, 77.2%.
- **Absolute judgments are capacity-limited.** Miller (1956) showed that absolute identification along a single dimension tops out at about 5–7 categories (about 2.5 bits). Five severity levels is near that limit, so adjacent levels must be clearly separated.
- **Showing all levels at once.** CSIQ (Larson & Chandler 2010) showed every distorted version of an image at once across four monitors. Raters placed them horizontally so that distance matched the perceived difference in quality (35 observers, 5,000 ratings). Showing the whole set side by side is an established design.

## 7. Design decision: the rater defines the scale

Sections 1–3 show that "perceptually linear" has several competing definitions. The tool does not choose one. It asks each rater for their own 1–5 score for every image and adjusts the glare until the scores come out 1, 2, 3, 4, 5. That rater's spacing is the definition, and the curve from glare strength to score is the result.

The literature still shapes how the question is asked:

1. **Don't pre-fill numbers.** A pre-filled 1–5 invites the rater to accept it. Leave the boxes empty so each score is a fresh judgment.
2. **Allow decimals and scores outside 1–5.** Whole numbers on a 1–5 scale are too coarse to say "between 2 and 3". A bounded scale also cannot express "this is far too much", which was the objection to a fixed number line.
3. **Expect range bias at the ends.** Raters tend to stretch whatever range they are shown across the whole scale (Poulton; Parducci's range-frequency model; Fotios & Kent for glare). Left alone, the mildest image tends to get a 1 and the strongest a 5 every round, so the ends never move. The instructions should say outright that the ends can move. Showing the clean image gives a fixed reference that does not change between rounds.
4. **Save the rater's curve,** not just the final 5 values. Raters can be compared by their curves, and the shape (quadratic, log, …) is a finding in itself.
5. **End with a blind absolute-label check.** Show single images in random order. Measure accuracy per level and compare it with DistortBench's 69.9%. This is the test that matters for annotation (Claim B in the blog post).
6. **Cross-checks if the results look odd:** MLDS triads (section 4) or pairwise comparisons (section 5). Neither one asks the rater for numbers.

## Sources

- Lin, Hosu, Saupe (2019). KADID-10k: A large-scale artificially distorted IQA database. [ResearchGate](https://www.researchgate.net/publication/332567482_KADID-10k_A_Large-scale_Artificially_Distorted_IQA_Database)
- Goyal, Eppa, Kumar (2026). DistortBench: Benchmarking vision language models on image distortion identification. [arXiv:2604.19966](https://arxiv.org/html/2604.19966)
- ITU-R BT.500-14 (2019). Methodologies for the subjective assessment of the quality of television images. [ITU](https://www.itu.int/dms_pubrec/itu-r/rec/bt/R-REC-BT.500-14-201910-S!!PDF-E.pdf)
- Stevens & Galanter (1957). Ratio scales and category scales for a dozen perceptual continua. *J. Exp. Psych.* Discussed in [Ratio Scales, Category Scales, and Variability](https://repository.library.northeastern.edu/files/neu:332266/fulltext.pdf)
- Stevens' power law and brightness exponents. [Wikipedia](https://en.wikipedia.org/wiki/Stevens's_power_law)
- Mertens, Mertens, Lerche (2021). On the difficulty to think in ratios: a methodological bias in Stevens' magnitude estimation procedure. *Atten. Percept. Psychophys.* [PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC8213559/)
- Poulton, bias in magnitude scaling. [On bias in magnitude scaling and some conjectures of Stevens](https://link.springer.com/article/10.3758/BF03193541)
- Fotios & Kent (2020). Measuring discomfort from glare: recommendations for good practice. [PDF](https://eprints.whiterose.ac.uk/id/eprint/165602/3/fotios%20kent%202020%20measuring%20discomfort%20AUTHORS%20FINAL%20VERSION.pdf)
- Kent & Fotios. The effect of a pre-trial range demonstration on category rating of discomfort due to glare. [LEUKOS](https://www.tandfonline.com/doi/full/10.1080/15502724.2019.1631177)
- Maloney & Yang (2003). Maximum likelihood difference scaling. *J. Vision* 3(8). R package: [MLDS vignette](https://cran.r-project.org/web/packages/MLDS/vignettes/MLDS.pdf)
- Charrier, Maloney, Cherifi, Knoblauch (2007). Maximum likelihood difference scaling of image quality in compression-degraded images. *JOSA A* 24(11). [Optica](https://opg.optica.org/josaa/abstract.cfm?uri=josaa-24-11-3418)
- Shooner & Mullen (2022). Linking perceived to physical contrast: comparing results from discrimination and difference-scaling experiments. *J. Vision*. [PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC8787651/)
- Aguilar, Wichmann, Maertens (2017). Comparing sensitivity estimates from MLDS and forced-choice methods in a slant-from-texture experiment. *J. Vision* 17. [JoV](https://jov.arvojournals.org/article.aspx?articleid=2600611)
- Mantiuk, Tomaszewska, Mantiuk (2012). Comparison of four subjective methods for image quality assessment. *Computer Graphics Forum* 31. [PDF](https://www.cl.cam.ac.uk/~rkm38/pdfs/mantiuk12cfms.pdf)
- Perez-Ortiz & Mantiuk (2017). A practical guide and software for analysing pairwise comparison experiments. [arXiv:1712.03686](https://arxiv.org/pdf/1712.03686) · [pwcmp](https://github.com/mantiuk/pwcmp) · [ASAP](https://github.com/gfxdisp/asap) · [Unified rating + pairwise scale](https://www.cl.cam.ac.uk/~rkm38/pdfs/perezortiz2019unified_quality_scale.pdf)
- Keelan & Urabe (2003). ISO 20462: a psychophysical image quality measurement standard. *SPIE 5294*. [SPIE](https://www.spiedigitallibrary.org/conference-proceedings-of-spie/5294/0000/ISO-20462-a-psychophysical-image-quality-measurement-standard/10.1117/12.532064.short) · [ISO 20462-3 sample](https://cdn.standards.iteh.ai/samples/54617/05b49e81d1104000920755b475ebd2ca/ISO-20462-3-2012.pdf)
- JPEG AIC-3 boosted triplet comparisons. [Fine-grained subjective visual quality assessment for high-fidelity compressed images](https://www.researchgate.net/publication/384929216_Fine-grained_subjective_visual_quality_assessment_for_high-fidelity_compressed_images) · [Methodology overview](https://www.emergentmind.com/topics/jpeg-aic-3-test-methodology)
- Larson & Chandler (2010). Most apparent distortion (CSIQ database). *J. Electronic Imaging*. [PDF](https://s2.smu.edu/~eclarson/pubs/2010JEI_MAD.pdf) · [CSIQ description](http://vision.eng.shizuoka.ac.jp/mod/page/view.php?id=23)
- Miller (1956). The magical number seven, plus or minus two. *Psychological Review* 63(2), 81–97.
- Parducci (1965). Category judgment: a range-frequency model. *Psychological Review* 72(6), 407–418.
