# Data DataSources (Implementações) - Clean Architecture

## 📚 Visão Geral

As **implementações de DataSources** na camada de **Data** definem **COMO** a comunicação real com fontes externas é executada. Elas implementam os contratos definidos na Infrastructure, lidando com protocolos específicos, serialização e comunicação de rede.

### 🎯 Princípios Fundamentais das Implementações DataSource

**O QUE as implementações FAZEM:**
- ✅ **Implementam Protocolos**: Executam comunicação real (HTTP, GraphQL, gRPC, DB)
- ✅ **Serializam Dados**: Convertem entre entities e formatos externos (JSON, XML)
- ✅ **Gerenciam Conexões**: Controlam timeouts, headers, autenticação
- ✅ **Executam I/O**: Fazem chamadas reais para APIs, banco de dados, cache
- ✅ **Otimizam Performance**: Connection pooling, compression, keep-alive

**O QUE as implementações NÃO FAZEM:**
- ❌ **Não contêm regras de negócio**: Apenas comunicação técnica
- ❌ **Não transformam para entities**: Retornam dados raw para infraestrutura
- ❌ **Não coordenam múltiplas fontes**: Foco em uma fonte específica
- ❌ **Não aplicam cache strategies**: Apenas executam operações diretas
- ❌ **Não tratam failures de domain**: Apenas erros técnicos de comunicação

### 🏗️ Localização e Estrutura

```
lib/src/data/datasources/
├── user_datasource.dart
├── product_datasource.dart
├── order_datasource.dart
└── notification_datasource.dart
```

---

## 🔍 Anatomia de um DataSource

### Estrutura Base

```dart
import 'package:base_core/base_core.dart' show Either, IHttpDriver, HttpDriverResponse;
import '../../core/utils/http_error_response.dart';
import '../../domain/entities/[entity]_entity.dart';
import '../../infra/datasources/i_[entity]_datasource.dart';
import '../../infra/models/[entity]_model.dart';

/// Implementação do datasource para comunicação com API de [Entity]
/// 
/// Esta classe executa a comunicação real com a API REST, lidando com
/// protocolos HTTP, serialização JSON e tratamento de erros de rede.
class [Entity]Datasource extends I[Entity]Datasource {
  [Entity]Datasource({required this.httpDriver});

  final IHttpDriver httpDriver;

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> get[Entity]() async {
    return httpDriver
        .get('/api/v1/[entity]')
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}
```

### Elementos Essenciais

1. **Herança da Interface**: Implementa contrato da Infrastructure
2. **Driver de Protocolo**: HTTP, Database, Cache driver específico
3. **Comunicação Real**: Chamadas reais para fontes externas
4. **Serialização**: Conversão de entities para formato de transporte
5. **Tratamento de Erros Técnicos**: Network, timeout, protocol errors

---

## 📚 Exemplo Prático: UserDatasource

### Implementação Real

```dart
import 'package:base_core/base_core.dart';

import '../../core/utils/http_error_response.dart';
import '../../domain/entities/user_entity.dart';
import '../../infra/datasources/i_user_datasource.dart';
import '../../infra/models/user_model.dart';

/// Implementação do datasource para comunicação com API de usuários
/// 
/// Esta classe executa a comunicação real com a API REST de usuários,
/// lidando com protocolos HTTP, serialização JSON e autenticação.
class UserDatasource extends IUserDatasource {
  UserDatasource({required this.httpDriver});

  final IHttpDriver httpDriver;

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser() async {
    return httpDriver
        .get('/api/v1/user/me')
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>>
      deleteUserAccount() async {
    return httpDriver
        .delete('/api/v1/user/account')
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
    required UserEntity data,
  }) {
    return httpDriver
        .patch(
          '/api/v1/user/${data.id}',
          data: UserModel.fromEntity(data).toMap,
        )
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  }) {
    return httpDriver
        .patch(
          '/api/v1/user/$id/password',
          data: {
            'newPassword': newPassword,
            'currentPassword': currentPassword,
          },
        )
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}
```

### Características da Implementação Real

✅ **Segue princípios SOLID:**
- Depende apenas de abstrações (`IHttpDriver`)
- Implementa interface específica (`IUserDatasource`)
- Responsabilidade única (comunicação HTTP com API de usuários)

✅ **Padrões de implementação:**
- Endpoints REST bem definidos
- Serialização via Model.fromEntity().toMap
- Tratamento padronizado de HttpErrorResponse

✅ **Características técnicas:**
- HTTP methods apropriados (GET, POST, PATCH, DELETE)
- URLs com parâmetros dinâmicos
- Serialização automática de dados

---

## 🎨 Padrões de Implementação

### 1. Implementação HTTP Simples
```dart
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser() async {
  return httpDriver
      .get('/api/v1/user/me')
      .then((value) => value.fold(HttpErrorResponse.left, Right));
}
```

