Ձեր համակարգիչում պետք է տեղադրված լինի python փաթեթը (ծրագրակազմը կարելի է տեղադրել python.org-ից)

Ստեղծում եք աշխատանքային ծրար, ենթադրենք RI անունով

Ներբեռնում եք ri_app.py (եթե գործածում եք jupyter notebook-ը RI-app.ipynb) և տեղադրում այդ ծրարում

Նույն ծրարում ստեղծում եք texts և results ծրարները

texts ծրարում տեղադրում եք աշխատանքային տեքստերը

terminal-ում (cmd, command line) տեղափոխվում եք աշխատանքային ծրարի մեջ

աշխատացնում եք python ri_app.py

հնարավոր է պահանջի matplotlib 

Գրում եք pip install matplotlib

Արդյունքները կգրվեն results ծրարում

## Data

A parallel Armenian-English corpus with Automated Readability Index (ARI) scores for English text pieces was built in order to calculate the corresponding ARI scores in Armenian.

## Format
Armenian textbooks data conforms to the following specifics:
* ARI estimated score is present as `# index_est_score = 12`
* Document long and short titles are present as
  * `# doc_title_long = ԱՇԽԱՐՀԱԳՐՈՒԹՅՈՒՆ 12: Հանրակրթական ավագ դպրոցի 12–րդ դասարանի դասագիրք / Մ. Մանասյան, Ա. Հովսեփյան, Ա. Պոտոսյան.— Եր.: «Զանգակ» հրատ., 2025.— 128 էջ:`
  * `# doc_title_short = Աշխարհագր.-12`
* Document boundaries are present as `# doc id = geographia/physica/see_domains-xxxx`
* Document-level unit boundaries are present as `newunit id = doc-xxxx`
* Lesson-level author boundaries are present as `newauthor id = doc-newunit-xxxx`
* Lessson boundaries are present as `newlesson id = doc-newunit-newauthorxxxx`

* Document, unit, author and lesson ids are 4-character base-32 numbers. They survive treebank updates.
