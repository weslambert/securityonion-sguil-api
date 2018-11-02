# securityonion-sguil-api
#### Very basic example API for categorizing Sguil alerts in an automated fashion
- Change the key!

TO-DO:
- Make this a real API and not just a concept ;)
- Add in DB auth vs flat file

````
from flask import Flask
from flask_jwt import JWT, jwt_required, current_identity
from werkzeug.security import safe_str_cmp

import subprocess

app = Flask(__name__)

class User(object):
    def __init__(self, id, username, password):
        self.id = id
        self.username = username
        self.password = password

    def __str__(self):
        return "User(id='%s')" % self.id

users = [
    User(1, 'user1', 'abcdefg')
]

username_table = {u.username: u for u in users}
userid_table = {u.id: u for u in users}

def authenticate(username, password):
    user = username_table.get(username, None)
    if user and safe_str_cmp(user.password.encode('utf-8'), password.encode('utf-8')):
        return user

def identity(payload):
    user_id = payload['identity']
    return userid_table.get(user_id, None)

app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'thisismysecret'

jwt = JWT(app, authenticate, identity)


# Example, issue a GET to:
# /sguil/cat/jimbob+1+None+3.7363"
@app.route("/sguil/cat/<username>+<cat_type>+<comment>+<eventid>")
@jwt_required()
def cat(username,cat_type,comment,eventid):
    cmd = ["tclsh", "/var/www/so/squert/.scripts/clicat.tcl", "0", username, cat_type, comment, eventid ]
    p = subprocess.Popen(
                         cmd,
                         stdout = subprocess.PIPE,
                         stderr=subprocess.PIPE,
                         stdin=subprocess.PIPE)
    out,err = p.communicate()
    if out == "0":
        return "Command issued!"
if __name__ == "__main__" :
    app.run(host='0.0.0.0')#, ssl_context='adhoc')
````
