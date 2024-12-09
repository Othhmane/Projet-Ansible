# Documentation du Projet Ansible : Déploiement GitLab et PostgreSQL

## Description du Projet

Ce projet utilise **Ansible** pour automatiser le déploiement d'une infrastructure comprenant :  
- **GitLab** : Plateforme de gestion du code source et de collaboration.  
- **PostgreSQL** : Base de données relationnelle pour GitLab ou d'autres applications.  

L'objectif est de garantir la reproductibilité et la centralisation de la gestion de l'infrastructure via des playbooks Ansible.

---

## Structure des Rôles Ansible

### Rôle : `gitlab`
#### Variables :
- `gitlab_domain` : Nom de domaine pour accéder à GitLab.  
- `gitlab_storage_path` : Chemin pour le stockage des données GitLab.

#### Tâches principales :
1. Télécharger et exécuter le script d'installation de GitLab.
2. Installer GitLab Enterprise Edition (EE).
3. Configurer GitLab :
   - Nom de domaine (`external_url`).
   - Chemin de stockage des données.  
4. Redémarrer GitLab après configuration.

#### Handler :
- Redémarrer GitLab avec `gitlab-ctl reconfigure`.

---

### Rôle : `bdd` (Base de Données)
#### Tâches principales :
1. Installer PostgreSQL.  
2. Configurer PostgreSQL :  
   - Modifier `pg_hba.conf` pour activer les connexions locales avec `trust`.
   - Permettre les connexions distantes en remplaçant `127.0.0.1/32` par `0.0.0.0/0`.  
3. Créer des bases de données :
   - `all`, `dev`, `stage`, `prod`.  
4. Créer un utilisateur PostgreSQL (`vagrant`) avec :
   - Tous les privilèges sur `all`.  
   - La propriété des bases `dev`, `stage`, et `prod`.  

#### Handler :
- Redémarrer PostgreSQL si une modification de configuration est effectuée.

---

## Fichiers Importants

### Inventaire : `inventory.yml`
Définit les hôtes cibles pour le déploiement :
```yaml
all:
  children:
    gitlab:
      hosts:
        gitlab-server:
          ansible_host: 192.168.224.130
          ansible_user: root
    postgres:
      hosts:
        postgres-server:
          ansible_host: 192.168.224.131
          ansible_user: root
          ansible_python_interpreter: /usr/bin/python3
