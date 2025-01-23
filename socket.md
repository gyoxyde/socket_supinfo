
## 1. Création d'une Socket TCP

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>  // Pour les fonctions de socket

int main() {
    int sockfd;
    struct sockaddr_in server_addr;

    // Création de la socket (IPv4, TCP)
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1) {
        perror("Erreur lors de la création de la socket");
        exit(EXIT_FAILURE);
    }

    // Configuration de l'adresse du serveur
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);  // Port 8080
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  // Adresse locale

    // Connexion au serveur
    if (connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("Erreur lors de la connexion au serveur");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    printf("Connexion réussie au serveur\n");

    // Fermeture de la socket
    close(sockfd);
    return 0;
}
```

## 2. Serveur TCP

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main() {
    int server_fd, client_fd;
    struct sockaddr_in server_addr, client_addr;
    char buffer[1024] = {0};
    socklen_t addr_len = sizeof(client_addr);

    // Création de la socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd == -1) {
        perror("Erreur lors de la création de la socket");
        exit(EXIT_FAILURE);
    }

    // Configuration du serveur
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);  // Port 8080
    server_addr.sin_addr.s_addr = INADDR_ANY;

    // Bind (attacher la socket à un port)
    if (bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("Erreur lors du bind");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // Écouter les connexions
    if (listen(server_fd, 5) == -1) {
        perror("Erreur lors du listen");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Serveur en écoute sur le port 8080...\n");

    // Accepter une connexion
    client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &addr_len);
    if (client_fd == -1) {
        perror("Erreur lors de l'acceptation d'une connexion");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Client connecté\n");

    // Recevoir un message du client
    recv(client_fd, buffer, sizeof(buffer), 0);
    printf("Message reçu : %s\n", buffer);

    // Envoyer une réponse
    send(client_fd, "Bonjour, client !", 17, 0);

    // Fermer les sockets
    close(client_fd);
    close(server_fd);

    return 0;
}
```

## 3. Client UDP

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>

int main() {
    int sockfd;
    struct sockaddr_in server_addr;
    char *message = "Hello UDP Server!";
    char buffer[1024] = {0};
    socklen_t addr_len;

    // Création de la socket UDP
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd == -1) {
        perror("Erreur lors de la création de la socket UDP");
        exit(EXIT_FAILURE);
    }

    // Configuration de l'adresse du serveur
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    // Envoyer un message au serveur
    sendto(sockfd, message, strlen(message), 0, (struct sockaddr *)&server_addr, sizeof(server_addr));
    printf("Message envoyé au serveur UDP\n");

    // Recevoir une réponse
    addr_len = sizeof(server_addr);
    recvfrom(sockfd, buffer, sizeof(buffer), 0, (struct sockaddr *)&server_addr, &addr_len);
    printf("Réponse du serveur : %s\n", buffer);

    // Fermer la socket
    close(sockfd);
    return 0;
}
```

## 4. Serveur UDP

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>

int main() {
    int sockfd;
    struct sockaddr_in server_addr, client_addr;
    char buffer[1024] = {0};
    socklen_t addr_len = sizeof(client_addr);

    // Création de la socket UDP
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd == -1) {
        perror("Erreur lors de la création de la socket UDP");
        exit(EXIT_FAILURE);
    }

    // Configuration du serveur
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);
    server_addr.sin_addr.s_addr = INADDR_ANY;

    // Bind (attacher la socket à un port)
    if (bind(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("Erreur lors du bind");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    printf("Serveur UDP en écoute sur le port 8080...\n");

    // Recevoir un message du client
    recvfrom(sockfd, buffer, sizeof(buffer), 0, (struct sockaddr *)&client_addr, &addr_len);
    printf("Message reçu : %s\n", buffer);

    // Envoyer une réponse
    sendto(sockfd, "Hello, UDP Client!", 18, 0, (struct sockaddr *)&client_addr, addr_len);

    // Fermer la socket
    close(sockfd);
    return 0;
}
```
