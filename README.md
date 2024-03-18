from flask import Flask, jsonify, request, make_response
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(_walletguard__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///fintech.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    password_hash = db.Column(db.String(100), nullable=False)
    balance = db.Column(db.Float, default=0)

@app.route('/register', methods=['POST'])
def register():
    data = request.json
    username = data.get('username')
    password = data.get('password')
    if not username or not password:
        return make_response(jsonify({'error': 'Username and password are required'}), 400)
    if User.query.filter_by(username=username).first():
        return make_response(jsonify({'error': 'Username already exists'}), 400)
    password_hash = generate_password_hash(password)
    new_user = User(username=username, password_hash=password_hash)
    db.session.add(new_user)
    db.session.commit()
    return jsonify({'message': 'User registered successfully'})

@app.route('/login', methods=['POST'])
def login():
    data = request.json
    username = data.get('username')
    password = data.get('password')
    user = User.query.filter_by(username=username).first()
    if not user or not check_password_hash(user.password_hash, password):
        return make_response(jsonify({'error': 'Invalid username or password'}), 401)
    # Here you can generate a JWT token for authentication
    return jsonify({'message': 'Login successful'})

@app.route('/balance', methods=['GET'])
def get_balance():
    # Assuming you have implemented JWT token authentication
    # You can extract user information from the token and retrieve balance
    # For demonstration purposes, let's assume the user is authenticated and the balance is fetched
    # Replace this logic with actual JWT authentication and balance retrieval
    username = 'demo_user'
    user = User.query.filter_by(username=username).first()
    if not user:
        return make_response(jsonify({'error': 'User not found'}), 404)
    return jsonify({'balance': user.balance})

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)
