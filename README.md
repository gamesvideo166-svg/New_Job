# New_Job
Gramin job
from flask import Flask, render_template, request, jsonify
import json
import os

app = Flask(__name__)

DB_FILE = 'labour_db.json'

# अगर डेटाबेस फाइल नहीं है, तो नई बनाओ
if not os.path.exists(DB_FILE):
    with open(DB_FILE, 'w') as f:
        json.dump([], f)

def get_all_labours():
    with open(DB_FILE, 'r') as f:
        return json.load(f)

def save_labour(data):
    labours = get_all_labours()
    labours.append(data)
    with open(DB_FILE, 'w') as f:
        json.dump(labours, f, indent=4)

@app.route('/')
def home():
    return render_template('index.html')

# 1. नया मजदूर रजिस्टर करने के लिए API
@app.route('/register', methods=['POST'])
def register():
    data = request.json
    name = data.get('name')
    phone = data.get('phone')
    skill = data.get('skill').lower()
    location = data.get('location').lower()

    if not (name and phone and skill and location):
        return jsonify({"status": "error", "message": "कृपया सभी जानकारी भरें!"})

    new_entry = {"name": name, "phone": phone, "skill": skill, "location": location}
    save_labour(new_entry)
    return jsonify({"status": "success", "message": "आपका नाम और नंबर सफलतापूर्वक दर्ज हो गया है!"})

# 2. ठेकेदार द्वारा मजदूर खोजने के लिए API
@app.route('/search', methods=['POST'])
def search():
    search_data = request.json
    req_skill = search_data.get('skill', '').lower()
    req_location = search_data.get('location', '').lower()

    all_labours = get_all_labours()
    results = []

    for labour in all_labours:
        if req_skill in labour['skill'] and req_location in labour['location']:
            results.append(labour)

    return jsonify({"status": "success", "results": results})

if __name__ == '__main__':
    import os
    port = int(os.environ.get("PORT", 5000))
    app.run(host='0.0.0.0', port=port)
