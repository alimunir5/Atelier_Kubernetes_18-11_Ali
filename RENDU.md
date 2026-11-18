# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : MUNIR / Ali**
**Groupe : RS2**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >`kubectl get nodes` affiche 3 noeuds. Deux agents et un serveur. Les agents n'ont pas de rôles définis et le server a le rôle "control-plane".
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >Des noms (notamment les deux agents, le serveur ainsi qu'un serveur supplémentaire) associées à des conteneurs et un statuS. J'en déduis que dans k3d, un noeud est un conteneur.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >Trois objets : un Pod (le conteneur nginx qui tourne). Le Service `web` vient lui de `kubectl expose` un Deployment `web` ,et et un ReplicaSet `web-68d995574f` 
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >L'ancien pod est passé en `Terminating`, et un nouveau pod a aussitôt été créé avec un nom différent

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >Non, Le nouveau pod affiche de nouveau « Welcome to nginx! ». Ma modification avait été écrite dans le système de fichiers du conteneur, qui est éphémère : quand le pod est supprimé, son conteneur et ses fichiers disparaissent. Le ReplicaSet recrée un pod neuf à partir de l'image `nginx` d'origine, qui ne contient pas ma modification.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Même lancé vers le Service (`svc/web`), `kubectl port-forward` choisit un seul pod au démarrage et crée un tunnel direct vers lui. Quand ce pod a été supprimé, le tunnel n'avait plus de destination. Le log l'indique : `error: lost connection to pod`. Le port-forward ne suit pas le nouveau pod : il faut le relancer.



## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > create c'est impératif, on donne un ordre une fois et si l'objet existe déjà ça fait une erreur. apply -f c'est déclaratif, on décrit ce qu'on veut dans un fichier YAML et Kubernetes crée ou met à jour pour que ça corresponde. On peut le relancer plusieurs fois sans problème et garder les fichiers dans Git.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector sert à dire au Service vers quels pods envoyer le trafic (ceux qui ont le label app: site). S'il correspond à aucun pod, le Service n'a pas d'endpoints et les requêtes n'arrivent nulle part, on le voit avec kubectl get endpoints site.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
   NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE
   site-5cc664db9c-5t6x5   1/1     Running   0          3m10s   10.42.0.9    k3d-lab-agent-0
   site-5cc664db9c-7tqvk   1/1     Running   0          2m56s   10.42.2.6    k3d-lab-server-0
   site-5cc664db9c-k8bgf   1/1     Running   0          3m4s    10.42.1.7    k3d-lab-agent-1
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le pod était en ErrImagePull (puis ImagePullBackOff). C'est kubectl describe pod -l app=site qui donne la cause dans les Events : l'image nginx:9.99 n'existe pas donc Kubernetes n'arrive pas à la télécharger.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui, côté cluster le site restait en ligne : pendant un rolling update les anciens pods ne sont supprimés que quand les nouveaux sont prêts, et le nouveau pod n'a jamais été prêt, donc les 3 pods en 1.28 ont continué à servir le site. Chez moi la page ne s'affichait plus mais c'était à cause du port-forward, qui avait déjà cassé pendant la mise à jour 1.27 vers 1.28 (lost connection to pod dans le log).

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://shiny-waffle-4jwr64xggxpjhg79-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui, on retrouve le Deployment mario avec kubectl get deploy mario. La seule différence c'est que Portainer a ajouté ses propres labels (io.portainer.kubernetes.application.owner: admin, stackid...). Portainer et kubectl parlent au même composant : l'API Server du control-plane. Portainer c'est juste une interface graphique qui envoie les mêmes requêtes que kubectl.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** 15 / 18
