RELATÓRIO TÉCNICO E GUIA DE ESTUDO

Localizador astronômico portátil

Montagem e fundamentos matemáticos\
da primeira versão

**ESP32-S3 • BNO085 • GPS NEO-6M ou NEO-M8N**

Protótipo alimentado por USB, com leitura no computador pelo Serial
Monitor.

Este manual conduz a montagem de três módulos, a validação dos sensores
e a transformação das coordenadas de uma estrela em instruções para
apontar o aparelho. A matemática começa em graus, vetores e
trigonometria e chega ao cálculo de altitude e azimute.

> **Resultado esperado**\
> Selecionar uma estrela de um pequeno catálogo local e visualizar no
> Serial Monitor a posição calculada, a orientação medida e uma
> indicação como “mova 14,15° para a direita e 5,89° para cima”.

**Público** Estudante de ensino médio técnico em informática\
**Versão** 1.0\
**Data** 16 de setembro de 2026

Montagem de bancada e validação ao ar livre. O computador fornece
energia e exibe os resultados. O funcionamento não depende de internet,
Wi-Fi ou Bluetooth.

# Sumário

[1 Objetivos e roteiro](#objetivos-e-roteiro) [3](#objetivos-e-roteiro)

[2 Componentes e funções](#componentes-e-funções)
[4](#componentes-e-funções)

[3 Arquitetura e convenções](#arquitetura-e-convenções)
[5](#arquitetura-e-convenções)

[4 Montagem e validação dos módulos](#montagem-e-validação-dos-módulos)
[6](#montagem-e-validação-dos-módulos)

[5 Fundamentos matemáticos](#fundamentos-matemáticos)
[10](#fundamentos-matemáticos)

[6 Como os sensores medem
orientação](#como-os-sensores-medem-orientação)
[14](#como-os-sensores-medem-orientação)

[7 Coordenadas terrestres e
celestes](#coordenadas-terrestres-e-celestes)
[18](#coordenadas-terrestres-e-celestes)

[8 Da esfera celeste à fórmula da
altitude](#da-esfera-celeste-à-fórmula-da-altitude)
[22](#da-esfera-celeste-à-fórmula-da-altitude)

[9 Exemplo numérico completo](#exemplo-numérico-completo)
[26](#exemplo-numérico-completo)

[10 Arquitetura do software](#arquitetura-do-software)
[28](#arquitetura-do-software)

[11 Integração e checklist das oito
metas](#integração-e-checklist-das-oito-metas)
[31](#integração-e-checklist-das-oito-metas)

[12 Problemas comuns e limites da
versão](#problemas-comuns-e-limites-da-versão)
[33](#problemas-comuns-e-limites-da-versão)

[13 Próximos passos e ficha de
ensaio](#próximos-passos-e-ficha-de-ensaio)
[34](#próximos-passos-e-ficha-de-ensaio)

[14 Referências técnicas](#referências-técnicas)
[35](#referências-técnicas)

**Como usar este manual**\
Leia primeiro as seções 1 a 4 para reconhecer os módulos e preparar os
testes. Estude as seções 5 a 9 para entender as contas. Use as seções 10
a 13 durante a programação e a validação. As referências ficam na seção
14.

> **Convenção de leitura**\
> Na explicação, a vírgula indica decimal: 23,55°. No pseudocódigo e nos
> exemplos de Serial, usa-se ponto: 23.55. “Alt” significa ângulo acima
> do horizonte; a altitude GPS, em metros, é outra grandeza.

# 1 Objetivos e roteiro

## 1.1 O que vamos construir

O localizador será um aparelho que conhece **onde está**, **que horas
são** e **para onde aponta**. Com essas informações e as coordenadas de
uma estrela, o ESP32-S3 calculará a direção em que ela deve aparecer no
céu. O computador exibirá o resultado pelo Serial Monitor: uma janela de
texto usada para acompanhar o programa.

Nesta primeira versão, o aparelho permanece conectado ao computador por
USB. O cabo fornece energia e transporta os resultados. Usaremos três
módulos principais: ESP32-S3, BNO085 e GPS. A observação inicial será de
estrelas cadastradas manualmente; o sistema não reconhecerá imagens do
céu.

> **Escopo da versão 1**\
> Sem Wi-Fi, Bluetooth, site, tela dedicada ou bateria acrescentada ao
> projeto. “Sem conexão externa” significa sem rede: a recepção dos
> satélites pelo GPS e o cabo USB continuam necessários. Não há motores;
> o usuário move o aparelho.

## 1.2 Primeira meta técnica: oito etapas

Execute na ordem abaixo. Marque cada item somente quando houver uma
evidência registrada no Serial Monitor ou no caderno de testes.

☐ **1 — Ler BNO085:** identificar o sensor e observar relatórios de
orientação contínuos ao movimentá-lo.

☐ **2 — Ler GPS:** receber mensagens legíveis e obter uma solução de
posição sob céu aberto.

☐ **3 — Validar hora/posição:** confirmar UTC, data completa, latitude,
longitude e idade das amostras.

☐ **4 — Cadastrar 5–10 estrelas:** registrar nome, ascensão reta,
declinação, unidade e referência do catálogo.

☐ **5 — Calcular Alt/Az:** reproduzir o exemplo numérico do manual e
conferir casos conhecidos.

☐ **6 — Comparar orientações:** alinhar os eixos, corrigir o norte
magnético e verificar os sinais das diferenças.

☐ **7 — Imprimir no Serial Monitor:** mostrar alvo, valores medidos,
calculados, qualidade e instrução de movimento.

☐ **8 — Testar no céu real:** comparar três estrelas visíveis, registrar
erros e repetir as medições.

## 1.3 Como saber que a etapa terminou

Um número aparecendo na tela não basta: ele precisa ter unidade, sentido
físico e atualização recente. Vamos separar falhas de ligação, leitura,
matemática e alinhamento. O objetivo inicial é demonstrar todo o caminho
dos dados; a precisão efetiva será medida nos testes, sem prometer um
erro angular antes da montagem.

# 2 Componentes e funções

## 2.1 Os três módulos principais

| **Componente** | **Função no projeto** | **O que conferir antes de comprar** |
|----|----|----|
| ESP32-S3 | Executa o programa, lê sensores e calcula a posição do astro. | Placa de desenvolvimento com USB de dados, regulador e esquema/pinagem identificáveis. |
| BNO085 genérico | Estima orientação usando sensores de movimento e campo magnético. | Breakout com SPI acessível: SCK, MISO, MOSI, CS, INT, RST e seleção PS0/PS1. |
| GPS NEO-6M ou NEO-M8N | Fornece posição geográfica, data e hora. | Placa com antena apropriada, UART e alimentação documentada; verificar a procedência. |

O ESP32-S3 é o coordenador. O BNO085 reúne acelerômetro, giroscópio,
magnetômetro e processamento de fusão. O GPS informa a localização, mas
não identifica a estrela nem a direção de um aparelho parado. O NEO-6M
atende à prova de conceito; o NEO-M8N oferece recepção de múltiplas
constelações GNSS. \[R1, R3, R4\]

## 2.2 Acessórios e ferramentas

Separe cabo USB **com dados**, fios curtos, protoboard ou conexões
soldadas, suporte rígido não magnético e acesso a um computador. Um
multímetro ajuda a conferir alimentação e continuidade. Esses itens
apoiam a montagem e não constituem novos módulos funcionais. A antena
pertence ao conjunto GPS; confirme se acompanha a placa.

## 2.3 Alimentação: leia o nome do pino

| **Dispositivo interno** | **Faixa operacional** | **Atenção na placa comprada** |
|----|----|----|
| BNO085 | VDD: 2,4–3,6 V; VDDIO: 1,65–3,6 V | O regulador e o pino de entrada do breakout precisam ser identificados. |
| NEO-6M / NEO-M8N | VCC: 2,7–3,6 V | Um VIN que aceita 5 V exige regulador na placa; o módulo interno não recebe 5 V. |
| ESP32-S3 | Lógica de 3,3 V | Não conectar sinais de 5 V aos GPIO. |

As faixas acima descrevem dispositivos internos, não uma entrada
universal dos breakouts. \[R1, R3–R5\] Use 3,3 V somente em entrada
explicitamente compatível. Uma placa com regulador pode precisar de
tensão maior no VIN; um pino “3V3” pode ser saída. Não decida pela cor
da placa ou por uma fotografia semelhante.

> **Antes de energizar**\
> Anote modelo, revisão e tensão de entrada de cada placa. Confirme
> terra comum, capacidade de alimentação e níveis lógicos. Utilize
> apenas a alimentação USB do protótipo; não acrescente bateria nesta
> versão.

# 3 Arquitetura e convenções

## 3.1 Quem envia o quê

O BNO085 entregará uma estimativa de orientação. O GPS entregará posição
e UTC. O catálogo ficará gravado no programa, com poucas estrelas e suas
coordenadas. O ESP32-S3 cruzará essas três fontes e enviará uma linha de
resultados ao computador.

SATÉLITES --\> antena + GPS --UART--\> ESP32-S3

MOVIMENTO / CAMPO MAGNÉTICO --\> BNO085 --SPI--\>

CATÁLOGO LOCAL DE ESTRELAS ------------------\>

ESP32-S3: validar dados e respectivas idades

--\> calcular tempo sideral e ângulo horário

--\> calcular altitude/azimute da estrela

--\> obter direção verdadeira do aparelho

--\> comparar as duas direções

--\> USB --\> Serial Monitor no computador

## 3.2 Regras que evitam confusão

| **Grandeza** | **Convenção usada neste manual** |
|----|----|
| Latitude / longitude | Norte e leste positivos; sul e oeste negativos. Valores em graus decimais. |
| Azimute, Az | Norte verdadeiro = 0°; leste = 90°; sul = 180°; oeste = 270°. |
| Altitude, Alt | Horizonte = 0°; zênite = +90°; abaixo do horizonte = valor negativo. |
| Hora | UTC, incluindo dia, mês e ano. O fuso serve somente para exibição ao usuário. |
| Ângulos no programa | Converter graus para radianos antes de funções trigonométricas; identificar unidades nos nomes. |

A “altitude” GPS é uma altura expressa em metros; **Alt**, aqui, é um
ângulo. A declinação celeste de uma estrela também é diferente da
declinação magnética usada para corrigir uma bússola.

## 3.3 Definir a seta de apontamento

Desenhe uma seta no suporte. Ela representa a direção em que queremos
apontar o aparelho. Registre como essa seta se relaciona aos eixos
impressos no BNO085. A montagem deve permanecer rígida depois do
alinhamento.

Não basta renomear qualquer valor de yaw como azimute ou pitch como
altitude: as convenções de eixo podem ser diferentes. A seção de
orientação mostrará como transformar o vetor da seta e verificar seu
sentido. O norte magnético medido será corrigido para o norte verdadeiro
antes da comparação astronômica. \[R1, R13\]

> **Regra de confiança**\
> Se posição, hora ou orientação estiverem ausentes, antigas ou não
> confiáveis, exibir o motivo e suspender a instrução de movimento. O
> programa nunca deve apresentar uma amostra antiga como uma nova
> medição.

# 4 Montagem e validação dos módulos

## 4.0 Plano de montagem por etapas

A montagem será incremental. Primeiro comprovamos que o computador
conversa com a ESP32-S3. Depois testamos cada sensor isoladamente; por
fim juntamos os dois. Essa ordem permite descobrir qual mudança
introduziu um problema.

**Passo A — Identificar as placas.** Fotografe os dois lados, registre
modelos e confira os esquemas. Identifique alimentação, GND, GPIO
livres, seleção de interface do BNO085 e níveis elétricos do GPS. O
número de um GPIO não é sua posição física no conector.

**Passo B — Preparar o computador.** Instale Arduino IDE, suporte
oficial ESP32 e as bibliotecas Adafruit BNO08x, Adafruit BusIO e
TinyGPSPlus. Faça isso antes do uso offline. Registre as versões
utilizadas. Escolha a placa e a porta corretas; USB nativo e ponte
USB–serial podem exigir configurações diferentes. \[R2, R7, R14\]

**Passo C — Testar somente a ESP32-S3.** Grave um programa que imprima
“Sistema iniciado” e um contador. Abra o Serial Monitor em 115200 baud.
Confirme que o contador avança e que reiniciar a placa produz uma nova
mensagem inicial.

**Passo D — Conectar o BNO085.** Desconecte o USB, monte o circuito SPI
da próxima página e confira cada fio. Energize novamente e execute o
teste de orientação. Deixe o GPS desconectado durante esse diagnóstico.

**Passo E — Testar o GPS.** Com a alimentação desligada, prepare sua
ligação UART. Comece pela leitura de mensagens, depois valide a posição
e a UTC sob céu aberto. As subseções seguintes detalham ambos os
ensaios.

**Passo F — Integrar.** Conecte os dois módulos, repita os testes e
observe se aparecem reinicializações, amostras atrasadas ou alterações
magnéticas. Só então acrescente o catálogo e os cálculos astronômicos.

## Checklist antes da integração

☐ Modelo e alimentação de cada placa identificados; nenhum sinal de 5 V
chega à ESP32-S3.

☐ ESP32-S3 imprime continuamente e reinicia de maneira previsível.

☐ BNO085 produz orientação recente e reage corretamente aos movimentos
conhecidos.

☐ GPS apresenta mensagens válidas, posição plausível e data/hora UTC
conferidas.

> **Critério de avanço**\
> Guarde uma captura ou transcrição de cada teste, com data e
> configuração. Se um ensaio falhar, corrija essa etapa antes de
> acrescentar funções; a seção de problemas comuns ajudará a localizar a
> causa.

## 4.1 Ligações SPI e UART

**Esta é uma pinagem proposta**, adequada somente após confirmar o
esquema da ESP32-S3 e dos breakouts. Todos os módulos compartilham GND.
A ligação de alimentação depende da entrada documentada de cada placa;
não existe um VIN universal.

| **Sinal do módulo** | **GPIO proposto** | **Papel da ligação** |
|----|----|----|
| BNO: SCL / SCK | 12 | Clock SPI enviado pela ESP32. |
| BNO: SDA / MISO | 13 | Dados recebidos do BNO085. |
| BNO: DI / MOSI / SA0 | 11 | Dados enviados ao BNO085. |
| BNO: CS | 10 | Seleção do BNO085, ativa em nível baixo. |
| BNO: INT | 7 | Aviso de evento; não omitir. |
| BNO: RST / NRST | 6 | Reinicialização do sensor; não omitir. |
| BNO: PS1 e PS0 | Conforme placa | Ambos altos na partida selecionam SPI. |
| GPS: TX | 18, RX da ESP | Mensagens enviadas pelo GPS. |
| GPS: RX | 17, TX da ESP | Opcional no primeiro teste; pode ficar desconectado. |

Em SPI, pinos rotulados SDA e SCL podem assumir MISO e SCK. Confira a
função, não apenas o nome. O BOOTN do BNO085 permanece no estado normal
previsto pelo breakout; é diferente do botão BOOT da ESP32. PS0/PS1 são
lidos no reset. \[R1\]

Exemplo de inicialização da UART: os dois últimos números indicam **RX e
TX da ESP32**, nessa ordem. O console USB permanece separado.

Serial.begin(115200);

Serial1.begin(9600, SERIAL_8N1, 18, 17);

O valor 9600 é ponto de partida documentado para GPS de fábrica, mas
placas reconfiguradas podem usar outro baud rate. \[R4, R7\] Não
confunda essa velocidade com os 115200 do console.

Reserve GPIO19/20 quando usados pelo USB nativo. Evite pinos de partida
0/3/45/46 e pinos ligados à memória; certas variantes reservam também
35/36/37. Confira LED e ponte serial da placa. \[R5, R8\]

> **Por que recomendamos SPI?**\
> A Adafruit relata problemas do I²C do BNO08x com ESP32-S3. SPI exige
> mais fios, mas mantém os três módulos. Um breakout somente I²C não
> deve ser comprado como se a compatibilidade estivesse garantida.
> \[R2\]

Como alternativa experimental, I²C usa PS1=PS0=0 e endereço 0x4A ou
0x4B, conforme SA0. Um scanner responder não comprova funcionamento
contínuo. \[R1\]

## 4.2 Primeiro teste do BNO085

## 4.2.1 Receber orientação antes de fazer astronomia

Use a biblioteca Adafruit BNO08x. Configure os GPIO SPI escolhidos,
informe RST na construção do objeto e inicialize com CS e INT. Comece
solicitando um único relatório de orientação, com taxa de projeto entre
20 e 50 atualizações por segundo. Essa taxa é uma escolha inicial, não
uma exigência do sensor. \[R6\]

Solicite **SH2_ROTATION_VECTOR** para obter orientação referida à
gravidade e ao norte magnético. O relatório
**SH2_GAME_ROTATION_VECTOR**, usado em um exemplo oficial, não fornece
referência absoluta de norte; não copie essa seleção sem modificá-la.
\[R1, R9\]

inicializar SPI nos pinos escolhidos

inicializar BNO085 com CS, INT e RST

habilitar SH2_ROTATION_VECTOR

repetir:

se o BNO085 reiniciou: habilitar relatório novamente

se chegou novo evento: guardar orientação e instante

imprimir leitura, qualidade e idade em ritmo limitado

getSensorEvent() permite verificar a chegada de uma nova amostra. Se
wasReset() indicar reinicialização, reconfigure os relatórios. Não
reutilize o instante antigo como se o sensor tivesse acabado de medir.
\[R6, R9\]

## 4.2.2 Observar, calibrar e registrar

A qualidade de relatório usa estados 0 a 3: não confiável, baixa, média
e alta. A estimativa angular de precisão do rotation vector usa
radianos; converta para graus ao exibi-la. Qualidade alta não é uma
comprovação independente de erro pequeno. \[R10, R11\]

Fixe o sensor em suporte não magnético, longe dos alto-falantes do
computador. Comece parado, depois explore diferentes orientações
suavemente e acompanhe a qualidade. Consulte o procedimento CEVA para
calibração e persistência; não suponha que os ajustes serão preservados
ao desligar. \[R12\]

☐ Receber amostras durante 60 segundos sem interrupção inesperada;
registrar resets se ocorrerem.

☐ Girar aproximadamente 90° e voltar à posição inicial três vezes;
anotar a dispersão observada.

☐ Inclinar a seta para cima e verificar que a altitude calculada
aumenta.

☐ Girar em torno da própria seta e verificar que a direção apontada
permanece equivalente.

☐ Afastar o computador e repetir; mudanças relevantes sugerem
interferência magnética.

> **Critério de aprovação**\
> A comunicação precisa ser contínua e os movimentos devem ter sentido
> correto. O alinhamento com o norte verdadeiro e a medição do erro
> final serão feitos depois; esta etapa comprova leitura e comportamento
> do sensor.

## 4.3 Primeiro teste do GPS

## 4.3.1 Mensagens primeiro, posição depois

O GPS pode transmitir mensagens mesmo sem conhecer sua posição. Conecte
TX do GPS ao RX da ESP32, além da alimentação correta e GND. Neste
primeiro teste, o RX do GPS pode ficar fisicamente desconectado, pois
ainda não enviaremos comandos.

Comece imprimindo os caracteres recebidos pela UART. Mensagens NMEA são
linhas de texto iniciadas por “\$”. Ver caracteres legíveis comprova o
caminho de comunicação; ainda precisamos verificar conteúdo e validade.
Use céu aberto, antena posicionada conforme sua documentação e tempo
para aquisição. Não há um tempo de espera garantido em qualquer
ambiente.

enquanto Serial1 tiver caracteres:

caractere = ler Serial1

enviar caractere ao parser TinyGPSPlus

se houver atualização:

mostrar posição, data, UTC, qualidade e idades

## 4.3.2 O que validar

RMC fornece, entre outros campos, data, hora e posição; GGA inclui hora,
qualidade do fix, satélites e altitude. GGA não fornece a data completa.
O parser interpreta essas mensagens e permite consultar validade e idade
de cada grandeza. \[R14–R16\]

isValid() informa que um campo foi aceito em algum momento. **Não
garante que ele acabou de ser atualizado.** Verifique também age() e o
recebimento contínuo de mensagens. Latitude, data e hora têm estados
próprios: testar somente um deles deixa lacunas. \[R15\]

☐ Confirmar mensagens legíveis e checksum aceito; ausência de fix não
deve ser exibida como posição válida.

☐ Conferir latitude e longitude com o local do teste; no Brasil,
geralmente ambas serão negativas.

☐ Conferir dia, mês, ano e UTC reais; um módulo antigo pode fornecer
data plausível, porém errada.

☐ Para atualização de 1 Hz, adotar inicialmente idade máxima de 2
segundos e mostrar expiração claramente.

☐ Registrar satélites e HDOP como diagnósticos; eles não substituem a
validade da solução.

☐ Após uma interrupção da recepção, verificar que os dados deixam de ser
tratados como recentes.

## 4.3.3 Condição para integrar

Posição e UTC precisam estar válidas, recentes e coerentes entre si.
Evite pausas longas no programa: a UART deve ser esvaziada
continuamente. Ao retirar a alimentação, o projeto pode precisar
adquirir novamente a solução; esta versão não depende de bateria de
backup.

> **Hora astronômica**\
> Guarde a hora em UTC junto com a data. Não subtraia o fuso antes das
> contas. Se mostrar hora local no console, identifique-a separadamente
> para evitar usar acidentalmente o horário errado no cálculo do céu.

# 5 Fundamentos matemáticos

## 5.1 Graus e radianos

Um ângulo descreve uma mudança de direção. Ao girar uma volta completa,
você retorna à direção inicial. Dividimos essa volta em **360 graus**,
escritos 360°. Um quarto de volta é 90°; meia volta é 180°. No
localizador, um erro de 5° significa que a direção apontada ainda
precisa mudar esse ângulo, não que faltam 5 centímetros.

O **radiano** mede o ângulo pela relação entre o comprimento de um arco
e o raio do círculo. Se o arco tem o mesmo comprimento que o raio, o
ângulo vale 1 radiano. Como a circunferência mede 2π vezes o raio, uma
volta completa mede 2π radianos. O número π vale aproximadamente
3,14159265.

``` math
\theta_{rad}\  = \ \frac{comprimento\ do\ arco}{raio}
```

``` math
360{^\circ}\  = \ 2\pi\ rad\ \ \ \ \ \ \ 180{^\circ}\  = \ \pi\ rad
```

``` math
\theta_{rad}\  = \ \theta_{graus}\  \times \ \frac{\pi}{180}
```

``` math
\theta_{graus}\  = \ \theta_{rad}\  \times \ \frac{180}{\pi}
```

**Exemplo passo a passo.** Para converter 30°: divida 30 por 180 e
multiplique por π. O resultado é π/6, aproximadamente 0,523599 rad. Para
voltar a graus, multiplique 0,523599 por 180/π: o resultado é
aproximadamente 30°.

| **Movimento**      | **Graus** | **Radianos**   |
|--------------------|-----------|----------------|
| Um quarto de volta | 90°       | π/2 ≈ 1,570796 |
| Meia volta         | 180°      | π ≈ 3,141593   |
| Uma volta          | 360°      | 2π ≈ 6,283185  |

> **Regra de programação**\
> As funções sin, cos, tan, asin, acos e atan2 de C/C++ trabalham em
> radianos. O Serial pode mostrar graus. Converter as entradas antes das
> contas e as saídas depois evita um dos erros mais comuns do projeto.

Graus também podem ser divididos em minutos e segundos de arco: 1° = 60′
e 1′ = 60″. Portanto 23°30′ = 23 + 30/60 = 23,5°. Isso é diferente de
minutos e segundos de um relógio. A ascensão reta, estudada adiante, usa
outra escala angular expressa em horas.

**Teste de entendimento.** Quanto valem 45° em radianos? Faça 45π/180 =
π/4 ≈ 0,785398. Na programação, use double nas contas astronômicas e
preserve mais casas internamente do que as exibidas.

## 5.2 Seno cosseno e tangente

Considere um triângulo retângulo, que possui um ângulo de 90°. A
hipotenusa é o lado oposto ao ângulo reto e o maior lado. Escolha um dos
outros ângulos, θ. O cateto que fica em frente a θ é o oposto; o que
encosta em θ é o adjacente.

/\|

hipotenusa / \| cateto oposto

/θ\|

/\_\_\|

cateto adjacente

No esquema, θ representa o ângulo no vértice inferior esquerdo. Para um
ângulo fixo, ampliar o triângulo multiplica todos os lados pelo mesmo
fator. As razões entre os lados continuam iguais; por isso elas podem
representar o ângulo.

``` math
sen(\theta)\  = \ \frac{cateto\ oposto}{hipotenusa}
```

``` math
\cos(\theta)\  = \ \frac{cateto\ adjacente}{hipotenusa}
```

``` math
\tan(\theta)\  = \ \frac{cateto\ oposto}{cateto\ adjacente}\  = \ \frac{sen(\theta)}{\cos(\theta)}
```

**Exemplo 3–4–5.** Imagine cateto oposto de 3 cm, adjacente de 4 cm e
hipotenusa de 5 cm. Temos sen(θ) = 3/5 = 0,6; cos(θ) = 4/5 = 0,8; tan(θ)
= 3/4 = 0,75. Esses números são razões sem unidade. Um triângulo de
lados 6, 8 e 10 cm produz os mesmos valores.

As funções inversas recuperam o ângulo: θ = arcsen(0,6) ≈ 36,87°. Em
código, escreve-se asin(0.6), cujo resultado precisa ser convertido de
radianos para graus. Da mesma forma, arccos corresponde a acos e
arco-tangente a atan.

> **Função inversa não é divisão**\
> arcsen(x) significa “qual ângulo tem seno x?”. Não significa 1/sen(x).
> Para evitar essa confusão, este manual prefere a palavra arcsen em vez
> de sen elevado a −1.

Seno e cosseno sempre ficam entre −1 e +1. A tangente pode assumir
valores muito grandes e não é definida quando cos(θ) = 0, como em 90°.
Mais adiante, atan2 permitirá descobrir uma direção sem dividir por uma
componente que pode ser zero.

**Ligação com o projeto.** Quando um vetor unitário aponta 30° acima do
horizonte, sua componente vertical é sen(30°) = 0,5. Sua projeção no
plano horizontal mede cos(30°) ≈ 0,866. As próximas fórmulas
astronômicas usarão exatamente essa ideia.

## 5.3 Círculo trigonométrico e vetores

Um vetor é uma seta com direção, sentido e comprimento. Um **vetor
unitário** tem comprimento 1: ele descreve uma direção sem precisar
indicar uma distância. Essa representação combina com o localizador,
porque precisamos apontar para uma estrela, não calcular a distância até
ela.

<figure>
<img src="media/image1.png" style="width:6.35in;height:3.84528in"
alt="01 circulo trigonometrico" />
<figcaption><p>Figura 1 O ponto do círculo é a ponta do vetor unitário.
As projeções nos eixos são cosseno e seno.</p></figcaption>
</figure>

No círculo de raio 1, medimos θ a partir do eixo x positivo, no sentido
anti-horário. O ponto alcançado tem coordenadas x = cos(θ) e y = sen(θ).
Para θ = 30°, x ≈ 0,866 e y = 0,500. O triângulo desenhado é apenas a
decomposição da mesma seta em duas direções perpendiculares.

``` math
x\  = \ \cos(\theta)\ \ \ \ \ \ \ y\  = \ sen(\theta)\ \ \ \ \ \ \ x^{2}\  + \ y^{2}\  = \ 1
```

No primeiro quadrante, x e y são positivos. No segundo, x é negativo e y
positivo. No terceiro, ambos são negativos. No quarto, x é positivo e y
negativo. Esses sinais são necessários para distinguir, por exemplo, 30°
de 150°: ambos têm seno 0,5, mas cossenos de sinais opostos.

**Exemplo de verificação.** 0,866² + 0,500² ≈ 0,999956. A pequena
diferença em relação a 1 vem do arredondamento. Com mais casas decimais,
a soma aproxima-se ainda mais de 1.

Em três dimensões acrescentamos uma componente vertical. Um vetor de
direção terá três números; eles não são três ângulos. Na base local
Leste, Norte, Cima, escreveremos esse vetor como (E, N, U). Seus
componentes podem ser positivos ou negativos, conforme o lado para o
qual a seta aponta.

> **Duas convenções diferentes**\
> O círculo matemático começa em x e cresce no sentido anti-horário.
> Nosso azimute começa no norte e cresce em direção ao leste. A função
> atan2 deve receber as componentes na ordem que corresponde à convenção
> usada.

## 5.4 Como atan2 descobre o quadrante

Se conhecemos x e y de um vetor, queremos recuperar seu ângulo. Usar
somente atan(y/x) perde informação: os vetores (1, 1) e (−1, −1) têm a
mesma divisão y/x = 1, mas apontam em sentidos opostos. Além disso,
dividir por x é um problema quando x = 0.

**atan2(y, x)** recebe os dois componentes separados. Assim, considera
seus sinais e escolhe o quadrante correto. Em C/C++, o primeiro
argumento é y e o segundo é x. O retorno fica usualmente entre −π e +π
radianos, equivalente a −180° e +180°.

| **x** | **y** | **atan2 em graus** | **Ângulo normalizado** |
|-------|-------|--------------------|------------------------|
| 1     | 1     | 45°                | 45°                    |
| −1    | 1     | 135°               | 135°                   |
| −1    | −1    | −135°              | 225°                   |
| 1     | −1    | −45°               | 315°                   |
| 0     | 1     | 90°                | 90°                    |

**Normalizar** significa escrever o mesmo ângulo no intervalo desejado.
−45° e 315° indicam a mesma direção: basta somar uma volta completa.
Para colocar qualquer ângulo entre 0° inclusive e 360° exclusivo,
calculamos o resto de uma divisão por 360 e corrigimos o sinal negativo.

wrap360(x):

r = fmod(x, 360.0)

se r \< 0: r = r + 360.0

retornar r

wrap180(x):

retornar wrap360(x + 180.0) - 180.0

Não use o operador % com double em C++; use fmod. O segundo procedimento
produz resultados entre −180° inclusive e +180° exclusivo. Ele será útil
para escolher o menor giro horizontal até o alvo.

**Exemplo no norte.** Aparelho em 358°, estrela em 2°. A subtração
simples é 2 − 358 = −356°. Aplicando wrap180, o resultado é +4°: basta
girar quatro graus para a direita. Não é preciso dar quase uma volta
inteira.

No sistema Leste–Norte–Cima, a direção norte deve resultar em 0°. Por
isso o azimute será atan2(E, N): a componente norte ocupa a posição de
referência, e a componente leste indica o crescimento do ângulo. Se E e
N forem ambos zero, a seta está vertical e **não existe azimute
definido**. O software deve tratar esse caso explicitamente.

# 6 Como os sensores medem orientação

## 6.1 Acelerômetro e direção vertical

O acelerômetro mede força específica em três eixos, normalmente em m/s².
Parado sobre uma mesa, ele não informa zero: o apoio da mesa impede a
queda e produz uma leitura com módulo próximo de g ≈ 9,81 m/s². Em queda
livre ideal, sua leitura aproxima-se de zero. Portanto, chamá-lo
simplesmente de “sensor de gravidade” esconde um detalhe importante.

Quando o aparelho está parado ou se move suavemente, a leitura permite
estimar a vertical. Adote, para este exemplo, eixos físicos **x para a
frente, y para a esquerda e z para cima**. Com a placa nivelada e a
convenção de sinal confirmada, a leitura é aproximadamente (0, 0, +g). O
vetor gravidade físico aponta para baixo; nesse repouso, tem sentido
oposto à força específica medida.

``` math
u\  = \ \frac{a}{\| a\|}\ \ \ \ \ \ \ g_{físico}\  \approx \  - a
```

Aqui a = (ax, ay, az), seu módulo é a raiz de ax² + ay² + az², e u é a
direção “para cima” escrita nos eixos do aparelho. Se a seta de
apontamento é o eixo +x, a projeção dessa seta na vertical vale ux. Logo
sua elevação pode ser calculada por:

``` math
{Alt}_{repouso}\  = \ atan2(ax,\ \sqrt{}(ay²\  + \ az²))
```

**Por que essa fórmula funciona?** ax é a parte vertical da seta na
geometria adotada. A raiz reúne a parte perpendicular a x. atan2 compara
a componente que sobe com a componente horizontal equivalente,
devolvendo um ângulo de −90° a +90°. O fator g aparece em ambas as
partes e se cancela.

**Exemplo.** Ao elevar a frente em 30°, leituras ideais seriam ax =
4,905; ay = 0; az ≈ 8,496 m/s². Então atan2(4,905; 8,496) ≈ 30°.
Nivelado, atan2(0; 9,81) = 0°.

Uma medida de inclinação lateral possível nessa convenção é atan2(ay,
az). Os sinais e os nomes “pitch” e “roll” variam entre bibliotecas; a
definição geométrica precisa vir primeiro. Teste fisicamente o aumento
da elevação antes de aceitar o sinal.

> **Limitação física**\
> Ao acelerar, sacudir ou vibrar o aparelho, a leitura mistura movimento
> com a referência vertical. E girar horizontalmente sobre a mesa não
> altera a gravidade medida. Só o acelerômetro não determina o norte nem
> o azimute.

## 6.2 Giroscópio e magnetômetro

### Giroscópio e integração angular

O giroscópio mede velocidade angular: quão rápido o aparelho gira em
cada eixo. Sua unidade pode ser graus por segundo ou radianos por
segundo; a interface usada deve informar qual. Para obter uma mudança de
ângulo, multiplicamos a velocidade pelo intervalo de tempo.

``` math
\Delta\theta\  = \ \omega\  \times \ \Delta t\ \ \ \ \ \ \ \theta_{novo}\  \approx \ \theta_{anterior}\  + \ \omega\  \times \ \Delta t
```

**Exemplo.** Uma rotação de 20°/s durante 0,5 s corresponde a 10°. Se as
leituras chegam a cada 0,02 s, uma leitura de 20°/s acrescenta 0,4°.
Somar várias parcelas desse tipo chama-se integração numérica. Use o
intervalo realmente medido, não apenas o intervalo esperado.

O sensor possui pequenos desvios. Um erro constante de 0,1°/s acumula 6°
em um minuto, mesmo que o aparelho esteja parado. Esse crescimento de
erro chama-se **deriva**. A integração é útil para acompanhar movimentos
rápidos, mas não cria uma referência absoluta de norte. Em três
dimensões, rotações sucessivas também dependem da ordem: somar ângulos
por eixo é só uma ilustração simplificada.

### Magnetômetro e direção do norte

O magnetômetro mede o campo magnético em três eixos, normalmente em
microteslas. O campo terrestre tem componentes horizontal e vertical.
Uma bússola usa a componente horizontal para indicar o norte magnético;
o norte geográfico é definido pelo eixo de rotação terrestre.

Em um caso didático nivelado, com o campo já expresso em componentes
norte e leste, a direção do campo pode ser obtida por atan2(componente
leste, componente norte). No sensor real, os eixos estão presos ao
aparelho e podem estar inclinados. É necessário compensar essa
inclinação e interpretar corretamente a orientação, tarefa na qual a
fusão do BNO085 ajuda.

Ímãs, alto-falantes, aço e correntes próximas alteram a leitura. Uma
calibração pode estimar parte dos efeitos fixos da montagem, mas não
torna inofensivo qualquer objeto que apareça perto durante o uso.

``` math
{Az}_{verdadeiro}\  = \ wrap360({Az}_{magnético}\  + \ D)
```

**D** é a declinação magnética, positiva para leste e negativa para
oeste. Se Az magnético = 100° e D = −20°, então Az verdadeiro = 80°.
Esses valores são fictícios. Consulte e registre D para o local e a data
do teste; guarde o valor no programa para operar offline. Não confunda D
com a declinação celeste δ de uma estrela. \[R13\]

## 6.3 Fusão de sensores no BNO085

O BNO085 combina três fontes que se complementam: o giroscópio acompanha
as mudanças rápidas, o acelerômetro ajuda a reconhecer a vertical e o
magnetômetro fornece referência magnética horizontal. O processamento
interno estima uma orientação conjunta. Essa combinação é chamada
**sensor fusion**, ou fusão de sensores. Ela reduz problemas de cada
medição isolada, mas não elimina perturbações físicas nem erros de
montagem.

Para esta aplicação, solicite **SH2_ROTATION_VECTOR**, que utiliza a
referência magnética. O **SH2_GAME_ROTATION_VECTOR** não usa o
magnetômetro como referência de norte; pode ser adequado para movimentos
relativos, mas não basta para uma bússola astronômica. \[R1, R9\]

| **Fonte** | **Contribuição principal** | **Limitação a observar** |
|----|----|----|
| Acelerômetro | Vertical em movimento suave | Aceleração externa e vibração |
| Giroscópio | Mudanças rápidas de orientação | Deriva acumulada |
| Magnetômetro | Referência magnética | Perturbações e calibração |
| Fusão | Orientação estimada conjunta | Depende das referências e da qualidade |

O resultado normalmente vem como **quaternion**, quatro números que
representam uma rotação tridimensional. Eles não são quatro ângulos.
Podemos escrevê-lo como q = (w, x, y, z), embora APIs possam organizar
ou nomear esses campos de outro modo. Confira sempre os nomes dos campos
e o sentido da transformação.

``` math
w^{2}\  + \ x^{2}\  + \ y^{2}\  + \ z^{2}\  \approx \ 1
```

**Intuição.** Imagine que a seta física do aparelho está desenhada na
placa. O quaternion é uma receita para girar essa seta e descobrir como
ela fica orientada em um referencial externo. A forma q e a forma −q
representam a mesma rotação; uma troca de sinais dos quatro componentes
não é, por si só, um salto físico.

O firmware deve registrar orientação, instante da amostra e qualidade
informada pelo sensor. O status de 0 a 3 indica níveis de confiança, não
uma garantia de erro em graus. A estimativa angular de precisão, quando
disponível, vem em radianos e precisa ser convertida antes de aparecer
em graus. \[R10, R11\]

> **Não é necessário programar a fusão nesta versão**\
> Nosso trabalho é ler o relatório adequado, identificar a convenção dos
> eixos, alinhar a seta de apontamento e avaliar a qualidade. O
> algoritmo interno do BNO085 já executa a fusão.

## 6.4 Da orientação do sensor à seta do aparelho

Marque uma seta rígida no suporte. Ela será a direção que o usuário
aponta para a estrela. Não presuma que uma borda da placa, o eixo +x do
chip e o “yaw” de um exemplo significam a mesma coisa. Primeiro registre
a orientação do sensor no suporte e qual vetor da placa corresponde à
seta.

Chamaremos esse vetor unitário de p. Se a seta coincide com +x do
sensor, p = (1, 0, 0). Caso coincida com outro eixo, p muda. A
transformação tem dois passos: aplicar a rotação medida e converter o
referencial da biblioteca para a base local Leste–Norte–Cima. Essa
conversão de eixos deve ser explícita no módulo imu.

``` math
v_{local}\  = \ R_{convenção}\  \times \ R(q)\  \times \ p
```

R(q) representa a rotação do quaternion, e R convenção representa a
troca de eixos e sinais necessária para a base escolhida. Se a API
fornecer a transformação no sentido inverso, use sua inversa; em um
quaternion unitário, isso corresponde ao conjugado (w, −x, −y, −z).

**Exemplo algébrico condicionado.** Depois de converter q para
representar, de fato, a rotação sensor → Leste–Norte–Cima, e adotando p
= (1, 0, 0), a primeira coluna da matriz de rotação fornece:

``` math
E\  = \ 1\  - \ 2(y^{2}\  + \ z^{2})
```

``` math
N\  = \ 2(xy\  + \ wz)\ \ \ \ \ \ \ U\  = \ 2(xz\  - \ wy)
```

Essas expressões pressupõem um quaternion unitário. Elas não devem ser
aplicadas cegamente ao pacote bruto de qualquer biblioteca. Para outro
vetor p, é preciso usar a matriz inteira ou uma função de rotação de
vetor.

``` math
{Alt}_{aparelho}\  = \ graus(atan2(U,\ \sqrt{}(E²\  + \ N²)))
```

``` math
{Az}_{magnético}\  = \ wrap360(graus(atan2(E,\ N)))
```

Depois, aplique a declinação magnética D para obter o azimute
verdadeiro. Se uma transformação anterior já incluiu essa correção, não
a aplique uma segunda vez.

**Validação física em quatro movimentos.** Nivele a seta: altitude
próxima de 0°. Eleve a frente: altitude aumenta. Gire horizontalmente do
norte para o leste: azimute aumenta cerca de 90°. Gire o suporte em
torno da própria seta: sua direção espacial deve permanecer
aproximadamente igual. Esse último teste detecta confusão entre roll e
direção apontada.

> **Uma correção fixa não resolve tudo**\
> Um deslocamento constante pode corrigir um alinhamento simples. Eixos
> trocados, inversão de rotação e distorções magnéticas que variam com a
> direção exigem corrigir a causa, não acrescentar um número ao yaw.

# 7 Coordenadas terrestres e celestes

## 7.1 Latitude longitude altitude e azimute

A **latitude φ** informa a posição ao norte ou ao sul do equador
terrestre: norte positivo, sul negativo. A **longitude λ** informa a
posição a leste ou a oeste de Greenwich: leste positivo, oeste negativo.
Um exemplo aproximado é φ = −23,55° e λ = −46,63°. O GPS fornece esses
números e a hora; ele não mede a direção para a qual um aparelho parado
aponta.

<figure>
<img src="media/image2.png" style="width:6.25in;height:3.85417in"
alt="02 coordenadas horizontais" />
<figcaption><p>Figura 2 Azimute é o giro no horizonte. Altitude é a
elevação da direção acima do horizonte.</p></figcaption>
</figure>

O **azimute Az** começa no norte geográfico: norte = 0°, leste = 90°,
sul = 180° e oeste = 270°. Seu valor cresce no sentido horário quando o
horizonte é visto de cima. A **altitude angular Alt** vale 0° no
horizonte, +90° no zênite, que fica diretamente acima de você, e é
negativa abaixo do horizonte.

Um astro com Az = 90° e Alt = 30° está na direção leste e 30° acima do
horizonte. Um astro com Alt = −10° está geometricamente abaixo do
horizonte; não deve gerar uma instrução de observação como se estivesse
visível. Prédios, árvores e montanhas ainda podem esconder um astro com
Alt positiva.

> **Duas altitudes diferentes**\
> A altitude do GPS é uma altura em metros, com referência que depende
> da mensagem. A altitude astronômica é um ângulo. Nesta primeira
> transformação de estrelas distantes, utilizamos latitude, longitude e
> UTC; não somamos metros do GPS a graus do céu.

O horizonte e o zênite pertencem ao observador. Por isso as coordenadas
Alt/Az de uma mesma estrela mudam com o lugar e o instante. Precisamos
de outro sistema para cadastrar as estrelas de forma estável: ascensão
reta e declinação.

## 7.2 Ascensão reta declinação e catálogo inicial

Imagine prolongar o plano do equador terrestre até o céu. Ele define o
**equador celeste**. A **declinação δ**, ou Dec, mede o afastamento
angular ao norte ou ao sul desse equador: de −90° a +90°. A **ascensão
reta α**, ou RA, mede a posição ao longo do equador a partir de uma
origem convencional ligada ao equinócio.

A RA costuma ser escrita em horas, minutos e segundos de ângulo. Essas
horas não são a hora do relógio nem o horário em que a estrela aparece.
Uma volta completa tem 24 h de RA e 360°, portanto:

``` math
24\ h\  = \ 360{^\circ}\ \ \ \ \ \ \ 1\ h\  = \ 15{^\circ}\ \ \ \ \ \ \ {RA}_{graus}\  = \ 15\  \times \ {RA}_{horas}
```

**Exemplo.** RA = 6 h 45 min equivale a 6 + 45/60 = 6,75 h.
Multiplicando por 15, obtemos 101,25°. Para incluir segundos, some
segundos/3600 antes de multiplicar. Para uma declinação negativa, o
sinal vale para tudo: −16°42′ = −(16 + 42/60) = −16,7°.

O catálogo abaixo reúne **oito estrelas reais**. As posições são
aproximadas, arredondadas de dados ICRS na época J2000 do SIMBAD. Grave
também o nome da referência e a época. Não misture essas coordenadas com
posições “da data” sem identificar a diferença. \[A6\]

| **Estrela**           | **RA em graus** | **Dec em graus** |
|-----------------------|-----------------|------------------|
| Sirius                | 101,287155      | −16,716116       |
| Canopus               | 95,987958       | −52,695661       |
| Alfa Centauri sistema | 219,902083      | −60,833972       |
| Rigel                 | 78,634467       | −8,201638        |
| Antares               | 247,351915      | −26,432003       |
| Spica                 | 201,298247      | −11,161319       |
| Aldebaran             | 68,980163       | +16,509302       |
| Achernar              | 24,428523       | −57,236753       |

Na primeira versão, use um índice para selecionar uma estrela no código
ou pelo Serial. Nem todas estarão visíveis na mesma noite. Para os
testes, prefira alvos calculados entre 20° e 70° de altitude,
identificados independentemente no céu.

> **Precisão desta primeira versão**\
> O eixo terrestre muda lentamente de direção por precessão. Aplicar
> diretamente um catálogo J2000 ao tempo sideral atual é uma aproximação
> didática. Identifique a saída como “J2000 sem precessão e sem
> refração”. O aperfeiçoamento será atualizar as coordenadas para a data
> de forma coerente.

## 7.3 Tempo sideral local e ângulo horário

O céu parece girar porque a Terra gira. Para descobrir onde está uma
estrela, precisamos relacionar sua RA ao giro atual do céu sobre o
observador. O **tempo sideral local**, LST, expressa essa relação: ele
corresponde à RA que está cruzando a parte de culminação superior do
meridiano local naquele instante. O meridiano é o círculo que passa
pelos polos celestes e pelo zênite. \[A2\]

Pense no LST como um indicador da “RA que está passando agora pelo
meridiano”. Ele não é a hora civil. Um ciclo sideral dura
aproximadamente 23 h 56 min de tempo civil. Por isso uma estrela tende a
cruzar o meridiano alguns minutos mais cedo a cada dia civil.

O LST depende da data, da hora e da longitude. A latitude entra depois,
na transformação de coordenadas, mas não no cálculo do LST. Como usamos
o GPS, a entrada temporal deve ser **UTC**, com data completa. Não
aplique o fuso horário local antes de calcular o céu.

``` math
H\  = \ LST\  - \ RA
```

O resultado H é o **ângulo horário**. Trabalhe com unidades iguais: se
LST e RA estão em graus, H sai em graus; se estão em horas, H sai em
horas e pode ser multiplicado por 15. Adotaremos H positivo para oeste
do meridiano e negativo para leste. Normalize para −180° a +180° usando
wrap180.

| **Situação** | **Exemplo** | **Interpretação**            |
|--------------|-------------|------------------------------|
| H = 0°       | LST = RA    | Astro na culminação superior |
| H \< 0°      | H = −40°    | Astro a leste do meridiano   |
| H \> 0°      | H = +40°    | Astro a oeste do meridiano   |

**Exemplo completo de unidades.** LST = 3 h 20 min = 3 + 20/60 =
3,333333 h. Em graus, vale 50°. Uma estrela com RA = 6 h tem RA = 90°.
Logo H = 50° − 90° = −40°. Esse é o valor que entra em sen(H) e cos(H),
após conversão para radianos.

Uma estrela a leste do meridiano não está obrigatoriamente visível: ela
ainda pode estar abaixo do horizonte. A altitude precisa ser calculada.
Também não se deve substituir H diretamente por azimute; H é medido no
sistema equatorial, enquanto Az é medido no horizonte local.

> **O que o GPS resolve**\
> O GPS fornece a posição terrestre e o instante. O ESP32 transforma
> esse instante em LST e usa o catálogo. Nenhum pedido a um servidor é
> necessário durante o funcionamento.

## 7.4 Da data UTC ao tempo sideral

Esta receita torna a função calculateLST reproduzível. Usaremos UTC como
aproximação de UT1, o tempo ligado à rotação terrestre. Para uma
demonstração com erros instrumentais de graus, essa simplificação
temporal é suficiente; precisão astronômica maior exige modelos e
escalas de tempo mais completos. \[A2\]

Primeiro converta a data gregoriana em **data juliana JD**, uma contagem
contínua de dias que inclui a fração do dia. Aqui JD não significa “dia
do ano”. A contagem muda de inteiro ao meio-dia: 01/01/2000 às 12:00 UTC
corresponde a JD = 2451545,0. \[A3\]

Entrada: Y, M, dia, hora, minuto, segundo em UTC

Se M \<= 2: Y = Y - 1; M = M + 12

A = floor(Y / 100)

B = 2 - A + floor(A / 4)

f = (hora + minuto/60 + segundo/3600) / 24

JD = floor(365.25 \* (Y + 4716))

\+ floor(30.6001 \* (M + 1))

\+ dia + B - 1524.5 + f

floor significa arredondar para baixo. Use divisão real onde for
necessário; em C++, dividir dois inteiros pode descartar a fração.
Valide o calendário antes: mês, dia e ano precisam existir, inclusive em
anos bissextos. A receita destina-se ao calendário gregoriano moderno
usado no projeto.

Depois separe a meia-noite anterior e as horas decorridas. Evite chamar
essas horas de H, pois H já significa ângulo horário.

``` math
{JD}_{0}\  = \ floor(JD\  - \ 0,5)\  + \ 0,5
```

``` math
h_{UT}\  = \ 24(JD\  - \ {JD}_{0})
```

``` math
D_{0}\  = \ {JD}_{0}\  - \ 2451545,0\ \ \ \ \ \ \ T\  = \ \frac{JD\  - \ 2451545,0}{36525}
```

Na expressão seguinte, o resultado está em horas. wrap24 faz a mesma
correção de volta que wrap360, mas com período 24. GMST é o tempo
sideral médio de Greenwich; somar longitude/15 o transforma em tempo
sideral médio local. \[A2\]

``` math
{GMST}_{h}\  = \ wrap24(6,697375\  + \ 0,065709824279\ D_{0}
```

``` math
+ \ 1,0027379\ h_{UT}\  + \ 0,0000258\ T^{2})
```

``` math
{LST}_{h}\  = \ wrap24({GMST}_{h}\  + \ \lambda/15)\ \ \ \ \ \ \ {LST}_{graus}\  = \ 15\ {LST}_{h}
```

**Teste de referência.** Para JD = 2451545,0 e longitude 0°, GMST ≈
18,697374888 h = 280,460623318°. Esse teste verifica o algoritmo; ele
não é o instante fictício do exemplo da seção 9. Na primeira versão,
registre o uso de tempo sideral médio e das aproximações do catálogo.

# 8 Da esfera celeste à fórmula da altitude

## 8.1 Esfera celeste e triângulo P Z S

Imagine uma esfera imensa centrada no observador, com cada estrela
marcada pela direção em que é vista. Essa **esfera celeste** não afirma
que todas as estrelas estejam à mesma distância. Ela é um mapa de
direções: move-se a ponta de uma seta pelo céu, e a ponta imaginária
encontra um ponto da esfera.

Um **círculo máximo** é uma circunferência cujo plano passa pelo centro
da esfera. O equador é um exemplo. Os caminhos mais curtos sobre uma
esfera seguem arcos de círculos máximos. Quando ligamos três pontos por
esses arcos, obtemos um triângulo esférico, diferente de um triângulo
plano.

<figure>
<img src="media/image3.png" style="width:6.25in;height:4.27083in"
alt="03 triangulo esferico" />
<figcaption><p>Figura 3 Esquema de um triângulo esférico. Os lados são
arcos e medem ângulos, não comprimentos sobre uma
folha.</p></figcaption>
</figure>

No nosso triângulo, **P** é o polo norte celeste, **Z** é o zênite do
observador e **S** é a estrela. Usaremos sempre o polo norte, inclusive
quando o observador estiver no hemisfério sul. Isso mantém uma única
convenção para as fórmulas.

O lado PZ liga o polo ao zênite; PS liga o polo à estrela; ZS liga o
zênite à estrela. O ângulo entre os dois arcos que saem de P está
relacionado ao ângulo horário H. A orientação leste/oeste dá sinal a H;
como a lei usada contém cos(H), o resultado da altitude é o mesmo para H
e −H.

> **Não aplicar trigonometria plana diretamente**\
> Os arcos estão sobre uma superfície curva. A soma dos ângulos internos
> de um triângulo esférico não precisa ser 180°. Usaremos a lei dos
> cossenos esférica para preservar essa geometria.

## 8.2 Por que aparecem ângulos de noventa graus

### O lado PZ é a colatitude

A declinação do polo norte celeste é +90°. A declinação do zênite é
igual à latitude φ do observador: a vertical local e o equador celeste
têm a mesma relação angular que a posição terrestre tem com o equador,
na aproximação geométrica adotada. A distância angular entre o polo e o
zênite é, portanto, 90° − φ.

``` math
PZ\  = \ 90{^\circ}\  - \ \varphi
```

No equador, φ = 0° e PZ = 90°. No polo norte terrestre, φ = +90° e PZ =
0°: o zênite coincide com o polo norte celeste. Para φ = −23,55°, PZ =
90° − (−23,55°) = 113,55°. Um arco maior que 90° é perfeitamente
possível. **Não use o valor absoluto da latitude.**

### O lado PS é a distância polar da estrela

A declinação δ da estrela é medida a partir do equador celeste. Do
equador até o polo norte há 90°. Assim, a distância polar da estrela é
90° − δ.

``` math
PS\  = \ 90{^\circ}\  - \ \delta
```

Para uma estrela com δ = +30°, PS = 60°. Para δ = −30°, PS = 120°. O
sinal negativo não desaparece: uma estrela ao sul do equador fica a mais
de 90° do polo norte celeste.

### O lado ZS é a distância zenital

A altitude é medida a partir do horizonte, onde Alt = 0°. O zênite está
a Alt = 90°. O arco entre zênite e estrela é a parte que falta para
completar esses 90°.

``` math
ZS\  = \ 90{^\circ}\  - \ Alt
```

Se Alt = 30°, ZS = 60°. Se a estrela está no zênite, Alt = 90° e ZS =
0°. Se Alt = −10°, ZS = 100°, coerente com uma direção abaixo do
horizonte.

### A lei dos cossenos esférica

Para lados a, b e c e ângulo A oposto ao lado a, a lei diz cos(a) =
cos(b)cos(c) + sen(b)sen(c)cos(A). Aplicando a = ZS, b = PZ, c = PS e A
correspondente a H:

``` math
\cos(ZS)\  = \ \cos(PZ)\cos(PS)\  + \ sen(PZ)sen(PS)\cos(H)
```

Os três lados são ângulos e precisam usar unidades coerentes nas funções
trigonométricas. Na próxima página vamos substituir cada um e
simplificar sem pular a passagem principal. \[A4\]

## 8.3 Derivação da altitude passo a passo

**Passo 1 — escreva a lei para o lado desconhecido.** Queremos Alt; ela
aparece no arco ZS. Então usamos:

``` math
\cos(ZS)\  = \ \cos(PZ)\cos(PS)\  + \ sen(PZ)sen(PS)\cos(H)
```

**Passo 2 — troque os arcos por suas expressões.** Substitua ZS por 90°
− Alt, PZ por 90° − φ e PS por 90° − δ. Para caber na página, a soma
aparece em duas linhas:

``` math
\cos(90{^\circ}\  - \ Alt)\  = \ \cos(90{^\circ}\  - \ \varphi)\cos(90{^\circ}\  - \ \delta)
```

``` math
+ \ sen(90{^\circ}\  - \ \varphi)sen(90{^\circ}\  - \ \delta)\cos(H)
```

**Passo 3 — reconheça as identidades complementares.** No círculo
trigonométrico, seno e cosseno trocam de papel quando substituímos um
ângulo por seu complemento. As identidades abaixo continuam válidas para
os ângulos negativos usados aqui.

``` math
\cos(90{^\circ}\  - \ x)\  = \ sen(x)\ \ \ \ \ \ \ sen(90{^\circ}\  - \ x)\  = \ \cos(x)
```

**Passo 4 — substitua termo por termo.** O lado esquerdo vira sen(Alt).
O primeiro produto vira sen(φ)sen(δ). O segundo produto vira
cos(φ)cos(δ)cos(H). Logo:

``` math
sen(Alt)\  = \ sen(\delta)sen(\varphi)\  + \ \cos(\delta)\cos(\varphi)\cos(H)
```

Essa é a fórmula procurada. Sua estrutura combina a latitude do
observador, a declinação do astro e a distância angular ao meridiano. A
mesma expressão é apresentada pela USNO para a transformação horizontal.
\[A1\]

**Passo 5 — calcule um número antes de extrair o ângulo.** Chame o lado
direito de U. Ele é também a componente vertical do vetor unitário da
estrela. Depois aplique a função inversa do seno:

``` math
U\  = \ sen(\delta)sen(\varphi)\  + \ \cos(\delta)\cos(\varphi)\cos(H)
```

``` math
Alt\  = \ arcsen(clamp(U,\  - 1,\  + 1))
```

clamp limita um valor ao intervalo indicado. Isso evita que um
arredondamento minúsculo, como U = 1,0000000001, cause erro de domínio
em asin. Porém, um erro grande, como U = 1,2, deve ser investigado; não
use o limite para esconder dados ou fórmulas erradas.

**Por que o arcsen não deixa uma ambiguidade aqui?** A altitude física
está no intervalo −90° a +90°. Esse é exatamente o intervalo principal
devolvido pelo arcsen. Para azimute, o círculo inteiro importa e
precisamos de atan2, que veremos a seguir.

## 8.4 Derivação das componentes e do azimute

Para descobrir o quadrante, vamos transformar a direção da estrela em
três componentes locais. Na base equatorial associada ao meridiano,
podemos representar a direção por X = cos(δ)cos(H), Y = cos(δ)sen(H) e Z
= sen(δ). O eixo Y cresce para oeste, porque H positivo aponta para
oeste.

**Primeiro, obtenha leste.** Como leste é o sentido oposto de Y, E = −Y.
Depois gire os eixos X e Z para a latitude do observador. A direção
vertical possui componentes (cos φ, 0, sen φ), e a direção norte possui
(−sen φ, 0, cos φ). Projetar o vetor sobre cada uma significa
multiplicar as componentes correspondentes e somar:

``` math
E\  = \  - \cos(\delta)sen(H)
```

``` math
N\  = \ sen(\delta)\cos(\varphi)\  - \ \cos(\delta)\cos(H)sen(\varphi)
```

``` math
U\  = \ sen(\delta)sen(\varphi)\  + \ \cos(\delta)\cos(H)\cos(\varphi)
```

Observe que U é a expressão já derivada pelo triângulo esférico. Duas
formas de enxergar o mesmo problema chegaram ao mesmo resultado. Também
devemos ter E² + N² + U² ≈ 1.

Uma direção descrita por Alt e Az possui projeção horizontal de tamanho
cos(Alt). Como o azimute começa no norte, suas componentes horizontais
são:

``` math
E\  = \ \cos(Alt)sen(Az)\ \ \ \ \ \ \ N\  = \ \cos(Alt)\cos(Az)
```

Assim, atan2(E, N) recupera Az com o quadrante correto. Na programação:

``` math
Az\  = \ wrap360(graus(atan2(E,\ N)))
```

| **Sinal de E** | **Sinal de N** | **Região** | **Azimute** |
|----------------|----------------|------------|-------------|
| Positivo       | Positivo       | Nordeste   | 0° a 90°    |
| Positivo       | Negativo       | Sudeste    | 90° a 180°  |
| Negativo       | Negativo       | Sudoeste   | 180° a 270° |
| Negativo       | Positivo       | Noroeste   | 270° a 360° |

**Teste rápido.** No equador, uma estrela no equador celeste com H =
−90° tem E = +1, N = 0 e U = 0. Portanto Alt = 0° e Az = 90°: nasce a
leste no modelo geométrico.

Se E² + N² for praticamente zero, a direção está no zênite ou no nadir.
Nesse caso, sinalize azimute indefinido. Perto do zênite, pequenas
mudanças espaciais podem mudar muito Az; evite instruções horizontais
instáveis.

# 9 Exemplo numérico completo

## 9.1 Entradas e cálculo da altitude

Usaremos uma estrela fictícia para conferir a matemática sem depender de
uma data específica. A latitude é plausível para o sudeste do Brasil. O
LST abaixo é um dado escolhido para o exercício, não calculado de um
horário real. No firmware ele virá de UTC e longitude.

| **Grandeza**     | **Valor**        | **Significado**                 |
|------------------|------------------|---------------------------------|
| Latitude φ       | −23,55°          | Observador ao sul do equador    |
| Declinação δ     | −30°             | Astro ao sul do equador celeste |
| RA               | 6 h = 90°        | Coordenada do catálogo fictício |
| LST              | 3 h 20 min = 50° | Estado escolhido do céu         |
| Ângulo horário H | 50° − 90° = −40° | A leste do meridiano            |

**1. Converta os ângulos.** Multiplicando cada valor por π/180, obtemos
φ ≈ −0,411025 rad; δ ≈ −0,523599 rad; H ≈ −0,698132 rad. Usaremos mais
casas no cálculo do que na apresentação.

**2. Calcule os senos e cossenos.**

| **Função**      | **Resultado aproximado** |
|-----------------|--------------------------|
| sen(φ) e cos(φ) | −0,399549 e +0,916712    |
| sen(δ) e cos(δ) | −0,500000 e +0,866025    |
| sen(H) e cos(H) | −0,642788 e +0,766044    |

**3. Calcule os produtos separadamente.** Isso permite encontrar erros
de sinal com facilidade.

``` math
sen(\delta)sen(\varphi)\  = \ ( - 0,500000)( - 0,399549)\  \approx \ 0,199775
```

``` math
\cos(\delta)\cos(\varphi)\cos(H)\  \approx \ 0,608159
```

**4. Some os produtos.**

``` math
U\  \approx \ 0,199775\  + \ 0,608159\  = \ 0,807934
```

**5. Recupere o ângulo.**

``` math
Alt\  = \ arcsen(0,807934)\  \approx \ 53,894562{^\circ}
```

O valor é positivo, então a estrela está acima do horizonte geométrico.
Está mais alta que 45° e ainda distante do zênite, o que ajuda na
estabilidade do teste. O arredondamento indicado para exibir o exemplo é
**Alt = 53,89°**. Essas casas decimais mostram a conta; não significam
que o protótipo terá precisão de centésimos de grau.

## 9.2 Azimute e orientação de movimento

**6. Calcule as componentes horizontais.** Use os mesmos valores
trigonométricos da página anterior.

``` math
E\  = \  - (0,866025)( - 0,642788)\  \approx \  + 0,556670
```

``` math
N\  = \ ( - 0,500000)(0,916712)
```

``` math
- \ (0,866025)(0,766044)( - 0,399549)
```

``` math
N\  \approx \  - 0,458356\  - \ ( - 0,265067)\  = \  - 0,193289
```

E positivo indica leste; N negativo indica sul. A estrela está no
quadrante sudeste, portanto esperamos um azimute entre 90° e 180°. Essa
previsão é uma maneira simples de conferir a conta.

``` math
Az\  = \ graus(atan2(0,556670,\  - 0,193289))\  \approx \ 109,148232{^\circ}
```

**7. Compare com a direção do aparelho.** Suponha que o BNO085, após
transformação dos eixos, alinhamento da seta e correção para norte
verdadeiro, indique Az = 95° e Alt = 48°.

``` math
\Delta Az\  = \ wrap180(109,148232{^\circ}\  - \ 95{^\circ})\  = \  + 14,148232{^\circ}
```

``` math
\Delta Alt\  = \ 53,894562{^\circ}\  - \ 48{^\circ}\  = \  + 5,894562{^\circ}
```

> **Instrução final do exemplo**\
> Mova 14,15° para a direita e 5,89° para cima. “Direita” significa
> aumentar o azimute ao girar no horizonte; “cima” significa aumentar a
> elevação. Recalcule durante o movimento.

Essas diferenças são ajustes de duas coordenadas. Não some seus módulos
para afirmar que a separação total é 20,04°. A distância angular real
entre duas direções é o arco mais curto sobre a esfera. Ela vem do
produto escalar dos vetores unitários:

``` math
c\  = \ sen({Alt}_{astro})sen({Alt}_{aparelho})
```

``` math
+ \ \cos({Alt}_{astro})\cos({Alt}_{aparelho})\cos({Az}_{astro}\  - \ {Az}_{aparelho})
```

``` math
\gamma\  = \ \arccos(clamp(c,\  - 1,\  + 1))
```

Neste exemplo, c ≈ 0,982752088 e **γ ≈ 10,656930°**. É esse γ que convém
usar para decidir se o apontamento entrou em uma tolerância espacial.
Para uma implementação mais robusta muito perto de zero, pode-se
calcular γ = atan2(módulo do produto vetorial, produto escalar).

Uma tolerância experimental de 5°, por exemplo, seria uma meta inicial a
medir, não uma promessa. A instrução de movimento só deve ser exibida
quando hora, posição e orientação estiverem válidas e recentes, e quando
o alvo estiver em uma condição observável definida pelo projeto.

# 10 Arquitetura do software

## 10.1 Arquivos responsabilidades e contratos

O firmware pode ser organizado em quatro módulos principais e um pequeno
catálogo. As funções abaixo são **contratos de projeto**, não uma
implementação completa pronta para gravar. Separar leitura de sensores
de matemática permite testar as contas mesmo antes da chegada dos
componentes.

| **Arquivo** | **Funções principais** | **Responsabilidade** |
|----|----|----|
| gps.cpp e gps.h | beginGPS, readGPS | Consumir NMEA, validar UTC e posição, registrar idade |
| imu.cpp e imu.h | beginIMU, getOrientation | Ler BNO085, tratar reset, converter quaternion e eixos |
| astronomy.cpp e astronomy.h | calculateLST, equatorialToHorizontal, angularDifference | Datas, ângulos, vetores e comparação |
| main.cpp | setup, loop | Coordenar estados, selecionar estrela e imprimir |
| catalog.h | lista de Star | Nome, RA, Dec e época de 5 a 10 estrelas |
| config.h opcional | pinos e parâmetros | D magnética, vetor da seta, limites de idade |

struct GPSData {

double latitudeDeg, longitudeDeg;

UTCDateTime utc;

bool positionValid, timeValid, dateValid;

uint32_t positionAgeMs, utcAgeMs;

};

struct Orientation {

double azTrueDeg, altDeg;

bool valid, azDefined;

uint8_t quality;

uint32_t ageMs;

};

struct Horizontal {

double azDeg, altDeg;

bool azDefined;

};

struct Guidance {

double deltaAzDeg, deltaAltDeg, separationDeg;

bool horizontalGuidanceValid;

};

UTCDateTime é um tipo a definir com ano, mês, dia, hora, minuto e
segundo. O campo utcAgeMs deve refletir uma data/hora coerente, não uma
mistura de mensagens de instantes diferentes. O parser precisa atualizar
todos os bytes recebidos, mesmo quando não existe fix.

**Contrato de unidades.** Interfaces externas usam graus decimais, UTC e
milissegundos. As funções trigonométricas trabalham em radianos
internamente. Declinação magnética e declinação celeste devem ter nomes
diferentes, por exemplo magneticDeclinationDeg e star.decDeg.

Não inicialize Wi-Fi ou Bluetooth. As bibliotecas e o pacote da placa
podem ser instalados antes do ensaio; o programa opera sem conexão de
rede. Registre as versões instaladas para repetir o teste depois.

## 10.2 Ordem de execução e estados

setup():

iniciar console Serial a 115200

carregar pinos, catalogo e D magnetica

iniciar SPI e BNO085

solicitar SH2_ROTATION_VECTOR

iniciar UART do GPS

estado = AGUARDANDO_DADOS

loop():

readGPS() // consumir todos os bytes disponiveis

getOrientation() // atualizar somente com novo evento

se BNO reiniciou: solicitar novamente seus relatorios

processar selecao de estrela pelo console

se GPS invalido ou UTC/posicao com idade \> 2000 ms:

estado = AGUARDANDO_GPS

senao se IMU invalida ou idade \> 200 ms:

estado = AGUARDANDO_IMU

senao se referencia de norte/eixos nao validada:

estado = ALINHAMENTO_PENDENTE

senao:

lst = calculateLST(gps.utc, gps.longitudeDeg)

alvo = equatorialToHorizontal(estrela, lst,

gps.latitudeDeg)

se alvo.altDeg \< 0:

estado = ALVO_ABAIXO_DO_HORIZONTE

senao se azimute indefinido ou perto do zenite:

estado = GUIA_HORIZONTAL_INDISPONIVEL

senao:

guia = angularDifference(alvo, aparelho)

estado = PRONTO

a cada 200 ms: imprimir estado e dados com suas idades

// retornar rapidamente; evitar esperas bloqueantes

Os limites de 2 s para GPS e 200 ms para IMU são escolhas iniciais do
projeto, a ajustar às taxas reais. Uma referência possível é GPS a 1 Hz,
IMU a 20–50 Hz e impressão a 5 Hz. Imprimir mais rápido que a leitura
não cria informação nova. Um agendador simples baseado em millis pode
controlar a impressão sem longos delay.

**Hora entre mensagens.** Na primeira implementação, pode-se recalcular
Alt/Az ao receber uma nova solução UTC e conservar o resultado até a
próxima, sempre exibindo a idade. Se extrapolar a hora, some apenas o
tempo monotônico decorrido desde a última UTC válida e limite o prazo.
Não renove a idade de uma informação só porque ela foi impressa.

> **Falha deve aparecer como estado**\
> Se o GPS perder validade, não substitua latitude e longitude por zero
> nem continue emitindo “mova” como se nada tivesse ocorrido. Preserve
> os diagnósticos e suspenda a orientação até recuperar dados adequados.

## 10.3 Pseudocódigo da matemática e saída Serial

equatorialToHorizontal(star, lstDeg, latDeg):

H = rad(wrap180(lstDeg - star.raDeg))

phi = rad(latDeg)

dec = rad(star.decDeg)

E = -cos(dec) \* sin(H)

N = sin(dec)\*cos(phi) - cos(dec)\*cos(H)\*sin(phi)

U = sin(dec)\*sin(phi) + cos(dec)\*cos(H)\*cos(phi)

altDeg = deg(asin(clamp(U, -1.0, 1.0)))

azDefined = (E\*E + N\*N \> epsilon)

azDeg = wrap360(deg(atan2(E, N))) se azDefined

retornar Horizontal(azDeg, altDeg, azDefined)

angularDifference(alvo, aparelho):

dAlt = alvo.altDeg - aparelho.altDeg

dAz = wrap180(alvo.azDeg - aparelho.azTrueDeg)

a = rad(alvo.altDeg); b = rad(aparelho.altDeg)

d = rad(alvo.azDeg - aparelho.azTrueDeg)

c = sin(a)\*sin(b) + cos(a)\*cos(b)\*cos(d)

sep = deg(acos(clamp(c, -1.0, 1.0)))

retornar Guidance(dAz, dAlt, sep, azimutesValidos)

epsilon é um pequeno limite numérico para detectar a direção vertical;
não é a tolerância física de apontamento. Use, por exemplo, 1e-12 na
norma horizontal ao quadrado e uma regra prática separada para
restringir o guia perto do zênite, como Alt acima de 85°. Se algum
azimute estiver indefinido, calcule a separação diretamente dos vetores
e não use dAz.

**Saída correspondente ao exemplo fictício da seção 9:**

MODO=SIMULACAO estrela=EXEMPLO LST_deg=50.0000

MODELO=GEOMETRICO norte=VERDADEIRO

alvo: Alt=53.8946 Az=109.1482

aparelho: Alt=48.0000 Az=95.0000

erro: dAlt=+5.8946 dAz=+14.1482 separacao=10.6569

GUIA: direita 14.15 graus; cima 5.89 graus

Na execução real, acrescente UTC completa, latitude, longitude,
nome/época do catálogo, status do fix, idades do GPS e IMU, qualidade do
BNO085 e D magnética configurada. Diferencie claramente SIMULACAO de
MEDICAO. Um registro de simulação verifica o cálculo; não comprova que
os sensores estão funcionando.

Ao entrar na tolerância escolhida, imprima “dentro da tolerância de X°”,
usando a separação angular. Se usar uma pequena zona morta para impedir
alternância rápida entre direita/esquerda, documente esse valor e
preserve os números brutos no registro de testes.

# 11 Integração e checklist das oito metas

A integração começa quando os testes individuais dos sensores já
funcionam. Fixe os módulos no suporte, marque a seta de apontamento e
mantenha o mesmo alinhamento durante os ensaios. Confira novamente os
fios com a alimentação desligada. Depois, conecte o USB e observe
primeiro os estados de inicialização, sem emitir instruções de movimento
prematuramente.

O programa deve continuar recebendo GPS e IMU enquanto calcula e
imprime. Use atualizações periódicas, evitando pausas longas. Uma
mensagem de erro precisa indicar qual requisito falta: posição, hora,
orientação recente, calibração ou alvo acima do horizonte. Não substitua
uma falha por valores numéricos aparentemente normais.

| **Meta técnica** | **O que executar** | **Evidência para marcar como concluída** |
|----|----|----|
| ☐ 1. Ler BNO085 | Receber rotation vector e status. | 60 s de amostras recentes; movimentos corretos nos eixos e recuperação após reset. |
| ☐ 2. Ler GPS | Receber e interpretar NMEA. | Bytes chegam; mensagens passam pelo checksum; solução aparece sob céu aberto. |
| ☐ 3. Validar hora/posição | Conferir UTC, data, latitude e longitude. | Data real, sinais corretos e idade dentro do limite configurado; perda é sinalizada. |
| ☐ 4. Cadastrar estrelas | Gravar manualmente 5–10 alvos. | Nome, RA, Dec e época conferidos; conversão horas–graus testada. |
| ☐ 5. Calcular Alt/Az | Executar LST e transformação. | Exemplo numérico e casos da próxima página aprovados; horizonte tratado. |
| ☐ 6. Comparar orientação | Converter a seta para norte verdadeiro. | Diferenças com sinal correto; cruzamento 359°/0° funciona; zênite tratado. |
| ☐ 7. Imprimir no Serial | Exibir estados, valores e instrução. | Registro legível e atualizado, incluindo idades, unidades e estrela selecionada. |
| ☐ 8. Testar no céu real | Repetir apontamentos de estrelas conhecidas. | Ficha preenchida, erros medidos e resultado comparado à meta experimental. |

## 11.1 Condição de aceitação da integração

O conjunto só está pronto para o primeiro ensaio quando as metas 1–7
foram verificadas e as falhas foram provocadas de forma controlada.
Interrompa a recepção GPS e confirme a expiração; reinicie a IMU e
confirme nova solicitação dos relatórios. Refaça a ligação apenas com a
alimentação desligada. Estados de validade e idade devem ser observados
separadamente. \[R9, R15\]

> **Resultado esperado nesta etapa**\
> Um console honesto: informa “aguardando dados” quando necessário e
> fornece orientação somente quando os requisitos estão satisfeitos.
> Registrar uma falha ajuda a melhorar o projeto; ocultá-la impede a
> validação.

## 11.2 Testes matemáticos e observação do céu

Antes de comparar sensores com estrelas, teste as contas com entradas
conhecidas. Assim, um problema elétrico ou magnético não será confundido
com um erro de sinal na trigonometria. Os resultados abaixo usam azimute
norte=0°, leste=90° e H positivo para oeste. Tolerância numérica
sugerida: 0,001° nos casos não singulares, com cálculos em double.

| **Entrada de teste**               | **Resultado esperado**                 |
|------------------------------------|----------------------------------------|
| φ=0°; δ=0°; H=−90°                 | Alt=0°; Az=90°: horizonte leste.       |
| φ=0°; δ=0°; H=+90°                 | Alt=0°; Az=270°: horizonte oeste.      |
| φ=−23,55°; δ=−23,55°; H=0°         | Alt=90°; azimute indefinido no zênite. |
| φ=−23,55°; δ=−30°; H=0°            | Alt=83,55°; Az=180°.                   |
| φ=−23,55°; δ=0°; H=0°              | Alt=66,45°; Az=0°.                     |
| Alvo Az=2°; aparelho Az=358°       | ΔAz=+4°: girar à direita.              |
| Direções idênticas; depois opostas | Separação 0°; depois 180°.             |
| 2000-01-01, 12:00:00 UTC           | JD=2451545,0; GMST≈18,697375 h.        |

## Ensaio com uma estrela identificada

**1. Prepare o local.** Escolha céu aberto, apoio firme e distância do
notebook, ímãs e peças de aço. Registre a declinação magnética utilizada
e verifique hora/posição. Selecione uma estrela conhecida,
aproximadamente entre 20° e 70° de altitude, evitando horizonte e zênite
no primeiro ensaio.

**2. Confira a previsão.** Compare Alt/Az com uma referência
independente configurada para o mesmo local e instante. Registre se ela
usa refração e coordenadas da data. Nosso cálculo simplificado com
catálogo J2000 pode apresentar diferença sistemática; não altere os
sensores para esconder esse efeito. \[A1, A5\]

**3. Aponte e registre.** Alinhe visualmente a seta à estrela, salve
cinco leituras espaçadas e afaste o apontamento. Repita três vezes. Faça
isso com três estrelas em regiões diferentes, quando disponíveis.
Registre ΔAz, ΔAlt e a separação esférica γ.

**4. Avalie.** Use como primeira meta experimental mediana de γ≤5° e
registre também o maior erro. Esse valor é um critério de
desenvolvimento, não garantia do BNO085. A incerteza do apontamento
visual também participa do resultado. Se não atingir a meta, investigue
os erros antes de apertar a tolerância.

☐ Arquivar entradas, versão do programa, condições do teste e
resultados; separar falhas de cálculo, calibração e alinhamento
mecânico.

# 12 Problemas comuns e limites da versão

Investigue uma causa por vez e anote a mudança feita. Volte ao teste
individual quando a origem da falha não estiver clara. Uma leitura
estável pode estar errada; estabilidade, validade e exatidão são
propriedades diferentes.

| **Sintoma** | **Causa provável** | **Verificação e ação** |
|----|----|----|
| Sem console ou caracteres ilegíveis | Cabo sem dados, porta incorreta ou configuração serial. | Testar mensagem simples; conferir USB/porta e velocidade do monitor. |
| BNO085 não inicia | Alimentação, seleção de interface ou sinais SPI. | Conferir esquema, PS0/PS1, CS, INT, reset e terra comum; não trocar fios energizados. |
| IMU trava após funcionar | Reset ou comunicação instável. | Marcar reset, solicitar relatórios novamente e revisar fios; I²C tem restrições nesta combinação. \[R2, R9\] |
| GPS transmite, mas não fixa | Antena obstruída ou recepção insuficiente. | Ensaiar sob céu aberto; distinguir chegada de NMEA de solução válida. |
| GPS mostra local/data absurdos | Sinal, unidade, configuração ou data antiga. | Conferir hemisférios, UTC e data real; investigar hardware antigo e rollover. |
| Direção deslocada por muitos graus | Norte magnético, eixos ou montagem. | Conferir declinação magnética, sentido do vetor e seta física; não ajustar números ao acaso. |
| Direção muda perto do computador | Interferência magnética local. | Afastar sensor, comparar locais e recalibrar no ambiente de uso. |
| Estrela calculada no lado errado | H invertido, longitude errada ou quadrante. | Conferir graus/radianos e atan2(E,N); executar os casos de teste. |
| Sistema continua orientando sem GPS | Dado válido, porém antigo. | Verificar idade de cada campo; bloquear instruções expiradas. \[R15\] |
| Azimute salta perto de 90° de Alt | Singularidade próxima do zênite. | Usar separação vetorial; sinalizar que o azimute fica indefinido no zênite. |

## O que limita a precisão

**Astronomia:** a versão didática usa aproximações. Precessão altera a
relação do catálogo J2000 com a data; refração desvia a direção
aparente, principalmente perto do horizonte. Nutação, aberração e
pequenas diferenças de escala de tempo pertencem a um aperfeiçoamento
posterior. Compare referências com essas opções registradas. \[A1, A2,
A5\]

**Medição:** calibração incompleta, materiais magnéticos, flexão do
suporte, movimentos bruscos e alinhamento visual acrescentam erros. O
status de precisão do sensor não certifica o erro final do aparelho.

> **Regra de diagnóstico**\
> Se a diferença varia com a direção, suspeite de distorção magnética ou
> transformação dos eixos. Se cresce com o tempo ou muda após
> reconectar, examine timestamps, resets e validade antes de mexer na
> fórmula astronômica.

# 13 Próximos passos e ficha de ensaio

O próximo passo concreto é comprar placas documentadas, conferir
alimentação e interfaces e completar a meta 1. Avance pela sequência da
seção 11, mantendo uma versão funcional de cada teste individual. A
primeira versão termina quando o conjunto demonstra orientação no céu,
com erros registrados e comportamento correto diante de dados inválidos.

Antes de acrescentar recursos, melhore o que já existe: organização dos
fios, suporte rígido, alinhamento da seta, qualidade dos registros e
repetibilidade. Depois, aperfeiçoe a compensação magnética e torne o
tratamento das coordenadas da data mais rigoroso. Um catálogo maior só é
útil depois que os primeiros alvos funcionarem de forma compreensível.

## 13.1 Ficha para copiar a cada sessão

| **Campo** | **Preenchimento do ensaio** |
|----|----|
| Identificação | Data: \_\_\_\_\_\_\_\_\_\_ Responsável: \_\_\_\_\_\_\_\_\_\_ Ensaio nº: \_\_\_\_\_\_ |
| Configuração | Placas/revisões: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Programa/versão: \_\_\_\_\_\_ |
| Local e tempo | Latitude: \_\_\_\_\_\_ Longitude: \_\_\_\_\_\_ UTC inicial: \_\_\_\_\_\_\_\_\_\_ |
| Referência magnética | D: \_\_\_\_\_\_° Fonte/data: \_\_\_\_\_\_\_\_\_\_ Calibração: \_\_\_\_\_\_\_\_\_\_ |
| Alvo e catálogo | Estrela: \_\_\_\_\_\_\_\_\_\_ RA: \_\_\_\_\_\_ Dec: \_\_\_\_\_\_ Época: \_\_\_\_\_\_ |
| Qualidade dos dados | GPS/fix: \_\_\_\_\_\_ Idades: \_\_\_\_\_\_ IMU/status: \_\_\_\_\_\_\_\_\_\_\_\_\_\_ |
| Comparação | Alt/Az previstos: \_\_\_\_\_\_\_\_\_\_ Alt/Az do aparelho: \_\_\_\_\_\_\_\_\_\_ |
| Resultado | ΔAz: \_\_\_\_\_\_° ΔAlt: \_\_\_\_\_\_° Separação γ: \_\_\_\_\_\_° |
| Repetições | Leituras arquivadas: \_\_\_\_\_\_ Mediana: \_\_\_\_\_\_° Máximo: \_\_\_\_° |
| Ambiente e conclusão | Interferências/obstruções: \_\_\_\_\_\_\_\_\_\_ Próxima ação: \_\_\_\_\_\_ |

## 13.2 Decisão após cada teste

**Aprovado:** critérios atingidos e registros completos; prosseguir para
a próxima meta. **Repetir:** resultado inconsistente; repetir nas mesmas
condições antes de alterar o projeto. **Corrigir:** causa identificada;
fazer uma alteração e executar novamente o teste afetado. Evite mudar
simultaneamente biblioteca, pinagem, calibração e fórmula: depois
ficaria difícil descobrir o que resolveu o problema.

Ao comparar sessões, mantenha o mesmo critério de separação angular e a
mesma forma de apontamento. Identifique os registros em que o GPS estava
expirado ou a IMU reiniciou; eles não devem entrar como medições normais
da precisão. Preserve também esses eventos para avaliar a confiabilidade
do sistema.

> **Conclusão da fase 1**\
> O entregável é um protótipo com três módulos principais, energia e
> console pelo USB, pequeno catálogo local e validação no céu. Display,
> bateria, Wi-Fi e serviços hospedados ficam fora desta montagem; só
> entram em uma etapa futura definida após a avaliação dos resultados.

# 14 Referências técnicas

As fontes abaixo permitem conferir especificações e aprofundar os temas.
Os esquemas da placa efetivamente comprada prevalecem sobre uma pinagem
genérica. Data de consulta: 16 de setembro de 2026. As referências de
bibliotecas apontam para projetos que podem receber atualizações;
registre a versão usada nos ensaios.

**\[R1\] CEVA — BNO08X Data Sheet.** Alimentação, interfaces e
relatórios de orientação.

https://www.ceva-ip.com/wp-content/uploads/BNO080_085-Datasheet.pdf

**\[R2\] Adafruit — BNO085 Orientation IMU.** Guia de ligação e
restrições de I²C.

https://learn.adafruit.com/adafruit-9-dof-orientation-imu-fusion-breakout-bno085?view=all

**\[R3\] u-blox — NEO-6 Data Sheet.** Características elétricas do
módulo NEO-6.

https://content.u-blox.com/sites/default/files/products/documents/NEO-6_DataSheet\_(GPS.G6-HW-09005).pdf

**\[R4\] u-blox — NEO-M8 Data Sheet.** Características do módulo e
interfaces.

https://content.u-blox.com/sites/default/files/NEO-M8-FW3_DataSheet_UBX-15031086.pdf

**\[R5\] Espressif — ESP32-S3 Series Data Sheet.** GPIO, alimentação e
limitações.

https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf

**\[R6\] Adafruit — Adafruit_BNO08x.cpp.** Implementação da biblioteca e
SPI.

https://github.com/adafruit/Adafruit_BNO08x/blob/master/src/Adafruit_BNO08x.cpp

**\[R7\] Espressif — Arduino ESP32 Serial API.** Configuração de UART.

https://docs.espressif.com/projects/arduino-esp32/en/latest/api/serial.html

**\[R8\] Espressif — ESP32-S3-DevKitC-1 v1.0.** Guia da placa de
referência.

https://documentation.espressif.com/esp-dev-kits/en/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.0.html

**\[R9\] Adafruit — Exemplo rotation_vector.** Eventos, escolha do
relatório e resets.

https://github.com/adafruit/Adafruit_BNO08x/blob/master/examples/rotation_vector/rotation_vector.ino

**\[R10\] CEVA — SH-2 Reference Manual.** Formatos e estados dos
relatórios.

https://www.ceva-ip.com/wp-content/uploads/2019/10/SH-2-Reference-Manual.pdf

**\[R11\] Adafruit — sh2_SensorValue.h.** Estruturas dos valores
recebidos.

https://github.com/adafruit/Adafruit_BNO08x/blob/master/src/sh2_SensorValue.h

**\[R12\] CEVA — BNO080/BNO085 Sensor Calibration Procedure.** Leitura
complementar; link oficial localizado, conteúdo não integralmente
verificado nesta edição.

https://www.ceva-ip.com/wp-content/uploads/2019/09/BNO080-BNO085-Sesnor-Calibration-Procedure.pdf

**\[R13\] NOAA/NCEI — Magnetic Declination.** Declinação magnética e
modelos.

https://www.ncei.noaa.gov/products/magnetic-declination

**\[R14\] Mikal Hart — TinyGPSPlus.** Projeto e exemplos do
interpretador NMEA.

https://github.com/mikalhart/TinyGPSPlus

**\[R15\] Mikal Hart — TinyGPS++.h.** Validade, atualização e idade dos
campos.

https://github.com/mikalhart/TinyGPSPlus/blob/master/src/TinyGPS%2B%2B.h

**\[R16\] u-blox — u-blox 6 Receiver Description / Protocol
Specification.** Mensagens e protocolo.

https://content.u-blox.com/sites/default/files/products/documents/u-blox6_ReceiverDescrProtSpec\_%28GPS.G6-SW-10018%29_Public.pdf

## 14.1 Astronomia, matemática e catálogo

**\[A1\] USNO — Computing Altitude and Azimuth from Greenwich Apparent
Sidereal Time.** Relação entre coordenadas equatoriais, posição do
observador e coordenadas horizontais; atenção à consistência entre
coordenadas médias/aparentes e tempo sideral.

https://aa.usno.navy.mil/faq/alt_az

**\[A2\] USNO — Computing Approximate Sidereal Time.** Fórmulas de
GMST/GAST e longitude com sinal positivo para leste.

https://aa.usno.navy.mil/faq/GAST

**\[A3\] USNO — Converting Between Julian Dates and Gregorian Calendar
Dates.** Definição da data juliana e conversão de calendário.

https://aa.usno.navy.mil/faq/JD_formula

**\[A4\] NASA NTRS — Geometry and Trigonometry Related to the Sphere.**
Trigonometria esférica e lei dos cossenos.

https://ntrs.nasa.gov/api/citations/19750065070/downloads/19750065070.pdf

**\[A5\] Astropy — AltAz.** Convenção de azimute a partir do norte para
leste e configuração dos efeitos de refração.

https://docs.astropy.org/en/stable/api/astropy.coordinates.AltAz.html

**\[A6\] CDS/SIMBAD — Consultas das oito estrelas do catálogo inicial.**
Coordenadas ICRS na época J2000; conversões decimais e arredondamentos
feitos para este manual. Cada endereço identifica o objeto consultado.

Sirius: https://simbad.cds.unistra.fr/simbad/sim-basic?Ident=Sirius

Canopus: https://simbad.cds.unistra.fr/simbad/sim-id?Ident=canopus

Alfa Centauri:
https://simbad.u-strasbg.fr/simbad/sim-basic?Ident=alpha+centauri&submit=SIMBAD+search

Rigel: https://simbad.cds.unistra.fr/simbad/sim-basic?Ident=Rigel

Antares: https://simbad.cds.unistra.fr/simbad/sim-basic?Ident=Antares

Spica: https://simbad.cds.unistra.fr/simbad/sim-basic?Ident=Spica

Aldebaran:
https://simbad.cds.unistra.fr/simbad/sim-basic?Ident=Aldebaran

Achernar: https://simbad.cds.unistra.fr/simbad/sim-id?Ident=Achernar

## Como conferir uma alteração no projeto

Consulte primeiro a fonte responsável pelo dado que será alterado. Para
alimentação e pinagem, use o fabricante e o esquema do breakout. Para o
significado de um relatório do sensor, use CEVA e a biblioteca
instalada. Para os campos GPS, use o protocolo u-blox e o interpretador
NMEA. Para coordenadas de estrelas, mantenha nome, referencial e época
junto dos números.

As explicações passo a passo, os exemplos fictícios, os diagramas e os
critérios de ensaio deste manual são material didático do projeto. Eles
não substituem os limites elétricos dos componentes nem representam uma
certificação de desempenho. O caminho de validação proposto permite
descobrir a precisão real do conjunto comprado e montado.

> **Referência magnética**\
> A documentação NOAA já consta em \[R13\]. Ela deve ser usada com lugar
> e data do ensaio; não é necessário repetir uma consulta à internet
> durante o funcionamento offline do protótipo.
