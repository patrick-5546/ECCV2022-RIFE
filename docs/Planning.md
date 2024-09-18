# Notes

## Infrastructure changes

- Used `uv` to create virtual environment
    - Original requirements.txt renamed to requirements.in
    - `uv pip compile requirements.in -o requirements.txt` to create lockfile
        - On Windows:

            ```
            uv pip compile requirements.in -o requirements.windows.txt --index-url https://download.pytorch.org/whl/cu124 --extra-index-url https://pypi.org/simple
            ```

    - `uv venv --python 3.11` to create virtual environment
    - `source .venv/bin/activate` to activate it
    - `uv pip sync requirements.txt` to install dependencies
        - On Windows:

            ```
            uv pip sync requirements.windows.txt --index-url https://download.pytorch.org/whl/cu124 --extra-index-url https://pypi.org/simple
            ```

- Updated Docker image to use Python 3.11 and made other optimizations

## Performance evaluations

- `python inference_video.py --exp=1 --video=videos/demo.mp4`
    - Ubuntu RTX 3050 baremetal: 4:32, 4.92it/s
    - Ubuntu RTX 3050 Docker: 4:30, 4.95it/s
    - Windows RTX 3050 baremetal: 4:26, 5.04it/s
    - WSL Ubuntu RTX 3050 baremetal: 4:27, 5.02it/s

## Breaking it down

### `inference_video.py`

Flow:

- Creates generator for video and writes frames into the read buffer
- Transforms images
    - (h, w, c) -> (batch_size=1, channels, height, width)
    - Normalizes pixel values to be in [0, 1]
    - Pads image to be a multiple of `tmp`
- Downsamples two frames and computes their SSIM
- If frames (f1, f2) are very similar
    - Second frame becomes the interpolation of f1 and f3, skipping f2
- Generates an interpolated frame unless they are very dissimilar
- Puts frames into the write buffer that writes them out to disk

Key functions (probably in increasing complexity):

- Buffer interface
- Pad
- Downsample (bilinear)
- Compute SSIM
- Interpolate

## Microarchitecture

Possible to run on DE1-SoC?
