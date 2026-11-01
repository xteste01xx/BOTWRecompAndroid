# Alterações para o POCO F6 Pro (Adreno 740)

Registro de cada modificação feita sobre o BOTWRecompAndroid (commit base `2a1bf32`, 0.2.6-beta). Branch: `adreno740`.
Regra deste registro: uma entrada só diz "corrigido" quando houver teste no aparelho que comprove.

## Estado

| # | Alteração | Arquivos | Status |
| --- | --- | --- | --- |
| 1 | Workflow do GitHub Actions que compila o APK arm64 (debug) | `.github/workflows/build-apk.yml` | 1ª execução (31/10/2026) **falhou** no passo do checksum: o APK não foi gerado |
| 2 | Workflow: falhar de verdade nos passos com `\| tee` (`shell: bash`, pipefail) e publicar o fim dos logs na branch `ci-logs` | `.github/workflows/build-apk.yml` | Substituído pelo #3 (a branch `ci-logs` falhou) |

| 3 | Workflow: logs como anotações da execução; `chmod +x gradlew` (o arquivo está sem permissão de execução no repositório: erro 126); 3 tentativas nos passos de dependências (queda de conexão ao baixar o FFmpeg) | `.github/workflows/build-apk.yml` | **Funcionou**: execução de 01/11/2026 terminou com `BUILD SUCCESSFUL` e gerou o APK debug arm64 (cerca de 97 MB zipado) |

| 4 | Junção (`merge`) do projeto original 0.2.10-beta na branch: traz as correções feitas desde a 0.2.6 (passes de renderização, arena de memória, ritmo de quadros, texturas do terreno) | todo o `runtime/` e `android/` | Compila (execução de 01/11/2026, `BUILD SUCCESSFUL`). **Não testado em aparelho** |

**APK gerado, mas ainda NÃO testado em aparelho.** É o projeto original (agora na 0.2.10-beta) compilado, sem nenhuma correção nossa do
renderizador. A causa da tela preta no Adreno 740 continua desconhecida: falta o log de uma execução com o defeito.

Na 1ª execução, os passos de dependências e do Gradle apareceram como "sucesso" porque o `tee` escondia o código de saída
do comando; por isso o erro real ainda não é conhecido.

Nenhuma alteração no renderizador, nos shaders ou no runtime até agora. A causa dos gráficos pretos no Adreno 740 **não foi
identificada**: falta o log de uma execução com o defeito (`botwrecomp-log-previous.txt`) e a versão exata do app.

## 1. Workflow de compilação

- Segue os passos documentados no README e no CONTRIBUTING: `build_deps.sh`, `build_ffmpeg.sh`, `build_gen_deps.sh` e
  `./gradlew assembleDebug`.
- Instala NDK `30.0.16248370`, CMake `3.30.4` e a plataforma Android 36 (as versões que o projeto fixa).
- Não usa nem precisa de arquivos do jogo: o código do jogo é gerado no próprio celular a partir do dump do usuário.
- O APK é de debug, assinado com a chave de debug. Para atualizar um app já instalado, a assinatura precisa ser a mesma
  do app instalado; se for outra, o Android recusa a atualização.
- Limites conhecidos: nunca rodou; as dependências levam bastante tempo na primeira execução (há cache nas seguintes);
  o espaço em disco do runner pode ficar apertado.

## Plano em etapas (um commit por etapa, branch `adreno740`)

| Etapa | O que | Depende de | Estado |
| --- | --- | --- | --- |
| 1 | Workflow de compilação automática | nada | feito, funcionando |
| 2 | Sincronizar com o projeto original (0.2.10) | nada | feito, compila |
| 3 | Instalar o APK da etapa 2 e ver se a tela preta persiste | teste no POCO F6 Pro | **aguardando o usuário** |
| 4 | Se persistir: ler o log (`botwrecomp-log-previous.txt`) e achar a causa: driver realmente em uso, pipelines de compute rejeitados, passes do G-buffer, caminhos que só valem no Adreno 8xx | log do aparelho | bloqueada |
| 5 | Correção mínima da causa achada, atrás de uma condição só para Adreno 7xx (`vk_adreno_8xx` em `device.c` é o ponto de partida) | etapa 4 | bloqueada |
| 6 | Perfil de desempenho para Adreno 740 (resolução, FPS, memória), com números medidos no aparelho | log com tempos de GPU | bloqueada |

Motivo do bloqueio: o projeto original já foi testado com Adreno 740 + Turnip V38 e o mundo renderiza (issue #43), então o
defeito pode ser da versão instalada, do driver escolhido ou de algo específico do aparelho. Sem o log, qualquer alteração no
renderizador seria um palpite, e `Adreno 7xx` e `8xx` hoje seguem caminhos de código diferentes.
