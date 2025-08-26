#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define TAMANHO_SENHA 4
#define MAX_TENTATIVAS 10

// Função para converter letra para nome completo da cor
void imprimir_cor(char c) {
    if (c == 'P') printf("Preto");
    else if (c == 'B') printf("Branco");
    else if (c == 'V') printf("Vermelho");
    else if (c == 'A') printf("Azul");
    else if (c == 'R') printf("Rosa");
    else if (c == 'M') printf("Marrom");
    else printf("Desconhecido");
}

// Função para gerar uma senha aleatória sem repetições
void gerar_senha_aleatoria(char *senha) {
    char cores[] = "PBAVRM";
    int usado[6] = {0};
    int i = 0, idx;
    
    while (i < TAMANHO_SENHA) {
        idx = rand() % 6;
        if (!usado[idx]) {
            senha[i] = cores[idx];
            usado[idx] = 1;
            i++;
        }
    }
    senha[TAMANHO_SENHA] = '\0';
}

// Função para converter para maiúsculas
void para_maiusculas(char *str) {
    int i = 0;
    while (str[i]) {
        if (str[i] >= 'a' && str[i] <= 'z') {
            str[i] = str[i] - 'a' + 'A';
        }
        i++;
    }
}

// Função para validar uma tentativa do usuário
int validar_tentativa(char *tentativa) {
    int i = 0, j;
    
    // Verificar tamanho
    while (tentativa[i]) i++;
    if (i != TAMANHO_SENHA) {
        printf("A tentativa deve ter exatamente 4 caracteres!\n");
        return 0;
    }
    
    // Verificar caracteres válidos e repetições
    for (i = 0; i < TAMANHO_SENHA; i++) {
        char c = tentativa[i];
        
        // Verificar se é uma cor válida
        if (c != 'P' && c != 'B' && c != 'V' && 
            c != 'A' && c != 'R' && c != 'M') {
            printf("Caracter invalido: %c. Use apenas: P, B, V, A, R, M.\n", c);
            return 0;
        }
        
        // Verificar repetições
        for (j = i + 1; j < TAMANHO_SENHA; j++) {
            if (c == tentativa[j]) {
                printf("Cores repetidas nao sao permitidas!\n");
                return 0;
            }
        }
    }
    
    return 1;
}

// Função para avaliar a tentativa em relação à senha
void avaliar_tentativa(char *senha, char *tentativa, char *resultado) {
    int certas_posicao = 0;
    int certas_cor = 0;
    int usados_senha[TAMANHO_SENHA] = {0};
    int usados_tentativa[TAMANHO_SENHA] = {0};
    int i, j;
    
    // Primeiro, verificar cores na posição correta
    for (i = 0; i < TAMANHO_SENHA; i++) {
        if (tentativa[i] == senha[i]) {
            certas_posicao++;
            usados_senha[i] = 1;
            usados_tentativa[i] = 1;
        }
    }
    
    // Depois, verificar cores corretas mas em posição errada
    for (i = 0; i < TAMANHO_SENHA; i++) {
        if (usados_tentativa[i]) continue;
        
        for (j = 0; j < TAMANHO_SENHA; j++) {
            if (!usados_senha[j] && tentativa[i] == senha[j]) {
                certas_cor++;
                usados_senha[j] = 1;
                break;
            }
        }
    }
    
    // Construir string de resultado
    int idx = 0;
    for (i = 0; i < certas_posicao; i++) {
        resultado[idx++] = 'x';
    }
    for (i = 0; i < certas_cor; i++) {
        resultado[idx++] = 'o';
    }
    for (i = idx; i < TAMANHO_SENHA; i++) {
        resultado[idx++] = '_';
    }
    resultado[idx] = '\0';
}

// Função para limpar a tela
void limpar_tela() {
    #ifdef _WIN32
        system("cls");
    #else
        system("clear");
    #endif
}

