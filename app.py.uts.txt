import re
import pickle
import pandas as pd  
from flask import Flask, request, render_template, redirect, url_for, session
import math

import nltk
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory
from Sastrawi.StopWordRemover.StopWordRemoverFactory import StopWordRemoverFactory
from nltk.tokenize import word_tokenize

nltk.download('punkt')
nltk.download('punkt_tab')

factory_stopword = StopWordRemoverFactory()
stopword_list_func = factory_stopword.get_stop_words()
factory_stemmer = StemmerFactory()
stemmer_func = factory_stemmer.create_stemmer()

def replace_taboo_words(text, kamus_tidak_baku):
    if isinstance(text, str):
        words = text.split()
        replaced_words = []
        for word in words:
            if word in kamus_tidak_baku:
                baku_word = kamus_tidak_baku[word]
                if isinstance(baku_word, str) and all(char.isalpha() for char in baku_word):
                    replaced_words.append(baku_word)
                else:
                    replaced_words.append(word) 
            else:
                replaced_words.append(word)
        replaced_text = ' '.join(replaced_words)
    else:
        replaced_text = ''
    return replaced_text

print("Memuat kamus normalisasi...")
try:
    kamus_path = 'kamuskatabaku.xlsx'
    kamus_data = pd.read_excel(kamus_path)
    kamus_tidak_baku_dict = dict(zip(kamus_data['tidak_baku'], kamus_data['kata_baku']))
    print("✅ Kamus normalisasi berhasil dimuat.")
except Exception as e:
    print(f"Error memuat kamus: {e}")
    kamus_tidak_baku_dict = {} 

model_path = 'svm_tfidf_model3.pkl'
with open(model_path, 'rb') as f:
    tfidf, svm_model = pickle.load(f)
print("✅ Model SVM/TF-IDF berhasil dimuat!")

def preprocess_text(text):
    text = text.lower()
    text = re.sub(r'https?://\S+|www\.\S+', '', text)
    text = re.sub(r'<.*?>', '', text) 
    emoji_pattern = re.compile("["
                           u"\U0001F600-\U0001F64F" u"\U0001F300-\U0001F5FF"
                           u"\U0001F680-\U0001F6FF" u"\U0001F700-\U0001F77F"
                           u"\U0001F780-\U0001F7FF" u"\U0001F800-\U0001F8FF"
                           u"\U0001F900-\U0001F9FF" u"\U0001FA00-\U0001FA6F"
                           u"\U0001FA70-\U0001FAFF" u"\U0001F004-\U0001F0CF"
                           u"\U0001F1E0-\U0001F1FF" "]+", flags=re.UNICODE)
    text = emoji_pattern.sub(r'', text)
    text = re.sub(r'[^\w\s]', '', text) 
    text = re.sub(r'\d', '', text) 
    text = text.strip()

    text = replace_taboo_words(text, kamus_tidak_baku_dict)

    tokens = word_tokenize(text)

    tokens = [word for word in tokens if word not in stopword_list_func]

    text_to_stem = ' '.join(tokens)
    stemmed_text = stemmer_func.stem(text_to_stem)
    
    return stemmed_text

app = Flask(__name__)
app.config['SECRET_KEY'] = 'MLnova100'

@app.route('/')
def home():
    history_list = session.get('history', [])
    prediction_result = session.pop('last_prediction', None)
    original_text_submitted = session.pop('last_original_text', None)
    confidence_result = session.pop('last_confidence', None)
    
    return render_template('index.html',
                           history=history_list,
                           prediction=prediction_result,
                           original_text=original_text_submitted,
                           confidence=confidence_result)
    
@app.route('/predict', methods=['POST'])
def predict():
    if request.method == 'POST':
        text = request.form['text']
        
        cleaned_text = preprocess_text(text) 
        vector = tfidf.transform([cleaned_text])
        
        prediction = svm_model.predict(vector)[0]
        score = svm_model.decision_function(vector)[0]
        probability = 1 / (1 + math.exp(-score))
        
        if prediction == 'Positive':
            confidence_percent = probability * 100
        else: 
            confidence_percent = (1 - probability) * 100
        
        if 'history' not in session:
            session['history'] = []

        session['history'].insert(0, {
            'text': text,
            'prediction': prediction,
            'confidence': confidence_percent
        })
        
        if len(session['history']) > 10:
            session['history'].pop() 
            
        session['last_prediction'] = prediction
        session['last_original_text'] = text
        session['last_confidence'] = confidence_percent
            
        session.modified = True 
        
        return redirect(url_for('home'))

@app.route('/clear')
def clear_history():
    session.pop('history', None) 
    return redirect(url_for('home'))

if __name__ == '__main__':
    app.run(debug=True)