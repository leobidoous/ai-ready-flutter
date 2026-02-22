# UseCases (Infraestrutura)

## O que são?

UseCases são as **implementações concretas** das interfaces de casos de uso definidas no Domain. Eles coordenam o fluxo de dados entre a camada de apresentação e os repositórios, podendo adicionar validações de negócio antes de delegar ao repositório.

## Responsabilidades

- Implementar a interface `IUserUsecase` do Domain
- Coordenar chamadas ao Repository
- Aplicar validações de negócio quando necessário
- Manter a lógica de orquestração simples e direta

## Estrutura

```dart
import 'package:base_core/base_core.dart';

import '../../domain/entities/user_entity.dart';
import '../../domain/entities/user_filters_entity.dart';
import '../../domain/entities/user_result_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../../domain/usecases/i_user_usecase.dart';

class UserUsecase extends IUserUsecase {
  UserUsecase({required this.repository});

  final IUserRepository repository;

  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
    return repository.getLoggedUser();
  }

  @override
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount() {
    return repository.deleteUserAccount();
  }

  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) {
    return repository.updateUser(data: data);
  }

  @override
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  }) {
    return repository.changeUserPassword(
      id: id,
      newPassword: newPassword,
      currentPassword: currentPassword,
    );
  }

  @override
  Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
    required UserFiltersEntity filters,
  }) {
    return repository.fetchUsers(filters: filters);
  }

  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id}) {
    return repository.getUserById(id: id);
  }

  @override
  Future<Either<IUserFailure, UserEntity>> updateUserById({
    required String id,
    required UserEntity data,
  }) {
    return repository.updateUserById(data: data, id: id);
  }

  @override
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  }) {
    return repository.createUser(data: data);
  }
}
```

## Características

### 1. Implementação da Interface

- Estende `IUserUsecase` do Domain
- Implementa todos os métodos definidos no contrato

### 2. Injeção de Dependência

- Recebe `IUserRepository` via construtor
- Mantém referência final ao repositório

### 3. Delegação ao Repository

- Repassa chamadas diretamente ao repositório
- Mantém assinatura de retorno `Either<IUserFailure, T>`

### 4. Validações (quando necessário)

- Pode adicionar validações antes de chamar o repositório
- Retorna `Left(failure)` em caso de validação falhar
- Exemplo: validar formato de email, tamanho de senha, etc.

## Quando adicionar validações?

Adicione validações no UseCase quando:

- Validar regras de negócio antes de enviar dados
- Verificar permissões do usuário
- Validar formato de dados (email, CPF, etc.)
- Aplicar regras que não dependem de dados externos

## Benefícios

- **Separação de responsabilidades**: UseCase orquestra, Repository transforma
- **Testabilidade**: Fácil mockar o repository para testar validações
- **Flexibilidade**: Pode adicionar lógica sem modificar o repository
- **Clareza**: Fluxo de dados explícito e direto
