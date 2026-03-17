# 1. Récupérer les nouveaux tags
git fetch upstream --tags
git push origin --tags

# 2. Arrêter Docker
docker compose down

# 3. Créer nouvelle branche
git checkout v25.28
git checkout -b v25.28-dev

# 4. Merger vos configs
git merge v25.27-dev

# 5. Rebuild et migrate
docker compose build --no-cache
docker compose up -d
docker compose run app python manage.py migrate
docker compose run app python manage.py collectstatic --noinput