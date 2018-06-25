### Overview of training process
- Prepare text file
- Make tiff/box pairs
- Make them to be .lstmf
- Send to model

### Input and Output of the model
#### Input
- Prepare:
  - text: create lstmf
  - unicharset, recoder
- Step: 

### Key functions and the purpose
#### tesstrain.sh
Note: the standard version can only take one text and one font
```
# standard version
training/tesstrain.sh --fonts_dir /usr/share/fonts --lang tha+eng --linedata_only \
  --noextract_font_properties --langdata_dir /data/dataset/ \
  --tessdata_dir ./tessdata --output_dir ~/tesstutorial/tha+eng_train

# edited version
training/tesstrain.sh --fonts_dir /c/windows/fonts/ --lang tha+eng --linedata_only \
  --noextract_font_properties  --langdata_dir train/langdata/ \
  --tessdata_dir ./tessdata --output_dir ~/i351756/tesstutorial/tha+eng_train --fontlist "Meiryo UI" "Meiryo UI Bold" \
  --exposures "0" --textlist "train/tex1.txt" "train/tex2.txt"
```
By enable ```--linedata_only```, it will prepare LSTM training data.

Inside tesstrain,
```
source "$(dirname $0)/language-specific.sh"
set_lang_specific_parameters ${LANG_CODE} # it predifines params for a lang, 
                                           # default mix_lang=eng
initialize_fontconfig # create a temp dir for font_config, store result of text2image in also

phase_I_generate_image 8 # generate the tiff/box pair using the fontconfig given above
phase_UP_generate_unicharset # generate unicharset and xheight from box file
if ((LINEDATA)); then
  phase_E_extract_features "lstm.train" 8 "lstmf" # create lstmf from box and tiff file
  make__lstmdata # call combine_lang_model
```

