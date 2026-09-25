# 🏗️ TP4 - Construction multi-étapes avec Alpine

## 🎯 Objectifs

- Utiliser Alpine Linux comme base minimale.
- Installer Python et ses outils uniquement pendant la phase de build.
- Utiliser un environnement virtuel Python (venv) pour séparer les dépendances.
- Créer une image légère, sécurisée et sans outils de build.
- Garder un conteneur fonctionnel, mais plus propre que les précédents.

👉 Ce TP fait suite à [`03-alpine`](../03-alpine).

## 📁 Structure du projet

```bash
04-multistage-build/
├── app/
│   ├── main.py
│   └── requirements.txt
├── .dockerignore
├── Dockerfile
```

## 📝 Dockerfile complet avec multi-stage

Si vous ne connaissez pas encore le langage python alors, je vais vous fournir
le code pour mettre en le multi-stage build.

```dockerfile
# Étape 1 : Étape de build avec outils Python + compilation
FROM alpine:3.18 AS builder

# Installer Python et outils de build
RUN apk add --no-cache python3 py3-pip build-base libffi-dev

# Créer un venv propre
RUN python3 -m venv /venv

# Activer le venv et installer les dépendances
ENV PATH="/venv/bin:$PATH"
WORKDIR /app
COPY app/ /app/
RUN pip install --no-cache-dir -r requirements.txt

# Étape 2 : Image finale minimaliste
FROM alpine:3.18

# Installer uniquement Python minimal pour faire tourner le venv
RUN apk add --no-cache python3

# Copier le venv depuis l'étape précédente
COPY --from=builder /venv /venv
ENV PATH="/venv/bin:$PATH"

# Copier l'application
WORKDIR /app
COPY app/ /app/

# Utilisateur non-root (optionnel)
RUN adduser -D appuser
USER appuser

# Exposer le port de l'API
EXPOSE 8000

# Commande de démarrage
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 🧪 Étapes de test

1️⃣ **Build de l’image** :

```bash
docker build -t fastapi-multistage .
```

🔍 Vérifiez que l’image est bien construite :

```bash
docker images | grep fastapi
fastapi-multistage   latest    c22318eebb18   4 seconds ago    85.6MB
fastapi-alpine       latest    ad9c28e9ae3e   21 minutes ago   116MB
fastapi-optimise     v4        bb2121d01538   31 minutes ago   236MB
fastapi-optimise     v1        d1c77c5394e7   47 minutes ago   256MB
fastapi-optimise     v2        ac5a57858d17   48 minutes ago   236MB
fastapi-optimise     v3        ac5a57858d17   48 minutes ago   236MB
fastapi-auto         latest    7e2e921dad53   53 minutes ago   678MB
```

💡 Vos tailles seront sans doute un peu différentes : elles varient avec les
versions des paquets installés. Comparez surtout les images entre elles.
Si votre Docker utilise le stockage d'images containerd, `docker images`
affiche deux colonnes, `DISK USAGE` et `CONTENT SIZE`, à la place de `SIZE`.
`DISK USAGE` additionne l'image décompressée et ses couches compressées : ce
chiffre est plus élevé que ceux ci-dessus, et ne se compare pas avec eux.

2️⃣ **Run + test de l’API** :

```bash
docker run -d -p 8000:8000 fastapi-multistage
curl http://localhost:8000/health
{"status":"ok"}
```

Youpi ! 🎉

## 🧠 Ce que vous avez appris

Vous avez appris à :

- Créer une image Docker multi-étapes.
- Utiliser un environnement virtuel Python pour isoler les dépendances.
- Réduire la taille de l’image finale en ne gardant que ce qui est nécessaire.
- Utiliser un utilisateur non-root pour des raisons de sécurité.
- Comprendre l'importance de la séparation entre l'étape de build et l'étape d'exécution.
- Utiliser `COPY --from=builder` pour transférer des fichiers entre les étapes.

| Technique                        | Avantage                               |
|----------------------------------|----------------------------------------|
| Multi-stage build                | Construit dans un environnement propre |
| Environnement virtuel (`venv`)   | Permet d’isoler les dépendances        |
| Suppression des outils de build  | Réduit la taille finale                |
| Utilisation d’un utilisateur non-root | Bonne pratique sécurité          |

---

## 🧩 Challenge

- Ajoutez une dépendance compilée (ex: `cryptography`) pour voir l’impact sur la
  taille et l’intérêt du multi-stage.
- Testez un 4ᵉ stage "debug" avec les outils de build conservés (optionnel).

---

Souhaites-tu que je te fournisse aussi le contenu du `README.md` pour le dossier
`05-multistage-build/` ?