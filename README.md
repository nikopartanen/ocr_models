# Introduction

I attempt to explain here how the compilation process has been done for these files. I've tried to document every command I've done, so that the results could be reproduced. This is just initial test, and more sophisticated versions will come soon. I haven't tested it yet with dictionary files etc. The result is indeed quite bad at the moment, but will certainly improve in the future.

    sudo apt-get install autoconf automake libtool
    sudo apt-get install libpng12-dev
    sudo apt-get install libjpeg62-dev
    sudo apt-get install libtiff4-dev
    sudo apt-get install zlib1g-dev

These are meant for the training tools, but of course we want them too.

    sudo apt-get install libicu-dev
    sudo apt-get install libpango1.0-dev
    sudo apt-get install libcairo2-dev

    mkdir ocr
    cd ocr

    wget http://www.leptonica.com/source/leptonica-1.72.tar.gz
    tar xzf

    cd leptonica-1.72
    ./configure
    make
    sudo make install
    cd ..

    wget https://github.com/tesseract-ocr/tesseract/archive/3.04.00.tar.gz
    tar xzf 3.04.00.tar.gz

    cd tesseract-3.04.00
    ./autogen.sh
    ./configure
    make
    sudo make install
    make training
    sudo make training-install
    sudo ldconfig

    cd ..

    wget https://tesseract-ocr.googlecode.com/files/eng.traineddata.gz
    gunzip eng.traineddata.gz
    sudo mv -v eng.traineddata /usr/local/share/tessdata/
    
    mkdir model
    echo "This is a test model." > ./model/model.txt

Let's cut a piece from the text file we want to use.

    cat kpv_izva_evv_orig.txt | head -n 3 > kpv_izva_evv.txt
    
That is this much text:

    1. as olem jylyś 

    1. me rød'iććyli požńa d'erevńayn kod uht'inskej rajonyn.
    d'erevńays sulale iźva ju bokyn, di vylyn.
    tulysnas gøger bośte va, da sijen i šueny olyśjasse “di vasa køćjasen”.
    d'erevńaas ńel'amynyś unǯyk kerka.
    jøzys stavys eni uǯaleny sovhozyn.
    sovhozys lois vo sajyn kolhoz mestae.
    sovhozas pe ōnyse burǯyk, taʒ́ šueny stavys.
    d'erevńaas em naćal'nej škola.
    sred'ńej školays sosnogorskyn, kodi vit kilametra sajyn.
    a das kilametra sajyn em uśt'-uhta śelo, seni em siʒ́ že sred'ńej škola køn veledeny komi ka vylyn.
    sosnogorskas i uśt'uhtaas vyjymeś ińt'ernatjas køn besplatne verdeny mijan d'erevńasa veleććyśjasse.

    export TESSDATA_PREFIX="/usr/local/share/tessdata/"

Now we can run:

    text2image --text=./ocr_models/kpv_izva_evv.txt --outputbase=./ocr_models/evv.UbuntuLightItalic.exp0 --font='Ubuntu Light Italic' --fonts_dir=/usr/share/fonts/

This creates the first files we need for the model into ´./model´ directory.

    cd model
    tesseract evv.UbuntuLightItalic.exp0.tif evv.UbuntuLightItalic.exp0 batch.nochop makebox
    
Now we can start to edit this boxfile. There are some editors one can use to do that. This works quite well:

    http://sourceforge.net/projects/vietocr/files/jTessBoxEditor/

To get it work we need to install Java Runtime Environment:

    sudo apt-get install default-jre
   
It opens with:

    java -jar jTessBoxEditor.jar

Actually, the file we got looks very very nice! I do not spot any mistakes. I do not really know whether I should remove the space characters, but maybe they are relevant, so better not.

Forward. This trains the model from the boxfile.

    tesseract evv.UbuntuLightItalic.exp0.tif evv.UbuntuLightItalic.exp0 box.train

Seems to work...  It gave some failure, but did generate the .tr file.

### Extracting the character set

    unicharset_extractor evv.UbuntuLightItalic.exp0.box
    
    echo "ubuntulightitalic 1 0 0 0 0 0" > font_properties
    
    shapeclustering -F font_properties -U unicharset evv.UbuntuLightItalic.exp0.tr
    
This gives us a new file called "shapetable".
    
    mftraining -F font_properties -U unicharset -O evv.unicharset evv.UbuntuLightItalic.exp0.tr
    
    tesseract evv.UbuntuLightItalic.exp0.tif evv.UbuntuLightItalic.exp0 -l evv batch.nochop makebox

    cntraining evv.UbuntuLightItalic.exp0.tr

This creates a file called normproto.

Now many of these files have to renamed into evv.normproto etc.

### Dictionary data

### Combining

    combine_tessdata evv.

This can be now moved into folder /usr/local/share/tessdata.

After this we can do:

    tesseract test.tiff test -l evv

And it works!
