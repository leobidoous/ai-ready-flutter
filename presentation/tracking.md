# Tracking & Analytics (Mixpanel Funnel)

Guia para criação de tracking mixins focados em funis de conversão no Mixpanel.

## Visão Geral

O tracking é implementado via **mixins** na camada de Presentation, acoplados às pages que possuem fluxos multi-step (PageView). Cada mixin é responsável por:

1. Definir os nomes dos steps do fluxo (`abstract final class`)
2. Mapear widgets → step names (`stepNameForIndex`)
3. Expor métodos tipados para cada evento do funil

## Serviço de Tracking

```dart
// Acesso via DI
NewAppTracking get appTracking => DM.i.get<NewAppTracking>();

// Uso básico
appTracking.log(
  event: LogEventEntity(
    name: 'event_name',
    parameters: {'key': 'value'},
  ),
);
```

`NewAppTracking` é a classe centralizada em `paypay_core` que despacha eventos para Mixpanel, AppsFlyer e Firebase Analytics.

## Estrutura do Mixin

### 1. Arquivo e `part of`

O mixin deve ser um arquivo `part of` da page principal:

```
{feature}/
├── {feature}_page.dart                    # Page principal
├── {feature}_tracking_mixin.dart          # Mixin de tracking (part of page)
└── widgets/
    └── views/
        ├── step_one_view.dart             # part of page
        └── step_two_view.dart             # part of page
```

Na page:

```dart
part '{feature}_tracking_mixin.dart';
```

No mixin:

```dart
part of '{feature}_page.dart';
```

### 2. Constantes de Steps

Defina uma `abstract final class` com os nomes dos steps. Use o prefixo do fluxo para evitar colisões no Mixpanel:

```dart
abstract final class {Feature}Steps {
  static const stepOne = '{prefix}_step_one';
  static const stepTwo = '{prefix}_step_two';
  static const status = '{prefix}_status';
}
```

**Convenção de prefixo**:

- `register_` → Cadastro novo
- `complete_register_` → Completar cadastro
- `migrate_` → Migração de conta
- `login_` → Login

### 3. Mixin com `stepNameForIndex`

```dart
mixin {Feature}TrackingMixin<T extends StatefulWidget> on State<T> {
  NewAppTracking get appTracking => DM.i.get<NewAppTracking>();

  String stepNameForIndex({
    required List<Widget> children,
    required int index,
  }) {
    return switch (children[index].runtimeType) {
      const (_StepOneView) => {Feature}Steps.stepOne,
      const (_StepTwoView) => {Feature}Steps.stepTwo,
      _ => '',
    };
  }
}
```

## Parâmetros dos Eventos

Nenhum parâmetro é obrigatório por padrão. O importante é que cada evento carregue os **atributos mais relevantes do contexto** para que o funil seja analisável no Mixpanel.

Pergunte-se: "Se eu estiver olhando esse evento no Mixpanel, quais informações preciso para entender o que aconteceu?"

**Exemplos de atributos relevantes por contexto:**

- Fluxos com sessão: `session_id`
- Fluxos com empresa: `company_identity`, `company_type`
- Fluxos com usuário: `user_id`, `phone_number`, `email`
- Steps de formulário: o valor preenchido ou selecionado
- Erros: `error_type`, `error_message`

**Regra geral**: passe o que ajuda a segmentar e debugar no Mixpanel. Evite parâmetros genéricos que não agregam valor à análise.

## Eventos Obrigatórios para Funil

Todo mixin de tracking DEVE implementar estes eventos para que o funil funcione no Mixpanel:

### Eventos de Ciclo de Vida do Fluxo

| Evento               | Quando disparar             | Parâmetros sugeridos             |
| -------------------- | --------------------------- | -------------------------------- |
| `{prefix}_started`   | Usuário entra no fluxo      | atributos relevantes do contexto |
| `{prefix}_completed` | Fluxo concluído com sucesso | atributos relevantes do contexto |
| `{prefix}_abandoned` | Usuário desiste do fluxo    | `step_name` + contexto           |

### Eventos de Navegação entre Steps

| Evento                   | Quando disparar            | Parâmetros sugeridos                                  |
| ------------------------ | -------------------------- | ----------------------------------------------------- |
| `screen_view`            | Usuário visualiza um step  | `step_name`, `screen_name`, `screen_class` + contexto |
| `{prefix}_step_complete` | Step concluído com sucesso | `step_name` + contexto + `...data?`                   |
| `{prefix}_step_back`     | Usuário volta um step      | `step_name` + contexto                                |

### Eventos de Erro

| Evento                  | Quando disparar            | Parâmetros sugeridos                     |
| ----------------------- | -------------------------- | ---------------------------------------- |
| `{prefix}_{step}_error` | Erro em um step específico | `error_type`, `error_message` + contexto |

## Implementação dos Métodos

### Métodos Base (copiar em todo mixin)

Os parâmetros abaixo são ilustrativos. Adapte conforme os atributos relevantes do seu fluxo.

