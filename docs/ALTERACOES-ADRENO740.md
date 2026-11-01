# Alterações para o POCO F6 Pro (Adreno 740)

Registro de cada modificação feita sobre o BOTWRecompAndroid (commit base `2a1bf32`, 0.2.6-beta). Branch: `adreno740`.
Regra deste registro: uma entrada só diz "corrigido" quando houver teste no aparelho que comprove.

## Estado

| # | Alteração | Arquivos | Status |
| --- | --- | --- | --- |
| 1 | Workflow do GitHub Actions que compila o APK arm64 (debug) | `.github/workflows/build-apk.yml` | 1ª execução (31/10/2026) **falhou** no passo do checksum: o APK não foi gerado |
| 2 | Workflow: falhar de verdade nos passos com `\| tee` (`shell: bash`, pipefail) e publicar o fim dos logs na branch `ci-logs` | `.github/workflows/build-apk.yml` | Substituído pelo #3 (a branch `ci-logs` falhou) |

| 3 | Workflow: logs como anotações da execução; `chmod +x gradlew` (o arquivo está sem permissão de execução no repositório: erro 126); 3 tentativas nos passos de dependências (queda de conexão ao baixar o FFmpeg) | `.github/workflows/build-apk.yml` | **Funcionou**: execução de 01/11/2026 terminou com `BUILD SUCCESSFUL` e gerou o APK debug arm64 (cerca de 97 MB zipado) |

**APK gerado, mas ainda NÃO testado em aparelho.** É o projeto original (0.2.6-beta) compilado, sem nenhuma correção do
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
