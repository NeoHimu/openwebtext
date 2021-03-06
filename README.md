# OpenWebText

[Joshua Peterson](http://joshpeterson.io), [Stephan Meylan](https://stephanmeylan.com), & David Bourgin

Open clone of OpenAI's unreleased WebText dataset ([blog](https://blog.openai.com/better-language-models/), [paper](https://d4mucfpksywv.cloudfront.net/better-language-models/language_models_are_unsupervised_multitask_learners.pdf), [code](https://github.com/openai/gpt-2)) scraper used to train GPT-2. The project was started via [this reddit post](https://www.reddit.com/r/MachineLearning/comments/aqzjv1/d_open_alternative_reddit_scraper_inspired_by/). The current result is just over 23 million URLs and over 10 million HTML pages.

This implementation mines and intelligently de-duplicates +3 karma URLs from pre-downloaded (monthly) pushshift.io Reddit submission dumps (which is much faster than making successive calls to the web API), downloads raw HTML, and extracts text. There's also an initial utility for tokenizing and we are looking to add BPE encoding soon. This code base is functional but in active development so please feel free to post issues or suggest improvements (pull requests welcome).

### Dependencies
If you use pipenv (`pip install --user pipenv`), cd to the project root and run
```
pipenv install 
pipenv shell
```
Otherwise, just run the following in a new virtual environment
```
pip3 install -r requirements.txt
```

### To Extract/Clean URLs Yourself
Pushshift dumps must be downloaded from [here](https://files.pushshift.io/reddit/submissions/) (auto-downloader to be added soon). Two examples are included in the repo in the "pushshift_dumps" folder. Then, extract good URLs using:
```
python extract_urls.py --single_file RS_v2_2005-06.xz
```
To process multiple pushshift files, specify year ranges:
```
python extract_urls.py --year_start 2016 --year_end 2018
```
To change the karma threshold:
```
python extract_urls.py --single_file RS_v2_2005-06.xz --min_karma 4
```
To de-duplicate the extracted URLs, provide a directory of all URL dumps:
```
python deduplicate_urls.py --input_dir url_dumps
```
The output of both `extract_urls.py` and `deduplicate_urls.py` are text files given that all 23 million "good" URLs only comprise 2GB.

### To Scrape HTML
This is done one month at a time given the compute/bandwidth required. `n_procs` is the number of cores to use for parallelization and should be at least `20-40` for fastest results. The script will output results in chunks of size `chunk_size`. If `timeout` is not set, or is set to `-1`, the downloader may hang on large files.
```
python download.py url_dumps_deduped/RS_20XX-XX.xz.deduped.txt --n_procs 100 --scraper raw --chunk_size 100000 --compress --timeout 30
```
The downloaded HTML is stripped of script/style tags and stored in compressed archives using LZMA compression, along with a small amount of meta.

### To Extract Text from HTML
```
python extract_text.py scraped/RS_20XX-XX-X_data.xz --n_procs 100 
```
This currently outputs txt files.

### Tokenization
The original WebText didn't use tokenization, but if you need it use:
```
python tokenize_text.py --input_glob "parsed/*.txt" --output_dir tokenized
```
This will be improved and parallelized soon.

### BPE Encoding
Coming soon...

### Original OpenAI project links
* Blog Post [(Better Language Models and Their Implications)](https://blog.openai.com/better-language-models/)
* Paper [(Language Models are Unsupervised Multitask Learners)](https://d4mucfpksywv.cloudfront.net/better-language-models/language_models_are_unsupervised_multitask_learners.pdf)
* Code [(https://github.com/openai/gpt-2)](https://github.com/openai/gpt-2)

### Other Implmentations
An alternative scraper based on the pushshift.io API and fork of the download code above can be found [here](https://github.com/eukaryote31/openwebtext)