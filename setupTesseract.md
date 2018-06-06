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

