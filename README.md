# UIT Data Science Challenge 2023 - Group B
Team: URA_FNU
Members:
- Lưu Chấn Hưng
- Trần Ngọc Oanh
- Nguyễn Đắc Hoàng Phú
- La Cẩm Huy
- Lê Quốc Bảo

# Approach Summary
### Approach Highlights
Our attempt to solve the Challenge includes the following highlights:
1. Data Cleaning
2. Fine-tuning Sentence Transformers for Evidence Retrieval with Multiple Negative Ranking Strategy
3. Fine-tuning mDeBERTA for Natural Language Inference
4. 2 steps Retrieval Process
5. Check special REFUTED cases with POS Tagging

### System 
The system can be explained in 4 steps:
1. We perform a 2 steps retrieval process.
    - First we retrieve Top 5 passages in the Context (which is divided by \n\n).
    - Then we retrieve Top 5 sentences in the Top 5 passages (which is divided by Underthesea Tokenizer).
2. After that, we perform checking the claim and each retrieved evidence to find special cases.
3. We then run a fine-tuned mDEBERTA model to predict verdict for each (claim, evidence) pair.
4. Finally, we combined the result
    - We select Checker result if it is not null
    - Then we select NLI result with the below rules:
        - If all 5 verdict prediction is NEI -> return NEI and set evidence to empty string ('')
        - If verdict prediction list contain SUPPORTED and REFUTED, we will select the resul with the highest match score from difflib.SequenceMatcher library

### Models
1. Retrieval
    - Base model: Vietnamese Sentence BERT (keepitreal/vietnamese-sbert)
    - Finetuned model path on Transformers library: Oztobuzz/sbert_mnr_5_epoch_v1
2. NLI
    - Base model: MoritzLaurer/mDeBERTa-v3-base-mnli-xnli
    - Finetuned model path: model/mDeBERTa (ft) V6
3. Checker
    - POS Tagger model: PhoNLP
    
# Source Code Explanation
### Note
In order to utilize GPU resource, we ran inference process for submission on Google Colab and Kaggle. This process is produced by multiple members, and the output for each step will be stored on Google Drive.
Therefore, we will provide exact location of each input and output for each step in this file.

### Test Data Inference Process
Our system includes 4 step:
1. Retrieval:
    - Source: inference/Step_1_Retrieval.ipynb
    - Platform: Kaggle 
    - Input: 
        - Private test dataset: data/private/ise-dsc01-private-test-offcial.json
    - Output:
        - Retrieval File: result/retrieval_result/private_test_retrieval_v1_top5_top_5.json
2. Checker:
    - Source: inference/Step_2_Checker.ipynb
    - Platform: Kaggle
    - Input:
        - Retrieval File: result/retrieval_result/private_test_retrieval_v1_top5_top_5.json
    - Output:
        - Checked File: result/private_test_retrieval_v1_top5_top_5_checked.json
3. NLI:
    - Source: inference/Step_3_NLI.ipynb
    - Platform: Google Colab
    - Model path: model/ mDeBERTA (ft) V6
    - Input:
        - Retrieval File: result/retrieval_result/private_test_retrieval_v1_top5_top_5.json
    - Output:
        - NLI_result (As our model output 2 config, using Mean Pooling or use CLS token as output, therefore it will provide 2 output)
            - result/private_test_retrieval_v1_top5_top_5_mDeBERTa (ft) V6(mean).json
4. Post Process
    - Source: inference/Step_4_Post_Processing.ipynb
    - Platform: Local Machine
    - Input:
        - Checker result: result/private_test_retrieval_v1_top5_top_5_checked.json
        - NLI result: result/private_test_retrieval_v1_top5_top_5_mDeBERTa (ft) V6(mean).json
    - Output:
        - Final Submission Result: result/private_submit_v3_mean_v1/private_result.json


### Data Processing
1. Clean NLI Training Dataset
In this process, we performed the following steps:
- We use our retrieval model to retrieve top 5 sentences in top 5 passages in train set.
- For REFUTED and SUPPORTED cases: we remove a sample if its evidence is not in our Retrieval Result
- For NEI cases, we choose the evidence with the highest score from difflib.SequenceMatcher library (the most similar evidence)
Source: processing/NLI_Train_Data_Cleaning.ipynb
Input: data/public/ise-dsc01-train.json
Output: result/processed_training_data/public_train_v4.json

2. Create Retrieval Training Dataset
To create a datase for Multiple Negative Ranking Training, we created 3 samples for each output:
- Positive Example: Evidence in the training dataset
- Hard Negative Example: Top 2 evidence retrieve by BM25 (Top 1 is positive evidence)
- Soft Negative Example: A random sentence from different sample in training data
Source: processing/Retrieval_Train_Data_Creation.ipynb
Input: data/public/ise-dsc01-train.json
Output: result/processed_training_data/retrieval_dataset.csv

### Training Process
1. NLI Finetuning
- Based Model: MoritzLaurer/mDeBERTa-v3-base-mnli-xnli
- Training Dataset: result/processed_data/public_train_v4.json
- Output Model: model/mDeBERTa (ft) V6
- Training Script: training/ NLI_Finetuning.ipynb
- Platform: Google Colab
2. Retrieval Finetuning
- Based model: Vietnamese Sentence BERT (keepitreal/vietnamese-sbert)
- Training Dataset: result/processed_training_data/retrieval_dataset.csv
- Output model: Oztobuzz/sbert_mnr_5_epoch_v1 (Transformers Library Path)
- Training Script: training/ Retrieval_Finetuning.ipynb
- Platform: Google Colab