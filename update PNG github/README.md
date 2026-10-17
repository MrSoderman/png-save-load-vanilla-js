# PngSaveJS
A tiny PNG **saver** and **loader** written from scratch in plain JavaScript,
in a single HTML file. No libraries, no compression, nothing hidden.

It is a great way to learn about PNG image handling : the code is short, flat, and commented for someone who has never looked inside a PNG file before.

## What it does

| | |
|---|---|
| **Saves** | the canvas as a 32-bit RGBA PNG (8 bits each for R, G, B, A), uncompressed |
| **Loads (own loader)** | uncompressed 8-bit RGBA (32-bit) and RGB (24-bit) PNGs, from this saver or any other program |
| **Loads (standard)** | any PNG at all, using the browser's built-in decoder |

## How to use it

1. Open `index.html` in a browser.
2. **Save PNG** downloads the canvas as `canvas.png`.
3. **Load PNG** opens a PNG file and draws it on the canvas (the canvas
   resizes to fit the image).

## Choosing the loader

The browser's standard loader is on by default. To use our own loader
instead, swap which line is commented out near the bottom of the file:

```js
loadWithBrowser(canvas, file);          // <-- standard loader
// loadWithOwnDecoder(canvas, file);    // <-- our own loader (uncompressed PNGs only)
```

Why use our own loader? It puts every pixel back **exactly** as it was saved.
The browser's loader can round semi-transparent colours very slightly.

## What the own loader refuses

These give a clear error message (never wrong pixels). Use the standard
loader for them:

- compressed PNGs (what almost all programs save by default)
- greyscale, grey + alpha, and palette (indexed-colour) images
- 16-bit images
- interlaced images

## PNG in a nutshell

A PNG file is:

1. **Signature**: 8 fixed bytes saying "I am a PNG".
2. **Chunks**: each one is `LENGTH | TYPE | DATA | CRC`.
   - `IHDR`: width, height, bit depth, colour type
   - `IDAT`: the pixel data (may be split over several IDAT chunks)
   - `IEND`: the end
3. **Pixel data**: stored row by row. Each row starts with a **filter byte**
   (0 = no filter). All the rows together are wrapped in a **zlib** stream:
   `2-byte header | DEFLATE blocks | Adler-32 checksum`.

The saver uses filter 0 and **stored** DEFLATE blocks (no compression). That
is 100% valid PNG. The files are just bigger.

The source file explains all of this step by step in its comments.

## The bug that was fixed

A stored DEFLATE block holds at most **65,535 bytes**, because its length
field is 16 bits. The original saver put the whole image into one block.
A 100×100 image (40,100 bytes) fitted by luck, but anything larger than
about 127×127 overflowed and produced a broken PNG. The saver now splits the
data into as many blocks as it needs.

## Code layout (`index.html`)

| Section | What's in it |
|---|---|
| PNG in a nutshell | the beginner explanation, as a comment |
| Checksums | `crc32` (for every chunk) and `adler32` (for zlib) |
| Byte helpers | `put32`, `get32` (big-endian), `putChunk` |
| Encoder | `encodePNG(w, h, rgba)` returns the PNG file bytes |
| Optional loader | `decodePNG(bytes)` sets `pngW`, `pngH`, `pngRGBA` |
| Page wiring | buttons, the two loaders, and the demo picture |

## Tested

- Save then load, byte-exact, at sizes from 1×1 up to 1000×700
- Uncompressed RGB and RGBA PNGs made by another program (Python's image library)
  load byte-exact
- Compressed PNGs are refused with the right message

## Author

Jimmy Söderman
