# Sign Language Translator ⠎⠇⠞

[![python](https://img.shields.io/pypi/pyversions/sign-language-translator)](https://pypi.org/project/sign-language-translator/)
[![PyPi](https://img.shields.io/pypi/v/sign-language-translator)](https://pypi.org/project/sign-language-translator/)
[![Downloads](https://static.pepy.tech/personalized-badge/sign-language-translator?period=total&units=international_system&left_color=grey&right_color=brightgreen&left_text=Downloads)](https://pepy.tech/project/sign-language-translator)

1. [Overview](#overview)
   1. [Solution](#solution)
   2. [Major Components and Goals](#major-components-and-goals)
2. [How to install the package](#how-to-install-the-package)
3. [Usage](#usage)
   1. [Command Line](#command-line)
      <!-- 1. [configure](#configure) -->
      1. [download](#download)
      2. [translate](#translate)
   2. [Python](#python)
      1. [basic translation](#basic-translation)
      2. [text language processor](#text-language-processor)
      3. [sign language processor](#sign-language-processor)
      4. [language models](#language-models)
4. [Directory Tree](#directory-tree)
5. [Research Paper](#research-paper)
6. [Credits and Gratitude](#credits-and-gratitude)
7. [Bonus](#bonus)
   1. number of lines of code
   2. just for fun
   3. publish package on PyPI

## Overview

Sign language consists of gestures and expressions used mainly by the hearing-impaired to talk. This project is an effort to bridge the communication gap between the hearing and the hearing-impaired community using Artificial Intelligence.

The goal is to provide a user friendly API to novel Sign Language Translation solutions that can easily adapt to any regional sign language. Unlike most other projects, this python library can translate full sentences and not just the alphabet. This is the package that powers the [slt_ai website](https://github.com/mdsrqbl/slt_ai).

A bigger hurdle is the lack of datasets and frameworks that deep learning engineers and software developers can use to build useful products for the target community. This project aims to empower sign language translation by providing robust components, tools and models for both sign language to text and text to sign language conversion. It seeks to advance the development of sign language translators for various regions while providing a way to sign language standardization.

### Solution

We've have built an *extensible rule-based* text-to-sign translation system that can be used to *train Deep Learning* models for both sign to text & text to sign translation.

To create a rule-based translation system for your regional language, you can inherit the TextLanguage and SignLanguage classes and pass them as arguments to the ConcatenativeSynthesis class. Later you can use that system to fine-tune our AI models.

### Major Components and Goals

1. `Sign language to Text`

    - Extract pose vectors (2D or 3D) from videos and map them to corresponding text representations of the performed signs.
    - Fine-tuned a neural network, such as a state-of-the-art speech-to-text model, with gradual unfreezing starting from the input layers to convert pose vectors to text.

2. `Text to Sign Language`
    - This is a relatively easier task if you parse input text and play appropriate video clips for each word.

    1. Motion Transfer
         - Concatenate pose vectors in the time dimension and transfer the movements onto any given image of a person. This ensures smooth transitions between video clips.
    2. Sign Feature Synthesis
         - Condition a pose sequence generation model on a pre-trained text encoder (e.g., fine-tuned decoder of a multilingual T5) to output pose vectors instead of text tokens. This solves challenges related to unknown synonyms or hard-to-tokenize/process words or phrases.

3. `Preprocessing Utilities`
    1. Pose Extraction
        - Mediapipe 3D world coordinates
        - Pose Visualization to aid in analysis and understanding.
    2. Text normalization
        - Normalize text input by substituting unknown characters/spellings with supported words.
        - Disambiguate ambiguous words to ensure accurate translation.

4. `Data Collection and Creation`
    - Capture variations in signs in a scalable and diversity accommodating way and enable advancing sign language standardization efforts.

      1. Clip extraction from long videos using timestamps
      2. Multithreaded Web scraping
      3. Language Models to generate sentences composed of supported word

5. `Datasets`

    The sign videos are categorized by:

    ```text
    1. country
    2. source organization
    3. session number
    4. camera angle
    5. person code ((d: deaf | h: hearing)(m: male | f: female)000001)
    6. equivalent text language word
    ```

    The files are labeled as follows:

    ```text
    country_organization_sessionNumber_cameraAngle_personCode_word.extension
    ```

    The text data includes:

    ```text
    1. word/sentence mappings to videos
    2. spoken language sentences and phrases
    3. spoken language sentences & corresponding sign video label sequences
    4. preprocessing data such as word-to-numbers, misspellings, named-entities etc
    ```

    [See the *sign-language-datasets* repo and its *release files* for the actual data & details](https://github.com/sign-language-translator/sign-language-datasets)

## How to install the package

Production mode:

```bash
pip install sign-language-translator
```

Editable mode:

```bash
git clone https://github.com/sign-language-translator/sign-language-translator.git
cd sign-language-translator
pip install -e .
```

```bash
pip install -e git+https://github.com/sign-language-translator/sign-language-translator.git#egg=sign_language_translator
```

## Usage

see the *test cases* or [the *notebooks* repo](https://github.com/sign-language-translator/notebooks) for detailed use

### Command Line

<!-- #### configure

(Optional) Set the dataset directory.

```bash
slt configure --dataset-dir "/path/to/sign-language-datasets"
``` -->

#### Download

Download dataset files or models. The parameters are regular expressions.

```bash
slt download --overwrite true '.*\.json' '.*\.txt'
```

```bash
slt download --progress-bar true 't2s_model_base.pth'
```

(By default, the dataset is downloaded into `/install-directory/sign_language_translator/sign-language-datasets/`)

#### Translate

Translate text to sign language using a rule-based model

```bash
slt translate \
--model-code "concatenative" \
--text-lang urdu --sign-lang psl \
--sign-features 'mp-landmarks' \
"وہ سکول گیا تھا۔" \
'مجھے COVID نہیں ہے!'
```

### Python

#### basic translation

```python
import sign_language_translator as slt

# download dataset (by default, dataset is downloaded within the install directory)
# slt.set_dataset_dir("path/to/sign-language-datasets") # optional
# slt.download("id")
```

```python
# Load text-to-sign model
# deep_t2s_model = slt.get_model("generative_t2s_base-01") # pytorch
# rule-based model (concatenates clips of each word)
t2s_model = slt.get_model(
    model_code = "ConcatenativeSynthesis", # slt.enums.ModelCodes.CONCATENATIVE_SYNTHESIS.value
    text_language = "English", # or object of any child of slt.languages.text.text_language.TextLanguage class
    sign_language = "PakistanSignLanguage", # or object of any child of slt.languages.sign.sign_language.SignLanguage class
    sign_feature_model = "mediapipe_pose_v2_hand_v1",
)

text = "hello world!"
sign_language_sentence = t2s_model(text)

# moviepy_video_object = sign_language_sentence.video()
# moviepy_video_object.ipython_display()
# moviepy_video_object.write_videofile(f"sentences/{text}.mp4")
```

```python
# load sign
video = slt.read_video("video.mp4")
# features = slt.extract_features(video, "mediapipe_pose_v2_hand_v1")

# Load sign-to-text model
deep_s2t_model = slt.get_model("gesture_mp_base-01") # pytorch
features = deep_s2t_model.extract_features(video)

# translate
text = deep_s2t_model(features)
print(text)
```

###### text language processor

```python

from sign_language_translator.languages.text import Urdu
ur_nlp = Urdu()

text = "hello جاؤں COVID-19."

normalized_text = ur_nlp.preprocess(text)
# normalized_text = 'جاؤں COVID-19.'

tokens = ur_nlp.tokenize(normalized_text)
# tokens = ['جاؤں', ' ', 'COVID', '-', '19', '.']

# tagged = ur_nlp.tag(tokens)
# tagged = [('جاؤں', Tags.SUPPORTED_WORD), (' ', Tags.SPACE), ...]

tags = ur_nlp.get_tags(tokens)
# tags = [Tags.SUPPORTED_WORD, Tags.SPACE, Tags.ACRONYM, ...]

# word_senses = ur_nlp.get_word_senses("میں")
# word_senses = [["میں(i)", "میں(in)"]]
```

###### sign language processor

```python
from sign_language_translator.languages.sign import PakistanSignLanguage

psl = PakistanSignLanguage()

tokens = ["he", " ", "went", " ", "to", " ", "school", "."]
tags = [Tags.WORD, Tags.SPACE] * 3 + [Tags.WORD, Tags.PUNCTUATION]
tokens, tags, _ = psl.restructure_sentence(tokens, tags) # ["he", "school", "go"]
signs  = psl.tokens_to_sign_dicts(tokens, tags)
# signs = [
#   {'signs': [['pk-hfad-1_وہ']], 'weights': [1.0]},
#   {'signs': [['pk-hfad-1_school']], 'weights': [1.0]},
#   {'signs': [['pk-hfad-1_گیا']], 'weights': [1.0]}
# ]
```

###### language models

```python
from sign_language_translator.models.language_models import BeamSampling, SimpleLanguageModel, Mixer, TransformerLanguageModel


```

## Directory Tree

```text
sign-language-translator
├── MANIFEST.in
├── README.md
├── poetry.lock
├── pyproject.toml
├── requirements.txt
├── roadmap.md
├── tests
│   └── *
│
└── sign_language_translator
    ├── cli.py
    ├── config
    │   ├── enums.py
    │   ├── helpers.py
    │   ├── settings.py
    │   └── urls.yaml
    │
    ├── data_collection
    │   ├── completeness.py
    │   ├── scraping.py
    │   └── synonyms.py
    │
    ├── languages
    │   ├── utils.py
    │   ├── vocab.py
    │   ├── sign
    │   │   ├── mapping_rules.py
    │   │   ├── pakistan_sign_language.py
    │   │   └── sign_language.py
    │   │
    │   └── text
    │       ├── english.py
    │       ├── text_language.py
    │       └── urdu.py
    │
    ├── models
    │   ├── utils.py
    │   ├── language_models
    │   │   ├── abstract_language_model.py
    │   │   ├── beam_sampling.py
    │   │   ├── mixer.py
    │   │   ├── simple_language_model.py
    │   │   └── transformer_language_model.py
    │   │
    │   ├── sign_to_text
    │   └── text_to_sign
    │       ├── concatenative_synthesis.py
    │       └── t2s_model.py
    │
    ├── sign-language-datasets
    │   └── *
    │
    ├── text
    │   ├── metrics.py
    │   ├── preprocess.py
    │   ├── subtitles.py
    │   ├── tagger.py
    │   ├── tokenizer.py
    │   └── utils.py
    │
    ├── utils
    │   ├── data_loader.py
    │   ├── download.py
    │   ├── landmarks_info.py
    │   ├── sign_data_attributes.py
    │   └── tree.py
    │
    └── vision
        ├── concatenate.py
        ├── embed.py
        ├── transforms.py
        └── visualization.py
```

## Research Paper

Stay Tuned!

## Credits and Gratitude

This project started in October 2021 as a BS Computer Science final year project with 3 students and 1 supervisor. After 9 months at university, it became a hobby project for Mudassar who has continued it till at least 2023-07-11.

Immense gratitude towards:

- [Mudassar Iqbal](https://github.com/mdsrqbl) for leading and coding the project so far.
- Rabbia Arshad for help in initial R&D and web development.
- Waqas Bin Abbas for assistance in video data collection process.
- Kamran Malik for setting the initial project scope, idea of motion transfer and connecting us with Hamza Foundation.
- [Hamza Foundation](https://www.youtube.com/@pslhamzafoundationacademyf7624/videos) (especially Ms Benish, Ms Rashda & Mr Zeeshan) for agreeing for collaboration and providing the reference clips, hearing-impaired performers for data creation, and creating the text2gloss dataset.
- [UrduHack](https://github.com/urduhack/urduhack) (espacially Ikram Ali) for their work on Urdu character normalization.

- [Telha Bilal](https://github.com/TelhaBilal) for help in designing the architecture of some modules.

## Bonus

Count total number of **lines of code** (Package: **7064** + Tests: **774**):

```bash
git ls-files | grep '\.py' | xargs wc -l
```

**Just for fun**

```text
Q: What was the deaf student's favorite course?
A: Communication skills
```

**Publish package on `PyPI`**

1. Install Poetry [(official docs)](https://python-poetry.org/docs/#installation) and twine:

    ```bash
    curl -sSL https://install.python-poetry.org | python3 -
    ```

    ```bash
    pip install twine
    ```

2. Initialize Poetry using the following command to create a new pyproject.toml file with the necessary project information

    existing project:

    ```bash
    poetry init
    ```

    new project:

    ```bash
    poetry new project-name
    ```

    add dependencies to pyproject.toml

    ```bash
    poetry add $(cat requirements.txt)
    ```

3. Build Distribution Files (might wanna add "dist/" to .gitignore)

    ```bash
    poetry build
    ```

4. Publish to PyPI

    ```bash
    twine upload dist/*
    ```

    Provide the credentials associated with your PyPI account.

5. Automate the Release Process (GitHub Actions)

    - Set up PyPI Configuration:
    - on Github repository page, go to "Settings" > "Secrets" and add PYPI_API_TOKEN as secret
    - Create and push .github/workflows/release.yml. Configure the workflow to:
    - trigger on the main branch update event.
    - check if version has updated using git diff
    - build and test your package.
    - publish the package to PyPI using Twine using secret credentials.

    - before actual publish, test your publishing process on test.pypi.org with `twine upload --repository testpypi dist/*`

    With this setup, whenever there is a new version update on the main branch, the CI tool will automatically build and release the package to PyPI.
