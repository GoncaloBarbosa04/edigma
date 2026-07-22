# Protótipo MUPI — TUB

Protótipo navegável do ponto de informação digital para paragens dos Transportes
Urbanos de Braga. Ficheiro único, sem dependências, sem passo de compilação.

## Como correr

Duplo clique em `index.html`. É tudo.

Para ver como fica no equipamento real, em modo kiosk:

```
# Windows
chrome.exe --kiosk --app="file:///C:/caminho/para/index.html"

# macOS
open -a "Google Chrome" --args --kiosk --app="file:///caminho/index.html"

# Linux
google-chrome --kiosk --app="file:///caminho/index.html"
```

Numa demonstração com o monitor em retrato, roda o ecrã pelas definições do
sistema operativo — o protótipo ajusta-se sozinho à janela.

## Painel de desenvolvimento

Tecla **D**. Permite forçar qualquer um dos seis estados operacionais sem
esperar pelos temporizadores, disparar um aviso prioritário e alternar
contraste e idioma. Serve para demonstrações: mostras os seis estados em
trinta segundos em vez de esperar que aconteçam.

## Mapa do ficheiro

O `index.html` está dividido em cinco secções assinaladas em comentário:

| Secção | O que contém | Quando mexer |
|---|---|---|
| `[1] TOKENS` | Cores, tipografia, espaçamentos, tema de alto contraste | Ao definir a identidade visual |
| `[2] DADOS` | Paragem, chegadas, tarifário, horários, campanhas | Ao ligar a dados reais |
| `[3] ESTADO` | Máquina de estados e temporizadores de inatividade | Ao afinar o comportamento |
| `[4] ECRÃS` | Uma função por ecrã, cada uma devolve HTML | Ao desenhar interfaces |
| `[5] ARRANQUE` | Eventos, temporizadores, escala, painel de dev | Raramente |

## Estados operacionais

Sete, selecionáveis pelo painel de desenvolvimento (tecla D):

| Estado | Situação | Conteúdo |
|---|---|---|
| Inativo | Sem interação | Publicidade, com as duas próximas chegadas sempre visíveis |
| Ativo | Passageiro a interagir | Informação de transporte |
| Prioridade | Aviso grave por ler | Aviso sobreposto, ocupa o ecrã |
| Levantada | Aviso já lido | Interface normal, com faixa vermelha na zona de avisos |
| Degradado | Sem tempo real | Horário planeado, com aviso do que mudou |
| Offline | Sem conectividade | Última informação em cache, com hora da receção |
| Manutenção | Fora de serviço | Ecrã institucional com alternativas |

Os dois estados de aviso formam um par: *prioridade* interrompe, *levantada*
acompanha. O passageiro passa do primeiro para o segundo ao tocar no ecrã, e
volta ao primeiro quando o equipamento adormece e outra pessoa chega — porque
o aviso existe para informar cada passageiro, não uma vez só.

## Decisões já tomadas no código

Vale a pena discuti-las antes de assumir que estão certas.

**A publicidade nunca ocupa o ecrã inteiro.** A faixa inferior mantém sempre as
duas próximas chegadas visíveis. Distingue um MUPI de transporte de um mero
painel publicitário e não reduz de forma significativa o espaço vendável.

**O timeout tem dois passos.** 60 s sem toque devolve ao ecrã inicial; mais 30 s
devolve à publicidade. Um salto direto para publicidade é agressivo para quem
está apenas a ler devagar.

**O estado degradado muda o formato, não só a cor.** Os tempos passam de "3 min"
para horas absolutas ("14:38") e perdem o verde. A mudança de formato comunica
sozinha que a informação é teórica, mesmo a quem não lê o aviso.

**O QR da app está embebido como SVG, não vem da rede.** O destino nunca muda,
portanto foi gerado uma vez e guardado dentro do ficheiro. Funciona com o
equipamento offline, que é o cenário em que a app é mais útil ao passageiro.
Para o regerar com outro endereço: `segno.make(url, error='h')` em Python, ou
qualquer gerador que exporte SVG. Correção de erros no nível H — sobrevive a
riscos e sujidade no vidro.

**O mapa é esquemático, não geográfico.** As coordenadas são próprias
(1000×880) e o traçado das ruas é inventado — serve para validar desenho e
interação, não para orientar ninguém. Para a versão real, a nota importante:
um mapa de quiosque nunca sai do sítio, portanto podem descarregar-se os
tiles de um raio de 1 km, guardá-los no equipamento e ter um mapa geográfico
verdadeiro que funciona sem rede. As coordenadas passam a latitude/longitude
projetadas em pixels; paragens, veículos e pontos de interesse mantêm a
estrutura que já têm.

**Os pontos de interesse são curados à mão.** Importar tudo do OpenStreetMap
enche o mapa de ruído e ninguém responde pela qualidade dos dados. Uma lista
curta, revista por alguém, vale mais — e permite decidir o que faz sentido
junto de um hospital.

**A escala é metade da real.** O painel é 3840×2160 rodado, ou seja 2160×3840.
Desenhar a 1080×1920 é mais confortável e as proporções são as mesmas. Em
produção remove-se o `scale()` da função `ajustarEscala`.
