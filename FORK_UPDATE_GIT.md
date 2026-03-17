# Étape 1 : Vérifier l'état actuel

# Voir où vous êtes
git status
git branch -v
git remote -v

# Normalement vous devriez voir :
# origin  https://github.com/opengisch/qfieldcloud.git



# Étape 2 : Sauvegarder vos modifications locales

# Stash vos modifications avec un message clair
git stash push -m "Config locale serveur dev - docker-compose et settings"

# Vérifier que c'est bien sauvegardé
git stash list



# Étape 3 : Configurer les remotes correctement

# Renommer le remote actuel en "upstream"
git remote rename origin upstream

# Ajouter VOTRE fork comme "origin"
git remote add origin https://github.com/VOTRE-USERNAME/qfieldcloud.git

# Vérifier la nouvelle configuration
git remote -v
# Devrait afficher :
# origin    https://github.com/VOTRE-USERNAME/qfieldcloud.git
# upstream  https://github.com/opengisch/qfieldcloud.git



# Étape 4 : Synchroniser tous les tags et branches avec upstream

# Récupérer TOUS les tags et branches de upstream
git fetch upstream --tags --prune

# Vérifier que vous avez bien les tags
git tag -l | grep v25
# Devrait afficher : v25.24, v25.27, etc.

# Pousser tous les tags vers VOTRE fork
git push origin --tags

# Pousser aussi la branche master si nécessaire
git push origin master



# Étape 5 : Créer une branche pour vos configurations sur v25.24

# Créer une branche à partir de votre position actuelle (v25.24)
git checkout -b v25.24-dev

# Réappliquer vos modifications
git stash pop

# Vérifier que tout est bon
git status
git diff

# Committer vos configurations
git add docker-compose.yml settings.py
git commit -m "Configuration serveur dev - docker-compose et settings"

# Pousser vers votre fork
git push origin v25.24-dev



# Étape 6 : Fermer Docker avant la mise à jour

# Arrêter tous les conteneurs
docker compose down

# Optionnel : nettoyer les images non utilisées
docker system prune -f



# Étape 7 : Mise à jour vers v25.27

# Checkout le nouveau tag
git checkout v25.27

# Créer une nouvelle branche pour cette version
git checkout -b v25.27-dev

# Merger vos configurations depuis v25.24-dev
git merge v25.24-dev

# Si pas de conflits, continuer
# Si conflits, les résoudre :
git status  # voir les fichiers en conflit
# Éditer les fichiers marqués en conflit
git add .
git commit -m "Merge config dev dans v25.27"

# Pousser vers votre fork
git push origin v25.27-dev


# Étape 8 : Reconstruire et relancer

# Reconstruction complète
docker compose build --no-cache

# Démarrer les services
docker compose up -d

# Attendre quelques secondes que les services démarrent
sleep 10

# Migrations Django
docker compose run app python manage.py migrate

# Collecter les fichiers statiques
docker compose run app python manage.py collectstatic --noinput


# Étape 9 : Vérification

# Vérifier les logs
docker compose logs -f app

# Vérifier que tous les services tournent
docker compose ps

# Tester l'application
curl http://localhost:8000/health  # ou votre URL
```

---

## Résumé de la nouvelle structure
```
upstream (opengisch) :
  master, v25.24, v25.27, v25.28...

origin (votre fork) :
  master
  v25.24-dev  ← vos configs sur v25.24
  v25.27-dev  ← vos configs sur v25.27 (branche active)
  
Local :
  v25.27-dev (checkout actuel)