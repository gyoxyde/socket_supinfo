
## Objectif
L'objectif de cet exercice est de créer un programme en C capable de se connecter à un serveur via le protocole HTTP, d'envoyer une requête GET sur le port 80 et de sauvegarder la réponse dans un fichier.

---

## Étapes à suivre

### 1. Création de la socket TCP
- Utilisez la fonction `socket()` pour créer une socket TCP (`AF_INET` et `SOCK_STREAM`).
- Vérifiez si la socket a été créée correctement avec des messages d'erreur clairs.

### 2. Résolution du nom de domaine
- Demandez à l'utilisateur d'entrer un nom de domaine (par exemple : `example.com`).
- Utilisez `gethostbyname()` ou `getaddrinfo()` pour convertir le nom de domaine en adresse IP.
- Assurez-vous que l'adresse IP a été résolue correctement et gérez les erreurs avec des messages explicites.

### 3. Configuration de l'adresse du serveur
- Configurez une structure `sockaddr_in` pour définir :
  - La famille d'adresses (IPv4).
  - Le port 80 (en utilisant `htons(80)`).
  - L'adresse IP récupérée dans l'étape précédente.

### 4. Connexion au serveur
- Connectez-vous au serveur à l'aide de la fonction `connect()`.
- Si la connexion échoue, affichez un message d'erreur détaillé et terminez le programme proprement.

### 5. Construction de la requête HTTP GET
- Une fois la connexion établie avec succès, construisez une requête GET. La requête doit inclure les éléments suivants :
  1. La méthode HTTP (`GET`).
  2. Le chemin (`/` pour la page d'accueil).
  3. La version du protocole (`HTTP/1.1`).
  4. L'en-tête `Host` pour indiquer le nom de domaine cible.
  5. L'en-tête `Connection: close` pour demander au serveur de fermer la connexion après l'envoi de la réponse.
  6. Une ligne vide pour indiquer la fin des en-têtes.

Exemple de requête GET :
```http
GET / HTTP/1.1\r\n
Host: <nom_de_domaine>\r\n
Connection: close\r\n
\r\n
```
- Utilisez `snprintf()` pour construire la chaîne complète de la requête de manière sécurisée.

---

### 6. Envoi de la requête
- Utilisez la fonction `send()` pour envoyer la requête au serveur via le socket :
```c
if (send(sockfd, request, strlen(request), 0) == -1) {
    perror("Erreur lors de l'envoi de la requête");
    close(sockfd);
    return 1;
}
```
- Assurez-vous que toute la requête est envoyée.

### 7. Lecture de la réponse
- Utilisez `recv()` pour lire la réponse envoyée par le serveur. Comme la réponse peut être longue, lisez-la par blocs :
```c
char buffer[4096];
ssize_t bytes_received;
while ((bytes_received = recv(sockfd, buffer, sizeof(buffer) - 1, 0)) > 0) {
    buffer[bytes_received] = '\0'; // Terminer correctement la chaîne
    // Écrivez le contenu dans un fichier (voir étape 8)
}
if (bytes_received == -1) {
    perror("Erreur lors de la réception des données");
}
```
- La boucle continue jusqu'à ce que le serveur ferme la connexion (indiqué par `recv()` qui retourne 0).

### 8. Sauvegarde de la réponse dans un fichier
- Avant de commencer à lire, ouvrez un fichier pour sauvegarder la réponse (par exemple `output.html`) :
```c
FILE *file = fopen("output.html", "w");
if (!file) {
    perror("Erreur lors de l'ouverture du fichier");
    close(sockfd);
    return 1;
}
```
- Pendant la lecture des blocs de réponse, écrivez chaque bloc dans le fichier :
```c
fprintf(file, "%s", buffer);
```
- Une fois la lecture terminée, fermez le fichier :
```c
fclose(file);
```

### 9. Nettoyage et fermeture des ressources
- Une fois toutes les étapes terminées, fermez le socket pour libérer la ressource :
```c
close(sockfd);
```
- Affichez un message indiquant que la réponse a été sauvegardée :
```c
printf("La réponse a été sauvegardée dans 'output.html'.\n");
```

---