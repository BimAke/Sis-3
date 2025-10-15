# Sis-3
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip git ufw mysql-server
pip install flask requests mysql-connector-python

sudo systemctl start mysql
sudo systemctl enable mysql

sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 5000/tcp
sudo ufw allow 3306/tcp
sudо ufw status

mkdir ~/flight-booking
cd ~/flight-booking

cat << 'EOF' > database_setup.py
import mysql.connector

connection = mysql.connector.connect(
    host="localhost",
    user="roоt",
    password="",
)

cursor = connection.cursor()
cursor.execute("CREATE DATABASE IF NOT EXISTS flight_booking_db")
cursor.execute("USE flight_booking_db")
cursor.execute("""
CREATE TABLE IF NOT EXISTS flights (
    id INT AUTO_INCREMENT PRIMARY KEY,
    flight_no VARCHAR(10),
    origin VARCHAR(50),
    destination VARCHAR(50),
    price DECIMAL(10, 2)
)
""")
cursor.execute("""
INSERT INTO flights (flight_no, origin, destination, price)
VALUES
('KC101', 'Almaty', 'Seoul', 420),
('SU245', 'Almaty', 'Moscow', 180),
('HY555', 'Almaty', 'Tashkent', 95)
""")
connection.cоmmit()
connection.close()
EOF

python3 database_setup.py

cat << 'EOF' > app.py
from flask import Flask, jsonify
import mysql.connector

app = Flask(__name__)

def get_flights():
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="",
        database="flight_booking_db"
    )
    cursor = connection.cursor(dictionary=True)
    cursor.execute("SELECT * FROM flights")
    datа = cursor.fetchall()
    connection.close()
    return data

@app.route('/')
def index():
    return jsonify({"message": "Flight Booking API running successfully"})

@app.route('/flights')
def flights():
    return jsonify(get_flights())

@app.route('/status')
def status():
    return jsonify({"status": "running", "database": "connected"})

if __namе__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

python3 app.py
