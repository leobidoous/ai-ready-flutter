# Controllers (Presentation)

## O que são?

Controllers são responsáveis por **gerenciar o estado da UI** e coordenar chamadas aos UseCases. Eles estendem `CustomController` que fornece gerenciamento de estado reativo.

## Responsabilidades

- Gerenciar estado da UI (loading, success, error)
- Chamar UseCases para executar operações
- Notificar a UI sobre mudanças de estado
- Armazenar dados temporários da tela
- Não conter lógica de negócio

## Estrutura

```dart
import 'package:base_core/base_core.dart' show CustomController;

import '../../domain/entities/user_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/usecases/i_user_usecase.dart';
import '../../infra/models/user_model.dart';

class UserController extends CustomController<IUserFailure, UserEntity> {
  UserController({required this.usecase}) : super(UserModel.fromMap({}));

  final IUserUsecase usecase;

  UserEntity data = UserModel.fromMap({});

  Future<void> getLoggedUser() async {
    await execute(() => usecase.getLoggedUser());
  }

  Future<void> getUserById({required String id}) async {
    await execute(() => usecase.getUserById(id: id));
  }

  Future<void> updateUser(UserEntity data) async {
    await execute(() => usecase.updateUser(data: data));
  }

  Future<void> createUser(UserEntity data) async {
    await execute(() => usecase.createUser(data: data));
  }

  Future<void> updateUserById({
    required String id,
    required UserEntity data,
  }) async {
    await execute(() => usecase.updateUserById(data: data, id: id));
  }
}
```

## Características

### 1. Herança de CustomController

```dart
class UserController extends CustomController<IUserFailure, UserEntity>
```

- Primeiro tipo genérico: `IUserFailure` (tipo de erro)
- Segundo tipo genérico: `UserEntity` (tipo de dados)
- Fornece gerenciamento de estado automático

### 2. Estado Inicial

```dart
UserController({required this.usecase}) : super(UserModel.fromMap({}));
```

- Define estado inicial vazio usando Model
- Evita valores null

### 3. Injeção de UseCase

```dart
final IUserUsecase usecase;
```

- Recebe UseCase via construtor
- Delega todas as operações ao UseCase

### 4. Método execute()

```dart
await execute(() => usecase.getLoggedUser());
```

- Gerencia estados automaticamente: loading → success/error
- Atualiza `state` com resultado
- Notifica listeners (UI)

### 5. Dados Temporários

```dart
UserEntity data = UserModel.fromMap({});
```

- Armazena dados temporários da tela
- Útil para formulários e edições

## Estados do Controller

O `CustomController` gerencia automaticamente os seguintes estados:

### Initial

- Estado inicial antes de qualquer operação
- UI pode mostrar placeholder ou tela vazia

### Loading

- Durante execução de operação assíncrona
- UI mostra loading indicator

### Success

- Operação concluída com sucesso
- `state.data` contém o resultado
- UI exibe os dados

### Error

- Operação falhou
- `state.failure` contém o erro
- UI exibe mensagem de erro

## Uso na UI

```dart
class UserPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = context.watch<UserController>();

    return controller.state.when(
      initial: () => Text('Carregando...'),
      loading: () => CircularProgressIndicator(),
      success: (user) => Text('Olá, ${user.name}'),
      error: (failure) => Text('Erro: ${failure.message}'),
    );
  }
}
```

## Benefícios

- **Separação de responsabilidades**: Controller não tem lógica de negócio
- **Reatividade**: UI atualiza automaticamente quando estado muda
- **Testabilidade**: Fácil mockar UseCase e testar estados
- **Simplicidade**: Código limpo e direto
- **Reutilização**: Mesmo controller pode ser usado em múltiplas telas
