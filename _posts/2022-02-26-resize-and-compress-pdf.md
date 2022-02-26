---
title: Resize and compress a PDF in Linux
date: 2022-02-26 00:00:00 +0100
categories: [Linux, command line]
tags: [pdf, boox]
---

![My Boox Note Air2](/assets/img/posts/boox_note_air2.jpg){: width="700"}

I recently got my [Onyx Boox Note Air2](https://shop.boox.com/products/noteair2) and I'm loving it! It's an e-ink Android tablet that can be used as a classical e-reader, but it also allows taking notes with a stylus. Boox notebooks can be exported to PDF, and these PDFs can then be retrieved through BooxDrop on your PC.

The only problem is that these PDFs are *huge*. A 15-page notepad I exported weighs 29 Mb. Here is my solution to this problem:
1. First, I scale the PDF to 50% size. While this does not impact the file size significantly, it reduces the physical paper size to something more reasonable (the Boox Note Air2 is approximately A5 size).
2. Then, I compress the PDF by specifying the DPI to be something that I still find readable while the file size is greatly reduced (by a factor 10 from my tests).

First, install the required packages: `cpdf` and `imagemagick`.

Scale PDF to 50%:

```bash
cpdf -scale-page "0.5 0.5" in.pdf -o in_scaled.pdf
```

Compress PDF (100 DPI, specified below as `100x100`, works fine for me):

```bash
convert -density 100x100 -compress jpeg in_scaled.pdf out.pdf
```

With the above, I managed to shrink my 15-page notebook from 29 Mb to only 2.8 Mb.