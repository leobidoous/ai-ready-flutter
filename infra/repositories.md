# Infrastructure Repositories (Implementações) - Clean Architecture

## 📚 Visão Geral

As **implementações de Repositories** na camada de **Infrastructure** definem **COMO** coordenar o acesso aos dados. Elas implementam os contratos definidos no Domain, orquestrando datasources, cache, tratamento de erros e transformações de dados.

### 🎯 Princípios Fundamentais das Implementações Repository

**O QUE as implementações FAZEM:**
- ✅ **Implementam Contratos**: Executam o que foi definido nas interfaces do Domain
- ✅ **Coordenam DataSources**: Orquestram múltiplas fontes de dados (API, cache, DB)
- ✅ **Transformam Dados**: Convertem entre Models (infraestrutura) e Entities (domain)
- ✅ **Tratam Erros**: Convertem erros técnicos em failures do Domain
- ✅ **Aplicam Estratégias**: Cache, fallback, retry, circuit breaker

**O QUE as implementações NÃO FAZEM:**
- ❌ **Não acessam dados diretamente**: Usam datasources para comunicação externa
- ❌ **Não contêm regras de negócio**: Apenas coordenação e transformação
- ❌ **Não dependem de tecnologias específicas**: Usam abstrações de datasources
- ❌ **Não quebram SOLID**: Dependem de abstrações, não implementações
- ❌ **Não misturam responsabilidades**: Foco apenas em acesso aos dados

### 🏗️ Localização e Estrutura

```
lib/src/infra/repositories/
├── user_repository.dart
├── product_repository.dart
├── order_repository.dart
└── notification_repository.dart
```

---

## 🔍 Anatomia de um Repository

### Estrutura Base

```dart
import 'package:base_core/base_core.dart' show Either, CrashLog;
import '../../domain/entities/[entity]_entity.dart';
import '../../domain/failures/i_[entity]_failures.dart';
import '../../domain/repositories/i_[entity]_repository.dart';
import '../datasources/i_[entity]_datasource.dart';
import '../models/[entity]_model.dart';

/// Implementação do repositório para acesso aos dados de [Entity]
/// 
/// Esta classe coordena datasources, aplica transformações de dados
/// e trata erros técnicos convertendo-os para failures do Domain.
class [Entity]Repository extends I[Entity]Repository {
  [Entity]Repository({
    required this.datasource,
    required this.crashLog,
  });

  final I[Entity]Datasource datasource;
  final CrashLog crashLog;

  @override
  Future<Either<I[Entity]Failure, [Entity]Entity>> get[Entity]() async {
    try {
      final result = await datasource.get[Entity]();
      
      return result.fold(
        (error) => Left([Entity]ServerError(message: error.message)),
        (response) => Right([Entity]Model.fromMap(response.data)),
      );
    } catch (exception, stackTrace) {
      crashLog.capture(exception: exception, stackTrace: stackTrace);
      return Left([Entity]UnknownError(message: '$exception'));
    }
  }
}
```

### Elementos Essenciais

1. **Herança da Interface**: Implementa contrato do Domain
2. **Coordenação de DataSources**: Orquestra fontes de dados
3. **Transformação de Dados**: Model ↔ Entity
4. **Tratamento de Erros**: Exception → Failure
5. **Logging e Monitoramento**: CrashLog para debugging

---

## 📚 Exemplo Prático: UserRepository

### Implementação Real

```dart
import 'package:base_core/base_core.dart';

import '../../domain/entities/user_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../datasources/i_user_datasource.dart';
import '../models/user_model.dart';

/// Implementação do repositório para acesso aos dados de usuário
/// 
/// Esta classe coordena o datasource de usuário, aplica transformações
/// de dados e trata erros técnicos convertendo-os para failures do Domain.
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
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount() async {
    return datasource.deleteUserAccount().then((value) {
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
          (r) => Right(UserModel.fromMap(r.data['data'])),
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
}
```

### Características da Implementação Real

✅ **Segue princípios SOLID:**
- Depende apenas de abstrações (`IUserDatasource`, `CrashLog`)
- Implementa interface específica (`IUserRepository`)
- Responsabilidade única (coordenação de acesso aos dados)

✅ **Padrões de implementação:**
- Transformação consistente: DataSource → Model → Entity
- Tratamento de erros padronizado: Exception → Failure
- Logging para debugging e monitoramento

🔄 **Oportunidades de melhoria:**
- Cache pode ser adicionado
- Fallback strategies podem ser implementadas
- Retry logic pode ser incluído

---

## 🎨 Padrões de Implementação

### 1. Implementação Simples (Delegação + Transformação)
```dart
@override
Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
  return datasource.getLoggedUser().then((value) {
    try {
      return value.fold(
        (error) => Left(UserServerError(message: error.message)),
        (response) => Right(UserModel.fromMap(response.data)),
      );
    } catch (exception, stackTrace) {
      crashLog.capture(exception: exception, stackTrace: stackTrace);
      return Left(UserUnknownError(message: '$exception'));
    }
  });
}
```

