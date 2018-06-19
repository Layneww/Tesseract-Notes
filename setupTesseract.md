# Build tesseract from source
### install required library
```
# install tesseract 4.x on Ubuntu 18.xx
apt install tesseract-ocr
# Developer Tools
apt install libtesseract-dev
```
if such error message appears,
```
dpkg: warning: 'ldconfig' not found in PATH or not executable.
dpkg: warning: 'start-stop-daemon' not found in PATH or not executable.
dpkg: error: 2 expected programs not found in PATH or not executable.
Note: root's PATH should usually contain /usr/local/sbin, /usr/sbin and /sbin.
E: Sub-process /usr/bin/dpkg returned an error code (2)
```
run
```export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin```
```
sudo apt-get install g++ # or clang++ (presumably)
sudo apt-get install autoconf automake libtool
sudo apt-get install autoconf-archive
sudo apt-get install pkg-config
sudo apt-get install libpng-dev
sudo apt-get install libjpeg8-dev
sudo apt-get install libtiff5-dev
sudo apt-get install zlib1g-dev

# for training tools
sudo apt-get install libicu-dev
sudo apt-get install libpango1.0-dev
sudo apt-get install libcairo2-dev
```


### build leptonica 1.76.0 from source
```
wget http://www.leptonica.org/source/leptonica-1.76.0.tar.gz
gunzip leptonica-1.76.0.tar.gz
tar -xvf leptonica-1.76.0.tar
cd leptonica-1.76.0
./configure
make
sudo make install
```
It is sufficient to just use 1.74 version.
### build tesseract 4.0 alpha from source
```
# library needed
apt-get install autoconf-archive automake g++ libtool libleptonica-dev make pkg-config
apt-get install asciidoc
apt-get install libpango1.0-dev

# clone the 4.00.00alpha branch
git clone https://github.com/tesseract-ocr/tesseract.git --branch 4.00.00alpha --single-branch tesseract-ocr
# clone the master branch (beta)
git clone https://github.com/tesseract-ocr/tesseract.git tesseract-ocr

cd tesseract
./autogen.sh
autoreconf -i

## version 1
./configure --enable-debug \
    --with-extra-libraries=/usr/local/lib \
    --with-extra-includes=/usr/local/include \
    LDFLAGS=-L/usr/local/lib \
    CPPFLAGS=-I/usr/local/include
make
## version 1 ends
## version 2
./configure
LDFLAGS="-L/usr/local/lib" CFLAGS="-I/usr/local/include" make
## version 2 ends
sudo make install
sudo make install -langs # install languages
sudo ldconfig

# build training tool
make training
sudo make training-install
# optional:
apt-get install default-jdk
make ScrollView.jar
export SCROLLVIEW_PATH=$PWD/java
```
### create training data

#### tesstrain (one command)
```
## for LSTM, enable --linedata_only
training/tesstrain.sh --fonts_dir /usr/share/fonts \
  --lang eng \
  --noextract_font_properties --langdata_dir /data/langdata \
  --output_dir /data/tesstutorial/engtrain
```

### langdata
It contains
```
desired_characters
eng.numbers
eng.punc
eng.training_text
eng.training_text.bigram_freqs
eng.training_text.unigram_freqs
eng.unicharambigs
eng.word.bigrams
eng.wordlist
```

#### step by step
To find available fonts in a specific fonts_dir
```
training/text2image --list_available_fonts --fonts_dir <the desired directory, e.g. /usr/share/fonts>
```
To generate the data pair, run:
```
training/text2image --fonts_dir=/usr/share/fonts --text=training_text.txt --outputbase=[lang].[fontname].exp0 --font='Font Name' 
```
```--output_individual_glyph_images```: If true also outputs individual character images  (type:bool default:false)

To save both the box/tiff and glphs data,
```
training/text2image --text=/data/langdata/eng/eng.training_text --outputbase=/data/text2image/eng/eng.Lato.exp0 --font='Liberation Sans' --output_individual_glyph_images=true --fonts_dir=/usr/share/fonts --glyph_resized_size=300
```

To list all fonts in your system which can render the training text, run:
```
training/text2image --text=training_text.txt --outputbase=eng --fonts_dir=/usr/share/fonts  --find_fonts --min_coverage=1.0 --render_per_font=false
```

