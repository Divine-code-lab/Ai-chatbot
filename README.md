from flask import Flask, request, jsonify
import openai
import firebase_admin
from firebase_admin import credentials, firestore

# Initialize Firebase
cred = credentials.Certificate("firebase_credentials.json")
firebase_admin.initialize_app(cred)
db = firestore.client()

# Initialize Flask App
app = Flask(__name__)

# OpenAI API Key
openai.api_key = "your_openai_api_key"

def generate_response(user_query):
    """Generate AI response using OpenAI API"""
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "system", "content": "You are an AI chatbot for an e-commerce store."},
                  {"role": "user", "content": user_query}]
    )
    return response["choices"][0]["message"]["content"]

def store_lead(name, email, message):
    """Store lead information in Firestore"""
    db.collection("leads").add({
        "name": name,
        "email": email,
        "message": message
    })

@app.route("/chat", methods=["POST"])
def chat():
    data = request.json
    user_query = data.get("message")
    response = generate_response(user_query)
    return jsonify({"response": response})

@app.route("/collect-lead", methods=["POST"])
def collect_lead():
    data = request.json
    name = data.get("name")
    email = data.get("email")
    message = data.get("message")
    store_lead(name, email, message)
    return jsonify({"message": "Lead collected successfully!"})

if __name__ == "__main__":
    app.run(debug=True)
    
