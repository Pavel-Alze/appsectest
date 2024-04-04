 # appsectest

Тестовое задание на стажировку AppSecCloudCamp

1. Вопросы для разогрева
Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались?
Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было?
Если у вас был опыт поиска уязвимостей, расскажите, как это было?
Почему вы хотите участвовать в стажировке?


2. Security code review

Часть 1. Security code review: GO

Требуется провести анализ кода на GO с точки зрения безопасности и подготовить отчет по следующим пунктам:

1)Какие уязвимости присутствуют в этом фрагменте кода?

2)Указать строки, в которых присутствуют уязвимости.

3)К каким последствиям может привести эксплуатация найденных уязвимостей злоумышленником?

4)Описать способы исправления уязвимостей.

5)Если уязвимость можно исправить несколькими способами, необходимо перечислить их, выбрать лучший по вашему мнению и аргументировать свой выбор.

#Ответ
В данном коде мною были обнаружены следующие уязвимости:
    1. SQL-injection >>  query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)
    Здесь совершается запрос к БД с использованием строки запроса, которая никак не обрабатывается/проверяется. Данная уязвимость может позволить злоумышленнику узнать конфиденциальные/полезные данные из БД, либо провзаимодействовать со структурой БД.
    Есть несколько способов избавиться от этой уязвимости на стороне сервера: использование ORM и Экранирование. Встроенный механизм ORM, который преобразует пользовательский запрос в запрос к бд, не позволит провести атаку. Экранирование символов, позволит избежать атаки на этапе проверки запроса. Можно сделать проверку на наличие определённых символов ( ' ) и фраз ( UNION ) и другие.
Для данного кода я бы предложил использовать вариант с экранирование, потому как для небольшой, тривиальной задачи по выводу продуктов использование ORM будет излишним.


package main

import (
    "database/sql"
    "fmt"
    "log"
    "net/http"
    "github.com/go-sql-driver/mysql"
)

var db *sql.DB
var err error

func initDB() {
    db, err = sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        log.Fatal(err)
    }

err = db.Ping()
if err != nil {
    log.Fatal(err)
    }
}

func searchHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        http.Error(w, "Method is not supported.", http.StatusNotFound)
        return
    }

searchQuery := r.URL.Query().Get("query")
if searchQuery == "" {
    http.Error(w, "Query parameter is missing", http.StatusBadRequest)
    return
}

query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)
rows, err := db.Query(query)
if err != nil {
    http.Error(w, "Query failed", http.StatusInternalServerError)
    log.Println(err)
    return
}
defer rows.Close()

var products []string
for rows.Next() {
    var name string
    err := rows.Scan(&name)
    if err != nil {
        log.Fatal(err)
    }
    products = append(products, name)
}

fmt.Fprintf(w, "Found products: %v\n", products)
}

func main() {
    initDB()
    defer db.Close()

http.HandleFunc("/search", searchHandler)
fmt.Println("Server is running")
log.Fatal(http.ListenAndServe(":8080", nil))
}


Часть 2: Security code review: Python
Требуется определить тип уязвимости в примерах кода на Python и ответить на следующие вопросы:

Указать строки, в которых присутствуют уязвимости.
К каким последствиям может привести эксплуатация данных уязвимостей злоумышленником?
Описать способы исправления уязвимостей.
Если уязвимость можно исправить несколькими способами, необходимо перечислить их, выбрать лучший по вашему мнению и аргументировать свой выбор.
Пример №2.1

from flask import Flask, request
from jinja2 import Template

app = Flask(name)

@app.route("/page")
def page():
    name = request.values.get('name')
    age = request.values.get('age', 'unknown')
    output = Template('Hello ' + name + '! Your age is ' + age + '.').render()
return output

if name == "main":
    app.run(debug=True)
Пример №2.2

from flask import Flask, request
import subprocess

app = Flask(name)

@app.route("/dns")
def dns_lookup():
    hostname = request.values.get('hostname')
    cmd = 'nslookup ' + hostname
    output = subprocess.check_output(cmd, shell=True, text=True)
return output
if name == "main":
    app.run(debug=True)
