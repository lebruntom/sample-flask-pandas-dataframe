# Flask Pandas

## Set up

### se connecter en ssh au serveur

### on se connecte à la machine google
```bash
jenkins.sh
```
### on clone le projet dans la vm
```bash
git clone https://github.com/lebruntom/sample-flask-pandas-dataframe.git
```

### on se deplace dans le projet
```bash
cd sample-flask-pandas-dataframe/
```

### on active l'environnement virtuel
```bash
source ../docker-aston-poec/venv/bin/activate
```

### on installe les dependances
```bash
pip3 install -r requirements.txt
```

### on créé la base de données flask
```bash
flask shell
>>> from app import db  # import SqlAlchemy interface
>>> db.create_all()     # create SQLite database and Data table
>>> quit()  
```
### on rempli la base de données
```bash
flask load-data titanic-min.csv
```

### on set les variables d'environnement
```bash
set FLASK_APP=app.py 
set FLASK_ENV=development
```

### on lance le projet sur le port 31201
```bash
flask run --host=0.0.0.0 --port=31201
```

### on peut maintenant s'y connecter à l'adresse
http://34.163.167.253:31201/


## Docker

### Création du Dockerfile

### création de l'image
```bash
docker build -t flask-panda .
```
### Connection à Docker Hub 
```bash
docker login -u omtee
```

### Tag l'image avec le nom d'utilisateur Docker Hub :
```bash
docker image tag flask-panda:latest omtee/flask-panda:latest
```
### On pousse l'image vers Docker Hub
```bash
docker push omtee/flask-panda:latest
```
### On execute le container
```bash
docker run -d --name flask-panda -p 31201:31201 flask-panda
```

### on peut maintenant s'y connecter à l'adresse
http://34.163.167.253:31201/


