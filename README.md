# Readability-app

A Python tool that measures how readable Armenian school textbooks are. For each book it calculates the **Automated Readability Index (ARI)** using coefficients fitted for Armenian. It also reports text statistics for the whole book, each unit and each lesson, and compares them with the values expected for the book's grade.

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Project structure](#project-structure)
- [Input format](#input-format)
- [Book index file](#book-index-file)
- [How it works](#how-it-works)
- [Output](#output)
- [Corpus](#corpus)
- [Known limitations](#known-limitations)
- [License](#license)

## Features

- ARI score and school level for every lesson, every unit and the whole book
- Character, word and unique word counts, and the word usage ratio
- Average word length and average sentence length, each shown next to the value expected for the grade
- The 7 longest words and the longest sentence of each text
- Charts of the word length and sentence length distribution for each lesson
- A chart of the ARI score of each lesson against the book's expected and actual score
- Processes several books in one run, writing each book's results to its own folder

## Requirements

- Python 3.10 or newer (the code uses `match` statements)
- [matplotlib](https://matplotlib.org/)

## Installation

```bash
git clone https://github.com/RLALT/Readability-app.git
cd Readability-app
pip install matplotlib
```

## Usage

1. Put each textbook in its own folder inside `texts/`, one `.txt` file per lesson (see [Input format](#input-format)).
2. Run the script from the project folder:

   ```bash
   python ri_app.py
   ```

   To use Jupyter instead, open `RI_app.ipynb` and run all cells. It has the same code.

3. Open `results/<book folder>/` to see the report and charts.

The script also prints the full report to the terminal. When the book chart opens in a window, close it so the script can go on to the next book.

## Project structure

```
Readability-app/
├── ri_app.py            # the application
├── RI_app.ipynb         # the same code as a Jupyter notebook
├── texts/               # input textbooks
│   ├── _domains.txt     # Armenian subject names and their domain codes
│   ├── geographia-12/   # one folder per book: <domain>-<grade>
│   │   ├── 01.txt       # one file per lesson
│   │   ├── ...
│   │   └── _index.txt   # the book's index file (see "Book index file")
│   └── physica-08/
├── data/                # parallel English–Armenian ARI corpus
│   ├── train_1-14-translatum.txt
│   └── test_1-14-translatum.txt
├── results/             # output (created if missing)
└── LICENSE.md
```

## Input format

Each lesson is a UTF-8 `.txt` file (a BOM is allowed). It starts with metadata lines of the form `# key = value`, followed by the lesson text.

```
# doc_title_long = ԱՇԽԱՐՀԱԳՐՈՒԹՅՈՒՆ 12: Հանրակրթական ավագ դպրոցի 12–րդ դասարանի դասագիրք / ...
# doc_title_short = Աշխարհագր.-12
# doc id = geographia-00AA
# index_est_score = 12
# newunit_title_long = ԱՇԽԱՐՀԻ ԻՆՔՆԱՏԻՊ ՏԱՐԱԾԱՇՐՋԱՆՆԵՐՆ ՈՒ ԵՐԿՐՆԵՐԸ
# newunit_title_short = ԱՇԽ. ԵՐԿՐ.
# newunit id = geographia-00AA-0001
# newauthor_name = Մանասյան Մ.
# newauthor id = geographia-00AA-0001-00A1
# newlesson_title = §1. ԻՆՔՆԱՏԻՊ ՏԱՐԱԾԱՇՐՋԱՆՆԵՐ
# newlesson id = geographia-00AA-0001-00A10AAA
# text = First line of the lesson text
The lesson text continues on the following lines...
```

| Key | Meaning |
|---|---|
| `doc_title_long`, `doc_title_short` | Full bibliographic title and short title of the book (the short title is used on the book chart) |
| `doc id` | Book id, `<domain>-xxxx`, with the domain code taken from [`texts/_domains.txt`](texts/_domains.txt) |
| `index_est_score` | **Required.** Expected ARI level of the book, a whole number from 0 to 14 |
| `newunit_title_long`, `newunit_title_short`, `newunit id` | Unit (chapter) titles and id, `<doc id>-xxxx` |
| `newauthor_name`, `newauthor id` | Lesson author and id, `<unit id>-xxxx` |
| `newlesson_title`, `newlesson id` | Lesson title and id, `<author id>xxxx` |
| `text` | Start of the lesson text; the text continues on the lines after it |

Ids use 4-character base-32 numbers.

A lesson without a valid `index_est_score` uses the value from the lessons before it. If there are no earlier lessons, it is skipped. Lessons with no text are skipped too.

## Book index file

Each book folder has an `_index.txt` file, which works as the book's table of contents. It lists the book's metadata once, then every unit, and under each unit the lesson files in reading order with their titles and authors. Both book folders in `texts/` include one, which you can use as a template: [geographia-12/_index.txt](texts/geographia-12/_index.txt) and [physica-08/_index.txt](texts/physica-08/_index.txt).

```
# index_id = ARI
# index_est_score = 12
# index_score =

# doc_title_long = ԱՇԽԱՐՀԱԳՐՈՒԹՅՈՒՆ 12: Հանրակրթական ավագ դպրոցի 12–րդ դասարանի դասագիրք / ...
# doc_title_short = Աշխարհագր.-12

# newunit_title_long = ԱՇԽԱՐՀԻ ԻՆՔՆԱՏԻՊ ՏԱՐԱԾԱՇՐՋԱՆՆԵՐՆ ՈՒ ԵՐԿՐՆԵՐԸ
# newunit_title_short = ԱՇԽ. ԵՐԿՐ.
# doc = 01.txt
# newlesson_title = §1. ԻՆՔՆԱՏԻՊ ՏԱՐԱԾԱՇՐՋԱՆՆԵՐ
# newauthor_name = Մանասյան Մ.
# doc = 02.txt
# newlesson_title = §2. ...
# newauthor_name = Հովսեփյան Ա.

# newunit_title_long = <next unit>
...
```

| Key | Meaning |
|---|---|
| `index_id` | The readability index used (`ARI`) |
| `index_est_score` | Expected score for the book |
| `index_score` | Left empty, for the calculated score |
| `doc_title_long`, `doc_title_short` | Book titles |
| `newunit_title_long`, `newunit_title_short` | Starts a unit. The lessons listed after it belong to that unit. |
| `doc` | Name of a lesson file in the folder |
| `newlesson_title`, `newauthor_name` | Title and author of the lesson file named in the `doc` line before them |

The script doesn't read `_index.txt`, because files whose names start with `_` are skipped. Each lesson file still needs its own metadata header, and the values should match the index. Use the index to plan a book's lessons and to check the lesson headers.

## How it works

1. **Finding books.** Each folder in `texts/` is one book. `.txt` files placed directly in `texts/` count as one more book. Inside a book, files are read in sorted path order, and files whose names start with `_` are ignored.
2. **Parsing.** Lines that start with `#` and contain `=` are read as metadata. All other non-empty lines are lesson text.
3. **Grouping.** Lessons that follow each other with the same `newunit_title_long` form one unit. All units together form the book.
4. **Counting.**
   - *Characters*: every character in the text except spaces and line breaks
   - *Words*: the text is lowercased and punctuation is removed, then it is split on whitespace. Reference numbers attached to words of 5 or more letters are removed.
   - *Sentences*: the text is split on `։`, `:`, `!`, `?` and line breaks
5. **Score.** ARI is calculated with the Armenian coefficients:

   ```
   ARI = 1.116 × (characters / words) + 0.661 × (words / sentences) − 6.494
   ```

6. **School level.** The score is mapped to a level: `Նախակրթարան` (below 1), then one grade per point up to `Տասներկուերորդ դասարան` (12 to 13), then `Բուհական` (13 to 15), `Հետբուհական` (15 to 18) and `Փորձագիտական` (18 and above).
7. **Comparison.** Average word length, average sentence length and the ARI score are compared with the expected values for the book's `index_est_score`:

| `index_est_score` | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Word length (chars) | 4.2 | 4.7 | 4.9 | 5.2 | 5.4 | 5.5 | 5.9 | 6.2 | 6.4 | 6.6 | 6.7 | 6.9 | 7.1 | 7.2 | 7.6 |
| Sentence length (words) | 7.0 | 7.9 | 8.4 | 8.8 | 9.2 | 9.7 | 10.1 | 10.6 | 11.0 | 11.4 | 11.9 | 12.3 | 13.2 | 15.0 | 17.6 |
| ARI | −1.8 | 0.58 | 1.61 | 2.96 | 3.94 | 4.6 | 6.32 | 7.73 | 8.71 | 9.68 | 10.34 | 11.32 | 12.58 | 13.98 | 16.96 |

## Output

For each book, files are written to `results/<book folder>/` (for example `results/geographia-12/`):

| File | Contents |
|---|---|
| `_արդիւնք.txt` | The full report: book, then units, then lessons. Each part has its ARI score, school level and statistics. |
| `<file>_<lesson title>_word.png` | Word length distribution of a lesson (% of unique words). An inset shows how far the average is from the expected value. |
| `<file>_<lesson title>_sent.png` | Sentence length distribution of a lesson (% of sentences), with the same inset |
| `by_lesson.png` | ARI score of each lesson, with lines for the book's expected and actual score |

In chart file names, characters that file names can't contain (such as `:`) are replaced with `_`. Names are cut to 80 characters.

## Corpus

`data/` holds a parallel English–Armenian corpus. Its English texts have known ARI levels from 1 to 14. It was built to fit the Armenian ARI coefficients used above.

- `train_1-14-translatum.txt` is the training split and `test_1-14-translatum.txt` is the test split
- Each text has these metadata lines:
  - `# doc_title_en`, `# doc_title`: corpus title in English and Armenian
  - `# index_est_score = ARI-01`: ARI level
  - `# doc_id = ARI-01-translatum`: level boundary
  - `# newtext_id = ARI-01-translatum-AA01`: text boundary
  - `# parallel_id = p_ARI-01-translatum-AA01`: links the English text to its Armenian translation
  - `# text_en = ...`, `# text_hy = ...`: start of the English and Armenian texts

Source: Anna S. Danielyan. *Parallel Texts for the Armenian Automated Readability Index*. Yerevan, 2025.

## Known limitations

- Only `.txt` input is supported.
- Line breaks end sentences, so headings and list items count as short sentences.
- `index_est_score` must be a whole number from 0 to 14. Values outside this range are treated as missing.
- With an interactive matplotlib backend, the book chart opens in a window and the script waits until it is closed.

## License

[CC BY-NC-ND 4.0](LICENSE.md): Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International.
