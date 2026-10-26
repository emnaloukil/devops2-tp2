

## Q1. Quelles variables d'environnement Strapi attend-il pour se connecter à postgres ? Quelle valeur donner à `DATABASE_HOST`, et pourquoi pas `localhost` ?

D'après le fichier `backend/config/database.js` fourni, Strapi lit les variables suivantes :

- `DATABASE_URL` : chaîne de connexion complète, si elle est utilisée.
- `DATABASE_HOST` : adresse du serveur postgres (valeur par défaut : `localhost`).
- `DATABASE_PORT` : port postgres (valeur par défaut : `5432`).
- `DATABASE_NAME` : nom de la base (valeur par défaut : `strapi`).
- `DATABASE_USERNAME` : utilisateur de la base (valeur par défaut : `strapi`).
- `DATABASE_PASSWORD` : mot de passe (valeur par défaut dans l'extrait : `strapi`).
- `DATABASE_SSL` : activation de SSL (valeur par défaut : `false`).
- `DATABASE_SCHEMA` : schéma postgres (valeur par défaut : `public`).


**Dans le Docker Compose de l'exercice, il faut mettre `DATABASE_HOST=postgres`**, car `postgres` est le nom du service de base de données sur le réseau Docker commun.
Il ne faut pas utiliser `localhost` depuis le conteneur Strapi : dans ce conteneur, `localhost` désigne **Strapi lui-même**, pas le conteneur postgres. donc si on met localhost il va cherché postgres dans lui meme

##  À quoi sert `backend/.env.example` ? Pourquoi ne jamais commiter le vrai fichier `.env` ?

Le fichier **`.env.example`indique quelles variables doivent être utilisé. Il ne doit pas être commité, car publier ce fichier dans Git pourrait exposer des informations sensibles. 

## Q2. Quelle différence entre docker compose down et docker compose down -v ?
La commande docker compose down permet d'arrêter et de supprimer les conteneurs, sans supprimer les volumes nommés.
La commande docker compose down -v effectue la même opération, mais supprime également les volumes nommés déclarés dans le fichier Compose ainsi que les volumes anonymes attachés aux conteneurs.


##  Q4. Dans le healthcheck, pourquoi écrit-on $${POSTGRES_USER} avec deux $ ? 

On utilise $$ pour empêcher Docker Compose de remplacer la variable et permettre au shell du conteneur de la remplacer par sa valeur depuis son propre environnement .

## Q5. Dans servers.json, pourquoi "Host": "postgres" ? 
On utilise postgres car c'est le nom du service  défini dans compose.yaml
## Q5. Que signifie :ro dans le montage ? 
Il s’agit d’un bind mount qui permet de monter le fichier servers.json de l’hôte dans le conteneur pgAdmin. L’option :ro (read-only) permet au conteneur de lire ce fichier sans pouvoir le modifier.

## Que veut dire ${PGADMIN_PORT:-5050} ? ?
utilise la valeur de la variable d'environnement PGADMIN_PORT. Si elle est absente ou vide, il utilise 5050 comme port par défaut.

## Q6. Pourquoi copier package*.json et lancer npm ci avant de copier le reste du code ? ?
pour accélérer les builds en évitant de réinstaller les dépendances à chaque modification du code.

Après modification d'un fichier dans backend/config/ et relancement du build :
 COPY . . et RUN npm run build 

## Q7. À quoi servent USER node et --chown=node:node?

user node c est pour executer l image avec utilisateur node au lieu de administrateur
--chown=node:node : pattribuer la propriété des fichiers copiés à l'utilisateur node et au groupe node.


## Q8. Que fait depends_on: condition: service_healthy ? Que se passerait-il sans ?

c'est verifier que l etat du postgres est healthy avant de demarrer strapi si nn il démarre meme quand postgres rencontre des erreurs

## Q9. Retrouvez la table articles dans pgAdmin. Combien de lignes pour un article publié  ?

il ya deux lignes pour chaque article une version draft et une version publiée

## Q10. Pourquoi STRAPI_URL=http://localhost:1337 ne fonctionnerait-il pas dans le conteneur ?
localhost désigne le conteneur Next.js lui-même.  donc il chercher strapi dans lui meme

## Q11. Combien d'étapes (FROM) a ce Dockerfile ? Qu'est-ce qui se retrouve dans l'image finale ? Retirez temporairement output: "standalone" et relancez le build : que se passe-t-il, et pourquoi ? Notez la taille de l'image finale (docker images tp2/frontend).

Le Dockerfile comporte trois étapes (deps, builder, runner)
L'image finale contient uniquement des fichiers necessaire a partir du résultat du etape build

Sans l'option output: "standalone", Next.js ne génère plus le dossier .next/standalone. Or, le Dockerfile tente de copier ce dossier dans l'image finale avec COPY --from=builder. Le build Docker échoue donc.

la taille de l'image est 85.6MB

## Q12 Pourquoi strapi est-il dans les deux réseaux ? Quel est l'intérêt de ne publier aucun port pour postgres ?

strapi appartient a les deux reseaux pour qu'il pouvoir communiquer avec frontend dans reseau fronted et avec postgres dans le reseau backend

Ne pas publier le port de PostgreSQL permet de renforcer la sécurité en évitant son accès direct depuis l'extérieur de Docker.

1.Protéger les mots de passe : ne pas mettre les mots de passe dans le code ou sur GitHub. Utiliser un gestionnaire de secrets.
2. Activer HTTPS : chiffrer les communications entre les utilisateurs et le site pour protéger les données échangées.

## Persistance : docker compose down puis docker compose up -d. Vos articles sont-ils toujours là ? Et après docker compose down -v ?

APRES docker compose down -v le volume se supprime donc il ya plus d article au contraire du docker compose -d

## docker compose stop strapi, rechargez http://localhost:3000. Que s'affiche-t-il ? Relancez avec docker compose start strapi.

Impossible de joindre Strapi : TypeError: fetch failed

##  Que fait restart: unless-stopped ? Pourquoi strapi ne redémarre-t-il pas tout seul après docker compose stop strapi (ou docker compose kill strapi) ? Dans quel cas Docker le redémarre-t-il automatiquement ?

restart: unless-stopped le container se redemarre autamatiquement sauf dans les cas volantaire. 
Après docker compose stop strapi  Docker ne redémarre pas Strapi car l'arrêt a été fait par l utilisateur.  il le redémarre automatiquement en cas d'arrêt inattendu.