// Função para jogar com senha definida pelo usuário
void jogar_com_senha_definida() {
    char senha[TAMANHO_SENHA + 1];
    char tentativa[TAMANHO_SENHA + 1];
    char resultado[TAMANHO_SENHA + 1];
    int tentativas = 0;
    int i, acertou;
    
    printf("\n=== Jogar com Senha Definida ===\n");
    
    // Solicitar a senha
    do {
        printf("Defina a senha de 4 cores (P,B,V,A,R,M) sem repetir: ");
        scanf("%s", senha);
        para_maiusculas(senha);
    } while (!validar_tentativa(senha));
    
    printf("Senha definida com sucesso! Pressione Enter para comecar...");
    
    // Limpar o buffer de entrada
    while (getchar() != '\n');
    // Esperar o Enter
    getchar();
    
    // Limpar a tela para esconder a senha
    limpar_tela();
    printf("Agora tente adivinhar a senha!\n\n");
    
    // Loop principal do jogo
    while (tentativas < MAX_TENTATIVAS) {
        printf("Tentativa %d/%d: ", tentativas + 1, MAX_TENTATIVAS);
        scanf("%s", tentativa);
        para_maiusculas(tentativa);
        
        // Validar tentativa
        if (!validar_tentativa(tentativa)) {
            continue;
        }
        
        // Avaliar tentativa
        avaliar_tentativa(senha, tentativa, resultado);
        printf("Resultado: %s\n", resultado);
        
        // Verificar se acertou
        acertou = 1;
        for (i = 0; i < TAMANHO_SENHA; i++) {
            if (resultado[i] != 'x') {
                acertou = 0;
                break;
            }
        }
        
        if (acertou) {
            printf("\nParabens! Voce acertou a senha em %d tentativas!\n", tentativas + 1);
            break;
        }
        
        tentativas++;
    }
    
    if (tentativas == MAX_TENTATIVAS) {
        printf("\nVoce perdeu! A senha era: ");
        for (i = 0; i < TAMANHO_SENHA; i++) {
            imprimir_cor(senha[i]);
            if (i < TAMANHO_SENHA - 1) printf(", ");
        }
        printf("\n");
    }
}

// Função para jogar com senha aleatória
void jogar_com_senha_aleatoria() {
    char senha[TAMANHO_SENHA + 1];
    char tentativa[TAMANHO_SENHA + 1];
    char resultado[TAMANHO_SENHA + 1];
    int tentativas = 0;
    int i, acertou;
    
    printf("\n=== Jogar com Senha Aleatoria ===\n");
    
    // Gerar senha aleatória
    gerar_senha_aleatoria(senha);
    
    printf("Senha gerada! Pressione Enter para comecar...");
    
    // Limpar o buffer de entrada
    while (getchar() != '\n');
    // Esperar o Enter
    getchar();
    
    // Limpar a tela
    limpar_tela();
    printf("Agora tente adivinhar a senha!\n\n");
    
    // Loop principal do jogo
    while (tentativas < MAX_TENTATIVAS) {
        printf("Tentativa %d/%d: ", tentativas + 1, MAX_TENTATIVAS);
        scanf("%s", tentativa);
        para_maiusculas(tentativa);
        
        // Validar tentativa
        if (!validar_tentativa(tentativa)) {
            continue;
        }
        
        // Avaliar tentativa
        avaliar_tentativa(senha, tentativa, resultado);
        printf("Resultado: %s\n", resultado);
        
        // Verificar se acertou
        acertou = 1;
        for (i = 0; i < TAMANHO_SENHA; i++) {
            if (resultado[i] != 'x') {
                acertou = 0;
                break;
            }
        }
        
        if (acertou) {
            printf("\nParabens! Voce acertou a senha em %d tentativas!\n", tentativas + 1);
            break;
        }
        
        tentativas++;
    }
    
    if (tentativas == MAX_TENTATIVAS) {
        printf("\nVoce perdeu! A senha era: ");
        for (i = 0; i < TAMANHO_SENHA; i++) {
            imprimir_cor(senha[i]);
            if (i < TAMANHO_SENHA - 1) printf(", ");
        }
        printf("\n");
    }
}

// Função principal
int main() {
    int opcao;
    
    // Inicializar a semente para números aleatórios
    srand(time(NULL));
    
    do {
        // Exibir menu
        printf("\n=== JOGO DE SENHA ===\n");
        printf("1. Jogar escolhendo a senha\n");
        printf("2. Jogar com senha aleatoria\n");
        printf("3. Sair\n");
        printf("Escolha uma opcao: ");
        scanf("%d", &opcao);
        
        // Processar opção escolhida
        switch(opcao) {
            case 1:
                jogar_com_senha_definida();
                break;
            case 2:
                jogar_com_senha_aleatoria();
                break;
            case 3:
                printf("Obrigado por jogar!\n");
                break;
            default:
                printf("Opcao invalida! Tente novamente.\n");
                break;
        }
        
        // Pausa antes de limpar a tela
        if (opcao != 3) {
            printf("\nPressione Enter para continuar...");
            while (getchar() != '\n'); // Limpar buffer
            getchar(); // Esperar Enter
            limpar_tela(); // Limpar tela
        }
        
    } while (opcao != 3);
    
    return 0;
}
