# Wear OS Test App — Design

## Objetivo

Projeto de teste para validar que é possível criar, buildar e rodar um app Android
para Wear OS. Não é um produto — é um protótipo mínimo para confirmar o fluxo de
desenvolvimento (Gradle build → instalação no emulador/relógio → interação com
hardware básico).

## Escopo

- Contador em tela: botão que incrementa um número exibido na tela.
- Ao clicar, o relógio vibra (testa acesso a hardware nativo).
- Sem persistência entre sessões, sem rede, sem segundo módulo de celular.

Fora de escopo: notificações, sensores de saúde, companion app no celular,
distribuição na Play Store.

## Arquitetura

Projeto Gradle Android **single-module** (`app`), standalone — roda sozinho no
relógio, sem exigir um app no celular. Essa é a forma mais simples de testar
localmente (sideload via ADB ou emulador Wear OS); a estrutura phone+wear só é
necessária para distribuição via Play Store, o que está fora de escopo aqui.

- **Linguagem/UI**: Kotlin + Jetpack Compose for Wear OS
  (`androidx.wear.compose:compose-material`, `compose-foundation`).
- **minSdk**: 30 (Wear OS 3+).
- **Entry point**: `MainActivity`, hospedando um único Composable.

## Componentes

- `MainActivity`: Activity única, configura o Compose e o `Vibrator`/`VibratorManager`.
- `CounterScreen` (Composable): exibe o valor do contador centralizado (layout
  pensado para tela redonda) e um botão.
- Estado local via `remember { mutableStateOf(0) }` — sem ViewModel, sem
  persistência, por ser um teste descartável.

## Fluxo de dados

1. Usuário toca no botão.
2. Estado do contador incrementa (`remember` state).
3. Efeito colateral dispara vibração curta (`VibrationEffect.createOneShot`).
4. UI recompõe mostrando o novo valor.

## Tratamento de erros

Nenhum tratamento de erro dedicado — escopo é propositalmente mínimo. Única
preocupação é verificar disponibilidade do `Vibrator` em tempo de execução (API
varia entre versões do Wear OS) para evitar crash ao vibrar.

## Teste

Manual, via Android Studio:

1. Build do projeto (`./gradlew assembleDebug`).
2. Instalar no emulador Wear OS ou relógio físico via ADB.
3. Clicar no botão e confirmar: (a) número incrementa na tela, (b) relógio vibra.

Sem testes automatizados — fora de escopo para um protótipo de validação.
