lab5:Ce projet FastAPI permet de créer une API pour gérer des questions à choix multiples stockées dans une base de données PostgreSQL. Deux modèles Pydantic (ChoiceBase et QuestionBase) sont utilisés pour valider les données entrantes. SQLAlchemy est utilisé pour modéliser les tables questions et choices et interagir avec la base. L'application propose des endpoints pour ajouter une question avec ses choix, lire une question par ID, et lire ses choix associés. La connexion à PostgreSQL est gérée proprement avec une fonction get_db pour l'injection de dépendances.
🔧 1. Création de l'application FastAPI
Dans le fichier main.py, on initialise une instance de FastAPI :

python
Copier le code
app = FastAPI()
C'est cette instance qui va gérer les routes (endpoints) de l’API.

🧾 2. Création des modèles Pydantic
On crée deux modèles de données avec Pydantic pour valider les données reçues via les requêtes HTTP :

python
Copier le code
class ChoiceBase(BaseModel):
    choice_text: str
    is_correct: bool

class QuestionBase(BaseModel):
    question_text: str
    choices: List[ChoiceBase]
QuestionBase contient l’énoncé de la question et une liste de choix.

ChoiceBase représente chaque choix avec son texte et un booléen indiquant s’il est correct.

🗄️ 3. Connexion à PostgreSQL (database.py)
On configure SQLAlchemy pour se connecter à une base de données PostgreSQL :

python
Copier le code
URL_DATABASE = 'postgresql://USERNAME:PASSWD@localhost:5432/quizApp'
engine = create_engine(URL_DATABASE)
SessionLocal = sessionmaker(...)
Base = declarative_base()
engine se connecte à la base.

SessionLocal permet de gérer les sessions avec la base.

Base est la classe de base pour déclarer les modèles SQLAlchemy.

🧱 4. Modélisation des tables (models.py)
Deux tables SQLAlchemy sont définies :

python
Copier le code
class Questions(Base):
    __tablename__ = 'questions'
    id = Column(Integer, primary_key=True)
    question_text = Column(String)

class Choices(Base):
    __tablename__ = 'choices'
    id = Column(Integer, primary_key=True)
    choice_text = Column(String)
    is_correct = Column(Boolean)
    question_id = Column(Integer, ForeignKey("questions.id"))
questions contient l’énoncé de chaque question.

choices contient les choix possibles liés à une question via question_id.

🔌 5. Création des tables dans la base de données
Dans main.py, on ajoute :

python
Copier le code
models.Base.metadata.create_all(bind=engine)
Cela crée les tables dans PostgreSQL si elles n'existent pas encore.

🔄 6. Connexion à la base (get_db)
On crée une fonction get_db() qui retourne une session de base de données. Elle est utilisée avec Depends pour l’injection automatique dans les routes.

python
Copier le code
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

db_dependency = Annotated[Session, Depends(get_db)]
📤 7. Ajout d’une question (POST /questions/)
On crée un endpoint pour ajouter une question et ses choix :

python
Copier le code
@app.post('/questions/')
def create_questions(question: QuestionBase, db: db_dependency):
    ...
La question est d’abord ajoutée.

Ensuite, on ajoute chaque choix lié à cette question.

📥 8. Récupération d'une question (GET /questions/{id})
On ajoute un endpoint pour récupérer une question par son id :

python
Copier le code
@app.get('/questions/{question_id}')
def read_questions(question_id: int, db: db_dependency):
    ...
Si la question n'existe pas, une erreur 404 est renvoyée.

📥 9. Récupération des choix (GET /choices/{question_id})
Ce endpoint permet de récupérer tous les choix d'une question :

python
Copier le code
@app.get('/choices/{question_id}')
def read_choices(question_id: int, db: db_dependency):
    ...
🚀 10. Exécution de l’application
On démarre l’API avec Uvicorn :

bash
Copier le code
uvicorn main:app --reload
Puis on teste les endpoints via Swagger UI accessible à http://127.0.0.1:8000/docs.

✅ Résultat final
L’API permet de :

Ajouter une question avec ses choix.

Récupérer une question spécifique.

Récupérer les choix d'une question.

lab6: Objectif général du projet :
Créer un scraper Python qui récupère les commentaires d’un post "Ask HN: Who is hiring?" sur Hacker News, analyse ces commentaires pour détecter des technologies mentionnées (ex: python, javascript, etc.), puis compte et affiche ces occurrences sous forme de graphique.

🔧 Étapes du projet expliquées en détail :
1. Initialisation du projet
Création d’un fichier scraper.py :

python
Copier le code
def main():
    print('Hello world!')

if __name__ == "__main__":
    main()
Création d’un environnement virtuel :

bash
Copier le code
python -m venv .venv
Activation de l’environnement virtuel :

bash
Copier le code
.\.venv\Scripts\activate
Exécution du fichier :

bash
Copier le code
python scraper.py
2. Récupération du contenu d'une page web
Installer la librairie requests :

bash
Copier le code
pip install requests
Ajouter l’URL de la page à scraper :

python
Copier le code
import requests

def main():
    url = "https://news.ycombinator.com/item?id=42919502"
    response = requests.get(url)
    print(f"Scraping: {url}")
    print(response)

if __name__ == "__main__":
    main()
On vérifie que la requête fonctionne avec le code de réponse 200.

3. Analyse du HTML avec BeautifulSoup
Installer beautifulsoup4 :

bash
Copier le code
pip install beautifulsoup4
Utiliser BeautifulSoup pour parser le HTML :

python
Copier le code
from bs4 import BeautifulSoup

soup = BeautifulSoup(response.content, "html.parser")
elements = soup.find_all(class_="comment")
Cette ligne récupère tous les commentaires du post Hacker News.

4. Extraction des commentaires utiles
Les commentaires qui nous intéressent ont un niveau d’indentation égal à 0, donc :

python
Copier le code
elements = soup.find_all(class_="ind", indent=0)
comments = [e.find_next(class_="comment") for e in elements]
On trouve les commentaires principaux (ceux postés directement en réponse au post initial) — probablement les offres d’emploi.

5. Nettoyage du texte extrait
On extrait le texte brut des balises HTML :

python
Copier le code
comment_text = comment.get_text()
6. Analyse des commentaires pour détecter les technologies
Création d’un dictionnaire de mots-clés (technologies à rechercher) :

python
Copier le code
keywords = {
  "python": 0, "javascript": 0, "typescript": 0, 
  "go": 0, "c#": 0, "java": 0, "rust": 0
}
Pour chaque commentaire :

Convertir en minuscules (.lower())

Découper en mots (.split(" "))

Nettoyer la ponctuation (strip)

Convertir la liste en set (ensemble) pour éviter les doublons

Incrémenter le compteur si le mot-clé est trouvé.

python
Copier le code
words = {w.strip(".,/:;!@") for w in comment_text.split(" ")}
for k in keywords:
    if k in words:
        keywords[k] += 1
7. Visualisation avec matplotlib
Installer matplotlib :

bash
Copier le code
pip install matplotlib
Tracer un graphique en barres :

python
Copier le code
import matplotlib.pyplot as plt

plt.bar(keywords.keys(), keywords.values())
plt.xlabel("Language")
plt.ylabel("# of Mentions")
plt.show()
8. Exporter les dépendances
Pour sauvegarder les bibliothèques utilisées :

bash
Copier le code
pip freeze > requirements.txt
