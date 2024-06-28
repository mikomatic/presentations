[[_TOC_]]

Ce projet utilise le npframework [Marp](https://marp.app/) pour écrire des slides-as-code.

# Usage avec NPM

## Pré-requis

```powershell
npm install
```

La configuration de Marp est disponible dans le fichier `.marprc` (cela contient uniquement une conf pour permettre d'utiliser les images locales dans la conversion `PDF`/`PPTX`)

## Usage Dev

Ci-dessous les différentes commandes pour travailler avec ce projet

```powershell
# Lance le projet en mode DEV - un navigateur avec livereload s'ouvre automagiquement
# Un export PPT et PDF est dispo depuis le navigateur
npm run start-dev
```

# Usage avec docker

## Server mode

```powershell
# Serve current directory in http://localhost:8080/
docker run --rm --init -v ${PWD}:/home/marp/app -e LANG=$LANG -e PUPPETEER_TIMEOUT=0 -p 8080:8080 -p 37717:37717 marpteam/marp-cli -s .
```

## Convert slide deck

```powershell
# HTML
docker run --rm -v ${PWD}:/home/marp/app/ -e LANG=$LANG  marpteam/marp-cli formation-java.md --html
# Powerpoint
docker run --rm -v ${PWD}:/home/marp/app/ -e LANG=$LANG -e PUPPETEER_TIMEOUT=0 marpteam/marp-cli formation-java.md --pptx
```

## Watch mode

```powershell
docker run --rm --init -v ${PWD}:/home/marp/app/ -e LANG = $LANG -p 37717:37717 marpteam/marp-cli -w formation-java.md
```

# Troubleshooting

- La génération PPTX et PDF peuvent prendre très longtemps. Vu que la génération se fait via
  Puppeteer, il est possible que le timeout soit déclenché si la génération prend plus de `30s`. Le
  timeout est désactivable en passant la variable `PUPPETEER_TIMEOUT=0` (en mode docker ou npm)
  [voir la discussion](https://github.com/marp-team/marp/discussions/225)