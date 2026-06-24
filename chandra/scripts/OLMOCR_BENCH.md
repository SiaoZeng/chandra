# Reproducing Chandra OCR 2 on olmOCR-bench

End-to-end reproduction of Chandra-OCR-2's score on the upstream
[olmOCR-bench](https://github.com/allenai/olmocr) (`allenai/olmOCR-bench`).

**Reference result:** ~**85.8%** overall.  This is slightly below the 85.9% we measured at launch - this is mainly due to some text normalization we used in our launch benchmarking that differed from the standard olmocr benchmark.

---

## 1. Clone + install Chandra

```bash
git clone https://github.com/datalab-to/chandra.git
cd chandra
pip install -e .                 # or: pip install chandra-ocr
pip install huggingface_hub unicodeit     # for the bench
```

## 2. Serve the model with vLLM

In one terminal (needs a GPU + Docker; see the main README):

```bash
chandra_vllm                     # serves datalab-to/chandra-ocr-2 on :8000
```

This uses the repo's serving config.

## 3. Install the upstream olmOCR bench (for scoring)

```bash
pip install "olmocr[bench]"
playwright install-deps && playwright install chromium   # KaTeX math rendering
```

## 4. Run the benchmark

In a second terminal:

```bash
python -m chandra.scripts.olmocr_bench --bench-dir ./olmOCR-bench/bench_data
```

This downloads olmOCR-bench (first run only), OCRs all ~1,400 pages via the vLLM
server, postprocesses, and prints the upstream score.

### Useful flags
- `--image-dpi 300` (default)
- `--workers 32` — concurrent vLLM requests.
- `--vllm-api-base http://localhost:8000/v1` — override the server URL.
- `--skip-inference` — reuse candidate markdown already written; only re-score.
- `--skip-scoring` — only produce candidate markdown (prints the olmocr command).
- `--stock` - use stock settings/no chandra format correction