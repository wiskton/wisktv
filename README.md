# WiskTV — IgorTV CUP

Site estático para acompanhar os campeonatos de futebol amador da IgorTV CUP: classificação, artilharia, jogos, elenco de jogadores e um simulador de partidas com escalação, sorteio de times e placar ao vivo (simulado).

## Stack

Sem build, sem dependências, sem framework — só HTML, CSS e JavaScript puro. O único requisito é servir os arquivos por HTTP (não abrir o `index.html` direto pelo `file://`, porque o carregamento dos dados usa `fetch`).

## Rodando localmente

```bash
python3 -m http.server
```

E acesse `http://localhost:8000`.

## Estrutura do projeto

```
index.html          Página única — toda a UI e a lógica das abas vivem aqui
css/
  default.css        Estilos base (layout, cores, componentes)
  tema-*.css          Temas visuais alternativos (clássico, deluxe, moderno, ultimate)
data.json            Dados principais (fixture no formato do Django): horários, jogadores,
                     campeonatos, partidas e escalações
jsons/legends.json   Jogadores da categoria especial LEGEND, mantidos à parte do data.json
imgs/                Fotos de jogadores, logos e artes dos torneios
```

## Dados

`index.html` busca `data.json` e `jsons/legends.json` em paralelo e concatena os dois antes de processar (se `jsons/legends.json` falhar ao carregar, o site continua funcionando normalmente, só sem os jogadores LEGEND). Ambos seguem o formato de fixture do Django — uma lista de objetos `{ "model": "...", "pk": ..., "fields": {...} }` — com os seguintes `model`s:

- `horarios.horario` — cada grade de horário/torneio (dia, local, admins)
- `jogadores.jogador` — cadastro de jogadores (nome, foto, posição, nível, estatísticas)
- `campeonatos.campeonato` — campeonatos dentro de um horário
- `partidas.partida` — jogos de cada campeonato
- `partidas.escalacao` — escalação de cada partida

Jogadores com `posicao: "GOL"` ou `ativo: false` ficam de fora de rankings, artilharia e do simulador de escalação titular.

## Abas do site

- **Artilheiros** — ranking de gols
- **Classificação** — tabela de pontos corridos do campeonato selecionado
- **Jogos** — lista de partidas (placar, rodada, status)
- **Simulador** — monta um confronto: seleciona jogadores, sorteia dois times equilibrados por posição e nível, e simula uma partida minuto a minuto com placar, artilheiro e review
- **Jogadores** — elenco completo com estatísticas individuais
- Um seletor de tema no topo troca entre os estilos visuais (`css/tema-*.css`)

## Simulador de partida

O simulador decide quem marca em cada evento de gol combinando, como média ponderada (não multiplicada, pra não compor vantagem demais):

1. **Pontos do time** — soma de nível dos jogadores selecionados
2. **Gols reais por jogo** — média histórica de gols de cada jogador nas partidas de verdade (pesa mais que os pontos)
3. **Nota do craque** — maior nível entre os jogadores do time, como diferencial de decidir a partida sozinho
4. **Goleiro/zagueiro adversário** — um goleiro ou zagueiro melhor do outro time dificulta um pouco o gol (peso bem suave)

Dentro do time, quem marca é sorteado com peso pelo histórico real de gols por jogo (o artilheiro de verdade puxa a maior parte das chances), com uma fadiga que reduz a chance do mesmo jogador repetir gol na mesma partida — mais forte contra defesa adversária boa. Jogadores da categoria **LEGEND** sempre têm 5x mais chance de gol que o melhor peso entre os demais do time.

O sorteio de times distribui os goleiros um pra cada time (nunca dois no mesmo) e depois faz um ajuste fino trocando jogadores de mesma posição entre os times até a soma de pontos ficar o mais próxima possível.