### 2. Implementação com Cache
```dart
@override
Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
  try {
    // 1. Tentar cache primeiro
    final cacheResult = await cacheDataSource.getLoggedUser();
    if (cacheResult.isRight()) {
      return cacheResult.fold(
        (error) => Left(UserServerError(message: error.message)),
        (response) => Right(UserModel.fromMap(response.data)),
      );
    }

    // 2. Se cache falhou, buscar na API
    final apiResult = await apiDataSource.getLoggedUser();
    
    return apiResult.fold(
      (error) => Left(UserServerError(message: error.message)),
      (response) async {
        final user = UserModel.fromMap(response.data);
        
        // 3. Salvar no cache para próxima vez
        await cacheDataSource.saveLoggedUser(user: user);
        
        return Right(user);
      },
    );
  } catch (exception, stackTrace) {
    crashLog.capture(exception: exception, stackTrace: stackTrace);
    return Left(UserUnknownError(message: '$exception'));
  }
}
```

### 3. Implementação com Múltiplos DataSources
```dart
class UserRepository extends IUserRepository {
  UserRepository({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.cacheDataSource,
    required this.crashLog,
  });

  final IUserRemoteDataSource remoteDataSource;
  final IUserLocalDataSource localDataSource;
  final IUserCacheDataSource cacheDataSource;
  final CrashLog crashLog;

  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
    try {
      // Estratégia: Cache → Local → Remote
      
      // 1. Verificar cache (mais rápido)
      final cacheResult = await cacheDataSource.getLoggedUser();
      if (cacheResult.isRight()) {
        return _transformToEntity(cacheResult);
      }

      // 2. Verificar storage local (offline)
      final localResult = await localDataSource.getLoggedUser();
      if (localResult.isRight()) {
        final entityResult = _transformToEntity(localResult);
        // Atualizar cache
        if (entityResult.isRight()) {
          await _updateCache(entityResult.getOrElse(() => throw Exception()));
        }
        return entityResult;
      }

      // 3. Buscar remotamente (online)
      final remoteResult = await remoteDataSource.getLoggedUser();
      
      return remoteResult.fold(
        (error) => Left(UserServerError(message: error.message)),
        (response) async {
          final user = UserModel.fromMap(response.data);
          
          // Salvar em todas as camadas para próximas consultas
          await _saveToAllLayers(user);
          
          return Right(user);
        },
      );
    } catch (exception, stackTrace) {
      crashLog.capture(exception: exception, stackTrace: stackTrace);
      return Left(UserUnknownError(message: '$exception'));
    }
  }
}
```

### 4. Implementação com Transformação Complexa
```dart
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  try {
    // 1. Transformar Entity para Model (dados de saída)
    final userModel = UserModel.fromEntity(data);
    
    // 2. Enviar para datasource
    final result = await datasource.updateUser(data: data);
    
    return result.fold(
      (error) => _mapHttpErrorToFailure(error),
      (response) {
        // 3. Transformar resposta para Entity (dados de entrada)
        // Nota: API retorna dados em estrutura aninhada
        final responseData = response.data['data'] as Map<String, dynamic>;
        final updatedUser = UserModel.fromMap(responseData);
        
        return Right(updatedUser);
      },
    );
  } catch (exception, stackTrace) {
    crashLog.capture(exception: exception, stackTrace: stackTrace);
    return Left(UserUnknownError(message: '$exception'));
  }
}

/// Mapeia erros HTTP específicos para failures do domain
Either<IUserFailure, T> _mapHttpErrorToFailure<T>(HttpErrorResponse error) {
  switch (error.statusCode) {
    case 400:
      return Left(UpdateUserDataError(message: error.message));
    case 401:
      return Left(UserUnauthenticatedError(message: error.message));
    case 404:
      return Left(UserNotFoundError(message: error.message));
    case 409:
      return Left(UserConflictError(message: error.message));
    default:
      return Left(UserServerError(message: error.message));
  }
}
```

### 5. Implementação com Retry e Circuit Breaker
```dart
@override
Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
  try {
    return await _retryWithExponentialBackoff(
      operation: () => datasource.getLoggedUser(),
      maxRetries: 3,
      baseDelay: Duration(milliseconds: 500),
    ).then((result) {
      return result.fold(
        (error) => Left(UserServerError(message: error.message)),
        (response) => Right(UserModel.fromMap(response.data)),
      );
    });
  } catch (exception, stackTrace) {
    crashLog.capture(exception: exception, stackTrace: stackTrace);
    return Left(UserUnknownError(message: '$exception'));
  }
}

Future<Either<HttpErrorResponse, HttpDriverResponse>> _retryWithExponentialBackoff({
  required Future<Either<HttpErrorResponse, HttpDriverResponse>> Function() operation,
  required int maxRetries,
  required Duration baseDelay,
}) async {
  for (int attempt = 0; attempt <= maxRetries; attempt++) {
    final result = await operation();
    
    if (result.isRight() || attempt == maxRetries) {
      return result;
    }
    
    // Exponential backoff
    await Future.delayed(baseDelay * (2 * attempt));
  }
  
  return operation(); // Fallback final
}
```

---

