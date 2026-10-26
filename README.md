# kovan-protocol/.github

Organization profile for [github.com/kovan-protocol](https://github.com/kovan-protocol).

- `profile/README.md` is rendered on the organization's landing page.
- `assets/` holds the banner, logo and organization avatar the profile and org settings use (`.svg` files are the sources; the `.png` files are what gets embedded or uploaded).

To publish: create a **public** repository named exactly `.github` under the `kovan-protocol` organization and push this folder to its default branch.

Regenerate the PNGs after editing the SVGs:

```bash
magick -density 192 -background none assets/banner.svg -resize 1600x400 assets/banner.png
magick -density 192 -background none assets/logo.svg -resize 512x512 assets/logo.png
```
magick -density 384 -background none assets/avatar.svg -resize 1024x1024 assets/avatar.png
```
