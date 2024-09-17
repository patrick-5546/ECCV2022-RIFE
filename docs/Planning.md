# Notes

## Infrastructure changes

- Used `uv` to create virtual environment
    - Original requirements.txt renamed to requirements.in
    - `uv pip compile requirements.in -o requirements.txt` to create lockfile
    - `uv venv --python 3.11` to create virtual environment
    - `source .venv/bin/activate` to activate it
    - `uv pip sync requirements.txt` to install dependencies
- Updated Docker image to use Python 3.11 and made other optimizations

## Performance evaluations

- `python inference_video.py --exp=1 --video=videos/demo.mp4`
    - Ubuntu baremetal: 4:32, 4.92it/s
    - Ubuntu Docker: 4:30, 4.95it/s

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
