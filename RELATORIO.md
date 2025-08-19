# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 09 syscalls
- ex1b_write: 07 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
Ambos os métodos tem a mesma finalidade: Mostrar na tela. O envio do dado à ser exibido pode ser feito tanto com o printf quanto com o write, entretanto, para a máquina, um é mais custoso que o outro, já que o printf armazena os textos em um BUFFER à ser enviado de uma vez ao finalizar o programa ou dar o fflush(), sem necessidade de multiplos syscalls, apesar de poder variar seu comportamento, porém, com o write, o texto sofre o syscall imediatamente. Grande diferença é o impacto de performace já que o write apesar de seguro, pesa mais para o sistema enquanto o printf, apesar de menos seguro, é menos impactante. 
```

**3. Qual método é mais previsível? Por quê?**

```
O método mais previsível é o Write já que, toda vez que realizar o write, é de garantia que ocorrerá o syscall.
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: 03
- Bytes lidos: 127

### Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### Análise

**1. Qual file descriptor foi usado? Por que não 0, 1 ou 2?**

```
Foi usado o descriptor 3 pois os descriptor 0, 1 e 2 são, respectivamente, configurados para stdin, stdout e stderr. O ponto é que o descriptor 3 mantém o descritor sem interromper o programa.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Pois todos as linhas de texto foram printadas no terminal ao rodar o código.
```

**3. O que acontece se esquecer de fechar o arquivo?**

```
Os recursos associados ao arquivo não são liberados causando vazamento de recursos, perda de dados e impedir acesso ao mesmo arquivo ou recurso.
```

**4. Por que verificar retorno de cada syscall?**

```
Para garantir que a operação seja um sucesso, evitando imprevisibilidades como erro, controle de fluxo e etc.
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000212 segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |       19        | 0.001178  |
| 64          |       23        | 0.000608  |
| 256         |       11        | 0.000000  |
| 1024        |       04        | 0.000000  |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o buffer, menor o número de syscalls pois o buffer armazena um bloco de dados. O buffer menor consegue levar blocos menores de dados para a syscall enquanto buffers maiores levam mais blocos de dados de uma vez só.
```

**2. Como você detecta o fim do arquivo?**

```
A variável bytes_lidos é do tipo ssize_t (semelhante ao sizeof mas como tipo) e ao receber o tamanho do arquivo, vai passando por cada quantidade de byte até atingir o número 0 que representa o momento de sair do loop e considerar que o arquivo foi finalizado.
```

**3. Todas as chamadas read() retornaram BUFFER_SIZE bytes?**

```
Não
```

**4. Qual é a relação entre syscalls e performance?**

```
Quanto mais syscalls, pior a performance pela necessidade de realizar mais chamadas no sistema o que afeta o fluxo do programa.
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: 1364
- Operações: 06
- Tempo: 0.000574 segundos
- Throughput: 2320.61 KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ V ] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para garantir que nenhum dado tenha se perdido no processo de cópia.
```

**2. Que flags são essenciais no open() do destino?**

```
O_WRONLY serve para indicar que o arquivo será aberto apenas para escrita. O_CREAT significa que, caso o arquivo especificado não exista, ele será criado. O_TRUNC serve para que, caso o arquivo já exista e foi aberto para escrita, seu conteúdo será apagado e seu tamanho será reduzido à zero como forma de limpar o arquivo para reescrevê-lo. Por fim, é necessário a permissão 0644 que permite que a flag O_CREAT funcione permitindo que o mesmo leia e escreva no arquivo.
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim, ambos arquivos tem númmero de reads e writes iguais pois, por se tratar de uma cópia, todo texto de um arquivo que teve seu trecho copiado pelo reads deve ser inserido ao outro arquivo por meio do write.
```

**4. Como você saberia se o disco ficou cheio?**

```
Saberia que o disco ficou cheio caso o número de bytes seja menor que o solicitado na escrita, indicando que o processo foi interrompido.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Os recursos associados ao arquivo não são liberados causando vazamento de recursos, perda de dados e impedir acesso ao mesmo arquivo ou recurso.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls demonstram esta transição por representarem a ponte entre o software e o hardware, controlando e permitindo programas que realizem ações de forma segura (modo usuário) ou interagir com recursos privilegiados (modo kernel).
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
File descriptors tem a importância de gerenciar as formas de entradas e saídas do sistema, abstraindo a complexidade de realizar essas ações no hardware para interação facilitada.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o buffer, maior a performance pois é necessário menos syscalls, que é muito custoso para a performance.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** ex4_copia

**Por que você acha que foi mais rápido?**

```
Pois provávelmente o buffer alocado no ex4_copia é maior que o buffer alocado pelo sistema para realizar a cópia dos dados.
```

---

## Entrega

Certifique-se de ter:
- [ V ] Todos os códigos com TODOs completados
- [ V ] Traces salvos em `traces/`
- [ V ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```