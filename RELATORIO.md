# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
O printf pode ter um comportamento diferente por conta 
do \n que acontece no final. 
Além de que, quando temos um \n duplo, no segundo o buffer é forçado 
a ser enviado como um único byte.
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
Método write. É este método porque, o quê exatamente acontece é que 
toda vez que temos uma chamada do write() temos uma syscall 
acontecendo.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
3. Porque está sendo aberto um arquivo, diferente do 0,
que é a entrada via teclado, o 1, que é saída no terminal e 
o 2, que é saída de erro no terminal. Neste caso, está 
acontecendo algo diferente.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Na função read, no terceiro parâmetro, temos o 
número máximo de bytes para ler. Rodando o
strace, podemos observar o '=' depois da função
de read, no qual, retornou a quantidade de bytes
que foi colocado na função.
```

**3. Por que verificar retorno de cada syscall?**

```
Para verificar se houve sucesso nessa chamada.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1325
- Chamadas read(): 21
- Tempo: 0.000428 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |83                 |0.001534           |
| 64          |21                 |0.000428           |
| 256         |6                 |0.000299           |
| 1024        |2                 |0.000251           |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Acontece que com o buffer, maior é possível ler mais caracteres,
que estão armazenados no mesmo ao realizar o read(). Logo, quanto mais 
caracteres armazenados no buffer(buffer grande) menor é a quantidade de 
syscalls.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não. Nos últimos read(), utilizando 64 de buffer por exemplo, o penúltimo retorna 45 e o 
último retorna 0. Isso acontece porque as linhas não possuem as mesmas quantidades de 
caracteres que o buffer tem de tamanho, 64 caracteres no caso. Então, próximo do final, vai 
sobrar alguns caracteres.
```

**3. Qual é a relação entre syscalls e performance?**

```
Acontece que cada chamada de syscall é custoso e leva tempo.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1397
- Operações: 6
- Tempo: 0.000569 segundos
- Throughput: 2397.64 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [x] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
É preciso verificar para saber se todos os dados recebidos no arquivo de origem foram armazenados no arquivo de 
destino.
```

**2. Que flags são essenciais no open() do destino?**

```
O_WRONLY para escrever no arquivo de destino.
O_CREAT para criar o arquivo, caso não exista.
O_TRUNC para sobrescrever o arquivo
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim. No código fazemos a chamada de uma syscall read e logo em seguida fazemos a chamada de uma syscall write. 
```

**4. Como você saberia se o disco ficou cheio?**

```
Poderíamos receber um erro durante o while. Quando fazemos uma chamada write(), retorna os bytes lidos. 
Se por acaso, retornar -1, ocorre algum erro. Ou também, ao analisar o strace, ver os retornos da syscalls de read()
retornando sempre a quantidade de bytes lidos, porém as syscall de write() não.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
A função close é feita para liberar os recursos do sistema. Além de que em um processo temos um número de file 
descriptors limitado e não fechando os arquivos, consequentemente não será possível criar mais file descriptors,
prejudicando o desenvolvimento de um processo.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
O usuário não possui autorização para realizar as operações privilegiadas: ler arquivo ou escrever. Apenas o Kernel 
consegue realizar essas ações. Desta forma, somente ao fazer as syscalls, essas operações podem ser realizadas. 
Usuário -> syscall read(), write() -> kernel.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Fazendo o código e analisando o strace, vi que o file descriptor é essencial para realizar as syscalls. Tanto para 
o write(), quanto para o read(), é necessário indicar qual o identificador para realizar a chamada.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Um buffer maior consegue armazenar mais dados, resultando em poucas chamadas de syscalls, diferente de um buffer 
pequeno, que por ser de um tamanho menor, deve realizar syscalls com mais frequência. Sabendo que syscalls levam 
tempo, podemos concluir que um buffer maior tem uma performance melhor. 
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** time ./ex4_copia

**Por que você acha que foi mais rápido?**

```
Porque no programa que foi feito vamos direto ao ponto: ler o arquivo e copiar no destino através das syscalls. 
Analisando o strace do 'cp dados/origem.txt dados/destino_cp.txt', é possível observar que ocorre um processo 
muito maior do que no programa que foi feito.
```

---

## 📤 Entrega
Certifique-se de ter:
- [x] Todos os códigos com TODOs completados
- [x] Traces salvos em `traces/`
- [x] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
