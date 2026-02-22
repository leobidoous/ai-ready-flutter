# DataSources (Data)

## O que são?

DataSources são as **implementações concretas** das interfaces de fonte de dados definidas na Infra. Eles executam a comunicação real com APIs, bancos de dados ou cache, usando Drivers.

## Responsabilidades

- Implementar a interface `IUserDatasource` da Infra
- Executar requisições HTTP usando `IHttpDriver`
- Transformar Entities em dados para envio (usando Models)
- Retornar `Either<HttpErrorResponse, HttpDriverResponse>`
- Não tratar erros de negócio (isso é responsabilidade do Repository)

## Estrutura

```dart
import 'package:base_core/base_core.dart';

import '../../core/utils/http_error_response.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/entities/user_filters_entity.dart';
import '../../infra/datasources/i_user_datasource.dart';
import '../../infra/models/user_filters_model.dart';
import '../../infra/models/user_model.dart';

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
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
    required UserEntity data,
  }) async {
    final formData = FormData.fromMap(
      await UserModel.fromEntity(data).toUpdateAsync,
    );
    return httpDriver
        .patch('/api/v1/user/me', data: formData)
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

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> fetchUsers({
    required UserFiltersEntity filters,
  }) {
    return httpDriver
        .get(
          '/api/v1/user/list',
          queryParameters: UserFiltersModel.fromEntity(filters).toMap,
        )
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({
    required UserEntity data,
  }) {
    return httpDriver
        .post('/api/v1/user', data: UserModel.fromEntity(data).toCreate)
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}
```

## Características

### 1. Injeção do Driver

```dart
UserDatasource({required this.httpDriver});

final IHttpDriver httpDriver;
```

- Recebe `IHttpDriver` para executar requisições HTTP
- Não conhece implementação concreta (Dio, http, etc.)

### 2. Métodos HTTP

#### GET - Buscar dados

```dart
return httpDriver
    .get('/api/v1/user/me')
    .then((value) => value.fold(HttpErrorResponse.left, Right));
```

#### GET com Query Parameters

```dart
return httpDriver
    .get(
      '/api/v1/user/list',
      queryParameters: UserFiltersModel.fromEntity(filters).toMap,
    )
    .then((value) => value.fold(HttpErrorResponse.left, Right));
```

#### POST - Criar dados

```dart
return httpDriver
    .post('/api/v1/user', data: UserModel.fromEntity(data).toCreate)
    .then((value) => value.fold(HttpErrorResponse.left, Right));
```

#### PATCH - Atualizar dados

```dart
return httpDriver
    .patch('/api/v1/user/me', data: formData)
    .then((value) => value.fold(HttpErrorResponse.left, Right));
```

#### DELETE - Remover dados

```dart
return httpDriver
    .delete('/api/v1/user/account')
    .then((value) => value.fold(HttpErrorResponse.left, Right));
```

### 3. Transformação de Dados para Envio

#### Dados Simples (Map)

```dart
data: {
  'newPassword': newPassword,
  'currentPassword': currentPassword,
}
```

#### Usando Model - toCreate

```dart
data: UserModel.fromEntity(data).toCreate
```

#### Usando Model - toUpdateSync

```dart
data: UserModel.fromEntity(data).toUpdateSync
```

#### Usando Model - toUpdateAsync (com arquivos)

```dart
final formData = FormData.fromMap(
  await UserModel.fromEntity(data).toUpdateAsync,
);
```

### 4. Tratamento de Resposta

```dart
.then((value) => value.fold(HttpErrorResponse.left, Right))
```

- Transforma erro do Driver em `HttpErrorResponse`
- Mantém sucesso como `HttpDriverResponse`
- Não trata erros de negócio (Repository faz isso)

## Fluxo de Dados

```
Entity → Model → HTTP → API
  ↓       ↓       ↓      ↓
Data   toCreate  POST  Server
```

### Envio:

1. **Entity** → Recebida do Repository
2. **Model** → `UserModel.fromEntity(data)`
3. **Map/FormData** → `.toCreate`, `.toUpdateSync`, `.toUpdateAsync`
4. **HTTP** → `httpDriver.post()`, `.patch()`, etc.

### Retorno:

1. **API** → Resposta do servidor
2. **HttpDriver** → `Either<HttpDriverError, HttpDriverResponse>`
3. **DataSource** → `Either<HttpErrorResponse, HttpDriverResponse>`
4. **Repository** → Transforma em `Either<Failure, Entity>`

## Benefícios

- **Isolamento**: Lógica de comunicação separada da lógica de negócio
- **Testabilidade**: Fácil mockar HttpDriver
- **Flexibilidade**: Trocar implementação do Driver sem afetar outras camadas
- **Clareza**: Cada método representa uma operação de API
- **Reutilização**: Models centralizam transformação de dados
