# Sudo Sequence

Minimal web viewer for finding genomic regions with high between-sample diversity. The pipeline converts multiple aligned sequencing files into fixed-width genomic "patches" and scores each patch by the variance of read counts across samples (days). The Flask app serves the top-scoring patches and lets a simple UI list them.

This project is part of the Cambridge Biohackathon and won first place.

This repo currently operates on BAM alignments (via `pysam`), not raw FASTQ. If you start from FASTQ, align to a reference and produce sorted BAM files first.

## How it works

1. `to_daru.py` loads multiple BAM files and iterates over reads in genomic order.
2. Reads are grouped into fixed-size genomic windows ("patches") based on `patch_size`.
3. For each patch, a score is computed as the variance of per-sample read counts.
4. Patches are serialized to a compact binary format (`daru.daru`) with an index (`daru.idaru`).
5. The Flask app exposes APIs to fetch top patches and per-patch read records.

## Repository layout

- `src/to_daru.py`: offline converter from BAM files to `daru` binary + index.
- `src/daru.py`: binary format definitions, read/write helpers.
- `src/__init__.py`: Flask app exposing APIs and serving the UI.
- `src/templates/index.html`: minimal Bootstrap UI to list patches.
- `setup.sh`: creates a virtualenv and installs Python dependencies.
- `server.sh`: activates the venv and launches the Flask app.

## Setup

```bash
./setup.sh
```

## Generate the data files

`src/to_daru.py` currently hardcodes four BAM paths and writes output to `daru.daru` and `daru.idaru` in the repo root.

Update the call at the bottom of `src/to_daru.py` to point at your BAMs and choose a patch size, then run:

```bash
python3 src/to_daru.py
```

Example expected output files:

- `daru.daru`
- `daru.idaru`

## Run the server

```bash
./server.sh
```

Open the app at `http://127.0.0.1:5000/`.

## API

- `GET /api/patches/`
  - Returns a JSON map of `{patch_byte_offset: score}` for the top 200 patches.
- `GET /api/patch/<patch_id>`
  - Returns a JSON array of read records for the patch at byte offset `patch_id`.

## Notes and limitations

- Input must be BAM. Convert FASTQ to BAM with your preferred aligner and sort/index as needed.
- Scoring uses variance of read counts per sample; it does not evaluate base-level diversity.
- `src/to_daru.py` uses `query_name` for `rname` in records; if you need reference name, adjust accordingly.
- The UI currently only lists patches; it does not yet plot or visualize reads.

## License

See LICENSE.