Help list
```
  --exposure  Exposure level in photocopier  (type:int default:0)
  --resolution  Pixels per inch  (type:int default:300)
  --xsize  Width of output image  (type:int default:3600)
  --ysize  Height of output image  (type:int default:4800)
  --max_pages  Maximum number of pages to output (0=unlimited)  (type:int default:0)
  --margin  Margin round edges of image  (type:int default:100)
  --ptsize  Size of printed text  (type:int default:12)
  --leading  Inter-line space (in pixels)  (type:int default:12)
  --box_padding  Padding around produced bounding boxes  (type:int default:0)
  --glyph_resized_size  Each glyph is square with this side length in pixels  (type:int default:0)
  --glyph_num_border_pixels_to_pad  Final_size=glyph_resized_size+2*glyph_num_border_pixels_to_pad  (type:int default:0)
  --tlog_level  Minimum logging level for tlog() output  (type:int default:0)
  --char_spacing  Inter-character space in ems  (type:double default:0)
  --underline_start_prob  Fraction of words to underline (value in [0,1])  (type:double default:0)
  --underline_continuation_prob  Fraction of words to underline (value in [0,1])  (type:double default:0)
  --min_coverage  If find_fonts==true, the minimum coverage the font has of the characters in the text file to include it, between 0 and 1.  (type:double default:1)
  --degrade_image  Degrade rendered image with speckle noise, dilation/erosion and rotation  (type:bool default:true)
  --rotate_image  Rotate the image in a random way.  (type:bool default:true)
  --strip_unrenderable_words  Remove unrenderable words from source text  (type:bool default:true)
  --ligatures  Rebuild and render ligatures  (type:bool default:false)
  --find_fonts  Search for all fonts that can render the text  (type:bool default:false)
  --render_per_font  If find_fonts==true, render each font to its own image. Image filenames are of the form output_name.font_name.tif  (type:bool default:true)
  --list_available_fonts  List available fonts and quit.  (type:bool default:false)
  --render_ngrams  Put each space-separated entity from the input file into one bounding box. The ngrams in the input file will be randomly permuted before rendering (so that there is sufficient variety of characters on each line).  (type:bool default:false)
  --output_word_boxes  Output word bounding boxes instead of character boxes. This is used for Cube training, and implied by --render_ngrams.  (type:bool default:false)
  --bidirectional_rotation  Rotate the generated characters both ways.  (type:bool default:false)
  --only_extract_font_properties  Assumes that the input file contains a list of ngrams. Renders each ngram, extracts spacing properties and records them in output_base/[font_name].fontinfo file.  (type:bool default:false)
  --output_individual_glyph_images  If true also outputs individual character images  (type:bool default:false)
  --text  File name of text input to process  (type:string default:)
  --outputbase  Basename for output image/box file  (type:string default:)
  --writing_mode  Specify one of the following writing modes.
'horizontal' : Render regular horizontal text. (default)
'vertical' : Render vertical text. Glyph orientation is selected by Pango.
'vertical-upright' : Render vertical text. Glyph  orientation is set to be upright.  (type:string default:horizontal)
  --font  Font description name to use  (type:string default:Arial)
  --unicharset_file  File with characters in the unicharset. If --render_ngrams is true and --unicharset_file is specified, ngrams with characters that are not in unicharset will be omitted  (type:string default:)
  --fontconfig_tmpdir  Overrides fontconfig default temporary dir  (type:string default:/tmp)
  --fonts_dir  If empty it use system default. Otherwise it overrides system default font location  (type:string default:)
```



#### training from scratch
```
mkdir -p ~/tesstutorial/engoutput
training/lstmtraining --debug_interval 100 \
  --traineddata ~/tesstutorial/engtrain/eng/eng.traineddata \
  --net_spec '[1,36,0,1 Ct3,3,16 Mp3,3 Lfys48 Lfx96 Lrx96 Lfx256 O1c111]' \
  --model_output ~/tesstutorial/engoutput/base --learning_rate 20e-4 \
  --train_listfile ~/tesstutorial/engtrain/eng.training_files.txt \
  --eval_listfile ~/tesstutorial/engeval/eng.training_files.txt \
  --max_iterations 5000 &>~/tesstutorial/engoutput/basetrain.log
```

