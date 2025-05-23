# BRIDGE: Pivot-based Suicidal text detection and analysis in Telugu via English using domain adversarial training (DAT).

This project focuses on detecting suicidal ideation in Telugu text using a fine-tuned MuRIL model, enhanced with explanations generated by Google's Gemini model. It employs Domain Adversarial Training (DAT) leveraging English data as a pivot to improve performance in the target Telugu language.

---

## Project Overview

The primary goal is to classify Telugu text as either "Suicidal" or "Non-Suicidal". The system leverages the multilingual capabilities of the MuRIL model, adapted for the specific nuances of Telugu through Domain Adversarial Training (DAT). Additionally, it integrates Gemini 1.5 Flash to provide reasoning behind the MuRIL model's predictions, offering insights into *why* certain text is classified in a particular way based on the tokens the MuRIL model focused on.

### Key Features

* **Telugu Suicide Detection:** Classifies input Telugu text for potential suicidal content.
* **Domain Adversarial Training (DAT):** Utilizes DAT to improve the model's performance on the target Telugu domain while leveraging a larger English dataset during training. The reference paper for this method is [Unsupervised Domain Adaptation by Backpropagation](https://arxiv.org/abs/1505.07818).
* **MuRIL Model:** Fine-tunes the `google/muril-base-cased` model, known for its strong performance on Indian languages.
* **Gemini-Powered Reasoning:** Employs the Gemini 1.5 Flash model to generate explanations for the MuRIL model's predictions, based on the important tokens identified via attention mechanisms.
* **Synthetic Dataset:** Uses a synthetically generated Telugu dataset created by translating an English suicide watch dataset using the `ai4bharat/indictrans2-en-indic-1B` model.

---

## Architecture

