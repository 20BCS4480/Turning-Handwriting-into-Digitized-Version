# Turning-Handwriting-into-Digitized-Version
Table of Contents
Tesseract OCR
About
Brief history
Installing Tesseract
Running Tesseract
For developers
Support
License
Dependencies
Latest Version of README
About
This package contains an OCR engine - libtesseract and a command line program - tesseract.

Handwritten Text Recognition with TensorFlow
Update 2023/2: a web demo is available
Update 2023/1: see HTRPipeline for a package to read full pages
Update 2021/2: recognize text on line level (multiple words)
Update 2021/1: more robust model, faster dataloader, word beam search decoder also available for Windows
Update 2020: code is compatible with TF2
Handwritten Text Recognition (HTR) system implemented with TensorFlow (TF) and trained on the IAM off-line HTR dataset. The model takes images of single words or text lines (multiple words) as input and outputs the recognized text. 3/4 of the words from the validation-set are correctly recognized, and the character error rate is around 10%.

htr

Run demo
Download one of the pretrained models
Model trained on word images: only handles single words per image, but gives better results on the IAM word dataset
Model trained on text line images: can handle multiple words in one image
Put the contents of the downloaded zip-file into the model directory of the repository
Go to the src directory
Run inference code:
Execute python main.py to run the model on an image of a word
Execute python main.py --img_file ../data/line.png to run the model on an image of a text line
The input images, and the expected outputs are shown below when the text line model is used.

test

> python main.py
Init with stored values from ../model/snapshot-13
Recognized: "word"
Probability: 0.9806370139122009
test

> python main.py --img_file ../data/line.png
Init with stored values from ../model/snapshot-13
Recognized: "or work on line level"
Probability: 0.6674373149871826
Command line arguments
--mode: select between "train", "validate" and "infer". Defaults to "infer".
--decoder: select from CTC decoders "bestpath", "beamsearch" and "wordbeamsearch". Defaults to "bestpath". For option "wordbeamsearch" see details below.
--batch_size: batch size.
--data_dir: directory containing IAM dataset (with subdirectories img and gt).
--fast: use LMDB to load images faster.
--line_mode: train reading text lines instead of single words.
--img_file: image that is used for inference.
--dump: dumps the output of the NN to CSV file(s) saved in the dump folder. Can be used as input for the CTCDecoder.
Integrate word beam search decoding
The word beam search decoder can be used instead of the two decoders shipped with TF. Words are constrained to those contained in a dictionary, but arbitrary non-word character strings (numbers, punctuation marks) can still be recognized. The following illustration shows a sample for which word beam search is able to recognize the correct text, while the other decoders fail.

decoder_comparison

Follow these instructions to integrate word beam search decoding:

Clone repository CTCWordBeamSearch
Compile and install by running pip install . at the root level of the CTCWordBeamSearch repository
Specify the command line option --decoder wordbeamsearch when executing main.py to actually use the decoder
The dictionary is automatically created in training and validation mode by using all words contained in the IAM dataset (i.e. also including words from validation set) and is saved into the file data/corpus.txt. Further, the manually created list of word-characters can be found in the file model/wordCharList.txt. Beam width is set to 50 to conform with the beam width of vanilla beam search decoding.

Train model on IAM dataset
Prepare dataset
Follow these instructions to get the IAM dataset:

Register for free at this website
Download words/words.tgz
Download ascii/words.txt
Create a directory for the dataset on your disk, and create two subdirectories: img and gt
Put words.txt into the gt directory
Put the content (directories a01, a02, ...) of words.tgz into the img directory
Run training
Delete files from model directory if you want to train from scratch
Go to the src directory and execute python main.py --mode train --data_dir path/to/IAM
The IAM dataset is split into 95% training data and 5% validation data
If the option --line_mode is specified, the model is trained on text line images created by combining multiple word images into one
Training stops after a fixed number of epochs without improvement
The pretrained word model was trained with this command on a GTX 1050 Ti:

python main.py --mode train --fast --data_dir path/to/iam  --batch_size 500 --early_stopping 15
And the line model with:

