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
 html
 <!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ग्रामीण रोजगार और लेबर खोज पोर्टल</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: Arial, sans-serif; }
        body { background: #f4f6f9; padding: 20px; }
        .container { max-width: 600px; margin: 0 auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        h1, h2 { text-align: center; color: #2c3e50; margin-bottom: 20px; }
        .section { border: 1px solid #ddd; padding: 15px; border-radius: 6px; margin-bottom: 20px; background: #fafafa; }
        input, select { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ccc; border-radius: 4px; font-size: 16px; }
        button { width: 100%; padding: 12px; background: #27ae60; color: white; border: none; border-radius: 4px; font-size: 16px; cursor: pointer; font-weight: bold; }
        button:hover { background: #219150; }
        .labour-card { background: white; border-left: 5px solid #27ae60; padding: 10px; margin: 10px 0; border-radius: 4px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
    </style>
</head>
<body>

<div class="container">
    <h1>💼 ग्रामीण रोजगार पोर्टल</h1>

    <div class="section">
        <h2>मजदूर भाई यहाँ अपना नाम दर्ज करें</h2>
        <input type="text" id="regName" placeholder="आपका पूरा नाम">
        <input type="text" id="regPhone" placeholder="मोबाइल नंबर">
        <input type="text" id="regSkill" placeholder="आपका काम (जैसे: राजमिस्त्री, ड्राइवर, पेंटर)">
        <input type="text" id="regLocation" placeholder="आपके जिले या शहर का नाम">
        <button onclick="registerLabour()">मेरा नंबर सेव करें</button>
    </div>

    <div class="section">
        <h2>ठेकेदार यहाँ से मजदूर खोजें</h2>
        <input type="text" id="searchSkill" placeholder="कौन सा कारीगर चाहिए? (उदा: ड्राइवर)">
        <input type="text" id="searchLocation" placeholder="किस शहर/जिले में चाहिए?">
        <button style="background: #2980b9;" onclick="searchLabour()">मजदूर खोजें</button>
    </div>

    <div id="searchResults"></div>
</div>

<script>
    async function registerLabour() {
        const name = document.getElementById('regName').value;
        const phone = document.getElementById('regPhone').value;
        const skill = document.getElementById('regSkill').value;
        const location = document.getElementById('regLocation').value;

        const response = await fetch('/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, phone, skill, location })
        });
        const data = await response.json();
        alert(data.message);
    }

    async function searchLabour() {
        const skill = document.getElementById('searchSkill').value;
        const location = document.getElementById('searchLocation').value;
        const resultDiv = document.getElementById('searchResults');
        resultDiv.innerHTML = "खोज रहे हैं...";

        const response = await fetch('/search', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ skill, location })
        });
        const data = await response.json();
        resultDiv.innerHTML = "<h2>उपलब्ध कारीगर:</h2>";

        if(data.results.length === 0) {
            resultDiv.innerHTML += "<p>इस इलाके में अभी कोई कारीगर नहीं मिला।</p>";
            return;
        }

        data.results.forEach(labour => {
            resultDiv.innerHTML += `
                <div class="labour-card">
                    <p><strong>नाम:</strong> ${labour.name}</p>
                    <p><strong>काम:</strong> ${labour.skill}</p>
                    <p><strong>शहर:</strong> ${labour.location}</p>
                    <p><strong>फ़ोन नंबर:</strong> <a href="tel:${labour.phone}">${labour.phone}</a></p>
                </div>
            `;
        });
    }
</script>

</body>
</html>