### 2. Implementação com Dados de Entrada
```dart
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
  required UserEntity data,
}) {
  return httpDriver
      .patch(
        '/api/v1/user/${data.id}',
        data: UserModel.fromEntity(data).toMap,
      )
      .then((value) => value.fold(HttpErrorResponse.left, Right));
}
```

### 3. Implementação com Headers Customizados
```dart
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> uploadUserAvatar({
  required String userId,
  required File imageFile,
}) async {
  final formData = FormData.fromMap({
    'avatar': await MultipartFile.fromFile(
      imageFile.path,
      filename: 'avatar.jpg',
    ),
  });

  return httpDriver
      .post(
        '/api/v1/user/$userId/avatar',
        data: formData,
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      )
      .then((value) => value.fold(HttpErrorResponse.left, Right));
}
```

### 4. Implementação com Query Parameters
```dart
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> searchUsers({
  required String query,
  int? page,
  int? limit,
}) {
  final queryParams = <String, dynamic>{
    'q': query,
    if (page != null) 'page': page,
    if (limit != null) 'limit': limit,
  };

  return httpDriver
      .get(
        '/api/v1/users/search',
        queryParameters: queryParams,
      )
      .then((value) => value.fold(HttpErrorResponse.left, Right));
}
```

### 5. Implementação com Cache Local
```dart
class UserCacheDataSource extends IUserCacheDataSource {
  UserCacheDataSource({required this.localStorageDriver});

  final ILocalStorageDriver localStorageDriver;

  @override
  Future<Either<CacheErrorResponse, CacheDriverResponse>> getLoggedUser() async {
    try {
      final cachedData = await localStorageDriver.get('logged_user');
      
      if (cachedData != null) {
        return Right(CacheDriverResponse(data: cachedData));
      } else {
        return Left(CacheErrorResponse(message: 'User not found in cache'));
      }
    } catch (exception) {
      return Left(CacheErrorResponse(message: 'Cache read error: $exception'));
    }
  }

  @override
  Future<Either<CacheErrorResponse, CacheDriverResponse>> saveLoggedUser({
    required UserEntity user,
  }) async {
    try {
      final userData = UserModel.fromEntity(user).toMap;
      await localStorageDriver.set('logged_user', userData);
      
      return Right(CacheDriverResponse(data: userData));
    } catch (exception) {
      return Left(CacheErrorResponse(message: 'Cache write error: $exception'));
    }
  }
}
```

### 6. Implementação com Database
```dart
class UserDatabaseDataSource extends IUserDatabaseDataSource {
  UserDatabaseDataSource({required this.databaseDriver});

  final IDatabaseDriver databaseDriver;

  @override
  Future<Either<DatabaseErrorResponse, DatabaseDriverResponse>> getUser({
    required String id,
  }) async {
    try {
      final result = await databaseDriver.query(
        'SELECT * FROM users WHERE id = ?',
        [id],
      );

      if (result.isNotEmpty) {
        return Right(DatabaseDriverResponse(data: result.first));
      } else {
        return Left(DatabaseErrorResponse(message: 'User not found'));
      }
    } catch (exception) {
      return Left(DatabaseErrorResponse(message: 'Database error: $exception'));
    }
  }

  @override
  Future<Either<DatabaseErrorResponse, DatabaseDriverResponse>> saveUser({
    required UserEntity data,
  }) async {
    try {
      final userMap = UserModel.fromEntity(data).toMap;
      
      await databaseDriver.insert(
        'users',
        userMap,
        conflictAlgorithm: ConflictAlgorithm.replace,
      );

      return Right(DatabaseDriverResponse(data: userMap));
    } catch (exception) {
      return Left(DatabaseErrorResponse(message: 'Database save error: $exception'));
    }
  }
}
```

### 7. Implementação com GraphQL
```dart
class UserGraphQLDataSource extends IUserGraphQLDataSource {
  UserGraphQLDataSource({required this.graphqlDriver});

  final IGraphQLDriver graphqlDriver;

  @override
  Future<Either<GraphQLErrorResponse, GraphQLDriverResponse>> getUser({
    required String id,
  }) {
    const query = '''
      query GetUser(\$id: ID!) {
        user(id: \$id) {
          id
          name
          email
          createdAt
          updatedAt
        }
      }
    ''';

    return graphqlDriver
        .query(
          query,
          variables: {'id': id},
        )
        .then((value) => value.fold(GraphQLErrorResponse.left, Right));
  }

  @override
  Future<Either<GraphQLErrorResponse, GraphQLDriverResponse>> updateUser({
    required UserEntity data,
  }) {
    const mutation = '''
      mutation UpdateUser(\$id: ID!, \$input: UserInput!) {
        updateUser(id: \$id, input: \$input) {
          id
          name
          email
          updatedAt
        }
      }
    ''';

    final variables = {
      'id': data.id,
      'input': UserModel.fromEntity(data).toMap,
    };

    return graphqlDriver
        .mutate(
          mutation,
          variables: variables,
        )
        .then((value) => value.fold(GraphQLErrorResponse.left, Right));
  }
}
```

---

