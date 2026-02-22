# Repositories (Infraestrutura)

## O que são?

Repositories são as **implementações concretas** das interfaces de repositório definidas no Domain. Eles fazem a ponte entre o UseCase e o DataSource, transformando dados de entrada/saída e tratando erros.

## Responsabilidades

- Implementar a interface `IUserRepository` do Domain
- Chamar o DataSource para comunicação com APIs
- Transformar `HttpDriverResponse` em Entities usando Models
- Transformar `HttpErrorResponse` em Failures específicas
- Capturar exceções e registrar no CrashLog
- Tratar erros desconhecidos

## Estrutura

```dart
import 'package:base_core/base_core.dart';

import '../../domain/entities/user_entity.dart';
import '../../domain/entities/user_filters_entity.dart';
import '../../domain/entities/user_result_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../datasources/i_user_datasource.dart';
import '../models/user_model.dart';
import '../models/user_result_model.dart';

class UserRepository extends IUserRepository {
  UserRepository({
    required this.crashLog,
    required this.datasource,
    required this.firebaseAuthService,
  });

  final CrashLog crashLog;
  final IUserDatasource datasource;
  final IFirebaseAuthService firebaseAuthService;

  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
    return datasource.getLoggedUser().then((value) {
      try {
        return value.fold(
          (l) => Left(UserServerError(message: l.message)),
          (r) => Right(UserModel.fromMap(r.data)),
        );
      } catch (exception, stackTrace) {
        crashLog.capture(exception: exception, stackTrace: stackTrace);
        return Left(UserUnknownError(message: '$exception'));
      }
    });
  }

  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) {
    return datasource.updateUser(data: data).then((value) {
      try {
        return value.fold(
          (l) => Left(UserServerError(message: l.message)),
          (r) => Right(UserModel.fromMap(r.data)),
        );
      } catch (exception, stackTrace) {
        crashLog.capture(exception: exception, stackTrace: stackTrace);
        return Left(UserUnknownError(message: '$exception'));
      }
    });
  }

  @override
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  }) {
    return datasource
        .changeUserPassword(
          id: id,
          newPassword: newPassword,
          currentPassword: currentPassword,
        )
        .then((value) {
          try {
            return value.fold(
              (l) => Left(UserServerError(message: l.message)),
              (r) => Right(unit),
            );
          } catch (exception, stackTrace) {
            crashLog.capture(exception: exception, stackTrace: stackTrace);
            return Left(UserUnknownError(message: '$exception'));
          }
        });
  }

  @override
  Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
    required UserFiltersEntity filters,
  }) {
    return datasource.fetchUsers(filters: filters).then((value) {
      try {
        return value.fold(
          (l) => Left(UserServerError(message: l.message)),
          (r) => Right(UserResultModel.fromMap(r.data)),
        );
      } catch (exception, stackTrace) {
        crashLog.capture(exception: exception, stackTrace: stackTrace);
        return Left(UserUnknownError(message: '$exception'));
      }
    });
  }

  @override
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  }) {
    return datasource.createUser(data: data).then((value) {
      try {
        return value.fold(
          (l) => Left(UserServerError(message: l.message)),
          (r) => Right(UserModel.fromMap(r.data)),
        );
      } catch (exception, stackTrace) {
        crashLog.capture(exception: exception, stackTrace: stackTrace);
        return Left(UserUnknownError(message: '$exception'));
      }
    });
  }
}
```

## Características

### 1. Injeção de Dependências

```dart
UserRepository({
  required this.crashLog,
  required this.datasource,
  required this.firebaseAuthService,
});
```

- `crashLog`: Para registrar exceções
- `datasource`: Para comunicação com API
- `firebaseAuthService`: Serviços auxiliares quando necessário

### 2. Padrão de Transformação

```dart
return datasource.getLoggedUser().then((value) {
  try {
    return value.fold(
      (l) => Left(UserServerError(message: l.message)),
      (r) => Right(UserModel.fromMap(r.data)),
    );
  } catch (exception, stackTrace) {
    crashLog.capture(exception: exception, stackTrace: stackTrace);
    return Left(UserUnknownError(message: '$exception'));
  }
});
```

### 3. Tratamento de Erros

- **Erro do servidor**: `Left(UserServerError(message: l.message))`
- **Sucesso**: `Right(UserModel.fromMap(r.data))`
- **Exceção**: Captura, registra no CrashLog e retorna `UserUnknownError`

### 4. Transformação de Dados

- **Entrada**: Recebe `HttpDriverResponse` do DataSource
- **Saída**: Retorna Entity usando Model para deserialização
- **Exemplo**: `UserModel.fromMap(r.data)` transforma Map em UserEntity

## Fluxo de Dados

```
DataSource → Repository → UseCase
   ↓             ↓           ↓
Either<       Either<    Either<
  HttpError,    Failure,   Failure,
  HttpResp>     Entity>    Entity>
```

### Transformações:

1. **HttpErrorResponse** → **UserServerError** (erro do servidor)
2. **HttpDriverResponse.data** → **UserEntity** (sucesso via Model)
3. **Exception** → **UserUnknownError** (erro desconhecido)

## Benefícios

- **Isolamento**: Domain não conhece detalhes de HTTP
- **Tratamento centralizado**: Todos os erros passam pelo mesmo fluxo
- **Rastreabilidade**: Exceções registradas no CrashLog
- **Tipagem forte**: Either garante tratamento de erros
- **Testabilidade**: Fácil mockar DataSource e testar transformações
