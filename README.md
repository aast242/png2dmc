# png2dmc
A python program to convert images (not just PNGs!) into DMC thread cross-stitch patterns. 
See the [latest release](https://github.com/aast242/png2dmc/releases) 
for the most recent, fully-tested version of png2dmc.

# Example program runs
Here's an example of how well the program works with various options on a simple picture of a cave salamander (_Eurycea lucifuga_).
The defualt operation of the program is in the far right column (k-means quantization and LAB matching).

![Cavesalamander_comp](https://github.com/aast242/png2dmc/assets/85575864/dd6fea2d-52ea-4573-87a1-a69bff5eea8d)

# An example pattern and key
Here's a pattern from the above example. This one is the 50 color one with default
parameters (bottom right in the example). In the key, the floss number is next to the marker, and
the number of stitches in that color is after the dividing pipe.

![example_pattern](https://github.com/aast242/png2dmc/assets/85575864/3a35a6c3-a410-421b-8b54-d8a07e764ab9)


# Dependencies
- [Pillow (>= 9.2)](https://pillow.readthedocs.io/en/stable/installation.html)
- [numpy (>= 1.20.0)](https://numpy.org/install/)
- [scikit-learn (>= 1.2.2)](https://scikit-learn.org/stable/install.html)
- [scikit-image (>= 0.21.0)](https://scikit-image.org/docs/stable/user_guide/install.html)

All the dependencies are used by png2dmc to process images or do image-related math.
Detailed instructions on how to install them can be found linked above,
however they can be very simply installed using pip: 

```
pip3 install --upgrade Pillow
pip3 install --upgrade numpy
pip3 install --upgrade scikit-learn
pip3 install --upgrade scikit-image
```

# Usage
If you want png2dmc to be available anywhere, simply add the following line to your `~/.bashrc` and re-source it

`export PATH=$PATH:"/path/to/install/dir"`

png2dmc is run via the script 'png2dmc', found in the base installation directory. Usage information is as follows:
```
usage: png2dmc <image_png> <shortest_side> <color_reduce> [options]

Program: png2dmc
Version: 1.1

positional arguments:
  image_png             The image to be converted into a cross-stitch pattern
  shortest_side         The shortest side of the scaled image in pixels. (5 to any integer, -1 for no rescaling)
  color_reduce          The target number of colors in the final pattern (3 to any integer, -1 for no reduction).

optional arguments:
  -h, --help            show this help message and exit
  --dmc_idx <file_path>
                        A list of all the DMC thread colors and their RGB values
  --marker_file <file_path>
                        A file containing pixelated markers for making the pattern guide
  --numbers_file <file_path>
                        A file containing pixelated numbers for making the pattern key
  --key_sort [floss, count]
                        Changes how the key is sorted (either by floss number or stitch count)
  --ignore_size_limit   bypasses the size limit and forces the program to process large images
  --no_reuse            Disables the program reusing markers and terminates the program if there are more colors than markers
  --random_seed         Instead of using the file name as a seed, use a random seed
  --random_quantize     uses the old method to quantize colors (not K-means)
  --euc_dmc             Matches pixels in the image using the shortest euclidean distance in RGB space.
  --force               allows the program to overwrite files
```

The simplest usage of the program is  `png2dmc image.png -1 -1`, though some advanced options are available
and described below.

The program will output six files in the directory containing the source image:
1. \<imagename\>_DMC.png
   - a scaled PNG of the original image with colors that match DMC thread colors
2. \<imagename\>_DMC_colors.txt
   - a tab-delimited list of all the colors, their names, and how many stitches there are. Useful if you need to compare
  to a spreadsheet or something of that ilk
3. \<imagename\>_DMC_key_count.png
   - a key for the pattern with markers overlaid. This key is sorted by stitch count. 
     The marker and floss number are on the left, while the stitch count is on the right
4. \<imagename\>_DMC_key_floss.png
   - a key for the pattern with markers overlaid. This key is sorted by floss number. 
     The marker and floss number are on the left, while the stitch count is on the right
5. \<imagename\>_DMC_large.png
   - a scaled PNG of the original image with colors that match DMC thread colors but each pixel is an 11x11 square
6. \<imagename\>_DMC_markers.png
   - a scaled PNG of the original image with colors that match DMC thread colors. Each pixel is an 11x11 square and
  a marker identifying the color has been placed within each pixel. Cross-reference with the key file!

**_The most important file is #6_**. It can be opened in an image manipulation software to keep track of your progress
while you're stitching. I put red dots over each square after I've stitched it! Because the program uses the file name
to seed the random number generator, runs with the same options will produce exactly the same output, with the colors
represented by the same markers.

### Rescaling and color reduction
png2dmc is extremely flexible in the types of images it can process, and has built-in scaling options for shrinking 
larger images. By default, the program will **_warn_** you that your image is a bit large if it contains more than
22,800 pixels (a 151x151 square). You can just ignore the warning and continue on if this is okay with you!
The program will **_refuse_** to process images without scaling them down if they contain more than 251,000 pixels
(a 501x501 square). This can be bypassed by using the `--ignore_size_limit` flag, but it's there for a reason! 
The program is relatively slow to process anything larger than a few hundred thousand pixels, and the resulting 
patterns are infeasible to actually stitch. As such, **_I do not recommend using the program to make patterns for 
anything larger than a 500x500 square_**, but you have the freedom to do whatever you want!

Image rescaling is controlled by the first entered integer. If the value is -1, the image is not rescaled. 
Values larger than the current shortest side of the image result in the image being scaled up. 
**Values around ~200-150 are usually the most useful for typical images**. 
You can play around with cropping and removing the background from images before feeding them to the program to get 
your desired look.

Color reduction is controlled by the second integer. If the value is -1, all colors are kept. The number of colors in 
the final pattern will likely be lower than this number! The photo is first color-reduced, and then those colors are
matched to their closest DMC counterpart. This often results in some of the colors being collapsed together. I usually
use around **25-30**, but using higher numbers (e.g., 150 or -1) will result in a more complex (and usually better
looking) piece.

You'll likely have to play around with these two values to get a pattern you're satisfied with. Even if you want to 
modify the pattern that the program outputs, it gives you a nice starting point for more complex projects!

### Changing the DMC index, marker file, and numbers file
The flags `--dmc_idx <str>`, `--marker_file <str>`, and `--numbers_file <str>` allow the user to customize the
DMC thread library, the markers used to denote squares in the generated pattern, and the numbers/letters used 
in the generated pattern key, respectively. Realistically, you're only ever going to use the `--dmc_idx` flag,
as it can allow you to restrict the program to your thread collection by removing colors you don't have from 
the index. The index is present at `utils/DMC_index.txt` (or an older, worse version at `utils/old_DMC_index.txt`),
and can easily be copied and edited in Excel. 

*Make sure you keep the column names the same!*

If you ever want to change the markers, you can copy and edit the `utils/marker_symbols_noborder.png` file. Markers
are 11x11 squares, and **ONLY** the top left square should remain completely transparent. I like the three-wide format
because it means that you can easily add markers without having to make an absurd number of new ones.

The `--numbers_file` flag is really only there for testing purposes. If you add any letters/numbers, you'd have to
edit the dictionary at the beginning. Because of this, **_I do not recommend using this flag_**.

### Changing how the key is sorted
By default, the pattern key is sorted in ascending order by the floss number. This can be changed using the flag 
`--key_sort count` to sort the pattern key by the number of stitches you'll make with each color.

### Other miscellaneous flags
The `--no_reuse` flag stops the program from reusing markers when it's making a pattern. By default, the program has
74 unique markers that it can use to identify colors. This can be expanded by the user if they desire 
(see "_Changing the DMC index, marker file, and numbers file_"), but colors are unable to be uniquely identified if
there are more than 74. I've solved this by simply reusing markers. If you aren't okay with having redundant markers,
use this flag and the program will terminate if the pattern needs to contain redundant markers. This can be solved by
reducing the color palette further, or by just using redundant markers. It doesn't bother me, but I thought some
people might want to have the option!

The `--random_seed` flag seeds the random number generator with a random seed instead of using the file name as a seed. 
This is useful if you don't like the markers that are being chosen and want to switch it up. This will make the output
of the program impossible to reproduce!

The `--random_quantize` flag makes the program use the older version of color quantization 
([using the pillow 'convert' function](https://pillow.readthedocs.io/en/stable/reference/Image.html#PIL.Image.Image.convert)).
The program now uses a K-means method of color quantization,
as detailed [here](https://scikit-learn.org/stable/auto_examples/cluster/plot_color_quantization.html). 
The K-means method also converts images into LAB color space, heavily inspired by swirlyclouds' 
[EmbroideryPatternScript repo](https://github.com/swirlyclouds/EmbroideryPatternScript).

The `--euc_dmc` flag makes the program match colored pixels to DMC thread colors using the euclidean distance between
the colors in RGB space. By default, the program matches colors in the more perceptually uniform LAB space using the
shortest distance from the Color Measurement Committee of the Society of Dyers and Colourists
(more info [here](https://scikit-image.org/docs/stable/api/skimage.color.html#skimage.color.deltaE_cmc)). Using 
this option is here for legacy purposes. The LAB conversion takes longer 
(for a 109x109 image with 500 colors: LAB is ~90 seconds, RGB is ~70 seconds), but it's not an absurd amount of time,
and I think the patterns are improved. If you want, you can create a pattern with and without this flag and see
which one you prefer!

The `--force` flag simply allows the program to overwrite files. I typically use it all the time, but it's there to 
make sure users don't accidentally overwrite projects.