## 📋 Template para Implementações DataSource

### Estrutura Básica (HTTP)

```dart
import 'package:base_core/base_core.dart' show Either, IHttpDriver, HttpDriverResponse;
import '../../core/utils/http_error_response.dart';
import '../../domain/entities/[entity]_entity.dart';
import '../../infra/datasources/i_[entity]_datasource.dart';
import '../../infra/models/[entity]_model.dart';

/// Implementação do datasource para comunicação com API de [Entity]
/// 
/// Esta classe executa a comunicação real com a API REST de [Entity],
/// lidando com protocolos HTTP, serialização JSON e tratamento de erros de rede.
class [Entity]Datasource extends I[Entity]Datasource {
  [Entity]Datasource({required this.httpDriver});

  final IHttpDriver httpDriver;

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> get[Entity]({
    required String id,
  }) async {
    return httpDriver
        .get('/api/v1/[entity]/$id')
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> create[Entity]({
    required [Entity]Entity data,
  }) {
    return httpDriver
        .post(
          '/api/v1/[entity]',
          data: [Entity]Model.fromEntity(data).toMap,
        )
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}
```

### Convenções de Implementação

**Nomenclatura:**
- Classe: `[Entity]Datasource extends I[Entity]Datasource`
- Arquivo: `[entity]_datasource.dart`

**Estrutura:**
- Construtor com injeção de driver específico
- Override de todos os métodos da interface
- Padrão de tratamento de erro consistente
- URLs e endpoints bem organizados

**Responsabilidades:**
- Executar comunicação real com fontes externas
- Serializar dados para formatos de transporte
- Gerenciar protocolos específicos
- Tratar erros técnicos de comunicação

---

## 📋 Checklist para Implementações DataSource

### Checklist de Criação ✅

**Estrutura da Classe:**
- [ ] Localizada em `lib/src/data/datasources/`
- [ ] Nome seguindo padrão `[Entity]Datasource`
- [ ] Herda da interface `I[Entity]Datasource`
- [ ] Construtor com injeção de driver apropriado
- [ ] Apenas dependencies de drivers abstratos

**Comunicação Externa:**
- [ ] Implementação de todos os métodos da interface
- [ ] URLs/endpoints bem definidos e consistentes
- [ ] HTTP methods apropriados para cada operação
- [ ] Headers necessários quando apropriado
- [ ] Query parameters para filtros e paginação

**Serialização de Dados:**
- [ ] Model.fromEntity() para dados de saída
- [ ] Estruturas JSON apropriadas para API
- [ ] Tratamento de formatos especiais (FormData, etc.)
- [ ] Validação de dados antes do envio

**Tratamento de Erros:**
- [ ] Padrão Either para todos os retornos
- [ ] Mapeamento de erros técnicos específicos
- [ ] Tratamento de timeouts e erros de rede
- [ ] Logs apropriados para debugging

**Padrões de Qualidade:**
- [ ] Documentação clara da classe e protocolo
- [ ] Métodos bem documentados com endpoints
- [ ] Tratamento consistente de erros
- [ ] Testes de integração correspondentes

---

## 🎯 Diretrizes para Implementações

### ✅ Boas Práticas

```dart
// ✅ URLs bem estruturadas
class UserDatasource extends IUserDatasource {
  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUser({
    required String id,
  }) async {
    return httpDriver
        .get('/api/v1/users/$id')  // URL RESTful clara
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}

// ✅ Serialização consistente
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({
  required UserEntity data,
}) {
  return httpDriver
      .post(
        '/api/v1/users',
        data: UserModel.fromEntity(data).toMap,  // Serialização padronizada
      )
      .then((value) => value.fold(HttpErrorResponse.left, Right));
}

// ✅ Headers quando necessário
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> uploadFile({
  required File file,
}) {
  return httpDriver
      .post(
        '/api/v1/upload',
        data: formData,
        headers: {'Content-Type': 'multipart/form-data'},
      )
      .then((value) => value.fold(HttpErrorResponse.left, Right));
}
```

### ❌ Evitar

```dart
// ❌ URLs inconsistentes
return httpDriver.get('/user_data');  // não RESTful
return httpDriver.get('/api/getUserById/$id');  // mistura padrões

// ❌ Serialização manual
return httpDriver.post('/api/users', data: {
  'name': data.name,
  'email': data.email,  // serialização manual propensa a erros
});

// ❌ Lógica de negócio no datasource
@override
Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({
  required UserEntity data,
}) {
  if (!data.isValid) {  // validação de negócio - pertence ao UseCase
    throw Exception('Invalid user');
  }
  return httpDriver.post('/api/users', data: data.toMap);
}

// ❌ Dependências diretas de tecnologia
class UserDatasource extends IUserDatasource {
  UserDatasource({required this.dio});  // dependência direta
  final Dio dio;
}
```

---

Esta estrutura garante que as implementações de DataSources sejam **eficientes**, **confiáveis** e **mantenham a separação clara** entre comunicação técnica e lógica de negócio! 🎯
