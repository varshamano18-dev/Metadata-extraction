# Metadata Extraction from Documents

An AI/ML system that extracts 6 metadata fields from rental agreement
documents (.docx, scanned .png, .pdf): Agreement Value, Agreement Start Date,
Agreement End Date, Renewal Notice (Days), Party One and Party Two.
No RegEx or rule-based extraction is used.

## Repository contents
| File | Description |
|---|---|
| data_preprocessing.ipynb | Colab notebook used to pre-process the raw data and produce the final cleaned CSV files |
| Model_train_evaluation.ipynb | Colab notebook that trains the model, predicts on the test set and computes recall |
| Data/train.zip, Data/test.zip | Training (10) and test (4) documents |
| Data/final_cleaned_train.csv, Data/final_cleaned_test.csv | Pre-processed metadata used for training and evaluation |
| predictions.csv | Model predictions for the test folder |

## Approach
1. Pre-processing: the raw data was cleaned in data_preprocessing.ipynb
   (Google Colab) and saved as final_cleaned_train.csv and
   final_cleaned_test.csv. In the model notebook the column names are also
   corrected (Aggrement -> Agreement) and Renewal Notice values are converted
   from 15.0 to 15.
2. Document reading: docx files are read with python-docx, and scanned
   png/pdf files with Tesseract OCR (PyMuPDF for pdf).
3. Model: a pretrained language model (Qwen2.5-1.5B-Instruct) is fine-tuned
   with LoRA on the 10 training documents. For each document it generates a
   JSON with the 6 fields.
4. Evaluation: the model is tested on the 4 documents in test.zip and the
   per-field recall (exact match) is computed against final_cleaned_test.csv.

## How to run
1. Open Model_train_evaluation.ipynb in Google Colab and select a
   T4 GPU runtime (Runtime -> Change runtime type).
2. Upload train.zip, test.zip, final_cleaned_train.csv and
   final_cleaned_test.csv from the Data folder.
3. Run the cells in order. Training takes about 3 minutes.
4. predictions.csv is created at the end.

## Predictions
predictions.csv contains the model predictions for the files in the test folder.

## Per-field recall (test set, 4 documents)
Strict exact match:
- Agreement Value: 1.0
- Agreement Start Date: 0.0
- Agreement End Date: 0.0
- Renewal Notice (Days): 0.25
- Party One: 0.75
- Party Two: 0.25
- Average: 0.38

The train CSV writes dates as DD-MM-YYYY while the test CSV uses DD.MM.YYYY,
so the strict match fails on the separator even when the date is correct.
Ignoring only the date separator:
- Agreement Value: 1.0
- Agreement Start Date: 1.0
- Agreement End Date: 0.75
- Renewal Notice (Days): 0.25
- Party One: 0.75
- Party Two: 0.25
- Average: 0.67

## Limitations
Only 10 training documents were available. The model predicted a Renewal
Notice of 60 for all test documents, and Party One and Party Two were swapped
in one document. More training data would improve these fields.
