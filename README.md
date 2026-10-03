# Image quality calibration

A single-page tool that calibrates synthetic glare against one person's sense of severity, where 1 is the mildest glare and 5 is the worst.

Background: [Image quality labels](https://vishalbakshi.com/blog/posts/2026-10-02-image-quality-labels/) and [docs/literature-review.md](docs/literature-review.md).

## How it works

1. Upload images. Everything runs in the browser and nothing is uploaded to a server. Five images works well.
2. Each image is shown as the clean original followed by five glare strengths, from weakest to strongest, with no labels.
3. The rater gives each image a 1–5 score. Decimals are allowed, and so are scores below 1 or above 5.
4. On submit, the tool takes the median score for each level across all images. It fits an increasing curve from log glare strength to score (isotonic regression over every round so far) and picks the strengths where that curve crosses 1, 2, 3, 4 and 5. If even the strongest image tested isn't a 5, or the faintest isn't a 1, the next round goes past it: 1.5× to 3× further for each missing score point. All images are then re-rendered at the new strengths.
5. Rounds repeat until every level's median is within 0.25 of its target. **Export JSON** saves the strengths, the fitted curve, the glare settings and every score.

Clicking an original image moves the glare center for that image. Clicking a glare image enlarges it, and you can enter its score there.

## Glare model

Glare adds light in linear RGB and saturates at white:

```
out = L + a · (1 − L),   a = min(1, strength · G(x, y))
```

`G` is an elliptical Gaussian core plus a wider, weaker halo. The core's size, aspect and angle and the default center are seeded from the file name and size, so each image keeps the same glare placement across rounds and reloads. Only `strength` changes between levels. Images are downscaled so the longest side is at most 768 px before rendering.

## Example images

These are mostly bookshelves (text on spines) plus a few standard test images with dark, bright and textured areas. Additive glare is much easier to see on dark regions, so a mix shows how much the scores depend on the image.

**Kodak Lossless True Color Image Suite** (768×512, released by Kodak for unrestricted use):

- [kodim08](https://r0k.us/graphics/kodak/kodim08.html): buildings with painted lettering
- [kodim01](https://r0k.us/graphics/kodak/kodim01.html): brick and wood texture
- [kodim02](https://r0k.us/graphics/kodak/kodim02.html): red door, smooth dark areas
- [kodim20](https://r0k.us/graphics/kodak/kodim20.html): airplane on a bright sky
- [kodim23](https://r0k.us/graphics/kodak/kodim23.html): parrots, saturated color
- [All 24 images](https://r0k.us/graphics/kodak/)

**Bookshelves on Wikimedia Commons:**

| Image | License |
|---|---|
| [Bookshelf (Unsplash)](https://commons.wikimedia.org/wiki/File:Bookshelf_(Unsplash).jpg) | CC0 |
| [Library shelf](https://commons.wikimedia.org/wiki/File:Library_shelf.jpg) | CC BY-SA 4.0 |
| [Books on a bookshelf](https://commons.wikimedia.org/wiki/File:Books_on_a_bookshelf.jpg) | CC BY-SA 4.0 |
| [Bookshelf (19404685092)](https://commons.wikimedia.org/wiki/File:Bookshelf_(19404685092).jpg) | CC BY-SA 2.0 |
| [Book shelf](https://commons.wikimedia.org/wiki/File:Book_shelf.jpg) | CC BY-SA 3.0 |

**[KADID-10k](https://database.mmsp-kn.de/kadid-10k-database.html)** has 81 reference images (512×384, Pixabay License). They are the clean images behind the DeepFL-IQA distortions, but they come only inside the full ~3 GB download.

Images are not stored in this repo. If you publish rendered images, credit the authors of the CC BY-SA photos.

## Run locally

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000.

## Hosting

GitHub Pages: **Settings → Pages → Deploy from a branch → `main` / root**.
