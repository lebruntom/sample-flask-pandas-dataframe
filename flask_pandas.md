# Projet fil-rouge

## Cahier des charges du fil-rouge
Le société DataMine veut mettre dans une chaîne CI/CD utilisant Jenkins son developpement
de produit informatique qui est, pour le moment, une application Python Flask de type data mining utilisant la librairie Pandas de Python .
Vous devez compléter le projet github existant, en précisant votre demarche pour porter ce projet dans la chaine ci/cd.

**Etapes:**
* Etudier le projet suivant   
  **https://github.com/app-generator/sample-flask-pandas-dataframe.git**  
  Faire un fork de toutes les branches dans votre repo perso github  
  etudiez le projet le fichier README.md, la partie setup.   
  Faire une clone de votre projet forke dans votre directory de travail sur votre laptop et
  dans la vm google.
* Faire fonctionner, en ligne de commande,  le projet dans un shell, precisez les commandes necessaires pour realiser ca dans un fichier **flask_pandas.md**, l'application flask doit etre accessible par l'adresse publique gcp google et par le **port 31201**. Lire correctement le fichier README du projet pour trouver les commandes necessaires. Faire regulierement des git commit, push pour enregister votre projet
* dans le projet, écrire un fichier **Dockerfile** pour conteneuriser cette application (pensez a faire git commit, push)
* en dehors de jenkins, sur votre vm google, creez une image **flask-panda**. Mettre l'image dans le repository docker hub
* en dehors de jenkins, sur votre vm google,  faire une docker run et verifier que l'application est disponible sur le port 31201.
* dans jenkins creez un job **flask-panda-jmeter**, creer un test plan pour verifier si l'application affiche des data.
* chainer les jobs jenkins pour faire un pipeline graphique
* mettre a jour le jenkins plugin de goland pour lancer des build depuis votre IDE goland
* mettre un webhook dans votre projet github, pour demarrer automatiquement votre ci/cd a chaque commit.

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

![Docker.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/docker.png)
![Result.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/result.png)

## Jmeter

### ouvir Jmetter

### Dans test plan créér 4 variables
>Name: IP, Value: "${__P(IP,34.163.167.253)}"
>Name: PORT, Value: "${__P(PORT,31201)}"
> Name: USER, Value: "${__P(USER,5)}"
>Name: DELAY, Value: "${__P(DELAY,60)}"

### ajout de "http request defaults"
> clique droit sur test plan
> add
> Config Element "Http request defaults"
> dans "server Name or ip" mettre ${IP}
> dans "Port number" mettre ${PORT}

### Ajout de thread group
> clique droit sur test plan
> add
> Threads (user)
> Thread group
> dans "number of threads (user)" mettre ${USER}
> dans "Ramp up period (seconds" mettre ${DELAY}
> dans "Loop count" mettre 1

### Ajout http request
>clique droit sur "thread group"
> add 
> sampler
> http request
> dans path mettre /data

### ajout response assertion
> clique droit http request
> add
> assertions
> Response assertion
> dans pattern to test mettre Rows = 0
> cocher dans pattern matching rules not et substring

![Jmeter.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/jmeter.png)

## test pour vérifier si l'application affiche des data
## Jenkins

### Se rendre sur l'url suivante
http://34.163.167.253:32500/

### Se connecter avec 
identifiant : admin
password : 12345678

### Cliquer sur "nouveau item"

### le nommer "flask-panda-jmeter" et choisir "construire un projet free-style"

### Gestion de code source
> selectionnez git 
> mettre l'url https://github.com/lebruntom/sample-flask-pandas-dataframe.git
> en branches to build mettre main

### Etapes du build
> selectionnez Exécuter un script shell
> entrer la ligne 

```bash
jmeter -JUSER=1 -JDELAY=1 -Jjmeter.save.saveservice.output_format=xml -Jjmeter.save.saveservice.response_data.on_error=true -n -t flask-pandas_test_plan.jmx  -l testresult.jtl
```
### premiere action à la suite du build
> selectionnez "Console output (build log) parsing"
> puis selectionnez use project rule et entrez "parserules"

### seconde action à la suite du build
> selectionnez Publish Performance test result report
> dans "Source data files (autodetects format):" entrez "testresult.jtl"

### sauvegarde du job
> cliquez sur sauvegarder

### run le job
> cliquez sur lancer un build

![Job.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/job.png)



## job test

### Se rendre sur l'url suivante
http://34.163.167.253:32500/

### Se connecter avec
identifiant : admin
password : 12345678

### Cliquer sur "nouveau item"

### le nommer "flask-panda-docker-build" et choisir "construire un projet free-style"

### Gestion de code source
> selectionnez git
> mettre l'url https://github.com/lebruntom/sample-flask-pandas-dataframe.git
> en branches to build mettre main

### Ce qui déclenche le build
> selectionnez "GitHub hook trigger for GITScm polling"


### Environnements de Build
> selectionnez "Use secret text(s) or file(s)"
> Username Variable : USERNAME
> Password Variable : PASSWORD
> Ajoutez egalement vos credentials

### Étapes du build
> selectionnez Exécuter un script shell
> entrer la ligne 
```bash
echo '
#docker build -t flask-panda .
#docker run -d --name flask-panda -p 31201:31201 flask-panda

#docker login -u omtee 
#docker image tag flask-panda:latest omtee/flask-pandas:latest
#docker push omtee/flask-panda:latest'
```

### Actions à la suite du build
> selectionnez "Construire d'autres projets (projets en aval)"
> selectionnez le job créé précedement "flask-panda-jmeter"

### sauvegarde du job
> cliquez sur sauvegarder

### run le job
> cliquez sur lancer un build

![Job2.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/job2.png)


### Chainage des jobs jenkins 

### Se rendre sur l'url suivante
http://34.163.167.253:32500/

### Se connecter avec
identifiant : admin
password : 12345678

### Cliquer sur "nouvelle vue"

### la nommer "flask-panda-pipeline" et choisir "build pipeline view"

### Pipeline Flow
> dans "Select Initial Job"
> Selectionnez le job créé à l'instant "flask-panda-docker-build"

### sauvegarde de la vue
> cliquez sur sauvegarder

### vos jobs sont maintenant chainés
![Vue.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/vue.png)

### visualisation dans goland
![Goland.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/goland.png)

## Webwook

### se rendre sur le repo
https://github.com/lebruntom/sample-flask-pandas-dataframe

### cliquer sur settings puis webhooks et add webhook

### payload url
> entrer http://34.163.167.253:32500/github-webhook/

### Content type
application/json

### sauvegarder votre webhook

### Maintenant a chaque commit votre ci/cd se lancera


![Webhook.](https://github.com/lebruntom/sample-flask-pandas-dataframe/blob/main/media/webhook.png)


