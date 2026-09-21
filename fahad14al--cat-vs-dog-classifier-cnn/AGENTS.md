# Copilot Instructions for Cat-vs-Dog-classifier-CNN

## Project Overview
This project is a simple image classifier that distinguishes between cats and dogs using a Convolutional Neural Network (CNN) built with TensorFlow/Keras. The main user interface is a Streamlit app (`app.py`) that loads a pre-trained model (`cats_dogs_model.h5`) and predicts the class of an uploaded image.

## Key Components
- **`app.py`**: Streamlit web app for image upload and prediction. Loads the model once using `@st.cache_resource` for efficiency.
- **`cats_vs_dogs.ipynb`**: Jupyter notebook for model training, preprocessing, and evaluation. Defines the CNN, trains on the TensorFlow Datasets `cats_vs_dogs`, and saves the model.
- **`cats_dogs_model.h5`**: Saved Keras model (binary classifier, sigmoid output).
- **`myenv/`**: Python virtual environment with all dependencies (TensorFlow, Streamlit, etc.).

## Developer Workflows
- **Run the app**: Activate the virtual environment and launch Streamlit:
  ```powershell
  .\myenv\Scripts\activate
  streamlit run app.py
  ```
- **Train or retrain the model**: Use the notebook (`cats_vs_dogs.ipynb`). Save the model as `cats_dogs_model.h5` for the app to use.
- **Dependencies**: All required packages are installed in `myenv`. Use `pip` inside the environment for new packages.

## Project-Specific Patterns
- **Image Preprocessing**: Images are resized to 128x128 and normalized to [0,1].
- **Model Output**: The model outputs a single sigmoid value; threshold at 0.5 for class (Dog if >0.5, else Cat).
- **Class Names**: `['Cat', 'Dog']` (index 0 = Cat, 1 = Dog).
- **Caching**: Use `@st.cache_resource` for model loading in Streamlit.

## Conventions & Tips
- Keep model input size and preprocessing consistent between notebook and app.
- Update `cats_dogs_model.h5` after retraining for changes to take effect in the app.
- No custom build/test scripts; use Jupyter for experimentation and Streamlit for serving.
- No additional configuration files (no requirements.txt, all dependencies in `myenv`).

## Example: Adding a New Model
1. Retrain in the notebook, save as `cats_dogs_model.h5`.
2. Restart the Streamlit app to load the new model.

## References
- Main app: `app.py`
- Training workflow: `cats_vs_dogs.ipynb`
- Model file: `cats_dogs_model.h5`

---
For questions or unclear conventions, check both `app.py` and `cats_vs_dogs.ipynb` for the latest patterns.

---
> Source: [fahad14al/Cat-vs-Dog-classifier-CNN](https://github.com/fahad14al/Cat-vs-Dog-classifier-CNN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-21 -->
