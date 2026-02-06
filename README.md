<p align="center">
<a href="https://en.wikipedia.org/wiki/Bayer_filter"><img width="500" src="https://raw.githubusercontent.com/letmaik/rawpy/main/logo/logo.png"></a>
</p>

# rawpy-demosaic

**This is a GPL3-licensed fork of [rawpy](https://github.com/letmaik/rawpy) that includes GPL2 and GPL3 demosaic packs by default.**

rawpy-demosaic is an easy-to-use Python wrapper for the [LibRaw library][libraw]
with GPL demosaic algorithms (AMaZe, VCD, Modified AHD, LMMSE, and more) enabled out of the box.
It also contains some extra functionality for finding and repairing hot/dead pixels.

> **License Notice**: This fork is licensed under GPL-3.0-or-later due to inclusion of
> GPL-licensed demosaic algorithms. It cannot be used in proprietary software.
> See [LICENSE](LICENSE) for details.

[API Documentation](https://letmaik.github.io/rawpy/api/)

[Jupyter notebook tutorials](https://github.com/letmaik/rawpy-notebooks/blob/master/README.md)

## Features over upstream rawpy

- AMaZe demosaic algorithm (GPL3)
- VCD, Modified AHD, LMMSE demosaic algorithms (GPL2)
- All GPL demosaic features enabled by default in all wheels

## Sample code

Load a RAW file and save the postprocessed image using default [parameters](https://letmaik.github.io/rawpy/api/rawpy.Params.html):

```python
import rawpy
import imageio.v3 as iio

path = 'image.nef'
with rawpy.imread(path) as raw:
    rgb = raw.postprocess()
iio.imwrite('default.tiff', rgb)
```

Save as 16-bit linear image:

```python
with rawpy.imread(path) as raw:
    rgb = raw.postprocess(gamma=(1,1), no_auto_bright=True, output_bps=16)
iio.imwrite('linear.tiff', rgb)
```

Use GPL demosaic algorithms:

```python
from rawpy import DemosaicAlgorithm

with rawpy.imread(path) as raw:
    # AMaZe algorithm (GPL3)
    rgb = raw.postprocess(demosaic_algorithm=DemosaicAlgorithm.AMAZE)

with rawpy.imread(path) as raw:
    # VCD algorithm (GPL2)
    rgb = raw.postprocess(demosaic_algorithm=DemosaicAlgorithm.VCD)

with rawpy.imread(path) as raw:
    # LMMSE algorithm (GPL2)
    rgb = raw.postprocess(demosaic_algorithm=DemosaicAlgorithm.LMMSE)
```

Extract embedded thumbnail/preview image and save as JPEG:

```python
with rawpy.imread(path) as raw:
    # raises rawpy.LibRawNoThumbnailError if thumbnail missing
    # raises rawpy.LibRawUnsupportedThumbnailError if unsupported format
    thumb = raw.extract_thumb()
if thumb.format == rawpy.ThumbFormat.JPEG:
    # thumb.data is already in JPEG format, save as-is
    with open('thumb.jpeg', 'wb') as f:
        f.write(thumb.data)
elif thumb.format == rawpy.ThumbFormat.BITMAP:
    # thumb.data is an RGB numpy array, convert with imageio
    iio.imwrite('thumb.jpeg', thumb.data)
```

Find bad pixels using multiple RAW files and repair them:

```python
import rawpy.enhance

paths = ['image1.nef', 'image2.nef', 'image3.nef']
bad_pixels = rawpy.enhance.find_bad_pixels(paths)

for path in paths:
    with rawpy.imread(path) as raw:
        rawpy.enhance.repair_bad_pixels(raw, bad_pixels, method='median')
        rgb = raw.postprocess()
    iio.imwrite(path + '.tiff', rgb)
```

## Installation

Install rawpy-demosaic by running:
```sh
pip install rawpy-demosaic
```

64-bit binary wheels are provided for Linux, macOS, and Windows.

### Stable vs. pre-release

All stable rawpy-demosaic releases are always built against a stable LibRaw library release.
You can output the LibRaw version with `print(rawpy.libraw_version)`.

rawpy-demosaic pre-releases have version numbers like `0.15.0a1` and are built against
a recent LibRaw snapshot. To install a pre-release, run:
```sh
pip install --pre rawpy-demosaic
```

### Optional features

The underlying [LibRaw library][libraw] supports several optional features.
The following table shows which PyPI binary wheels support which features.

| Feature            | Windows | macOS | Linux |
| ------------------ | ------- | ----- | ----- |
| LCMS color engine  | yes     | yes   | yes   |
| RedCine codec      | yes     | yes   | yes   |
| DNG deflate codec  | yes     | yes   | yes   |
| DNG lossy codec    | yes     | yes   | yes   |
| Demosaic Pack GPL2 | yes     | yes   | yes   |
| Demosaic Pack GPL3 | yes     | yes   | yes   |
| OpenMP             | yes     | no    | yes   |

Tip: You can dynamically query supported features by inspecting the `rawpy.flags` dictionary.

### Installation from source on Linux/macOS

For macOS, LibRaw is built as part of the rawpy-demosaic build (see external/).
For Linux, you need to install the LibRaw library on your system.

On Ubuntu, you can get (an outdated) version with:

```sh
sudo apt-get install libraw-dev
```

Or install the latest release version from the source repository:

```sh
git clone https://github.com/LibRaw/LibRaw.git libraw
git clone https://github.com/LibRaw/LibRaw-cmake.git libraw-cmake
cd libraw
git checkout 0.20.0
cp -R ../libraw-cmake/* .
cmake .
sudo make install
```

After that, install rawpy-demosaic using:

```sh
git clone https://github.com/exfab/rawpy-demosaic
cd rawpy-demosaic
pip install numpy cython
pip install .
```

On Linux, if you get the error "ImportError: libraw.so: cannot open shared object file: No such file or directory"
when trying to use rawpy, then do the following:

```sh
echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/99local.conf
sudo ldconfig
```

The LibRaw library is installed in /usr/local/lib (if installed manually) and apparently this folder is not searched
for libraries by default in some Linux distributions.

### Installation from source on Windows

These instructions are experimental and support is not provided for them.
Typically, there should be no need to build manually since wheels are hosted on PyPI.

You need to have Visual Studio installed to build rawpy-demosaic.

In a PowerShell window:
```sh
$env:USE_CONDA = '1'
$env:PYTHON_VERSION = '3.7'
$env:PYTHON_ARCH = '64'
$env:NUMPY_VERSION = '1.14.*'
git clone https://github.com/exfab/rawpy-demosaic
cd rawpy-demosaic
.github/scripts/build-windows.ps1
```
The above will download all build dependencies (including a Python installation)
and is fully configured through the four environment variables.
Set `USE_CONDA = '0'` to build within an existing Python environment.

## FAQ

### I'm getting deadlocks when using multiprocessing on Linux

If you're experiencing deadlocks when using rawpy with Python's `multiprocessing` module on Linux, this is caused by an interaction between OpenMP (which is enabled in the Linux wheels) and the default `fork` start method used by multiprocessing.

**The Problem:**
When a process using OpenMP is forked, OpenMP's internal thread pool state becomes inconsistent in the child process, which can cause deadlocks on subsequent calls to rawpy functions.

**The Solution:**
Use the `spawn` or `forkserver` start method instead of `fork`:

```python
import multiprocessing as mp
import rawpy

def process_raw(filename):
    with rawpy.imread(filename) as raw:
        rgb = raw.postprocess()
    return rgb

if __name__ == '__main__':
    # Set the start method to 'spawn' before creating any processes
    mp.set_start_method('spawn')

    with mp.Pool(processes=4) as pool:
        results = pool.map(process_raw, ['image1.nef', 'image2.nef'])
```

**Note:** The start method can only be set once per program, and must be called within an `if __name__ == '__main__':` guard. The `spawn` method creates a fresh Python interpreter process, avoiding the OpenMP thread state issue entirely.

For more information, see:
- [Python multiprocessing documentation on start methods](https://docs.python.org/3/library/multiprocessing.html#contexts-and-start-methods)
- [OpenMP and fork() interaction issues](https://github.com/isl-org/Open3D/wiki/Deadlock-with-multiprocessing-(using-fork)-and-OpenMP)

### I'm getting "LibRawFileUnsupportedError: Unsupported file format or not RAW file"

This error occurs when rawpy/LibRaw cannot recognize the file as a supported RAW image format. Common causes include:

1. **The file is not actually a RAW file** - Make sure you're trying to open a RAW image file (e.g., .NEF, .CR2, .ARW, .DNG, etc.) and not a regular image format like JPEG or PNG.

2. **The file is corrupted or incomplete** - If the file was not fully downloaded or is damaged, LibRaw cannot read it properly.

3. **The file lacks proper headers** - Some proprietary or headerless RAW formats are not supported by LibRaw. RAW files need to contain proper metadata headers that identify the camera model, sensor configuration, and other essential information for LibRaw to decode them.

4. **Unsupported camera or RAW format** - While LibRaw supports a [wide range of cameras](https://www.libraw.org/supported-cameras), some very new or obscure camera models may not be supported yet. Check the LibRaw website for the list of supported cameras.

**What you can do:**
- Verify the file is a genuine RAW file from a supported camera
- Try opening the file with the camera manufacturer's software to confirm it's valid
- Check if you're using the latest version of rawpy-demosaic, as newer versions may support additional cameras
- If you have a headerless or proprietary RAW format, you may need to convert it to a standard format like DNG using the camera manufacturer's tools first

## Upstream

This fork is based on [rawpy](https://github.com/letmaik/rawpy) by Maik Riechert.
To sync with upstream:

```bash
git remote add upstream https://github.com/letmaik/rawpy.git
git fetch upstream
git merge upstream/main
```

[libraw]: https://www.libraw.org
