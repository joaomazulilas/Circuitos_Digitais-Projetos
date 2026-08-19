# Circuitos_Digitais-Projeto: Guess the Number!
Projeto de Circuitos Digitais do Curso de Ciência da Computação.


Feito por:

Pedro Felipe Bezerra Araujo;
João Pedro dos Santos Nascimento Silva.


# ---RELATÓRIO DO PROJETO DE CIRCUITOS DIGITAIS---

# *INTRODUÇÃO*:
Este projeto consiste no desenvolvimento de um sistema digital em ambiente Logisim que simula um jogo de adivinhação. O objetivo principal do hardware é processar um palpite de 4 bits inserido pelo usuário, compará-lo com um valor gerado internamente pela máquina e fornecer feedbacks visuais dinâmicos de precisão. Todo o ecossistema foi construído utilizando ferramentas e conceitos básicos de circuitos lógicos combinacionais e aritméticos, sem o uso de microcontroladores ou linhas de programação de software.


# *SUBCIRCUITO DECODER (DISPLAY DE 7 SEGMENTOS)*:
O decodificador é o circuito responsável por converter o palpite do jogador de binário para hexadecimal. O motivo dessa abordagem é que o display de 7 segmentos normalmente é limitado a mostrar caracteres de 0 a 9. Com o sistema hexadecimal, conseguimos expandir essa capacidade visual até o número 15, exibindo a letra correspondente (de A a F) sempre que o valor escolhido ultrapassa o dígito 9.

Além de fazer essa conversão, o circuito controla diretamente o acendimento dos filamentos do display, o que melhora a interface visual e garante que o jogador entenda seu palpite de forma rápida e fácil. O maior desafio nesta etapa foi realizar o preenchimento correto da tabela-verdade por meio da ferramenta de Análise Combinatória do Logisim, garantindo que o programa gerasse a malha lógica automatizada exatamente como planejado.


# *SUBCIRCUITO COMPARADOR*:
O comparador é o bloco responsável por confrontar o número secreto com o palpite escolhido pelo jogador, testando em tempo real se a resposta está correta. A montagem física do circuito foi relativamente simples, mas o verdadeiro desafio esteve na estruturação da sua lógica de validação.
Inicialmente, tentou-se utilizar a mesma ferramenta de Análise Combinatória aplicada no decodificador, mas o preenchimento da tabela-verdade para múltiplos bits simultâneos se mostrou confuso e propenso a erros. Por isso, optou-se por desenvolver a lógica e dispor as portas lógicas manualmente, utilizando o princípio de portas XNOR para testar a igualdade bit a bit. Após a construção, o comparador foi integrado ao lado do decodificador para a realização de testes superficiais, garantindo o funcionamento correto e estável de ambos.


# *SUBCIRCUITO DA DIFERENÇA*:
O circuito da diferença foi projetado com o objetivo de obter a distância absoluta (o módulo) do número chutado em relação ao número aleatório armazenado. A saída gerada por este bloco serve diretamente como entrada para o controle do LED RGB. Esta parte do projeto foi desenvolvida com ferramentas nativas do Logisim e é composta, de forma macro, por somadores de 4 bits, um multiplexador e blocos de complemento de 2. A lógica central consiste em receber os barramentos do Chute e do Aleatório, efetuar a comparação de magnitude e encaminhar os dados para uma soma de 4 bits que resulta na diferença em módulo.
No bloco do multiplexador, o circuito compara os números Chute e Aleatório bit a bit, partindo do MSB (bit mais significativo) até o LSB (bit menos significativo). Dessa forma, o sistema detecta se os números são iguais ou diferentes. Caso sejam diferentes, inicia-se uma rotina lógica para descobrir qual dos dois é maior, negando a suposição de um bit menor e realizando uma operação AND com o suposto maior. A partir daí, obtêm-se duas saídas de controle (Aleatório > Chute ou Chute > Aleatório), além do sinal de igualdade gerado por portas XNOR.