## 📋 Template para Implementações Repository

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart' show Either, CrashLog;
import '../../domain/entities/[entity]_entity.dart';
import '../../domain/failures/i_[entity]_failures.dart';
import '../../domain/repositories/i_[entity]_repository.dart';
import '../datasources/i_[entity]_datasource.dart';
import '../models/[entity]_model.dart';

/// Implementação do repositório para acesso aos dados de [Entity]
/// 
/// Esta classe coordena datasources, aplica transformações de dados
/// e trata erros técnicos convertendo-os para failures do Domain.
class [Entity]Repository extends I[Entity]Repository {
  [Entity]Repository({
    required this.datasource,
    required this.crashLog,
    // Outros datasources podem ser injetados quando necessário
  });

  final I[Entity]Datasource datasource;
  final CrashLog crashLog;

  @override
  Future<Either<I[Entity]Failure, [Entity]Entity>> get[Entity]() async {
    try {
      // 1. Chamar datasource
      final result = await datasource.get[Entity]();
      
      // 2. Transformar resultado
      return result.fold(
        (error) => Left([Entity]ServerError(message: error.message)),
        (response) => Right([Entity]Model.fromMap(response.data)),
      );
    } catch (exception, stackTrace) {
      // 3. Capturar exceptions não mapeadas
      crashLog.capture(exception: exception, stackTrace: stackTrace);
      return Left([Entity]UnknownError(message: '$exception'));
    }
  }
}
```

### Convenções de Implementação

**Nomenclatura:**
- Classe: `[Entity]Repository extends I[Entity]Repository`
- Arquivo: `[entity]_repository.dart`

**Estrutura:**
- Construtor com injeção de dependências
- Apenas dependencies de abstrações (interfaces)
- Override de todos os métodos da interface
- Padrão de transformação consistente

**Responsabilidades:**
- Implementar contratos do Domain
- Coordenar datasources
- Transformar dados (Model ↔ Entity)
- Tratar erros técnicos

---

## 📋 Checklist para Implementações Repository

### Checklist de Criação ✅

**Estrutura da Classe:**
- [ ] Localizada em `lib/src/infra/repositories/`
- [ ] Nome seguindo padrão `[Entity]Repository`
- [ ] Herda da interface `I[Entity]Repository`
- [ ] Construtor com injeção de dependências
- [ ] Apenas dependencies de abstrações

**Coordenação de DataSources:**
- [ ] Injeção de datasources via construtor
- [ ] Chamadas assíncronas para datasources
- [ ] Tratamento de Either dos datasources
- [ ] Estratégias de cache quando apropriado

**Transformação de Dados:**
- [ ] Model.fromMap() para converter responses
- [ ] Model.fromEntity() para dados de saída
- [ ] Conversão automática para Entity via extends
- [ ] Tratamento de estruturas complexas de response

**Tratamento de Erros:**
- [ ] try/catch para exceptions não mapeadas
- [ ] Conversão de erros HTTP para failures específicos
- [ ] Logging com crashLog.capture()
- [ ] Mapeamento consistente de status codes

**Padrões de Qualidade:**
- [ ] Documentação clara da classe e responsabilidades
- [ ] Métodos bem documentados com transformações
- [ ] Tratamento consistente de erros
- [ ] Testes unitários correspondentes

---

## 🎯 Diretrizes para Implementações

### ✅ Boas Práticas

```dart
// ✅ Injeção de dependências clara
class UserRepository extends IUserRepository {
  UserRepository({
    required this.datasource,
    required this.crashLog,
    required this.cacheService,
  });

  final IUserDatasource datasource;
  final CrashLog crashLog;
  final ICacheService cacheService;
}

// ✅ Transformação consistente
return result.fold(
  (error) => Left(UserServerError(message: error.message)),
  (response) => Right(UserModel.fromMap(response.data)),
);

// ✅ Tratamento granular de erros HTTP
Either<IUserFailure, T> _mapHttpErrorToFailure<T>(HttpErrorResponse error) {
  switch (error.statusCode) {
    case 400: return Left(UpdateUserDataError(message: error.message));
    case 401: return Left(UserUnauthenticatedError(message: error.message));
    case 404: return Left(UserNotFoundError(message: error.message));
    default: return Left(UserServerError(message: error.message));
  }
}
```

### ❌ Evitar

```dart
// ❌ Dependências de implementações concretas
class UserRepository extends IUserRepository {
  final UserDatasource datasource; // implementação concreta
  final DioHttpClient httpClient;  // tecnologia específica
}

// ❌ Regras de negócio no repository
@override
Future<Either<IUserFailure, UserEntity>> updateUser() async {
  if (!user.isAdult) { // regra de negócio - pertence ao UseCase
    return Left(UserValidationError());
  }
  return datasource.updateUser();
}

// ❌ Acesso direto a tecnologias
@override
Future<Either<IUserFailure, UserEntity>> getUser() async {
  final response = await dio.get('/api/users'); // acesso direto
  return Right(UserEntity.fromMap(response.data));
}
```

---

Esta estrutura garante que as implementações de Repositories sejam **bem organizadas**, **resilientes** e **mantenham a separação de responsabilidades** da Clean Architecture! 🎯
