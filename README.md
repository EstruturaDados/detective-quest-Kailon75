#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define HASH_SIZE 10  // Tamanho da tabela hash

// =======================
// ESTRUTURAS
// =======================

// Árvore binária dos cômodos
typedef struct Sala {
    char nome[30];
    struct Sala *esquerda;
    struct Sala *direita;
    char *pista; // Pista encontrada nesta sala (NULL se não houver)
} Sala;

// BST para pistas
typedef struct Pista {
    char nome[30];
    struct Pista *esquerda;
    struct Pista *direita;
} Pista;

// Lista encadeada para tabela hash (suspeitos)
typedef struct SuspeitoNode {
    char suspeito[30];
    struct SuspeitoNode *prox;
} SuspeitoNode;

typedef struct HashNode {
    char pista[30];
    SuspeitoNode *suspeitos;
    struct HashNode *prox;
} HashNode;

HashNode* tabelaHash[HASH_SIZE];

// =======================
// FUNÇÕES DE ÁRVORE DE SALAS
// =======================
Sala* criarSala(char *nome, char *pista, Sala *esq, Sala *dir) {
    Sala *s = (Sala*) malloc(sizeof(Sala));
    strcpy(s->nome, nome);
    if (pista) {
        s->pista = (char*) malloc(strlen(pista)+1);
        strcpy(s->pista, pista);
    } else {
        s->pista = NULL;
    }
    s->esquerda = esq;
    s->direita = dir;
    return s;
}

// =======================
// FUNÇÕES DE BST PARA PISTAS
// =======================
Pista* inserirPista(Pista *raiz, char *nome) {
    if (!raiz) {
        Pista *novo = (Pista*) malloc(sizeof(Pista));
        strcpy(novo->nome, nome);
        novo->esquerda = novo->direita = NULL;
        return novo;
    }
    if (strcmp(nome, raiz->nome) < 0)
        raiz->esquerda = inserirPista(raiz->esquerda, nome);
    else if (strcmp(nome, raiz->nome) > 0)
        raiz->direita = inserirPista(raiz->direita, nome);
    return raiz;
}

void emOrdemPistas(Pista *raiz) {
    if (!raiz) return;
    emOrdemPistas(raiz->esquerda);
    printf("- %s\n", raiz->nome);
    emOrdemPistas(raiz->direita);
}

// =======================
// FUNÇÕES DE HASH PARA SUSPEITOS
// =======================
int hash(char *chave) {
    int soma = 0;
    for (int i = 0; i < strlen(chave); i++)
        soma += chave[i];
    return soma % HASH_SIZE;
}

void inserirNaHash(char *pista, char *suspeito) {
    int idx = hash(pista);
    HashNode *atual = tabelaHash[idx];

    // Procura se já existe a pista
    while (atual) {
        if (strcmp(atual->pista, pista) == 0) break;
        atual = atual->prox;
    }

    if (!atual) { // Não existe → cria
        atual = (HashNode*) malloc(sizeof(HashNode));
        strcpy(atual->pista, pista);
        atual->suspeitos = NULL;
        atual->prox = tabelaHash[idx];
        tabelaHash[idx] = atual;
    }

    // Insere suspeito na lista da pista
    SuspeitoNode *novo = (SuspeitoNode*) malloc(sizeof(SuspeitoNode));
    strcpy(novo->suspeito, suspeito);
    novo->prox = atual->suspeitos;
    atual->suspeitos = novo;
}

void mostrarHash() {
    printf("\n--- Associações Pista → Suspeito ---\n");
    for (int i = 0; i < HASH_SIZE; i++) {
        HashNode *atual = tabelaHash[i];
        while (atual) {
            printf("%s: ", atual->pista);
            SuspeitoNode *s = atual->suspeitos;
            while (s) {
                printf("%s ", s->suspeito);
                s = s->prox;
            }
            printf("\n");
            atual = atual->prox;
        }
    }
}

// =======================
// FUNÇÃO DE EXPLORAÇÃO
// =======================
void explorarSalas(Sala *atual, Pista **raizPistas) {
    char opc;
    while (atual) {
        printf("\nVocê está na sala: %s\n", atual->nome);

        // Coleta pista se existir
        if (atual->pista) {
            printf("Você encontrou a pista: %s!\n", atual->pista);
            *raizPistas = inserirPista(*raizPistas, atual->pista);
            // Exemplo: associa cada pista a um suspeito (hardcoded para demonstração)
            inserirNaHash(atual->pista, "Sr. X");
            inserirNaHash(atual->pista, "Sra. Y");
        }

        if (!atual->esquerda && !atual->direita) {
            printf("Fim do caminho.\n");
            break;
        }

        printf("Escolha direção: (e=esquerda, d=direita, s=sair): ");
        scanf(" %c", &opc);

        if (opc == 'e') atual = atual->esquerda;
        else if (opc == 'd') atual = atual->direita;
        else if (opc == 's') break;
        else printf("Opção inválida!\n");
    }
}

// =======================
// MAIN
// =======================
int main() {
    // Inicializa tabela hash
    for (int i = 0; i < HASH_SIZE; i++) tabelaHash[i] = NULL;

    // Construção estática da mansão
    Sala *sala5 = criarSala("Biblioteca", "Pista Livro", NULL, NULL);
    Sala *sala4 = criarSala("Jardim", "Pista Flor", NULL, NULL);
    Sala *sala3 = criarSala("Cozinha", "Pista Receita", NULL, NULL);
    Sala *sala2 = criarSala("Quarto", NULL, sala4, sala5);
    Sala *sala1 = criarSala("Sala de Estar", "Pista Sofá", sala2, sala3);

    Pista *raizPistas = NULL;

    explorarSalas(sala1, &raizPistas);

    printf("\n--- Todas as pistas coletadas (em ordem) ---\n");
    emOrdemPistas(raizPistas);

    mostrarHash();

    return 0;
}