![model architechture](https://github.com/user-attachments/assets/ca10c611-b496-477d-b0e9-631330d637f2)


---

## Project Information


![Project_Info](https://github.com/user-attachments/assets/6b0897b5-27bd-4f7c-9e9e-fd5090c2f44c)

---

## Work-Flow

![Inference_Flow](https://github.com/user-attachments/assets/ced46f55-f694-4495-be70-d8cd2ed60816)

---

## Repository Structure

```
NLP/
├── Traning and Inference Code/
│   ├── Telugu_Synthetic_Dataset translation code.ipynb  # Code to generate Telugu dataset
│   ├── Muril_Model_Training_code.ipynb                 # MuRIL DAT training code
│   └── Muril-Model-Inference and Reasoning.ipynb      # Inference and Gemini reasoning code
├── Inference_Flow.jpeg                                  # Diagram of the inference process
├── Model_Architechture.png                              # Diagram of the model architecture
└── Project_Info.jpeg                                    # Image containing project details/flow
```
---

## Installation

1.  Clone the repository:
    ```bash
    git clone [https://github.com/bharateesha2004/NLP.git](https://github.com/bharateesha2004/NLP.git)
    cd NLP
    ```

2.  Create and activate a virtual environment (recommended):
    ```bash
    python -m venv venv
    # On Linux/macOS:
    source venv/bin/activate
    # On Windows:
    # venv\Scripts\activate
    ```

3.  Install required dependencies:
    ```bash
    pip install torch transformers sentencepiece numpy pandas requests scikit-learn google-generativeai bitsandbytes git+[https://github.com/VarunGumma/IndicTransToolKit](https://github.com/VarunGumma/IndicTransToolKit)
    ```
    *(Note: `bitsandbytes` and `IndicTransToolKit` are primarily needed for the dataset translation notebook. `google-generativeai` is needed for the inference/reasoning notebook. Other packages are used across notebooks).*

4.  Set up your Google API key (for Inference/Reasoning):
    * Get your API key from [Google AI Studio](https://aistudio.google.com/app/apikey).
    * In `Traning and Inference Code/Muril-Model-Inference and Reasoning.ipynb`, replace the placeholder `"Your api key here"` with your actual key. Alternatively, configure it as an environment variable if preferred.

---

## Datasets

1.  **English Source Dataset:** The initial English dataset was sourced from Kaggle: [Suicide Watch Dataset](https://www.kaggle.com/datasets/nikhileswarkomati/suicide-watch). Used for translation and for DAT training.
2.  **Telugu Synthetic Dataset:** The first 50,000 rows of the English dataset were translated into Telugu using the `ai4bharat/indictrans2-en-indic-1B` model. The code for this translation process is available in `Traning and Inference Code/Telugu_Synthetic_Dataset translation code.ipynb`. The resulting dataset contains 'text' and 'class' columns (with Telugu text and labels like "ఆత్మహత్య" or "ఆత్మహత్య కాదు") and was converted to binary class labels and then used for DAT training.

---

## Model & Training

* **Base Model:** `google/muril-base-cased`.
* **Training Method:** Domain Adversarial Training (DAT). This technique adapts the model trained on a source domain (English) to perform well on a target domain (Telugu) by making the model's internal representations domain-invariant.
    * **Feature Extractor:** The base MuRIL model acts as the feature extractor, generating text representations.
    * **Task Classifier:** A linear layer predicts the main task (Suicidal vs. Non-Suicidal) based on the extracted features. It is trained to minimize the task loss (Cross-Entropy Loss in this case).
    * **Domain Classifier:** Another linear layer tries to predict the domain (English vs. Telugu) from the same features. It is trained to minimize the domain classification loss.
    * **Gradient Reversal Layer (GRL):** This crucial layer is placed between the feature extractor and the domain classifier. During backpropagation, it reverses the gradient flowing *to* the feature extractor *from* the domain classifier loss. This encourages the feature extractor to learn representations that are *useful* for the task classifier but *confusing* for the domain classifier, thus achieving domain invariance.
* **Training Details:** The model was trained for 3 epochs using both English and Telugu data with a balanced batch sampler. Mixed-precision training (`torch.cuda.amp`) was used for efficiency. The total loss optimized was a combination of task loss and domain loss, weighted by a factor `lambda_`.
* **Results (Logged During Training After Epoch 3):**
    * **Training:** Task Accuracy: 98.77%
    * **Validation (Overall):** Task Accuracy: 96.56%
    * **Validation (Telugu Only):** Task Accuracy: 93.22%
    *(Note: The Telugu validation accuracy reported during training might differ from detailed evaluation due to potential logging nuances.)*

* **Detailed Evaluation Results (Epoch 3 Model - Separate Script):**
    * **Validation (Overall - 56,668 samples):**
        * Accuracy: 0.9864
        * Weighted Precision: 0.9864
        * Weighted Recall: 0.9864
        * Weighted F1-Score: 0.9864
    * **Validation (English Only - 46,668 samples):**
        * Accuracy: 0.9869
        * Weighted Precision: 0.9869
        * Weighted Recall: 0.9869
        * Weighted F1-Score: 0.9869
    * **Validation (Telugu Only - 10,000 samples):**
        * Accuracy: 0.9841
        * Weighted Precision: 0.9841
        * Weighted Recall: 0.9841
        * Weighted F1-Score: 0.9841

---

## Evaluation Confusion Matrices

### Overall Validation Set

![confusion_matrix_overall_validation](https://github.com/user-attachments/assets/2286d7cf-cc82-46e3-bc02-ceff90fc9ac9)

### English Validation Set

![confusion_matrix_english_validation](https://github.com/user-attachments/assets/2770dcd9-bd98-4b5f-bb90-802f7566edea)


### Telugu Validation Set (Main Task!)

![confusion_matrix_telugu_validation](https://github.com/user-attachments/assets/54475a28-f03c-40bf-bbe6-3ea7a4408983)

---

## Inference and Reasoning

The `Traning and Inference Code/Muril-Model-Inference and Reasoning.ipynb` notebook handles the inference process:

1.  Loads the fine-tuned MuRIL DAT model checkpoint.
2.  Takes input Telugu text.
3.  Predicts "Suicidal" or "Non-Suicidal" using the MuRIL model.
4.  Extracts attention weights from the MuRIL model to identify the most important tokens influencing the prediction.
5.  Sends the input text, prediction, and important tokens to the Gemini 1.5 Flash API.
6.  Gemini generates an explanation in English, referencing the key Telugu tokens, detailing *why* the MuRIL model likely made its prediction.
---

### Sample Output Format
```
--- Results for Input Telugu Text ---
Input: నా చివరి రోజు ఇది. ఎవరూ నన్ను ఆపకండి.
Prediction (MuRIL): Suicidal
Identified Important Tokens (MuRIL Focus): ['నా', '.', '##పకం', 'చివరి', 'రోజు']

Explanation (Generated by Gemini based on MuRIL focus):
The model likely classified the text as suicidal because of the phrase "నా చివరి రోజు," meaning "my last day."
The inclusion of  "ఎవరూ నన్ను ఆపకండి,"  ("Don't stop me"), further reinforces this interpretation.
The model's attention to "##పకం" (part of "ఆపకండి") suggests it recognized the assertive and potentially desperate tone of refusal to be stopped.
The combination of "నా చివరి రోజు" and the refusal to be stopped led the model to predict a suicidal intent.
```
---

## Setup and Usage Notes

### Running the Code

1.  **Dataset Generation (Optional):**
    * Open `Traning and Inference Code/Telugu_Synthetic_Dataset translation code.ipynb`.
    * Ensure the input English dataset CSV path is correct (code uses `/kaggle/input/suicide-watch/Suicide_Detection.csv`).
    * Run the notebook cells. This will download the `ai4bharat/indictrans2-en-indic-1B` model and translate the data, saving it (code saves to `/kaggle/working/Translated_Suicide_Detection.csv` change the output path accordingly). *Note: This requires significant computational resources (GPU recommended) and time.*

2.  **Model Training (Optional):**
    * Open `Traning and Inference Code/Muril_Model_Training_code.ipynb`.
    * Update the paths to the English and Telugu datasets if needed (code uses `/kaggle/input/english-ds-binary/BinaryCsv_English_dataset_UTF8.csv` and `/kaggle/input/telugu-ds-binary/Binary_telugu_dataset.csv`, change the paths accordingly).
    * Run the notebook cells. This will download the `google/muril-base-cased` model, train the DAT model for 3 epochs, print validation results, and save model checkpoints (e.g., `model_epoch_3.pth`). *Note: This requires a GPU setup (preferably multi-GPU) and considerable time.*

3.  **Inference and Reasoning:**
    * Open `Traning and Inference Code/Muril-Model-Inference and Reasoning.ipynb`.
    * **API Key:** Ensure your Gemini API key is correctly inserted into the code.
    * **Model Path:** Ensure the `state_dict_path` variable points to your trained MuRIL model checkpoint (e.g., `model_epoch_3.pth`). You might need to download this checkpoint if trained elsewhere. The example uses `/kaggle/input/dat_nlp_model1/pytorch/default/1/model_epoch_3.pth`.
    * Modify the `telugu_text` variable in the example cells to test with your own input.
    * Run the notebook cells to perform predictions and generate explanations.

---

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues to improve the project.

---

## License

[Choose an appropriate license, e.g., MIT, Apache 2.0, or specify if it's private]

---

## Acknowledgements

* The MuRIL model by Google Research.
* The IndicTrans2 model by AI4Bharat.
* The original English Suicide Watch dataset provider on Kaggle "https://www.kaggle.com/datasets/nikhileswarkomati/suicide-watch".
* The authors of the Domain Adversarial Training paper "https://arxiv.org/abs/1505.07818".
* Google for the Gemini API.
