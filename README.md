# PngSaveJS

Save and load 32-bit RGBA PNG images in about 75 lines of plain JavaScript. No dependencies, no compression. Written to be read: a short way to learn how a PNG file is actually built.

## What it does

- `encodePNG(w, h, rgba)` turns RGBA pixels into PNG file bytes (a `Uint8Array`).
- `decodePNG(bytes)` turns those bytes back into `{ w, h, rgba }`.
- Pixels round-trip exactly, with no rounding. The pixel bytes are the same format `getImageData` gives and `ImageData` takes.

## How a PNG is built

```
PNG file = signature (8 bytes) + chunks: IHDR, IDAT, IEND
chunk    = LENGTH (4) + TYPE (4) + DATA + CRC (4)      big-endian, CRC over TYPE+DATA
IHDR     = width, height, bit depth 8, colour type 6 (RGBA), 0, 0, 0
IDAT     = zlib stream: 78 01 + deflate blocks + Adler-32 of the raw rows
row      = filter byte 0 + the row's RGBA pixels
```

The trick that keeps it small: each row becomes one *stored* (uncompressed) deflate block, consisting of BFINAL, LEN and NLEN (little-endian), then the row bytes unchanged. Real PNG tools compress instead, but this is a fully valid PNG that any viewer opens.

The loader runs the same steps backwards. It checks the signature, verifies each chunk's CRC, joins the stored blocks back into rows, and drops each row's filter byte.

## Limits (on purpose, to stay simple)

- Width is at most 16383 pixels, so that one row fits in one 64 KB stored block.
- Files are big: about 4 bytes per pixel.
- The loader reads uncompressed 8-bit RGBA PNGs whose rows use filter 0. That covers everything this code saves. Compressed or filtered PNGs from other programs get a clear error message; load those with the browser instead (`createImageBitmap`).

## Use

Open `pngsimple.html` in a browser. *Save PNG* downloads the canvas and *Load PNG* reads it back. To use it elsewhere, copy `crc32`, `be32`, `get32`, `encodePNG` and `decodePNG`.