```dart
// ── Screen Views ──

void trackStepView({
  required Type screenClass,
  required String stepName,
  Map<String, Object>? context,
}) {
  appTracking.log(
    event: LogEventEntity(
      name: 'screen_view',
      parameters: {
        'step_name': stepName,
        'screen_name': '{Feature}Page',
        'screen_class': screenClass.toString(),
        ...?context,
      },
    ),
  );
}

// ── Funnel ──

void track{Feature}Started({Map<String, Object>? context}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_started',
      parameters: {...?context},
    ),
  );
}

void trackStepComplete({
  required String stepName,
  Map<String, Object>? data,
}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_step_complete',
      parameters: {
        'step_name': stepName,
        ...?data,
      },
    ),
  );
}

void trackStepBack({required String stepName, Map<String, Object>? context}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_step_back',
      parameters: {'step_name': stepName, ...?context},
    ),
  );
}

// ── Conversion ──

void track{Feature}Completed({Map<String, Object>? context}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_completed',
      parameters: {...?context},
    ),
  );
}

void track{Feature}Abandoned({
  required String stepName,
  Map<String, Object>? context,
}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_abandoned',
      parameters: {'step_name': stepName, ...?context},
    ),
  );
}
```

### Métodos Step-Specific

Para cada step, crie um método tipado que chama `trackStepComplete`. Passe os atributos que fazem sentido para aquele step:

```dart
void trackPersonAddressCompleted({String? sessionId}) {
  trackStepComplete(
    stepName: {Feature}Steps.personAddress,
    data: {'session_id': ?sessionId},
  );
}

void trackCompanyTypeCompleted({
  required String companyType,
  required double monthlyRevenue,
}) {
  trackStepComplete(
    stepName: {Feature}Steps.companyType,
    data: {
      'company_type': companyType,
      'monthly_revenue': monthlyRevenue,
    },
  );
}
```

### Métodos de Erro

Para steps que podem falhar (validações, chamadas de API):

```dart
void trackPhoneCodeError({
  String? message,
  String? errorType,
  String? sessionId,
}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_phone_code_error',
      parameters: {
        'error_type': errorType ?? '',
        'error_message': message ?? '',
        'session_id': ?sessionId,
      },
    ),
  );
}
```

## Uso na Page

### Aplicar o mixin no State

```dart
class _MyPageState extends State<MyPage> with MyTrackingMixin {
  // ...
}
```

### Disparar eventos no PageView

```dart
PageView(
  onPageChanged: (index) {
    setState(() => _pageIndex = index);
    trackStepView(
      stepName: stepNameForIndex(children: _pageViewChildren, index: index),
      screenClass: _pageViewChildren[index].runtimeType,
      context: {'session_id': _sessionId},
    );
  },
  // ...
)
```

### Disparar step_back no botão voltar

```dart
void _onBack() {
  if (_pageIndex > 0 && !_isLastPage) {
    trackStepBack(stepName: _currentStepName);
    _onPreviousPage();
  }
}
```

### Disparar step_complete nos callbacks de onNext

```dart
_PersonAddressView(
  onNext: (address) {
    trackPersonAddressCompleted(sessionId: _sessionId);
    _controller.data = _controller.data.copyWith(homeAddress: address);
    _onNextPage();
  },
),
```

## Eventos Opcionais (UI Interactions)

Adicione conforme necessidade do produto:

```dart
// Help button
void trackHelpClicked({required String stepName, Map<String, Object>? context}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_help_clicked',
      parameters: {'step_name': stepName, ...?context},
    ),
  );
}

// Exit confirmation bottom sheet
void trackConfirmExitViewed({required String calledFrom, Map<String, Object>? context}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_confirm_exit_viewed',
      parameters: {'called_from': calledFrom, ...?context},
    ),
  );
}

// KYC URL received
void trackKycUrlReceived({required String openMode, String? sessionId}) {
  appTracking.log(
    event: LogEventEntity(
      name: '{prefix}_kyc_url_received',
      parameters: {'open_mode': openMode, 'session_id': ?sessionId},
    ),
  );
}
```

## Referências de Implementação

Mixins existentes no projeto:

- `paypay-auth/.../register/register_tracking_mixin.dart` — Fluxo de cadastro completo (referência principal)
- `paypay-auth/.../login/login_tracking_mixin.dart` — Login (mixin simples, sem PageView)
- `paypay/.../complete_register/complete_register_tracking_mixin.dart` — Completar cadastro
- `paypay/.../migrate_account/migrate_account_tracking_mixin.dart` — Migração de conta

## Checklist para Novo Tracking Mixin

- [ ] Criar arquivo `{feature}_tracking_mixin.dart` como `part of` da page
- [ ] Adicionar `part '{feature}_tracking_mixin.dart';` na page
- [ ] Definir `abstract final class {Feature}Steps` com constantes
- [ ] Implementar `stepNameForIndex` mapeando todos os widgets
- [ ] Implementar eventos obrigatórios: `started`, `step_complete`, `step_back`, `completed`, `abandoned`
- [ ] Implementar `screen_view` com `trackStepView`
- [ ] Criar métodos tipados para cada step (`track{Step}Completed`)
- [ ] Criar métodos de erro para steps com validação/API
- [ ] Aplicar `with {Feature}TrackingMixin` no State da page
- [ ] Disparar `trackStepView` no `onPageChanged` do PageView
- [ ] Disparar `trackStepBack` no botão voltar
- [ ] Disparar `track{Step}Completed` nos callbacks `onNext`
- [ ] Disparar `track{Feature}Started` no `initState`
- [ ] Disparar `track{Feature}Abandoned` no exit confirmation