python main.py --mode train --fast --data_dir path/to/iam  --batch_size 250 --early_stopping 10
Fast image loading
Loading and decoding the png image files from the disk is the bottleneck even when using only a small GPU. The database LMDB is used to speed up image loading:

Go to the src directory and run create_lmdb.py --data_dir path/to/iam with the IAM data directory specified
A subfolder lmdb is created in the IAM data directory containing the LMDB files
When training the model, add the command line option --fast
The dataset should be located on an SSD drive. Using the --fast option and a GTX 1050 Ti training on single words takes around 3h with a batch size of 500. Training on text lines takes a bit longer.

Information about model
The model is a stripped-down version of the HTR system I implemented for my thesis. What remains is the bare minimum to recognize text with an acceptable accuracy. It consists of 5 CNN layers, 2 RNN (LSTM) layers and the CTC loss and decoding layer. For more details see this Medium article.

References

Tesseract 4 adds a new neural net (LSTM) based OCR engine which is focused on line recognition, but also still supports the legacy Tesseract OCR engine of Tesseract 3 which works by recognizing character patterns. Compatibility with Tesseract 3 is enabled by using the Legacy OCR Engine mode (--oem 0). It also needs traineddata files which support the legacy engine, for example those from the tessdata repository.

Stefan Weil is the current lead developer. Ray Smith was the lead developer until 2018. The maintainer is Zdenko Podobny. For a list of contributors see AUTHORS and GitHub's log of contributors.

Tesseract has unicode (UTF-8) support, and can recognize more than 100 languages "out of the box".

Tesseract supports various image formats including PNG, JPEG and TIFF.

Tesseract supports various output formats: plain text, hOCR (HTML), PDF, invisible-text-only PDF, TSV and ALTO (the last one - since version 4.1.0).

You should note that in many cases, in order to get better OCR results, you'll need to improve the quality of the image you are giving Tesseract.

This project does not include a GUI application. If you need one, please see the 3rdParty documentation.

Tesseract can be trained to recognize other languages. See Tesseract Training for more information.

Brief history
Tesseract was originally developed at Hewlett-Packard Laboratories Bristol UK and at Hewlett-Packard Co, Greeley Colorado USA between 1985 and 1994, with some more changes made in 1996 to port to Windows, and some C++izing in 1998. In 2005 Tesseract was open sourced by HP. From 2006 until November 2018 it was developed by Google.

Major version 5 is the current stable version and started with release 5.0.0 on November 30, 2021. Newer minor versions and bugfix versions are available from GitHub.

Latest source code is available from main branch on GitHub. Open issues can be found in issue tracker, and planning documentation.

See Release Notes and Change Log for more details of the releases.

Installing Tesseract
You can either Install Tesseract via pre-built binary package or build it from source.

A C++ compiler with good C++17 support is required for building Tesseract from source.

Running Tesseract
Basic command line usage:

tesseract imagename outputbase [-l lang] [--oem ocrenginemode] [--psm pagesegmode] [configfiles...]
For more information about the various command line options use tesseract --help or man tesseract.

Examples can be found in the documentation.

For developers
Developers can use libtesseract C or C++ API to build their own application. If you need bindings to libtesseract for other programming languages, please see the wrapper section in the AddOns documentation.

Documentation of Tesseract generated from source code by doxygen can be found on tesseract-ocr.github.io.

Support
Before you submit an issue, please review the guidelines for this repository.

For support, first read the documentation, particularly the FAQ to see if your problem is addressed there. If not, search the Tesseract user forum, the Tesseract developer forum and past issues, and if you still can't find what you need, ask for support in the mailing-lists.

Mailing-lists:

tesseract-ocr - For tesseract users.
tesseract-dev - For tesseract developers.
Please report an issue only for a bug, not for asking questions.

License
The code in this repository is licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
NOTE: This software depends on other packages that may be licensed under different open source licenses.

Tesseract uses Leptonica library which essentially uses a BSD 2-clause license.

Dependencies
Tesseract uses Leptonica library for opening input images (e.g. not documents like pdf). It is suggested to use leptonica with built-in support for zlib, png and tiff (for multipage tiff).

