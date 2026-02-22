# Drivers (Data)

## O que são?

Drivers são **abstrações de bibliotecas externas** que executam operações de baixo nível como requisições HTTP, acesso a banco de dados, cache, etc. Eles isolam a aplicação de dependências específicas.

## Responsabilidades

- Executar operações de comunicação (HTTP, DB, Cache)
- Fornecer interface padronizada para a aplicação
- Gerenciar configurações (headers, tokens, base URL)
- Retornar respostas padronizadas
- Não conhecer regras de negócio

## IHttpDriver

Interface que define o contrato para comunicação HTTP. Implementações comuns usam Dio, http, etc.

### Estrutura

```dart
abstract class IHttpDriver {
  Future<Either<HttpDriverResponse, HttpDriverResponse>> get(
    String path, {
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
    HttpDriverProgressCallback? onReceiveProgress,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> post(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
    HttpDriverProgressCallback? onReceiveProgress,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> patch(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
    HttpDriverProgressCallback? onReceiveProgress,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> put(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
    HttpDriverProgressCallback? onReceiveProgress,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> delete(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> sendFile(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
    HttpDriverProgressCallback? onReceiveProgress,
    HttpDriverProgressCallback? onSendProgress,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> getFile(
    String path, {
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
  });

  IHttpDriver copyWith();
  void resetContentType();
  Future<dynamic> interceptRequests(Future request);
  Map<String, String>? get getHeaders;
}
```

### HttpDriverResponse

```dart
class HttpDriverResponse {
  HttpDriverResponse({
    required this.data,
    this.statusCode,
    this.statusMessage,
  });

  final int? statusCode;
  String? statusMessage;
  final dynamic data;
}
```

### HttpDriverOptions

```dart
class HttpDriverOptions {
  HttpDriverOptions({
    this.apiKey,
    this.baseUrl,
    this.tenantId,
    this.channelId,
    this.customerId,
    this.contentType,
    this.accessToken,
    this.responseType,
    this.extraHeaders,
    this.apiMapKey = 'apiKey',
    this.accessTokenType = 'Bearer',
  });

  final String? apiKey;
  final String? tenantId;
  final String? apiMapKey;
  final String? channelId;
  final String? contentType;
  final HttpBaseUrl? baseUrl;
  final String accessTokenType;
  final ResponseType? responseType;
  final HttpAccessToken? accessToken;
  final HttpCustomerId? customerId;
  final Map<String, dynamic>? extraHeaders;
}
```

## Métodos HTTP

### GET - Buscar dados

```dart
await httpDriver.get(
  '/api/v1/user/me',
  queryParameters: {'page': 1, 'limit': 10},
);
```

### POST - Criar dados

```dart
await httpDriver.post(
  '/api/v1/user',
  data: {'name': 'João', 'email': 'joao@email.com'},
);
```

### PATCH - Atualizar parcialmente

```dart
await httpDriver.patch(
  '/api/v1/user/123',
  data: {'name': 'João Silva'},
);
```

### PUT - Atualizar completamente

```dart
await httpDriver.put(
  '/api/v1/user/123',
  data: {'name': 'João', 'email': 'joao@email.com'},
);
```

### DELETE - Remover dados

```dart
await httpDriver.delete('/api/v1/user/123');
```

### SEND FILE - Enviar arquivos

```dart
await httpDriver.sendFile(
  '/api/v1/user/avatar',
  data: formData,
  onSendProgress: (count, total) {
    print('${(count / total * 100).toStringAsFixed(0)}%');
  },
);
```

### GET FILE - Baixar arquivos

```dart
await httpDriver.getFile(
  '/api/v1/documents/123',
  options: HttpDriverOptions(responseType: ResponseType.bytes),
);
```

## Configurações

### Base URL

```dart
HttpDriverOptions(
  baseUrl: () => 'https://api.example.com',
)
```

### Access Token

```dart
HttpDriverOptions(
  accessToken: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
  accessTokenType: 'Bearer',
)
```

### Headers Customizados

```dart
HttpDriverOptions(
  extraHeaders: {
    'X-Custom-Header': 'value',
    'X-Tenant-Id': 'tenant123',
  },
)
```

### Content Type

```dart
HttpDriverOptions(
  contentType: 'application/json',
)
```

## Outros Drivers

### ILocalStorageDriver

- Armazenamento local (SharedPreferences, Hive)
- Métodos: `save()`, `get()`, `delete()`, `clear()`

### ICacheDriver

- Cache em memória ou disco
- Métodos: `set()`, `get()`, `delete()`, `clear()`

### IDatabaseDriver

- Acesso a banco de dados local (SQLite, Realm)
- Métodos: `insert()`, `query()`, `update()`, `delete()`

## Benefícios

- **Isolamento**: Aplicação não depende de biblioteca específica
- **Testabilidade**: Fácil mockar Drivers em testes
- **Flexibilidade**: Trocar implementação sem afetar outras camadas
- **Padronização**: Interface única para diferentes implementações
- **Manutenibilidade**: Mudanças em bibliotecas externas ficam isoladas
