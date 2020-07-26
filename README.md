# SQL-Injection-Demo
import sqlite3
import hashlib
from json import JSONDecodeError
#app.secret_key = 'hackthissites'

# Below are SQL injection statements
# users?rank=user' OR '1'='1
# users?rank=user'UNION ALL SELECT*FROM SSN--
# users?rank=admin
# users?rank=admin
# login?username=admin&password=wrong_pass
# login?username=admin&password=l33t
# login?username=admin&password=l33t
from flask import Flask, request, render_template, jsonify

app = Flask(__name__)


def connect():
    conn = sqlite3.connect(':memory:', check_same_thread=False)
    c = conn.cursor()
    c.execute("CREATE TABLE users (username TEXT, password TEXT, rank TEXT)")
    c.execute("INSERT INTO users VALUES ('dsam12', 'samd1111', 'admin')")
    c.execute("INSERT INTO users VALUES ('sophia9', 'badpass2', 'user')")
    c.execute("INSERT INTO users VALUES ('matt29', 'pass129', 'customer')")

    c.execute("CREATE TABLE SSN(user_id INTEGER, name TEXT)")
    c.execute("INSERT INTO SSN VALUES (1, 'David Hack')")
    c.execute("INSERT INTO SSN VALUES (2, 'Sophia Crack')")
    c.execute("INSERT INTO SSN VALUES (3, 'Matt Pentest')")

    conn.commit()
    return conn


CONNECTION = connect()

# login endpoint
@app.route("/login")
def login():
    username = request.args.get('username', '')
    password = request.args.get('password', '')
    md5 = hashlib.new('md5', password.encode('utf-8'))
    password = md5.hexdigest()
    c = CONNECTION.cursor()
    c.execute("SELECT * FROM users WHERE username = ? and password = ?", (username, password))
    data = c.fetchone()
    if data is None:
        return 'Incorrect username and password.'
    else:
        return 'Welcome %s! Your rank is %s.' % (username, data[2])


# users endpoint
@app.route("/users")
def list_users():
    rank = request.args.get('rank', '')
    if rank == 'admin':
        return "Can't list admins!"
    c = CONNECTION.cursor()
    c.execute("SELECT username, rank FROM users WHERE rank = '{0}'".format(rank))
    data = c.fetchall()
    return str(data)


if __name__ == '__main__':
    app.run(host='192.168.0.27')