O bloco de complemento de 2 atua como o passo intermediário para transformar o somador em um subtrator. O circuito recebe a entrada do respectivo menor número, nega cada um de seus bits através de inversores e soma +1 ao resultado utilizando um somador de 1 bit. Para somar esse bit extra, o LSB recebe a entrada do próprio LSB inicial negado e uma constante de valor 1 com Carry-In em zero. Posteriormente, o Carry-Out deste primeiro estágio é conectado ao Carry-In do próximo bit, que por sua vez é somado a uma constante 0, repetindo o processo em cascata até o MSB.

O somador de 4 bits foi construído em uma arquitetura de camadas, projetando-se primeiro o somador completo de 1 bit para depois agrupar quatro deles em série. A lógica aplicada nos somadores individuais é a mesma do bloco de complemento, alterando apenas as entradas principais para os valores do Aleatório e do Chute. No primeiro estágio do somador de 4 bits, o Carry-In foi conectado ao Ground (Terra) para garantir que o "vai um" inicial seja zero. O circuito também prevê o tratamento de um eventual overflow ao final da operação.

Na junção final deste circuito, os bits do Chute e do Aleatório entram no sistema e seguem dois caminhos. Eles alimentam o multiplexador para a comparação de magnitude e, simultaneamente, partem para um conjunto de portas AND que recebem os sinais de seleção do multiplexador. Isso garante que apenas os bits do maior número passem adiante. Após essa filtragem seletiva, os sinais chegam ao somador de 4 bits, combinados ao complemento de 2 do menor número. Por fim, o resultado passa por portas OR para consolidar a saída final de 4 bits, uma operação segura e sem prejuízos de sinal, dado que não há fluxo de dados vindo do caminho aritmético oposto.


# *SUBCIRCUITO DO LED RGB*:
O circuito do LED RGB tem como propósito exibir visualmente a proximidade do acerto do jogador, utilizando a saída do circuito da diferença como seu sinal de controle. Ele é estruturado a partir de dois microcircuitos independentes: um focado no canal do LED Vermelho e outro no canal do LED Azul. Para que a transição de cores ocorra de forma gradual (e não apenas entre estados puramente ligados ou desligados), foi necessária uma extensão de bits nas conexões do componente. Utilizou-se um Splitter (divisor de barramento) para encaixar os 4 bits vindos da diferença nos pinos mais significativos (MSBs) do LED, enquanto os pinos menos significativos (LSBs) foram conectados a uma constante fixa de valor "f" (0x4 em hexadecimal, correspondente a 1111), limitando as variações mais sutis de intensidade.

No canal do LED Vermelho, a lógica foi desenvolvida para que o brilho atinja a intensidade máxima quando o jogador acertar o número secreto. No cenário de acerto, a diferença matemática será 0000. Para que o barramento envie a potência máxima de acendimento (11111111), o circuito aplica uma inversão lógica (portas NOT) em cada um dos bits da diferença. Isso estabelece uma relação inversa: quanto maior for a distância do erro, menos vermelho o LED acenderá; e quanto menor for a diferença, mais intensa será a cor vermelha.

No canal do LED Azul, o comportamento é invertido, visando o acendimento máximo no cenário de erro total. Quando ocorre o maior afastamento possível entre os números, a diferença resulta no valor 1111. Para ativar o canal azul em sua totalidade (11111111), o circuito simplesmente mantém a integridade dos bits da diferença, sem aplicar inversões. Isso configura uma proporção direta, onde erros maiores geram uma assinatura de cor predominantemente azul, e a aproximação do acerto atenua o brilho deste canal.

# *CONCLUSÃO*:
A integração de todos os subcircuitos no módulo principal demonstrou o funcionamento correto das regras lógicas projetadas para o jogo de adivinhação. Ademais, foi perceptível o aprendizado e implementação prática dos conteúdos abordados na disciplina. 

*Para mais detalhes sobre o circuito acesse individualmente cada commit dos circuitos, basta olhar os commits anteriores.*
