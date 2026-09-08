# IcarusBiomes

Datapack com biomas customizados (registro novo, cores próprias) para o
plugin [IcarusRPG](https://github.com/IcaroFranca/IcarusRPG). Existe como
repositório separado porque criar um bioma **de verdade novo** (namespace e
cores próprias, não só pintar um bioma vanilla existente) não é algo que a
API de plugin do Bukkit/Paper consegue fazer em tempo de execução — só dá
pra registrar isso via datapack. A Biome's Wand do IcarusRPG
(`/biomewand`) só pinta biomas vanilla que já existem no registro; este
datapack é o que cria o bioma que ela (ou o comando `/fillbiome` do
próprio Minecraft) pode aplicar depois.

A raiz deste repositório **é** a raiz do datapack (`pack.mcmeta` na raiz),
então instalar é só copiar o repo inteiro pra dentro de
`world/datapacks/` do servidor — ver "Instalação" abaixo.

## Bioma: Cemitério Sombrio / Shadowed Graveyard

- **ID**: `icarusrpg:shadowed_graveyard`
- **Arquivo**: `data/icarusrpg/worldgen/biome/shadowed_graveyard.json`

Namespace `icarusrpg` (não `icarusbiomes`) de propósito: o plano é que a
zona de spawn dos mobs da ilha (`IslandMobZone`, no IcarusRPG) passe a
usar esse bioma como critério mais pra frente, e o código do plugin já cria
todas as suas próprias `NamespacedKey` a partir do namespace do plugin
(`icarusrpg`, derivado do `name: IcarusRPG` do `plugin.yml`) — manter o
bioma no mesmo namespace evita ter dois nomes diferentes pra a mesma coisa
mais tarde. O fato do JSON morar num repositório *separado* é só uma
questão de onde o arquivo fisicamente vive/é versionado, não muda o
namespace do registro.

### Cores

| Efeito | Hex | Decimal (usado no JSON) |
|---|---|---|
| Grama (`grass_color`) | `#8E6FCE` | `9334734` |
| Folhagem (`foliage_color`) | `#5B3E96` | `5979798` |
| Água (`water_color`) | `#4A3B6B` | `4864875` |
| Neblina da água (`water_fog_color`) | `#2E2447` | `3023943` |
| Neblina do ar (`fog_color`) | `#7B6C9E` | `8088734` |
| Céu (`sky_color`) | `#4A3B94` | `4864916` |

Um roxo ametista saturado na grama, mais escuro/azulado na folhagem pra dar
profundidade, água indigo/parada quase preta no fundo, e um céu roxo
escuro (mais escuro que a água, clima de tempestade sobrenatural) —
paleta fechada de ponta a ponta, nada fica no azul vanilla padrão.

Pra trocar qualquer cor depois: edite o valor decimal correspondente em
`shadowed_graveyard.json` (Minecraft só aceita inteiro decimal, não hex,
nos campos de `effects`) — a tabela acima já traz os dois lado a lado pra
não precisar reconverter.

### Clima

`temperature: 0.7`, `downfall: 0.5`, `has_precipitation: true` — chuva
normal (não neve; neve vanilla só entra com temperatura bem mais baixa),
nem seco nem encharcado. Combina com o efeito de "chuva ocasional caindo
num cemitério roxo" sem forçar neve numa ilha flutuante.

### Por que `carvers`/`features`/`spawners` estão vazios

Esse bioma não é pensado pra gerar terreno do zero — a ilha de combate já
existe pronta (construída à mão/schematic), e o bioma só vai ser *pintado*
na área dela (via `/fillbiome` ou a Biome's Wand, se ela ganhar suporte a
biomas custom depois). Deixar geração/estruturas/spawners vazios evita
qualquer efeito colateral esquisito (cavernas, minérios, mobs vanilla
"de bioma cemitério" que não existem) numa área que na prática nunca passa
pelo pipeline de geração de chunk normal. Mob spawn na zona da ilha já é
tratado à parte pelo `island-mobs` do IcarusRPG (que também suprime spawn
natural ali).

## Instalação

1. Copie (ou dê `git clone`) este repositório inteiro pra dentro de
   `world/datapacks/` do seu servidor — o resultado deve ser
   `world/datapacks/IcarusBiomes/pack.mcmeta` (não um subdiretório a mais
   dentro dele).
2. Se o servidor já estava rodando, `/reload` (ou reinicie) pra carregar o
   datapack novo. O chat/log deve confirmar o datapack ativo; se aparecer
   um aviso de "Pack ... was designed for a different version", é só o
   `pack_format` deste `pack.mcmeta` não bater exatamente com o do seu
   servidor — o datapack carrega mesmo assim (é aviso, não erro bloqueante),
   mas ajuste `pack_format`/`supported_formats` em `pack.mcmeta` pro valor
   correto da sua versão se quiser silenciar o aviso.
3. Confirme que o bioma existe no registro: `/execute in minecraft:overworld
   run locate biome icarusrpg:shadowed_graveyard` (só acha depois de pintado
   em algum chunk) ou simplesmente tente pintá-lo (próximo passo) — se o ID
   não existisse, o comando teria dado erro na hora.

## Testando na ilha

A ilha de combate já é terreno existente, então o bioma se aplica pintando,
não gerando:

```
/fillbiome ~-10 ~-5 ~-10 ~10 ~20 ~10 icarusrpg:shadowed_graveyard
```

(ajuste as coordenadas pra área real da ilha). Depois, F3 mostra o bioma
sob a mira/pés do jogador — confirme que aparece `icarusrpg:shadowed_graveyard`
e que a cor da grama/folhagem realmente mudou pro roxo. `/fillbiome` muda
biomas em chunks já gerados, então dá pra testar em qualquer lugar já
carregado sem precisar gerar terreno novo ou reiniciar o mundo.

A Biome's Wand do IcarusRPG (`/biomewand`) **não** pinta este bioma ainda —
ela só lista os 15 biomas vanilla curados em `BiomeOption`. Dar suporte a
biomas customizados (deste datapack) na wand é trabalho futuro no
repositório do IcarusRPG, não algo que este datapack por si resolve.

## Função utilitária: pintar uma área grande de uma vez (`paint_test_area`)

`/fillbiome` sozinho só cobre até 32768 blocos de volume por chamada (mesmo
teto do `/fill` vanilla) — pintar uma área de segurança grande (ex.: pra
testar se o `sky_color` só aparece quando o bioma domina uma região ampla,
não só a área pisável) exigiria dezenas de comandos manuais. A função
`data/icarusrpg/function/paint_test_area.mcfunction` faz isso num
comando só: pinta um bloco de 320x320 (16 de altura, `~-8` a `~8`)
centrado em quem executa, em 64 chamadas de `/fillbiome` de 40x40x16 cada
(dentro do limite), ladrilhando a área toda.

Uso: fique parado no centro da área que quer pintar e rode
```
/function icarusrpg:paint_test_area
```
Se o servidor reclamar de posição não carregada, primeiro force a geração
dos chunks (`/forceload add <x1> <z1> <x2> <z2>`, também limitado a 256
chunks por chamada) na mesma área antes de rodar a função.

É uma ferramenta de diagnóstico/teste (nasceu de investigar por que o céu
não mudava de cor numa área pequena pintada) — não é usada pela ilha de
combate em si, que é bem menor que 320x320.

## Próximo passo (não implementado aqui)

Uma vez que o datapack esteja instalado e o bioma pintado na ilha, a ideia
é migrar `IslandMobZone` (hoje um retângulo fixo de coordenadas X/Z) pra
detectar a zona de spawn pelo bioma do bloco (`icarusrpg:shadowed_graveyard`)
em vez de coordenadas cravadas no `config.yml` — isso é trabalho no
repositório do **IcarusRPG**, fora do escopo deste datapack.
