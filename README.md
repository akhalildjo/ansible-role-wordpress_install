# Documentation de déploiement automatique WordPress avec Ansible et Docker

## 📌 Objectif

Ce projet automatise le déploiement complet de WordPress avec Apache, PHP et MariaDB sur deux conteneurs Docker distincts (Ubuntu et Rocky Linux), en utilisant Ansible comme outil d'orchestration.

## 🧱 Arborescence du projet

```bash
│       ├── defaults
│       │   └── main.yml                  # Valeurs par défaut (ex: ports, users)
│       ├── handlers
│       │   └── main.yml                  # Reload Apache
│       ├── tasks                         # Découpe logique des étapes
│       │   ├── apache_config.yml         # Configuration Apache + vhost
│       │   ├── install_packages.yml      # Installation de Apache, PHP, MariaDB, etc.
│       │   ├── main.yml                  # Inclusion des sous-tâches
│       │   ├── mysql_config.yml          # Création DB, user, sécurisation
│       │   ├── mysql_repair_root.yml     # Réinitialisation mot de passe root si besoin
│       │   └── wordpress_setup.yml       # Téléchargement, permissions et config WP
│       ├── templates
│       │   ├── wordpress.conf.j2         # Template du virtualhost Apache
│       │   └── wp-config.php.j2          # Fichier de config WordPress
│       └── vars
│           └── main.yml                  # Variables sensibles ou de surcharges
```

## ⚙️ Fonctionnement global

Ce role fait la chose suivante:

3. **Ansible** : Configure automatiquement Apache, MariaDB et WordPress pour un environnement ubuntu et Rocky.

## 🚀 Étapes d'utilisation

### 1. Prérequis

Avoir le requirements.yml suivant :

```yaml
roles:
  - name: akhalildjo.wordpress_install
```

Executer la commande suivante pour installer les dépendances :

```bash
ansible-galaxy install -r requirements.yml
```

> Exemple de playbook : 
```yaml
- name: Déploiement WordPress conteneurisé
  hosts: wordpress
  become: true
  roles:
    - role: akhalildjo.wordpress_install
```

### 2. Déploiement

Lancer la commande suivante pour déployer les conteneurs :

```bash
ansible-playbook -i inventory.yml site_galaxy_role.yml -vv
```


### 2. Vérifier l’installation

* Ouvrir [http://localhost:8080](http://localhost:8080) pour Ubuntu
* Ouvrir [http://localhost:8081](http://localhost:8081) pour Rocky

Vous devriez voir l’assistant de configuration WordPress (choix de langue). pour Ubuntu et Rocky

Si les pages ne s'affichent pas, mercie de passer par le repo de développement suivant : https://github.com/akhalildjo/ansible-wp-ak_djoghlal/tree/main et de suivre la documentation.

## 🧩 Variables personnalisables

Définies dans `roles/wordpress_install/vars/main.yml` :

```yaml
wordpress_db_name: wordpress
wordpress_db_user: example
wordpress_db_password: examplePW
mysql_root_password: examplerootPW
wordpress_web_dir: /var/www/html
```

## 🧩 Explication détaillée du rôle `wordpress_install`

Ce rôle est découpé en plusieurs fichiers pour plus de lisibilité et de réutilisabilité :

### `install_packages.yml`

Installe les paquets nécessaires pour chaque distribution. Les modules conditionnels (`apt` ou `dnf`) sont utilisés en fonction de la famille de l’OS.

### `mysql_config.yml`

* Sécurisation de MariaDB (suppression des utilisateurs anonymes et de la base `test`)
* Définition du mot de passe root
* Création de la base `wordpress`
* Création de l’utilisateur `example`
* Attribution des droits nécessaires
* Détection automatique du bon socket (`/var/run/mysqld/mysqld.sock` ou `/var/lib/mysql/mysql.sock`)

### `wordpress_setup.yml`

* Téléchargement de WordPress depuis `wordpress.org`
* Décompression dans le répertoire défini (`wordpress_web_dir`)
* Création du fichier `wp-config.php` depuis un template Jinja2 dynamique
* Application des bons droits sur les fichiers pour `www-data` (Debian) ou `apache` (RedHat)
* Nettoyage de la page par défaut Apache

### `apache_config.yml`

* Déploiement du fichier virtualhost depuis un template (`wordpress.conf.j2`)
* Activation du site et du module `rewrite` sur Debian
* Configuration manuelle du `ServerName` sur Rocky
* Gestion conditionnelle du redémarrage Apache :

  * `service apache2 reload` sur Debian
  * message informatif sur Rocky (Apache tourne en arrière-plan dans le conteneur, pas de reload possible)

### `handlers/main.yml`

Déclenche le redémarrage du service Apache si un fichier de configuration est modifié (template ou copy).

## 👨‍🎓 Auteur

**Ahmed-Khalil DJOGHLAL** — Évaluation DevOps : Déploiement automatisé d'applications Web avec Ansible et Docker.

---
