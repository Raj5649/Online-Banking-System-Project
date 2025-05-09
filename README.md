from flask import Flask, render_template, request, flash
import json
import os.path

app = Flask(__name__)
app.secret_key = 'your_secret_key_here'

acc_num_global = {}

@app.route('/')
def home():
    return render_template('home.html')

@app.route('/new-user')
def new_user():
    return render_template('new_user.html')

@app.route('/existing-user')
def existing_user():
    return render_template('existing_user.html')

@app.route('/customer-details', methods=['GET', 'POST'])
def customer_details():
    if request.method == 'POST':
        login = {}
        if os.path.exists('login.json'):
            with open('login.json') as login_file:
                login = json.load(login_file)
        if request.form['type'] == 'new':
            login[request.form['name']] = request.form['password']
            with open('login.json', 'w') as login_file:
                json.dump(login, login_file)
        if request.form['type'] == 'existing':
            if request.form['name'] not in login or login[request.form['name']] != request.form['password']:
                flash('Access Denied!!')
                return render_template('existing_user.html')
        return render_template('customer_details.html', name=request.form['name'])
    return render_template('home.html')

@app.route('/new-customer')
def new_customer():
    return render_template('new_customer.html')

@app.route('/existing-customer')
def existing_customer():
    return render_template('existing_customer.html')

@app.route('/transaction', methods=['GET', 'POST'])
def transaction():
    global acc_num_global
    if request.method == 'POST':
        customer = {}
        if os.path.exists('customer.json'):
            with open('customer.json') as customer_file:
                customer = json.load(customer_file)
        if request.form['type'] == 'new':
            customer[request.form['acc_num']] = {
                'name': request.form['name'],
                'number': request.form['acc_num'],
                'balance': request.form['balance']
            }
            with open('customer.json', 'w') as customer_file:
                json.dump(customer, customer_file)
        if request.form['type'] == 'existing':
            if request.form['acc_num'] not in customer:
                flash('Access Denied!!')
                return render_template('existing_customer.html')
        acc_num_global = request.form['acc_num']
        return render_template('transaction.html',
                               name=customer[acc_num_global]['name'],
                               number=customer[acc_num_global]['number'],
                               balance=customer[acc_num_global]['balance'])
    return render_template('home.html')
