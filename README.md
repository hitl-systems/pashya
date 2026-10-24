# pashya

Small, fast, on-device document OCR. Finds the text lines on a page, reads them, and puts them back in reading order as markdown. CPU-only, 3.29 MB, no GPU and no network.

This repository holds the model file and its card. It contains no source code and no runtime.

## The file

| | |
|---|---|
| `pashya-v1-min.gguf` | 3,288,704 bytes (3.29 MB), GGUF, 128 tensors |
| sha256 | `6f4d39cea7169a41242127591a16e4b0ab80f83296bbed9943e7834767afe1f9` |
| parameters | 2,139,997 - a 130,343-parameter line detector and a 2,009,654-parameter recogniser |
| charset | 630 characters |
| quantisation | Q8_0 where the shape allows it, F32 for the 42 tensors whose fastest dimension is not a multiple of 32 |

The file is self-contained: both networks, the charset, and all 29 decode and post-processing constants. It runs under an empty environment with no arguments beyond the pack and the page.

## What it is for

Scanned and photographed document pages, read end to end on a CPU: a phone, a laptop, a Raspberry Pi. It is built to be small and to run anywhere, not to be the most accurate OCR available.

## Results

Page-level character error rate on OmniDocBench after reading-order assembly. Lower is better. Greedy decoding, no language model, one CPU thread.

Every number below was produced by running the file in this repository, not an unquantised copy of it. That distinction is not pedantry: Q8_0 changes 447 of 4,321 recognised lines against the same weights at F32.

The model has two operating points - the same weights at different detection canvases. On a 60-page stratified subset:

| | page CER | per page | peak RSS |
|---|---|---|---|
| fixed 512 canvas | 0.4856 | 5.1 s | 349 MB |
| adaptive to 2048 | **0.3872** | 9.7 s | 1,535 MB |

The fixed-512 point is the one that fits a phone or a Pi, and it costs about 0.098 page CER to get there.

On the **full 715-page set** at the adaptive point, this file scores **0.3807**. The same weights held at F32 score 0.3799, so quantisation costs 0.0008 here — against 0.0004 on the 60-page subset, and against a 0.0010 gain at the fixed canvas. The effect is small, real, and not consistently in one direction.

Speed and memory were measured separately on 18 size-stratified pages at one thread: median 3,748 ms a page at the fixed canvas and 8,210 ms adaptive. Load time is 8.8 ms.

Scoring is our own harness, not an official leaderboard run, so an official evaluation may differ.

## Limits

- **This is a small model and it reads a benchmark page wrong about a third of the time.** 0.3807 page CER is not production-grade accuracy on hard scans; it is what 3.29 MB buys.
- **The weakness is segmentation and reading order, not finding the text.** Giving this recogniser better line boxes closes about three quarters of the error while the order-free word error barely moves. The text is being found and then cut into the wrong lines and read in the wrong sequence.
- **Memory is dominated by the detection canvas, not the weights.** A 3.29 MB model peaks at 1.5 GB at the 2048 canvas, because the detector's first convolution alone holds a 268 MB activation. The fixed-512 point avoids this at a real accuracy cost.
- **630 characters.** Latin, common punctuation and a limited CJK set. Anything outside that cannot be emitted at all, whatever the image says.
- **No layout classification.** Page headers, footers, page numbers and table furniture are read as body text. A learned filter for this was built and measured and did not help on the full benchmark, so none ships.
- **Reading order is a geometric XY cut with no learned component.** It is worth about 0.14 page CER over naive raster order, and it is also where the remaining error lives.
- **No language model by default.** One exists and is worth 0.0047 page CER, which did not justify its 18.9 MB.
- **One seed.** Every number here is a single training run, not a mean over seeds.

## Reproducibility

The pipeline is deterministic: the same page and the same pack give byte-identical output across runs and processes. The numbers above were produced by the file in this repository.

## License

Creative Commons Attribution-NonCommercial-ShareAlike 4.0 (CC BY-NC-SA 4.0) for the model weights and these documents: attribution required, non-commercial use only, and adaptations must be shared under the same licence. Full text in `LICENSE`.

Training data includes the CORD receipt dataset (Park et al.), used under CC-BY-4.0.

## Status

Version 1, first public release, and the smallest of three planned sizes. The two larger ones are specified but not trained.
