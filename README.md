# n8n

## 1. n8n Cloud

Rendez-vous sur https://n8n.io, créez un compte gratuit.

Tout est déjà hébergé, vous avez 15 jours d'essai et la possibilité exporter vos workflow.

## 2. Docker local

Si vous avez docker sur votre poste : Commande à lancer dans un terminal :

```shell
docker run -it --rm \
  -p 5678:5678 \
  -e N8N_SECURE_COOKIE=false \
  -e N8N_PERSONALIZATION_ENABLED=false \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
    -e N8N_BASIC_AUTH_PASSWORD=admin \
  n8nio/n8n:latest
```

Puis ouvrez http://localhost:5678.
